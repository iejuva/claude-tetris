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

Tres archivos: `index.html`, `style.css`, `game.js`. Toda la lógica está en `game.js` (~305 líneas), sin dependencias externas.

**Estado del juego** (variables globales en `game.js`):
- `board` — matriz 10×20 donde 0 = vacío, 1–7 = índice de color de la pieza bloqueada
- `current` — pieza activa: `{ shape, x, y }`
- `next` — pieza siguiente para el preview

**Game loop** (`requestAnimationFrame`):
1. Acumula tiempo transcurrido
2. Al superar `dropInterval`, baja la pieza o la bloquea (`lockPiece`)
3. Dibuja el tablero, la ghost piece y el preview
4. Solicita el siguiente frame

**Funciones clave**:
- `collide()` — detecta colisiones con bordes y piezas bloqueadas
- `tryRotate()` — rotación CW con wall kicks (±0, ±1, ±2 columnas)
- `lockPiece()` → `merge()` → `clearLines()` — bloquea, fusiona y limpia líneas
- `hardDrop()` / `softDrop()` — caída instantánea / acelerada
- `ghostY()` — calcula la posición de aterrizaje para la ghost piece

**Dos canvas**:
- `#board` (300×600 px) — tablero principal
- `#next-canvas` (120×120 px) — preview de la siguiente pieza

## Controles

| Tecla | Acción |
|-------|--------|
| `←` / `→` | Mover izquierda/derecha |
| `↑` o `X` | Rotar en sentido horario |
| `↓` | Soft drop (+1 pt por fila) |
| `Space` | Hard drop instantáneo (+2 pt por celda) |
| `P` | Pausar/reanudar |

## Comportamientos no obvios

- **Auto-start**: `init()` se llama al cargar la página — no hay pantalla de inicio.
- **Hard drop scoring**: suma 2 puntos por cada celda caída (`score += (ghostY - current.y) * 2`).
- **Velocidad de caída**: `Math.max(100, 1000 - (level - 1) * 90)` ms — mínimo 100ms al nivel 10+.
- **Wall kicks**: `tryRotate` intenta offsets `[0, -1, 1, -2, 2]` en X antes de rechazar la rotación.

## Constantes importantes

Todas en la parte superior de `game.js`:

| Constante | Valor por defecto | Nota |
|-----------|-------------------|------|
| `COLS` | 10 | Si se cambia, actualizar el ancho del canvas en `index.html` |
| `ROWS` | 20 | Si se cambia, actualizar la altura del canvas en `index.html` |
| `BLOCK` | 30 px | Tamaño de cada bloque |
| `LINE_SCORES` | `[0,100,300,500,800]` | Puntos por 1/2/3/4 líneas × nivel |
