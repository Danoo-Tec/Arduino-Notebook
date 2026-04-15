---
title: Proyecto 20 - Joystick y visualizador de 4 dígitos
date: 2026-04-10
draft: false
---

## Funcionamiento y montaje del visualizador y del joystick

Como habréis podido apreciar en el título, hoy en clase hemos añadido dos nuevos componentes a nuestra lista: el visualizador de 7 segmentos de 4 dígitos y el joystick.

Vamos a ir uno por uno, empezando por el visualizador, que ya habíamos visto en un proyecto anterior, pero entonces era de un solo dígito en vez de 4.
La parte de conectar los segmentos y el punto decimal es igual que en el anterior; me refiero al funcionamiento, porque el *pinout* ahora lo veremos. Lo interesante es que tenemos 4 pines más (**D1, D2, D3 y D4**). Estos pines son los que nos servirán para elegir con qué dígito vamos a trabajar. En efecto, solo vamos a poder trabajar con un dígito a la vez, no con los 4 a la vez.

Para entender la manera en la que vamos a elegir qué dígito encender y qué dígito apagar, tenemos que distinguir entre si es **cátodo común** o **ánodo común**. Esto es porque, para uno de los dos, vamos a tener que poner todos a `HIGH` y el que queremos mostrar en `LOW`, y viceversa, siendo el primer caso el que nos encontramos con el visualizador que viene en nuestro kit de Arduino UNO.

Una cosa importante antes de cablearlo es verificar bien la polaridad y la orientación del visualizador. Para eso nos podemos fijar en el **punto decimal**, ya que nos sirve como referencia visual para colocarlo en la posición correcta y no montar luego los pines al revés.

También conviene dejar claro que el display de 4 dígitos tiene **12 pines**:

* **8 pines compartidos** para los segmentos y el punto decimal.
* **4 pines comunes**, uno para cada dígito.

Una vez entendido esto, vamos a montarlo de la siguiente manera:

* 8 cables a pines digitales para los 7 segmentos y el punto decimal.
* 4 más, también a pines digitales. Estos nos servirán para elegir qué dígito encender.
* 4 resistencias de **330 Ω**. Estas van para esos 4 pines, para elegir el dígito a encender.

El *pinout* del visualizador es el siguiente:

```ascii
            VISUALIZADOR 4 DÍGITOS

      12     11    10     9     8     7
      D1      A     F    D2    D3     B
       |      |     |     |     |     |
   +---------------------------------------+
   |                                       |
   |       [ 8.]  [ 8.]  [ 8.]  [ 8.]      |
   |                                       |
   +---------------------------------------+
       |      |      |     |     |     |
       1      2      3     4     5     6
       E      D   decimal  C     G    D4

  ┌────────────────────────────────────────┐
  │  PIN   SEÑAL   FUNCIÓN                 │
  ├────────────────────────────────────────┤
  │   1      E     Segmento E              │
  │   2      D     Segmento D              │
  │   3     DEC    Punto decimal           │
  │   4      C     Segmento C              │
  │   5      G     Segmento G              │
  │   6      D4    Dígito 4 (común)        │
  │   7      B     Segmento B              │
  │   8      D3    Dígito 3 (común)        │
  │   9      D2    Dígito 2 (común)        │
  │  10      F     Segmento F              │
  │  11      A     Segmento A              │
  │  12      D1    Dígito 1 (común)        │
  └────────────────────────────────────────┘
```

Ahora vamos a pasar al joystick, que creo que la manera más fácil de entenderlo es verlo como **dos potenciómetros** y un pin digital. Esto lo digo ya que realmente es así: el joystick va a tener un eje **X** y un eje **Y**, y estos van a ir conectados a pines analógicos como los potenciómetros; luego, el pin digital hace la función de pulsador, ya que el joystick se puede pulsar. Pensarlo así nos va a servir para luego trabajarlo en el código y también para conectar bien los pines.

El *pinout* del joystick es el siguiente (de izquierda a derecha):

* `GND` -> tierra
* `5V` -> 5V
* `VRx` -> pin analógico
* `VRy` -> pin analógico
* `SW` -> pin digital (botón del joystick)

Una comprobación que viene bien hacer es mirar en el **monitor serie** los valores de `VRx` y `VRy` tanto en reposo como en los extremos. Lo normal es que, cuando el joystick está centrado, el valor ronde más o menos la mitad del rango analógico, y cuando lo llevamos a los extremos se acerque a `0` o a `1023`.

También hay que tener en cuenta que el pulsador `SW` normalmente se usa con `INPUT_PULLUP`, así que al pulsar se lee `LOW` y en reposo se lee `HIGH`. Aun así, en el código, para trabajar de manera más intuitiva, eso se puede invertir.

Vale, ahora que ya tenemos montado el circuito, es decir, tanto el visualizador como el joystick, podemos hablar del proyecto de hoy, en qué consiste.
Bueno, pues el proyecto de hoy va a ser desarrollar un visualizador de movimientos de ajedrez en el que vamos a tener las siguientes características:

* El movimiento ejecutado se mostrará iluminando todos los puntos del visualizador de 7 segmentos.
* El eje X del joystick **seleccionará cuál de los cuatro dígitos** **del visualizador se desea mover**.
* El eje Y del joystick **moverá hacia arriba o hacia abajo el** **valor** visualizado en el dígito seleccionado.
* **El pulsador** del joystick servirá para dar la **orden de ejecutar** **el movimiento** o comenzar uno nuevo.
* El dígito seleccionado se representa mediante el parpadeo de su punto.
* Las órdenes serán del tipo **E2 E4**, es decir, el primer y tercer dígito mostrarán letras, mientras que el segundo y cuarto mostrarán números.

Antes de pasar al código, hay una cosa importante sobre cómo pintamos símbolos en el visualizador. En un display de 7 segmentos no se puede escribir cualquier letra bien, sino solo aquellas que se pueden representar de forma más o menos clara con los segmentos disponibles. Por eso usamos letras como **A, b, C, d, E, F, g, H** y no cualquier carácter.

Además, en Arduino Uno los pines analógicos como `A0`, `A1`, `A2`, etc. también pueden referenciarse como números digitales altos. Por eso a veces veréis `A0` y otras veces `14`, `15`, `16`, dependiendo de cómo esté escrito el código.

Para pintar un símbolo lo que hacemos es guardar un patrón binario donde cada bit representa si un segmento se enciende o no. Luego usamos `bitRead()` para leer cada bit de ese patrón y mandarlo al segmento correspondiente.

Una tabla pequeña para entenderlo mejor sería esta:

| Símbolo | Patrón binario |
| ------- | -------------- |
| 0       | `0b11111100`   |
| 1       | `0b01100000`   |
| 2       | `0b11011010`   |
| 3       | `0b11110010`   |
| 4       | `0b01100110`   |
| 5       | `0b10110110`   |
| 6       | `0b10111110`   |
| 7       | `0b11100000`   |
| 8       | `0b11111110`   |
| 9       | `0b11100110`   |
| A       | `0b11101110`   |
| b       | `0b00111110`   |
| C       | `0b10011100`   |
| d       | `0b01111010`   |
| E       | `0b10011110`   |
| F       | `0b10001110`   |
| g       | `0b11110110`   |
| H       | `0b01101110`   |

Perfecto, pues una vez vistas las características del proyecto, vamos a ir directamente al código, el cual es largo, ya que implementa varias funciones, pero nada que no hayamos visto ya.

### Código

```c++
#include <Arduino.h>

// Pines del joystick
#define PIN_PULSADOR 2
#define PIN_X A1
#define PIN_Y A2
#define Y_MAYOR_700_SUBE false         // Cambia este valor para invertir el sentido vertical del joystick

// Pines de los segmentos
#define SEGMENTO_A 3
#define SEGMENTO_B 4
#define SEGMENTO_C 5
#define SEGMENTO_D 6
#define SEGMENTO_E 7
#define SEGMENTO_F 8
#define SEGMENTO_G 9
#define SEGMENTO_PUNTO 10

// Pines de los digitos
#define PIN_DIGITO_1 11
#define PIN_DIGITO_2 12
#define PIN_DIGITO_3 13
#define PIN_DIGITO_4 14

void apagarDigitos();                  // Apaga los 4 digitos
void elegirDigito(int digito);         // Enciende el dígito pasado y los demás los mantiene apagados
void siguientePantalla();              // Pasa a trabajar con la siguiente pantalla
void pintarSegmento(byte patron);      // Enciende los segmentos del dígito con el valor correspondiente
void mostrarInfo();                    // Imprime información relevante
void leerJoystick();                   // Lee los valores digitales y analógicos del joystick y los actualiza
void cambiarDigitoActualX();           // Cambia el digito actual con el eje X
void cambiarValorDigitoY();            // Cambia el valor del digito con el eje Y
void parpadeoPunto();                  // Hace parpadear el punto del digito actual y pinta el simbolo actual en ese momento
void ejecutarMovimiento();             // Ejecuta la jugada al pulsar el boton del joystick
void valoresDefecto();                 // Pone los valores iniciales para los 4 digitos: A0A0
bool detectar_flanco_subida();         // Detecta una pulsacion unica del boton

// Todos las letras y números codificados para el visualizador de 7 segmentos
const byte SEGMENTOS[] = {
  0b11111100,  // 0
  0b01100000,  // 1
  0b11011010,  // 2
  0b11110010,  // 3
  0b01100110,  // 4
  0b10110110,  // 5
  0b10111110,  // 6
  0b11100000,  // 7
  0b11111110,  // 8
  0b11100110,  // 9
  0b11101110,  // A
  0b00111110,  // b
  0b10011100,  // C
  0b01111010,  // d
  0b10011110,  // E
  0b10001110,  // F
  0b11110110,  // g
  0b01101110   // H
};

// Array constante de los diferentes números o letras que mostramos en el visualizador de 7 segmentos
const char SIMBOLOS[] = {
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
  'A', 'b', 'C', 'd', 'E', 'F', 'g', 'H'
};

// Declaración de variables globales

// Varaibles asociadas al visualizador de 7 segmentos
int simboloActual = 0;                  // Variable que guarda el simbolo actual que vamos o ya hemos pintado en uno de los digitos
int digitoActual = 1;                   // Variable que guarda el dígito con el que estamos trabajando en ese momento
int valoresDigitos[4] = {10, 0, 10, 0}; // Valores guardados en los 4 digitos (Por defecto la letra A y el número 0)

// Variables asociadas al joystick
int valor_x = 0;                        // Variable que guarda el valor en todo momento de la posición X del Joystick (Valor analógico)
int valor_y = 0;                        // Variable que guarda el valor en todo momento de la posición Y del Joystick (Valor analógico)
bool joystick_pulsado = 0;              // Variable booleana que guarda si el botón del Joystick está pulsado o no en todo momento

void setup() {
  Serial.begin(9600);

  pinMode(PIN_PULSADOR, INPUT_PULLUP);
  pinMode(PIN_X, INPUT);
  pinMode(PIN_Y, INPUT);

  pinMode(SEGMENTO_A, OUTPUT);
  pinMode(SEGMENTO_B, OUTPUT);
  pinMode(SEGMENTO_C, OUTPUT);
  pinMode(SEGMENTO_D, OUTPUT);
  pinMode(SEGMENTO_E, OUTPUT);
  pinMode(SEGMENTO_F, OUTPUT);
  pinMode(SEGMENTO_G, OUTPUT);
  pinMode(SEGMENTO_PUNTO, OUTPUT);

  pinMode(PIN_DIGITO_1, OUTPUT);
  pinMode(PIN_DIGITO_2, OUTPUT);
  pinMode(PIN_DIGITO_3, OUTPUT);
  pinMode(PIN_DIGITO_4, OUTPUT);

  // Nada más empezar el programa apago los digitos y enciendo el primero
  apagarDigitos();
  elegirDigito(digitoActual);

  // Para el primer digito elijo el valor establecido por defecto y lo pinto
  simboloActual = valoresDigitos[digitoActual - 1];
  pintarSegmento(SEGMENTOS[simboloActual]);
}

void loop() {
  leerJoystick();
  cambiarDigitoActualX();
  cambiarValorDigitoY();
  ejecutarMovimiento();
  parpadeoPunto();
  // mostrarInfo();
}

void apagarDigitos() {
  digitalWrite(PIN_DIGITO_1, HIGH);
  digitalWrite(PIN_DIGITO_2, HIGH);
  digitalWrite(PIN_DIGITO_3, HIGH);
  digitalWrite(PIN_DIGITO_4, HIGH);
}

void pintarSegmento(byte patron) {
  digitalWrite(SEGMENTO_A, bitRead(patron, 7));
  digitalWrite(SEGMENTO_B, bitRead(patron, 6));
  digitalWrite(SEGMENTO_C, bitRead(patron, 5));
  digitalWrite(SEGMENTO_D, bitRead(patron, 4));
  digitalWrite(SEGMENTO_E, bitRead(patron, 3));
  digitalWrite(SEGMENTO_F, bitRead(patron, 2));
  digitalWrite(SEGMENTO_G, bitRead(patron, 1));
  digitalWrite(SEGMENTO_PUNTO, bitRead(patron, 0));
}

void elegirDigito(int digito) {
  apagarDigitos();

  switch (digito) {
    case 1:
      digitalWrite(PIN_DIGITO_1, LOW);
      break;
    case 2:
      digitalWrite(PIN_DIGITO_2, LOW);
      break;
    case 3:
      digitalWrite(PIN_DIGITO_3, LOW);
      break;
    case 4:
      digitalWrite(PIN_DIGITO_4, LOW);
      break;
  }
}

void siguientePantalla() {
  digitoActual++;

  if (digitoActual > 4) {
    digitoActual = 1;
  }

  elegirDigito(digitoActual);
}

void mostrarInfo() {
  unsigned long ahora = millis();
  static unsigned long ultima_muestra = 0;

  // Imprimo mensajes cada medio segundo
  if (ahora - ultima_muestra >= 500) {
    Serial.print("Trabajando con el Digito: ");
    Serial.print(digitoActual);
    Serial.print("\tValor pintado: ");
    Serial.print(SIMBOLOS[simboloActual]);
    Serial.print("\tCoordenada X: ");
    Serial.print(valor_x);
    Serial.print("\tCoordenada Y: ");
    Serial.print(valor_y);
    Serial.print("\tJoystick Pulsado: ");
    if (joystick_pulsado) Serial.println("Si");
    else Serial.println("No");

    ultima_muestra = ahora; // Actualizo el valor de tiempo
  }
}

void leerJoystick() {
  valor_x = analogRead(PIN_X);
  valor_y = analogRead(PIN_Y);
  joystick_pulsado = digitalRead(PIN_PULSADOR) == LOW;
}

void cambiarDigitoActualX() {
  static bool joystickCentradoX = true;

  if (valor_x >= 480 && valor_x <= 580) {
    joystickCentradoX = true;
  }

  if (valor_x > 700 && joystickCentradoX) {
    digitoActual++;

    if (digitoActual > 4) {
      digitoActual = 1;
    }

    simboloActual = valoresDigitos[digitoActual - 1];
    elegirDigito(digitoActual);
    joystickCentradoX = false;
  }

  if (valor_x < 300 && joystickCentradoX) {
    digitoActual--;

    if (digitoActual < 1) {
      digitoActual = 4;
    }

    simboloActual = valoresDigitos[digitoActual - 1];
    elegirDigito(digitoActual);
    joystickCentradoX = false;
  }
}

void cambiarValorDigitoY() {
  static bool joystickCentradoY = true;
  int posicion = digitoActual - 1;
  bool subir = false;
  bool bajar = false;

  if (valor_y >= 480 && valor_y <= 580) {
    joystickCentradoY = true;
  }

  if (valor_y > 700 && joystickCentradoY) {
    if (Y_MAYOR_700_SUBE) subir = true;
    else bajar = true;
  }

  if (valor_y < 300 && joystickCentradoY) {
    if (Y_MAYOR_700_SUBE) bajar = true;
    else subir = true;
  }

  if (subir) {
    valoresDigitos[posicion]++;

    if (digitoActual == 1 || digitoActual == 3) {
      if (valoresDigitos[posicion] > 17) {
        valoresDigitos[posicion] = 10;
      }
    } else {
      if (valoresDigitos[posicion] > 9) {
        valoresDigitos[posicion] = 0;
      }
    }

    simboloActual = valoresDigitos[posicion];
    joystickCentradoY = false;
  }

  if (bajar) {
    valoresDigitos[posicion]--;

    if (digitoActual == 1 || digitoActual == 3) {
      if (valoresDigitos[posicion] < 10) {
        valoresDigitos[posicion] = 17;
      }
    } else {
      if (valoresDigitos[posicion] < 0) {
        valoresDigitos[posicion] = 9;
      }
    }

    simboloActual = valoresDigitos[posicion];
    joystickCentradoY = false;
  }
}

void parpadeoPunto() {
  unsigned long ahora = millis();
  static unsigned long ultimoParpadeo = 0;
  static bool puntoEncendido = false;
  byte patron = SEGMENTOS[simboloActual];

  if (ahora - ultimoParpadeo >= 500) {
    puntoEncendido = !puntoEncendido;
    ultimoParpadeo = ahora;
  }

  if (puntoEncendido) {
    patron = patron | 0b00000001; // Operación bitwise, hago un or con el último bit y así se enciende
  }

  pintarSegmento(patron);
}

void ejecutarMovimiento() {
  if (detectar_flanco_subida()) {
    Serial.print("Jugada ejecutada: ");
    Serial.print(SIMBOLOS[valoresDigitos[0]]);
    Serial.print(SIMBOLOS[valoresDigitos[1]]);
    Serial.print(SIMBOLOS[valoresDigitos[2]]);
    Serial.println(SIMBOLOS[valoresDigitos[3]]);

    valoresDefecto();
  }
}

void valoresDefecto() {
  valoresDigitos[0] = 10;
  valoresDigitos[1] = 0;
  valoresDigitos[2] = 10;
  valoresDigitos[3] = 0;

  digitoActual = 1;
  simboloActual = valoresDigitos[0];
  elegirDigito(digitoActual);
}

bool detectar_flanco_subida() {
  unsigned long ahora = millis();
  static bool estado_joystick_anterior = false;
  static unsigned long ultima_vez_leido = 0;

  // Compruebo cada 20ms
  if (estado_joystick_anterior != joystick_pulsado && ahora - ultima_vez_leido > 20) {
    ultima_vez_leido = ahora;

    if (estado_joystick_anterior == LOW && joystick_pulsado == HIGH) {
      estado_joystick_anterior = joystick_pulsado;
      return true;
    }

    estado_joystick_anterior = joystick_pulsado;
  }

  return false;
}
```

## Detalles sobre el código y posibles mejoras

* He implementado una **zona muerta del joystick** con rangos como `480–580` para evitar movimientos involuntarios cuando el joystick no está perfectamente centrado.
* El pulsador tiene **antirrebote por tiempo** usando `millis()`, de forma que no se detecten varias pulsaciones por una sola pulsación física.
* La constante `Y_MAYOR_700_SUBE` permite adaptar el sentido vertical del joystick sin tener que reescribir la lógica del programa.
* Otra posibilidad habría sido usar el **registro de desplazamiento 74HC595** para ahorrar pines del Arduino y simplificar parte del cableado del visualizador.
