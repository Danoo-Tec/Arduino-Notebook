---
title: "Funciones Básicas de Arduino"
date: 2025-09-11
draft: false
---
## Variables Eléctricas

**Tensión (V):** diferencia de potencial entre dos puntos. Siempre se mide respecto a **tierra (GND)** en electrónica.
**Corriente (I):** flujo de carga por un conductor debido a una diferencia de potencial.
**Resistencia (R):** oposición al paso de corriente; relaciona V e I. Si entra la frecuencia, hablamos de **impedancia**.
**Potencia (P):** lo que se “calienta/consume”: **P=V\*I**

> La **tensión “empuja”**, la **corriente “circula”**, la **resistencia “frena”**, y la **potencia “calienta”**.

**Ley de Ohm**: V = I\*R

**Leyes de Kirchhoff**:
- **Corrientes (KCL):** lo que entra a un nodo = lo que sale. Suma de corrientes entrantes = suma de salientes.
- **Tensiones (KVL):** en cualquier lazo cerrado, subidas de tensión = bajadas de tensión.

**Componentes analógicos básicos**:
- **Pulsador:** por defecto abierto; solo conduce al pulsarlo. Útil para entradas digitales.
- **Resistencias:** caen tensión y limitan corriente; código de colores/valor SMD.
- **Condensadores:** almacenan energía; suavizan alimentación de sensores/motores (desacoplo).
- **Bobinas (inductores):** se oponen a cambios bruscos de corriente; ojo con sobretensiones.
- **Diodos:** conducen en un sentido (ánodo→cátodo). El cátodo suele llevar franja.
- **LEDs:** diodos que emiten luz; **siempre** con resistencia en serie. Ánodo = pata larga.
- **LED RGB:** cuatro patillas; mezclan colores variando tensiones/corrientes por canal.
- **Transistor BJT (como interruptor):** emisor a GND, colector al dispositivo, base a la señal de control (Arduino) con resistencia; al excitarlo, “cierra” el circuito y el actuador recibe energía.

> **IMPORTANTE**: **Todo loop empieza y termina en GND.** Si no ves el camino de vuelta, el circuito no funciona. (KVL/KCL)

**Qué es un puerto serie y qué pines usa**:
- **Puerto = interfaz de comunicación** (física o virtual) entre dispositivos.
- En **serie** la info viaja **bit a bit** por un único canal; en **paralelo**, varios bits a la vez por múltiples conductores (más difícil de sincronizar en cable).
- Mínimo necesitas **TX** (transmite), **RX** (recibe) **y GND común**. Sin tierra compartida, no hay comunicación fiable.

**Terminología clave:**
- **UART**: hardware del micro que convierte datos en **secuencia de bits** y gestiona TX/RX a una **velocidad** (baud).
- **TTL**: niveles lógicos típicos 0 V ↔ **Vcc** (3.3 V/5 V).

> El IDE de Arduino suele esperar **ASCII “básico” (32–126 imprimibles)** cuando usas el monitor serie. Si le mandas otra cosa, puede mostrar “basura”.

**Arduino: hardware de serie en la placa**
- **Arduino UNO**: un puerto serie en **D0 (RX)** y **D1 (TX)**, **compartido con USB**.
- **Advertencia**: si usas el puerto serie para el PC, **no uses D0/D1 como E/S normales** a la vez: se interfieren con la comunicación y la carga de programas.

#### Funciones en Arduino

```c
Serial.begin(valocidad);          --> "configura la UART (p. ej., 9600, 115200)."
Serial.print(dato, formato);      --> "Imprime texto por pantalla"
Serial.println(dato, formato);    --> "Imprime texto por pantalla y añade un salto de línea"
Serial.write(dato);               --> "bytes crudos. Por ejemplo si pones un números te imprime su valor en ASCII"
Serial.available();               --> "¿hay datos recibidos?"
Serial.read();                    --> "Lee un byte por consola como por ejemplo un char"
```

> **Serial.begin(9600);** se debe definir dentro de la función *setup()* 

#### Variables en Arduino

```c
int entero: 7;
float flotante: 3.5;
String = "Hola Mundo!";
boolean si_no: True;
```

#### Conceptos básicos para trabajar con la protoboard - (Leds, Pulsadores...)
- **Barra roja** = +5 V; **barra azul** = GND. Son **buses** para repartir alimentación.
- **Filas de 5** en el centro, **cada fila** es **un mismo nodo eléctrico**. La resistencia siempre debe ir **entre dos filas distintas**.
- **LED**: **ánodo (largo) al + (La fila donde está la resistencia)**, **cátodo (corto) al − (Con un cable a tierra)**.
	- El **LED** es un **diodo** por lo que solo deja pasar la corriente en un sentido, por eso puede no funcionar dependiendo de como se haya puesto

> **Nunca pongas LEDs directamente en paralelo compartiendo una única resistencia; cada LED debe tener su propia resistencia**, porque cada uno tiene su **tensión directa (Vf)** y el de **Vf más baja** se encenderá antes, **robará más corriente** y, por tolerancias y calentamiento, puede llegar a **quemarse**

> Un pulsador de 4 patas tiene **dos pares unidos internamente** (1–2 y 3–4) y, al pulsar, **conecta ambos pares**. En la protoboard, cada **fila de 5 agujeros** es el **mismo nodo**, y la **grieta central** separa eléctricamente ambos lados. Si colocas el pulsador **atravesando la grieta**, garantizas que cada par caiga en **lados distintos**, de modo que al pulsar **unes dos nodos realmente separados** (actúa como interruptor de verdad). Esto evita errores típicos de poner dos patas del **mismo par o de la misma fila** (LED siempre encendido o nunca encendido) y es la **buena práctica** recomendada.

#### Conceptos pinMode y digitalWrite
- **Por defecto**: pines en `INPUT` (alta impedancia).
- **`OUTPUT`**: el pin **fija** 0 V/5 V → ideal para LED/actuadores.
- **`INPUT`**: **no fija** nivel; usa **pull-up/down** externo.
- **`INPUT_PULLUP`**: pull-up **interna** a 5 V; **botón a GND**; lógica invertida (pulsado=LOW).
- `digitalWrite(HIGH)` en **`INPUT`** solo **activa la pull-up** → puede encender un LED **débilmente** (no es un “HIGH real” de salida).