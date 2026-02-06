---
title: Proyecto 13 - Control del motor y debouncing
date: 2026-02-06
draft: false
---

## Control del motor en Arduino

En la clase de hoy hemos empezado un proyecto que consta de dos partes.

* **Parte A:** verificación del control de potencia sobre un **motor DC** con un montaje sencillo.
* **Parte B:** terminar el proyecto añadiendo **pulsadores** (acelerador/freno) e **implementando “debounce”** para evitar rebotes.

Antes de nada, decir que el proyecto se puede alimentar con los **5V** de la placa de Arduino, o con una **pila de 9V** (la típica del kit). Si usas la pila, necesitas el **clip/conector** de 9 V (vas a tener que pelar dos cables) para sacar **+** y **−**.

Los componentes que vamos a utilizar para hacer funcionar el motor son los siguientes:

* **Diodo** (la raya marca el **cátodo**).
* **Resistencia** de **220 Ω**
* **Transistor NPN**
* **Motor DC** (el gris).

### Montaje

El montaje es el típico para activar una carga con un **transistor NPN** (conmutación en el lado de masa):

* El transistor del kit es **NPN**.
* Mirando el transistor desde la **parte plana**, las patas suelen ser: **emisor (E)**, **base (B)** y **colector (C)**.
* La **base** se conecta a un **pin PWM** (los que tienen el símbolo **~**) **a través de** una resistencia de **220Ω**.
* El **colector** se conecta al cable **negro** del motor.
* El otro cable del motor va a **5V**.
* El **emisor** del transistor va a **GND** (tierra).
* El **diodo** se coloca **en paralelo con el motor** como protección frente a picos de tensión:

	- **Cátodo** (la raya) a **5V**.
	- **Ánodo** al lado del motor que va al **colector** (lado “−” del motor).

> Nota importante: el diodo de protección es imprescindible porque el motor es una carga inductiva. Si se corta la corriente de golpe, puede aparecer un pico de tensión que dañe el transistor o el Arduino.

El código que he creado pregunta al usuario:

1. **Cuántas velocidades distintas** quiere (número de “marchas”).
2. **Cuántos segundos** quiere entre cambios de velocidad.

Luego el programa empieza en la velocidad más baja y va subiendo cada *X* segundos hasta la máxima velocidad, y vuelve a empezar. Además, en el puerto serie se muestra la velocidad actual.

## Código (Parte A)

```c
#define PIN_MOTOR 3

int contador = 1;
int velocidades = 0;
int velocidad = 0;
int segundos = 0;

void setup() {
  Serial.begin(9600);
  pinMode(PIN_MOTOR, OUTPUT);

  // Pedir número de velocidades
  Serial.print("Cuantas velocidades quieres?: ");
  while (Serial.available() == 0) {
    // Espera a que el usuario escriba
  }
  velocidades = Serial.parseInt();

  // Vaciar el buffer (resto de caracteres como '\n')
  while (Serial.available() > 0) {
    Serial.read();
  }

  // Pedir segundos entre velocidades
  Serial.print("Cuantos segundos quieres entre cada velocidad?: ");
  while (Serial.available() == 0) {
    // Espera a que el usuario escriba
  }
  segundos = Serial.parseInt();

  while (Serial.available() > 0) {
    Serial.read();
  }

  if (velocidades < 1) velocidades = 1;
  if (segundos < 0) segundos = 0;

  Serial.print("\nVelocidades (marchas): ");
  Serial.println(velocidades);
  Serial.print("Segundos por marcha: ");
  Serial.println(segundos);
}

void loop() {
  for (int marcha = 1; marcha <= velocidades; marcha++) {
    int pwm = map(marcha, 1, velocidades, 0, 255);

    analogWrite(PIN_MOTOR, pwm);

    Serial.print("Marcha ");
    Serial.print(marcha);
    Serial.print(" -> PWM: ");
    Serial.println(pwm);

    delay(segundos * 1000);
  }
}
```

