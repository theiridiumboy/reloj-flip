# Reloj Flip — Mesa de Diálogo Venezuela

Widget banner (980×200, responsive) tipo pizarra split-flap que cuenta el tiempo transcurrido desde que arrancó la mesa de diálogo en Venezuela. Todo el banner es un link clickeable a la cobertura en LaPatilla.com.

Un solo archivo autocontenido: `index.html` (CSS y logo embebidos, sin dependencias externas).

## URL en vivo

**https://theiridiumboy.github.io/reloj-flip/**

Este link se sirve directo desde GitHub Pages, apuntando a la rama `main` de este repo. Cualquier commit pusheado a `main` queda en línea en 1-2 minutos, sin pasos manuales.

## Para el webmaster: cómo embeberlo en LaPatilla.com

Usar **iframe**, nunca copiar el HTML crudo al editor de WordPress — el CSS del widget puede chocar con los estilos globales del sitio (`html`/`body`).

```html
<iframe
  src="https://theiridiumboy.github.io/reloj-flip/"
  width="980"
  height="200"
  style="max-width:100%; border:0;"
  loading="lazy"
  title="Contador: mesa de diálogo en Venezuela">
</iframe>
```

El widget ya es responsive internamente (escala como una unidad rígida), así que el iframe puede vivir dentro de un contenedor más angosto sin romperse.

## Widget IA: el slot es reutilizable

`index.html` ya no es solo "el reloj" — es el espacio en vivo del **Widget
IA**, un slot editorial que puede mostrar distintas ideas con el tiempo.
Cada idea, incluido el reloj, vive archivada en `variants/` (ver
`variants/README.md` para el catálogo y el flujo completo de swap).

## Cómo hacer cambios (al variant en vivo)

1. Editar el archivo del variant correspondiente en `variants/` (o pedirle
   a Claude que lo haga).
2. Verificar en el navegador antes de subir (`gstack browse`).
3. Copiar el variant a `index.html`, actualizar `variants/README.md`.
4. Commit + push a `main`:
   ```bash
   git add index.html variants/
   git commit -m "descripción del cambio"
   git push
   ```
5. GitHub Pages redeploya solo. Verificar con `curl` que sirvió el cambio
   antes de avisar que ya está en vivo.

## Fecha de inicio del contador

Hardcodeada en `index.html`, buscar `var START`:

```js
var START = new Date('2026-08-06T16:32:00-04:00').getTime();
```

Si la fecha real de arranque de la mesa cambia, actualizar ese valor y pushear.

## Estructura del repo

- `index.html` — el widget, listo para producción.
- `DESIGN.md` — contrato de dirección visual, decisiones de diseño detalladas.
- `PRODUCT.md` — notas de producto.
- `experiments/` — versiones descartadas o de prueba, no usar en producción.
- `variants/` — catálogo de ideas archivadas para el Widget IA (ver `variants/README.md`). `index.html` siempre es un espejo exacto del variant marcado "En vivo" ahí.
