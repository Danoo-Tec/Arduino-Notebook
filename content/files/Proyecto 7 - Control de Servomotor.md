---
title: Proyecto 7 - Control de Servomotor
date: 2025-11-06
draft: false
---
## Utilizando un servomotor

En la clase de hoy hemos visto **cómo** trabajar con un **servomotor**, que básicamente es un pequeño motor con realimentación de posición. Tiene tres cables: uno a **5 V**, otro a **GND** (tierra) y, por último, otro que va conectado a un **pin digital** en la placa (no es necesario que sea **PWM**). Este servomotor va a girar a **X** grados, los cuales vamos a poder controlar: los valores mínimo y máximo en grados suelen ser de **0º a 180º**. Para controlar este motor vamos a implementar un **potenciómetro**, ya usado en proyectos anteriores, para leer su valor por un **pin analógico** y, dependiendo de él, girar el servomotor a X grados. Los grados finales los vamos a **mapear** haciendo uso de la función `map()`, a la cual le pasaremos el valor leído por el pin analógico (entre 0 y 1023) y luego el valor final que queremos (entre 0 y 180).

Lo más novedoso de este proyecto es **cómo** le vamos a indicar a este servomotor **cuántos** grados queremos que se mueva. Esto lo vamos a hacer incluyendo la librería `Servo.h`, la cual nos va a permitir definir el nombre de nuestro servomotor y asignarle un pin, además de poder modificar los valores por defecto de los pulsos (en microsegundos) que interpreta, que por defecto están entre **544** y **2400 µs**. Por último, vamos a usar el nombre de nuestro servo —en mi caso `motor`— con el método `write`: `motor.write(valor)`. Ese `valor` es la cantidad de grados que queremos que se mueva, entre 0 y 180. Una pequeña **corrección** que se puede hacer es aumentar el valor máximo por defecto por si no gira lo suficiente o no llega a esos **180º**.

El código de este proyecto va a ser el siguiente:

```c
#include <Servo.h>

#define PIN_SERVO 6
#define PIN_ANALOG A0

Servo motor;

int valor = 0;

void setup() {
  Serial.begin(9600);
  motor.attach(PIN_SERVO, 544, 2400);
}

void loop() {
  delay(1);
  valor = map(analogRead(PIN_ANALOG), 0, 1023, 0, 180);
  Serial.println("Ángulo mandado: " + String(valor));
  motor.write(valor);
}
```

### Teoría adicional sobre los timers y compatibilidades
Los **timers** son contadores hardware que marcan tiempos, generan **PWM** y soportan funciones como `millis()`/`micros()`. Varias librerías los usan; si una “toma” un timer, las **salidas PWM** vinculadas a ese timer pueden quedar **inhabilitadas** o cambiar de frecuencia.

Mapa rápido de los timers:
- **Timer0 (8‑bit)** → `millis()/micros()/delay()`. **PWM en 5 y 6**. **No tocar** su configuración o romperás el tiempo del sistema.
- **Timer1 (16‑bit)** → lo usa `Servo.h`. **PWM en 9 y 10**. Cuando `Servo.h` está activa, **9/10 no dan PWM** con `analogWrite()`.
- **Timer2 (8‑bit)** → lo usa `tone()` (y a veces IR). **PWM en 3 y 11**. Mientras suena un tono, **3/11 pierden PWM**.

## Creando nuestra propia librería para el servomotor

Respecto del código anterior en el que **utilizábamos** la librería `Servo.h`, ahora vamos a intentar crear, entre comillas, **nuestra propia librería** para poder controlar el movimiento del servo a **X** grados. Esto lo vamos a lograr teniendo en cuenta que **1 ms** será equivalente, aproximadamente, a **0°** y **2 ms** a **180°**, y que cualquier valor intermedio corresponderá a un ángulo proporcional. Al igual que en el código anterior, controlaremos el giro del **servomotor** con un valor leído del **potenciómetro**; lo único que ahora, en vez de mapearlo para obtener un valor final entre **0 y 180°**, lo mapearemos para obtener un valor final entre **1 y 2 ms**.

El truco en esta función que vamos a usar es **encender** el servomotor durante _x_ microsegundos (pulso en **HIGH**) y, dependiendo de cuánto sea, fijar el ángulo final. Esto lo podemos hacer **repitiendo** muchas veces (por ejemplo, 20 veces): ponemos el pin en **HIGH**, luego en **LOW**, y esperamos entre medias esos microsegundos calculados a partir del valor mapeado del potenciómetro. Por último, completamos el **período** de **20 ms** restando al total del cuadro el tiempo del pulso (es decir, `20000 - pulso_us`), ya que el protocolo típico de servos utiliza un período cercano a **20 ms** (50 Hz).

El código de esta otra versión es el siguiente:

```c
#define PIN_SERVO 6
#define PIN_ANALOG A0

const int MIN_US = 1000;
const int MAX_US = 2000;
int valor = 0;

void setup() {
  Serial.begin(9600);
  pinMode(PIN_SERVO, OUTPUT);
  digitalWrite(PIN_SERVO, LOW);
}

void loop() {
  delay(1);
  valor = map(analogRead(PIN_ANALOG), 0, 1023, 0, 180);
  mi_servo(valor);
}

void mi_servo(int valor) {
  // Alternativa equivalente sin map():
  // int pulso_us = 1000 + ((float)valor * 1000.0 / 180.0);
  int pulso_us = map(valor, 0, 180, MIN_US, MAX_US);

  Serial.println("Ángulo mandado: " + String(valor) + "\t" + "Pulso (µs): " + String(pulso_us));

  for (int i = 0; i < 20; i++) {
    digitalWrite(PIN_SERVO, HIGH);
    delayMicroseconds(pulso_us);
    digitalWrite(PIN_SERVO, LOW);
    delayMicroseconds(20000 - pulso_us);
  }
}
```
