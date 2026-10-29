# Arte del juego

Esta carpeta es del artista: el juego carga de aquí los sprites al arrancar. Si un fichero falta o no
carga, el juego usa su dibujo interno por código (y avisa en la consola del navegador), así que nunca se
rompe por un recurso a medias.

Se cargan desde aquí **los aviones** y **las estructuras** (objetivos, sus ruinas, edificios del
aeródromo y árboles de la vista lateral). El terreno, las nubes y los efectos siguen dibujados por código.

## Ficheros

```
manifest.json                    # qué imagen usa cada avión y sus puntos de anclaje
aircraft/<avión>.side.png        # vista lateral
aircraft/<avión>.top.png         # vista cenital
aircraft/enemy-<familia>.*.png   # enemigos (vuelan aviones de la misma época que el jugador)
```

Aviones del jugador: `biplane`, `p51`, `me262`, `mig21`, `f14`, `f15`.
Familias de enemigos: `biplane`, `prop`, `earlyjet`, `delta`, `tomcat` (F-14), `eagle` (F-15).

## Reglas de dibujo

- **PNG con transparencia**, sin fondo.
- **Vista lateral**: el avión **mira a la derecha**, con la cabina **arriba**. El juego lo gira para
  cabecear y lo voltea al invertirse.
- **Vista cenital**: el avión **mira hacia arriba**. El juego lo rota según el rumbo y genera él solo la
  sombra a partir de la silueta.
- El sprite se centra en el centro de la imagen: deja el avión centrado y sin márgenes de más.
- Escala: con `"scale": 1` (lo habitual) **un píxel de la imagen es un píxel de la pantalla del juego**,
  así que todo el detalle que dibujes se ve tal cual. Los aviones actuales miden ~50–66 px de largo y
  ~18–24 de alto en lateral (el doble del arte original, de ahí los bloques de 2×2: puedes refinarlos).
- Mantén ese tamaño. El juego pinta a 960×600: dibujar más grande y reducir con `scale` < 1 **no** da
  más detalle (se pierden píxeles y salen desiguales), y dibujar más grande con `scale: 1` hace el avión
  más grande en pantalla.
- La zona de impacto está ajustada a ese tamaño (radio ~20 unidades, ~12 de alto) y no se deduce del
  dibujo: si un avión cambia mucho de tamaño, hay que ajustarla en el código (`src/sim/world.ts`).
- Si cambias el tamaño de la imagen, actualiza también `nose` y `exhausts`, que van en píxeles de la imagen.

## Puntos de anclaje (`manifest.json`)

Coordenadas en píxeles de **tu** imagen (origen arriba a la izquierda):

- `nose`: buje de la hélice. El juego dibuja encima la hélice girando (solo en aviones de hélice).
- `exhausts`: lista de toberas. El juego dibuja ahí la llama del postquemador (solo reactores).

```json
"f15": {
  "side": { "image": "aircraft/f15.side.png", "nose": [33, 6.5], "exhausts": [[0, 6]], "scale": 1 },
  "top":  { "image": "aircraft/f15.top.png",  "nose": [12.5, 0], "exhausts": [[10, 15], [15, 15]], "scale": 1 }
}
```

## Estructuras (`structures/` y `"structures"` en el manifest)

Una pieza por fichero, con el nombre de su entrada en el manifest:

| Piezas | Qué son |
| --- | --- |
| `factory`, `fuel`, `flak`, `hangar` `.side` / `.top` | Objetivos enemigos |
| `<objetivo>.wreck.side` / `.wreck.top` | Su ruina tras destruirlo |
| `flak.barrel.side` / `.top` | Cañón del antiaéreo (pieza móvil) |
| `tower`, `airfield-hangar`, `windsock` `.side` / `.top` | Aeródromo propio |
| `windsock.sock.side` / `.top` | Manga de viento (pieza móvil) |
| `tree.pine.0.side` … `tree.bush.1.side` | Árboles de la vista lateral: `pine`, `round` o `bush`, variantes `0` y `1` |

- **Lateral**: el edificio apoyado en el suelo. **Cenital**: visto desde arriba con el **norte arriba**;
  el juego lo gira con el mapa. Incluye la sombra en el dibujo cenital si la quieres.
- `anchor`: el píxel que se coloca sobre el punto del mundo. En lateral, el **centro de la base**
  (donde toca el suelo); en cenital, el **centro** del edificio.
- **Piezas móviles** (`flak.barrel.*`, `windsock.sock.*`): dibújalas **apuntando a la derecha**. Su
  `anchor` es el punto de giro, y `mount` dice dónde va ese punto respecto al `anchor` de su edificio,
  en unidades de juego (= píxeles con `scale: 1`). El juego las gira: el cañón apunta al avión y la
  manga se mueve con el viento.
- `lights`: puntos (en píxeles de la imagen) donde el juego dibuja una luz roja parpadeante, como la
  baliza de la torre.
- La **zona de impacto** de los objetivos la fija el juego por tipo (fábrica 80×56, depósito 40×36,
  antiaéreo 28×20, hangar 72×40 unidades): mantén los dibujos aproximadamente de ese tamaño.

## Flujo de trabajo

1. `pnpm dev` y abre el juego. Atajos útiles en la URL: `?plane=6` (avión 1–6), `?view=top`
   (empezar en cenital), `?seed=12345` (mismo mapa siempre).
2. Edita el PNG (Aseprite, LibreSprite, Photoshop…) y recarga el navegador.
3. `pnpm export-art` vuelve a crear los ficheros que falten a partir del arte interno, **sin tocar los
   existentes** (las estructuras se renderizan con Chrome/Edge; si no lo encuentra, define
   `CHROME_PATH`). `node scripts/export-art.ts --force` o `pnpm export-structures --force` lo
   sobrescriben todo (cuidado: borra tu trabajo).
4. `?at=x,y` en la URL empieza en otro punto del mapa, útil para ir directo a ver un edificio.
