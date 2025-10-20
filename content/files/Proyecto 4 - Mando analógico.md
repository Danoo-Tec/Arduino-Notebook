En este proyecto usamos por primera vez un **potenciómetro** para generar una señal analógica que Arduino leerá por un pin **analógico** (A0). El cableado es: una patilla exterior del potenciómetro a **5V**, la otra exterior a **GND**, y la patilla central a **A0**. Así, al girar el pote, la tensión en A0 cambia suave entre 0 V y 5 V. Importante: **GND común** entre todo.

Luego vamos a usar 5 Leds todos ellas del mismo color por preferencia y que todas ellas utilicen la misma resistencia. 

Para leer el mando usamos `analogRead(A0)`, que devuelve un número **0–1023** proporcional a la tensión en A0 (0 → aprox. 0 V, 1023 → aprox. 5 V). La tensión la calculamos para comparar y así trabajar en función de ellos

En este primer código vamos a usar dos estructuras condicionales principales para que según vaya subiendo el valor del **potenciómetro** se vayan enciendo más luces e igual al contrario pero apagándolas. El código sería el siguiente:


```c
#define PIN_1 6
#define PIN_2 5
#define PIN_3 4
#define PIN_4 3
#define PIN_5 2
#define ANALOG_A0 A0

int lectura = 0;
float tension = 0.0;

void setup() {
  Serial.begin(9600);
  pinMode(PIN_1, OUTPUT);
  pinMode(PIN_2, OUTPUT);
  pinMode(PIN_3, OUTPUT);
  pinMode(PIN_4, OUTPUT);
  pinMode(PIN_5, OUTPUT);
  pinMode(ANALOG_A0, INPUT);
}

void loop() {
  delay(1);
  lectura = analogRead(A0);
  tension = (lectura * 0.5) / 1024.0;

  Serial.print("La lectura del analog read: " + String(lectura));
  Serial.println("\t\tLa tension será: " + String(tension));

  if (tension >= 0.10) {
    digitalWrite(PIN_1, HIGH);
    if (tension >= 0.20) {
      digitalWrite(PIN_2, HIGH);
      if (tension >= 0.30) {
        digitalWrite(PIN_3, HIGH);
        if (tension >= 0.40) {
          digitalWrite(PIN_4, HIGH);
          if (tension >= 0.45) {
            digitalWrite(PIN_5, HIGH);
          }
        }
      }
    }
  } 

  if (tension <= 0.45) {
    digitalWrite(PIN_5, LOW);
    if (tension <= 0.40) {
      digitalWrite(PIN_4, LOW);
      if (tension <= 0.30) {
        digitalWrite(PIN_3, LOW);
        if (tension <= 0.20) {
          digitalWrite(PIN_2, LOW);
          if (tension <= 0.10) {
            digitalWrite(PIN_1, LOW);
          }
        }
      }
    }
  } 
}
```

Para este segundo código ya no vamos a imprimir el valor de la tensión si no que vamos a imprimir solo el de la función ``analogRead(A0)`` pudiendo así apreciar el ruido que hay con la gráfica ya incluida en el IDE de Arduino. 

Luego la otra diferencia respecto del otro código es que ahora en vez de ir encendiendo o apagando las luces hasta que bien todas estén encendidas (**potenciómetro** a máxima potencia) o todas apagadas (**potenciómetro** a mínima potencia) vamos a ir encendiendo solo una y si la potencia sube enciendo la siguiente pero apago la anterior. El código es el siguiente: 

```c
#define PIN_1 6
#define PIN_2 5
#define PIN_3 4
#define PIN_4 3
#define PIN_5 2
#define ANALOG_A0 A0
int lectura = 0;
float tension = 0.0;

void setup() {
  Serial.begin(9600);
  pinMode(PIN_1, OUTPUT);
  pinMode(PIN_2, OUTPUT);
  pinMode(PIN_3, OUTPUT);
  pinMode(PIN_4, OUTPUT);
  pinMode(PIN_5, OUTPUT);
  pinMode(ANALOG_A0, INPUT);
}

void loop() {
  delay(1);
  lectura = analogRead(A0);
  tension = (lectura * 0.5) / 1024.0;
  
  Serial.println(lectura);  // Haciendo este print podemos apreciar el ruido por la gráfica

  if (tension > 0.0 && tension < 0.1) {
    digitalWrite(PIN_1, HIGH);
    digitalWrite(PIN_2, LOW);
    digitalWrite(PIN_3, LOW);
    digitalWrite(PIN_4, LOW);
    digitalWrite(PIN_5, LOW);
  } else if (tension > 0.1 && tension < 0.2) {
    digitalWrite(PIN_1, LOW);
    digitalWrite(PIN_2, HIGH);
    digitalWrite(PIN_3, LOW);
    digitalWrite(PIN_4, LOW);
    digitalWrite(PIN_5, LOW);
  } else if (tension > 0.2 && tension < 0.3) {
    digitalWrite(PIN_1, LOW);
    digitalWrite(PIN_2, LOW);
    digitalWrite(PIN_3, HIGH);
    digitalWrite(PIN_4, LOW);
    digitalWrite(PIN_5, LOW);
  } else if (tension > 0.3 && tension < 0.4) {
    digitalWrite(PIN_1, LOW);
    digitalWrite(PIN_2, LOW);
    digitalWrite(PIN_3, LOW);
    digitalWrite(PIN_4, HIGH);
    digitalWrite(PIN_5, LOW);
  } else if (tension > 0.4 && tension < 0.5) {
    digitalWrite(PIN_1, LOW);
    digitalWrite(PIN_2, LOW);
    digitalWrite(PIN_3, LOW);
    digitalWrite(PIN_4, LOW);
    digitalWrite(PIN_5, HIGH);
  }
}
```

