# Manifiesto Técnico — Arquitectura de Referencia del Taller
## Brief Técnico común para los casos A y B

| Metadato | Valor |
|---|---|
| **Tipo de documento** | Brief Técnico (definición de alto nivel de arquitectura) |
| **Aplica a** | Caso A `ORI-LI` · Caso B `PRC-ENG` |
| **Arquitectura** | Front estático de una página + Back Node.js/Express + Data source en archivos JSON |
| **Ejecución** | Ambiente web de Claude Code provisto por AWS, puerto `3000` |
| **Punto de partida** | Ambiente vacío: todo se crea desde cero durante el taller |

> Este documento define **restricciones y decisiones de arquitectura**, no el diseño detallado. El diseño detallado (contratos, funciones, pruebas) lo construye cada participante en su **Spec Técnico**.

---

## 1. Principios de arquitectura

1. **Arquitectura simple, experiencia completa:** un solo proceso Node.js sirve el front y la API, construible desde cero en 35 minutos; la aplicación se ve y se siente como un producto real.
2. **Dominio puro y probado:** las reglas de negocio viven en funciones puras, sin acceso a HTTP ni a archivos, con pruebas unitarias.
3. **Back como fuente de verdad:** el front valida para dar buena experiencia, pero el back vuelve a validar todo.
4. **Datos parametrizados, no quemados:** tasas, umbrales y tablas se leen de JSON; el código no contiene valores de política.
5. **Trazabilidad:** código y pruebas citan los IDs del spec (`// RN-04`, `// CA-07`).
6. **Cero dependencias innecesarias:** la única dependencia externa permitida es `express`.
7. **Front y back separados:** el front vive en `frontend/` y el back en `backend/`; solo se comunican por la API `/api`.
8. **Buenas prácticas obligatorias:** SOLID, criterio de microservicio y cero código espagueti, según la sección 8.

## 2. Vista de alto nivel

```
┌──────────────────────────────────┐
│  Navegador                       │
│  frontend/ (SPA, varias vistas)  │
└────────────────┬─────────────────┘
                 │ fetch JSON  /api/*
┌────────────────▼─────────────────┐
│  backend/ · microservicio Node.js│
│                                  │
│  server.js      arranque         │
│      │                           │
│  routes.js      HTTP + validación│
│      │                 │         │
│  domain.js      repository.js    │
│  reglas puras   lectura/escritura│
└────────────────────────┬─────────┘
                         │ fs
┌────────────────────────▼─────────┐
│  backend/data/*.json             │
│  parámetros · maestro · registro │
└──────────────────────────────────┘
```

Las dependencias van en una sola dirección: `routes` usa a `domain` y a `repository`; `domain` no conoce a nadie.

## 3. Front

| Aspecto | Definición |
|---|---|
| Tecnología | HTML5 + CSS + JavaScript vanilla con módulos ES. **Sin frameworks, sin librerías y sin build.** |
| Ubicación | Carpeta `frontend/`, separada del back. Express la sirve como estática |
| Navegación | **Aplicación de una sola página (SPA)** con barra de navegación entre las vistas del caso, sin recargar la página |
| Estados de UI | Cada acción muestra estado de carga, éxito y error; las listas vacías muestran un mensaje amigable |
| Formato | Moneda en pesos colombianos con formato `es-CO` (`$1.234.567`); tasas con coma decimal; fechas `dd/mm/aaaa hh:mm` |

### 3.1 Lineamientos de experiencia de usuario

La app debe verse como un **producto bancario real**, no como un formulario de prueba.

| Elemento | Lineamiento |
|---|---|
| **Encabezado** | Barra superior con el nombre del producto, "Banco Demo" y la navegación entre vistas, con la vista activa resaltada |
| **Layout** | Contenido centrado (máx. ~1100 px), organizado en **tarjetas** con título, sombra suave y bordes redondeados |
| **Indicadores** | Los totales y conteos se muestran como **tarjetas KPI** (número grande y etiqueta) |
| **Estados de negocio** | **Insignias de color:** verde aprobado/aceptado, ámbar revisar, rojo rechazado, gris no evaluable o desistido |
| **Gráficos** | Barras, medidores y cascadas hechos con **CSS o SVG nativo**, sin librerías |
| **Tablas** | Encabezado fijo, filas alternadas, números alineados a la derecha |
| **Formularios** | Etiqueta en cada campo, validación en línea con el mensaje junto al campo y botón principal destacado |
| **Responsive** | Se usa bien desde 360 px de ancho; en pantallas pequeñas las tarjetas se apilan |
| **Accesibilidad** | Contraste AA, foco visible y el color nunca es el único indicador (la insignia también lleva texto) |
| **Paleta (sugerida)** | Fondo `#F5F7FA`, tarjetas blancas, estructura en azul marino `#213E7F`, acento `#0097D9`, textos `#1A1A1A` / `#4D4D4D`, bordes `#E3E7ED`; verde `#1E8E3E`, ámbar `#E8A100`, rojo `#D93025`, gris `#80868B`. Tipografía del sistema (`system-ui`, Arial). |

## 4. Back

| Aspecto | Definición |
|---|---|
| Runtime | Node.js 20 o superior, módulos ES (`"type": "module"`) |
| Framework | Express 4 (única dependencia) |
| Estilo de API | REST + JSON bajo el prefijo `/api` |
| Organización | En `backend/src/`: `server.js` (arranque) · `routes.js` (HTTP y validación) · `domain.js` (reglas puras) · `repository.js` (acceso a JSON) |
| Códigos HTTP | `200` OK · `201` registro creado · `400` entrada inválida · `404` no encontrado |
| Modelo de error | `{ "codigo": "VAL-01", "mensaje": "texto claro" }` |

## 5. Data source (archivos JSON)

| Aspecto | Definición |
|---|---|
| Ubicación | Carpeta `backend/data/` (los datos pertenecen al back) |
| Tipos de archivo | **Parámetros** (política, solo lectura) · **Maestro** (clientes, solo lectura) · **Registro** (solicitudes o cotizaciones, se agrega al final) |
| IDs | Secuenciales con prefijo y 4 dígitos (`SOL-0001`, `COT-0001`), calculados a partir del último registro |
| Fechas | ISO 8601 (`2026-10-15T14:30:00.000Z`) |
| Semillas | Se crean **exactamente** con el contenido del anexo de este documento; no se modifican |

## 6. Estructura de proyecto

```
taller-sdd-<caso>/
├── CLAUDE.md          # reglas del proyecto para Claude Code
├── package.json       # único del proyecto; scripts: start, test
├── kit/               # manifiestos entregados (Briefs)
├── specs/             # spec funcional, spec técnico, plan, UAT
├── frontend/              # todo lo que corre en el navegador
│   ├── index.html         # estructura, encabezado y contenedor de vistas
│   ├── styles.css         # estilos y componentes visuales
│   └── js/
│       ├── app.js         # arranque y navegación entre vistas
│       ├── api.js         # llamadas fetch a /api
│       ├── formato.js     # moneda, tasas y fechas
│       └── vistas/        # un archivo por vista
└── backend/               # el microservicio: API, reglas y datos
    ├── src/
    │   ├── server.js      # arranque: Express, sirve frontend/, middleware de errores, puerto
    │   ├── routes.js      # endpoints: validan entrada, llaman al dominio y al repositorio
    │   ├── domain.js      # reglas de negocio puras
    │   └── repository.js  # lectura y escritura de backend/data/*.json
    ├── data/              # archivos JSON (anexo de su caso)
    └── test/
        └── domain.test.js # pruebas node:test del dominio
```

## 7. Calidad y pruebas

- Pruebas unitarias con el runner nativo **`node:test`** y `node:assert` (sin dependencias).
- Mínimo: **una prueba por cada escenario** del manifiesto funcional que involucre reglas, ejecutada contra las funciones del dominio.
- Para valores de cuota se acepta una **tolerancia de ±1%**; tasas, puntajes y estados deben ser exactos.
- Scripts obligatorios en `package.json`:
  - `npm start` → `node backend/src/server.js`
  - `npm test` → `node --test backend/test/`

## 8. Buenas prácticas de desarrollo

> Son obligatorias y se verifican en el **Gate 2** (el Spec Técnico debe reflejarlas) y al cerrar la implementación. Se aplican con criterio KISS: lo mínimo que mantiene el código limpio, sin sobreingeniería.

### 8.1 SOLID aplicado a módulos y funciones de JavaScript

| Principio | Qué significa en este proyecto |
|---|---|
| **S** · Responsabilidad única | Cada módulo tiene un solo motivo para cambiar: `server.js` arranca, `routes.js` traduce HTTP, `domain.js` decide, `repository.js` persiste. Cada función hace una sola cosa. |
| **O** · Abierto/cerrado | Una regla o variable nueva se agrega en el JSON de parámetros o como una función más en una lista de reglas, sin modificar las existentes. Ejemplo: las reglas de evaluación son un arreglo de funciones que se recorre en orden. |
| **L** · Sustitución de Liskov | Las funciones del mismo tipo comparten firma y forma de resultado (por ejemplo, toda regla devuelve `{ cumple, motivo }`), así que son intercambiables. |
| **I** · Segregación de interfaces | Cada función recibe solo los datos que necesita: el dominio no recibe el `request` de Express ni el archivo JSON completo. |
| **D** · Inversión de dependencias | El dominio no importa `express` ni `fs`: recibe los parámetros como argumentos. `routes.js` recibe el repositorio por parámetro (`crearRutas({ repositorio })`), lo que permite probar sin tocar archivos. |

### 8.2 Criterio de microservicio

Cada caso se construye como **un microservicio autocontenido**. Con criterio KISS, no se divide en varios procesos dentro del taller: el criterio se aplica al servicio completo.

| Característica | Cómo se cumple |
|---|---|
| **Contexto acotado** | El servicio resuelve un solo contexto de negocio (originación o pricing) y no asume responsabilidades de otro. |
| **Dueño de sus datos** | Solo el back lee y escribe `backend/data/`; el front nunca accede a los datos directamente. |
| **Contrato primero** | La API se define en el Spec Técnico antes del código, y el código respeta ese contrato. |
| **Sin estado en memoria** | Todo estado vive en `backend/data/`. Reiniciar el proceso no pierde nada, lo que permite escalar horizontalmente (en AWS: Lambda o contenedores). |
| **Configuración externa** | Puerto por variable `PORT`; valores de política en JSON; nada de configuración dentro del código. |
| **Salud observable** | `GET /api/health` responde `{ "estado": "OK" }`. |
| **Despliegue independiente** | Arranca con un solo `npm start`, sin depender de otros servicios. |

### 8.3 Cero código espagueti

- **Flujo en una sola dirección:** `routes` → `domain` y `routes` → `repository`. El dominio nunca llama hacia arriba y no hay dependencias circulares.
- **Funciones cortas:** como guía, máximo 30 líneas y 3 niveles de anidamiento. Se prefieren retornos tempranos a `if` anidados.
- **Sin lógica de negocio fuera del dominio:** ni en las rutas ni en el front. El front solo captura datos y presenta resultados.
- **Sin estado global mutable** ni variables compartidas entre peticiones.
- **Sin números mágicos:** todo valor de política sale del JSON; las demás constantes tienen nombre.
- **Sin duplicación:** los cálculos comunes (cuota, redondeo, búsqueda en rangos) viven en una sola función.
- **Sin código muerto** ni bloques comentados.

### 8.4 Prácticas mínimas adicionales

| Práctica | Regla |
|---|---|
| **Nombres** | En español y del lenguaje del negocio (`calcularCuota`, `evaluarSolicitud`). `camelCase` en funciones y variables, `MAYUSCULAS_CON_GUION` en constantes. |
| **Validación en el borde** | Toda entrada se valida en `routes.js` antes de llegar al dominio y falla rápido con el modelo de error estándar. |
| **Errores centralizados** | Un único middleware de errores en `server.js`. Nunca se muestra un *stack trace* al usuario. |
| **Inmutabilidad** | Las funciones de dominio no modifican sus argumentos: devuelven objetos nuevos. |
| **Precisión numérica** | Se calcula con todos los decimales y se redondea solo al final, según la regla del spec. |
| **Seguridad básica** | El back no confía en el front. En el navegador se usa `textContent`, no `innerHTML`, para mostrar datos. Nada de `eval`. |
| **Logs** | Una línea por petición (método, ruta, código, duración), sin datos personales. |
| **Pruebas** | Estructura Preparar / Ejecutar / Verificar. El nombre de cada prueba cita su CA y escenario. Son independientes entre sí y no tocan los archivos reales. |
| **Comentarios** | Solo el *porqué* y los IDs de trazabilidad (`// RN-06`). El *qué* lo explica el código. |

### 8.5 Checklist de buenas prácticas

- [ ] Las reglas de negocio (`domain.js`) no dependen del servidor web ni de los archivos: reciben los datos como parámetros y devuelven un resultado. En ese archivo no aparece `import express` ni `import fs`.
- [ ] Front y back están en carpetas separadas (`frontend/` y `backend/`) y solo se comunican por la API `/api`: ningún archivo de una carpeta importa archivos de la otra.
- [ ] Ninguna regla de negocio vive en `routes.js` ni en `frontend/`.
- [ ] Cada vista del front vive en su propio archivo dentro de `frontend/js/vistas/`.
- [ ] Ningún valor de política está escrito en el código.
- [ ] Ninguna función supera ~30 líneas ni 3 niveles de anidamiento.
- [ ] `GET /api/health` responde y el servicio no guarda estado en memoria.
- [ ] Las pruebas del dominio corren sin leer ni escribir archivos.

## 9. Restricciones no funcionales

| Restricción | Valor |
|---|---|
| Ejecución | Ambiente web de Claude Code, puerto 3000 (configurable por variable `PORT`) |
| Seguridad | Sin autenticación (fuera de alcance). Solo datos ficticios; prohibido usar datos personales reales |
| Dependencias | Solo `express` |
| Persistencia | Solo archivos JSON; sin bases de datos |
| Idioma | Español en interfaz, mensajes, documentos y comentarios |

**Modo sin dependencias (contingencia):** si el registro npm no está disponible, se reemplaza Express por el módulo nativo `node:http` con un enrutador mínimo propio. El resto de la arquitectura no cambia.

## 10. API mínima sugerida por caso

> Es una guía para mantener el alcance; el Spec Técnico define el contrato final.

| Caso | Endpoints sugeridos |
|---|---|
| Todos | `GET /api/health` (salud del servicio) |
| A · Originación | `POST /api/simulaciones` · `POST /api/solicitudes` (evalúa, registra y devuelve plan de pagos) · `PATCH /api/solicitudes/:id` (aceptar/desistir) · `GET /api/solicitudes` (panel con indicadores) |
| B · Pricing | `POST /api/cotizaciones` (calcula y registra) · `POST /api/comparaciones` (todos los plazos, sin registrar) · `GET /api/cotizaciones?producto=` · `GET /api/parametros` |

## 11. Referencia de evolución a AWS (fuera del alcance del taller)

Cómo evolucionaría esta arquitectura hacia producción en AWS, manteniendo el spec como contrato:

| Capa del taller | Evolución en AWS |
|---|---|
| `frontend/` | Amazon S3 + Amazon CloudFront |
| Express `/api` | Amazon API Gateway + AWS Lambda (o Amazon ECS en AWS Fargate) |
| Registros JSON | Amazon DynamoDB |
| Parámetros JSON | AWS AppConfig o AWS Systems Manager Parameter Store |
| Logs de consola | Amazon CloudWatch |
| Asistente de desarrollo | Claude Code con modelos Claude en Amazon Bedrock |

---

# Anexos — Datos semilla por caso

> Cada participante usa **solo el anexo de su caso**. Los archivos se crean en `backend/data/` con el nombre indicado y el contenido exacto.

## Anexo A — Caso A · Originación de Crédito de Libre Inversión

### `backend/data/parametros-producto.json`
```json
{
  "versionPolitica": "ORI-2026-10-v1",
  "producto": "LIBRE_INVERSION",
  "montoMin": 1000000,
  "montoMax": 50000000,
  "multiploMonto": 100000,
  "plazosMeses": [12, 24, 36, 48, 60],
  "edadMin": 18,
  "edadMax": 75,
  "scoreMinimo": 600,
  "porcentajeCapacidad": 0.40,
  "tasaReferencialEA": 26.0,
  "bandasTasa": [
    { "scoreDesde": 800, "scoreHasta": 999, "tasaEA": 18.0 },
    { "scoreDesde": 700, "scoreHasta": 799, "tasaEA": 22.0 },
    { "scoreDesde": 600, "scoreHasta": 699, "tasaEA": 26.0 }
  ],
  "motivos": {
    "NE-01": "Cliente sin información en la base crediticia",
    "R-01": "Reportado en listas restrictivas",
    "R-02": "Edad fuera de política",
    "R-03": "Score inferior al mínimo",
    "R-04": "Sin capacidad de pago"
  }
}
```

### `backend/data/clientes.json`
```json
[
  { "cedula": "1001", "nombre": "Ana María Gómez", "fechaNacimiento": "1990-05-14", "scoreBuro": 820, "obligacionesMensuales": 500000, "listasRestrictivas": false },
  { "cedula": "1002", "nombre": "Carlos Andrés Ruiz", "fechaNacimiento": "1985-11-02", "scoreBuro": 720, "obligacionesMensuales": 1200000, "listasRestrictivas": false },
  { "cedula": "1003", "nombre": "Laura Pérez", "fechaNacimiento": "1995-02-20", "scoreBuro": 640, "obligacionesMensuales": 300000, "listasRestrictivas": false },
  { "cedula": "1004", "nombre": "Jorge Díaz", "fechaNacimiento": "1988-07-09", "scoreBuro": 550, "obligacionesMensuales": 400000, "listasRestrictivas": false },
  { "cedula": "1005", "nombre": "Marta León", "fechaNacimiento": "1979-01-30", "scoreBuro": 780, "obligacionesMensuales": 200000, "listasRestrictivas": true },
  { "cedula": "1006", "nombre": "Pedro Salas", "fechaNacimiento": "1948-03-10", "scoreBuro": 760, "obligacionesMensuales": 0, "listasRestrictivas": false }
]
```

### `backend/data/solicitudes.json`
```json
[]
```

---

## Anexo B — Caso B · Pricing Engine

### `backend/data/parametros-pricing.json`
```json
{
  "versionTabla": "PRC-2026-10-v1",
  "productos": [
    { "codigo": "LIBRE_INVERSION", "nombre": "Crédito Libre Inversión", "tasaBaseEA": 20.0, "tasaPisoEA": 14.0, "tasaTopeEA": 25.0, "montoMin": 1000000, "montoMax": 80000000 },
    { "codigo": "VEHICULO", "nombre": "Crédito de Vehículo", "tasaBaseEA": 15.0, "tasaPisoEA": 14.0, "tasaTopeEA": 22.0, "montoMin": 10000000, "montoMax": 200000000 }
  ],
  "multiploMonto": 100000,
  "plazosMeses": [12, 24, 36, 48, 60, 72],
  "canales": ["DIGITAL", "OFICINA"],
  "spreadRiesgo": { "A": 0.0, "B": 2.5, "C": 5.0 },
  "bandasNoCotizables": ["D", "E"],
  "ajustePlazo": [
    { "desdeMeses": 12, "hastaMeses": 24, "puntos": 0.0 },
    { "desdeMeses": 36, "hastaMeses": 48, "puntos": 0.5 },
    { "desdeMeses": 60, "hastaMeses": 72, "puntos": 1.0 }
  ],
  "descuentos": {
    "nomina": 1.0,
    "canalDigital": 0.5
  },
  "vigenciaCotizacionDias": 15
}
```

### `backend/data/cotizaciones.json`
```json
[]
```

---

*Manifiesto Técnico del taller SDD · GFT Technologies & AWS · Los datos son ficticios y de uso exclusivo para el taller.*
