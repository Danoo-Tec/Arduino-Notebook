---
title: Proyecto 15 - Registro de desplazamiento
date: 2026-02-27
draft: false
---

## Uso del registro de desplazamiento en Arduino (Entrada del usuario)

Como ya sabemos, el número de pines digitales que podemos utilizar en Arduino es limitado. En un Arduino UNO disponemos de 14 pines digitales (0–13), teniendo en cuenta que los pines 0 y 1 se emplean para el puerto serie, y 6 pines analógicos (A0–A5) que también pueden configurarse como digitales. En total, podemos llegar a tener hasta 20 pines digitales configurables.

Aunque al principio puede parecer que sobran, cuando conectas elementos como una pantalla LCD y más componentes de HW, los pines vuelan. Debido a esta problemática, hoy en clase hemos visto cuál podría ser una buena solución: el **registro de desplazamiento**.

Este pequeño chip dispone de 8 pines por lado, es decir, 16 en total, y se debe montar atravesando la línea central de la **protoboard**. La disposición de los pines es la siguiente. Para guiarnos con izquierda y derecha, debemos fijarnos en la muesca del chip y en el punto que indica el pin 1. La numeración continúa en sentido antihorario.
### Lado Izquierdo (pines 1–8)

| **Pin** | **Nombre** | **Función**     |
| ------- | ---------- | --------------- |
| 1       | Q1         | Salida paralela |
| 2       | Q2         | Salida paralela |
| 3       | Q3         | Salida paralela |
| 4       | Q4         | Salida paralela |
| 5       | Q5         | Salida paralela |
| 6       | Q6         | Salida paralela |
| 7       | Q7         | Salida paralela |
| 8       | GND        | Tierra (0V)     |

### Lado Derecho (pines 9–16)

| **Pin** | **Nombre** | **Función**                                         |
| ------- | ---------- | --------------------------------------------------- |
| 9       | Q7’        | Salida serie                                        |
| 10      | MR         | Reset maestro (activo en LOW)                       |
| 11      | SH_CP      | Clock del registro de desplazamiento (SRCLK)        |
| 12      | ST_CP      | Clock del registro de almacenamiento (Latch / RCLK) |
| 13      | OE         | Habilitación de salida (activo en LOW)              |
| 14      | DS         | Entrada de datos serie (SER)                        |
| 15      | Q0         | Salida paralela                                     |
| 16      | VCC        | Alimentación (+5V)                                  |

---

Una vez sabemos cómo funciona, vamos a ver qué podemos lograr con este registro de desplazamiento.

Lo primero que ya podemos notar es que tenemos 8 salidas disponibles para elementos de HW (Q0–Q7) y lo mejor es que lo vamos a poder conseguir haciendo uso de solo 3 pines digitales del Arduino.

Estos tres pines van a ser el de datos (DS), el reloj del registro de desplazamiento (SH_CP) y el reloj del registro de almacenamiento o Latch (ST_CP). Aparte de estos tres pines, es muy importante también conectar:

* GND a tierra.
* VCC a +5V.
* OE a GND (para habilitar las salidas).
* MR a +5V (para evitar el reset).

Es importante que estén correctamente conectados; si no, el circuito no funcionará bien.

El 74HC595 es un registro de desplazamiento de **entrada serie y salida paralela de 8 bits**. Solo sirve para aumentar las **salidas digitales**, no las entradas. Cada pin puede suministrar aproximadamente ±6 mA a 5V.

Una breve mención sobre el pin Q7’ es que es un pin que nos permite concatenar varios registros de desplazamiento y así, con los mismos tres pines digitales, tener 16 bits (o más) de salida disponibles. Aunque no lo utilizaremos en el proyecto de hoy.

El proyecto de hoy consistirá en tener 8 LEDs conectados a los 8 pines del **registro de desplazamiento 74HC595**, cada uno con una resistencia de **500Ω** (según el montaje propuesto). En nuestro caso utilizaremos resistencias de **1kΩ**, teniendo en cuenta que los LEDs lucirán con menor intensidad.

Los materiales que vamos a utilizar son los siguientes:

* 8 LEDs (en mi caso he utilizado 4 de un color y 4 de otro)
* 8 resistencias de **500Ω** (o 1kΩ si no se dispone de 500Ω)
* Registro de desplazamiento **74HC595**

Ahora quizás te estés preguntando: vale, pero ¿cómo mando yo datos a esos 8 pines? Pues hay una función ya incluida que es la siguiente:

`shiftOut(PIN_DS, PIN_SH_CP, MSBFIRST/LSBFIRST, valor)`

Esta función:

1. Envía los bits correspondientes a `valor` (un byte entre 0 y 255).
2. Lo hace en serie a través del pin de datos.
3. De forma síncrona con una señal de reloj en el pin de reloj.
4. Permite elegir el orden de envío: `MSBFIRST` (bit más significativo primero) o `LSBFIRST` (bit menos significativo primero).

Pero no solo funcionará así, sino que antes y después de llamar a esa función hay que provocar un "pulso" en el pin Latch (ST_CP) para que los datos pasen del shift register al storage register y podamos ver los cambios reflejados en los LEDs.
El 74HC595 captura los datos en el flanco de subida del reloj (SH_CP) y transfiere los valores a las salidas en el flanco de subida del Latch (ST_CP).

Esto lo podemos hacer de la siguiente manera:

```c++
digitalWrite(PIN_ST_CP, LOW);
shiftOut(PIN_DS, PIN_SH_CP, LSBFIRST, valor);
digitalWrite(PIN_ST_CP, HIGH);
```

En este caso será el pin Latch el que empiece en LOW y, una vez enviado el valor, lo ponemos en HIGH para que se actualicen las salidas.

Perfecto, pues ahora podemos ver el código que he creado, que utiliza todas estas funciones para quedarse en un estado bloqueante pidiendo números al usuario, comprobando si cumplen el rango 0–255 y mandarlos para ver la representación binaria de ese número a través de los LEDs:

```c++
#include <Arduino.h> // No necesario si trabajas en el IDE de Arduino

#define PIN_DS 8      // Data (SER)    - PIN 74HC595: 14
#define PIN_ST_CP 9   // Latch (RCLK)  - PIN 74HC595: 12
#define PIN_SH_CP 10  // Clock (SRCLK) - PIN 74HC595: 11

// Prototipos de las funciones
byte leer_numero();
void actualizar_leds(byte v);

void setup() {
  Serial.begin(9600);

  Serial.println("=== Registro de Desplazamiento 74HC595 ===");
  Serial.println("Introduce numeros en el rango: 0-255");

  pinMode(PIN_DS, OUTPUT);
  pinMode(PIN_SH_CP, OUTPUT);
  pinMode(PIN_ST_CP, OUTPUT);

  digitalWrite(PIN_ST_CP, LOW);
  digitalWrite(PIN_SH_CP, LOW);
  digitalWrite(PIN_DS, LOW);

  actualizar_leds(0); // Por defecto apagamos todos los LEDs: 0 -> 00000000
}

void loop() {
  byte num = leer_numero();
  actualizar_leds(num);
}

void actualizar_leds(byte valor) {
  Serial.print("Actualizando a: ");
  Serial.println(valor);

  digitalWrite(PIN_ST_CP, LOW);
  shiftOut(PIN_DS, PIN_SH_CP, LSBFIRST, valor);
  digitalWrite(PIN_ST_CP, HIGH);
}

byte leer_numero() {
  Serial.print("Introduce un numero 0-255: ");
  while (Serial.available() == 0) {
    // Mientras no haya nada que leer esperamos que el usuario escriba algo
  }

  long num = Serial.parseInt();

  // Limpio el buffer
  while (Serial.available() > 0) Serial.read();

  Serial.println("");

  if (num < 0) {
    Serial.println("Has introducido un numero menor a 0");
    Serial.println("Ajustando a 0...");
    num = 0;
  } else if (num > 255) {
    Serial.println("Has introducido un numero mayor a 255");
    Serial.println("Ajustando a 255...");
    num = 255;
  }

  return (byte)num;
}
```

---

## Registro de desplazamiento con un potenciómetro

En este otro proyecto, muy parecido al anterior, en vez de utilizar la entrada del usuario para establecer qué valor mandamos al registro de desplazamiento, lo voy a hacer con un potenciómetro. Se leerá su valor analógico (0–1023) y se mapeará al rango de un byte (0–255).

Luego se comprobará si ha pasado un tiempo correspondiente de medio segundo (500 ms) usando `millis()` (para que sea no bloqueante) y también si el valor que se está intentando mandar es diferente al anterior. Si estas dos condiciones se cumplen, se actualizará el valor.

El código sería el siguiente:

```c++
#include <Arduino.h> // No necesario si trabajas en el IDE de Arduino

#define PIN_DS 8              // Data (SER)    - PIN 74HC595: 14
#define PIN_ST_CP 9           // Latch (RCLK)  - PIN 74HC595: 12
#define PIN_SH_CP 10          // Clock (SRCLK) - PIN 74HC595: 11
#define PIN_POTENCIOMETRO A0  // Pin del potenciómetro

// Prototipos de las funciones
void actualizar_leds(byte v);

// Variables para un sistema no bloqueante con millis
unsigned long actual = 0;
unsigned long pasado = 0;
unsigned long milisegundos = 500;
byte valor_anterior = 0;

void setup() {
  Serial.begin(9600);

  Serial.println("=== Registro de Desplazamiento 74HC595 ===");

  pinMode(PIN_DS, OUTPUT);
  pinMode(PIN_SH_CP, OUTPUT);
  pinMode(PIN_ST_CP, OUTPUT);
  pinMode(PIN_POTENCIOMETRO, INPUT);

  digitalWrite(PIN_ST_CP, LOW);
  digitalWrite(PIN_SH_CP, LOW);
  digitalWrite(PIN_DS, LOW);

  actualizar_leds(0);
}

void loop() {
  int lectura = analogRead(PIN_POTENCIOMETRO);

  byte valor_actual = map(lectura, 0, 1023, 0, 255);

  actual = millis();

  if (actual - pasado >= milisegundos && valor_actual != valor_anterior) {
    actualizar_leds(valor_actual);
    pasado = actual;
    valor_anterior = valor_actual;
  }
}

void actualizar_leds(byte valor) {
  Serial.print("Actualizando a: ");
  Serial.println(valor);

  digitalWrite(PIN_ST_CP, LOW);
  shiftOut(PIN_DS, PIN_SH_CP, LSBFIRST, valor);
  digitalWrite(PIN_ST_CP, HIGH);
}](<#include %3CArduino.h%3E // No necesario si trabajas en el IDE de Arduino

#define PIN_DS 8              // Data (SER)    - PIN 74HC595: 14
#define PIN_ST_CP 9           // Latch (RCLK)  - PIN 74HC595: 12
#define PIN_SH_CP 10          // Clock (SRCLK) - PIN 74HC595: 11
#define PIN_POTENCIOMETRO A0  // Pin del potenciómetro

// Prototipos de las fucniones
void actualizar_leds(byte v);

// Variables para un sistema no bloqueante con millis
unsigned long actual = 0;         // Variable en la que voy guardando el tiempo actual
unsigned long pasado = 0;         // Variable para comprobar si ha pasado el tiempo
unsigned long milisegundos = 500; // Espero medio segundo entre actualizaciones de Leds
byte valor_anterior = 0;          // Variable para no actualizar los leds si es el mismo valor


void setup() {
  Serial.begin(9600);

  Serial.println("=== Registro de Desplazamiento 74HC595 ===");
  Serial.println("Introduce un numero 0-255");

  pinMode(PIN_DS, OUTPUT);
  pinMode(PIN_SH_CP, OUTPUT);
  pinMode(PIN_ST_CP, OUTPUT);
  pinMode(PIN_POTENCIOMETRO, INPUT);

  digitalWrite(PIN_ST_CP, LOW);
  digitalWrite(PIN_SH_CP, LOW);
  digitalWrite(PIN_DS, LOW);

  actualizar_leds(0); // Por defecto apagamos todos los leds: 0 -> 00000000 (8 Leds en OFF)
}

void loop() {
  int lectura = analogRead(PIN_POTENCIOMETRO);

  // Mapeo el valor analógico a un valor en el rango de un Byte
  byte valor_actual = map(lectura, 0, 1023, 0, 255);

  actual = millis(); // A cada vuelta del bucle me quedo con el tiempo actual (ms)

  // Si ha pasado el tiempo correspondiente actualizo los leds
  if (actual - pasado >= milisegundos && valor_actual != valor_anterior) {
    actualizar_leds(valor_actual);
    pasado = actual;               // Actualizo el tiempo
    valor_anterior = valor_actual; // Actualizo los valores
  }
}

void actualizar_leds(byte valor) {
  Serial.print("Actualizando a: ");
  Serial.println(valor);

  digitalWrite(PIN_ST_CP, LOW);
  shiftOut(PIN_DS, PIN_SH_CP, LSBFIRST, valor);  // <-- Envio el valor
  digitalWrite(PIN_ST_CP, HIGH);
}
```

Como reflexión final, creo que el registro de desplazamiento es una pieza clave para escalar los proyectos, ya que permite aumentar el número de salidas digitales disponibles utilizando solo tres pines del microcontrolador.
