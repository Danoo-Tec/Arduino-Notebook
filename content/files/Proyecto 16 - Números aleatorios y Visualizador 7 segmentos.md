---
title: Proyecto 16 - Números aleatorios y Visualizador 7 segmentos
date: 2026-03-06
draft: false
---

## Generación de números aleatorios

En la clase de hoy hemos visto varias cosas, entre ellas la generación de números aleatorios en Arduino; bueno, **pseudoaleatorios**, ya que es muy importante tener en cuenta que la aleatoriedad no existe realmente en informática. Si tú le pasas las mismas entradas, siempre te dará las mismas salidas.

Pero claro, si yo quiero crear un número aparentemente aleatorio voy a tener que hacer uso de una **semilla** basada en un valor que cambia constantemente. Una práctica muy usada es utilizar como semilla el **tiempo Unix**, que es un valor de tiempo que siempre aumenta; este empezó el **1 de enero de 1970**.

Aunque, ya que estamos en Arduino, vamos a utilizar para la semilla un cable conectado a un **pin analógico que está al aire**, ya que si leemos su valor con `analogRead()` sí podemos ver que es un valor que fluctúa bastante.

La manera en la que podemos establecer esta semilla es con la siguiente función:

```cpp
randomSeed(analogRead(PIN_ANALOG_AIRE));
```

Como vemos, a esta función le estamos pasando la lectura del pin analógico que se encuentra conectado a nada.

> Muy importante que esto esté situado (la llamada a esta función) en el **setup()**.

Perfecto, una vez ya hemos establecido la semilla, ya podemos generar números aleatorios (bueno, pseudoaleatorios), pero ¿y cómo lo hago? Bueno, pues con la siguiente función:

```cpp
long num = random(MIN_INCLUSIVE, MAX_EXCLUSIVE);
```

Ciertas cosas a tener en cuenta sobre esta función es que el primer número que le paso **sí lo incluye**, pero **no el último**, ya que es exclusivo. Por ejemplo, si quiero generar números aleatorios del **1 al 6** como un dado, tendría que hacer lo siguiente: `random(1, 7)`.

Luego, otro punto importante es que el valor devuelto hay que guardarlo en una variable de tipo **long**, ya que es el tipo de dato que devuelve esta función.

El siguiente código es un ejemplo de cómo crear números aleatorios del **1 al 100** (ambos incluidos) e imprimirlos por pantalla:

```cpp
#include <Arduino.h> // No es necesario si utilizamos el IDE de Arduino

#define PIN_ANALOG_AIRE A0

void setup() {
  Serial.begin(9600);

  // Establezco la semilla a la entrada
  randomSeed(analogRead(PIN_ANALOG_AIRE));
}

void loop() {
  // Genero el número aleatorio
  long rand_num = random(1, 101);
  
  Serial.println(rand_num);
  delay(1000); // Imprimo un número aleatorio cada segundo
}
```

## Uso del visualizador de 7 segmentos

Bien, ahora que ya hemos visto cómo generar números pseudoaleatorios en Arduino, vamos a aprender a utilizar el **visualizador de 7 segmentos**, que es el componente que tiene pintado como un "0" con rayitas y un punto abajo a la derecha.

Vamos a ver cómo conectar este visualizador:

- Lo primero es que es interesante conectarlo **sobre la línea central de la protoboard**.
- Luego, como vemos, tiene **5 pines a cada lado** corto, de los cuales los **centrales de arriba y abajo** deben ir a tierra con una resistencia de **330Ω**, por ejemplo.
- Solo hace falta utilizar **uno a tierra**, no los dos, ya que por dentro están interconectados.

También es importante tener en cuenta que existen dos tipos de visualizadores de 7 segmentos:  
  
- **Cátodo común**: el pin común va a **GND** y los segmentos se encienden con **HIGH**.  
- **Ánodo común**: el pin común va a **5V** y los segmentos se encienden con **LOW**.  
  
Ahora vamos a ver qué pines van a cada barra:

```text
                      GND común
           10      9      8      7      6
            |      |      |      |      |
         __________________________________
        |                                  |
        |           ____________           |
        |          /      A     \          |
        |         /______________\         |
        |        /|              |\        |
        |       / |              | \       |
        |      | F|              |B |      |
        |      |  |______________|  |      |
        |      |  /       G      \  |      |
        |      |  \______________/  |      |
        |      |  |              |  |      |
        |      |E |              | C|      |
        |      |  |______________|  |      |
        |       \ \       D      / /       |
        |        \ \____________/ /  (DP)  |
        |                                  |
        |__________________________________|
            |      |      |      |      |
            1      2      3      4      5
                      GND común
```

|**Barra / segmento**|**Número de pin**|
|---|---|
|A|7|
|B|6|
|C|4|
|D|2|
|E|1|
|F|9|
|G|10|
|DP|5|
|común|3 y 8|

Perfecto, pues ya con toda esta información podemos montar el circuito conectando una **tierra a GND con una resistencia de 330Ω**, y los demás pines de los segmentos a **pines digitales**.

Puede ser interesante conectar los pines, por ejemplo, del **pin 4 al 11** para poder establecer todos como **OUTPUT** fácilmente o apagarlos todos, etc.

Una vez visto todo esto vamos a entrar ya en el proyecto de hoy, que mezcla la creación de números pseudoaleatorios y el visualizador de 7 segmentos con **dos botones**: uno para indicar cuándo empezar a crear el número aleatorio y mostrarlo, y otro para **resetear**.

Antes de nada decir que mi código funciona suponiendo que el display es de **cátodo común**, por lo que el pin común es a tierra.

Lo que hará el programa es lo siguiente:

- Nada más compilar no hace nada.
- Una vez pulsado el botón **trigger**, se activará una animación.
- A los **segundos** (entre **2 y 5**) se mostrará un número aleatorio de un **dado de 6 caras**.
- A partir de ahí lo único que puedo hacer es pulsar el botón de **reset** para dejar todo como al principio.

El código es el siguiente:

```cpp
#include <Arduino.h> // Con el IDE de Arduino no hace falta

// PIN botones
#define PIN_BTN_RESET 2
#define PIN_BTN_TRIGGER 3

// PIN segmentos de arriba
#define PIN_SEGMENTO_G 4
#define PIN_SEGMENTO_F 5
#define PIN_SEGMENTO_A 6
#define PIN_SEGMENTO_B 7

// PIN segmentos de abajo
#define PIN_SEGMENTO_E 8
#define PIN_SEGMENTO_D 9
#define PIN_SEGMENTO_C 10
#define PIN_DP 11

#define PIN_ANALOG_AIRE A0

#define MIN_SEGMENT 4
#define MAX_SEGMENT 11

// Prototipos de funciones
void tirando_dado();
void poner_numero(int num);
void apagar_segmentos();
bool detectar_flanco_subida_reset(bool estado_nuevo);
bool detectar_flanco_subida_trigger(bool lectura);

void setup() {
  Serial.begin(9600);

  // Pongo los pines de los segmentos como OUTPUT
  for (int i = MIN_SEGMENT; i <= MAX_SEGMENT; i++) {
    pinMode(i, OUTPUT);
  }

  pinMode(PIN_BTN_RESET, INPUT_PULLUP);
  pinMode(PIN_BTN_TRIGGER, INPUT_PULLUP);

  // Establezco la semilla a la entrada
  randomSeed(analogRead(PIN_ANALOG_AIRE));
}

void loop() {
  bool estado_btn_trigger = digitalRead(PIN_BTN_TRIGGER);
  bool estado_btn_reset = digitalRead(PIN_BTN_RESET);

  // Creo el número aleatorio para un dado de 6 caras
  long num = random(1, 7);

  if (detectar_flanco_subida_trigger(estado_btn_trigger)) {
    // Si se pulsa el botón de trigger, inicio animación
    tirando_dado();

    // Muestro el número del dado aleatorio
    poner_numero(num);

    // Mientras no se pulse reset, no hago nada
    while (!detectar_flanco_subida_reset(estado_btn_reset)) {
      estado_btn_reset = digitalRead(PIN_BTN_RESET);
    }

    apagar_segmentos(); // Apago todos los segmentos
  }
}

void tirando_dado() {
  long segundos = random(2, 6);
  unsigned long inicio = millis();

  while (millis() - inicio < segundos * 1000) {
    digitalWrite(PIN_SEGMENTO_A, HIGH);
    delay(200);
    digitalWrite(PIN_SEGMENTO_A, LOW);
    digitalWrite(PIN_SEGMENTO_B, HIGH);
    delay(200);
    digitalWrite(PIN_SEGMENTO_B, LOW);
    digitalWrite(PIN_SEGMENTO_C, HIGH);
    delay(200);
    digitalWrite(PIN_SEGMENTO_C, LOW);
    digitalWrite(PIN_SEGMENTO_D, HIGH);
    delay(200);
    digitalWrite(PIN_SEGMENTO_D, LOW);
    digitalWrite(PIN_SEGMENTO_E, HIGH);
    delay(200);
    digitalWrite(PIN_SEGMENTO_E, LOW);
    digitalWrite(PIN_SEGMENTO_F, HIGH);
    delay(200);
    digitalWrite(PIN_SEGMENTO_F, LOW);
  }
}

void poner_numero(int num) {
  if (num < 0 || num > 9) return;

  switch (num) {

    case 0:
    digitalWrite(PIN_SEGMENTO_A, HIGH);
    digitalWrite(PIN_SEGMENTO_B, HIGH);
    digitalWrite(PIN_SEGMENTO_C, HIGH);
    digitalWrite(PIN_SEGMENTO_D, HIGH);
    digitalWrite(PIN_SEGMENTO_E, HIGH);
    digitalWrite(PIN_SEGMENTO_F, HIGH);
    digitalWrite(PIN_SEGMENTO_G, LOW);
    break;

    case 1:
    digitalWrite(PIN_SEGMENTO_A, LOW);
    digitalWrite(PIN_SEGMENTO_B, HIGH);
    digitalWrite(PIN_SEGMENTO_C, HIGH);
    digitalWrite(PIN_SEGMENTO_D, LOW);
    digitalWrite(PIN_SEGMENTO_E, LOW);
    digitalWrite(PIN_SEGMENTO_F, LOW);
    digitalWrite(PIN_SEGMENTO_G, LOW);
    break;

    case 2:
    digitalWrite(PIN_SEGMENTO_A, HIGH);
    digitalWrite(PIN_SEGMENTO_B, HIGH);
    digitalWrite(PIN_SEGMENTO_C, LOW);
    digitalWrite(PIN_SEGMENTO_D, HIGH);
    digitalWrite(PIN_SEGMENTO_E, HIGH);
    digitalWrite(PIN_SEGMENTO_F, LOW);
    digitalWrite(PIN_SEGMENTO_G, HIGH);
    break;

    case 3:
    digitalWrite(PIN_SEGMENTO_A, HIGH);
    digitalWrite(PIN_SEGMENTO_B, HIGH);
    digitalWrite(PIN_SEGMENTO_C, HIGH);
    digitalWrite(PIN_SEGMENTO_D, HIGH);
    digitalWrite(PIN_SEGMENTO_E, LOW);
    digitalWrite(PIN_SEGMENTO_F, LOW);
    digitalWrite(PIN_SEGMENTO_G, HIGH);
    break;

    case 4:
    digitalWrite(PIN_SEGMENTO_A, LOW);
    digitalWrite(PIN_SEGMENTO_B, HIGH);
    digitalWrite(PIN_SEGMENTO_C, HIGH);
    digitalWrite(PIN_SEGMENTO_D, LOW);
    digitalWrite(PIN_SEGMENTO_E, LOW);
    digitalWrite(PIN_SEGMENTO_F, HIGH);
    digitalWrite(PIN_SEGMENTO_G, HIGH);
    break;

    case 5:
    digitalWrite(PIN_SEGMENTO_A, HIGH);
    digitalWrite(PIN_SEGMENTO_B, LOW);
    digitalWrite(PIN_SEGMENTO_C, HIGH);
    digitalWrite(PIN_SEGMENTO_D, HIGH);
    digitalWrite(PIN_SEGMENTO_E, LOW);
    digitalWrite(PIN_SEGMENTO_F, HIGH);
    digitalWrite(PIN_SEGMENTO_G, HIGH);
    break;

    case 6:
    digitalWrite(PIN_SEGMENTO_A, HIGH);
    digitalWrite(PIN_SEGMENTO_B, LOW);
    digitalWrite(PIN_SEGMENTO_C, HIGH);
    digitalWrite(PIN_SEGMENTO_D, HIGH);
    digitalWrite(PIN_SEGMENTO_E, HIGH);
    digitalWrite(PIN_SEGMENTO_F, HIGH);
    digitalWrite(PIN_SEGMENTO_G, HIGH);
    break;

    case 7:
    digitalWrite(PIN_SEGMENTO_A, HIGH);
    digitalWrite(PIN_SEGMENTO_B, HIGH);
    digitalWrite(PIN_SEGMENTO_C, HIGH);
    digitalWrite(PIN_SEGMENTO_D, LOW);
    digitalWrite(PIN_SEGMENTO_E, LOW);
    digitalWrite(PIN_SEGMENTO_F, LOW);
    digitalWrite(PIN_SEGMENTO_G, LOW);
    break;

    case 8:
    digitalWrite(PIN_SEGMENTO_A, HIGH);
    digitalWrite(PIN_SEGMENTO_B, HIGH);
    digitalWrite(PIN_SEGMENTO_C, HIGH);
    digitalWrite(PIN_SEGMENTO_D, HIGH);
    digitalWrite(PIN_SEGMENTO_E, HIGH);
    digitalWrite(PIN_SEGMENTO_F, HIGH);
    digitalWrite(PIN_SEGMENTO_G, HIGH);
    break;

    case 9:
    digitalWrite(PIN_SEGMENTO_A, HIGH);
    digitalWrite(PIN_SEGMENTO_B, HIGH);
    digitalWrite(PIN_SEGMENTO_C, HIGH);
    digitalWrite(PIN_SEGMENTO_D, HIGH);
    digitalWrite(PIN_SEGMENTO_E, LOW);
    digitalWrite(PIN_SEGMENTO_F, HIGH);
    digitalWrite(PIN_SEGMENTO_G, HIGH);
    break;
  }
}

void apagar_segmentos() {
  // Apago todos, los pongo en LOW
  for (int i = MIN_SEGMENT; i <= MAX_SEGMENT; i++) {
    digitalWrite(i, LOW);
  }
}

bool detectar_flanco_subida_reset(bool lectura) {
  static bool anterior = HIGH;
  static unsigned long antes = 0;

  if (anterior != lectura && millis() - antes > 20) {
    antes = millis();
    anterior = lectura;
    if (lectura == LOW) return true;   // HIGH → LOW
  }

  return false;
}

bool detectar_flanco_subida_trigger(bool lectura) {
  static bool anterior = HIGH;
  static unsigned long antes = 0;

  if (anterior != lectura && millis() - antes > 20) {
    antes = millis();
    anterior = lectura;
    if (lectura == LOW) return true;   // HIGH → LOW
  }

  return false;
}
```

Para detectar cuándo se pulsa un botón he utilizado `INPUT_PULLUP`, por lo que el pin está normalmente en **HIGH** gracias a la resistencia interna de Arduino y, cuando pulso el botón, pasa a **LOW**.

Por eso, en este caso la pulsación realmente se detecta en el cambio de **HIGH → LOW**.

Además, he implementado un pequeño **debouncing por software** para evitar lecturas falsas, ya que los botones físicos producen **rebotes eléctricos** al pulsarse. Durante unos milisegundos el pin puede oscilar entre **HIGH** y **LOW**, así que para evitar falsas detecciones se ignoran cambios durante aproximadamente **20 ms**.