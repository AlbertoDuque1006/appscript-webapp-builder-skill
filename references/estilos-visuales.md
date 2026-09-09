# Estilos visuales para la web app

Todos los estilos usan el mismo esqueleto de variables CSS en `:root` y `[data-theme="dark"]`, cambia solo los valores. Esto mantiene el toggle claro/oscuro funcionando igual sin importar el estilo elegido.

Variables base que SIEMPRE deben existir (los componentes del Paso 2 del SKILL.md dependen de ellas):
`--bg`, `--bg-elevated`, `--bg-solid`, `--text`, `--text-muted`, `--border`, `--accent`, `--danger`, `--warning`, `--success`, `--shadow`, `--radius`, `--transition`.

---

## Apple (glassmorphism)

- Fuente: `-apple-system, BlinkMacSystemFont, "SF Pro Display", "SF Pro Text", "Helvetica Neue", Arial, sans-serif`.
- Fondos elevados con `backdrop-filter: saturate(180%) blur(20px)` y `rgba(255,255,255,0.72)` (o `rgba(28,28,30,0.72)` en oscuro).
- Bordes muy sutiles: `rgba(0,0,0,0.08)`.
- Botones tipo píldora (`border-radius: 999px`).
- Acento: azul `#0071e3`.
- Animaciones suaves, `cubic-bezier(0.25, 0.8, 0.25, 1)`, nada abrupto.
- `--radius: 18px` en tarjetas, `999px` en botones/inputs de búsqueda.

## Minimalista plano

- Fuente: system-ui o Inter.
- Sin sombras pronunciadas ni blur — bordes sólidos de 1-2px en vez de `box-shadow`.
- Esquinas menos redondeadas: `--radius: 8px`.
- Paleta reducida: blanco/negro/un solo acento.
- Botones rectangulares o con esquinas ligeramente redondeadas, sin gradientes.

## Corporativo / serio

- Fuente: "Segoe UI", Arial, sans-serif.
- Paleta: azul marino (`#1a2f4b`), grises, acento contenido (no colores saturados).
- Tablas densas, con líneas divisorias visibles, poco espacio en blanco.
- Botones rectangulares, radios pequeños (`--radius: 6px`).
- Evitar animaciones llamativas — transiciones cortas (120-150ms) o ninguna.

## Colorido / juguetón

- Fuente: redondeada (Poppins, Nunito, o system-ui con `font-weight` alto en títulos).
- Gradientes en KPIs y botones primarios.
- `--radius: 20px`+ en casi todo.
- Colores saturados para los badges de estado, iconos/emojis usados con más libertad en botones y toasts.
- Animaciones más notorias (rebote, escala) al interactuar.

## Oscuro / técnico (tipo panel de monitoreo)

- Fondo casi negro por defecto (no hace falta ni toggle claro, pero inclúyelo igual si el usuario lo pide).
- Acentos neón (verde `#00ff9c`, cian, o el color de marca) sobre fondo oscuro.
- Tipografía monoespaciada para códigos/IDs (`"SF Mono", "Roboto Mono", monospace`).
- Bordes finos brillantes en vez de sombras difusas.
- Tablas con más contraste entre filas pares/impares.

---

Al aplicar cualquiera de estos, conserva intactos los nombres de las clases usadas por `JS.html` (`.kpi-card`, `.badge`, `.toast`, `.modal-overlay`, `.btn-primary`, etc.) — solo cambia cómo se ven, nunca sus nombres, o el JavaScript deja de encontrarlas.
