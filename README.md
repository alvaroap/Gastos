# Gastos — Control de gastos personal

Web-app de archivo único (`index.html`) para el control de gastos mensuales a lo largo de un año fiscal. Sin dependencias externas, sin backend, sin build step.

---

## Contexto y propósito

- Control de gastos mes a mes durante 12 meses
- El año fiscal no sigue el calendario natural: **empieza en Feb–Mar y termina en Ene–Feb**
- Los ciclos mensuales van del **día 20 al día 20** del mes siguiente
- Los gastos están repartidos entre dos bancos: **La Caixa** y **BBVA**
- La nómina se cobra en BBVA; desde ahí se transfiere a La Caixa lo necesario para cubrir los gastos domiciliados allí

---

## Lógica de cálculo

### Nómina
Cada mes tiene un campo de nómina independiente. Permite reflejar variaciones por bonus u otros ingresos extraordinarios.

### A La Caixa
Suma de todos los gastos domiciliados en La Caixa ese mes.

### BBVA Gastos Mes (colchón)
Cantidad fija configurable que se reserva en BBVA para gastos no fijos no registrados en la app. Por defecto: **800 €**. Se puede modificar desde la vista de detalle de mes.

### Ahorro / Inversión
```
Ahorro = Nómina − Gastos totales (sin predescontados) − BBVA Gastos Mes
```

### Gastos predescontados
Algunos cargos ya vienen descontados de nómina en origen (ej: Seguro Coche). Se muestran en la lista y suman al total visible, pero **se excluyen del cálculo de Ahorro / Inversión** para evitar doble contabilización. Se marcan con la flag `predescontado: true`.

---

## Estructura de datos

### Meses
12 periodos ordenados del 0 al 11:

| id | Periodo     |
|----|-------------|
| 0  | Feb – Mar   |
| 1  | Mar – Abr   |
| 2  | Abr – May   |
| 3  | May – Jun   |
| 4  | Jun – Jul   |
| 5  | Jul – Ago   |
| 6  | Ago – Sep   |
| 7  | Sep – Oct   |
| 8  | Oct – Nov   |
| 9  | Nov – Dic   |
| 10 | Dic – Ene   |
| 11 | Ene – Feb   |

### Gasto
```js
{
  id:             Number,      // único
  name:           String,      // nombre del cargo
  day:            Number,      // día del mes en que se carga
  cat:            String,      // categoría (ver abajo)
  bank:           String,      // 'caixa' | 'bbva'
  predescontado:  Boolean,     // opcional, default false
  amounts:        Number[12],  // importe por mes, índice = id de mes
}
```

### Estado global (`S`)
```js
{
  expenses: Gasto[],
  nominas:  Number[12],   // nómina por mes
  colchon:  Number,       // BBVA Gastos Mes (global, aplica a todos los meses)
  nextId:   Number,
}
```
Persistido en `localStorage` bajo la clave `gastos_v5`.

---

## Categorización

### La Caixa

| Categoría | Cargos |
|-----------|--------|
| Casa      | Hipoteca, Comunidad, IBI, Vodafone, Iberdrola, Seviam Plus, Vida Familiar, Segurcaixa |
| Seguros   | Sanitas (Navarra), Seguro Coche _(predescontado)_ |
| Colegio   | Colegio ayuda, Colegio, Libros |
| ONGs      | Anesvad, El Refugio, Cruz Roja, MSF, WWF Adena |
| Banca     | Visa Electron |
| Otros     | Google Cloud, Spotify |

### BBVA

| Categoría | Cargos |
|-----------|--------|
| Otros     | DAZN, Amazon Prime, Apple, Play Station Network, Ayto. Coche, Piscina, Clases Inglés |

---

## Vistas

### Vista general
- Grid de tarjetas, una por mes, máximo **2 columnas**
- Cada tarjeta muestra: nombre del periodo, total de gastos, campo de nómina editable, subtotal La Caixa y Ahorro / Inversión calculado
- El campo de nómina es editable directamente en la tarjeta

### Vista de mes (detalle)
- KPI strip: Nómina / Total gastos / A La Caixa / Ahorro–Inversión
- Barra de nómina + BBVA Gastos Mes (colchón)
- Lista de gastos agrupada por **banco → categoría**
- Separador visual más oscuro entre categorías
- Edición inline de importes: tap/click sobre la cifra, Enter o Tab para confirmar, Escape para cancelar
- Botón de añadir cargo al pie de la lista

---

## Funcionalidades

- **Edición inline** de cualquier importe en la vista de mes
- **Añadir cargo** desde el header (desktop) o el footer (móvil)
- **Eliminar cargo** (afecta a todos los meses)
- **Guardar** estado en localStorage
- **Exportar CSV** compatible con Numbers/Excel (separador `;`, decimales con coma, BOM UTF-8). Exporta: todos los cargos con banco/categoría/día/importes por mes + fila de nóminas + BBVA Gastos Mes + totales + ahorro por mes

---

## Diseño

### Tipografía
**Plus Jakarta Sans** — weights 300 (light) y 400 (regular) únicamente, más 800 (bold) para énfasis de datos.

```html
<link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400&display=swap" rel="stylesheet">
```

### Paleta de color base (escala cálida `#edebe6`)

| Token        | Valor     | Uso |
|--------------|-----------|-----|
| `--grey-50`  | `#f5f3ee` | Fondos de superficies, sidebar |
| `--grey-100` | `#ece9e2` | Fondos de inputs, nomina-cur |
| `--grey-200` | `#edebe6` | Bordes suaves (color base ★) |
| `--grey-300` | `#d8d5ce` | Bordes fuertes, bank-section bg |
| `--grey-400` | `#b8b4ac` | Bordes bank-section |
| `--grey-500` | `#6e6a65` | Texto faint |
| `--grey-600` | `#484440` | Texto muted, labels |
| `--grey-900` | `#1e1c1a` | Texto principal |

### Colores semánticos

| Token            | Valor     | Uso |
|------------------|-----------|-----|
| `--accent-green` | `#38b820` | Ahorro positivo |
| `--accent-red`   | `#dc2626` | Gastos, inversión negativa |
| `--accent-blue`  | `#2563eb` | Inversión positiva (KPI detalle) |

### Colores de categoría

| Categoría | Color     |
|-----------|-----------|
| Casa      | `#d97706` |
| Colegio   | `#0284c7` |
| Seguros   | `#7c3aed` |
| ONGs      | `#059669` |
| Banca     | `#9ca3af` |
| Otros     | `#9ca3af` |

### Bank section
Estilo unificado para La Caixa y BBVA:

| Elemento   | Valor     |
|------------|-----------|
| background | `#d8d5ce` |
| border     | `#b8b4ac` |
| dot        | `#2c2a28` |

### Tokens de diseño

```css
--font:           'Plus Jakarta Sans', sans-serif;
--fw-light:       300;
--fw-regular:     400;
--fw-bold:        800;

--text-xs:        11px;
--text-sm:        12px;
--text-base:      14px;
--text-md:        16px;
--text-lg:        20px;
--text-xl:        26px;

--sp-1 … --sp-8:  4px … 32px;

--r-sm:   6px;
--r-md:   10px;
--r-lg:   16px;
--r-xl:   22px;
--r-full: 999px;
```

---

## Layout y responsive

- Contenedor principal: `max-width: 980px`, centrado con `margin: 0 auto`
- **Mobile** (`< 768px`): bottom nav con 3 ítems (General / Mes / Añadir), sin sidebar, modal como sheet inferior
- **Desktop** (`≥ 768px`): sidebar fija izquierda (216px) + contenido principal, sin bottom nav
- Botón "+ Cargo" en header solo visible en desktop
- `safe-area-inset-bottom` aplicado para iPhone con notch

---

## Persistencia y exportación

- Datos guardados en `localStorage` (clave `gastos_v5`)
- Al cambiar la estructura de datos se debe incrementar la versión de la clave para evitar conflictos con datos cacheados
- Exportación a CSV con separador `;` y BOM UTF-8 para compatibilidad con Numbers en macOS en locale `es-ES`

---

## Ficheros

```
gastos.html   — app completa (HTML + CSS + JS en un único archivo)
README.md     — este documento
```

---

## Despliegue

Alojado en GitHub Pages desde el repositorio `alvaroap/Gastos`.
URL: `https://alvaroap.github.io/Gastos/gastos.html`
