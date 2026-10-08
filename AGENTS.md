# AGENTS.md

## Qué es este repo

- Juego Pac-Man en vanilla JS + HTML + CSS (`src/`). **Sin build, sin npm, sin tests, sin lint.**
  No hay `package.json`: no intentes instalar ni ejecutar scripts que no existen.
- Proyecto de curso para practicar **Spec Driven Development** (ver `README.md`).
- UI y comentarios del código en español; mantén ese idioma.

## Cómo ejecutarlo

- Abrir `src/index.html` directamente en el navegador funciona (no hay ES modules,
  `fetch` ni `localStorage`; solo `<script src>` clásicos).
- Si prefieres servidor estático: `npx serve src` o
  `python -m http.server --directory src`.

## Arquitectura (lo que un agente no adivina)

- Cuatro scripts en **scope global**, sin `import`/`export`. El orden de carga en
  `src/index.html` es contrato: `maze.js -> game.js -> render.js -> main.js`.
  Cualquier archivo nuevo o reordenamiento hay que registrarlo ahí.
- Capas:
  - `maze.js`: mapa ASCII `MAZE_STR` -> `MAZE` (tiles: `1` pared, `2` dot, `3`
    puerta, `0` vacío), `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`.
  - `game.js`: estado y reglas (`createGame`, `update`); depende de las globals de
    `maze.js`.
  - `render.js`: dibujo en canvas (`draw`), `TILE = 20`.
  - `main.js`: bucle `requestAnimationFrame`, teclado e overlay de inicio/victoria.
- `createGame()` copia `MAZE` a `game.grid` para comer dots sin tocar el original.
  No mutes `MAZE` durante el juego.
- Lienzo fijo en `src/index.html` (`560x620`) y `TILE = 20` en `render.js` -> 28x31
  celdas. Cambiar `MAZE_STR` o `TILE` **no** redimensiona el canvas: ajusta ambos a mano.
- Velocidades en `game.js`: `PACMAN_SPEED = 1/8` celda/frame (alinia cada 8 frames)
  y `GHOST_SPEED = 1/10`. El giro en esquinas depende de `aligned()` con epsilon
  `1e-3`; usa velocidades que dividan 1 para no romper el alineado.
- Laberinto con túnel lateral: filas sin muro en bordes + `wrapTunnel`.

## Flujo de trabajo (skills del repo)

- Skills `spec` y `spec-impl` instaladas vía `skills-lock.json` / `.agents/skills`.
- Orden esperado: `spec` (diseñar la spec y que el usuario la apruebe) ->
  `spec-impl` (implementar en branch con revisiones de diff).

## Convenciones de código

- Espacios dentro de paréntesis: `if ( x )`, `f( a, b )`; llave de apertura en la
  misma línea; comillas simples en JS. Copia el estilo del archivo que edites.
- Comentarios en español, solo donde aportan (las cabeceras de archivo documentan
  sus dependencias).

## No tocar

- `open-pacman.zip`: snapshot del proyecto terminado (referencia del curso).
  No es fuente de verdad para edits.
