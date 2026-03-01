---
title: Proyecto 13 - Control del motor y debouncing
date: 2026-02-06
draft: false
---

## Control del motor en Arduino

En la clase de hoy hemos empezado un proyecto que consta de dos partes.

* **Parte A (06/02/2026):** verificación del control de potencia sobre un **motor DC** con un montaje sencillo. 
* **Parte B  (13/02/2026):** terminar el proyecto añadiendo **pulsadores** (acelerador/freno) e **implementando “debounce”** para evitar rebotes.

Antes de nada, decir que el proyecto se puede alimentar con los **5 V** de la placa de Arduino o con una **pila de 9 V** (la típica del kit). Si usas la pila, necesitas el **clip/conector** de 9 V (habrá que pelar dos cables) para sacar **+** y **−**.

Los componentes que vamos a utilizar para hacer funcionar el motor son los siguientes:

* **Diodo** (la raya marca el **cátodo**).
* **Resistencia** de **220 Ω** (para la base del transistor).
* **Transistor NPN** (por ejemplo, 2N2222).
* **Motor DC** (el gris del kit).

### Montaje

El montaje es el típico para activar una carga con un **transistor NPN** (conmutación en el lado de masa):

* El transistor del kit es **NPN**.
* Mirando el transistor desde la **parte plana**, las patas suelen ser: **emisor (E)**, **base (B)** y **colector (C)**.
* * El **emisor** del transistor va a **GND** (tierra).
* La **base** se conecta a un **pin PWM** (los que tienen el símbolo **~**) **a través de** una resistencia de **220 Ω**.
* El **colector** se conecta al cable negativo del motor.
* El otro cable del motor va a **5 V**.
* El **diodo** se coloca **en paralelo con el motor** como protección frente a picos de tensión:

  * **Cátodo** (la raya) a **5 V**.
  * **Ánodo** al lado del motor que va al **colector** (lado negativo del motor).

> Nota importante: el diodo de protección es imprescindible porque el motor es una carga inductiva. Si se corta la corriente de golpe, puede aparecer un pico de tensión elevado (V = L * di/dt) que dañe el transistor o el Arduino.

El código que he creado pregunta al usuario:

1. **Cuántas velocidades distintas** quiere.
2. **Cuántos segundos** quiere entre cambios de velocidad.

Luego el programa empieza en la velocidad más baja y va subiendo cada *X* segundos hasta la máxima velocidad, y vuelve a empezar. Además, en el puerto serie se muestra la velocidad actual.

El código es el siguiente:

```c
#include <Arduino.h> // No hace falta si se está trabajando con el IDE de Arduino

#define PIN_MOTOR 3

int contador_vueltas = 1;
int max_velocidades = 0;
int max_segundos = 0;

// Prototipos
long leer_numero();
void validar_rangos(int velocidades, int segundos);

void setup() {
  Serial.begin(9600);
  pinMode(PIN_MOTOR, OUTPUT);

  // Pedir número de velocidades
  Serial.print("Cuantas velocidades quieres?: ");
  max_velocidades = leer_numero();

  // Pedir segundos entre velocidades
  Serial.print("Cuantos segundos quieres entre cada velocidad?: ");
  max_segundos = leer_numero();

  // Valido los rangos
  validar_rangos(max_velocidades, max_segundos);

  // Imprimo información final:
  Serial.println("");
  Serial.print("Velocidades: ");
  Serial.println(max_velocidades);
  Serial.print("Segundos por velocidad: ");
  Serial.println(max_segundos);
  Serial.println("");
}

void loop() {
  // Imprimo la vuelta actual
  Serial.print("=== VUELTA ");
  Serial.print(contador_vueltas);
  Serial.println(" ===");

  for (int velocidad = 1; velocidad <= max_velocidades; velocidad++) {
    int pwm = map(velocidad, 1, max_velocidades, 0, 255);

    analogWrite(PIN_MOTOR, pwm);

    Serial.print("Velocidad: ");
    Serial.print(velocidad);
    Serial.print(" -> Valor PWM: ");
    Serial.println(pwm);

    delay(max_segundos * 1000);
  }

  contador_vueltas++; // Sumo una vuelta una vez completada
  Serial.println("");
}

long leer_numero() {
  while (!Serial.available());
  long num = Serial.parseInt();

  // Limpio el buffer
  while (Serial.available()) Serial.read();

  // Dejo salto de línea
  Serial.println("");

  return num;
}

void validar_rangos(int velocidades, int segundos) {
  if (velocidades < 1) {
    Serial.println("Has introducido una velocidad no válida");
    Serial.println("Por defecto se dejará en 6 velocidades");
    max_velocidades = 6;
  }

  if (segundos < 0) {
    Serial.println("Has introducido un número de segundos no válido");
    Serial.println("Por defecto se dejará en 1 segundo");
    max_segundos = 1;
  }
}
```

## Debouncing para los botones

Perfecto, una vez visto cómo podemos controlar la velocidad del motor, vamos a ver ahora cómo podemos implementar dos botones (uno que acelera y otro que frena) para que pueda subir o bajar de velocidad pulsando uno u otro. Para hacer esto vamos a necesitar implementar un sistema antirrebotes, ya que cuando pulsamos un botón no se produce una única transición limpia, sino múltiples cambios rápidos entre HIGH y LOW antes de estabilizarse.

Eso es a lo que llamamos **rebotes**, y es lo que vamos a intentar evitar con el siguiente programa.

Antes de ir directamente al código vamos a hablar de una palabra clave que he añadido al código, y esta es `static` justo antes de declarar una variable. Esta palabra, utilizándola al declarar una variable dentro de una función, nos permite que esa variable conserve su valor entre llamadas.

Luego otra cosa que he añadido es que he utilizado una **referencia** para poder modificar esa variable de estado anterior en la función de detección de flanco de subida.

Ahora bien, ¿qué es eso del flanco de subida? Consiste en detectar cuándo el botón ha pasado de estado apagado a encendido. Para lograr esto hay que saber qué estado es cada uno. En un montaje con **pull-down externo**, al no pulsar estará en **LOW** y al pulsar estará en **HIGH**. En cambio, con **INPUT_PULLUP** sería al revés.

Una vez sabemos esto, la comprobación es sencilla:

Habrá habido un cambio de apagado a encendido cuando el estado actual del botón esté en HIGH y el anterior en LOW (para pull-down externo).

Pero claro, al haber rebotes y poder leer *ceros* y *unos* durante unos milisegundos antes de que la señal sea estable, vamos a hacer uso de la función `millis()` para solo validar lecturas cada *15 ms*, que es un valor típico suficiente para evitar este tipo de rebotes.

Los dos botones que he instalado tienen el siguiente montaje (leerá sin pulsar = LOW, pulsado = HIGH):

* Está instalado atravesando la línea central de la protoboard.
* Por un lado, una pata va a **5 V**.
* La otra pata va a **GND** a través de una resistencia de **10 kΩ** (pull-down).
* En el nodo entre la resistencia y el pulsador conecto el cable al pin digital correspondiente.

Una vez explicado esto, ya podemos pasar al código.

```c++
#include <Arduino.h> // No hace falta si se está trabajando con el IDE de Arduino

#define PIN_MOTOR 3
#define PIN_BOTON_FRENO 8
#define PIN_BOTON_ACELERADOR 9

// Variables motor
int max_velocidades = 0;
int velocidad = 1; // Empezamos en la primera velocidad

// Variables botones
// Esperaremos 15 ms entre lecturas para evitar rebotes
unsigned long espera_lecturas_ms = 15;

unsigned long actual_ms = 0;
unsigned long pasado_ms = 0;

bool freno_estado = false;
bool acelerador_estado = false;

// Prototipos
long leer_numero();
void validar_rangos(int velocidades);
bool leer_freno();
bool leer_acelerador();
bool flanco_subida(bool estado_actual, bool &estado_anterior);
void cambiar_velocidad(int velocidad);

void setup() {
  Serial.begin(9600);

  pinMode(PIN_MOTOR, OUTPUT);
  pinMode(PIN_BOTON_FRENO, INPUT);
  pinMode(PIN_BOTON_ACELERADOR, INPUT);

  // Pedir número de velocidades
  Serial.print("Cuantas velocidades quieres?: ");
  max_velocidades = leer_numero();

  // Valido los rangos
  validar_rangos(max_velocidades);

  // Imprimo información final:
  Serial.println("");
  Serial.print("Velocidades: ");
  Serial.println(max_velocidades);

  Serial.println("Comenzando en velocidad 1...");
  cambiar_velocidad(1);

  Serial.println("Utilice los botones para cambiar de velocidad");
}

void loop() {
  actual_ms = millis(); // En cada vuelta del bucle actualizo el tiempo

  // Si han pasado los 15 ms leo los botones otra vez y cambio de velocidad
  if (actual_ms - pasado_ms > espera_lecturas_ms) {
    freno_estado = leer_freno();
    acelerador_estado = leer_acelerador();

    static bool freno_anterior = false;
    static bool acelerador_anterior  = false;

    bool freno_pulsado = flanco_subida(freno_estado, freno_anterior);
    bool acelerador_pulsado  = flanco_subida(acelerador_estado, acelerador_anterior);

    if (freno_pulsado) {
      if (velocidad > 1) {
        velocidad--;
        cambiar_velocidad(velocidad);
        Serial.print("Frenando a velocidad número: ");
        Serial.println(velocidad);
      } else {
        Serial.println("Ya te encuentras a la mínima velocidad");
      }
    }

    if (acelerador_pulsado) {
      if (velocidad < max_velocidades) {
        velocidad++;
        cambiar_velocidad(velocidad);
        Serial.print("Acelerando a velocidad número: ");
        Serial.println(velocidad);
      } else {
        Serial.println("Ya te encuentras a la máxima velocidad");
      }
    }

    pasado_ms = actual_ms;
  }
}

long leer_numero() {
  while (!Serial.available());
  long num = Serial.parseInt();

  // Limpio el buffer
  while (Serial.available()) Serial.read();

  // Dejo salto de línea
  Serial.println("");

  return num;
}

void validar_rangos(int velocidades) {
  if (velocidades < 1) {
    Serial.println("Has introducido una velocidad no válida");
    Serial.println("Por defecto se dejará en 6 velocidades");
    max_velocidades = 6;
  }
}

bool leer_freno() {
  return digitalRead(PIN_BOTON_FRENO);
}

bool leer_acelerador() {
  return digitalRead(PIN_BOTON_ACELERADOR);
}

bool flanco_subida(bool estado_actual, bool &estado_anterior) {
  bool evento = (estado_actual == true && estado_anterior == false); // LOW -> HIGH, 0 -> 1
  estado_anterior = estado_actual;
  return evento;
}

void cambiar_velocidad(int velocidad) {
  // OJO: Importante, aquí la primera velocidad será 0, es decir apagado el motor
  int pwm = map(velocidad, 1, max_velocidades, 0, 255);

  analogWrite(PIN_MOTOR, pwm);
}
```
