---
title: "Proyecto 0 - Usando el puerto serie"
date: 2025-09-18
draft: false
---

```c
void setup() {
  Serial.begin(9600);               // Abrimos el puerto serie a 9600 Baudios (Habilitamos el UART)
  Serial.println("Bienvenido!!");   // Imprimimos un mensaje de bienvenida (Solo se imprime una vez)
}

void loop() {
  Serial.println("Hola Mundo!!");  // La imprimimos con un salto de línea
  delay(500);                      // Esperamos medio segundo entre iteración
}
```


