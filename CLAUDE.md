# Tareas — app personal de escritorio

## Comunicación
Respuestas mínimas. Sin explicar lo que vas a hacer ni resumir lo hecho. Si hay duda real, una pregunta corta; si no, decide y sigue.

## Qué es
Gestor personal de tareas para Rubén. Un solo `index.html` (HTML + CSS + JS vanilla, sin build, sin dependencias externas, sin frameworks). Se abre en Edge/Chrome desde el disco. Sustituye las banderas y categorías de color de Outlook. No es Planner: nada de recordatorios, notificaciones, subtareas, adjuntos ni colaboración.

Principio: capturar y mover tareas tiene que ser "pim pam". Una línea de texto para crear, un clic o una tecla para mover.

## Modelo de datos

```json
{
  "v": 1,
  "settings": { "captureIn": "week", "waitDays": 5, "warnToday": 5, "theme": "light" },
  "programs":  [{ "id": "p1", "name": "ALPHA" }, { "id": "p2", "name": "Catálogo y Taller de Precios" }],
  "projects":  [{ "id": "pr1", "programId": "p1", "name": "Norte", "fav": true }],
  "people":    [{ "id": "me", "name": "Yo", "color": "black" }, { "id": "u1", "name": "Marta", "color": "blue" }],
  "tasks": [{
    "id": "t1", "title": "Pedir presupuesto licencias",
    "personId": "u1", "projectId": "pr1",
    "type": "hacer | urgente",
    "date": "2026-10-08",
    "waiting": false, "remindOn": null,
    "note": "", "order": 0,
    "done": false, "doneAt": null, "createdAt": "2026-10-05T10:00:00Z"
  }]
}
```

- `date` es la fecha en la que quiero ver la tarea, no un vencimiento. Siempre tiene valor.
- `waiting: true` = la tarea está en Esperando; `remindOn` es la fecha de reclamo.
- Persona `me` existe siempre, no se puede borrar, color negro.
- Cada proyecto pertenece a un programa. Las tareas solo llevan proyecto.

## Columnas (calculadas, nunca almacenadas)
Las columnas se derivan de `date` / `waiting` cada vez que se renderiza, con la fecha de hoy. Una tarea para el jueves está en "Esta semana" el lunes y en "Hoy" el jueves, sin intervención.

| Columna | Regla |
|---|---|
| Hoy | `date <= hoy` y no waiting, **o** waiting con `remindOn <= hoy` |
| Mañana | `date == hoy+1` |
| Esta semana | `date` entre hoy+2 y el domingo de esta semana |
| Próxima semana | `date` en la semana siguiente (lunes a domingo) |
| Más adelante | `date` posterior |
| Esperando | `waiting` y `remindOn > hoy` |

Semana empieza en lunes. Hoy no tiene límite; si hay más de `warnToday` tareas, el contador se pone en rojo con texto "demasiadas, reparte". Las de Esperando que llegan a Hoy se muestran como reclamo (fondo ámbar suave) y con texto "reclamar · desde dd/mm".

## Layout
Pantalla de escritorio, 1280+ px. Fondo claro `#f3f2ee`, tipografía system-ui (sin fuentes externas), texto `#1b1b1a`.

1. **Cabecera**: título, fecha de hoy. A la derecha: filtros (ver abajo), botón Hechas (N), botón Ajustes.
2. **Línea rápida**: input grande (46 px, 16 px de texto). Debajo, una línea de ayuda con la sintaxis y los atajos.
3. **Hoy**: panel blanco a la izquierda, ~35 % del ancho. Tarjetas grandes: título 16 px, chip de persona, chip de proyecto, fecha, y botones siempre visibles: `→ mañ`, `→ vie`, `⏸ espera`, `✓`. Orden manual por arrastre dentro de Hoy.
4. **Resto**: cinco columnas a la derecha en formato lista compacta (una fila = título 13 px; segunda línea chip persona + fecha). Sin botones visibles; al pasar el ratón aparecen: sol (a Hoy), `→ mañ`, `⏸`, `✓`. Esperando con fondo `#f1eae1`, borde `#dccbb8`, cabecera ámbar, y la fecha de cada fila dice "reclamar jue 8".
5. **Pie**: leyenda de colores y estado de guardado ("Guardado en tareas.json · hh:mm" o "Solo navegador · vincular archivo en Ajustes").

Referencia visual: la maqueta acordada (panel Hoy grande a la izquierda, listas compactas a la derecha). No copiar el estilo del Cockpit: nada de campos "Persona / Fecha límite" vacíos en cada tarjeta.

## Colores
Dos dimensiones separadas:

**Sin punto de tipo.** Urgente: barra roja `#c2321e` a la izquierda, título en 600 y fondo `#fff7f5`. **Tarjeta ámbar** (fondo `#faf6f1`, borde `#dccbb8`): la tarea es de otra persona (chip ≠ Yo) o es un reclamo que vuelve de Esperando. Si es urgente y de otra persona, fondo ámbar con barra roja.

**Chip = persona**. "Depende de alguien" no es un tipo: es que el chip no es Yo, y la tarjeta se pinta ámbar. Paleta de 8, fondo claro + texto oscuro (contraste ≥ 4.5:1), asignada en orden al crear personas; Yo siempre negro/blanco:

| nombre | fondo | texto |
|---|---|---|
| blue | #dbe8f7 | #1f4e8c |
| green | #d9efe8 | #1b6b54 |
| violet | #ece0f5 | #5e3a8c |
| orange | #fbe4d0 | #8a4a0f |
| teal | #d4eef0 | #0f6a70 |
| pink | #f8dde6 | #8c2f52 |
| olive | #e8ecd2 | #5a6414 |
| slate | #e2e5ea | #3b4756 |

## Línea rápida
Un input. Intro crea la tarea y vacía el input. Sintaxis, en cualquier orden:

- `@nombre` persona. Al teclear `@` se abre un desplegable con las personas guardadas, filtrando al escribir; flechas + Intro o clic selecciona. Última opción siempre "Crear persona: xxx"; una persona nueva **solo** se crea eligiendo esa opción explícitamente, nunca por un texto suelto. Nueva persona recibe el siguiente color de la paleta.
- `#proyecto` idéntico, con "Crear proyecto: xxx" (pide el programa en un desplegable pequeño).
- Fecha: `hoy`, `mañ`/`mañana`, `lun`…`dom` (próximo día de esa semana; si es hoy, hoy), `12/10`, `12/10/26`, `+3` (días), `nov`/`dic`… (día 1 del mes). Admite acentos y mayúsculas.
- `!` urgente. Sin marca: hacer. (`?` no tiene significado.)
- `Ctrl+Intro` en vez de `Intro` crea la tarea directamente en Hoy.
- Al crear: urgente sin fecha explícita → `date = hoy`. Persona ≠ Yo (y no urgente) → entra en Esperando con `remindOn = fecha explícita` o, si no hay, `hoy + waitDays`.
- Sin `@`: persona Yo. Sin fecha: según `settings.captureIn` (por defecto `week` = viernes de esta semana; si hoy es viernes o después, viernes de la siguiente). Opciones: today, tomorrow, week, nextweek.

Lo que queda después de extraer los tokens es el título, recortado.

## Mover tareas
Tres caminos, los tres cambian `date`/`waiting`:

- **Botones** en tarjeta (descritos en Layout).
- **Teclado** con una fila seleccionada (clic o flechas arriba/abajo): `H` hoy · `M` mañana · `S` esta semana (viernes) · `P` próxima semana (lunes) · `L` más adelante (+30 días) · `E` esperando · `X` o `Supr` hecha · `Intro` abrir edición · `Esc` deseleccionar.
- **Arrastre** entre columnas: soltar en Mañana → hoy+1; Esta semana → viernes de esta semana (si hoy ≥ viernes, viernes siguiente); Próxima semana → lunes siguiente; Más adelante → +30 días; Esperando → `waiting=true`, `remindOn = hoy + settings.waitDays`; Hoy → hoy. Dentro de Hoy, arrastrar reordena (`order`).

Pasar a Esperando (botón, tecla, arrastre, panel o al crear) usa `remindOn = hoy + waitDays`; si cae en sábado o domingo, pasa al lunes siguiente. Sol en una tarea de Esperando = reclamar ya: `waiting=false`, `date=hoy`. Tecla `U` o botón `!` alterna urgente; al marcar urgente, `date = hoy` y sale de Esperando.

Hecha: `done=true, doneAt=ahora`; desaparece de las columnas. Botón "Hechas (N)" muestra una lista con fecha y permite recuperar. Las hechas de más de 90 días se pueden purgar desde Ajustes.

## Editar
**En sitio**, sin abrir nada: clic en título → edición inline (Intro guarda, Esc cancela; acepta la misma sintaxis `@ # !` y la aplica). Clic en chip persona → desplegable (incluye Yo). Clic en fecha → selector con atajos hoy · mañ · vie · lun + calendario nativo. Clic en chip proyecto → desplegable.

**Panel lateral** (doble clic o Intro): todos los campos + nota libre (`note`, textarea) + botón Borrar (con confirmación). Esc cierra.

## Filtros
Cabecera: chip "Todo", "Mías" (persona = Yo), "Reclamar" (persona ≠ Yo o waiting), y pestañas por programa; con un programa activo aparecen los chips de sus proyectos. Un filtro afecta a todas las columnas. `/` enfoca un buscador de texto que filtra por título y nota.

## Atajos globales
`N` enfoca la línea rápida · `/` buscador · `?` muestra la tabla de atajos. Las teclas de mover solo actúan con una fila seleccionada y nunca mientras se escribe en un input.

## Persistencia
1. Siempre: `localStorage` (clave `tareas.v1`), guardado en cada cambio.
2. Preferente: archivo `tareas.json` vinculado con File System Access API (`showSaveFilePicker` / `showOpenFilePicker`). Botón "Vincular archivo" en Ajustes; el handle se guarda en IndexedDB para no pedirlo cada vez; al abrir se pide permiso si hace falta. Guardado en cada cambio, con debounce 500 ms. Si hay archivo y localStorage, manda el archivo.
3. Exportar / Importar JSON manual siempre disponible en Ajustes (respaldo y para navegadores sin la API).

`tareas.json` nunca se sube al repo (`.gitignore`).

## Ajustes (modal)
Capturar en · Alerta de espera (días) · Aviso en Hoy a partir de N · Tema claro/oscuro · Personas (lista editable: renombrar actualiza sus tareas, color, borrar solo si no tiene tareas) · Programas y proyectos (crear, renombrar, favorito, mover de programa) · Archivo vinculado · Exportar / Importar · Purgar hechas antiguas.

## Fuera de alcance (no implementar)
Recordatorios, notificaciones, correo, integración con Outlook (futuro), subtareas, adjuntos, multiusuario, sincronización, login, backend. Nada de librerías externas ni CDN.

## Calidad
- Todo en `index.html`. Código organizado en secciones: estado, parser, cálculo de columnas, render, acciones, persistencia, ajustes.
- Render completo a partir del estado tras cada acción (sin DOM parcial frágil). Debe ir fluido con 500 tareas.
- Fechas siempre en local, formato ISO `YYYY-MM-DD` en datos; mostrar como "jue 8", "12/10" o "nov".
- Botones y chips son `<button>` reales; `aria-label` en los de icono.
- Probar en Edge y Chrome. Sin consola con errores.
