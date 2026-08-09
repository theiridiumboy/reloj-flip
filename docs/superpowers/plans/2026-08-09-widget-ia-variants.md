# Widget IA — Archivar el reloj y crear el catálogo de variants Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Sentar la base del mecanismo Widget IA — archivar el reloj actual como el primer variant nombrado, crear el catálogo de variants, y documentar el flujo de swap — sin tocar el contenido que está en vivo hoy.

**Architecture:** Trabajo puramente de archivos dentro del repo `theiridiumboy/reloj-flip`. Ninguna lógica nueva ni cambio de comportamiento del widget: `index.html` no cambia de contenido en ningún paso de este plan. Se agrega la carpeta `variants/` (un archivo autocontenido por idea) y se documenta el flujo en `README.md` y `variants/README.md`.

**Tech Stack:** Bash (`cp`, `diff`, `curl`, `grep`), Markdown, Git.

## Global Constraints

- `index.html` es siempre un espejo exacto del variant actualmente en vivo — ningún paso de este plan lo deja distinto de como está hoy.
- Cada variant es un archivo autocontenido en `variants/<slug>.html`, sin dependencias externas, diseñado para 980×200 fijo.
- Los variants nunca se borran al reemplazarse — quedan archivados como rollback instantáneo.
- Todo swap futuro se verifica con `gstack browse` en local antes de copiar a `index.html`, y con `curl` a la URL de Pages después del push, para confirmar que lo servido no es una respuesta cacheada.
- No se le pide nada al webmaster en ningún paso de este plan — el iframe en WordPress no cambia.

---

### Task 1: Archivar el reloj actual como el primer variant

**Files:**
- Create: `variants/reloj-mesa-dialogo.html`

**Interfaces:**
- Consumes: contenido actual de `index.html` en el repo (se copia tal cual, no se reescribe).
- Produces: la convención de nombre `variants/reloj-mesa-dialogo.html` que las Tasks 2 y 3 referencian.

- [ ] **Step 1: Crear la carpeta y copiar el archivo**

```bash
mkdir -p variants
cp index.html variants/reloj-mesa-dialogo.html
```

- [ ] **Step 2: Verificar que la copia es idéntica byte a byte**

Run: `diff index.html variants/reloj-mesa-dialogo.html`
Expected: sin salida (ningún diff) — confirma que el variant archivado es exactamente lo que está en vivo hoy.

- [ ] **Step 3: Commit**

```bash
git add variants/reloj-mesa-dialogo.html
git commit -m "Archivar el reloj como variants/reloj-mesa-dialogo.html"
```

---

### Task 2: Crear el catálogo de variants

**Files:**
- Create: `variants/README.md`

**Interfaces:**
- Consumes: nombre de archivo producido en Task 1 (`reloj-mesa-dialogo.html`).
- Produces: formato de tabla (columnas Archivo | Estado | Publicado/Archivado | Descripción) que futuros variants deben seguir.

- [ ] **Step 1: Crear `variants/README.md`**

```markdown
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
```

- [ ] **Step 2: Verificar que el archivo se creó con la tabla correcta**

Run: `grep -q "reloj-mesa-dialogo.html" variants/README.md && echo OK`
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add variants/README.md
git commit -m "Agregar catálogo de variants (variants/README.md)"
```

---

### Task 3: Documentar el mecanismo en el README raíz

**Files:**
- Modify: `README.md` (secciones "Cómo hacer cambios" y "Estructura del repo")

**Interfaces:**
- Consumes: nada nuevo de tasks anteriores más allá de la existencia de `variants/`.
- Produces: nada que otras tasks consuman — último eslabón de documentación.

- [ ] **Step 1: Reemplazar la sección "## Cómo hacer cambios" por lo siguiente**

Buscar el heading `## Cómo hacer cambios` en `README.md` y reemplazar esa sección completa (heading + contenido hasta el siguiente `##`) por:

```markdown
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
```

- [ ] **Step 2: En la sección "## Estructura del repo", agregar una línea nueva después de la línea de `experiments/`**

```markdown
- `variants/` — catálogo de ideas archivadas para el Widget IA (ver `variants/README.md`). `index.html` siempre es un espejo exacto del variant marcado "En vivo" ahí.
```

- [ ] **Step 3: Verificar**

Run: `grep -q "Widget IA" README.md && grep -q "variants/" README.md && echo OK`
Expected: `OK`

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "Documentar el mecanismo Widget IA en el README"
```

---

### Task 4: Push y verificación de producción

**Files:** ninguno nuevo — push de los commits de Tasks 1-3.

**Interfaces:**
- Consumes: los 3 commits anteriores.
- Produces: nada — task de verificación final.

- [ ] **Step 1: Confirmar que `index.html` no quedó modificado en el working tree**

Run: `git status --short index.html`
Expected: sin salida — `index.html` no aparece como modificado (ninguna task de este plan lo tocó).

- [ ] **Step 2: Push**

```bash
git push
```

- [ ] **Step 3: Esperar el redeploy de GitHub Pages y verificar que producción no cambió**

Run:
```bash
sleep 90
curl -s https://theiridiumboy.github.io/reloj-flip/ | diff - index.html
```
Expected: sin salida — confirma que lo que ve el lector en LaPatilla.com es idéntico a `index.html` local, y que el push no rompió nada en vivo.

- [ ] **Step 4: Avisar que el mecanismo está listo**

Reportar a Rory: `variants/reloj-mesa-dialogo.html` archivado, catálogo (`variants/README.md`) y documentación (`README.md`) en línea, el reloj sigue en vivo sin cambios. Listo para la primera idea nueva.
