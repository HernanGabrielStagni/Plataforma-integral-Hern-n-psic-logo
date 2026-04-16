# Plataforma Clínica — Gestión Integral Consultorio Psicológico y AT

## Proyecto

Aplicación web SPA completa en un único archivo HTML autocontenido.
Uso personal exclusivo del Lic. Hernán Gabriel Stagni (MP 11919, Córdoba, Argentina).
Se abre directamente en Chrome/Edge en Windows, sin servidor.

## Stack

- React 18+ y ReactDOM desde CDN (unpkg o esm.sh)
- Tailwind CSS desde CDN
- Babel standalone para JSX en browser
- Chart.js desde CDN para gráficos
- Lucide Icons desde CDN o emoji
- Google Fonts: Playfair Display, Nunito, DM Mono
- Persistencia: localStorage (prefijo `hgsp_` en todas las claves)
- Sin backend, sin login, sin APIs externas

## Reglas de trabajo

- Todo en UN SOLO ARCHIVO .html. No separar CSS ni JS.
- Comentar el código por secciones/módulos con separadores claros.
- Si el archivo crece, usar ediciones quirúrgicas (no reescribir completo).
- NO usar alert(), confirm(), prompt() nativos. Usar Modal y Toast custom.
- NO sobre-ingenierear: no agregar patterns, capas de abstracción ni features que no estén especificados.
- NO crear archivos temporales ni adicionales.
- NO usar fuentes del sistema. Siempre Google Fonts.
- NO hardcodear textos en inglés. Todo en español.
- NO dejar console.log de depuración en código final.
- NO usar border-radius cuadrados. Todo es 50px (botones/inputs) o 20px (cards/contenedores).
- NO usar azul corporativo, blanco estéril ni estética genérica de dashboard SaaS.
- Si encontrás ambigüedad, preguntá antes de asumir.

## Comandos útiles

```bash
# Abrir el archivo para verificar visualmente
start plataforma_clinica.html          # Windows
```

---

## SISTEMA DE DISEÑO — IDENTIDAD VISUAL OBLIGATORIA

El estilo es **"clínico-cálido con toque técnico"**. Esta sección es la referencia estética maestra. Seguir al pie de la letra en TODO el proyecto.

### Tipografías

Incluir en `<head>` antes de cualquier `<style>`:

```html
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@600;700;800&family=Nunito:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
```

| Variable | Fuente | Uso |
|----------|--------|-----|
| `--serif` | Playfair Display | Títulos, nombres, page headings, brand |
| `--sans` | Nunito | Cuerpo, labels, botones, UI general |
| `--mono` | DM Mono | IDs, fechas, montos, log, timestamps, códigos |

### Paleta de colores (copiar tal cual en :root)

```css
:root {
  /* VERDES — estructura principal */
  --vd: #1E3D2B;    /* verde oscuro — sidebar, header, botones primarios, cards premium */
  --vm: #2D5A3D;    /* verde medio — bordes activos, hovers, border-left de cards */
  --vc: #4A7C5F;    /* verde claro — focus states, acentos secundarios */
  --vs: #D4E6DA;    /* verde suave — backgrounds de alertas */
  --vu: #EEF5F0;    /* verde ultra claro — fondos de íconos */

  /* DORADOS — acentos y branding */
  --do: #C8922A;    /* dorado principal — items activos, badges, íconos, CTA */
  --dc: #F5E6C8;    /* dorado claro — fondos de alerta cálida */

  /* CREMAS — fondos del área principal */
  --cr: #F5F0E8;    /* crema base — inputs, fondos secundarios */
  --cro: #EDE6D8;   /* crema oscuro — tabs inactivos, separadores */
  --crb: #D8D0C0;   /* crema borde — bordes de cards, inputs */

  /* TEXTOS */
  --tx: #1A2E22;    /* casi negro verde — títulos principales */
  --tx2: #4A6357;   /* gris-verde — subtítulos, valores */
  --tx3: #7A9485;   /* gris suave — labels, hints, timestamps */
  --bl: #FFFFFF;    /* blanco — texto sobre fondos oscuros */

  /* WARM ACCENTS */
  --wb: #FFF8E8;
  --wc: #7A5200;

  /* RADIOS */
  --r: 20px;        /* cards, contenedores grandes */
  --rsm: 12px;      /* elementos pequeños, logs */

  /* FUENTES */
  --serif: 'Playfair Display', serif;
  --sans: 'Nunito', sans-serif;
  --mono: 'DM Mono', monospace;
}

body {
  background: #EDE8DC;
  font-family: var(--sans);
  color: var(--tx);
  font-size: 14px;
  line-height: 1.6;
}
```

### Componentes UI — Especificaciones exactas

#### TOPBAR
```
background: var(--vd); height: 64px; position: sticky; top: 0; z-index: 100;
border-bottom: 2px solid rgba(200,146,42,.4);
```
- Izquierda: ícono circular 42px (fondo --do, border-radius: 50%, box-shadow: 0 0 0 3px rgba(200,146,42,.35), emoji 🌱) + "Plataforma Clínica" en Playfair 22px bold color --do + sub "Hernán Gabriel Stagni · MP 11919 · v1.0" Nunito 12px blanco 75%
- Centro: nombre del módulo activo en Playfair 15px semibold blanco 90%
- Derecha: botón CTA outline (borde blanco 15%, fondo transparente, border-radius: 50px)

#### SIDEBAR
```
width: 240px; background: var(--vd); position: sticky; top: 64px; height: calc(100vh - 64px);
```
- Secciones: label --do, uppercase 11px bold, letter-spacing .07em, border-top: 1px solid rgba(200,146,42,.35)
- PRINCIPAL → Agenda 📅, Pacientes 🌿, Facturación 💰, Planillas AT 📋, Gastos 📊
- SISTEMA → Eventos 🔔, Configuración ⚙️
- Item activo: bg var(--do), color var(--vd), font-weight 700, border-radius 50px
- Item inactivo: color rgba(255,255,255,.78), hover bg rgba(255,255,255,.08)
- Íconos: círculos 28px con fondo rgba(255,255,255,.12)
- Badge: fondo --do, color --vd, DM Mono 10px bold, border-radius 10px
- Pie: "ESTADO DEL SISTEMA" con dots 10px (.on verde con glow, .off gris)

#### CARDS (fondo claro)
```
background: #FAF7F2; border: 2px solid #B8B0A0; border-left: 5px solid var(--vm);
border-radius: var(--r); padding: 22px;
```
- Header: ícono circular 32px (fondo --vu) + título Playfair 16px bold color --vd

#### TARJETAS DE PACIENTE (fondo oscuro — estilo premium)
```
background: var(--vd); border: 2px solid var(--vm); border-radius: var(--r); padding: 18px;
::before { height: 4px; background: linear-gradient(90deg, var(--do), #E8B84B); }
:hover { border-color: var(--do); transform: translateY(-3px); box-shadow: 0 8px 24px rgba(30,61,43,.35); }
```
- Avatar: círculo 42px, fondo --do, color --vd, Playfair 16px bold, box-shadow glow
- Nombre: Playfair 16px color --do, text-shadow oscuro

#### TABLAS
```
thead: bg var(--vd); color rgba(255,255,255,.75); DM Mono 10px uppercase; letter-spacing .08em;
th:first-child { border-radius: 10px 0 0 10px; }
th:last-child { border-radius: 0 10px 10px 0; }
tbody tr: border-bottom: 1px solid #DDD5C4; hover: bg #F0EBE0;
```

#### BOTONES (todos border-radius: 50px, Nunito, font-weight: 600)
| Tipo | Estilos |
|------|---------|
| Primario .btn-p | bg var(--vd), color var(--bl), hover bg var(--vm) |
| Gold .btn-g | bg var(--do), color var(--vd), hover bg #B8821F |
| Secundario | border var(--crb), bg var(--bl), hover border var(--vc) |
| Danger | border rojo suave, texto rojo. NUNCA fondo rojo sólido |
| CTA topbar | border rgba(255,255,255,.25), bg rgba(255,255,255,.1), color blanco 92% |

#### INPUTS
```
border-radius: 50px; padding: 9px 18px; border: 2px solid var(--vm);
background: #F0EBE0; font-family: var(--sans);
:focus { border-color: var(--do); background: #FFF8EC; }
```
- Valores técnicos (montos, IDs, fechas): font-family: var(--mono)

#### MODALES
```
Overlay: bg rgba(30,61,43,.55) — NO negro
Card: bg var(--vd); border: 2px solid var(--vm); border-radius: var(--r);
Barra superior: linear-gradient(90deg, var(--do), #E8B84B), height 4px
Título: Playfair, color var(--do), text-shadow: 0 1px 3px rgba(0,0,0,.4)
```
- Inputs en modales: fondo rgba(255,255,255,.06), borde rgba(200,146,42,.3), color blanco 90%
- Labels en modales: DM Mono 10px, color rgba(255,255,255,.45), uppercase

#### TABS
```
Contenedor: bg var(--cro); border-radius: 50px; padding: 4px;
Activo: bg var(--vd); color var(--bl); bold 700;
Inactivo: color var(--tx2); hover: color var(--vd);
```

#### BADGES
```
border-radius: 50px; padding: 5px 12px; font-weight: 700;
OK:      bg #1E3D2B, color #A8DDB8
Warning: bg #7A5200, color #FDEAAA
Error:   bg #7A2800, color #FFD0B8
```

#### TOASTS
```
position: fixed; bottom-right; bg var(--vd); color crema; border-radius: 12px;
Duración: 3 segundos con barra de progreso dorada animada (CSS transition)
```

#### CHECKBOXES CUSTOM
```
20x20px; border: 2.5px solid var(--do); border-radius: 5px; bg #FAF7F2;
Checked: bg var(--vd); border var(--vd); checkmark '✓' color var(--do);
```

#### LOG / TERMINAL
```
bg var(--vd); border-radius: var(--rsm); DM Mono 12px; line-height: 1.9;
OK: #7DCC90 | Warning: #F0C860 | Info: #7AB8E0 | Timestamps: blanco 25%
```

#### STATS (tarjetas de métricas)
```
bg #FAF7F2; border: 2px solid #B8B0A0; border-radius: var(--r); padding: 18px 20px;
::before { height: 3px; bg var(--do); top: 0; }
Label: DM Mono 10px uppercase, color var(--tx3)
Valor: Playfair 28px bold, color var(--vd)
Grid: repeat(4, 1fr) desktop, 2 tablet, 1 móvil
```

### Responsive (< 768px)

- Sidebar → barra inferior fija con íconos solamente
- Header → solo título del módulo
- Cards → full-width apiladas
- Tablas → overflow-x: auto
- Stats → 2 cols tablet, 1 col móvil

---

## ARQUITECTURA DE DATOS (localStorage)

### Stores

```
hgsp_pacientes      → Array de objetos Paciente
hgsp_sesiones       → Array de objetos Sesión/Turno
hgsp_cobros         → Array de objetos Cobro
hgsp_gastos         → Array de objetos Gasto
hgsp_at_registros   → Array de objetos Registro AT
hgsp_eventos        → Array de objetos Evento
hgsp_config         → Objeto de configuración general
```

### Helpers genéricos

```javascript
function getData(key) { return JSON.parse(localStorage.getItem(key) || '[]'); }
function setData(key, data) { localStorage.setItem(key, JSON.stringify(data)); }
function genId() { return Date.now().toString(36) + Math.random().toString(36).slice(2); }
```

### Esquema: PACIENTE

```
{
  id,
  /* DATOS PERSONALES */
  nombre*, apellido*, fechaNacimiento, dni*, telefono*, email, domicilio, localidad,
  genero: 'masculino'|'femenino'|'otro'|'no_especifica',
  estadoCivil: 'soltero'|'casado'|'divorciado'|'viudo'|'union_convivencial',
  ocupacion, nombreEmergencia, telefonoEmergencia,

  /* COBERTURA */
  cobertura: 'particular'|'obra_social'|'mixto',
  obraSocial, planOS, nroAfiliado*, nroCredencial, vencimientoCredencial,
  cuitTitular, requiereOrden: boolean,
  modalidadAutorizacion: 'online'|'presencial'|'planilla_mensual'|'autorizacion_previa',
  requiereFirma: boolean,

  /* VALORES ECONÓMICOS */
  coseguro: number, valorOrden: number, valorParticular: number,

  /* CLÍNICOS */
  tipo: 'psicologia'|'AT'|'ambos',
  frecuencia: 'semanal'|'quincenal'|'mensual'|'intensiva',
  diasHorarios: [{dia, horaInicio, horaFin}],
  motivoConsulta, derivadoPor, medicacion: boolean, medicacionDetalle,
  diagnosticoCIE,

  /* ESTADO */
  fechaInicio*, fechaAlta: null, motivoBaja, activo: boolean, notas,
  fechaUltimaModificacion: timestamp
}
```
Campos con * son obligatorios. nroAfiliado solo obligatorio si cobertura incluye obra_social.

### Esquema: SESIÓN/TURNO

```
{ id, pacienteId, fecha, horaInicio, horaFin,
  tipo: 'individual'|'AT'|'pareja'|'grupo',
  estado: 'pendiente'|'asistio'|'cancelo'|'reprogramo',
  cobrado: boolean, montoCobrado: number, notas }
```

### Esquema: COBRO

```
{ id, pacienteId, sesionId, fecha,
  concepto: 'particular'|'coseguro'|'liquidacion',
  monto: number, cobrado: boolean,
  metodoPago: 'efectivo'|'transferencia'|'obra_social' }
```

### Esquema: GASTO

```
{ id, mes, anio, categoria, concepto,
  montoEstimado: number, montoReal: number, pagado: boolean }
```

### Esquema: REGISTRO AT

```
{ id, pacienteId, obraSocial, mes, anio,
  sesiones: [{fecha, horaIngreso, horaEgreso, horas, observacion, firmado}],
  totalHoras: number }
```

### Esquema: EVENTO

```
{ id, descripcion, fecha, hora, estado: 'pendiente'|'resuelto' }
```

---

## MÓDULOS (referencia rápida)

El detalle completo de cada módulo está en los prompts de ejecución por fase. Aquí va el resumen para mantener contexto:

| # | Módulo | Descripción |
|---|--------|-------------|
| 1 | Agenda Semanal | Pantalla principal. Grilla L-S, 8:00-20:00, bloques por tipo, modales crear/editar turno, vista mensual toggle |
| 2 | ABM Pacientes | Listado con filtros, formulario 4 secciones, perfil detalle con tabs, baja lógica |
| 3 | Facturación | Liquidaciones OS, cobros particulares, dashboard financiero con Chart.js |
| 4 | Planillas AT | Tabla mensual por paciente AT, cálculo horas, vista imprimible |
| 5 | Gastos Mensuales | Por categoría, estimado vs real, comparativo ingresos vs gastos |
| 6 | Eventos | Lista cronológica, estados, filtros, formulario inline |

---

## DATOS DE EJEMPLO

Cargar SOLO si localStorage está vacío: `if (!localStorage.getItem('hgsp_pacientes')) { ... }`

**Pacientes:**
- Selena Gómez / semanal / Particular / $28.000
- Franco Rodríguez / semanal / Mutual Médica / coseguro $14.000
- Cindi Suárez / quincenal / OSSEG / coseguro $12.000
- Gabino Fernández / mensual / Particular / $28.000
- Alma Morales / AT / ASPURC / por horas
- Ignacio Ceballos / AT / OSSACRA / por horas

**Turnos:** al menos 3 por día laboral de la semana actual, variedad de estados.
**Gastos:** estructura completa del mes actual con valores realistas.
**Evento:** "Facturación mensual OSSACRA" para el día 5 del próximo mes.

## Datos del prestador (para planillas AT y encabezados)

```
Lic. Stagni Hernán Gabriel
MP 11919
CUIT 20.24521983.1
```
