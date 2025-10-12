---
title: "Proyecto 3 - Interfaz de Nave Espacial"
date: 2025-10-09
draft: false
---

Bien para este proyecto lo primero que vamos a tener que hacer es decidir si para la lectura del estado de un botón vamos a querer usar un **Pull-Down** **externo** o un **Pull-UP interno**. Las principales diferencias es que con el externo utilizas más hardware, y con el interno haces uso de la resistencia interna del pin a +5V, en el que se invierte la lógica, es decir, cuando pulsas estas en LOW y cuando no estas en HIGH.

Una vez sabemos esto es importante saber como montar ambos sistemas:
- **INPUT_PULLUP ⇒ Botón a GND** (lógica invertida: pulsado=LOW).
- **INPUT + 10 kΩ a GND ⇒ Botón a +5 V** (pulsado=HIGH).
En ambos casos, el botón **no va en el circuito del LED**: solo entrega una **señal lógica**; el **pin de salida** es quien alimenta el LED.

Luego lo demás es bastante parecido a la conexión de tres LEDs del proyecto anterior con la diferencia de que vamos  a hacer uso de la instrucción **delay** para encender y apagar los LEDs durante *x* tiempo.

```c
const int VERDE = 9;
const int ROJO_1 = 10;
const int ROJO_2 = 11;
const int BTN = 6;
char opcion = '\0';
int encendido = 1;

void setup() {
  Serial.begin(9600);
  pinMode(VERDE, OUTPUT);
  pinMode(ROJO_1, OUTPUT);
  pinMode(ROJO_2, OUTPUT);

  pinMode(BTN, INPUT_PULLUP);  // Suelto = High, Pulsado = Low

  // Estado inicial: verde ON, rojos OFF
  digitalWrite(VERDE, HIGH);
  digitalWrite(ROJO_1, LOW);
  digitalWrite(ROJO_2, LOW);

  Serial.println("#######################################################################");
  Serial.println("Bienvenido al programa");
  Serial.println("-- NAVE ESPACIAL --");
  Serial.println("Si no haces nada, si mantiene solo el LED verde");
  Serial.println("Si mantienes el botón pulsado parpadean los rojos y el verde se apaga");
  Serial.println("Pulsando la tecla 'o' pones todo en OFF");
  Serial.println("Pulsando la tecla 'e' pones todo en ON");
  Serial.println("#######################################################################");
}

void loop() {
  bool pulsado = (digitalRead(BTN) == LOW);

  if (Serial.available() > 0) {
    opcion = Serial.read();
    
    if (opcion == '\n' || opcion == '\r') return;
    
    if (opcion == 'o') {
      digitalWrite(VERDE, LOW);
      digitalWrite(ROJO_1, LOW);
      digitalWrite(ROJO_2, LOW);
      encendido = 0;
    } else if (opcion == 'e') {
      encendido = 1;
    }
  }

  if (encendido) {
    if (!pulsado) {
      digitalWrite(VERDE, HIGH);
      digitalWrite(ROJO_1, LOW);
      digitalWrite(ROJO_2, LOW);
    } else {
      digitalWrite(VERDE, LOW);
      digitalWrite(ROJO_1, HIGH);
      delay(100);
      digitalWrite(ROJO_1, LOW);
      digitalWrite(ROJO_2, HIGH);
      delay(100);
      digitalWrite(ROJO_1, HIGH);
      digitalWrite(ROJO_2, LOW);
    }
  }
}
```