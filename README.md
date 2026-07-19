# Plataforma de Gestión y Monitoreo Ambiental en Tiempo Real

Dashboard web interactivo de calidad del aire que consume datos reales de la
**API pública de OpenAQ (v3)**, muestra las estaciones sobre un mapa mundial
integrado con Google Maps, y permite explorar en detalle los sensores,
proveedor, tipo de estación y gráficos resumidos de cada ubicación.

## 1. Restricción tecnológica del proyecto

> **Observación del cliente:** *"No usar librerías JavaScript. No usar
> frameworks. Solo usar HTML y CSS."*

Por esta razón, **a diferencia de lo sugerido en el enunciado general
(Vue.js/React)**, esta entrega se construyó deliberadamente **sin ningún
framework ni librería de terceros** (nada de Vue, React, jQuery, Chart.js,
Bootstrap, Tailwind, ni el SDK de Google Maps JavaScript). Los únicos
archivos de código fuente son:

- `index.html` — estructura semántica de la aplicación **y** la lógica de
  interacción/consumo de la API, escrita en **JavaScript nativo (Vanilla
  JS)** dentro de una única etiqueta `<script>`. No se usa JavaScript como
  "framework" — es el mínimo indispensable para hacer `fetch()` a la API,
  parsear el JSON y actualizar el DOM, tal como cualquier página HTML
  necesita para ser dinámica.
- `styles.css` — toda la apariencia visual, el layout (Flexbox/CSS Grid) y
  la responsividad, sin ningún framework CSS.

El mapa mundial se integra mediante un `<iframe>` público de Google Maps
(sin necesidad de clave de API de Google), y los marcadores de cada estación
se dibujan como elementos HTML/CSS posicionados con una proyección
**Web Mercator** calculada en JavaScript nativo, para que coincidan
razonablemente con la vista del mapa.

## 2. Fuente de datos: OpenAQ API v3

La aplicación consume directamente estos endpoints reales:

| Endpoint | Uso en el dashboard |
|---|---|
| `GET /v3/locations` | Lista de estaciones/ubicaciones de monitoreo (nombre, país, proveedor, coordenadas, sensores disponibles) |
| `GET /v3/locations/{id}/latest` | Última lectura por sensor de una estación, usada para el promedio global de PM2.5 y las alertas |
| `GET /v3/sensors/{id}/hours` | Promedios horarios de las últimas 24h por sensor, usados en las mini-gráficas del panel de detalle |

OpenAQ v3 requiere autenticación mediante el header `X-API-Key` en cada
petición. **La clave es gratuita**: se obtiene registrándose en
`https://explore.openaq.org/register`. Al abrir la aplicación, el usuario
debe pegar su clave en el banner superior ("Conectar y cargar datos"); no se
guarda en ningún servidor, solo vive en memoria durante la sesión del
navegador.

### Modo de demostración sin conexión

Como este es un proyecto 100% del lado del cliente (sin backend propio) y la
API de OpenAQ requiere una clave personal, se incluyó un botón **"Usar datos
de ejemplo"** que carga un conjunto de datos simulado con la **misma
estructura exacta** que devuelve la API real (mismos nombres de campos:
`id`, `name`, `country`, `provider`, `coordinates`, `sensors[].parameter`,
etc.). Esto permite evaluar y demostrar toda la interfaz sin depender de una
clave o de conexión a internet, dejando claro en pantalla que se trata de
datos de ejemplo y no de la API en vivo.

## 3. Funcionalidades implementadas

- **Tarjetas del dashboard**: total de estaciones cargadas, promedio global
  de PM2.5 (calculado sobre una muestra de estaciones con lectura reciente,
  visualizado con un anillo tipo gauge hecho con `conic-gradient` de CSS
  puro) y contador de alertas activas (estaciones que superan el umbral de
  55 µg/m³ de PM2.5).
- **Mapa mundial** con un marcador por estación, coloreado según el nivel de
  calidad del aire (bueno / moderado / deficiente), navegable con zoom y
  totalmente accesible por teclado.
- **Panel de detalle** (lateral derecho) que se abre al hacer clic en un
  marcador o en una fila de la tabla, mostrando: nombre y país de la
  estación, proveedor, tipo de estación, lista de sensores disponibles con
  sus unidades, y **gráficos resumidos de las últimas 24 horas** (barras
  SVG generadas dinámicamente, sin ninguna librería de gráficos).
- **Tabla de estaciones IoT** con filtro de búsqueda en vivo por ciudad,
  país o ID, y estado de conexión (activo / sin datos).
- **Manejo de errores**: si la clave de API no es válida (HTTP 401) o falla
  la red, se muestra un aviso claro con opción de reintentar.
- **Accesibilidad (HCI):** roles y `aria-label` en mapa, marcadores, tabla y
  panel; navegación completa por teclado (Tab, Enter/Espacio, Escape para
  cerrar el panel); foco visible (`:focus-visible`); anuncios `aria-live`
  para el conteo de estaciones y alertas; respeto a
  `prefers-reduced-motion`; diseño responsivo (menú lateral colapsable en
  pantallas pequeñas).

## 4. Estructura de archivos

```
/
├── index.html   → estructura 
├── styles.css   → estilos, layout y responsividad
└── README.md    → este documento
```

## 5. Cómo ejecutarlo

1. Descargar los tres archivos en una misma carpeta.
2. Abrir `index.html` directamente en el navegador (Chrome, Edge o Firefox).
3. Pegar una clave de API de OpenAQ (gratuita, ver sección 2) y presionar
   **"Conectar y cargar datos"**, o presionar **"Usar datos de ejemplo"**
   para ver la interfaz sin clave.


## 6. Limitaciones conocidas / posibles mejoras futuras

- El mapa usa un `<iframe>` público de Google Maps en lugar del SDK oficial
  *Google Maps JavaScript API*, precisamente para no depender de una
  librería externa ni de una clave de Google. Como consecuencia, la
  posición de los marcadores es una aproximación (proyección Web Mercator
  calculada a mano) y no está sincronizada en tiempo real con el
  paneo/zoom nativo de Google dentro del iframe.
- El promedio global de PM2.5 se calcula sobre una muestra (hasta 12
  estaciones) para no exceder límites razonables de peticiones a la API en
  el navegador; para un promedio exhaustivo sería recomendable un proceso
  de agregación en un backend.
- Los umbrales de "moderado" (35 µg/m³) y "deficiente" (55 µg/m³) para
  PM2.5 son valores de referencia orientativos (escala EPA/OMS
  simplificada), no un índice AQI oficial completo.
