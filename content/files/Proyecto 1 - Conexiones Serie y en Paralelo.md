---
title: "Proyecto 1 - Conexiones Serie y en Paralelo"
date: 2025-09-25
draft: false
---
## 1. Con pulsador

**Objetivos**:
- Usar la **placa Arduino como fuente de 5 V y GND** en la *protoboard*.
- Entender qué cambia entre **serie** y **paralelo** (corriente/tensión).
- Controlar el **encendido de LEDs** con **pulsadores**.

**Componentes y símbolos:**
- **LED**, **resistencia 220 Ω**, **pulsador** (push button).
- Recuerda: LED → **ánodo (pata larga) a +**, **cátodo (pata corta) a –**.

**Soluciones posibles:**
 Si quieres hacerlo **sin código**, basta con intercalar el **pulsador en serie** en el camino del LED: por ejemplo, **+5 V → resistencia 220 Ω → ánodo LED → cátodo LED → pulsador → GND** (o poner el pulsador entre +5 V y la resistencia; eléctricamente es equivalente). Así, al pulsar **cierras el circuito** y el LED se enciende; al soltar, se **abre** y se apaga. 
 
 Si prefieres controlarlo **por código**, entonces separas claramente **entrada** y **salida**: el LED se conecta como **salida** desde el **pin 9** (pin 9 → resistencia 220 Ω → ánodo LED; cátodo a GND), y el **pulsador como entrada** en el **pin 2** usando **INPUT_PULLUP**, que activa una resistencia interna al +5 V. En esta configuración el botón **solo** va entre **pin 2 y GND** (no toca el circuito del LED), de modo que **suelto = HIGH (1)** y **pulsado = LOW (0)**; es decir, la lógica queda **invertida**. En el programa lees el estado del pin 2 y, según sea LOW o HIGH, mandas **HIGH o LOW** al pin 9 para encender o apagar el LED. 
 
 > Error común: **no mezclar** el botón con el nodo del LED; comparten **GND**, pero cada uno sigue su camino.

El código es el siguiente:

```c
const int PIN_LED = 9;
const int PIN_BTN = 2;

void setup() {
  Serial.begin(9600);
  pinMode(PIN_LED, OUTPUT);
  pinMode(PIN_BTN, INPUT_PULLUP); // activa pull-up interno
}

void loop() {
  int estado = digitalRead(PIN_BTN); // 1 = suelto, 0 = pulsado
  Serial.println(estado);

  if (estado == LOW) {     // pulsado
  digitalWrite(PIN_LED, HIGH);
  } else {                  // suelto
    digitalWrite(PIN_LED, LOW);
  }
  // digitalWrite(PIN_LED, (estado == LOW) ? HIGH : LOW);
}
```

## 2. Sin pulsador - Por consola

En este caso, para controlar el encendido y apagado del led solo y exclusivamente desde la consola no nos hará falta un pulsador, solo un pin el cual pondremos en modo **output** para poder mandar o bien 0V (**Low**) o bien 5V (**High**). Con la función **Serial.available() > 0** comprobamos si hay algo que leer por consola y en caso de haberlo entramos en la condición y leemos lo que haya, que será un char. Y ya dependiendo de su valor actuamos en consecuencia.

```c
const int PIN = 9;
char opcion = '\0';

void setup() {
  Serial.begin(9600);
  pinMode(PIN, OUTPUT);
  Serial.println("Bienvenido al programa");
}

void loop() {
  if (Serial.available() > 0) {
    opcion = Serial.read();
    opcion = tolower(opcion);

    if (opcion == '\n') return; // Ignora este tipo de caracteres

    switch (opcion) {
      case 'e': digitalWrite(PIN, HIGH); break;
      case 'a': digitalWrite(PIN, LOW); break;
      default: Serial.println("Opcion no valida");
    }

	// También se podría hacer con una estructra condicional
    // if (opcion == 'e') {
    //   digitalWrite(PIN, HIGH);
    // } else if (opcion == 'a') {
    //   digitalWrite(PIN, LOW);
    // } else {
    //   Serial.println("Opcion no valida");
    // }
  }
}
```