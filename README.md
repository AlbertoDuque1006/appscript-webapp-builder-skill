# appscript-webapp-builder

Skill de Claude para crear, corregir y extender aplicaciones web (dashboards, sistemas CRUD, formularios, herramientas de administración) construidas con **Google Apps Script** sobre **Google Sheets**.

No asume un dominio fijo — sirve para inventarios, clínicas, ventas, asistencia, tareas, reservas, o cualquier otro caso de uso. Si el usuario todavía no tiene la hoja de datos, la skill también ayuda a crearla desde cero.

## Contenido

```
appscript-webapp-builder-skill/
├── SKILL.md                          # Instrucciones principales (se cargan siempre que la skill se activa)
├── assets/                           # Plantillas de código base, listas para adaptar
│   ├── Code.gs.template
│   ├── Index.html.template
│   ├── Stylesheet.html.template
│   └── JS.html.template
└── references/                       # Documentación consultada según necesidad
    ├── estilos-visuales.md           # Variantes visuales: Apple, minimalista, corporativo, colorido, oscuro
    ├── patron-alertas.md             # Alertas por umbral (visual + correo con disparador de tiempo)
    └── patrones-avanzados.md         # Exportar CSV/PDF, gráficos, paginación, roles, multi-hoja, etc.
```

## Qué hace

- **Crea proyectos nuevos**: entrevista rápida (columnas, funcionalidad, estilo visual) → entrega los 4 archivos de Apps Script (`Code.gs`, `Index.html`, `Stylesheet.html`, `JS.html`) listos para pegar.
- **Crea la hoja de datos si no existe**, generando un Excel inicial con formato y validaciones.
- **Itera sobre proyectos existentes**: da parches puntuales en vez de regenerar todo, para agregar funcionalidades o corregir errores sin romper lo que ya funciona.
- **Diagnostica errores comunes de Apps Script** (archivo HTML no encontrado, hoja no encontrada, cambios no reflejados por falta de reimplementación, etc.).
- **Trae patrones avanzados listos**: exportar CSV/PDF, gráficos con Chart.js, paginación, datos relacionados en varias hojas, roles y permisos, carga masiva, historial por registro, búsqueda avanzada, duplicar registro.

## Instalación en Claude

1. Empaqueta esta carpeta como `.skill` (ver abajo) o usa el archivo `.skill` ya generado si lo tienes.
2. Ábrelo en el chat de Claude.
3. Haz clic en **"Guardar skill"** en la tarjeta que aparece.

### Reempaquetar después de editar

Si modificas el contenido de esta carpeta y quieres regenerar el archivo `.skill`:

```bash
python -m scripts.package_skill appscript-webapp-builder-skill ./dist
```
(requiere el script `package_skill.py` de la skill `skill-creator` de Claude)

## Licencia

Uso personal — ajusta según lo que necesites.
