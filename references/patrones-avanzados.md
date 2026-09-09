# Patrones avanzados

Funcionalidades más allá del CRUD básico. Aplícalas solo si el usuario las pide o si claramente las necesita según lo que describe — no las agregues todas por defecto, cada una suma complejidad y superficie de error.

---

## 1. Exportar a CSV / PDF

**CSV (simple, nativo):**
```javascript
function exportarCSV() {
  const res = obtenerRegistros();
  if (!res.success) throw new Error(res.error);
  const items = res.data;
  if (!items.length) return "";
  const headers = Object.keys(items[0]).filter((k) => k !== "rowIndex");
  const filas = [headers.join(",")].concat(
    items.map((it) => headers.map((h) => `"${String(it[h] ?? "").replace(/"/g, '""')}"`).join(","))
  );
  return filas.join("\n");
}
```
En el cliente, recibe el string y fuerza la descarga con un `Blob` + `<a download>`, o usa `google.script.run` para escribirlo a Drive con `DriveApp.createFile()` y devolver el link.

**PDF (vía hoja de cálculo → exportación nativa):** genera una hoja temporal formateada con los datos filtrados y usa `DriveApp` + la URL de exportación de Sheets (`/export?format=pdf`) con `UrlFetchApp` para obtener el blob. Es más pesado — solo si el usuario pide específicamente un PDF con formato, no como primera opción.

## 2. Gráficos (Chart.js vía CDN)

En `Index.html`, agrega un `<canvas id="graficoEstado"></canvas>` y en el `<head>` o antes de cerrar `</body>`:
```html
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
```
En `JS.html`, dentro de `renderKPIs(stats)`, construye el gráfico con `stats.porEstado` o `stats.porTipo` (ya vienen como objetos `{clave: conteo}` desde `obtenerEstadisticas`):
```javascript
new Chart(document.getElementById('graficoEstado'), {
  type: 'doughnut',
  data: {
    labels: Object.keys(stats.porEstado),
    datasets: [{ data: Object.values(stats.porEstado) }]
  }
});
```
Si el gráfico se vuelve a pintar en cada carga, guarda la instancia en una variable global y llama `.destroy()` antes de crear una nueva, o Chart.js va a apilar gráficos invisibles encima.

## 3. Paginación / grandes volúmenes de datos

Cuando la hoja tiene cientos o miles de filas, no traigas todo de una vez:
```javascript
function obtenerRegistrosPaginado(pagina, tamanoPagina) {
  const sheet = _getSheet();
  const last = sheet.getLastRow();
  const inicio = 2 + (pagina - 1) * tamanoPagina;
  const cantidad = Math.min(tamanoPagina, Math.max(0, last - inicio + 1));
  if (cantidad <= 0) return { success: true, data: [], total: last - 1 };
  const values = sheet.getRange(inicio, 1, cantidad, sheet.getLastColumn()).getValues();
  return {
    success: true,
    data: values.map((row, i) => _rowToObject(row, inicio + i)),
    total: last - 1,
  };
}
```
En el cliente, agrega controles de "Anterior/Siguiente" y guarda la página actual en una variable. La búsqueda y filtros en este caso conviene hacerlos también en el backend (recibiendo el texto de búsqueda como parámetro) en vez de filtrar solo lo ya cargado, para no perder resultados de páginas no traídas.

## 4. Multi-hoja / datos relacionados

Cuando hay una entidad que depende de otra (ej. "Categorías" separada de "Productos", o "Pacientes" separado de "Citas"):
- Cada hoja tiene su propio `_getSheet()` con su propio nombre constante.
- Las relaciones se resuelven por código/ID, no por nombre (evita romper si renombran algo).
- Al construir el objeto para el frontend, puedes "enriquecer" la fila principal con datos de la hoja relacionada:
  ```javascript
  function obtenerRegistrosConCategoria() {
    const categorias = _getCategoriasMap(); // { idCategoria: nombreCategoria }
    const res = obtenerRegistros();
    if (!res.success) return res;
    res.data.forEach((it) => { it.nombreCategoria = categorias[it.categoriaId] || "Sin categoría"; });
    return res;
  }
  ```
- Para selects poblados desde la hoja relacionada (ej. un dropdown de categorías en el formulario), trae la lista aparte con una función propia (`obtenerCategorias()`) en vez de derivarla de los productos ya cargados — así existen categorías aunque todavía no tengan productos.

## 5. Roles y permisos simples

Cuando no todos los usuarios deben poder crear/editar/eliminar:
```javascript
const CORREOS_ADMIN = ["admin@dominio.com"];

function _esAdmin() {
  return CORREOS_ADMIN.indexOf(Session.getActiveUser().getEmail()) !== -1;
}
```
Llama `_esAdmin()` al inicio de las funciones de escritura y lanza un error claro (`"No tienes permisos para esta acción."`) si no lo es. En el frontend, oculta los botones de crear/editar/eliminar si `obtenerEstadisticas()` (o una función `obtenerPermisos()` dedicada) indica que el usuario no es admin — pero recuerda que ocultar el botón es solo cosmético, la validación real siempre va en el backend.

## 6. Carga masiva (pegar/importar varias filas)

Para permitir pegar datos desde Excel/Sheets directamente en un textarea y crear varios registros de una vez:
```javascript
function agregarRegistrosMasivo(filasTexto) {
  // filasTexto: string con una fila por línea, columnas separadas por tab (pegado directo de Excel)
  const lineas = filasTexto.trim().split("\n");
  const resultados = lineas.map((linea) => {
    const columnas = linea.split("\t");
    // AJUSTAR: mapear columnas a los campos de agregarRegistro
    return agregarRegistro({ nombre: columnas[0], /* ... */ });
  });
  const exitosos = resultados.filter((r) => r.success).length;
  return { success: true, data: { exitosos, total: resultados.length } };
}
```
Ojo: cada llamada a `agregarRegistro` toma y libera el lock por separado — para volúmenes grandes (cientos de filas) es más eficiente escribir todas las filas con un solo `setValues()` sobre un rango, en vez de fila por fila.

## 7. Historial de cambios por registro (no solo log global)

Si además del log general el usuario quiere ver el historial de un registro específico dentro del modal de edición, agrega una columna oculta con un ID estable y filtra la hoja `Log` por ese código al abrir el modal:
```javascript
function obtenerHistorial(codigo) {
  const sheet = _getLogSheet();
  const last = sheet.getLastRow();
  if (last < 2) return { success: true, data: [] };
  const values = sheet.getRange(2, 1, last - 1, 5).getValues();
  const data = values
    .filter((row) => row[3] === codigo)
    .map((row) => ({ fecha: row[0], usuario: row[1], accion: row[2], detalle: row[4] }));
  return { success: true, data };
}
```

## 8. Búsqueda avanzada (múltiples criterios combinados)

Cuando un solo campo de texto no alcanza (ej. rango de fechas + categoría + estado a la vez), pasa un objeto de filtro estructurado en vez de un string:
```javascript
function busquedaAvanzada(filtro) {
  // filtro: { texto, estado, categoria, fechaDesde, fechaHasta }
  let items = obtenerRegistros().data;
  if (filtro.fechaDesde) items = items.filter((it) => new Date(it.fecha) >= new Date(filtro.fechaDesde));
  if (filtro.fechaHasta) items = items.filter((it) => new Date(it.fecha) <= new Date(filtro.fechaHasta));
  // ... resto de condiciones
  return { success: true, data: items };
}
```
En el frontend, esto normalmente se ve como un panel de filtros expandible en vez de un solo buscador.

## 9. Duplicar un registro

Útil cuando muchos registros nuevos se parecen a uno existente:
```javascript
function duplicarRegistro(codigo) {
  const res = obtenerRegistros();
  const original = res.data.find((it) => it.codigo === codigo);
  if (!original) throw new Error("No se encontró el registro a duplicar.");
  const copia = Object.assign({}, original);
  delete copia.codigo;
  delete copia.rowIndex;
  return agregarRegistro(copia);
}
```
En el frontend, un botón extra junto a editar/eliminar que llama esto y luego recarga.

---

## Checklist antes de agregar cualquier patrón avanzado

- [ ] ¿El usuario lo pidió explícitamente, o es claramente necesario por el volumen/complejidad de sus datos? No agregar "por si acaso".
- [ ] ¿El patrón nuevo respeta el contrato `{success, data, error}` del resto del backend?
- [ ] ¿Las operaciones de escritura nuevas también usan `LockService` si tocan la hoja?
- [ ] ¿Se explicó al usuario cualquier paso manual adicional que requiera (permisos nuevos, disparadores, librerías CDN)?
