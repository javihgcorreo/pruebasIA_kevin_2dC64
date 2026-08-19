Perfecto. Vamos a hacer **la Fase 1: un prototipo jugable en BASIC V2**, pero intentando que desde el principio la estructura sea compatible con el futuro motor 6502.

La primera versión tendrá:

* 🧍 Jugador como sprite.
* ← → Movimiento horizontal.
* 🪂 Gravedad.
* ⬆ Salto.
* 🧱 Suelo.
* 🧱 Varias plataformas.
* 💥 Colisiones.
* 🔄 Bucle de juego.
* Variables separadas para que después podamos modificar fácilmente física y comportamiento.

**No metería todavía scroll ni enemigos.** Primero necesitamos que la física sea sólida.

### Primer objetivo

En el emulador deberíamos acabar viendo algo conceptualmente así:

```text
              ┌───────┐
              │       │
              └───────┘
                     🧍
          ──────────────────

      ───────────────────────────
```

El jugador podrá desplazarse y saltar entre plataformas.

### Pero hay una decisión técnica importante

Para **BASIC V2**, tenemos dos posibilidades:

1. **Prototipo rápido con caracteres PETSCII**
   Más sencillo y permite centrarnos en física y colisiones.

2. **Prototipo directamente con sprites VIC-II**
   Es algo más complejo, pero merece más la pena porque nuestro objetivo final es un juego con sprites.

Yo elegiría la **opción 2**. Así desde el primer prototipo estaremos trabajando con la arquitectura real del juego.

La siguiente versión de `basic/prototype.bas` debería encargarse de:

```text
INIT
 │
 ├── Inicializar VIC-II
 ├── Crear sprite
 ├── Configurar color
 ├── Configurar posición
 └── Crear plataformas
       │
       ▼
 GAME LOOP
 │
 ├── Leer teclado/joystick
 ├── Actualizar jugador
 ├── Aplicar gravedad
 ├── Detectar colisiones
 ├── Actualizar sprite
 └── Repetir
```

Y las variables principales:

```text
PX   posición X
PY   posición Y
VX   velocidad X
VY   velocidad Y

GRAVITY
JUMP
SPEED

ON_GROUND
```

**Vamos a hacerlo directamente con sprite VIC-II y teclado**, y después probamos el código línea por línea en el C64 Online Emulator.


Sí. Vamos a crear una **primera versión jugable y sencilla** de `basic/prototype.bas`.

Como estamos en BASIC V2, voy a priorizar que sea **fácil de entender y modificar**. El jugador será un sprite del VIC-II, y las plataformas serán bloques de caracteres. Esto nos permitirá probar primero la física antes de complicarlo con gráficos.

```basic
10 REM ==========================================
20 REM C64 PLATFORM ENGINE - PROTOTYPE
30 REM FASE 1: MOVIMIENTO + GRAVEDAD + SALTO
40 REM ==========================================
50 :
60 REM --- CONSTANTES VIC-II ---
70 VIC=53248
80 SPRITE0=VIC+21
90 SPRITEPTR=2040
100 SPRITEDATA=12288
110 :
120 REM --- VARIABLES DEL JUGADOR ---
130 PX=120
140 PY=180
150 VX=0
160 VY=0
170 :
180 REM --- FISICA ---
190 GRAVITY=1
200 JUMP=-9
210 SPEED=3
220 MAXFALL=8
230 ONGROUND=0
240 :
250 REM --- INICIALIZACION ---
260 GOSUB 1000
270 :
280 REM --- BUCLE PRINCIPAL ---
290 GOSUB 2000
300 GOSUB 3000
310 GOSUB 4000
320 GOSUB 5000
330 GOTO 290
340 :
350 REM ==========================================
360 REM FIN
370 REM ==========================================
999 END
1000 REM ==========================================
1010 REM INICIALIZAR VIC-II
1020 REM ==========================================
1030 POKE VIC+17,0
1040 POKE VIC+22,16
1050 :
1060 REM ACTIVAR SPRITE 0
1070 POKE SPRITE0,1
1080 :
1090 REM COLOR SPRITE
1100 POKE VIC+39,1
1110 :
1120 REM POSICION INICIAL
1130 POKE VIC,PX
1140 POKE VIC+1,PY
1150 :
1160 REM CREAR DATOS DEL SPRITE
1170 GOSUB 1500
1180 :
1190 REM BORRAR PANTALLA
1200 PRINT CHR$(147);
1210 :
1220 REM CREAR PLATAFORMAS
1230 GOSUB 6000
1240 RETURN
1250 :
1500 REM ==========================================
1510 REM CREAR SPRITE 0
1520 REM ==========================================
1530 REM SPRITE EN 12288
1540 FOR I=0 TO 62
1550 POKE SPRITEDATA+I,0
1560 NEXT I
1570 :
1580 REM CABEZA
1590 POKE SPRITEDATA+2,24
1600 POKE SPRITEDATA+3,60
1610 POKE SPRITEDATA+4,126
1620 :
1630 REM CUERPO
1640 POKE SPRITEDATA+20,126
1650 POKE SPRITEDATA+21,255
1660 POKE SPRITEDATA+22,255
1670 POKE SPRITEDATA+23,126
1680 :
1690 REM BRAZOS
1700 POKE SPRITEDATA+35,60
1710 POKE SPRITEDATA+36,126
1720 POKE SPRITEDATA+37,60
1730 :
1740 REM PIERNAS
1750 POKE SPRITEDATA+50,60
1760 POKE SPRITEDATA+51,60
1770 POKE SPRITEDATA+52,102
1780 POKE SPRITEDATA+53,102
1790 :
1800 REM PUNTERO DEL SPRITE
1810 POKE SPRITEPTR,SPRITEDATA/64
1820 RETURN
1830 :
2000 REM ==========================================
2010 REM INPUT
2020 REM ==========================================
2030 GET K$
2040 :
2050 IF K$="A" THEN VX=-SPEED
2060 IF K$="D" THEN VX=SPEED
2070 IF K$=" " AND ONGROUND=1 THEN VY=JUMP
2080 :
2090 REM FRENAR
2100 IF K$="" THEN VX=0
2110 RETURN
2120 :
3000 REM ==========================================
3010 REM FISICA
3020 REM ==========================================
3030 VY=VY+GRAVITY
3040 IF VY>MAXFALL THEN VY=MAXFALL
3050 :
3060 PX=PX+VX
3070 PY=PY+VY
3080 :
3090 REM LIMITES LATERALES
3100 IF PX<24 THEN PX=24
3110 IF PX>296 THEN PX=296
3120 :
3130 ONGROUND=0
3140 :
3150 REM SUELO
3160 IF PY>=210 THEN PY=210:VY=0:ONGROUND=1
3170 RETURN
3180 :
4000 REM ==========================================
4010 REM COLISIONES
4020 REM ==========================================
4030 REM PLATAFORMA 1
4040 IF PX>70 AND PX<150 THEN GOSUB 4500
4050 :
4060 REM PLATAFORMA 2
4070 IF PX>180 AND PX<260 THEN GOSUB 4600
4080 RETURN
4090 :
4500 REM PLATAFORMA 1
4510 IF PY>=160 AND PY<=170 AND VY>=0 THEN PY=160:VY=0:ONGROUND=1
4520 RETURN
4530 :
4600 REM PLATAFORMA 2
4610 IF PY>=120 AND PY<=130 AND VY>=0 THEN PY=120:VY=0:ONGROUND=1
4620 RETURN
4630 :
5000 REM ==========================================
5010 REM ACTUALIZAR SPRITE
5020 REM ==========================================
5030 POKE VIC,PX
5040 POKE VIC+1,PY
5050 RETURN
5060 :
6000 REM ==========================================
6010 REM DIBUJAR PLATAFORMAS
6020 REM ==========================================
6030 REM SUELO
6040 FOR X=0 TO 39
6050 POKE 1024+21*40+X,160
6060 POKE 55296+21*40+X,5
6070 NEXT X
6080 :
6090 REM PLATAFORMA 1
6100 FOR X=10 TO 18
6110 POKE 1024+17*40+X,160
6120 POKE 55296+17*40+X,5
6130 NEXT X
6140 :
6150 REM PLATAFORMA 2
6160 FOR X=23 TO 31
6170 POKE 1024+13*40+X,160
6180 POKE 55296+13*40+X,5
6190 NEXT X
6200 RETURN
```

### ⚠️ Una corrección importante antes de ejecutarlo

He usado `A` y `D` para izquierda/derecha y **espacio para saltar**. Esto es deliberado para que podamos probarlo cómodamente con teclado.

Pero hay un detalle importante del C64: **la posición Y de los sprites y las coordenadas de las plataformas están en sistemas de coordenadas diferentes**. Por eso esta primera versión es un prototipo de física y tendremos que ajustar las cajas de colisión.

### Cómo probarlo

En el emulador:

1. Abre el C64.
2. Escribe:

```text
NEW
```

3. Introduce el programa.
4. Ejecuta:

```text
RUN
```

Si el editor del emulador permite pegar directamente varias líneas, puedes pegar el bloque completo.

### Controles

| Tecla     | Acción    |
| --------- | --------- |
| `A`       | Izquierda |
| `D`       | Derecha   |
| `ESPACIO` | Saltar    |

### Lo que quiero que comprobemos primero

Antes de añadir nada más, quiero que consigamos que funcione este ciclo:

```text
              🧍
              ↓
            GRAVEDAD
              ↓
       ───────────────
            SUELO

A ←  🧍  → D

          ESPACIO
             ↑
             🧍
            ╱
           ╱
          ╱
```

**Una vez comprobado, el siguiente paso será separar las plataformas de `prototype.bas` y crear un auténtico `LEVEL`**, de manera que podamos cargar mapas como `level01.dat`, `level02.dat`, etc. Ahí empezaremos a convertirlo realmente en un motor de plataformas reutilizable.

