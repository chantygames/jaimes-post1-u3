# Laboratorio: Exploración con DEBUG en DOSBox
**Arquitectura de Computadores — Unidad 3: Manejo del DEBUG**
Post-Contenido 1 | Ingeniería de Sistemas | 2026

---

## Descripción del Laboratorio

El presente laboratorio tiene como objetivo configurar el entorno DOSBox, acceder al depurador DEBUG de MS-DOS y utilizar los comandos R, D, F y U para inspeccionar registros del procesador, rellenar y volcar bloques de memoria, y desensamblar código máquina en memoria. Cada etapa queda documentada mediante capturas de pantalla que evidencian los resultados obtenidos.

---

## Parte A — Configuración del Entorno DOSBox

Se montó la carpeta de trabajo del sistema anfitrión como unidad C: virtual en DOSBox mediante el comando `MOUNT C C:\DOSWork`, seguido de la creación de la subcarpeta `LAB3POST1` con `MD LAB3POST1`. Todos los archivos generados durante el laboratorio se almacenaron dentro de esta subcarpeta. El DEBUG se inició desde el prompt `C:\LAB3POST1>` ejecutando simplemente `DEBUG`, lo que presentó el prompt `-` indicando que el depurador estaba listo.

**Comandos ejecutados:**
```
Z:\> MOUNT C C:\DOSWork
Z:\> C:
C:\> MD LAB3POST1
C:\> CD LAB3POST1
C:\LAB3POST1> DEBUG
-
```

---

## Parte B — Inspección de Registros (Comando R)

### Checkpoint 1 — Estado de registros

El comando `R` sin argumentos despliega el estado completo del procesador: los cuatro registros de propósito general (AX, BX, CX, DX), los registros de índice (SP, BP, SI, DI), los cuatro registros de segmento (DS, ES, SS, CS), el puntero de instrucción IP y las banderas de estado.

**Salida observada:**
```
-R
AX=0000  BX=0000  CX=0000  DX=0000  SP=FFFE  BP=0000  SI=0000  DI=0000
DS=1357  ES=1357  SS=1357  CS=1357  IP=0100   NV UP EI PL NZ NA PO NC
1357:0100 CD20          INT 20
```

**Observaciones:**
- AX, BX, CX y DX se inicializan en 0x0000 porque el DEBUG no ha ejecutado ninguna instrucción de carga.
- SP apunta a 0xFFFE, que representa el tope inicial de la pila del segmento actual.
- Los cuatro registros de segmento (DS, ES, SS, CS) apuntan al mismo párrafo 0x1357, correspondiente al PSP asignado por DOS al proceso.
- IP = 0x0100 indica que la siguiente instrucción a ejecutar es la primera dirección válida después del PSP (los primeros 256 bytes, 0x0000–0x00FF, pertenecen al PSP).

Para demostrar la modificación selectiva de registros, se cargó el valor 0x1234 en AX:
```
-R AX
AX 0000
:1234
```
La verificación posterior confirmó que solo AX cambió a 0x1234; el resto de registros permaneció intacto.

**Captura:** `capturas/CP1_registros.png`

---

## Parte C — Volcado de Memoria (Comandos D y F)

### Checkpoint 2 — Volcado hexadecimal anotado

**Relleno del bloque con F:**
```
-F 200 L40 AB CD EF
```
El comando `F` (Fill) inicializó 64 bytes (0x40) a partir de la dirección DS:0200 con el patrón cíclico AB CD EF. El DEBUG no emite confirmación; el regreso al prompt `-` indica ejecución sin errores.

**Volcado con D:**
```
-D 200 L40
1357:0200  AB CD EF AB CD EF AB CD-EF AB CD EF AB CD EF AB  ................
1357:0210  CD EF AB CD EF AB CD EF-AB CD EF AB CD EF AB CD  ................
1357:0220  EF AB CD EF AB CD EF AB-CD EF AB CD EF AB CD EF  ................
1357:0230  AB CD EF AB CD EF AB CD-EF AB CD EF AB CD EF AB  ................
```

**Descripción de las columnas del comando D:**

La salida del comando `D` se organiza en tres columnas. La primera columna muestra la dirección segmentada (segmento:offset) del primer byte de cada fila, lo que permite ubicar exactamente cada dato dentro del espacio de memoria de 1 MB del modo real. La segunda columna presenta los bytes de esa fila en formato hexadecimal, agrupados de a 8 bytes y separados por un guion en el centro, lo que facilita la identificación visual de patrones y la localización de bytes individuales por su posición. La tercera columna muestra la interpretación ASCII de esos mismos bytes: si un byte cae en el rango imprimible 0x20–0x7E se muestra el carácter correspondiente; de lo contrario se muestra un punto. En este volcado, los bytes 0xAB, 0xCD y 0xEF no son caracteres ASCII imprimibles, por lo que toda la columna derecha muestra puntos, evidenciando que la región contiene datos binarios, no texto.

**Captura:** `capturas/CP2_volcado_memoria.png`

---

## Parte D — Desensamblado (Comando U)

### Checkpoint 3 — Sesión de ensamblado y desensamblado

Se usó el comando `A` para escribir un programa de suma directamente en CS:0100:

```
-A 100
1357:0100 MOV AX, 0005
1357:0103 MOV BX, 0003
1357:0106 ADD AX, BX
1357:0108 INT 20
1357:010A
```

La verificación con `U` mostró la correspondencia exacta entre mnemónicos e instrucciones codificadas:

```
-U 100 109
1357:0100 B80500        MOV AX,0005
1357:0103 BB0300        MOV BX,0003
1357:0106 03C3          ADD AX,BX
1357:0108 CD20          INT 20
```

**Análisis de la codificación:**
- `MOV AX, 0005` → bytes `B8 05 00`: el opcode B8 indica MOV al registro AX con un inmediato de 16 bits; el valor 0x0005 se almacena en little-endian como 05 00.
- `MOV BX, 0003` → bytes `BB 03 00`: opcode BB para MOV BX con inmediato; el valor 0x0003 se almacena como 03 00.
- `ADD AX, BX` → bytes `03 C3`: opcode 03 indica ADD r16, r/m16; el byte ModRM C3 codifica los operandos AX y BX.
- `INT 20` → bytes `CD 20`: opcode CD indica INT con número de interrupción inmediato; 20 (hexadecimal) es la interrupción de terminación de programas DOS.

El programa completo ocupa 10 bytes en memoria.

**Captura:** `capturas/CP3_ensamblado_desensamblado.png`

---

## Conclusiones

La realización de este laboratorio permitió verificar de manera concreta que el DEBUG expone la arquitectura del procesador 8086 en su nivel más fundamental. La inspección de registros con `R` demostró el estado inicial del PSP y la segmentación de memoria; el uso de `F` y `D` evidenció cómo los datos se organizan en bloques de bytes sin ninguna abstracción de tipo; y el ciclo `A`→`U` mostró la correspondencia directa entre instrucciones ensamblador y su representación en código máquina, incluyendo el convenio little-endian para valores de 16 bits. Estos conceptos subyacen a toda la arquitectura x86 moderna y constituyen la base sobre la que se construyen los niveles superiores de abstracción del software.

---

## Referencias

- Irvine, K. R. (2019). *Assembly Language for x86 Processors* (8th ed.). Pearson.
- Patterson, D. A., & Hennessy, J. L. (2020). *Computer Organization and Design: The Hardware/Software Interface* (RISC-V ed.). Morgan Kaufmann.
