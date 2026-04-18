# Prompt para Claude Code — Módulo "Vida y Hogar" v2 (Compras + Fechas + Sueño)

## Contexto del entorno

Estoy trabajando en **Antigravity (Anysphere)**. El proyecto es un único archivo HTML: `plataforma_clinica.html`, con React 18 + Babel + Tailwind CDN + **Chart.js ya cargado** (línea 20 del HTML), y persistencia en `localStorage` con prefijo `hgsp_`. **Leé el archivo completo antes de hacer cambios** para respetar patrones existentes (estilos, paleta, estructura de datos, helpers `getData`/`setData`, `showToast`, componentes `Modal`, `StatCard`, `Tabs`, `Badge`, etc.).

## Objetivo general

Agregar un **nuevo módulo "Vida y Hogar"** separado del área clínica. Es un **árbol navegable de sub-módulos** sobre organización personal. La mayoría están "por construirse" pero con estructura visible y estados de color. En esta fase construimos **tres sub-módulos completos**: **Lista de Compras**, **Fechas Importantes** y **Sueño**.

## 1. Cambios en el sidebar

En `NAV_SECTIONS` (~línea 1458), agregar una tercera sección `VIDA` entre `PRINCIPAL` y `SISTEMA`:

```js
{ id: 'vida_hogar', label: 'Vida y Hogar', emoji: '🌳', group: 'VIDA' },
```

El grupo `VIDA` se renderiza con el mismo estilo que los otros grupos. Badge del ítem = suma de "cosas pendientes" en todos los sub-módulos activos (ítems por reponer + fechas próximas 7 días + si no registró sueño anoche).

## 2. Arquitectura del módulo "Vida y Hogar"

Al hacer clic en el ítem se renderiza `SeccionVidaHogar`, que muestra un **árbol de categorías y sub-módulos** con estados de color.

### Estructura del árbol (hardcodeada)

```js
const ARBOL_VIDA = [
  {
    id: 'hogar_logistica',
    label: 'Hogar y Logística',
    emoji: '🏠',
    hijos: [
      { id: 'compras',       label: 'Lista de compras e inventario', emoji: '🛒', estado: 'activo' },
      { id: 'mantenimiento', label: 'Mantenimiento de vivienda',     emoji: '🔧', estado: 'pendiente' },
      { id: 'servicios',     label: 'Servicios y vencimientos',      emoji: '📄', estado: 'enlazado', enlaceA: 'gastos' },
      { id: 'mascotas',      label: 'Mascotas (Lola y Missy)',       emoji: '🐕', estado: 'pendiente' },
    ]
  },
  {
    id: 'salud_cuerpo',
    label: 'Salud y Cuerpo',
    emoji: '❤️',
    hijos: [
      { id: 'medicacion',   label: 'Medicación y suplementos',       emoji: '💊', estado: 'pendiente' },
      { id: 'turnos_med',   label: 'Turnos médicos y controles',     emoji: '🩺', estado: 'pendiente' },
      { id: 'ejercicio',    label: 'Ejercicio y mediciones',         emoji: '🏋️', estado: 'pendiente' },
      { id: 'sueno',        label: 'Sueño',                          emoji: '😴', estado: 'activo' },
    ]
  },
  {
    id: 'vinculos_social',
    label: 'Vínculos y Social',
    emoji: '👥',
    hijos: [
      { id: 'contactos',   label: 'Contactos clave y última vez',    emoji: '📇', estado: 'pendiente' },
      { id: 'fechas_imp',  label: 'Fechas importantes',              emoji: '🎂', estado: 'activo' },
      { id: 'compromisos', label: 'Compromisos sociales pendientes', emoji: '🤝', estado: 'pendiente' },
    ]
  },
  {
    id: 'aprendizaje',
    label: 'Aprendizaje y Proyectos',
    emoji: '📚',
    hijos: [
      { id: 'cursos',    label: 'Cursos en curso',    emoji: '🎓', estado: 'pendiente' },
      { id: 'proyectos', label: 'Proyectos activos',  emoji: '🚀', estado: 'pendiente' },
      { id: 'capturas',  label: 'Capturas e ideas',   emoji: '💡', estado: 'pendiente' },
    ]
  },
  {
    id: 'habitos',
    label: 'Hábitos y Chequeo Interno',
    emoji: '🧘',
    hijos: [
      { id: 'terapia',     label: 'Terapia personal',             emoji: '🪑', estado: 'pendiente' },
      { id: 'canto',       label: 'Clases de canto (lunes)',      emoji: '🎤', estado: 'pendiente' },
      { id: 'journaling',  label: 'Estado de ánimo / journaling', emoji: '📓', estado: 'pendiente' },
      { id: 'tiempo_apps', label: 'Tiempo en apps',               emoji: '📱', estado: 'pendiente' },
    ]
  },
];
```

### Estados de color (3 estados)

- **`activo`**: dorado (`--do`, #C8922A). Funcional.
- **`enlazado`**: verde claro (`--vc`, #4A7C5F) con flecha →. Navega a otro módulo.
- **`pendiente`**: gris apagado (#A0998C). NO usar rojo. Al clickear despliega panel inline "Por construirse — [descripción breve]".

### Regla de color en categorías padre

Categoría padre pinta verde activo si al menos un hijo está `activo`. Si no, gris pendiente. Los enlazados no cuentan como activos.

### Layout del árbol

- Tarjetas expansibles tipo acordeón.
- Categoría padre → expande/colapsa hijos.
- Hijo `activo` → abre vista del sub-módulo.
- Hijo `enlazado` → navega a `activeSection` destino.
- Hijo `pendiente` → panel inline "Por construirse".
- Contador pequeño: "X activos · Y pendientes" por categoría.

## 3. Sub-módulo COMPRAS (desarrollo completo)

### 3.1 Estructura de datos

`hgsp_compras_items`:
```js
{
  id, nombre, categoria, unidad, cantidadPorCompra,
  cicloDias, ultimaCompra, precioReferencia,
  lugar, activo, notas, fechaCreacion
}
```

`hgsp_compras_historial`:
```js
{ id, itemId, fecha, cantidad, precio, notas }
```

### 3.2 Lógica de reposición — `calcularEstadoReposicion(item)`

- **`sin_datos`**: sin `ultimaCompra` → "Cargá primera compra"
- **`ok`**: días desde última < ciclo - 3 → verde
- **`reponer_pronto`**: ciclo - 3 ≤ días ≤ ciclo → dorado, "Quedan ~X días"
- **`vencido`**: días > ciclo → rojo, "Hace X días del ciclo"
- **`inactivo`**: `item.activo === false` → gris

### 3.3 Vistas (tabs)

1. **⚠️ Reponer pronto** (por defecto) — `reponer_pronto` + `vencido` ordenados por urgencia.
2. **📋 Lista completa** — tabla filtrable por categoría y estado.
3. **📊 Historial y gastos** — cronológico con total por mes.
4. **➕ Nuevo ítem**.

### 3.4 Stats cards

- Ítems activos
- Para reponer pronto
- Vencidos
- Gasto del mes actual (suma historial)

### 3.5 Acciones por ítem

- **✅ Marcar comprado hoy** → mini-modal: fecha (hoy), cantidad (prellenada), precio (prellenado). Al guardar: crea registro historial, actualiza `ultimaCompra`, **crea entrada en `hgsp_gastos`** con `origenCompras: true`. Toast: "🛒 Compra registrada y agregada a gastos".
- **✏️ Editar** | **🗑️ Eliminar** (con confirmación) | **⏸️ Activar/Desactivar**

### 3.6 Formulario nuevo/editar ítem

Campos: nombre, categoría (select), unidad (select), cantidad por compra, ciclo días (con presets 7/15/30/60/90), precio referencia (opcional), lugar habitual, notas, toggle activo.

### 3.7 Seed inicial (si clave vacía)

```js
[
  { nombre: 'Jabón líquido ropa',              categoria: 'Limpieza',         unidad: 'unidad',  cantidadPorCompra: 1, cicloDias: 30, precioReferencia: 4500 },
  { nombre: 'Papel higiénico',                 categoria: 'Higiene personal', unidad: 'paquete', cantidadPorCompra: 1, cicloDias: 21, precioReferencia: 8000 },
  { nombre: 'Alimento perros (Lola y Missy)',  categoria: 'Otro',             unidad: 'kg',      cantidadPorCompra: 15, cicloDias: 30, precioReferencia: 25000 },
  { nombre: 'Creatina',                         categoria: 'Otro',             unidad: 'unidad',  cantidadPorCompra: 1, cicloDias: 60, precioReferencia: 15000 },
  { nombre: 'Proteína en polvo',                categoria: 'Otro',             unidad: 'unidad',  cantidadPorCompra: 1, cicloDias: 45, precioReferencia: 35000 },
  { nombre: 'Bizcochos',                        categoria: 'Alimentos',        unidad: 'paquete', cantidadPorCompra: 6, cicloDias: 7,  precioReferencia: 1500 },
  { nombre: 'Yerba',                            categoria: 'Alimentos',        unidad: 'kg',      cantidadPorCompra: 1, cicloDias: 20, precioReferencia: 3500 },
]
```

Todos `activo: true`, `ultimaCompra: null`, `fechaCreacion` = hoy.

## 4. Sub-módulo FECHAS IMPORTANTES (desarrollo completo)

### 4.1 Estructura de datos

`hgsp_fechas_importantes`:
```js
{ id, tipo, persona, fecha: "MM-DD", anio: number|null, recurrente: boolean, notas, activo }
```

### 4.2 Vistas (tabs)

1. **📅 Próximas 30 días** (por defecto) — ordenadas por cercanía.
2. **🗓️ Calendario anual** — lista mes por mes.
3. **➕ Nueva fecha**

### 4.3 Stats cards

- Próximas 7 días | Próximas 30 días | Total del año | Del mes actual

### 4.4 Lógica de "próximas"

Días faltantes: si recurrente, usar este año o próximo (lo que dé días positivos menores). Destacar:
- ≤ 3 días → alerta rojiza
- ≤ 7 días → dorado
- ≤ 30 días → verde

### 4.5 Seed inicial

```js
[
  { tipo: 'cumpleaños', persona: 'Hernán (yo)',       fecha: '11-15', recurrente: true },
  { tipo: 'cumpleaños', persona: 'Laureano (Polaco)', fecha: '',     recurrente: true, notas: 'Completar fecha' },
  { tipo: 'cumpleaños', persona: 'Leonel (Pepo)',     fecha: '',     recurrente: true, notas: 'Completar fecha' },
  { tipo: 'cumpleaños', persona: 'Papá',              fecha: '',     recurrente: true, notas: 'Completar fecha' },
]
```

## 5. Sub-módulo SUEÑO (desarrollo completo) ⭐ módulo estrella

Este módulo es prioritario. Su objetivo **no es solo registrar**, es **acompañar un proceso de cambio**: Hernán se duerme entre 2-4am y quiere revertir ese patrón. El módulo debe ser fácil de cargar, visualmente claro, y motivador sin ser cursi.

### 5.1 Estructura de datos

`hgsp_sueno_registros`:
```js
{
  id: string,
  fechaDormir: "YYYY-MM-DD",   // fecha del día en que se acostó
  horaAcostarse: "HH:MM",       // ej "02:15"
  fechaDespertar: "YYYY-MM-DD",
  horaDespertar: "HH:MM",
  duracionMin: number,          // calculada automáticamente
  calidad: 1 | 2 | 3 | 4 | 5,   // estrellas
  notas: string,
  fechaRegistro: ISO string
}
```

`hgsp_sueno_config`:
```js
{
  metaHoraAcostarse: "00:30",   // meta configurable, default 00:30
  metaHorasDormir: 7,            // meta de horas totales
  fechaMetaCreada: "YYYY-MM-DD"  // para saber desde cuándo rige
}
```

`hgsp_sueno_objetivos_historial`:
```js
[
  { metaHoraAcostarse, fechaDesde, fechaHasta: null|"YYYY-MM-DD", cumplida: bool|null }
]
```

### 5.2 Normalización de hora de acostarse

Importante: si se acuesta a las 02:15, eso **cuenta como "tarde" del día anterior**, no temprano. Para cálculos de desviación respecto a la meta, normalizar: horas entre 00:00 y 06:00 se tratan como "hora + 24" (ej: 02:15 → 26:15) para comparar con meta.

### 5.3 Vistas (tabs)

1. **🌙 Hoy y esta semana** (por defecto) — dashboard principal
2. **📈 Tendencia del mes** — gráfico y estadísticas
3. **🎯 Objetivos** — meta actual, historial de metas, próximo objetivo sugerido
4. **📝 Historial** — tabla de todos los registros

### 5.4 Dashboard "Hoy y esta semana"

Elementos en orden vertical:

**A. Tarjeta grande de estado actual**
- Si **no registró anoche** → fondo dorado, botón grande **"¿A qué hora te dormiste anoche?"** con campo hora rápido y botón guardar.
- Si **ya registró** → mostrar resumen: "Anoche: dormiste a las HH:MM · HH:MM dormido · Calidad ⭐⭐⭐⭐"

**B. Botones de acción dual**
- **"💤 Registro rápido"** → mini-modal con solo hora acostarse + hora despertar + calidad (3 campos, 10 segundos).
- **"📝 Registro completo"** → modal con todos los campos incluida nota.

**C. Mensaje motivacional contextual** (ver sección 5.8)

**D. Gráfico de la semana** (Chart.js)
- Tipo: barras verticales, una por día (Lun a Dom).
- Eje Y: hora de acostarse (normalizada, 20:00 a 30:00 / o sea hasta las 06:00 del día siguiente).
- **Línea horizontal punteada dorada** en el valor de `metaHoraAcostarse`.
- Barras: **verdes** si cumplen la meta, **doradas** si se pasaron hasta 1h, **rojas** si se pasaron más.
- Tooltip al pasar: fecha, hora exacta, duración, calidad.

**E. Stats de la semana**
- Promedio hora de acostarse
- Días cumplidos / 7
- Promedio horas dormidas
- Racha actual (días consecutivos cumpliendo meta)

### 5.5 Vista "Tendencia del mes"

- **Gráfico de línea** de los últimos 30 días con hora de acostarse normalizada, meta como línea horizontal.
- Debajo: métricas grandes
  - Promedio del mes
  - Mejor racha del mes
  - % de cumplimiento de meta (ej: "63% — 19 de 30 días")
  - Tendencia últimos 7 días vs 7 días previos (↑ mejorando / → estable / ↓ empeorando)
- **Sección "Correlación con notas"**: si hay al menos 5 registros con nota, listar las palabras más frecuentes en noches de acostarse tarde vs temprano (simple: palabras de 4+ letras, excluidas stopwords básicas, top 5 de cada lado).

### 5.6 Vista "Objetivos" (sistema de progresión) 🎯

**Principio**: el sistema no impone objetivos rígidos. **Acompaña progresivamente**, sugiriendo el próximo escalón cuando el anterior se consolida.

**Lógica de sugerencia de próximo objetivo**:

```js
function sugerirProximoObjetivo(registros, configActual) {
  // Necesita al menos 7 registros en los últimos 14 días
  const recientes = registros filtrados últimos 14 días;
  if (recientes.length < 7) return { sugerir: false, mensaje: "Seguí registrando para que pueda sugerirte un próximo paso" };
  
  const cumplimiento = % de días que cumplieron la meta actual en últimos 14 días;
  const promedioActual = promedio hora acostarse últimos 14 días (normalizada);
  
  if (cumplimiento >= 70%) {
    // Está consolidado, proponer adelantar 30 minutos
    const nuevaMeta = metaActual - 30 minutos;
    return {
      sugerir: true,
      nuevaMeta,
      razon: `Cumpliste la meta de ${metaActual} el ${cumplimiento}% de los días. Probemos adelantar 30 minutos.`
    };
  }
  if (cumplimiento >= 40%) {
    return { sugerir: false, mensaje: "Seguí consolidando la meta actual antes de avanzar" };
  }
  // Cumplimiento bajo → sugerir meta más accesible
  const metaRealista = promedioActual - 15 minutos;
  return {
    sugerir: true,
    nuevaMeta: metaRealista,
    razon: `Ajustemos la meta a algo más cercano a tu ritmo actual. Pequeños pasos son más sostenibles que saltos grandes.`
  };
}
```

En la vista, mostrar:
- **Meta actual** con opción de editarla manualmente.
- **Tu progreso con esta meta**: "Llevás 14 días · cumpliste 9 · 64%"
- **Próximo objetivo sugerido** (tarjeta destacada) si aplica, con botón "Aceptar nuevo objetivo" que actualiza la meta y guarda la anterior en historial.
- **Historial de metas** (lista): "Desde DD/MM hasta DD/MM — 00:30 — Logrado/Abandonado"

### 5.7 Vista "Historial"

Tabla con todos los registros: fecha, hora acostarse, hora despertar, duración, calidad, nota (truncada, hover muestra completa). Paginada o con scroll. Eliminar individual.

### 5.8 Mensajes motivacionales contextuales 💬

Sistema sobrio, tono de coach directo, no Instagram. Las frases se eligen según contexto:

```js
function mensajeMotivacional(registros, meta) {
  const ultimo = registros[0];
  const racha = calcularRachaActual();
  const cumplimientoSemana = % últimos 7 días;
  
  // Sin registros
  if (!ultimo) return "Empezá registrando anoche. No hace falta que sea perfecto, solo que sea real.";
  
  // Racha buena
  if (racha >= 7) return `${racha} días seguidos cumpliendo la meta. Esto ya no es casualidad, es un hábito formándose.`;
  if (racha >= 3) return `${racha} días seguidos. Va tomando forma.`;
  
  // Cumplió anoche pero sin racha
  if (cumplioAnoche && racha < 3) return "Bien anoche. Lo importante ahora es repetir.";
  
  // No cumplió anoche pero viene bien
  if (!cumplioAnoche && cumplimientoSemana >= 60) return "Una noche fuera del objetivo no rompe el proceso. Lo que cuenta es el patrón de la semana.";
  
  // No cumplió y viene mal
  if (!cumplioAnoche && cumplimientoSemana < 40) return "Ajustá la meta si hace falta. Una meta alcanzable es mejor que una ideal que se siente inalcanzable.";
  
  // Mejoró respecto a la semana anterior
  if (tendencia === 'mejorando') return "Los últimos días vienen mejor que los anteriores. Eso es lo que importa.";
  
  // Default
  return "Seguí registrando. Lo que se mide, se puede cambiar.";
}
```

Tono: directo, honesto, sin exclamaciones excesivas, sin "¡Vamos campeón!". Una sola frase por mensaje.

### 5.9 Stats cards del módulo Sueño (arriba)

- Promedio de la semana (hora)
- Cumplimiento meta (% últimos 7 días)
- Racha actual (días)
- Horas dormidas promedio

### 5.10 Configuración inicial (primer uso)

Si `hgsp_sueno_config` no existe, al entrar al módulo mostrar un onboarding de un solo paso:

> "**Definamos tu meta de hora de acostarse**
> 
> Elegí una hora realista. Podés ajustarla en cualquier momento.
> 
> [input hora, default 01:00]
> 
> [Botón: Guardar meta y empezar]"

Mensaje: "Esta va a ser tu referencia. El objetivo no es cumplirla perfecto desde el día uno — es tener un patrón visible."

## 6. Sub-módulos "pendientes" (panel inline)

Al clickear ítem `pendiente`, panel debajo con:
- Título "Por construirse"
- Descripción breve (1-2 líneas)

Descripciones breves:
- **Mantenimiento**: "Tareas de la casa con ciclo de recurrencia y estado hecho/pendiente."
- **Mascotas**: "Control de Lola y Missy: vacunas, antiparasitarios, veterinario, alimento."
- **Medicación**: "Registro de medicación y suplementos con recordatorio de stock y ciclo."
- **Turnos médicos**: "Agenda de controles médicos con alertas de vencimientos anuales."
- **Ejercicio**: "Registro de entrenamientos, mediciones corporales y rehabilitación."
- **Contactos**: "Libreta con última vez que viste a cada persona clave — contrapesa patrones de aislamiento."
- **Compromisos**: "Cosas sociales pendientes: devolver llamada, invitar, responder."
- **Cursos**: "Avance de cursos en los que estás inscripto con % de progreso."
- **Proyectos**: "Proyectos personales activos con estado y próxima acción."
- **Capturas**: "Ideas, frases, imágenes capturadas para procesar después."
- **Terapia**: "Registro semanal de sesiones y temas a traer."
- **Canto**: "Clases de canto del lunes con repertorio y ejercicios."
- **Journaling**: "Chequeo diario de estado de ánimo con nota breve."
- **Tiempo en apps**: "Autorregistro de tiempo en TikTok/Netflix para visibilizar uso."

## 7. Estilo visual

- Respetar paleta existente: `--vd`, `--vm`, `--do`, `--vc`, `--cr`, etc.
- Componentes existentes: `Modal`, `Tabs`, `StatCard`, `Badge`, `SearchInput`.
- Clases existentes: `.card`, `.btn`, `.btn-p`, `.btn-g`, `.btn-sec`, `.btn-danger`, `.alert-warn`, `.alert-info`, `.input`, `.tbl`, `.modal-input`, `.modal-select`, `.modal-textarea`.
- Transiciones suaves de expand/collapse.
- Responsive con `.tbl-wrap` para scroll horizontal.

## 8. Integración con Gastos

Al registrar compra → entrada en `hgsp_gastos`:
```js
{
  id: genId(),
  mes: <mes actual>,
  anio: <año actual>,
  categoria: 'Vida',
  concepto: item.nombre,
  montoEstimado: item.precioReferencia,
  montoReal: precioIngresado,
  pagado: true,
  origenCompras: true,
  itemComprasId: item.id
}
```

## 9. Actualizar `computeBadges`

En `computeBadges` (~línea 3884):

```js
const comprasItems = getData('hgsp_compras_items');
const itemsReponer = comprasItems.filter(i => {
  if (!i.activo || !i.ultimaCompra) return false;
  const dias = Math.floor((new Date() - new Date(i.ultimaCompra)) / 86400000);
  return dias >= i.cicloDias - 3;
}).length;

const fechas = getData('hgsp_fechas_importantes');
const fechasProximas = /* calcular fechas próximas ≤ 7 días */;

const suenoRegistros = getData('hgsp_sueno_registros');
const hoyStr = new Date().toISOString().slice(0,10);
const registroHoy = suenoRegistros.some(r => r.fechaDespertar === hoyStr);
const suenoPendiente = registroHoy ? 0 : 1;

return {
  // ... los existentes
  vida_hogar: itemsReponer + fechasProximas + suenoPendiente
};
```

## 10. Resumen ordenado de tareas

1. Leer el archivo completo.
2. Agregar entrada al sidebar + grupo `VIDA`.
3. Crear `SeccionVidaHogar` con árbol de categorías y 3 estados de color.
4. Crear `SubModuloCompras` completo (4 tabs, CRUD, lógica reposición, integración Gastos).
5. Crear `SubModuloFechasImportantes` completo (3 tabs, CRUD, cálculo próximas).
6. Crear `SubModuloSueno` completo: onboarding inicial, dashboard con estado de anoche, registro dual (rápido/completo), gráficos Chart.js semanal y mensual, vista de objetivos con sugerencia progresiva, mensajes motivacionales contextuales.
7. Crear componente genérico `PanelPorConstruirse`.
8. Función `seedDataVidaHogar()` que carga ejemplos si las claves están vacías — llamarla desde `seedData()` existente.
9. Actualizar `computeBadges` y enrutamiento en `renderSection` de `App`.
10. Probar navegación completa y los 3 módulos funcionales.

## 11. Lo que NO quiero

- No crear archivos separados. Todo en un único HTML.
- No agregar librerías nuevas (Chart.js ya está).
- No tocar código del área clínica existente.
- No usar rojo para pendientes (solo gris).
- No hacer los módulos pendientes funcionales — solo estructura visual + panel "Por construirse".
- No frases motivacionales tipo Instagram ni emoticones excesivos en los mensajes de Sueño — tono sobrio, directo.

## 12. Notas finales para Claude Code

- Si el archivo es muy grande para procesar de una, **hacelo por fases**: primero el esqueleto del módulo + árbol, después un sub-módulo por vez (Sueño primero si hay que priorizar, porque es el más complejo y el más importante para mí).
- Antes de terminar, verificá que: los 3 módulos activos funcionan end-to-end, el badge del sidebar se calcula bien, Chart.js renderiza sin errores de consola, el seed no duplica si se ejecuta dos veces, y los módulos "pendientes" despliegan correctamente.
