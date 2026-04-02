---
title: Proyecto 18 - Puente en H
date: 2026-03-20
draft: false
---

## Uso del puente en H para controlar la potencia y el sentido de un motor DC

En la clase de hoy hemos estado trabajando con un nuevo componente, el **puente en H**. En un motor, el sentido de la corriente determina el campo magnético que se crea en el rotor; por tanto, el sentido del giro vendrá dado por el sentido de la corriente.

El puente en H es un circuito integrado que permite controlar de manera eléctrica el sentido de la circulación de la corriente, por lo cual podremos seleccionar que la corriente circule en un sentido u otro y, por tanto, el sentido de giro del motor.

El esquema lógico incluye dos pares de interruptores, a través de los cuales se determina que un motor DC gire en un sentido u otro. Si todos los interruptores pueden controlarse de forma individual, también se puede forzar un cortocircuito en los bornes del motor como ayuda para frenarlo. El puente en H, además, tiene capacidad de controlar hasta **600 mA** con tensiones entre **4,5 y 36 VDC**. Está preparado para manejar **dos cargas reactivas**, incluyendo los **diodos** para evitar picos en la interrupción de las corrientes.

Para este puente en H, en Arduino utilizaremos el chip **L293D**, para el cual tenemos el siguiente **pinout**:

| Pin | Nombre | Función                        |
| --- | ------ | ------------------------------ |
| 1   | 1,2EN  | Habilita las salidas 1Y y 2Y   |
| 2   | 1A     | Entrada lógica para 1Y         |
| 3   | 1Y     | Salida 1                       |
| 4   | GND    | Tierra                         |
| 5   | GND    | Tierra                         |
| 6   | 2Y     | Salida 2                       |
| 7   | 2A     | Entrada lógica para 2Y         |
| 8   | VCC2   | Alimentación del motor / carga |
| 9   | 3,4EN  | Habilita las salidas 3Y y 4Y   |
| 10  | 3A     | Entrada lógica para 3Y         |
| 11  | 3Y     | Salida 3                       |
| 12  | GND    | Tierra                         |
| 13  | GND    | Tierra                         |
| 14  | 4Y     | Salida 4                       |
| 15  | 4A     | Entrada lógica para 4Y         |
| 16  | VCC1   | Alimentación lógica            |

> La numeración empieza donde está la muesca del chip: mirando el chip con la muesca arriba, el pin 1 queda arriba a la izquierda, y luego la numeración continúa en sentido antihorario.

> Los pines de **GND** son comunes, pero en la práctica conviene conectar **todos los pines de tierra** correspondientes para asegurar un funcionamiento correcto y una mejor disipación.

**Funcionamiento básico de cada canal:**

* Si **EN = 1**, la salida **Y** sigue a la entrada **A**.
* Si **A = 1**, entonces **Y = VCC2**.
* Si **A = 0**, entonces **Y = 0 V**.
* Si **EN = 0**, la salida queda en **alta impedancia**.

Para el montaje de este chip conectaremos cada pin a lo siguiente:

* El **pin 1**, habilitador, se conectará a un pin con señales **PWM (~)** para poder controlar qué porcentaje del tiempo se envía corriente, de forma que se pueda regular la velocidad del motor.
* Los pines de entrada **2** y **7** estarán conectados a dos pines digitales del Arduino, de forma que:
	- Si se pone **5 V** en el pin **2** y **0 V** en el pin **7**, el motor girará en un sentido.
	- Si se pone **5 V** en el pin **7** y **0 V** en el pin **2**, girará en el sentido contrario.
* Los pines de **tierra** irán a **GND**, el **pin 16** a **5 V** y el **pin 8** al positivo de la batería de **9 V** si tenemos una, o a una alimentación adecuada para el motor.

Perfecto, pues una vez que tenemos toda la teoría, vamos a por el proyecto que realizaremos hoy, el cual va a llevar los siguientes componentes:

* Chip **L293D** (*puente en H*)
* Motor **DC** (*a poder ser con el ventilador para conectarlo*)
* Batería de **9 V** (*con los 5 V del Arduino también podría funcionar en algunos casos, pero normalmente irá más lento o con menos fuerza*)
* Potenciómetro

El objetivo del proyecto es el siguiente: el usuario controlará el movimiento de un motor DC mediante un potenciómetro, de manera que en las zonas cercanas a la posición intermedia se parará el movimiento del motor.

Si desde esa posición central se mueve el potenciómetro hacia un extremo, el motor comenzará a girar en ese sentido, aumentando la velocidad hasta el máximo cuando llegue al final del recorrido.

Igual ocurrirá girando el potenciómetro en sentido contrario, girando el motor también en sentido contrario.

Para el **SW**, es decir, para el código de este proyecto, he utilizado varias funciones que me han permitido modularizar el código hasta un punto en el cual en la función `loop()` solo llamo a funciones, y lo hago en el siguiente orden:

* Primero leo el valor del potenciómetro.
* Acto seguido, dependiendo de dónde esté situado el potenciómetro, elijo el sentido de giro.
* Elijo la velocidad con el valor del potenciómetro y mapeándolo a los valores del pin PWM.
* Por último, meto potencia, es decir, enciendo el motor en un sentido y con una potencia determinada.

El código es el siguiente:

```c++
#include <Arduino.h>

#define PIN_ENABLE 10
#define PIN_1A 9
#define PIN_2A 8
#define PIN_POTEN A0

#define CENTRO 512
#define ZONA_MUERTA 25

bool sentido = 0;
bool sentido_anterior = 0;

int lectura_poten = 0;
int lectura_dividida = 0;
int velocidad = 0;
int velocidad_anterior = -1;

// Prototipos de funciones
void lee_potenciometro();
void elegir_sentido();
void elegir_velocidad();
void meter_potencia();

void setup() {
  Serial.begin(9600);

  pinMode(PIN_ENABLE, OUTPUT);
  pinMode(PIN_1A, OUTPUT);
  pinMode(PIN_2A, OUTPUT);

  analogWrite(PIN_ENABLE, 0);
  digitalWrite(PIN_1A, LOW);
  digitalWrite(PIN_2A, LOW);
}

void loop() {
  lee_potenciometro();
  elegir_sentido();
  elegir_velocidad();
  meter_potencia();
}

void lee_potenciometro() {
  lectura_poten = analogRead(PIN_POTEN);
}

void elegir_sentido() {
  if (lectura_poten < CENTRO) {
    sentido = 1;  // izquierda
  } else {
    sentido = 0;  // derecha
  }
}

void elegir_velocidad() {
  lectura_dividida = abs(lectura_poten - CENTRO);

  // Zona muerta de ±25 alrededor del centro
  if (lectura_dividida <= ZONA_MUERTA) {
    velocidad = 0;
  } else {
    velocidad = map(lectura_dividida, ZONA_MUERTA, CENTRO, 0, 255);
    velocidad = constrain(velocidad, 0, 255);  // Protección extra, buena práctica
  }
}

void meter_potencia() {
  if (velocidad == 0) {
    analogWrite(PIN_ENABLE, 0);
    digitalWrite(PIN_1A, LOW);
    digitalWrite(PIN_2A, LOW);

    if (velocidad != velocidad_anterior) {
      Serial.println("Motor parado");
    }
  } else {
    analogWrite(PIN_ENABLE, velocidad);

    if (sentido) {
      digitalWrite(PIN_1A, HIGH);
      digitalWrite(PIN_2A, LOW);

      if (velocidad != velocidad_anterior || sentido != sentido_anterior) {
        Serial.print("Velocidad establecida hacia la izquierda: ");
        Serial.println(velocidad);
      }
    } else {
      digitalWrite(PIN_1A, LOW);
      digitalWrite(PIN_2A, HIGH);

      if (velocidad != velocidad_anterior || sentido != sentido_anterior) {
        Serial.print("Velocidad establecida hacia la derecha: ");
        Serial.println(velocidad);
      }
    }
  }

  velocidad_anterior = velocidad;
  sentido_anterior = sentido;
}
```
