---
title: Proyecto 9 - Sensor de humedad y temperatura DHT11
date: 2025-11-20
draft: false
---
## Creando un sensor de Humedad y Temperatura

Bueno, para el proyecto vamos por primera vez a instalar una librería que no viene por defecto instalada en el IDE de Arduino para construir un **sensor de humedad y temperatura**. Esta librería que vamos a instalar se llama **DHT sensor library**, y es de _Adafruit_. Esta librería es la que nos va a permitir leer la temperatura y la humedad del sensor.

Pero claro, os estaréis preguntando qué sensor usamos. Bueno, pues la nueva pieza de hardware que añadimos esta semana se llama **DHT11**, y viene por defecto en la caja de Arduino UNO.

Algunas cositas a tener en cuenta a la hora de usar este sensor es que hay que fijarse muy bien en cómo lo conectamos, y que, en caso de conectarlo al revés, lo podemos quemar. La manera de nunca liarla al conectar el sensor es fijarse donde pone la **"S" de Signal**, que será el pin digital al que lo conectamos. Luego, en el kit, suele ser que hacia el otro extremo estén **5V** y **GND**, pero es cuestión de mirarlo.

La librería, una vez instalada, la vamos a incluir con:

```cpp
#include <DHT.h>
```

Luego, para crear nuestro sensor, hacemos:

```cpp
DHT sensorName(SENSOR_PIN, DHT11);
```

El `DHT11` se pone al final porque ese es el tipo de sensor que tenemos en el kit.

Para leer la temperatura y la humedad lo haremos con los siguientes métodos:

```cpp
sensorName.readTemperature();
sensorName.readHumidity();
```

Por último, para tener algo más funcional y chulo que simplemente leer la temperatura y la humedad, vamos a tener un LED para cada uno de esos valores, y vamos a hacer que se enciendan si pasan de X valor.

El código es el siguiente:

```cpp
#include <DHT.h>

#define LED_ROJO_PIN 8
#define LED_VERDE_PIN 9
#define SENSOR 7

DHT sensor_temp(SENSOR, DHT11);

void setup() {
  Serial.begin(9600);
  sensor_temp.begin();
  pinMode(LED_ROJO_PIN, OUTPUT);
  pinMode(LED_VERDE_PIN, OUTPUT);

  // Apago las luces por defecto
  digitalWrite(LED_ROJO_PIN, LOW);
  digitalWrite(LED_VERDE_PIN, LOW);
}

void loop() {
  // En caso de poner true dentro del paréntesis, leerás Fahrenheit
  float temperatura = sensor_temp.readTemperature();
  float humedad = sensor_temp.readHumidity();

  Serial.println("Temperatura leida: " + String(temperatura) + "\tHumedad: " + String(humedad));

  // LED rojo: en base a la humedad
  if (humedad >= 50) {
    digitalWrite(LED_ROJO_PIN, HIGH);
  } else {
    digitalWrite(LED_ROJO_PIN, LOW);
  }

  // LED verde: en base a la temperatura
  if (temperatura >= 25) {
    digitalWrite(LED_VERDE_PIN, HIGH);
  } else {
    digitalWrite(LED_VERDE_PIN, LOW);
  }
}
```

## Sensor con valores establecidos por el usuario

Bueno, esto es una pequeña modificación añadida al proyecto anterior. El cambio principal es que he añadido la funcionalidad para que el usuario introduzca los **valores límite** antes de que se encienda el LED respectivo. Luego, por otro lado, la funcionalidad que enciende y apaga los LEDs en base a estos valores introducidos por el usuario la he sacado a una función externa llamada `led_control()`.

Para leer la entrada del usuario lo hacemos con la siguiente línea de código:

```cpp
while (Serial.available() == 0)
```

Esperamos mientras la entrada sea 0, es decir, mientras el usuario no haya introducido nada. Luego, ese valor lo parseamos, es decir, lo convertimos de un **string** que hemos leído a un **entero** para poder trabajar con él.

El código es el siguiente:

```cpp
#include <DHT.h>

#define LED_ROJO_PIN 8
#define LED_VERDE_PIN 9
#define SENSOR 7

DHT sensor_temp(SENSOR, DHT11);
int limite_temp = 0;
int limite_hum = 0;

void setup() {
  Serial.begin(9600);
  sensor_temp.begin();
  pinMode(LED_ROJO_PIN, OUTPUT);
  pinMode(LED_VERDE_PIN, OUTPUT);

  // Apago las luces por defecto
  digitalWrite(LED_ROJO_PIN, LOW);
  digitalWrite(LED_VERDE_PIN, LOW);

  Serial.println("=============================================================================");
  Serial.println("Los valores que introduzcas serán a partir de los cuales saltarán las alarmas");
  Serial.println("=============================================================================");

  Serial.println("Introduce el límite de temperatura para la alarma: ");
  while (Serial.available() == 0) {
    delay(10);
  }

  limite_temp = Serial.parseInt();
  
  // Limpia el buffer de saltos líneas (\n) etc..
  Serial.read();

  Serial.println("Introduce el límite de humedad para la alarma: ");
  while (Serial.available() == 0) {
    delay(10);
  }

  limite_hum = Serial.parseInt();

  Serial.println("El límite de temperatura para la alarma es: " + String(limite_temp));
  Serial.println("El límite de humedad para la alarma es: " + String(limite_hum));
  delay(5000);
}

void loop() {
  float temperatura = sensor_temp.readTemperature();
  float humedad = sensor_temp.readHumidity();

  Serial.println("Temperatura leída: " + String(temperatura) + "	Humedad: " + String(humedad));
  led_control(temperatura, limite_temp, humedad, limite_hum);
}

void led_control(float temperatura, int temp_lim, float humedad, int hum_lim) {
  // LED rojo: en base a la humedad
  if (humedad >= hum_lim) {
    digitalWrite(LED_ROJO_PIN, HIGH);
  } else {
    digitalWrite(LED_ROJO_PIN, LOW);
  }

  // LED verde: en base a la temperatura
  if (temperatura >= temp_lim) {
    digitalWrite(LED_VERDE_PIN, HIGH);
  } else {
    digitalWrite(LED_VERDE_PIN, LOW);
  }
}
```