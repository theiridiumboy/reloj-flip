# Design

<!-- impeccable:design-schema 1 -->

## World

Pizarra split-flap de aeropuerto oscura, fiel a un vector de referencia entregado por el usuario (Freepik "realistic scoreboard alphabet/numbers"). Diseño base 980×200; desde 2026-08-06 escala proporcional a cualquier ancho de contenedor (ver sección "Responsive" más abajo) — ya no requiere un ancho fijo para embeberse.

## Palette

Oscura, fiel al vector de referencia (revisión 2026-08-05; reemplaza la paleta clara de la v1).

- `--bg`: `#2c2d30` — fondo del banner, panel gris carbón (no negro puro: así el marco negro de cada tarjeta contrasta y se lee como pieza física, tal como en el vector).
- `--card-hi` / `--card-lo`: `#313236` / `#1c1d20` — gradiente de bisel por mitad de tarjeta.
- `--frame`: `#050505` — marco exterior grueso de cada tarjeta y banda de la costura (negro real, no gris).
- `--ink`: `#f3f2ee` — dígitos y título, blanco hueso (no blanco puro).
- `--label`: `#9096a1` — labels bajo cada tarjeta.
- `--tab-hi` / `--tab-lo`: `#45474d` / `#0c0d0e` — gradiente de los dos tabs de bisagra.

**Corrección 2026-08-05 (segunda ronda):** la primera versión oscura usaba un pin plateado único cruzando toda la tarjeta — Rory señaló que el vector real no tiene eso: es un marco negro grueso alrededor de toda la tarjeta, una banda negra sólida (no gris) en la costura, y DOS tabs oscuros independientes que sobresalen a los lados izquierdo y derecho (no un pin centrado). Verificado recortando y haciendo zoom al `.jpg` de referencia (`~/Downloads/realistic-scoreboard-alphabet/2068946.jpg`) antes de corregir, en vez de asumir de memoria.

## Type

Arial / Arial Narrow (system stack, decisión intencional: widget "operate", fidelidad a la referencia, evita fuentes AI-sobreusadas: Inter/Roboto/Fraunces/Geist/Plus Jakarta/Space Grotesk). Título en versalitas trackeadas; dígitos bold; labels versalitas trackeadas pequeñas.

## Components

**Tarjeta flip por dígito** (revisión 2026-08-05: antes era una tarjeta por unidad con dígitos juntos; el vector de referencia nuevo trae un carácter por tarjeta, así que se corrigió para calzar). 9 tarjetas en fila: Días = 3 tarjetas, Horas/Minutos/Segundos = 2 tarjetas cada una.

**Modelo actual (revisión 2026-08-05, cuarta ronda): una tarjeta rígida, no dos mitades clipeadas.** Cada dígito tiene solo DOS elementos con texto: `.face` (estático, siempre el valor actual) y `.leaf` (la tarjeta que cae, muestra el valor viejo mientras gira). Ninguno de los dos recorta el glifo por la mitad — cada uno renderiza el número COMPLETO, centrado normal (`display:flex;align-items:center;justify-content:center`, `inset:3px` igual en ambos). La costura (`.seam`) es puramente decorativa: una línea de 4px dibujada ENCIMA (`z-index:6`) de todo, cruzando el dígito, igual que en una pizarra real — no es un límite de recorte. Esto es más simple que el modelo anterior de dos copias con `clip-path` y, de paso, hace estructuralmente imposible el bug de desincronización de la ronda 3 (ya no hay dos mitades que puedan quedar en valores distintos, solo un `.face` y un `.leaf`, un texto cada uno).

Mecanismo de flip: UNA sola pieza rígida (`.leaf`) gira sobre su propio CENTRO (`transform-origin: center`, hinge = la costura), de `rotateX(0deg)` a `rotateX(-90deg)` en una animación (`fallCard`, ~420ms), con `backface-visibility:hidden` para que desaparezca al llegar al canto. Confirmado cuadro por cuadro (`getAnimations()` + `currentTime` scrubbing, no capturas al azar): la tarjeta entera se comprime de forma uniforme como una sola pieza (no dos mitades con timing distinto), igual que una tarjeta física cayendo desde su bisagra central.

**Marco y bisagra (`::before` + `.seam` + `.tab.left`/`.tab.right`):** marco negro grueso (`border-radius:13px; background:var(--frame)` en `.digit`, con `.face`/`.leaf` insertos a `inset:3px` dejando ver el marco alrededor), bevel highlight interior vía `::before` con `inset: 3px` y `box-shadow: inset`, banda de costura negra sólida (`.seam`, `height:4px`) y dos tabs independientes (`.tab.left`/`.tab.right`, 15×13px) que sobresalen `-6px` más allá del borde de la tarjeta a cada lado — no un pin único centrado.

## Motion

Una sola animación por tarjeta (`fallCard`) — reemplaza el modelo anterior de dos hojas independientes (`.leaf.top-half`/`.leaf.bottom-half` con timing distinto), que Rory señaló correctamente como "no física real, caen 3 tarjetas en vez de 1". Cada tarjeta lleva además `filter: drop-shadow(...)` fija para la sombra de caída bajo la tarjeta en reposo. Respeta `prefers-reduced-motion: reduce` (sin animación, cambio instantáneo).

**Corrección 2026-08-05 (quinta ronda): de "flip digital" a física analógica real.** Rory señaló que aunque ya era una sola tarjeta, el movimiento se sentía como "un flip digital que hace aparecer el número", no como un reloj analógico. La curva anterior (`cubic-bezier(0.55,0,0.85,0.35)`, 420ms) era una desaceleración suave y pareja — matemáticamente correcta pero perceptualmente sintética, porque un mecanismo real no frena así, se detiene en seco contra un tope. Tres cambios:

1. **Anticipación:** la tarjeta ahora se "levanta" 3° antes de caer (`0%→12%: rotateX(3deg)→rotateX(0deg)`), imitando el instante en que el mecanismo suelta el gancho antes de que la gravedad tome el control. Sin esto, el ojo lee la animación como un efecto de UI, no como un objeto con inercia.
2. **Caída acelerada, no suavizada:** de `12%` a `100%` la curva es `cubic-bezier(0.7,0,1,1)` (ease-in fuerte, sin ease-out) — la tarjeta acelera todo el trayecto y se detiene abruptamente al llegar a `-90deg`, en vez de desacelerar suavemente como hacía antes. Verificado cuadro por cuadro (`getAnimations()`/`currentTime`): a los 100ms ya muestra compresión fuerte, a los 180ms casi al canto — mucho más rápido que la progresión pareja de la versión anterior.
3. **Duración más corta:** 420ms → 320ms, más parecido al golpe seco de una hoja física que a un glide de interfaz.

`animation-timing-function` se define POR KEYFRAME (soportado en CSS `@keyframes`) en vez de una sola curva para toda la animación, para poder darle a cada tramo (anticipación vs caída) su propio carácter.

**Tamaño (revisión 2026-08-05, cuarta ronda):** tarjeta 68×112px → 76×128px, `font-size` 60px → 76px. Rory pidió números más grandes porque sentía que "la bisagra borra el número" — en realidad la costura siempre fue decorativa (nunca recortaba nada); el problema real era que a 60px el glifo se veía chico dentro de la tarjeta. A 76px se lee con más autoridad sin que la costura estorbe. Verificado que el banner de 200px de alto sigue sin recortarse (~11px de margen vertical libre, ajustado).

## Behavior

Cuenta hacia arriba desde el momento en que el script carga (`Date.now()` al iniciar) — no desde una fecha fija de la mesa de diálogo real. **Pendiente**: fijar `START` a la fecha/hora real de arranque de la mesa cuando se conozca (marcado `ponytail:` en el código, línea del `var START`).

**Cascada de arranque (2026-08-05, sexta ronda):** a pedido de Rory, cuando alguien carga la página, las 9 tarjetas NO aparecen directo en su valor real — parten todas en "0" y cada una gira por una ráfaga rápida de valores (`SPIN_STEPS=6`, `SPIN_STEP_MS=100`, dígitos aleatorios vía `randomDigitExcept`) antes de aterrizar exacta en el dígito real, con un desfase de `CASCADE_STAGGER_MS=70ms` entre tarjeta y tarjeta (izquierda→derecha: días→horas→minutos→segundos), dando el efecto de cascada. Es puramente decorativo — los valores intermedios NO son un conteo literal (días podría ser un número grande si la mesa lleva mucho corriendo; contar dígito por dígito sería lentísimo), es solo movimiento antes de aterrizar en el valor correcto.

La velocidad del flip es ajustable vía la custom property `--fall-duration` en `.digit` (default `0.32s`, el `setDigit(el, value, durationMs)` la sobreescribe inline para la ráfaga rápida del intro, y se limpia con `style.removeProperty` al terminar la cascada para que el tick normal de cada segundo vuelva a la velocidad mecánica de siempre). Al terminar la cascada (tiempo total calculado, no un valor fijo a ojo) recién arranca `setInterval(tick, 1000)`.

`prefers-reduced-motion: reduce` salta la cascada por completo (no solo desactiva cada animación individual): construye las tarjetas directo en su valor real y arranca el tick normal de inmediato, porque una sucesión rápida de cambios de valor es en sí mismo un tipo de movimiento que alguien con esa preferencia probablemente no quiere ver, más allá de que cada flip individual ya esté desactivado por CSS.

Verificado en navegador real (no solo lectura de código): recarga en frío, la cascada aterriza exactamente en el tiempo transcurrido real (ej. `000000021` = 0d 0h 0m 21s, coincidiendo con el tiempo real desde la carga), `--fall-duration` vuelve a `0.32s` tras la cascada, y el tick normal post-intro sigue sin desincronías (mismo chequeo de `face`/`leaf` de rondas anteriores).

## Accessibility

`.board` (visual) marcado `aria-hidden="true"`; un `<p class="sr-only" aria-live="polite">` paralelo expone un resumen en texto plano ("Mesa de diálogo activa desde hace X días..."), actualizado una vez por minuto (no por segundo, para no saturar lectores de pantalla).

## Crédito "Desarrollado por" (2026-08-05, séptima ronda)

Logo de LaPatilla (imagotipo: patilla + wordmark) descargado desde Drive (`1TwBeHs8-JZf4BaWrnrj-Q5AMTsl5Rah1`, `Copia de Imagotipo.png`), recortado a su bounding box real (el archivo original es un cuadrado 2550×2550 con muchísimo relleno transparente alrededor) y reescalado a 434×96px. Fuente guardada en el proyecto como `lp-logo.png` (original) y `lp-logo-crop.png` (recortado, el que se usa). Embebido como `data:image/png;base64` directo en el `<img>` del HTML — no una referencia a archivo externo — para que el widget siga siendo un único archivo autocontenido, embebible tal cual en LaPatilla.com sin depender de subir un asset aparte.

**El wordmark del logo es gris carbón (~`#3f3f3f`), diseñado para fondo claro.** Puesto directo sobre el banner oscuro (`--bg: #2c2d30`) el contraste falla groseramente. Solución inicial: una píldora blanca (`.credit-badge`, `background:#fff; border-radius:999px`) detrás del logo, no un capricho estético — era lo que lo hacía legible.

**Layout (corregido en la misma ronda tras choque real):** el primer intento puso el crédito en fila horizontal (label + badge) pegado a `bottom:12px; right:16px`, y chocó visualmente con el label "SEGUNDOS" — medido con `getBoundingClientRect()`: el `.board` (tarjetas) solo usa 792 de los 980px del banner, centrado, dejando apenas 94px libres a la derecha del último grupo de tarjetas, no abajo. Una fila horizontal no entra ahí a los tamaños originales. Fix intermedio: `.credit` pasó a `flex-direction: column; align-items: flex-end` (texto arriba, badge abajo, apilado y angosto).

## Corrección 2026-08-06 (octava ronda): una sola línea, sin píldora blanca

Rory pidió volver a una sola línea y quitar el fondo blanco del logo. En vez de mantener la píldora (necesaria solo porque el wordmark original es oscuro), se generó `lp-logo-light.png`: un pase con PIL sobre `lp-logo-crop.png` que recolorea todo píxel casi-gris (`max(r,g,b)-min(r,g,b) < 24`, lo que cubre el wordmark "lapatilla" y sus bordes con antialiasing) a un ink claro (`rgb(243,242,238)`), preservando el canal alpha original (así el antialiasing no se pierde) y sin tocar los píxeles NO grises (rojo/verde de la sandía). Verificado con un escaneo completo del archivo (no muestreo): cero píxeles grises remanentes fuera del color objetivo, colores de la sandía intactos. Con el wordmark ahora claro, la píldora blanca ya no hace falta — se eliminó.

**Trade-off de espacio, decidido explícitamente por Rory:** con el margen real de 94px, un crédito en una sola línea a tamaño cómodamente legible (8-9px texto + 9-10px logo) no entra — solo entra si se achica el `gap` de 26px entre los grupos Días/Horas/Minutos/Segundos del reloj principal para ganar margen extra. Se le presentó la opción a Rory (achicar el reloj vs. crédito chico) y eligió expresamente NO tocar el espaciado del reloj. Resultado: `.credit` en fila (`flex-direction: row`), `right: 5px`, texto `font-size: 7px` (cerca del piso de legibilidad, aceptado a propósito) y logo `height: 9px` (~41px de ancho). Verificado con `getBoundingClientRect()`: el bloque completo mide 91.9px de ancho, cabe dentro del margen sin tocar ni las tarjetas ni el label "SEGUNDOS" (que sigue vertical, pero horizontalmente separado, sin colisión).

## Bug real corregido (2026-08-05, tercera ronda): mitad superior desincronizada

Rory reportó "la parte superior no coincide con la inferior". Verificado empíricamente (no por inspección visual): en reposo, `.leaf.top-half` es la capa que realmente se ve (está encima de `.face.top-half`, opaca, `rotateX(0)` en estado idle) — pero el código solo actualizaba el texto de `.face.top-half` al terminar el flip (`setTimeout` de 520ms), nunca el de `.leaf.top-half`. Resultado: la mitad superior de cada tarjeta quedaba **permanentemente un valor atrás** de la inferior (medido: `faceTop=6` mientras `leafTop=5`, en cada tick). La mitad inferior sí estaba bien sincronizada.

Fix en su momento: `snapDigit()` actualizaba las 4 capas (`face.top`, `leaf.top`, `face.bottom`, `leaf.bottom`) al mismo valor al asentarse, no solo `face.top`. (Nota: esas 4 capas y sus nombres `.top-half`/`.bottom-half` ya no existen — la cuarta ronda, más abajo, las reemplazó por solo `.face`/`.leaf` sin split. La guarda contra doble-flip descrita aquí sí se mantiene, adaptada a `snapDigit(el, value)` con las 2 capas actuales.)

## Responsive (2026-08-06, novena ronda)

Rory preguntó por qué el widget no era responsive; se le explicó que fue decisión de producto explícita (`PRODUCT.md`), no un descuido — 980×200 fijo, cero `@media` queries, todo en `px`. Al preguntar si volverlo responsive arriesgaba romper el trabajo ya afinado (animación de flip, cascada, seam decorativo, crédito — todo verificado en vivo esta sesión), se le ofrecieron dos caminos con su trade-off honesto: **(a)** escalar todo el banner proporcional, como una foto — cero riesgo a lo interno, pero tarjetas chicas en celular (~29px); **(b)** reordenar el layout en pantallas chicas para tarjetas más grandes — mejor legibilidad mobile, pero más superficie de cambio. **Rory eligió (a) expresamente.**

**Técnica: `transform: scale()` vía CSS Container Query units, no media queries con reflow.** `.banner` mantiene sus `width:980px; height:200px` fijos exactamente como estaban — ni un valor interno (fuente, gaps, seam, `--fall-duration`) se tocó o recalculó. Se envolvió en un nuevo `.banner-wrap` con `container-type: inline-size; width:100%; max-width:980px; aspect-ratio:980/200`, y se le agregaron a `.banner` solo dos líneas: `transform: scale(calc(100cqw / 980px)); transform-origin: top left;`. `cqw` = 1% del ancho real del `.banner-wrap`; dividir una longitud (`100cqw`) entre otra (`980px`) en `calc()` da un número unitless, exactamente lo que `scale()` necesita — matemática CSS válida, no un truco. El resultado: `.banner` se renderiza y anima idéntico a como estaba, solo visualmente escalado como bloque rígido; imposible que se desincronice nada porque nada interno cambió.

`html,body` pasó de `width:980px; height:200px; overflow:hidden` (fijo) a `width:100%` (fluido) — ya no fuerza que la página entera sea del tamaño del banner.

**Verificado en navegador real, no solo lectura de código:** anchos de 1200px (sin escalar, banner centrado a 980px), 600px (escala a 600×122.44, calza exacto con el aspect-ratio 980:200), 375px y 320px (sin overflow horizontal en ningún caso, confirmado con `document.documentElement.scrollWidth`). Un flip forzado a 375px con `getAnimations()`/`currentTime` scrubbing (misma técnica de toda la sesión) confirmó que la animación se ve proporcional y sin artefactos a escala chica.

## Copy (2026-08-06, décima ronda)

Título cambiado de "Mesa de Diálogo en Venezuela — Tiempo en curso" a "La mesa de diálogo en Venezuela inició hace:" (Rory pidió el texto exacto sin tilde en "inicio"; se corrigió a "inició" porque es verbo conjugado, no sustantivo — la frase sin tilde no cierra bien en español). Aplicado tanto en `index.html` como en `experiments/flip-v2-realista.html` por ser corrección de copy, no parte del experimento visual. Verificado que cabe sin desbordar a 980px.

## Alineación (2026-08-06, onceava ronda)

Título pasó de centrado a alineado a la izquierda, calzando con el borde izquierdo real de la primera tarjeta (`margin-left: 94px`, el mismo inset que separa `.board` del borde del banner de 980px) en vez de pegarlo al borde crudo del banner (`x:0`). `align-self: flex-start` en `.title` dentro del `.banner` (que sigue siendo `flex-direction:column; align-items:center` para el resto de los hijos). Aplicado en ambos archivos.

## Promoción a producción + banner clickeable (2026-08-06, doceava ronda)

Rory pidió hacer oficial el experimento (`experiments/flip-v2-realista.html`) — el que ganó fue: tarjeta de dos caras girando 180°, color cartulina, dígito "3" en rojo LaPatilla. `index.html` original (una sola cara, paleta oscura) se guardó como `index.html.bak-v1` por si hace falta volver atrás; `experiments/flip-v2-realista.html` queda intacto como registro histórico del experimento, ya no se sincroniza hacia adelante.

**Banner completo clickeable:** toda la superficie (980×200, incluido el crédito) envuelta en un solo `<a class="banner-link">` que rodea a `.banner-wrap` — sin zonas muertas, sin elementos interactivos anidados adentro que compitan por el click. `max-width:980px` y el centrado (`margin:0 auto`) se movieron del `.banner-wrap` al `<a>` para que el link mismo defina el área clickeable real, no solo lo que hay debajo visualmente. Apunta a `https://lapatilla.com/secciones/nacionales/mesa-negociacion/` con UTM (`utm_source=newsletter&utm_medium=email&utm_campaign=Buenosdias-Whatsapp&utm_content=cuerpo`), `target="_blank" rel="noopener noreferrer"` (no saca al lector del artículo donde está embebido), `aria-label` describiendo el destino para lectores de pantalla (el board visual sigue `aria-hidden`). Affordance sutil: `filter:brightness(1.03)` en hover, contorno rojo en `:focus-visible` para navegación por teclado.

Verificado en navegador real: `getBoundingClientRect()` del `<a>` calza exacto con `.banner-wrap` (cobertura 100%, no parcial), `href`/`target`/`rel`/`aria-label` correctos, y el mecanismo de flip (incluida la coloración roja del "3") sigue funcionando igual que en el experimento original tras la promoción.

## Open decisions

- Fecha real de inicio del conteo (actualmente = momento de carga del script).
- Si la mesa de diálogo se interrumpe/retoma, no hay mecanismo de pausa o reinicio; se definirá cuando aplique.
- Vector original (Freepik, `.ai`/`.eps` en `~/Downloads/realistic-scoreboard-alphabet/` y `.../realistic-scoreboard-style-alphabet/`) no se pudo convertir a SVG editable (sin Inkscape/Ghostscript instalados); el estilo se recreó a mano en CSS. Si se necesita fidelidad pixel-perfect al vector, instalar Inkscape y reimportar.
- En celular (ancho de contenedor ~375px), las 9 tarjetas quedan legibles pero chicas (~29px cada una) y el crédito "Desarrollado por" (ya al límite de legibilidad a tamaño completo) se vuelve prácticamente invisible. Aceptado explícitamente por Rory al elegir escalado proporcional sobre reflow — si en el futuro se prioriza legibilidad mobile por sobre "cero riesgo", ahí está la opción (b) descrita arriba, no implementada.
