# C64 Platform Engine

Motor de juego de plataformas para **Commodore 64 (C64)**, diseñado para crear juegos con desplazamiento horizontal, gravedad, plataformas, enemigos y sprites personalizables.

El proyecto parte de un prototipo desarrollado y probado en un emulador de C64 y está pensado para evolucionar progresivamente desde **BASIC V2** hacia **6502 Assembly**, utilizando el hardware del C64 para conseguir un rendimiento adecuado.

## 🎮 Características previstas

- [ ] Movimiento del jugador izquierda/derecha
- [ ] Gravedad
- [ ] Salto
- [ ] Colisiones con plataformas
- [ ] Sprites del jugador
- [ ] Animación de sprites
- [ ] Enemigos configurables
- [ ] Diferentes tipos de enemigos
- [ ] Scroll horizontal
- [ ] Sistema de cámara
- [ ] Múltiples pantallas/niveles
- [ ] Sistema de vidas
- [ ] Objetos y coleccionables
- [ ] Pantallas de inicio y fin
- [ ] Sonido y música
- [ ] Editor de niveles
- [ ] Sistema de configuración de personajes y enemigos

---

## 🕹️ Objetivo

El objetivo es crear una pequeña **engine de plataformas reutilizable para C64**.

La idea es separar el motor de los datos del juego para que sea posible crear diferentes juegos sin tener que modificar continuamente la lógica principal.

Por ejemplo:

```text
ENGINE
│
├── INPUT
├── PLAYER
├── PHYSICS
├── COLLISION
├── CAMERA
├── SPRITES
├── ENEMIES
├── LEVELS
└── RENDER
```

Los elementos específicos del juego se definirán mediante datos:

```text
PLAYER
    velocidad
    fuerza de salto
    gravedad
    sprite
    animaciones

ENEMY
    tipo
    velocidad
    comportamiento
    sprite
    vida

LEVEL
    ancho
    plataformas
    enemigos
    objetos
    posición inicial
```

---

# 🏗️ Arquitectura

El juego utilizará un sistema de coordenadas de **mundo** independiente de las coordenadas de pantalla.

```text
                 MUNDO DEL JUEGO

  0        100       200       300       400       500
  │---------│---------│---------│---------│---------│
                      │
                      │
                   PLAYER
                      │
                ┌─────┴─────┐
                │   CAMERA  │
                └─────┬─────┘
                      │
                      ▼

              ┌─────────────────┐
              │    PANTALLA     │
              │                 │
              │      PLAYER     │
              │                 │
              └─────────────────┘
```

El jugador tendrá una posición dentro del mundo:

```text
PLAYER_X
PLAYER_Y
```

mientras que la cámara determinará qué parte del mundo se muestra:

```text
SCREEN_X = PLAYER_X - CAMERA_X
SCREEN_Y = PLAYER_Y - CAMERA_Y
```

Esto permitirá crear niveles más grandes que una única pantalla.

---

# ⚙️ Física

La física inicial será sencilla y estará adaptada a las limitaciones del C64.

Variables principales:

```text
VX      Velocidad horizontal
VY      Velocidad vertical
GRAVITY Gravedad
JUMP    Fuerza del salto
```

La actualización básica será:

```text
VY = VY + GRAVITY

X = X + VX
Y = Y + VY
```

Al saltar:

```text
VY = JUMP
```

Por ejemplo:

```text
GRAVITY = 1
JUMP    = -8
SPEED   = 3
```

Estos valores serán configurables para modificar la sensación del juego.

---

# 🧱 Sistema de plataformas

Los niveles podrán representarse mediante mapas sencillos.

Ejemplo:

```text
........................................
........................................
...............###......................
........................................
......####....................#####.....
........................................
############.......##########..........
```

Significado:

```text
. = espacio vacío
# = plataforma
P = jugador
E = enemigo
C = objeto
G = salida
```

Ejemplo:

```text
........................................
........E.........................E.....
...................###.................
......###..............................
.P.........................###..........
########################################
```

Esto permitirá crear niveles rápidamente y posteriormente desarrollar un sistema de editor.

---

# 👾 Enemigos

Los enemigos serán definidos mediante tipos.

Ejemplo:

```text
TYPE 0
Patrulla horizontal

TYPE 1
Persigue al jugador

TYPE 2
Enemigo volador

TYPE 3
Enemigo que salta

TYPE 4
Enemigo que dispara
```

Cada enemigo podrá tener sus propios parámetros:

```text
X
Y
VX
VY
TYPE
STATE
SPRITE
FRAME
LIFE
```

La inteligencia artificial se mantendrá separada del sistema de renderizado.

---

# 🧍 Jugador

El jugador tendrá diferentes estados:

```text
IDLE
WALK
JUMP
FALL
DEAD
```

Ejemplo:

```text
          IDLE
            │
       LEFT / RIGHT
            │
           WALK
            │
          JUMP
            │
           FALL
            │
       ┌────┴────┐
       │         │
     FLOOR      DEATH
       │
       ▼
      IDLE
```

Esto permitirá añadir posteriormente animaciones y comportamientos diferentes.

---

# 🎨 Sprites

El C64 dispone de sprites hardware que serán utilizados para los elementos móviles del juego.

La asignación inicial podría ser:

```text
SPRITE 0 → Jugador
SPRITE 1 → Enemigo 1
SPRITE 2 → Enemigo 2
SPRITE 3 → Enemigo 3
SPRITE 4 → Objeto
SPRITE 5 → Proyectil
```

Cada personaje podrá disponer de varios frames:

```text
PLAYER

FRAME 0 → parado
FRAME 1 → caminar
FRAME 2 → caminar
FRAME 3 → salto
FRAME 4 → caída
```

La animación se controlará desde el motor y no desde la lógica específica del personaje.

---

# 🖥️ Scroll horizontal

Uno de los objetivos principales del proyecto es conseguir un **scroll horizontal suave**.

La cámara seguirá al jugador:

```text
PLAYER → → → → →

┌─────────────────────────────┐
│                             │
│                  PLAYER     │
│                             │
└─────────────────────────────┘
                CAMERA → → →
```

Cuando el jugador alcance una determinada zona de la pantalla, la cámara comenzará a desplazarse.

El objetivo final es utilizar las capacidades del **VIC-II** para conseguir un desplazamiento fluido.

---

# 💾 Tecnología

## Hardware objetivo

- Commodore 64
- CPU MOS 6510
- VIC-II
- SID
- 64 KB RAM

## Lenguajes

### Prototipo

- BASIC V2

### Motor

- 6502 Assembly

El desarrollo comenzará con BASIC para validar la lógica del juego y posteriormente las partes críticas se implementarán en código máquina para mejorar el rendimiento.

---

# 📁 Estructura prevista del proyecto

```text
c64-platform-engine/
│
├── README.md
│
├── basic/
│   ├── prototype.bas
│   ├── player.bas
│   └── collision.bas
│
├── asm/
│   ├── main.asm
│   ├── player.asm
│   ├── physics.asm
│   ├── collision.asm
│   ├── camera.asm
│   ├── sprites.asm
│   ├── enemies.asm
│   └── levels.asm
│
├── sprites/
│   ├── player/
│   ├── enemies/
│   └── objects/
│
├── levels/
│   ├── level01.dat
│   ├── level02.dat
│   └── level03.dat
│
├── tools/
│   └── ...
│
├── docs/
│   ├── architecture.md
│   ├── physics.md
│   ├── sprites.md
│   └── levels.md
│
└── build/
```

---

# 🚧 Estado del proyecto

**En desarrollo — fase de prototipo.**

Actualmente el proyecto se encuentra en la fase inicial de diseño del motor.

### Fase 1 — Motor básico

- [ ] Inicialización del C64
- [ ] Crear jugador
- [ ] Movimiento horizontal
- [ ] Gravedad
- [ ] Salto
- [ ] Colisión con suelo

### Fase 2 — Plataformas

- [ ] Múltiples plataformas
- [ ] Colisiones
- [ ] Caídas
- [ ] Techo
- [ ] Paredes

### Fase 3 — Cámara

- [ ] Coordenadas de mundo
- [ ] Cámara horizontal
- [ ] Scroll
- [ ] Niveles superiores a una pantalla

### Fase 4 — Enemigos

- [ ] Sistema de entidades
- [ ] Enemigo patrulla
- [ ] Enemigo volador
- [ ] Enemigo perseguidor
- [ ] Colisión jugador/enemigo

### Fase 5 — Sprites

- [ ] Sprites personalizados
- [ ] Animaciones
- [ ] Gestión de frames
- [ ] Más de 8 objetos mediante multiplexación

### Fase 6 — Juego

- [ ] Sistema de vidas
- [ ] Puntuación
- [ ] Objetos
- [ ] Pantallas
- [ ] Sonido
- [ ] Música
- [ ] Pantalla de inicio
- [ ] Pantalla de Game Over

### Fase 7 — Herramientas

- [ ] Editor de niveles
- [ ] Editor de sprites
- [ ] Conversor de mapas
- [ ] Herramientas de compilación

---

# 🧪 Desarrollo

Durante las primeras fases se utilizará un emulador de C64 para probar rápidamente el código.

El prototipo puede ejecutarse en:

**C64 Online Emulator**

https://c64online.com/c64-online-emulator/

El objetivo final será generar un programa ejecutable en un C64 real o en cualquier emulador compatible.

---

# 🎯 Filosofía del proyecto

El proyecto pretende mantener una separación clara entre:

```text
MOTOR
   +
DATOS
   +
RECURSOS
```

De esta manera, cambiar el juego no debería requerir modificar el motor.

Por ejemplo:

```text
ENGINE
   │
   ├── PLAYER
   ├── PHYSICS
   ├── COLLISION
   ├── CAMERA
   └── RENDER
           │
           ▼
       GAME DATA
           │
     ┌─────┼─────┐
     ▼     ▼     ▼
  LEVELS ENEMIES SPRITES
```

Esto permitirá utilizar el motor como base para diferentes juegos de plataformas para Commodore 64.

---

# 📚 Objetivos de aprendizaje

Además de crear un juego, el proyecto servirá para estudiar:

- Arquitectura del Commodore 64
- CPU 6502/6510
- Memoria y direccionamiento
- VIC-II
- Sprites hardware
- Raster interrupts
- Scroll hardware
- Colisiones
- Física 2D
- Animación
- Gestión de memoria
- Optimización de código máquina
- Diseño de motores 2D
- Creación de herramientas para desarrollo retro

---

# 📜 Licencia

Proyecto experimental/educativo.

La licencia definitiva se establecerá cuando se publique la primera versión estable del motor.