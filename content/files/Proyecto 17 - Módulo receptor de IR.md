---
title: Proyecto 17 - Módulo receptor de IR
date: 2026-03-13
draft: false
---

## Recibir información a través del receptor de infrarrojos

En la clase de hoy hemos trabajado con un nuevo componente, el **receptor de infrarrojos**. Este módulo tiene 3 pines, de los cuales 2 son para **tierra (GND)** y **5V (VCC)**, mientras que el otro es la **salida digital de señal (S)**, que conectaremos a un pin digital de Arduino.

Este receptor (**HX1838**) se va a encargar de recibir la información mandada por un **mando IR**. Este mando es el que nos permite enviar información de forma inalámbrica a una placa de Arduino. La información emitida es digital y se agrupa en tramas de duración y número de bits fijados; es decir, cada botón del mando envía una serie distinta de códigos en el espectro infrarrojo, los cuales serán detectados e interpretados por el receptor. Para que esta comunicación sea posible se hace uso de **protocolos de comunicación** para determinar una codificación de la información. Por tanto, esta codificación deberá ser conocida tanto por el **emisor** como por el receptor.

Un protocolo muy conocido es el protocolo **NEC**, que es el que utiliza el **mando IR** incluido en Arduino. La onda portadora tiene un periodo de **26 microsegundos**, mientras que cada pulso es de **560 microsegundos**, algo más de 21 ciclos de la señal portadora. Para codificar los ceros y los unos se utiliza la diferencia en tiempo entre un pulso y el siguiente:

* Si llega **2.25 ms** después, se considera un **1**.
* Si llega **1.12 ms** después, se considera un **0**.

Una vez entendida gran parte de la teoría de cómo funciona este receptor de infrarrojos y a través de qué le vamos a poder enviar datos, vamos a ver el proyecto que montaremos hoy. Para el proyecto de hoy vamos a utilizar los siguientes componentes:

* Mando a distancia de infrarrojos
* Receptor de infrarrojos
* LED (en este caso utilizaremos uno rojo, pero el color es indiferente)
* Resistencia de **220 Ω** (para el LED)

El objetivo del proyecto va a ser el siguiente: mediante el mando IR, deberemos ser capaces de encender y apagar el **LED**, y además poder controlar la potencia luminosa del LED entre **9 posibles niveles**. Para poder controlar la potencia del LED deberemos conectarlo a un pin digital **PWM**, ya que es el que nos permitirá variar la potencia media mediante la función `analogWrite()`.

Perfecto, pues antes de ir directos al código vamos a ver cómo podemos controlar este receptor y el mando IR a través del código.

Para usar el receptor con **IRremote**, lo primero que debemos hacer es importar una librería creada por: *shirriff*, *z3t0* y *ArminJo*; esta se llama **[IRremote](https://github.com/Arduino-IRremote/Arduino-IRremote)** y se puede incluir de la siguiente manera:

```c++
#include <IRremote.h>
```

Por otro lado, tendremos que iniciar las comunicaciones con la siguiente instrucción en la función `setup()`:

```c++
IrReceiver.begin(PIN_RECEPTOR);
```

Algunas funciones útiles de la librería **IRremote** son:

```c++
// Devuelve true si se ha decodificado algún dato
bool variable = IrReceiver.decode();

// Guarda la trama completa recibida
unsigned long variable = IrReceiver.decodedIRData.decodedRawData;

// Guarda solo el comando recibido, sin la dirección
byte variable = IrReceiver.decodedIRData.command;

// Muestra por el monitor serie un resumen del protocolo y de los datos recibidos
IrReceiver.printIRResultShort(&Serial);

// Reinicia el receptor IR para realizar una nueva lectura
IrReceiver.resume();
```

Después de leer y guardar la información que nos interese, es obligatorio usar:

```c++
// Reinicia el receptor IR para permitir una nueva lectura
IrReceiver.resume();
```

Si no se llama a **resume()**, el receptor queda bloqueado y no podrá recibir otra señal.

> Otra cosa a tener en cuenta, y esto lo puedo decir por experiencia, es que esta librería puede generar conflictos con ciertos temporizadores o pines PWM según la placa y la versión de la librería. Como recomendación, mejor utilizar otro pin PWM distinto al 3 como por ejemplo el **5**.

Pues ahora ya sí que sí vamos con el código del proyecto:

```c++
#include <Arduino.h>
#include <IRremote.h>

#define PIN_RECEPTOR 2
#define PIN_LED_PWM 5

// === Códigos de los botones del mando ===
// Estos valores pueden cambiar según el mando.
// Si no coinciden con el mando que se está utilizando,
// habrá que corregirlos mirando lo que se recibe
// por el monitor serie y cambiar los números de aquí.

#define BOTON_ON_OFF 8   // Por ejemplo: botón POWER
#define BOTON_1      17
#define BOTON_2      18
#define BOTON_3      19
#define BOTON_4      20
#define BOTON_5      21
#define BOTON_6      22
#define BOTON_7      23
#define BOTON_8      24
#define BOTON_9      25

bool ledEncendido = false;
byte nivelBrillo = 9;   // Nivel actual: de 1 a 9

int obtenerPWM(byte nivel);
void actualizarLed();

void setup() {
  Serial.begin(9600);

  pinMode(PIN_LED_PWM, OUTPUT);
  analogWrite(PIN_LED_PWM, 0); // Potencia 0 al LED, lo apago

  IrReceiver.begin(PIN_RECEPTOR); // Inicio las comunicaciones

  // Aviso al usuario de que el sistema ha comenzado
  Serial.println("Sistema IR iniciado");
  Serial.println("Pulsa botones del mando...");
}

void loop() {
  if (IrReceiver.decode()) { // Si hay algún valor decodificado
    byte comando = IrReceiver.decodedIRData.command; // Guardo solo el comando recibido, sin la dirección

    // Muestro información útil por el monitor serie
    // Resumen del protocolo y datos recibidos
    IrReceiver.printIRResultShort(&Serial);
    Serial.print("Comando: ");
    Serial.println(comando);

    // Encender / apagar LED
    if (comando == BOTON_ON_OFF) {
      ledEncendido = !ledEncendido;
      actualizarLed();
    }

    // Niveles de brillo del 1 al 9
    else if (comando == BOTON_1) {
      nivelBrillo = 1;
      ledEncendido = true;
      actualizarLed();
    }
    else if (comando == BOTON_2) {
      nivelBrillo = 2;
      ledEncendido = true;
      actualizarLed();
    }
    else if (comando == BOTON_3) {
      nivelBrillo = 3;
      ledEncendido = true;
      actualizarLed();
    }
    else if (comando == BOTON_4) {
      nivelBrillo = 4;
      ledEncendido = true;
      actualizarLed();
    }
    else if (comando == BOTON_5) {
      nivelBrillo = 5;
      ledEncendido = true;
      actualizarLed();
    }
    else if (comando == BOTON_6) {
      nivelBrillo = 6;
      ledEncendido = true;
      actualizarLed();
    }
    else if (comando == BOTON_7) {
      nivelBrillo = 7;
      ledEncendido = true;
      actualizarLed();
    }
    else if (comando == BOTON_8) {
      nivelBrillo = 8;
      ledEncendido = true;
      actualizarLed();
    }
    else if (comando == BOTON_9) {
      nivelBrillo = 9;
      ledEncendido = true;
      actualizarLed();
    }

    // Permite recibir una nueva señal
    IrReceiver.resume(); // En caso de no ponerlo, las comunicaciones quedarían bloqueadas
  }
}

// Convierte un nivel de 1 a 9 en un valor PWM de 0 a 255
int obtenerPWM(byte nivel) {
  return map(nivel, 1, 9, 28, 255);
}

// Actualiza el LED según si está encendido y el nivel actual
void actualizarLed() {
  if (ledEncendido) {
    int nuevo_brillo = obtenerPWM(nivelBrillo);
    analogWrite(PIN_LED_PWM, nuevo_brillo);
    Serial.print("Actualizado a un nivel de brillo de: ");
    Serial.println(nuevo_brillo);
  } else {
    analogWrite(PIN_LED_PWM, 0);
    Serial.println("LED apagado!");
  }
}
```

## Versión mejorada: Mostramos el número en un visualizador de 7 segmentos

Para esta nueva versión mejorada lo que vamos a añadir es un **visualizador de 7 segmentos** y, manteniendo gran parte de la lógica de la versión anterior con la que podemos cambiar la potencia luminosa del LED, ahora vamos a mostrar los números del **1 al 9** dependiendo del botón pulsado en el mando. En esta versión no tenemos botón de encendido y apagado, sino que el número **1** deja el LED en el nivel mínimo y el **9** lo enciende a la máxima potencia. Por lo demás es igual.

El código es el siguiente:

```c++
#include <Arduino.h>
#include <IRremote.h>

#define PIN_RECEPTOR 2
#define PIN_LED_PWM 5

// === Códigos de los botones del mando ===
#define BOTON_1 17
#define BOTON_2 18
#define BOTON_3 19
#define BOTON_4 20
#define BOTON_5 21
#define BOTON_6 22
#define BOTON_7 23
#define BOTON_8 24
#define BOTON_9 25

// === Pines del visualizador de 7 segmentos ===
#define PIN_SEGMENTO_A 6
#define PIN_SEGMENTO_B 7
#define PIN_SEGMENTO_C 8
#define PIN_SEGMENTO_D 9
#define PIN_SEGMENTO_E 10
#define PIN_SEGMENTO_F 11
#define PIN_SEGMENTO_G 12
#define PIN_DP 13

#define MIN_SEGMENT 6
#define MAX_SEGMENT 13

// Prototipos
void actualizar_led_pwm(int num);
void poner_numero(int num);
int extraer_num(byte comando);
void apagar_segmentos();
int obtener_pwm(int num);

void setup() {
  Serial.begin(9600);

  IrReceiver.begin(PIN_RECEPTOR);

  for (int i = MIN_SEGMENT; i <= MAX_SEGMENT; i++) {
    pinMode(i, OUTPUT);
  }

  pinMode(PIN_LED_PWM, OUTPUT);

  // Por defecto muestro el número 0 en el visualizador
  poner_numero(0);
  analogWrite(PIN_LED_PWM, 0); // LED apagado al inicio

  Serial.println("Bienvenido al programa");
  Serial.println("Empieza a pulsar los botones del 1 al 9 de tu mando");
  Serial.println("Cada número pulsado se mostrará en el visualizador");
  Serial.println("1 deja el LED en el nivel mínimo y 9 lo enciende a máxima potencia");
  Serial.println("Los demás números ajustan una potencia intermedia");
}

void loop() {
  if (IrReceiver.decode()) {
    byte comando = IrReceiver.decodedIRData.command;

    Serial.print("Comando recibido: ");
    Serial.println(comando);

    int numero_a_mostrar = extraer_num(comando);

    // Solo actualizo el número si realmente se ha pulsado un número válido
    if (numero_a_mostrar != 0) {
      poner_numero(numero_a_mostrar);
      actualizar_led_pwm(numero_a_mostrar);

      Serial.print("Número mostrado: ");
      Serial.println(numero_a_mostrar);
    }

    IrReceiver.resume();
  }
}

void actualizar_led_pwm(int num) {
  int pwm = obtener_pwm(num);
  analogWrite(PIN_LED_PWM, pwm);

  Serial.print("Potencia del LED actualizada: ");
  Serial.println(pwm);
}

int obtener_pwm(int num) {
  // Convierte niveles 1..9 en PWM 0..255
  return map(num, 1, 9, 0, 255);
}

void poner_numero(int num) {
  if (num < 0 || num > 9) return;

  apagar_segmentos();

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

int extraer_num(byte comando) {
  switch (comando) {
    case BOTON_1: return 1;
    case BOTON_2: return 2;
    case BOTON_3: return 3;
    case BOTON_4: return 4;
    case BOTON_5: return 5;
    case BOTON_6: return 6;
    case BOTON_7: return 7;
    case BOTON_8: return 8;
    case BOTON_9: return 9;
    default: return 0;
  }
}

void apagar_segmentos() {
  for (int i = MIN_SEGMENT; i <= MAX_SEGMENT; i++) {
    digitalWrite(i, LOW);
  }
}
```
