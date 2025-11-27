---
title: Proyecto 10 - Reloj de arena digital
date: 2025-11-27
draft: false
---

## Reloj de arena básico en Arduino

Hoy hemos visto cómo montar un reloj de arena en Arduino. Para el proyecto de hoy vamos a utilizar los siguientes componentes:

- 6 **LEDs** (a poder ser del mismo color)
- 6 resistencias de **220 Ω** para cada LED (si son del mismo color podemos utilizar solo una resistencia común)
- 1 resistencia de **10 kΩ** en caso de no querer usar el _pull-up_ interno
- 1 sensor de inclinación

Como vemos en los componentes, el nuevo componente de hardware del que vamos a aprender hoy es el sensor de inclinación, que básicamente es una pieza que dentro tiene una bolita que hará o no contacto. En mi caso, como utilizo el _pull-up_ interno, si hay contacto dará un `0` (0V) y si hay contacto, es decir, si volteo el sensor, dará un `1` (5V).

El circuito es realmente algo muy sencillo. Simplemente tendremos 6 LEDs conectados con una o varias resistencias dependiendo de si son o no del mismo color, con cada LED a su pin digital, y luego por otro lado el sensor de inclinación conectado por un lado a tierra y por otro a un pin digital, a través del cual leeremos el estado del sensor.

En este proyecto vamos a introducir una nueva característica para reemplazar el mítico `delay()`. Para ello vamos a hacer uso de la función `millis()` ya incluida por defecto en el IDE de Arduino. Lo que hace realmente es devolver, cada vez que la llamamos, el número de milisegundos que han pasado desde que se encendió o se reseteó la placa.

La manera de hacer "delays" con `millis()` consiste en comparar el tiempo que ha pasado. Por ejemplo, si han pasado ya 3000 milisegundos, es decir, 3 segundos, pues ejecuto X acción. Pero claro, vamos a tener que usar referencias temporales para saber cuánto tiempo ha pasado ya; si ponemos directamente en la condición `3000`, se cumplirá una vez, pero luego el tiempo seguirá subiendo.

El código es el siguiente:

```c++
// PINS Leds usados:
// 13, 12, 11, 10, 9, 8
// PIN sensor de inclinación: 2

#define SENSOR 2
#define MAX_PIN_LED 13
#define MIN_PIN_LED 8

unsigned long referencia_temporal = 0;

// Abrimos el puerto Serie, establecemos todos los leds en OUTPUT
// Y activamos la resistencia interna con INPUT_PULLUP para el sensor
void setup() {
  Serial.begin(9600);
  pinMode(SENSOR, INPUT_PULLUP);

  for (int i = MIN_PIN_LED; i <= MAX_PIN_LED; i++) {
    pinMode(i, OUTPUT);
  }
}

void loop() {
  unsigned long lectura = millis();
  bool estadoSensor = digitalRead(SENSOR);
  gestorLeds(lectura, estadoSensor);
}

void gestorLeds(unsigned long lectura, bool estado) {
  // Si no hay contacto en el sensor de inclinación apago todos
  // los leds y vuelvo a empezar
  if (estado == 1) {
    apagadoLeds(6);
    referencia_temporal = lectura;
    return;
  }

  unsigned long t = lectura - referencia_temporal;
  int led = t / 3000;

  // Comentario para ver como va todo
  Serial.println("Milisegundos: " + String(lectura) + "\tLeds encendidos: " + String(led) + "\tEstado del sensor de inclinacion: " + String(estado));

  if (led >= 1 && led <= 6) {
    encendidoLeds(led);
  }
}

void encendidoLeds(int num_leds) {
  for (int i = 8; i < MIN_PIN_LED + num_leds; i++) {
    digitalWrite(i, HIGH);
  }
}

void apagadoLeds(int num_leds) {
  for (int i = MIN_PIN_LED; i <= MAX_PIN_LED; i++) {
    digitalWrite(i, LOW);
  }
}

```

Como podéis ver, el código está muy organizado en distintas funciones, que es la mejor manera de trabajarlo, ya que cuando escalemos el programa no se nos hará tan complicado de mantenerlo. De este código la opción quizás más interesante es la de `gestorLeds()`, que será la que encienda los LEDs cada 3 segundos.

Esto se hace de la siguiente manera: Al principio del código he declarado una variable que guarda una referencia de tiempo, que al ejecutar el programa será de `0`. Luego, cuando el sensor está en la posición de "reposo" (sin contacto, lectura `1`, ya que estamos en la pull-up interna), igualamos `referencia_temporal` al valor actual que devuelva la función `millis()`. Una vez tenemos eso, crearemos dos variables más: una que será la que guarde la lectura en tiempo real menos nuestra referencia, y luego en otra variable ese resultado entre `3000` para que se trunquen los decimales y saber la cantidad de LEDs que hay que encender, si solo 1, 2 o hasta 6, que en nuestro caso será el máximo que tenemos. Entre medias también hay un pequeño **print** con información que nos puede ayudar a ver el transcurso del programa.

Por último, tenemos una función para apagar todos los LEDs y otra que enciende _x_ número de LEDs.

