---
title: Proyecto 6 - Mezclando Colores
date: 2025-10-30
draft: false
---
## 1. Mezclando colores en función de la luz

Para el proyecto de hoy, vamos a hacer uso de las salidas **pseudoanalógicas**, que son aquellos pines con *virgulilla* (~), los cuales nos van a permitir generar una salida **PWM** que indicaremos con la función `analogWrite()`. Esta función nos permite controlar la potencia media en el pin y, de ahí, entregar más o menos **brillo** a un LED. Esto lo vamos a juntar con un LED RGB para conseguir una amplia gama de colores. Aparte, haremos que la potencia que entregamos a cada uno de estos colores se mida mediante una fotorresistencia, es decir, una resistencia que varía según la luminosidad.

Los materiales ideales serían usar **3 fotorresistencias**, pero los kits de Arduino solo vienen con dos, por tanto usaremos dos fotorresistencias y un potenciómetro para conseguir un resultado parecido. Por lo demás:

- 1 LED RGB
- 2 Resistencias de 10 kΩ (para las fotorresistencias)
- 3 Resistencias de 220 Ω (una para cada color del LED)
- 2 Fotorresistencias (LDR, light-dependent resistor) – A mayor luz, menor resistencia
- 1 Potenciómetro

Estas fotorresistencias tienen dos terminales y no tienen polaridad. Se conectan formando un **divisor de tensión** con una resistencia fija de 10 kΩ: un extremo del LDR va a **5 V**, el otro extremo va al **pin analógico** y desde ese nodo se coloca la **resistencia de 10 kΩ a GND** (o al revés: LDR a 5 V y resistencia a GND, manteniendo el punto medio al pin analógico). El potenciómetro, que tiene tres terminales, se conecta con los extremos a **5 V** y **GND**, y el **cursor (wiper)** al **pin analógico**.

Los valores que recoge la función `analogWrite()` son entre **0 y 255**, siendo **0** apagado y **255** encendido a máxima potencia (ciclo de trabajo del 100 %).

Una función nueva que vemos también es la función **`map()`**, que nos permitirá mapear una señal **origen** comprendida entre dos valores (mínimo y máximo) a otra que tiene otros valores de mínimo y máximo. Se usa de la siguiente manera:

```c
 destino = map(valor_origen, minimo_origen, maximo_origen, minimo_destino, maximo_destino);
```

En este ejemplo, el **valor origen** será el que leamos del pin analógico. El **mínimo y máximo de origen** son valores que calibramos nosotros dependiendo de qué valores obtenemos con una luz muy baja (por ejemplo, tapando la fotorresistencia) y con una luz muy alta (por ejemplo, apuntando con la linterna del móvil). Ahora bien, eso cambia si la utilizamos para el potenciómetro ya que sabemos que ese siempre va a ir de 0 a 1023. Por otro lado, el **mínimo y máximo de destino** serán los valores que queremos entregar a `analogWrite()`, en este caso **entre 0 y 255**.

El código final es el siguiente:

```c
#define LED_ROJO 11
#define LED_VERDE 10
#define LED_AZUL 9

#define POTEN A2
#define FOTO_1 A1
#define FOTO_2 A0

const int MINIMO_ORIGEN = 500;
const int MAXIMO_ORIGEN = 1000;

int lectura_1 = 0;
int lectura_2 = 0;
int lectura_3 = 0;

void setup() {
  Serial.begin(9600);
  pinMode(LED_ROJO, OUTPUT);
  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_AZUL, OUTPUT);

  pinMode(POTEN, INPUT);
  pinMode(FOTO_1, INPUT);
  pinMode(FOTO_2, INPUT);

  digitalWrite(LED_ROJO, LOW);
  digitalWrite(LED_VERDE, LOW);
  digitalWrite(LED_AZUL, LOW);
}

void loop() {
  lectura_1 = map(analogRead(A0), MINIMO_ORIGEN, MAXIMO_ORIGEN, 0, 255);
  lectura_2 = map(analogRead(A1), MINIMO_ORIGEN, MAXIMO_ORIGEN, 0, 255);
  lectura_3 = map(analogRead(A2), 0, 1023, 0, 255);

  Serial.println("Lectura 1: " + String(lectura_1) + "\t" + "Lectura 2: " + String(lectura_2) + "\t" + "Lectura 3: " + String(lectura_3));

  leds(lectura_1, lectura_2, lectura_3);
}

void leds(int uno, int dos, int tres) {
  analogWrite(LED_ROJO, uno);
  analogWrite(LED_VERDE, dos);
  analogWrite(LED_AZUL, tres);  
}
```

## 2. Arcoíris RGB

Para este otro proyecto no vamos a necesitar ni fotorresistencias ni potenciómetros; simplemente vamos a utilizar el **LED RGB** para conseguir un efecto **arcoíris**, es decir, que pase por todos los colores RGB en bucle. Como es lógico, vamos a añadir un pequeño `delay` entre cada cambio de color para que nuestro ojo lo pueda apreciar.

El código sería el siguiente:

```c
#define LED_ROJO 11
#define LED_VERDE 10
#define LED_AZUL 9

void setup() {
  Serial.begin(9600);
  pinMode(LED_ROJO, OUTPUT);
  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_AZUL, OUTPUT);

  digitalWrite(LED_ROJO, LOW);
  digitalWrite(LED_VERDE, LOW);
  digitalWrite(LED_AZUL, LOW);
}

void loop() {
  for (int i = 0; i <= 255; i++) {
    analogWrite(LED_ROJO, 255 - i);
    analogWrite(LED_VERDE, i);
    analogWrite(LED_AZUL, 0);
    delay(10);
  }

  for (int i = 0; i <= 255; i++) {
    analogWrite(LED_ROJO, 0);
    analogWrite(LED_VERDE, 255 - i);
    analogWrite(LED_AZUL, i);
    delay(10);
  }

  for (int i = 0; i <= 255; i++) {
    analogWrite(LED_ROJO, i);
    analogWrite(LED_VERDE, 0);
    analogWrite(LED_AZUL, 255 - i);
    delay(10);
  }
}
```
