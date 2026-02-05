---
title: Proyecto 11 - Pantalla LCD
date: 2025-12-04
draft: false
---

## Pantalla LCD: Montaje

En la clase de hoy hemos visto por primera vez la pantalla LCD, la cual contiene 2 filas y 16 columnas. Esta tiene muchas opciones en cuanto a cómo manejarla en software con una librería llamada `LiquidCrystal.h`; sin embargo, primero hay que montarla y hacerlo bien. El montaje de la placa es el siguiente:

* **VSS** -> GND
* **VCC** -> 5V
* **V0** -> Pin central del potenciómetro (controlamos el contraste)
* **RS** -> Pin digital (Register Selection)
* **R/W** -> GND
* **E** -> Pin digital
* **D4, D5, D6, D7** -> Pines digitales
* **A (+)** -> 5V (Ánodo) (con resistencia de 220 Ω)
* **K (-)** -> GND (Cátodo)

### Tips y consideraciones prácticas

* La pantalla LCD puede trabajar en **modo de 8 bits o de 4 bits**. Normalmente se utiliza el modo de 4 bits porque permite ahorrar pines del Arduino sin perder funcionalidad.
* El **potenciómetro es fundamental** para ajustar el contraste. Si está mal ajustado, la pantalla puede parecer apagada o mostrar solo bloques negros.
* La **resistencia de 220 Ω** en el pin de la retroiluminación protege el LED interno del LCD. Sin esta resistencia existe riesgo de dañar la pantalla.
* Es obligatorio inicializar la pantalla con `lcd.begin(columnas, filas);` antes de usar cualquier método de la librería.
* No es recomendable usar `lcd.clear()` continuamente dentro del `loop()` porque puede producir parpadeos en la pantalla. Es mejor sobrescribir el contenido usando `lcd.setCursor()`.

Una vez tengamos todos estos pines conectados a la pantalla LCD, ya podemos configurar el código para empezar a mostrar información por la pantalla. Lo primero que debemos hacer es incluir la librería `LiquidCrystal.h`. Una vez incluida, podemos crear nuestra pantalla en el código con la siguiente instrucción:

```cpp
LiquidCrystal lcd(PIN_RS, PIN_ENABLE, PIN_D4, PIN_D5, PIN_D6, PIN_D7);
```

Un proyecto que se podría hacer ahora que ya sabemos manejar esta pantalla LCD es mejorar el proyecto anterior del reloj de arena para mostrar, en bloques o caracteres personalizados, por ejemplo el tiempo restante.
Yo, en mi caso, no he hecho un proyecto como tal, pero sí la creación de dos caracteres: la 'ñ' y la 'Ñ'. Estos caracteres no están por defecto, ya que el LCD usa un set de caracteres propio compatible con el controlador HD44780, pero permite crear hasta 8 caracteres personalizados si queremos.

```cpp
#include <LiquidCrystal.h>

// PIN de RS (Register Selection), selector de registro (para enviar datos o comandos)
#define PIN_RS 8

// PIN Enable para indicar al LCD cuándo debe leer lo que hay en las líneas de datos
#define PIN_ENABLE 9

// Pines D4-D7
#define PIN_D4 13
#define PIN_D5 12
#define PIN_D6 11
#define PIN_D7 10

LiquidCrystal lcd(PIN_RS, PIN_ENABLE, PIN_D4, PIN_D5, PIN_D6, PIN_D7);

// --------- Creación de mis letras --------- //
// ñ minúscula:
const byte minus_spain_n = 0;
const byte mat_minus_spain_n[8] = {
  B01101,
  B10010,
  B00000,
  B10110,
  B11001,
  B10001,
  B10001,
  B00000
};

// Ñ mayúscula:
const byte mayus_spain_n = 1;
const byte mat_mayus_spain_n[8] = {
  B01101,
  B10010,
  B10001,
  B11001,
  B10101,
  B10011,
  B10001,
  B00000
};

void setup() {
  Serial.begin(9600);
  
  lcd.begin(16, 2);

  // Creamos los caracteres para poder usarlos
  lcd.createChar(minus_spain_n, mat_minus_spain_n);
  lcd.createChar(mayus_spain_n, mat_mayus_spain_n);

  // Limpiamos la pantalla y mostramos un mensaje
  // Utilizando el carácter custom
  lcd.clear();
  lcd.print("VIVA ESPA");
  lcd.write(mayus_spain_n);
  lcd.print("A");
}

void loop() {}
```

## Funciones / Objetos

Por aquí dejo una pequeña tabla con las diferentes funciones o métodos que tenemos disponibles utilizando esta librería:

| Función / Método                 | Descripción                                                                                               |
| -------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `lcd.begin(columnas, filas)`     | Inicializa la comunicación con la pantalla LCD indicando el número de columnas y filas.                   |
| `lcd.print(data)`                | Escribe datos en formato ASCII en la pantalla, empezando desde la posición actual del cursor.             |
| `lcd.write(byte)`                | Escribe un carácter directamente mediante su valor en byte (útil para caracteres personalizados).         |
| `lcd.clear()`                    | Borra todo el contenido de la pantalla y coloca el cursor en la posición inicial (0,0).                   |
| `lcd.setCursor(col, fila)`       | Coloca el cursor en la columna y fila indicadas (empieza en 0,0).                                         |
| `lcd.createChar(indice, matriz)` | Permite crear un carácter personalizado. Se pueden crear hasta 8 caracteres (índices del 0 al 7).         |
| `lcd.autoscroll()`               | Activa el desplazamiento automático del contenido hacia la izquierda cuando se escribe un nuevo carácter. |
| `lcd.noAutoscroll()`             | Desactiva el desplazamiento automático.                                                                   |
| `lcd.scrollDisplayLeft()`        | Desplaza todo el contenido de la pantalla una posición hacia la izquierda.                                |
| `lcd.scrollDisplayRight()`       | Desplaza todo el contenido de la pantalla una posición hacia la derecha.                                  |
