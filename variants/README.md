# Widget IA: catálogo de variants

Cada idea que ocupa el iframe 980×200 del home vive aquí como archivo autocontenido.
`index.html` (raíz) es siempre una copia exacta del variant en vivo.

| Variant | Qué es | Estado |
|---|---|---|
| `reloj-mesa-dialogo.html` | Reloj split-flap: tiempo desde el arranque de la mesa de diálogo (6 ago 2026, 4:32 p.m. VET). El conteo sale de una fecha fija, no se pierde al sacarlo del aire. | En vivo 2026-08-06 a 2026-11-03. Archivado, vuelve al terminar la elección. |
| `midterms-2026.html` | Elecciones de medio término EE.UU. 2026: barras Senado y Cámara en vivo. Lee `live.json` de la rama `datos` (raw.githubusercontent, caché 5 min), que la Mac de Rory empuja cada 2 min. Imágenes en `assets/midterms/`. | En vivo desde 2026-11-03. |

## Restaurar un variant

```sh
cp variants/<slug>.html index.html && git commit -am "Restaurar <slug>" && git push
```

Luego confirmar con `curl https://theiridiumboy.github.io/reloj-flip/` que se sirve el HTML nuevo.
Copia exacta del reloj también en la etiqueta git `reloj-mesa-dialogo`.
