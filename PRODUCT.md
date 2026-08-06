# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

## Stack

static HTML/CSS/JS (no build step, single embeddable widget)

## Users

Lectores de LaPatilla.com viendo la cobertura de la mesa de diálogo en Venezuela. Necesitan ver, de un vistazo, cuánto tiempo lleva activa la mesa de diálogo.

## Product Purpose

Banner-widget de 980x200 embebido en LaPatilla.com: un contador tipo "split-flap" (pizarra de aeropuerto) que cuenta hacia arriba (días, horas, minutos, segundos) desde el momento en que arranca a correr, representando la duración en curso de la mesa de diálogo en Venezuela. Éxito = lectura instantánea del tiempo transcurrido, estética de pizarra aeropuerto fiel a la referencia entregada.

## Positioning

Pieza editorial narrativa (no analítica): convierte un dato de duración en una imagen memorable y compartible sobre la mesa de diálogo, en vez de una cifra suelta en texto.

## Operating Context

Vive embebido en una página de LaPatilla.com (script/iframe). Corre en el navegador del lector, cliente-side, sin backend. El punto de arranque del conteo es la fecha/hora en que se instale el widget (momento de publicación de la mesa de diálogo como tema activo). Ya no requiere un contenedor de ancho fijo: escala a cualquier ancho de contenedor, útil tanto en desktop como en la versión mobile de un artículo.

## Capabilities and Constraints

- Cuenta hacia arriba en 4 unidades: Días, Horas, Minutos, Segundos.
- Animación de flip por dígito: una tarjeta de dos caras por carácter que gira 180° (promovido de `experiments/flip-v2-realista.html` a producción el 2026-08-06 — ver `DESIGN.md` para el historial completo de ajustes, incluida la ronda anterior de una sola cara que se descartó).
- Diseño base 980x200 (banner de sitio), responsive desde 2026-08-06: escala proporcional como una unidad (no reordena el layout) hasta un ancho mínimo de contenedor; ver `DESIGN.md` para la técnica.
- Todo el banner es clickeable (un solo `<a>` envolviendo la superficie completa, sin zonas muertas), apunta a la sección interna de la mesa de diálogo en LaPatilla.com con UTM propio, abre en pestaña nueva.
- Sin backend ni base de datos: la fecha de inicio se fija en el código/config del widget.
- Undecided: mecanismo exacto de reinicio/actualización de la fecha de inicio si la mesa de diálogo se interrumpe o retoma (a definir cuando sea necesario).

## Brand Commitments

Identidad visual de LaPatilla (rojo CC0A00, negro 111111, tipografía Arial) disponible vía el skill `lapatilla-docx-skill`. Desde 2026-08-06 el rojo de marca sí entró al widget de forma puntual: el dígito "3" en todas las tarjetas se muestra en rojo LaPatilla (#ff0400), el resto de los dígitos queda negro. Logo de LaPatilla también presente en el crédito "Desarrollado por".

## Evidence on Hand

Imagen de referencia de estilo split-flap countdown (stock Adobe, `/Users/rorybranker/Downloads/1000_F_196845844_lmASItTWUpowGg0m6rt5pp8kUX38RQN2.jpg`): 4 tarjetas flip blancas (Days/Hours/Minutes/Seconds), fondo gris azulado claro, título superior, labels inferiores. Se usa solo como referencia de mecanismo visual, no de marca ni copy final.

## Product Principles

1. Fidelidad al mecanismo visual de referencia (flip card de aeropuerto), no una reinterpretación libre.
2. Cero dependencias pesadas: JS vanilla, sin frameworks, liviano para embeber.
3. Legible a la distancia de un banner de sitio: jerarquía clara entre número, label y título.
4. Corre indefinidamente sin intervención (contador que sigue sumando, no expira).
