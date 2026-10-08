# SPEC 01 — Cuatro fantasmas con personalidades arcade

> **Status:** Approved
> **Depends on:** none
> **Date:** 2026-10-08
> **Objective:** Añadir cuatro fantasmas con comportamientos diferenciados al estilo arcade (Blinky, Pinky, Inky, Clyde), con fases scatter/chase, todos partiendo dentro del pen y usando decisiones por objetivo (no aleatorias).

## Scope

**In:**

- Ampliar `GHOST_STARTS` a 4 entradas, todas dentro del recinto del pen (`y=14`, `x` entre 12 y 15 según el arcade: Blinky, Pinky, Inky, Clyde en sus posiciones clásicas).
- Asignar a cada fantasma un `kind` que determine su comportamiento objetivo: `blinky` (persecución directa), `pinky` (emboscada 4 celdas delante), `inky` (combinación con Blinky), `clyde` (persigue si lejos, huye si cerca).
- Añadir un temporizador de fases `scatter`/`chase` que alterna periódicamente (ciclo arcade simplificado). Las fases cambian en intervalos fijos definidos en `game.js`.
- Mantener las reglas de movimiento existentes: alineado a celda para decidir, no puede girar 180° salvo callejón sin salida, túnel funcional, bloqueo solo por muro (1), puerta (3) no bloquea a fantasmas.
- Mantener misma velocidad para todos (`GHOST_SPEED = 1/10`).

**Out of scope (for future specs):**

- Modo "frightened" (azules, comibles) al comer power-up.
- Power-ups (energizers).
- "Elroy" (aceleración de Blinky con pocos dots).
- Salida escalonada de los fantasmas del pen (todos salen juntos o según orden).
- Scatter a esquinas exactas animadas; se usan objetivos puntuales por fase.

## Data model

```js
// maze.js (modificación)
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'blinky' }, // fuera del pen en arcade, pero aquí dentro
  { x: 14, y: 14, kind: 'pinky'  },
  { x: 12, y: 14, kind: 'inky'   },
  { x: 15, y: 14, kind: 'clyde'  },
];

// game.js (nuevo estado)
const SCATTER_CHASE = [
  { mode: 'scatter', dur: 420 }, // frames ~7s a 60fps aprox
  { mode: 'chase',   dur: 1020 },
  { mode: 'scatter', dur: 420 },
  { mode: 'chase',   dur: 1020 },
  { mode: 'scatter', dur: 300 },
  { mode: 'chase',   dur: Infinity },
];

const SCATTER_TARGETS = {
  blinky: { x: 25, y:  0 }, // esquina superior derecha
  pinky:  { x:  2, y:  0 }, // esquina superior izquierda
  inky:   { x: 27, y: 30 }, // esquina inferior derecha
  clyde:  { x:  0, y: 30 }, // esquina inferior izquierda
};
```

Notas:
- `SCATTER_CHASE` define la alternancia; `dur` en frames (consistente con `requestAnimationFrame`).
- `scatter` usa `SCATTER_TARGETS` por `kind`.
- `chase` usa objetivo por fantasma tal como se describe abajo.

## Implementation plan

1. **Actualizar posiciones iniciales** en `src/js/maze.js`: cambiar `GHOST_STARTS` a 4 con `kind` `blinky,pinky,inky,clyde`. Todos en `y=14`, `x` 13,14,12,15. Verificar que no solapan la puerta (3).
2. **Añadir configuración de fases y targets** en `src/js/game.js`: `SCATTER_CHASE`, `SCATTER_TARGETS`. Mantener `DIRS`, `OPPOSITE`, `PACMAN_SPEED`, `GHOST_SPEED` tal cual.
3. **Ampliar estado de juego** en `createGame()`: añadir `mode = 'scatter'`, `modeTimer = 0`, `modeIndex = 0` (o equivalente) para controlar el ciclo. Inicializar en `scatter` con `dur` 0/primer tramo.
4. **Implementar cálculo de objetivos por fantasma**:
   - `blinky`: objetivo = posición actual redondeada de Pac-Man (`px,py`).
   - `pinky`: objetivo = 4 celdas delante de Pac-Man en su dirección actual. Si la dirección apunta hacia arriba en el clásico hay un offset, pero con el mapa actual basta con `px + 4*dx, py + 4*dy` usando `DIRS[p.dir]`.
   - `inky`: tomar posición 2 celdas delante de Pac-Man (`px2,py2`), vector desde `blinky` a ese punto, doblarlo (`px2 + (px2-bx), py2 + (py2-by)`).
   - `clyde`: si distancia (Manhattan) entre clyde y Pac-Man > 8, objetivo = Pac-Man. Si <= 8, objetivo = `SCATTER_TARGETS.clyde`.
5. **Reescribir `decideGhost`** para elegir dirección que minimice distancia euclídea/Manhattan al objetivo (Manhattan como en clásico), sin volver atrás salvo callejón sin salida. Seguir filtrando opciones por `canMove` y `OPPOSITE`.
6. **Actualizar temporizador de fases** en `update()` antes de mover fantasmas: `modeTimer++`, si alcanza `dur` pasar al siguiente tramo y resetear timer. No afecta a movimiento ni colisiones.
7. **Actualizar render** en `src/js/render.js`: ampliar `GHOST_COLORS` a 4 (rojo, rosa, cian, naranja) asignados por índice según orden de `game.ghosts` (coincidir con `blinky,pinky,inky,clyde`).
8. **Verificación manual**: abrir `src/index.html` en navegador. Comprobar que aparecen 4 fantasmas con colores distintos, se mueven con diferentes trayectorias hacia objetivos, el túnel funciona, colisión resta vida y reinicia posiciones.

## Acceptance criteria

- [ ] Existen exactamente 4 fantasmas con `kind` `blinky`, `pinky`, `inky`, `clyde`.
- [ ] Todos los fantasmas nacen dentro del pen (`y === 14` y `x` entre 12-15).
- [ ] `GHOST_COLORS` tiene 4 colores y se asigna a los 4 fantasmas.
- [ ] El temporizador de fases `scatter/chase` avanza en `update()` y cambia `mode` al alcanzar `dur`.
- [ ] `blinky` elige dirección hacia la posición redondeada de Pac-Man.
- [ ] `pinky` elige dirección hacia ~4 celdas delante de Pac-Man según su dirección.
- [ ] `inky` usa referencia a `blinky` y al punto 2 celdas delante para calcular su objetivo.
- [ ] `clyde` cambia entre perseguir a Pac-Man o huir a su esquina según distancia Manhattan <= 8.
- [ ] En callejón sin salida, permite giro de 180° (comportamiento existente preservado).
- [ ] Túnel lateral sigue funcionando para todos los fantasmas.
- [ ] Colisiones, vidas, victoria/derrota y conteo de dots siguen funcionando sin cambios visibles.
- [ ] No se muta `MAZE` original durante la partida (se sigue copiando a `grid`).

## Decisions taken and discarded

- **Yes:** Estilo arcade clásico con 4 personalidades. Aporta variedad real sin aleatoriedad total.
- **Yes:** Todos dentro del pen inicialmente. Más fiel a la colocación clásica (Blinky suele salir primero, pero para este ejercicio partimos todos juntos).
- **Yes:** Fases `scatter/chase` con ciclo simplificado. Diferencia el comportamiento sin añadir power-ups aún.
- **Yes:** Misma velocidad para todos. Evita tocar balance y mantiene `GHOST_SPEED` estable.
- **No:** "Elroy" o aceleración por dots. Va fuera de alcance para esta spec.
- **No:** Modo frightened/comible. Requiere power-ups; se deja para otra spec.
- **No:** Salida escalonada del pen. Aumenta estado (tiempo de salida por fantasma). Postergado.

## Identified risks

| Risk | Mitigation |
|---|---|
| Cálculo de `inky` depende de `blinky` existir | Asumimos orden de `ghosts` coherente con `GHOST_STARTS` (blinky primero). Validar acceso. |
| Objetivo puede caer en muro/túnel | Elegimos dirección por menor distancia a objetivo (no teletransportamos); el pathing por celdas sigue las reglas existentes. |
| Ciclo `scatter/chase` infinito en el último tramo | Se usa `dur: Infinity` para mantener modo chase hasta fin (simplificación arcade). |

## What is **not** in this spec

- Power-ups, energizers o modo frightened.
- Aceleración de Blinky ("Cruise Elroy").
- Animaciones de salida del pen o timers individuales de liberación.
- Cambio visual de fantasmas (parpadeo azul/ojos) fuera del color base.
- Multiplayer o IA distinta a los 4 tipos arcade.
