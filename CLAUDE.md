# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Ejecutar el proyecto

Sin dependencias ni proceso de build. Abre directamente en el navegador:

```bash
# Servidor estático local (recomendado)
python3 -m http.server 8000
# o
npx serve .
```

Luego abre `http://localhost:8000`. También funciona abrir `index.html` directamente en el navegador.

No hay pruebas automatizadas — la verificación es manual en el navegador.

## Arquitectura

Tres archivos: `index.html`, `style.css`, `game.js`. Toda la lógica está en `game.js` (~326 líneas), sin dependencias externas.

**Estado del juego** (variables globales en `game.js`):
- `board` — matriz 10×20 donde 0 = vacío, 1–7 = índice de color de la pieza bloqueada
- `current` — pieza activa: `{ type, shape, x, y }` (`type` es el índice 1–7 que referencia `COLORS`/`PIECES`)
- `next` — pieza siguiente para el preview
- `animId` — ID del `requestAnimationFrame` activo; se cancela al pausar o terminar

**Game loop** (`requestAnimationFrame`):
1. Acumula tiempo transcurrido en `dropAccum`
2. Al superar `dropInterval`, baja la pieza o la bloquea (`lockPiece`)
3. Llama a `draw()` en cada frame (redibujado completo, no incremental)
4. Solicita el siguiente frame

**Funciones clave**:
- `collide()` — detecta colisiones con bordes y piezas bloqueadas
- `rotateCW(shape)` — transpone la matriz para rotar 90° en sentido horario
- `tryRotate()` — rotación CW con wall kicks (±0, ±1, ±2 columnas)
- `lockPiece()` → `merge()` → `clearLines()` — bloquea, fusiona y limpia líneas
- `spawn()` — activa la pieza siguiente y detecta game over si colisiona al spawnar
- `hardDrop()` / `softDrop()` — caída instantánea / acelerada
- `ghostY()` — calcula la posición de aterrizaje para la ghost piece
- `drawGrid()` — dibuja la cuadrícula; lee `--grid-color` via `getComputedStyle` para adaptarse al tema
- `drawBlock()` — dibuja un bloque con highlight; acepta `alpha` para la ghost (0.2)
- `applyTheme(light)` — aplica el tema claro/oscuro: toggle en `body.light-mode`, icono y `localStorage`

**Dos canvas**:
- `#board` (300×600 px) — tablero principal
- `#next-canvas` (120×120 px) — preview de la siguiente pieza

**Overlay** (`#overlay`):
- Cubre todo el `.wrapper` con `position: absolute; inset: 0`
- Sirve tanto para pausa como para game over; la diferencia es que `#resume-btn` aparece solo en pausa (se alterna con `.hidden`)
- La clase CSS `.hidden` (`display: none`) controla su visibilidad

## Tema visual (claro/oscuro)

Controlado por la clase `body.light-mode` y CSS custom properties definidas en `:root` (modo oscuro) y `body.light-mode` (modo claro).

- El toggle (`#theme-toggle`) es un checkbox oculto; el estado visual lo da `.theme-track::after` via `transform: translateX`
- `applyTheme(light)` sincroniza la clase, el checkbox, el icono (`☀` / `☾`) y persiste en `localStorage`
- Clave de `localStorage`: `'tetris-theme'` — valores `'light'` o `'dark'`
- `drawGrid()` usa `getComputedStyle(document.body).getPropertyValue('--grid-color')` para que la cuadrícula respete el tema sin reescribir el canvas

## Título visual (pixel art CSS)

El título "TETRIS" se construye letra por letra con `<i>` posicionados en un CSS Grid de 3×5 celdas mediante custom properties `--x`/`--y` (`grid-column: var(--x); grid-row: var(--y)`). Para modificar o añadir letras: editar los `<i>` en `index.html` y el color en `style.css`.

## Controles

| Tecla | Acción |
|-------|--------|
| `←` / `→` | Mover izquierda/derecha |
| `↑` o `X` | Rotar en sentido horario |
| `↓` | Soft drop (+1 pt por fila) |
| `Space` | Hard drop instantáneo (+2 pt por celda) |
| `P` | Pausar/reanudar |
| `R` | Reiniciar juego |

## Comportamientos no obvios

- **Auto-start**: `init()` se llama al cargar la página — no hay pantalla de inicio.
- **Hard drop scoring**: suma 2 puntos por cada celda caída (`score += (ghostY - current.y) * 2`).
- **Velocidad de caída**: `Math.max(100, 1000 - (level - 1) * 90)` ms — mínimo 100ms al nivel 10+.
- **Wall kicks**: `tryRotate` intenta offsets `[0, -1, 1, -2, 2]` en X antes de rechazar la rotación.
- **Progresión de nivel**: `level = Math.floor(lines / 10) + 1` — sube cada 10 líneas acumuladas.
- **Space y scroll**: `hardDrop` llama `e.preventDefault()` — sin esto Space hace scroll en el navegador.
- **`P` y `R` ignoran el estado de pausa/game over**: el handler de `keydown` los procesa antes del guard `if (paused || gameOver) return`.
- **Guard anti-loop-duplicado en `init()`**: llama `cancelAnimationFrame(animId)` antes de arrancar un nuevo loop — sin esto, presionar `R` mientras el juego corre generaría dos loops en paralelo acelerando la caída indefinidamente.

## Constantes importantes

Todas en la parte superior de `game.js`:

| Constante | Valor por defecto | Nota |
|-----------|-------------------|------|
| `COLS` | 10 | Si se cambia, actualizar el ancho del canvas en `index.html` |
| `ROWS` | 20 | Si se cambia, actualizar la altura del canvas en `index.html` |
| `BLOCK` | 30 px | Tamaño de cada bloque |
| `LINE_SCORES` | `[0,100,300,500,800]` | Puntos por 1/2/3/4 líneas × nivel |
| `PIECES` / `COLORS` | arrays 1–7 | Comparten el mismo índice de pieza (0 = `null`/vacío). Al añadir una pieza, actualizar ambos en paralelo. |
