# Widget IA — diseño del slot editorial reutilizable

**Nombre:** Widget IA
**Fecha:** 2026-08-09
**Repo:** `theiridiumboy/reloj-flip`
**Estado:** aprobado por Rory, pendiente de implementación

## Contexto

El widget de reloj (split-flap, contador de la mesa de diálogo en Venezuela)
vive embebido en producción en LaPatilla.com vía un `<iframe>` fijo apuntando
a `https://theiridiumboy.github.io/reloj-flip/` (GitHub Pages, deploy
automático en cada push a `main`). Rory quiere convertir ese mismo espacio en
un slot editorial reutilizable: no solo el reloj, sino cualquier idea nueva
que exploremos juntos con el tiempo, sin que el webmaster tenga que volver a
tocar el sitio.

## Decisiones (confirmadas con Rory)

1. **Mismo iframe, reemplaza al reloj.** Cuando se publica una idea nueva,
   el contador deja de mostrarse mientras esa idea esté activa. No es un
   slot en paralelo — es el mismo espacio, un contenido a la vez.
2. **Disparador de ideas: ambos.** Rory puede pedir una idea directamente, y
   Claude puede proponer una por iniciativa propia si detecta una coyuntura
   que encaje — sin restricción de que solo pueda ser a pedido.
3. **Alcance de contenido: abierto.** No limitado al espíritu del reloj
   (contadores/comparadores de coyuntura) — cualquier widget que corra
   estático en un iframe cabe, a discreción de lo que se proponga y apruebe.
4. **Tamaño fijo: 980×200 siempre.** Toda idea se diseña dentro de ese
   formato banner. Nunca se le pide al webmaster que ajuste el iframe —
   esa es la propiedad que hace funcionar el pipeline sin fricción.

## Arquitectura

Mismo repo, mismo sitio, mismo iframe en producción. `index.html` sigue
siendo el único archivo que GitHub Pages sirve como producción — su rol no
cambia: es siempre "lo que está en vivo ahora mismo". Lo nuevo es una
carpeta `variants/` que archiva cada idea (incluido el reloj) como su
propio archivo autocontenido, siguiendo el mismo patrón que ya usa
`experiments/` en este repo.

## Componentes

- **`variants/<slug>.html`** — una idea, un archivo, sin dependencias
  externas, 980×200 fijo, mismo stack que el reloj (HTML/CSS/JS vanilla,
  sin build step). El reloj actual se archiva como
  `variants/reloj-mesa-dialogo.html` antes de tocar `index.html`.
- **`variants/README.md`** — catálogo: nombre del variant, fecha en que se
  publicó/archivó, una línea describiendo qué es. Evita perder la pista de
  qué existe para restaurar más adelante.
- **`index.html`** — siempre un espejo exacto del contenido de
  `variants/<slug-actual>.html`. Nunca se edita "a mano" de forma
  independiente del variant correspondiente — cualquier cambio se hace
  primero en el variant, después se copia.

## Flujo de datos (swap)

1. Surge la idea — Rory la pide, o Claude la propone por coyuntura.
2. Claude construye `variants/<slug>.html`, la verifica en local con
   `gstack browse` (screenshot).
3. Rory revisa y aprueba (o pide ajustes, se repite el paso 2).
4. Claude copia ese archivo a `index.html`, actualiza
   `variants/README.md`, commit + push a `main`.
5. GitHub Pages redeploya solo (1-2 min). Claude confirma con `curl` a la
   URL de Pages que el HTML servido es el nuevo (no una versión cacheada),
   y avisa a Rory que ya está en vivo.

**Restaurar una idea anterior** (incluido el reloj) es el mismo flujo
invertido: copiar el `variants/<slug>.html` correspondiente a `index.html`,
commit, push, verificar.

## Manejo de errores

- Nada se copia a `index.html` sin verificarse antes en `gstack browse`
  contra el archivo local del variant.
- El variant saliente nunca se borra al reemplazarlo — queda en
  `variants/` como rollback instantáneo si algo se ve mal ya en
  producción (mismo flujo de "restaurar", solo que hacia el variant
  anterior en vez de hacia el reloj).
- Si el `push` falla o GitHub Pages tarda más de lo esperado en
  redeployar, Claude reintenta y avisa a Rory del retraso — no se declara
  "en vivo" sin la confirmación por `curl`.

## Testing

- Antes de publicar: `gstack browse` sobre el archivo local del variant,
  captura de pantalla pa verificación visual.
- Después del push: `curl` a `https://theiridiumboy.github.io/reloj-flip/`
  y confirmar que el HTML devuelto corresponde al variant nuevo (no una
  respuesta cacheada de GitHub Pages/CDN).

## Fuera de alcance (por ahora)

- Múltiples ideas corriendo en paralelo (distintos iframes/URLs) — se
  descartó explícitamente porque le pediría al webmaster tocar el sitio
  cada vez, rompiendo la propiedad central de este pipeline.
- Tamaños de iframe distintos a 980×200 — descartado por la misma razón.
- Mecanismo de rotación automática/programada entre variants — no
  discutido, no se asume; cada swap es una decisión explícita de Rory.
