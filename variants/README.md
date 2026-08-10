# Catálogo de variants — Widget IA

Cada fila es una idea archivada en esta carpeta. `index.html` (raíz del
repo) es siempre un espejo exacto del variant marcado como **En vivo**.

| Archivo | Estado | Publicado/Archivado | Descripción |
|---|---|---|---|
| `reloj-mesa-dialogo.html` | En vivo | 2026-08-06 | Contador split-flap desde el arranque de la mesa de diálogo en Venezuela. Banner clickeable a la cobertura en LaPatilla.com. |

## Cómo hacer un swap

1. Construir la idea nueva en `variants/<slug>.html`, verificar con `gstack browse` en local.
2. Mostrarle a Rory, esperar aprobación.
3. Copiar ese archivo a `index.html` (raíz del repo).
4. Actualizar esta tabla: la fila del variant nuevo pasa a "En vivo", la fila anterior pasa a "Archivado" con la fecha del swap.
5. Commit + push a `main`.
6. Verificar con `curl` que `https://theiridiumboy.github.io/reloj-flip/` sirve el contenido nuevo (no una respuesta cacheada).

Ningún variant se borra al dejar de estar en vivo — queda archivado acá
como rollback instantáneo.
