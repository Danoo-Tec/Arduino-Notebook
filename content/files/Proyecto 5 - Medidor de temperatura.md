---
title: Proyecto 5 - Medidor de temperatura
date: 2025-10-23
draft: false
---

Lo primero nuevo que vemos es un **NTC** (_Negative Temperature Coefficient_), es decir, un coeficiente de temperatura negativo. Se trata de un tipo de **termistor**, una resistencia que varía con la temperatura.

Se comporta de la siguiente manera:
- A baja temperatura, el **NTC** tiene más resistencia.
- A alta temperatura, su resistencia **disminuye.**
- Esta relación no es lineal, pero en un rango corto (por ejemplo, de **22 a 30 ºC**) podemos aproximarla con una recta

**Mini esquema:** Más calor **→** menos resistencia **→** más corriente **→** tensión más baja en el pin analógico.

Ahora bien, **Arduino** no mide resistencias directamente, solo puede leer tensión (voltios), por tanto tendremos que montar un circuito como el siguiente

```css
5V ──[ NTC ]───●───[ 10 kΩ ]─── GND
               │
	           A0
```

> La tensión en A0 dependerá de la resistencia del **NTC**:

Trabajar con kΩ en lugar de Ω se hace simplemente por comodidad, ya que es una escala más fácil de leer e interpretar.

En el programa, la **sensibilidad** se calcula como la _resistencia a 30 ºC menos la resistencia a 22 ºC (temperatura ambiente)_, dividido entre la diferencia de temperatura (8 ºC en este caso).

Este valor lo interpretamos como la **pendiente** de la recta que **une los puntos** de ambas resistencias con sus respectivas temperaturas. El **resultado** será **negativo**, lo que indica que al **aumentar la temperatura**, **disminuye la resistencia**.

El código quedaría así:

```c
#define LED_1 7
#define LED_2 6
#define LED_3 5
#define TEMP A0

float tension = 0;
float resistencia = 0;
float tempC = 0;

const float R22 = 10.46;   // kΩ a 22°C (temp ambiente)
const float R30 = 6.50;    // kΩ a 30°C (sensor calentado)
const float PENDIENTE = (R30 - R22) / 8.0;  // pendiente (kΩ/°C)

void setup() {
  Serial.begin(9600);

  // Iniciamos los tres leds en modo OUTPUT 
  // para poder entregar o bien 0V o 5V. 
  pinMode(LED_1, OUTPUT);
  pinMode(LED_2, OUTPUT);
  pinMode(LED_3, OUTPUT);

  // Iniciamos el pin analógico con INPUT
  pinMode(TEMP, INPUT);
}

void loop() {
  // Un pequeño delay antes de leer es buena práctica para añadir estabilidad al programa
  delay(1);

  // Calculamos la tensión multiplicando el valor leido por 5 que son los máximos voltios y 
  // todo ello entre 1023 que es el valor máximo que puede devolver la función analogRead
  tension = (analogRead(TEMP) * 5.0) / 1023.0;

  // Calculo la resistencia del NTC (en kΩ)
  // Fórmula correcta con NTC arriba y 10 kΩ abajo
  resistencia = (50.0 / tension) - 10.0;

  // Calculo la temperatura en °C usando la fórmula lineal
  tempC = calc_temp(resistencia);

  // Mostrar por consola los valores
  printValues();

  // Creamos estructura condicional para manejar
  // el encendido de los leds en base a la temp
  if (tempC < 23) {
    digitalWrite(LED_1, LOW);
    digitalWrite(LED_2, LOW);
    digitalWrite(LED_3, LOW);
  } else if (tempC >= 23 && tempC < 25) {
    digitalWrite(LED_1, HIGH);
    digitalWrite(LED_2, LOW);
    digitalWrite(LED_3, LOW);
  } else if (tempC >= 25 && tempC < 27) {
    digitalWrite(LED_1, HIGH);
    digitalWrite(LED_2, HIGH);
    digitalWrite(LED_3, LOW);
  } else if (tempC >= 27 && tempC < 29) {
    digitalWrite(LED_1, HIGH);
    digitalWrite(LED_2, HIGH);
    digitalWrite(LED_3, HIGH);
  } else if (tempC >= 29) {
    digitalWrite(LED_1, LOW);
    digitalWrite(LED_2, LOW);
    digitalWrite(LED_3, LOW);
    delay(200);
    digitalWrite(LED_1, HIGH);
    digitalWrite(LED_2, HIGH);
    digitalWrite(LED_3, HIGH); 
  }
}

// Creamos una función para el cálculo de la temp
float calc_temp(float resistencia) {
  // Aplicamos la fórmula lineal para obtener la temperatura
  // resistencia actual - resistencia temp ambiente, todo ello
  // entre la pendiente + 22.
  return 22.0 + (resistencia - R22) / PENDIENTE;
}

// Creamos una función que imprima los valores en bonito
void printValues() {
  Serial.print("V0 = ");
  Serial.print(tension, 3);
  Serial.print(" V   |   R = ");
  Serial.print(resistencia, 3);
  Serial.print(" kΩ   |   T ≈ ");
  Serial.print(tempC, 2);
  Serial.println(" °C");
}
```

**Pequeño resumen del proyecto:**

Este proyecto es un medidor de temperatura con Arduino. Utiliza un termistor NTC de 10 kΩ conectado en un divisor de tensión junto a una resistencia fija también de 10 kΩ.  Arduino mide la tensión en el punto medio y, a partir de esa tensión, calcula la resistencia del NTC con la fórmula **R = 50 / V₀ − 10.**

Después, aplico una ecuación lineal entre dos puntos conocidos (22 °C y 30 °C) para obtener la temperatura aproximada.  Según el valor, se encienden distintos LEDs: uno para 23–25 °C, dos para 25–27 °C, tres para 27–29 °C, y si pasa de 29 °C parpadean todos.  He organizado el código en funciones separadas: una que calcula la temperatura y otra que muestra los valores, lo que permite activar o desactivar fácilmente la salida por consola.

