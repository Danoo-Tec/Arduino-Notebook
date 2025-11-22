---
title: Proyecto 8 - Theremín controlado por luz
date: 2025-11-13
draft: false
---
## Entendiendo el Theremín y un ejemplo básico

En la clase de hoy hemos creado nuestro primer **theremín**. El theremín es un instrumento que cambia el sonido que se reproduce dependiendo de la luz.

La manera en la que vamos a lograr esto es haciendo uso de un **fotorresistor (LDR)**, ya usado anteriormente en otros proyectos para medir la luz. Este lee un valor entre **0 y 1023**, ya que está conectado a un **pin analógico**.

Luego, dependiendo de ese valor vamos a usar una pieza nueva llamada **zumbador** (buzzer), que es básicamente el reproductor de sonido. Existen dos tipos:

- **Activo**: solo puede estar encendido (5V) o apagado (0V), ya que internamente genera su propia señal.
- **Pasivo**: nos permite enviarle una frecuencia (Hz) para que suene de diferentes maneras.

Para poder mandarle esa frecuencia vamos a hacer uso de una función que ya viene preinstalada, una función _built‑in_ llamada **`tone()`**. A esta función le pasamos dos argumentos: el pin del zumbador, que tenemos definido al principio del programa, y la **frecuencia** a la que queremos que suene.

**OJO:** es importante saber que el tono no para hasta que:

- desconectemos la placa,
- cambiemos el firmware,
- o utilicemos la función **`noTone()`** con el pin del zumbador.

El código es el siguiente:

```cpp
#define FOTO_PIN A0
#define ZUMBADOR 9

const int MINIMO_ORIGEN = 500;
const int MAXIMO_ORIGEN = 1000;

int lectura_foto = 0;
int frecuencia = 0;

void setup() {
  Serial.begin(9600);
}

void loop() {
  delay(1);
  lectura_foto = analogRead(FOTO_PIN);

  frecuencia = map(lectura_foto, MINIMO_ORIGEN, MAXIMO_ORIGEN, 50, 6000);
  tone(ZUMBADOR, frecuencia);
  Serial.println("Lectura fotorresistor: " + String(lectura_foto));
}
```

### Comprobaciones de seguridad básicas

- Revisar que el zumbador pasivo está conectado sin necesidad de respetar polaridad.
- Evitar usar `tone()` en los pines **3** o **11** si se están utilizando funciones PWM, ya que comparten el **Timer 2**.
- Asegurarse de que el **LDR** no se conecta directamente entre 5V y GND sin una resistencia fija, para evitar riesgos eléctricos.
### Justificación del rango 50–6000 Hz

Utilizamos el rango de **50 Hz a 6000 Hz** porque:

- El oído humano es capaz de percibir frecuencias entre **20 Hz y 20 kHz**, pero en la práctica la mayoría de personas no oye por encima de **14 kHz**.
- El zumbador pasivo empleado en esta práctica ofrece su mejor rendimiento y claridad dentro del rango **50 Hz a 6 kHz**, por lo que es el intervalo más adecuado para que el theremín suene correctamente.

## Theremín con potenciómetro

Ahora que ya hemos visto cómo funciona el theremín básico, vamos a crear una **versión mejorada** que incluye un **potenciómetro**. ¿Para qué sirve? Pues nos permite **cambiar la frecuencia** que le mandamos al zumbador según la posición del potenciómetro: podemos generar tonos más graves, más agudos o incluso **apagar el zumbador**, algo muy útil porque a veces puede resultar bastante molesto.

En esencia, hacemos lo mismo que en el proyecto anterior: seguimos leyendo el valor del **fotorresistor (LDR)** para generar el tono, pero **mapeamos a rangos de frecuencia diferentes** dependiendo del valor del potenciómetro.

El código es el siguiente:

```cpp
#define FOTO_PIN A0
#define POTEN_PIN A1
#define ZUMBADOR 9

const int MINIMO_ORIGEN = 500;
const int MAXIMO_ORIGEN = 1000;

int lectura_foto = 0;
int lectura_poten = 0;
int frecuencia = 0;
String nivel = "Apagado";

void setup() {
  Serial.begin(9600);
}

void loop() {
  delay(1);
  lectura_foto = analogRead(FOTO_PIN);
  lectura_poten = analogRead(POTEN_PIN);

  Serial.println("Lectura fotorresistor: " + String(lectura_foto) + "	" +
                 "Lectura potenciómetro: " + String(lectura_poten) + "	" +
                 "Nivel de frecuencia: " + nivel);

  if (lectura_poten >= 0 && lectura_poten < 100) {
    noTone(ZUMBADOR);
    nivel = "Apagado";

  } else if (lectura_poten >= 100 && lectura_poten < 300) {
    // Frecuencia muy grave
    frecuencia = map(lectura_foto, MINIMO_ORIGEN, MAXIMO_ORIGEN, 100, 400);
    nivel = "Muy Grave";
    tone(ZUMBADOR, frecuencia);

  } else if (lectura_poten >= 300 && lectura_poten < 600) {
    // Frecuencia grave
    frecuencia = map(lectura_foto, MINIMO_ORIGEN, MAXIMO_ORIGEN, 400, 800);
    nivel = "Grave";
    tone(ZUMBADOR, frecuencia);

  } else if (lectura_poten >= 600 && lectura_poten < 900) {
    // Frecuencia poco aguda
    frecuencia = map(lectura_foto, MINIMO_ORIGEN, MAXIMO_ORIGEN, 800, 2000);
    nivel = "Agudo";
    tone(ZUMBADOR, frecuencia);

  } else if (lectura_poten >= 900 && lectura_poten < 1024) {
    // Frecuencia muy aguda
    frecuencia = map(lectura_foto, MINIMO_ORIGEN, MAXIMO_ORIGEN, 2000, 6000);
    nivel = "Muy Agudo";
    tone(ZUMBADOR, frecuencia);
  }
}
```

## Theremín con potenciómetro y alarma

Vamos con el último proyecto del día, una **versión modificada del theremín con potenciómetro**, a la que añadimos una **alarma SOS en código Morse**, activada mediante un **botón**.

La funcionalidad principal es la misma que en el proyecto anterior: el potenciómetro controla diferentes rangos de frecuencia del zumbador dependiendo de su posición. Pero ahora, si pulsamos el botón, el theremín **deja de funcionar temporalmente** para reproducir la secuencia SOS:

- **S → ...** (tres puntos)
- **O → ---** (tres rayas)
- **S → ...** (tres puntos)

La alarma suena **solo mientras mantenemos pulsado el botón**. En cuanto lo soltamos, el theremín vuelve a funcionar con normalidad. Para organizar mejor el código, añadimos dos funciones auxiliares: `punto()` y `raya()`.

El código es el siguiente:

```cpp
#define FOTO_PIN A0
#define POTEN_PIN A1
#define ZUMBADOR 9
#define BTN_PIN 7

const int MINIMO_ORIGEN = 500;
const int MAXIMO_ORIGEN = 1000;

int lectura_foto = 0;
int lectura_poten = 0;
int frecuencia = 0;

String nivel = "Apagado";

void setup() {
  Serial.begin(9600);
  pinMode(BTN_PIN, INPUT_PULLUP);
}

void loop() {
  delay(1);
  lectura_foto = analogRead(FOTO_PIN);
  lectura_poten = analogRead(POTEN_PIN);
  int estado = digitalRead(BTN_PIN);
  
  while (estado == LOW) {
    // Alarma SOS en Morse
    // S → ...
    punto(); punto(); punto();
    delay(300);

    // O → ---
    raya(); raya(); raya();
    delay(300);

    // S → ...
    punto(); punto(); punto();
    delay(1000);

    estado = digitalRead(BTN_PIN);
  }

  Serial.println("Lectura fotorresistor: " + String(lectura_foto) + "	" +
                 "Lectura potenciómetro: " + String(lectura_poten) + "	" +
                 "Nivel de frecuencia: " + nivel);
  
  if (lectura_poten >= 0 && lectura_poten < 100) {
    noTone(ZUMBADOR);
    nivel = "Apagado";

  } else if (lectura_poten >= 100 && lectura_poten < 300) {
    // Frecuencia muy grave
    frecuencia = map(lectura_foto, MINIMO_ORIGEN, MAXIMO_ORIGEN, 100, 400);
    nivel = "Muy Grave";
    tone(ZUMBADOR, frecuencia);

  } else if (lectura_poten >= 300 && lectura_poten < 600) {
    // Frecuencia grave
    frecuencia = map(lectura_foto, MINIMO_ORIGEN, MAXIMO_ORIGEN, 400, 800);
    nivel = "Grave";
    tone(ZUMBADOR, frecuencia);

  } else if (lectura_poten >= 600 && lectura_poten < 900) {
    // Frecuencia aguda
    frecuencia = map(lectura_foto, MINIMO_ORIGEN, MAXIMO_ORIGEN, 800, 2000);
    nivel = "Agudo";
    tone(ZUMBADOR, frecuencia);

  } else if (lectura_poten >= 900 && lectura_poten < 1024) {
    // Frecuencia muy aguda
    frecuencia = map(lectura_foto, MINIMO_ORIGEN, MAXIMO_ORIGEN, 2000, 6000);
    nivel = "Muy Agudo";
    tone(ZUMBADOR, frecuencia);
  }
}

void punto() {
  tone(ZUMBADOR, 1000);
  delay(200);
  noTone(ZUMBADOR);
  delay(200);
}

void raya() {
  tone(ZUMBADOR, 1000);
  delay(600);
  noTone(ZUMBADOR);
  delay(200);
}
```

