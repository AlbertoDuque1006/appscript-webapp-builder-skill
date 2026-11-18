# Apps Script WebApp Builder (skill de Claude)

Repositorio PÚBLICO: https://github.com/AlbertoDuque1006/appscript-webapp-builder-skill (licencia MIT). Es una skill de Claude que crea, corrige y amplía apps web de Google Apps Script sobre Google Sheets (CRUD, dashboards, formularios). Responde siempre en español.

- `SKILL.md`: las instrucciones que Claude carga al activar la skill. Cámbialo con cuidado: la descripción del encabezado es la que decide cuándo se activa.
- `assets/*.template`: plantillas base (`Code.gs`, `Index.html`, `Stylesheet.html`, `JS.html`).
- `references/`: estilos visuales, alertas por umbral y patrones avanzados (CSV/PDF, gráficos, roles, varias hojas).
- Para empacarla como `.skill` se usa el script `package_skill.py` de la skill `skill-creator`.
- Las convenciones de los proyectos reales de Apps Script (marca blanca, `{success, data|error}`, LockService, lectura por nombre de columna, pruebas con mock) están en los repos privados de reservas y restaurante del autor. Si la skill mejora, puede tomar ideas de ahí, pero sin nombres de empresas ni nada de clientes.

Commits solo con la cuenta AlbertoDuque1006.
