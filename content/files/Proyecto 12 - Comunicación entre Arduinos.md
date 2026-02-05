---
title: Proyecto 12 - Comunicación entre Arduinos
date: 2026-01-29
draft: false
---

## Trabajando con dos Arduinos

En la clase de hoy hemos visto cómo establecer comunicaciones entre dos placas de Arduino mediante el empleo del puerto serie.
El proyecto funciona de la siguiente manera: hay dos placas, una de ellas será la placa maestra mientras que la otra será la esclava.
La placa **maestra** poseerá 5 botones que emitirán comunicaciones codificadas en Morse.

* 1 (p) → Punto
* 2 (r) → Raya
* 3 (c) → Conversión a carácter
* 4 (b) → Borrar último código introducido
* 5 (e) → Fin de transmisión

Esos caracteres al lado de los números son aquellos que se emitirán por el puerto serie.
Por otro lado, tenemos la placa **esclava**, que será la que tiene configurada la pantalla LCD y la que recibirá los diferentes caracteres mandados por los botones de la **maestra**.

Muchas de las funciones de este proyecto ya las tenemos incluidas en el proyecto. Entre ellas, todo el código de la maestra está hecho.
Durante el proyecto nos vamos a centrar en que el código recibido por los dos primeros botones (punto y raya) se escriba en la fila inferior y los caracteres convertidos en la superior. Por último, con el botón 5 borraremos todo.

> Para comunicar dos placas, en este caso vamos a tener que conectar el pin **TX** (de transmisión) de la placa **maestra**, que es la que transmite los caracteres según los botones que pulsemos, al pin **RX** de la **esclava**, que es quien los recibe y trabaja con ellos.

> Además, para que se puedan comunicar ambas deben tener una referencia de tierra común. Esto lo podemos lograr conectando la tierra (GND) de una placa a la tierra de la otra (fila negra en la protoboard).

A continuación dejo la funcionalidad o código que he agregado a la placa esclava para que esto funcione:

```cpp
// Variables
char recibido = 0;
char letra = 0;
String mensaje = "";   // letras ya convertidas (fila superior)
String simbolo = "";   // puntos y rayas del carácter actual (fila inferior)

// Se crea el objeto pantalla con seis pines de control y datos
LiquidCrystal lcd(pin_RS, pin_E, pin_DB4, pin_DB5, pin_DB6, pin_DB7);

void setup() {
  Serial.begin(9600);

  lcd.begin(16, 2);
  lcd.clear();

  // Arranco con ambas líneas vacías
  lcd.setCursor(0, 0);
  lcd.print("");
  lcd.setCursor(0, 1);
  lcd.print("");
}

void loop() {
  if (Serial.available() > 0) {
    recibido = Serial.read();

    switch (recibido) {
      case 'p':   // Un punto
        simbolo += ".";
        lcd.setCursor(0, 1);
        lcd.print("                ");
        lcd.setCursor(0, 1);
        lcd.print(simbolo);
        break;

      case 'r':  // Una raya
        simbolo += "-";
        lcd.setCursor(0, 1);
        lcd.print("                ");
        lcd.setCursor(0, 1);
        lcd.print(simbolo);
        break;
      
      case 'c':  // Convierte el símbolo en letra
        letra = conversion_letra(simbolo);
        mensaje += letra;
        simbolo = "";

        // Actualizar fila superior (mensaje)
        lcd.setCursor(0, 0);
        lcd.print("                ");
        lcd.setCursor(0, 0);
        lcd.print(mensaje);

        // Actualizar fila inferior (símbolo vacío)
        lcd.setCursor(0, 1);
        lcd.print("                ");
        break;

      case 'b': // Se borra el último punto o raya recibido usando quitarcaracter()
        simbolo = quitarcaracter(simbolo);

        // Actualizar fila inferior (símbolo)
        lcd.setCursor(0, 1);
        lcd.print("                ");  // limpia fila inferior (16 espacios)
        lcd.setCursor(0, 1);
        lcd.print(simbolo);
        break;

      case 'e': // Fin del mensaje: borrado de toda la pantalla
        simbolo = "";
        mensaje = "";

        // Borrar pantalla y dejarla limpia
        lcd.clear();

        // Asegurar ambas líneas vacías
        lcd.setCursor(0, 0);
        lcd.print("                ");
        lcd.setCursor(0, 1);
        lcd.print("                ");
        break;
    }
  }
}
```

Con el código anterior todo ya debería funcionar, es decir, deberías ser capaz de mandar **puntos** o **rayas** con los dos primeros botones y luego convertir eso a un carácter con el tercer botón. Además, si has escrito una raya o punto y quieres borrar, puedes usar el cuarto botón y, finalmente, para borrar toda la pantalla puedes usar el quinto y último botón.

---

## Mejora con dos LEDs para verificación

Una mejora que podemos implementar es que cuando yo pulse el tercer botón (con el cual transmito la letra 'c') y se convierta al carácter, se transmita por ejemplo el número 1 para **SUCCESS** o el número 0 para **ERROR** desde la placa esclava a la maestra, y que en esta última se encienda o bien un LED verde o uno rojo dependiendo del estado recibido.

Esto se hará para cada carácter convertido y el 1 o 0 lo mandaremos con dos botones montados en la placa esclava.
Habrá que montar también los dos LEDs en la placa maestra.

A continuación dejo el código necesario para implementar esta funcionalidad.

### Cambios en el Arduino esclavo

```cpp
#define pin_OK   8   // Botón SUCCESS -> manda '1'
#define pin_ERR  9   // Botón ERROR   -> manda '0'

bool esperandoEstado = false;  // estamos esperando pulsar OK/ERR tras convertir
```

En el `setup()` después de `Serial.begin(9600);`

```cpp
pinMode(pin_OK, INPUT_PULLUP);
pinMode(pin_ERR, INPUT_PULLUP);
```

En el `loop()` al final del `case 'c'`

```cpp
esperandoEstado = true;   // ahora toca responder 1/0 con los botones
```

Por último, al final del `loop()`

```cpp
// Si se ha convertido un carácter, enviar estado a la maestra con botones 8/9
if (esperandoEstado) {
  if (!digitalRead(pin_OK)) {        // pulsado (INPUT_PULLUP)
    Serial.write('1');               // SUCCESS
    esperandoEstado = false;
    delay(150);                      // antirrebote simple
  } else if (!digitalRead(pin_ERR)) {
    Serial.write('0');               // ERROR
    esperandoEstado = false;
    delay(150);
  }
}
```

---

### Cambios en el Arduino maestro

```cpp
#define led_verde 8
#define led_rojo  9
```

En el `setup()`

```cpp
pinMode(led_verde, OUTPUT);
pinMode(led_rojo, OUTPUT);
digitalWrite(led_verde, LOW);
digitalWrite(led_rojo, LOW);
```

Por último, al final del `loop()` añadimos lo siguiente:

```cpp
// Leer respuesta de la esclava: '1' SUCCESS, '0' ERROR
if (Serial.available() > 0) {
  char resp = Serial.read();
  if (resp == '1') {
    digitalWrite(led_verde, HIGH);
    digitalWrite(led_rojo, LOW);
  } else if (resp == '0') {
    digitalWrite(led_verde, LOW);
    digitalWrite(led_rojo, HIGH);
  }
}
```

> Importante: Nada de esto va a funcionar si no conectamos otro cable, en este caso desde el **TX de la esclava** al **RX de la maestra** (Arduino esclavo → Arduino maestro).
