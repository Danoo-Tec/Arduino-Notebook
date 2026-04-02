---
title: Proyecto 19 - Motor paso a paso (PaP)
date: 2026-03-27
draft: false
---

## Funcionamiento y montaje del motor paso a paso (PaP)

En la clase de hoy hemos introducido un nuevo componente, el motor paso a paso, el cual es bastante diferente al motor DC. El motor paso a paso funciona de la siguiente manera: al hacer pasar corriente por una bobina del estator, el campo magnético creado hace que las marcas dentadas del estator y del rotor se alineen. En este tipo de motores, es el rotor el que está formado por un imán permanente, por lo que no necesita escobillas.

Algunas características de los motores PaP son las siguientes:

* Los motores paso a paso giran en incrementos discretos de forma precisa como respuesta a los impulsos eléctricos recibidos.
* La dirección de rotación del motor estará determinada por el orden de la secuencia de los pulsos recibidos.
* La **velocidad de rotación** del motor está determinada por la **frecuencia de los pulsos** recibidos.
* En este tipo de motores, **la posición es conocida mediante el seguimiento de los pulsos** que se le han enviado, **no existiendo** una realimentación de dicha posición.
* Comparado con un motor DC, su par motor disminuye a velocidades elevadas, pero es superior a velocidades bajas.

El modelo que encontramos en el kit de Arduino es el **28BYJ-48**, el cual tiene una caja reductora 1:64 que aumenta el torque y reduce la velocidad. Por otro lado, va a haber 32 posiciones para llegar a 360º (**2048 pasos** debido a esta reductora en paso completo).

Para controlar este motor paso a paso vamos a utilizar el siguiente driver (**ULN2003**), el cual es un circuito integrado capaz de manejar las corrientes demandadas por el motor **28BYJ-48**.
Las características de este driver son:

* 4 LED que indicarán cuál de las bobinas está activa en cada momento.
* Contiene diodos para el recorte de las sobretensiones por variación de corriente en las bobinas.
* Contiene un conector que evita conexiones erróneas.

Vista gran parte de la teoría sobre el nuevo componente, el motor paso a paso, y cómo conectarlo con el driver, vamos a ver el objetivo del proyecto de hoy:

Lo primero es que para el proyecto vamos a necesitar, como es lógico, el motor paso a paso y el driver para poder conectarlo. El proyecto consiste en realizar el segundero de un reloj analógico; es decir, desde una posición inicial cualquiera, el motor paso a paso (PaP) deberá realizar una vuelta completa en exactamente un minuto.

Y ahora, claro, te preguntarás: ¿y cómo consigo crear este segundero? Bueno, pues para lograrlo debemos tener varias cosas en cuenta, y entre ellas una de las más importantes es que para dar una vuelta completa (360º) deben darse **2048 pasos** (esto dando por hecho que es funcionamiento de **paso completo**; para funcionamiento de **medio paso** serían **4096 pasos**). Y como queremos dar una vuelta en 60 segundos, pues tendremos que dar 2048 pasos en 60.000 ms, que, sacando factor común 32, obtenemos que deben darse **64 pasos cada 1875 milisegundos**.

Un método simple sería repetir los siguientes 64 pasos un total de 32 veces:

* Quince repeticiones de 3 pasos de 29 ms.
* Quince repeticiones de 1 paso de 30 ms.
* 4 pasos de 30 ms.

Tal y como he comentado, este motor posee 4 bobinas en el estator, cuya activación sucesiva hace que se mueva el rotor, pero este movimiento se puede hacer de 3 formas distintas:

* **Wave drive**: se activa una sola bobina cada vez, y el rotor avanza un paso en cada cambio.
* **Paso completo**: se activan dos bobinas a la vez; el rotor avanza igualmente un paso, pero con más fuerza.
* **Medio paso**: combina una bobina y luego dos, pasando por una posición intermedia; así hace el doble de pasos por vuelta.

> Comentar también que el **pinout** del driver se compone de un pin que va a **VCC (+5 V)**, otro a **GND**, y 4 más (**IN1, IN2, IN3, IN4**) que van a pines digitales y hacen referencia a las diferentes bobinas que hay en el estator.

Bueno, bien, pero ahora te preguntarás: ¿y cómo escribo un paso? Bueno, pues lo primero que haremos será crear una tabla de pasos; esta será un *array*, en este caso para una ejecución de **wave drive**, con una sola bobina encendida cada vez, en la que estarán definidos todos los posibles casos, que serán o bien la primera, segunda, tercera o cuarta bobina encendida. Esto se ve de la siguiente manera:

```c++
const byte TABLA_PASOS[4] = { B1000, B0100, B0010, B0001 };
```

Donde haya un 1 será que la bobina está encendida y donde haya un 0, lo contrario.
Una vez definida la tabla, la manera que utilizaremos para encender o apagar cada bobina será de la siguiente manera:

```c++
digitalWrite(PIN_BOBINA1, bitRead(TABLA_PASOS[paso], 3));
digitalWrite(PIN_BOBINA2, bitRead(TABLA_PASOS[paso], 2));
digitalWrite(PIN_BOBINA3, bitRead(TABLA_PASOS[paso], 1));
digitalWrite(PIN_BOBINA4, bitRead(TABLA_PASOS[paso], 0));
```

Esto lo que dice es que, para cada pin de la bobina, lo encenderá o no dependiendo del paso. Es decir, por ejemplo, para el paso 1, que es **B1000**, pondrá activa la primera bobina y apagará las demás.

La función `bitRead(variable, num)` devolverá el bit que ocupa la posición *num*, empezando por el **LSB** (*Least Significant Bit*), es decir, el bit menos significativo.

Ahora ya sí podemos pasar al código:

```c++
#include <Arduino.h>

// Pines del ULN2003
#define PIN_BOBINA1 11
#define PIN_BOBINA2 10
#define PIN_BOBINA3 9
#define PIN_BOBINA4 8

// Wave drive: una bobina activa en cada paso
const byte TABLA_PASOS[4] = { B1000, B0100, B0010, B0001 };

void meterPaso(int paso);
void pasoReloj();
void apagarMotor();
void hacerBloque64Pasos();

void setup() {
  Serial.begin(9600);

  pinMode(PIN_BOBINA1, OUTPUT);
  pinMode(PIN_BOBINA2, OUTPUT);
  pinMode(PIN_BOBINA3, OUTPUT);
  pinMode(PIN_BOBINA4, OUTPUT);

  apagarMotor();
}

void loop() {
  // 32 bloques de 64 pasos = 2048 pasos = 1 vuelta en 60 s
  for (int i = 0; i < 32; i++) {
    hacerBloque64Pasos();
  }

  Serial.println("Vuelta completada en 60 segundos");

  // Después de dar una vuelta en 60 segundos se parará el motor y se quedará en bucle
  // infinito hasta que se pulse el botón de RESET y así volver a empezar
  apagarMotor();
  while (true) {}
}

void hacerBloque64Pasos() {
  // 15 veces: 3 pasos de 29 ms + 1 paso de 30 ms = 60 pasos
  for (int i = 0; i < 15; i++) {
    pasoReloj();
    delay(29);

    pasoReloj();
    delay(29);

    pasoReloj();
    delay(29);

    pasoReloj();
    delay(30);
  }

  // 4 pasos finales de 30 ms
  for (int i = 0; i < 4; i++) {
    pasoReloj();
    delay(30);
  }
}

void meterPaso(int paso) {
  // Así B1000 corresponde a la bobina 1, B0100 a la 2, etc.
  digitalWrite(PIN_BOBINA1, bitRead(TABLA_PASOS[paso], 3));
  digitalWrite(PIN_BOBINA2, bitRead(TABLA_PASOS[paso], 2));
  digitalWrite(PIN_BOBINA3, bitRead(TABLA_PASOS[paso], 1));
  digitalWrite(PIN_BOBINA4, bitRead(TABLA_PASOS[paso], 0));
}

void pasoReloj() {
  static int paso = 0;

  meterPaso(paso);
  paso = (paso + 1) % 4;
}

void apagarMotor() {
  digitalWrite(PIN_BOBINA1, LOW);
  digitalWrite(PIN_BOBINA2, LOW);
  digitalWrite(PIN_BOBINA3, LOW);
  digitalWrite(PIN_BOBINA4, LOW);
}
```
