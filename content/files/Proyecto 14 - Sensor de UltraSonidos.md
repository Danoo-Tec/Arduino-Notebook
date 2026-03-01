---
title: Proyecto 14 - Sensor de ultrasonidos
date: 2026-02-20
draft: false
---

## Sensor de ultrasonidos en Arduino

Bueno, como veis en el título del proyecto de hoy vamos a aprender a utilizar un sensor de ultrasonidos en Arduino, con el cual vamos a poder medir la distancia al objeto que se encuentre enfrente del sensor e incluso la velocidad a la que se ha movido un objeto de un lado a otro en una ventana de tiempo.

Pero vamos por partes. Para el proyecto de hoy vamos a utilizar los siguientes componentes:

* Sensor de **ultrasonidos (HC-SR04)**
* Pantalla LCD 16x2
* Potenciómetro (para controlar el contraste de la pantalla LCD)
* Resistencia de 220 Ω (para la retroiluminación del LCD)

Perfecto, dicho esto vamos a ver cómo montar este **sensor de ultrasonidos**. Mirando el módulo de frente (es decir, viendo los dos transductores), el pin de la izquierda (VCC) irá a **5V** y el de la otra esquina, nombrado como **GND**, irá a **tierra**.

En el centro tenemos, en orden de izquierda a derecha, el pin de **Trigger** y el pin de **Echo**. Estos dos pines van conectados a pines digitales del Arduino.

Vale, pero ¿qué hacen exactamente estos dos pines? Primero tenemos el pin **Trigger**, que es muy importante que lo configuremos como **OUTPUT**, ya que será el que dispare la onda ultrasónica. Por otro lado, tenemos el pin **Echo**, que estará configurado como **INPUT**, ya que es el que utilizaremos con una función que nos medirá el tiempo que ha tardado la onda disparada por el **Trigger** en ir y volver.

Esta función es:

```cpp
pulseIn(PIN_ECHO, HIGH);
```

A esta función le pasamos el pin **Echo** y el estado **HIGH**, porque queremos medir cuánto tiempo permanece el pin en nivel alto, lo cual corresponde al tiempo total de ida y vuelta de la onda.

La función `pulseIn()` devuelve un valor de tipo `unsigned long` (32 bits), expresado en microsegundos, lo que permite medir tiempos de hasta varios segundos sin desbordamiento.

Ahora bien, ¿cómo enviamos el pulso a través del pin **Trigger**? De la siguiente manera:

```cpp
digitalWrite(PIN_TRIGGER, HIGH);
delayMicroseconds(10);
digitalWrite(PIN_TRIGGER, LOW);
```

En este caso ponemos el **Trigger** a 5V durante 10 microsegundos para generar el pulso de disparo. La función `delayMicroseconds(10)` es necesaria porque el sensor HC-SR04 requiere un pulso de al menos **10 µs** en el pin TRIGGER para iniciar la medición.

Tras recibir ese pulso de 10 µs en el pin **TRIGGER**, el módulo **HC-SR04 emite 8 ciclos de ultrasonido a una frecuencia de 40 kHz**, iniciando así la medición del tiempo de vuelo de la onda.

Es estándar y buena práctica asegurar que el **Trigger** esté en **LOW** un momento antes del pulso, ya que puede evitar lecturas erróneas si el pin no está claramente en LOW antes. Esto se puede lograr añadiendo lo siguiente antes de poner el **Trigger** en **HIGH**:

```cpp
digitalWrite(PIN_TRIGGER, LOW);
delayMicroseconds(2);
```

Otro punto importante es que podemos calcular la distancia. Para entender cómo lograrlo, tenemos que saber que la velocidad del sonido en el aire es aproximadamente **343 metros por segundo**.

La fórmula que utilizamos es: 

D = (t * v) / 2

Donde:

- D es la distancia.
- t es el tiempo total de ida y vuelta.
- v es la velocidad del sonido.

Se divide entre 2 porque el tiempo medido corresponde al recorrido de ida y vuelta.

> Punto importante: `pulseIn()` devuelve el tiempo en **microsegundos**, por lo que debemos usar la velocidad en unidades coherentes. Una forma práctica es utilizar directamente la velocidad del sonido como **0.0343 cm/µs**.

Bueno, ahora que ya sabemos toda la teoría importante, vamos al código que he creado. Empieza preguntando al usuario por un valor en milisegundos que espera entre cálculos del tiempo y la distancia. Luego, haciendo uso de `millis()`, realizo esto de forma no bloqueante. Por último, aparte de hacer los cálculos, voy imprimiendo el tiempo transcurrido para esa onda (en microsegundos) y la distancia calculada (en centímetros).

El código es el siguiente:

```cpp
#include <Arduino.h>        // No necesario si estas trabajando con el IDE de Arduino
#include <LiquidCrystal.h>

// PIN de RS (Register Selection), selector de registro (para enviar datos o comandos)
#define PIN_RS 8

// PIN Enable para indicar al LCD cuándo debe leer lo que hay en las líneas de datos
#define PIN_ENABLE 9

// Pines D4-D7
#define PIN_D4 10
#define PIN_D5 11
#define PIN_D6 12
#define PIN_D7 13

// Pines sensor de ultrasonidos
#define PIN_ECHO 7
#define PIN_TRIGGER 6

LiquidCrystal lcd(PIN_RS, PIN_ENABLE, PIN_D4, PIN_D5, PIN_D6, PIN_D7);

// Variables
unsigned long actual = 0;
unsigned long pasado = 0;
unsigned long valor_ms = 0;

// Prototipos
long leer_numero();
unsigned long calcular_tiempo();
unsigned long calcular_distancia(unsigned long tiempo);
void mostrar_info_lcd(unsigned long tiempo, unsigned long distancia);

void setup() {
  Serial.begin(9600);
  lcd.begin(16, 2); // Inicializo la pantalla LCD

  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_TRIGGER, OUTPUT);

  Serial.println("Bienvenido al medidor de ultrasonidos");
  Serial.print("Cada cuanto quieres mediar la distancia (ms): ");

  // Le pregunto al usuario por los datos
  valor_ms = leer_numero();
}

void loop() {
  actual = millis(); // Guardo el tiempo actual (tiempo transcurrido desde que empezó el programa)

  // Calculo la distancia cada X tiempo
  if (actual - pasado >= valor_ms) {
    // Calculo del tiempo
    unsigned long tiempo = calcular_tiempo();

    // Calculo la distancia
    unsigned long distancia = calcular_distancia(tiempo);

    // Imprimo por consola
    Serial.print("Tiempo (microsegundos): ");
    Serial.print(tiempo);
    Serial.print("    Distancia (cm): ");
    Serial.println(distancia);

    // Imprimo los detalles
    mostrar_info_lcd(tiempo, distancia);

    pasado = actual;
  }
}

long leer_numero() { // Función que lee un número (es bloqueante)
  while (!Serial.available()); // Espero mientras el usuario no introduzca nada
  long num = Serial.parseInt();

  // Limpio el buffer
  while (Serial.available()) Serial.read();

  // Añado salto de línea;
  Serial.println("");

  // Compruebo los valores introducidos
  if (num <= 0) {
    num = 500;
    Serial.println("Se ha ledido un valor no válido");
    Serial.println("Por defecto se pondrá 500ms");
  } else {
    Serial.print("El usuario ha introducido ");
    Serial.print(num);
    Serial.println("ms.");
  }
  return num;
}

unsigned long calcular_tiempo() {
  digitalWrite(PIN_TRIGGER, LOW);
  delayMicroseconds(2);
  digitalWrite(PIN_TRIGGER, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIGGER, LOW);

  // Guardo el tiempo que ha tardado la onda
  unsigned long tiempo = pulseIn(PIN_ECHO, HIGH);

  return tiempo;
}

unsigned long calcular_distancia(unsigned long tiempo) {
  // Calculo la distancia
  unsigned long distancia = (tiempo * 0.0343 / 2);

  return distancia;
}

void mostrar_info_lcd(unsigned long tiempo, unsigned long distancia) {
  lcd.clear();  // Limpio la pantalla

  // Primera línea: Tiempo
  lcd.setCursor(0, 0);
  lcd.print("T(us): ");
  lcd.print(tiempo);

  // Segunda línea: Distancia
  lcd.setCursor(0, 1);
  lcd.print("D(cm): ");

  // Rango típico HC-SR04: 2 cm – 400 cm
  if (distancia < 2 || distancia > 400) {
    lcd.print("Err rango");
  } else {
    lcd.print(distancia);
  }
}

```

---

## Cálculo de la velocidad

Como he dicho antes, aparte del tiempo que tarda una onda en ir y volver y la distancia al objeto, también se puede calcular la velocidad de un objeto que se ha movido en una ventana de tiempo.

En este caso, en la pantalla LCD, en vez de imprimir el tiempo y la distancia, vamos a imprimir en la fila de arriba (fila 0) la distancia al objeto en tiempo real y, en la de abajo, la velocidad calculada en la ventana de tiempo introducida por el usuario al principio del programa.

No entraré en más detalles sobre cómo calculo la velocidad, ya que el código está bastante bien comentado.

El código sería el siguiente:

```cpp
#include <Arduino.h>        // No necesario si estas trabajando con el IDE de Arduino
#include <LiquidCrystal.h>

// PIN de RS (Register Selection), selector de registro (para enviar datos o comandos)
#define PIN_RS 8

// PIN Enable para indicar al LCD cuándo debe leer lo que hay en las líneas de datos
#define PIN_ENABLE 9

// Pines D4-D7
#define PIN_D4 10
#define PIN_D5 11
#define PIN_D6 12
#define PIN_D7 13

// Pines sensor de ultrasonidos
#define PIN_ECHO 7
#define PIN_TRIGGER 6

LiquidCrystal lcd(PIN_RS, PIN_ENABLE, PIN_D4, PIN_D5, PIN_D6, PIN_D7);

// Variables
unsigned long actual = 0;
unsigned long pasado = 0;
unsigned long valor_ms = 0;

unsigned long distancia_anterior_cm = 0;
bool primera_medicion = true;

// Prototipos
long leer_numero();
unsigned long calcular_tiempo_us();
unsigned long calcular_distancia_cm(unsigned long tiempo_us);
float medir_velocidad_kmh(unsigned long distancia_anterior_cm, unsigned long distancia_actual_cm, unsigned long delta_t_ms);
void mostrar_info_lcd(unsigned long tiempo_us, unsigned long distancia_cm, float velocidad_kmh, bool distancia_valida);
void mostrar_info_monitor(unsigned long tiempo_us, unsigned long distancia_cm, float velocidad_kmh);
bool distancia_en_rango(unsigned long distancia_cm);

void setup() {
  Serial.begin(9600);
  lcd.begin(16, 2); // Inicializo la pantalla LCD

  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_TRIGGER, OUTPUT);

  Serial.println("Bienvenido al medidor de ultrasonidos");
  Serial.print("Cada cuanto quieres mediar la distancia (ms): ");

  // Le pregunto al usuario por los datos
  valor_ms = leer_numero();
  pasado = millis(); // inicializo referencia temporal
}

void loop() {
  actual = millis();

  // Medición cada X ms (valor_ms)
  if (actual - pasado >= valor_ms) {

    // Δt real entre muestras
    unsigned long delta_t_ms = actual - pasado;

    // 1) Tiempo del echo (us) (ida y vuelta de la onda)
    unsigned long tiempo_us = calcular_tiempo_us();

    // 2) Distancia (cm)
    unsigned long distancia_cm = calcular_distancia_cm(tiempo_us);

    // 3) Validación de rango
    bool validez_dist_actual = distancia_en_rango(distancia_cm);
    bool validez_dist_anterior = distancia_en_rango(distancia_anterior_cm);

    // 4) Velocidad (km/h)
    float velocidad_kmh = 0.0f;

    // Si no estoy en la primera medición y los valores de distancia son ambos válidos
    if (primera_medicion == false && validez_dist_actual && validez_dist_anterior) {
      velocidad_kmh = medir_velocidad_kmh(distancia_anterior_cm, distancia_cm, delta_t_ms);
    } else {
      // Primera medición o valores inválidos: no calculo velocidad
      velocidad_kmh = 0.0f;
    }

    // Consola / Serial Monitor - Imprimo la info por el monitor
    mostrar_info_monitor(tiempo_us, distancia_cm, velocidad_kmh);

    // LCD - Muestro la info en la pantalla LCD
    mostrar_info_lcd(tiempo_us, distancia_cm, velocidad_kmh, validez_dist_actual);

    // Actualizo estado para siguiente iteración
    distancia_anterior_cm = distancia_cm;
    primera_medicion = false;

    pasado = actual;
  }
}

long leer_numero() { // Función que lee un número (es bloqueante)
  while (!Serial.available()); // Espero mientras el usuario no introduzca nada
  long num = Serial.parseInt();

  // Limpio el buffer
  while (Serial.available()) Serial.read();

  // Añado salto de línea;
  Serial.println("");

  // Compruebo los valores introducidos
  if (num <= 0) {
    num = 500;
    Serial.println("Se ha ledido un valor no válido");
    Serial.println("Por defecto se pondrá 500ms");
  } else {
    Serial.print("El usuario ha introducido ");
    Serial.print(num);
    Serial.println("ms.");
  }
  return num;
}

unsigned long calcular_tiempo_us() {
  digitalWrite(PIN_TRIGGER, LOW);
  delayMicroseconds(2);
  digitalWrite(PIN_TRIGGER, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIGGER, LOW);

  // Guardo el tiempo que ha tardado la onda
  unsigned long tiempo = pulseIn(PIN_ECHO, HIGH);

  return tiempo;
}

unsigned long calcular_distancia_cm(unsigned long tiempo_us) {
  // Velocidad del sonido ~0.0343 cm/us
  // distancia = tiempo * v / 2 (ida y vuelta)
  unsigned long distancia = (tiempo_us * 0.0343 / 2.0);

  return distancia;
}

bool distancia_en_rango(unsigned long distancia_cm) {
  // Compruebo que esté dentro de un rango razonable
  return !(distancia_cm <= 2 || distancia_cm >= 2800);
}

float medir_velocidad_kmh(unsigned long distancia_anterior_cm, unsigned long distancia_actual_cm, unsigned long delta_t_ms) {
  // Devuelve km/h.
  // Signo:
  // - positivo => se aleja (distancia aumenta)
  // - negativo => se acerca (distancia disminuye)

  if (delta_t_ms == 0) return 0.0f;

  // Δd en cm (puede ser negativo)
  long delta_d_cm = (long)distancia_actual_cm - (long)distancia_anterior_cm;

  // Convertimos:
  // cm -> km : dividir entre 100000
  // ms -> h  : ms / (1000*3600) => dividir por 3,600,000
  // v(km/h) = (delta_d_cm/100000) / (delta_t_ms/3600000)

  float delta_d_km = (float)delta_d_cm / 100000.0;
  float delta_t_h  = (float)delta_t_ms / 3600000.0;

  return delta_d_km / delta_t_h;
}

void mostrar_info_lcd(unsigned long tiempo_us, unsigned long distancia_cm, float velocidad_kmh, bool distancia_valida) {
  lcd.clear();

  // Línea 1: Distancia
  lcd.setCursor(0, 0);
  lcd.print("D: ");
  if (!distancia_valida) {
    lcd.print("Err");
  } else {
    lcd.print(distancia_cm);
    lcd.print("cm");
  }

  // Línea 2: Velocidad (km/h)
  lcd.setCursor(0, 1);
  lcd.print("V: ");
  if (!distancia_valida) {
    lcd.print("Err");
  } else {
    // Muestra con 1 decimal (en 16x2 cabe mejor)
    lcd.print(velocidad_kmh, 1);
    lcd.print("km/h");
  }
}

void mostrar_info_monitor(unsigned long tiempo_us, unsigned long distancia_cm, float velocidad_kmh) {
  Serial.print("Tiempo (us): ");
  Serial.print(tiempo_us);
  Serial.print("\tDistancia (cm): ");
  Serial.print(distancia_cm);

  Serial.print("\tVelocidad (km/h): ");
  Serial.println(velocidad_kmh, 2);
}

```