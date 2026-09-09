---
name: appscript-webapp-builder
description: Crea, corrige y extiende aplicaciones web (dashboards, sistemas CRUD, formularios, administración) con Google Apps Script sobre Google Sheets — incluyendo crear desde cero la hoja de datos base si el usuario aún no tiene ninguna. Úsala siempre que pidan crear una "web app", "aplicación", "dashboard", "sistema" o "automatización" conectada a Sheets con Apps Script, sin importar el dominio (inventarios, clínicas, ventas, asistencia, tareas, reservas, etc.); cuando pidan agregar, corregir o modificar un proyecto de Apps Script ya existente (alertas, estilo, campos nuevos, errores de despliegue); cuando reporten errores típicos como "No se ha encontrado el archivo HTML", "No existe la hoja X", o cambios que no se reflejan en la URL publicada; o cuando pidan armar una base de datos/inventario en Excel o Sheets pensando en conectarla luego a una app. Aplica también si mencionan "Code.gs", "google.script.run", "doGet", o piden una plantilla/prompt para este tipo de proyectos.
---

# Generador de web apps con Google Apps Script

Skill genérica para construir aplicaciones web completas (backend + frontend) sobre Google Sheets usando Google Apps Script, y para iterar sobre ellas con correcciones y nuevas funcionalidades sin romper lo que ya existe.

No asume un dominio fijo (no es solo para inventarios): sirve igual para un CRM simple, control de asistencia, gestión de citas, seguimiento de tareas, registro de pacientes, etc. La estructura de datos (columnas) y el estilo visual se adaptan cada vez según lo que diga el usuario.

## Cuándo estás creando un proyecto nuevo vs. iterando sobre uno existente

Antes de escribir código, identifica en cuál de estos dos casos estás:

- **Proyecto nuevo**: el usuario no tiene código de Apps Script todavía, o quiere empezar de cero.
- **Iteración sobre un proyecto existente**: el usuario ya tiene los 4 archivos (de esta skill o de una conversación anterior) y pide agregar/corregir algo puntual. En este caso, **nunca regeneres los 4 archivos desde cero** — identifica qué archivo(s) hay que tocar y da instrucciones de reemplazo puntual (tipo parche), preservando nombres de funciones, IDs de HTML y estructura ya existentes. Esto evita romper cosas que ya funcionan y hace que el usuario pueda copiar/pegar solo lo que cambió.

---

## Paso 0 — ¿Ya existe la hoja de datos?

Antes de la entrevista del Paso 1, determina si el usuario ya tiene la hoja de Google Sheets con sus datos, o si hay que crearla:

- **Si subió un archivo** (`.xlsx` u otro) con los datos o una estructura de columnas: léelo directamente y usa esas columnas reales — no vuelvas a preguntarlas. Si el archivo es `.xlsx`, recuérdale que para que Apps Script lo pueda leer nativamente necesita estar como Google Sheets (Archivo → Guardar como Hojas de cálculo de Google, o subirlo a Drive y abrirlo con Sheets), no como archivo Excel suelto.
- **Si no tiene ninguna hoja todavía**: pregúntale qué información necesita registrar y arma tú la estructura de columnas (nombre, tipo de dato, si es una lista de estados fija, etc.), confirma con el usuario, y **genera la hoja inicial como archivo `.xlsx` descargable** siguiendo las convenciones de la skill de xlsx del sistema (`/mnt/skills/public/xlsx/SKILL.md`): encabezados con formato, ancho de columna automático, validación de datos (lista desplegable) en columnas tipo estado, primera fila congelada, y unas filas de ejemplo con datos ficticios para que el usuario vea la estructura funcionando. Explícale que debe subir ese archivo a Google Drive y convertirlo a Google Sheets antes del Paso 2 (Apps Script no puede leer un `.xlsx` suelto, necesita que sea nativamente una Hoja de cálculo de Google).
- **Si hay un conector de Google Drive/Sheets disponible en la conversación**: puedes ofrecer crear la hoja directamente ahí en vez de generar un `.xlsx` para subir manualmente — pregúntale al usuario si lo prefiere así.

En cualquiera de los tres casos, al terminar este paso ya debes tener: el nombre exacto de la pestaña de datos y la lista de columnas en orden — eso es lo que alimenta el Paso 1 y el resto del flujo.

---

## Paso 1 — Entrevista rápida (solo para proyectos nuevos)

Antes de generar código, confirma con el usuario (una sola tanda de preguntas, no una por una):

1. **Nombre exacto de la pestaña (tab)** de Google Sheets donde están los datos — no el nombre del archivo. Recuérdale que el nombre del archivo (arriba) y el nombre de la pestaña (abajo) pueden ser distintos, y que lo que necesita el código es el de la pestaña.
2. **Columnas exactas**, en orden, con su tipo (texto, número, fecha, lista de valores fijos como un "Estado").
3. **Qué debe poder hacer la app**: ¿solo ver datos (dashboard de lectura) o también crear/editar/eliminar (CRUD completo)? ¿Necesita filtros, búsqueda, KPIs, alguna alerta (ej. stock bajo, fecha próxima a vencer, etc.)?
4. **Estilo visual** — pregúntalo siempre, no asumas un estilo por defecto. Opciones típicas para ofrecer: Apple (glassmorphism, minimalista, modo claro/oscuro), minimalista plano, corporativo/serio, colorido/juguetón, oscuro tipo "panel técnico". Si el usuario no tiene preferencia, usa el estilo Apple del ejemplo de `references/estilo-apple.md` como default razonable, pero dilo explícitamente en vez de asumirlo en silencio.

Con eso, sigue al Paso 2.

---

## Paso 2 — Arquitectura estándar (siempre igual, cambia el contenido no la forma)

**Punto de partida obligatorio:** antes de escribir código desde cero, lee las 4 plantillas en `assets/` (`Code.gs.template`, `Index.html.template`, `Stylesheet.html.template`, `JS.html.template`). Ya traen el esqueleto completo (CRUD, KPIs, modales, toasts, tema claro/oscuro, validaciones, lock de escritura) con comentarios `AJUSTAR:` marcando exactamente qué reemplazar según las columnas y el dominio del usuario. Es más rápido y más consistente partir de ahí y rellenar los placeholders que escribir todo de nuevo cada vez. `Stylesheet.html.template` ya viene con la variante Apple aplicada por defecto — si el usuario eligió otro estilo, sobrescribe solo las variables CSS según `references/estilos-visuales.md`, sin tocar los nombres de clase.

Genera siempre estos 4 archivos, con estos nombres exactos (son sensibles a mayúsculas/minúsculas):

- **`Code.gs`** — backend.
- **`Index.html`** — estructura HTML, incluye a los otros dos con `<?!= include('Stylesheet'); ?>` y `<?!= include('JS'); ?>`.
- **`Stylesheet.html`** — todo el CSS.
- **`JS.html`** — todo el JavaScript de cliente.

Usa nombres cortos y sin ambigüedad para los includes (`JS`, no `JavaScript` — Apps Script ha dado problemas reales cuando el nombre del archivo y el string pasado a `include()` no coinciden letra por letra).

### Backend (`Code.gs`) — patrón obligatorio

```javascript
const SHEET_NAME = "NOMBRE_EXACTO_DE_LA_PESTAÑA";
const LOG_SHEET_NAME = "Log";

function doGet(e) {
  const template = HtmlService.createTemplateFromFile("Index");
  return template.evaluate()
    .setTitle("TÍTULO DE LA APP")
    .addMetaTag("viewport", "width=device-width, initial-scale=1")
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}

function include(filename) {
  return HtmlService.createHtmlOutputFromFile(filename).getContent();
}
```

Reglas para el resto del backend:
- Toda función pública (llamada desde `google.script.run`) devuelve `{ success: true, data: ... }` o `{ success: false, error: "mensaje" }`. Nunca dejes que una excepción se propague sin capturar.
- Las operaciones de escritura (crear/editar/eliminar) usan `LockService.getScriptLock()` con `waitLock()` y `releaseLock()` en un `finally`.
- Los códigos/IDs se autogeneran revisando los existentes para no duplicar (patrón `PREFIJO-0001`, incrementando hasta encontrar uno libre).
- Cada creación/edición/eliminación se registra en una hoja oculta `Log` (fecha, usuario vía `Session.getActiveUser().getEmail()`, acción, id, detalle). Créala sola la primera vez si no existe.
- Incluye una función `obtenerEstadisticas()` que calcule KPIs relevantes al dominio (totales, conteos por estado/categoría, alertas de umbral) — el frontend nunca debe calcular agregados que el backend ya puede dar.
- Si el usuario pide una alerta basada en un umbral (stock bajo, fecha próxima, etc.), sigue el patrón de `references/patron-alertas.md`.

### Frontend — patrón obligatorio

- **KPIs arriba** en tarjetas.
- **Buscador con debounce** + **filtros por select** (poblados dinámicamente desde los datos reales, no hardcodeados).
- **Tabla principal** con acciones de editar/eliminar por fila.
- **Modal de crear/editar** con formulario y validación básica en cliente antes de llamar al backend.
- **Modal de confirmación** antes de eliminar (nunca eliminar directo desde el botón).
- **Toasts** para éxito/error de cada operación.
- **Loader/spinner global** mientras hay una llamada a `google.script.run` en curso.
- **Modo claro/oscuro** con variables CSS (`:root` / `[data-theme="dark"]`) guardado en `localStorage`, aplicado antes de pintar contenido si es posible.
- Toda llamada al backend usa `.withSuccessHandler().withFailureHandler()`, nunca se asume que la llamada va a funcionar.
- Responsive: al menos un breakpoint (~900px) que colapse KPIs a 2 columnas y el formulario a 1 columna.

Para el detalle visual según el estilo elegido en el Paso 1, consulta `references/estilos-visuales.md` (ahí están las variables CSS y convenciones para Apple, minimalista, corporativo, colorido y oscuro/técnico).

---

## Paso 3 — Iterar sobre un proyecto existente

Cuando el usuario pida agregar o corregir algo sobre una app que ya existe (en esta conversación o pegando su código actual):

1. Pide (o localiza en la conversación) el contenido actual de los 4 archivos si no los tienes ya.
2. Identifica el **mínimo conjunto de archivos** que hay que tocar. La mayoría de pedidos caen en un patrón conocido:
   - "Agrega una alerta de X" → `Code.gs` (cálculo + función de alerta) + `Index.html` (contenedor del banner) + `Stylesheet.html` (estilos del banner) + `JS.html` (mostrar/ocultar). Ver `references/patron-alertas.md`.
   - "Cambia el estilo / hazlo más X" → solo `Stylesheet.html`, normalmente. Cambia variables CSS en `:root`, no reescribas toda la hoja de estilos si no es necesario.
   - "Agrega un campo nuevo" → `Code.gs` (columna en `_rowToObject`, validación, alta/edición), `Index.html` (input nuevo en el form y columna en la tabla), `JS.html` (leer/escribir el campo nuevo).
   - "Agrega un reporte/exportar" → función nueva en `Code.gs` que use `SpreadsheetApp` o `DriveApp`, más un botón en `Index.html`.
3. Da el cambio como un reemplazo puntual y localizable ("busca esta línea/bloque y reemplázalo por esto"), no el archivo completo, salvo que el usuario lo pida explícitamente o el cambio toque la mayoría del archivo.
4. Recuérdale siempre, después de cualquier cambio: **hay que reimplementar una nueva versión** (Implementar → Administrar implementaciones → ✏️ → Nueva versión) para que la URL `.exec` refleje el cambio. Guardar (Ctrl+S) no es suficiente.

---

## Paso 4 — Guía de despliegue (dar siempre en un proyecto nuevo)

1. Abrir el proyecto desde **Extensiones → Apps Script dentro del Google Sheet** (no desde script.google.com aparte) para que quede vinculado (`SpreadsheetApp.getActiveSpreadsheet()` necesita esto).
2. Crear los 4 archivos con los nombres exactos indicados arriba.
3. Guardar todo.
4. Implementar → Nueva implementación → tipo "Aplicación web" → Ejecutar como "Yo" → Acceso según lo que necesite el usuario.
5. Cada cambio posterior exige una **nueva versión** de la implementación, no solo guardar.
6. Probar en pestaña de incógnito para evitar caché.

---

## Errores comunes — usa esta tabla para diagnosticar

| Mensaje de error | Causa casi segura | Qué revisar |
|---|---|---|
| `No se ha encontrado el archivo HTML denominado X` | El nombre en `include('X')` no es idéntico al nombre real del archivo | Compara letra por letra, mayúsculas y espacios; si hay duda, renombra ambos a algo corto como `JS` |
| `No existe la hoja "X"` | El nombre de la pestaña (tab) no coincide con `SHEET_NAME`, o se está confundiendo con el nombre del archivo completo | Pide al usuario el nombre exacto de la pestaña de abajo, no el título del archivo |
| Los cambios no aparecen en la URL | Se guardó pero no se reimplementó una nueva versión | Implementar → Administrar implementaciones → Nueva versión |
| Pide autorización al ejecutar | Primera ejecución de una función que usa servicios con permisos (Mail, Sheets, etc.) | Revisar permisos → cuenta → Avanzado → Ir al proyecto (no seguro) → Permitir |
| El script no encuentra ninguna hoja "activa" | El proyecto se abrió desde script.google.com en vez de desde el Sheet | Recrear el proyecto desde Extensiones → Apps Script dentro del Sheet, o usar `SpreadsheetApp.openById(ID)` explícito |

Función de diagnóstico rápida para pegar y ejecutar manualmente cuando algo no cuadra:

```javascript
function diagnostico() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  Logger.log('Archivo vinculado: ' + ss.getName());
  const nombres = ss.getSheets().map(function(s){ return '[' + s.getName() + ']'; });
  Logger.log('Pestañas encontradas: ' + nombres.join(', '));
}
```

---

## Paso 5 — Funcionalidades avanzadas (solo si aplica)

Si el usuario pide algo que va más allá del CRUD básico — exportar a CSV/PDF, gráficos, paginación para muchos datos, datos relacionados en varias hojas, roles/permisos, carga masiva, historial por registro, búsqueda avanzada con varios criterios, o duplicar un registro — consulta `references/patrones-avanzados.md` antes de improvisar. Cada patrón ahí incluye el código base y las advertencias de cuándo NO conviene (ej. no cargar miles de filas de una vez, no hacer permisos solo en el frontend). No agregues estas funcionalidades por iniciativa propia si el usuario no las pidió ni las necesita claramente — cada una suma complejidad.

---

## Archivos de este skill

**Plantillas (usar como punto de partida en proyectos nuevos):**
- `assets/Code.gs.template` — backend completo con placeholders `{{...}}` y comentarios `AJUSTAR:`.
- `assets/Index.html.template` — estructura HTML completa con placeholders.
- `assets/Stylesheet.html.template` — CSS completo, variante Apple por defecto.
- `assets/JS.html.template` — JavaScript de cliente completo con placeholders.

**Referencias (consultar según necesidad):**
- `references/estilos-visuales.md` — variables CSS y convenciones para cada estilo visual (Apple, minimalista, corporativo, colorido, oscuro/técnico). Léelo después de saber qué estilo eligió el usuario.
- `references/patron-alertas.md` — cómo agregar una alerta basada en umbral (visual + correo opcional con disparador de tiempo), generalizado más allá de "stock bajo".
- `references/patrones-avanzados.md` — exportar CSV/PDF, gráficos, paginación, multi-hoja, roles y permisos, carga masiva, historial por registro, búsqueda avanzada, duplicar registro.
