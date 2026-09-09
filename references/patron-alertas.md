# Patrón de alerta por umbral (visual + correo opcional)

Generalización del patrón usado para "alerta de stock bajo", aplicable a cualquier condición de umbral: fechas próximas a vencer, cantidad mínima, monto máximo, pendientes atrasados, etc.

## 1. Backend (`Code.gs`)

- Agrega una constante para el umbral y, si habrá correo, una para el destinatario:
  ```javascript
  const UMBRAL_ALERTA = 50; // ajustar a la unidad de negocio real
  const EMAIL_ALERTA = "correo@dominio.com";
  ```
- Dentro de `obtenerEstadisticas()`, calcula y devuelve la lista de elementos que cruzan el umbral, no solo el conteo:
  ```javascript
  const itemsAlerta = items
    .filter((it) => /* condición del umbral, ej. it.cantidad < UMBRAL_ALERTA */)
    .map((it) => ({ codigo: it.codigo, nombre: it.nombre, valor: it.cantidad }))
    .sort((a, b) => a.valor - b.valor);
  ```
  Devuélvelo como `itemsAlerta` en el objeto de datos junto al conteo (`stockBajo`, `pendientes`, o el nombre que tenga sentido en el dominio).
- Si el usuario quiere correo automático, agrega una función separada que NO se llama desde el frontend:
  ```javascript
  function enviarAlerta() {
    const res = obtenerEstadisticas();
    if (!res.success || !res.data.itemsAlerta.length) return;
    const filas = res.data.itemsAlerta.map((it) => `- ${it.codigo} · ${it.nombre} · ${it.valor}`).join("\n");
    MailApp.sendEmail({
      to: EMAIL_ALERTA,
      subject: `⚠️ Alerta — ${res.data.itemsAlerta.length} elemento(s)`,
      body: `Se detectaron ${res.data.itemsAlerta.length} elemento(s) que requieren atención:\n\n${filas}`,
    });
  }
  ```
  Esta función se activa con un **disparador de tiempo** (Triggers → Añadir disparador → impulsado por tiempo), nunca se llama desde `google.script.run` directamente, para no enviar correos en cada carga de la página.

## 2. Frontend (`Index.html`)

Agrega un contenedor de banner cerca de los KPIs, oculto por defecto:
```html
<div class="alert-banner" id="alertaBanner" style="display:none;">
  <span class="alert-icon">⚠️</span>
  <span id="alertaBannerTexto"></span>
  <button class="alert-close" id="cerrarAlerta">✕</button>
</div>
```

## 3. Estilos (`Stylesheet.html`)

```css
.alert-banner {
  margin-top: 18px; display: flex; align-items: center; gap: 10px;
  padding: 12px 18px; border-radius: var(--radius-sm, 12px);
  background: rgba(255, 159, 10, 0.12);
  border: 1px solid rgba(255, 159, 10, 0.35);
  color: var(--warning); font-size: 13.5px; font-weight: 600;
  animation: riseIn 400ms ease both;
}
.alert-banner span:nth-child(2) { flex: 1; }
.alert-close { border: none; background: transparent; cursor: pointer; color: var(--warning); opacity: 0.7; }
.alert-close:hover { opacity: 1; }
```
(reutiliza la animación `riseIn` si ya existe en el archivo; si no, agrega un `@keyframes riseIn` simple de fade+slide).

## 4. Cliente (`JS.html`)

Dentro de la función que pinta los KPIs (la que recibe la respuesta de `obtenerEstadisticas`), agrega:
```javascript
function renderAlertaBanner(items) {
  const banner = document.getElementById("alertaBanner");
  const texto = document.getElementById("alertaBannerTexto");
  if (!items.length) { banner.style.display = "none"; return; }
  const nombres = items.slice(0, 3).map((it) => it.nombre).join(", ");
  const extra = items.length > 3 ? ` y ${items.length - 3} más` : "";
  texto.textContent = `${items.length} elemento(s) requieren atención: ${nombres}${extra}.`;
  banner.style.display = "flex";
}
```
Y un listener para el botón de cerrar:
```javascript
document.getElementById("cerrarAlerta").addEventListener("click", () => {
  document.getElementById("alertaBanner").style.display = "none";
});
```

## Checklist al aplicar este patrón

- [ ] El umbral y la condición están claros y confirmados con el usuario (¿menor que? ¿mayor que? ¿fecha dentro de N días?).
- [ ] `itemsAlerta` se limita a lo necesario para no mandar objetos gigantes al cliente si el listado es muy largo (considera un `.slice(0, 50)` si aplica).
- [ ] Si hay correo, se explicó al usuario que debe crear el disparador de tiempo manualmente (esto no se puede hacer por código de forma silenciosa, requiere autorización).
- [ ] Se probó que el banner no se rompe cuando `itemsAlerta` viene vacío.
