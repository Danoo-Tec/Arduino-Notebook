---
title: "Proyecto 2 - Control Mediante Puerto Serie"
date: 2025-10-02
draft: false
---
## 1. Invertir la lógica

En el anterior proyecto vimos como controlar el encendido de un **LED** por instrucciones desde la consola siendo **HIGH** encendido y **LOW** apagado, pero que pasa si quiero **invertir la lógica:**

Para invertir el encendido del LED (que LOW encienda y HIGH apague) usamos hundimiento de corriente (sinking). El cableado sería el siguiente:

+5 V → ánodo del LED → (LED) → cátodo → resistencia → pin 9. 

Así, cuando el pin 9 = LOW (≈0 V), la corriente fluye +5 V → LED → R → pin 9 → transistor interno → GND, hay diferencia de potencial y el LED enciende. Cuando el pin 9 = HIGH (≈5 V), ambos extremos del conjunto LED+R quedan casi al mismo potencial, no circula corriente y el LED apaga.

> El código en este caso sería el mismo que el anterior pero tendría sentido cambiar las letras '*e*' por '*a*' para que tenga más sentido

## 2. Dos brillos (normal y fuerte)

Bien, para lograr esto hay dos maneras, una de ellas es hacer uso de dos resistencias diferentes una por ejemplo de 220Ω y otra de 330Ω haciendo obviamente que cuando pase por la resistencia más grande frenará más y por tanto pasará menos potencia y por la resistencia más pequeña pasará más teniendo así más intensidad.

Pero, hay otra manera de hacerlo y es mi manera favorita, que utilizar exclusivamente software por lo que hace falta hacer uso de otra resistencia. Esto se puede lograr haciendo uso de la función **analogWrite(PIN, 0-255)**, siendo 0 el LED apagado y 255 el LED encendido a su máxima intensidad. 

> Algo muy importante para usar esta función es que si o si hay que utilizar los pines que tengan al lado la virgulilla (~), que en Arduino UNO son 3, 5, 6, 9, 10, 11.

Luego una vez esté eso funcionando puedes usar el siguiente código:

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
      // Se enciende con POCA intensidad
      case 'p': analogWrite(PIN, 128); break;
      // Se enciende a MÁXIMA intensidad
      case 'm': analogWrite(PIN, 255); break;
      // Se apaga el LED
      case 'a': digitalWrite(PIN, LOW); break;
      default: Serial.println("Opcion no valida");
    }
  }
}
```


## 3. Manejar 3 LEDs por consola (rojo, verde, azul)

Bueno, pues para manejar 3 **LEDs** por consola primero hay que montar el circuito que no es nada diferente a lo que teníamos antes lo único que ahora habrá que usar tres resistencias (una para cada led) y los tres leds. El código sería el siguiente:

```c
const int ROJO = 10;
const int VERDE = 9;
const int AZUL = 11;
char opcion = '\0';

void setup() {
  Serial.begin(9600);
  pinMode(ROJO, OUTPUT);
  pinMode(VERDE, OUTPUT);
  pinMode(AZUL, OUTPUT);

  Serial.println("Bienvenido al juego los tres LEDs");
  Serial.println("Podra controlar el encendido y apagado de cada uno de ellos");
  Serial.println("Minuscula enciende y Mayuscula apaga:");
  Serial.println("r = Rojo, v = Verde, a = Azul");
}

void loop() {
  if (Serial.available() > 0) {
    opcion = Serial.read();

    if (opcion == '\n') return;

    switch (opcion) {
      case 'r': digitalWrite(ROJO, HIGH); break;
      case 'R': digitalWrite(ROJO, LOW); break;
      case 'v': digitalWrite(VERDE, HIGH); break;
      case 'V': digitalWrite(VERDE, LOW); break;
      case 'b': digitalWrite(AZUL, HIGH); break;
      case 'B': digitalWrite(AZUL, LOW); break;
      default: Serial.println("Opcion no valida");
    }
  }
}
```
