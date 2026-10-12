# Manifiesto Técnico — Arquitectura de Referencia del Taller
## Brief Técnico común para los casos A, B y C

| Metadato | Valor |
|---|---|
| **Tipo de documento** | Brief Técnico (definición de alto nivel de arquitectura) |
| **Aplica a** | Caso A `ORI-LI` · Caso B `PRC-ENG` · Caso C `RSK-PER` |
| **Arquitectura** | Front estático de una página + Back Node.js/Express + Data source en archivos JSON |
| **Ejecución** | Ambiente web de Claude Code provisto por AWS, puerto `3000` |
| **Punto de partida** | Ambiente vacío: todo se crea desde cero durante el taller |

> Este documento define **restricciones y decisiones de arquitectura**, no el diseño detallado. El diseño detallado (contratos, funciones, pruebas) lo construye cada participante en su **Spec Técnico**.

---

## 1. Principios de arquitectura (KISS)

1. **Simple y construible desde cero en 35 minutos:** un solo proceso Node.js sirve el front y la API.
2. **Dominio puro y probado:** las reglas de negocio viven en funciones puras, sin acceso a HTTP ni a archivos, con pruebas unitarias.
3. **Back como fuente de verdad:** el front valida para dar buena experiencia, pero el back vuelve a validar todo.
4. **Datos parametrizados, no quemados:** tasas, umbrales y tablas se leen de JSON; el código no contiene valores de política.
5. **Trazabilidad:** código y pruebas citan los IDs del spec (`// RN-04`, `// CA-07`).
6. **Cero dependencias innecesarias:** la única dependencia externa permitida es `express`.

## 2. Vista de alto nivel

```
┌──────────────────────────────┐
│  Navegador                   │
│  public/ (una sola página)   │
└──────────────┬───────────────┘
               │ fetch JSON  /api/*
┌──────────────▼───────────────┐
│  src/server.js · Express     │
│  rutas, validación, lectura  │
│  y escritura de JSON         │
│          │                   │
│  src/domain.js               │
│  reglas de negocio (puras)   │
└──────────────┬───────────────┘
               │ fs
┌──────────────▼───────────────┐
│  data/*.json                 │
│  parámetros · maestro · registro │
└──────────────────────────────┘
```

## 3. Front

| Aspecto | Definición |
|---|---|
| Tecnología | HTML5 + CSS + JavaScript vanilla. **Sin frameworks ni build.** |
| Servido por | Express como estático desde `public/` |
| Pantallas | **Una sola página** (`index.html`): formulario + resultado |
| Estados de UI | Cada acción muestra estado de carga, éxito y error |
| Formato | Moneda en pesos colombianos con formato `es-CO` (`$1.234.567`); tasas con coma decimal |
| Look & feel (sugerido) | Fondo blanco, Arial, estructura en azul marino `#213E7F`, un único acento `#0097D9`, textos `#000000` / `#4D4D4D`, bordes finos `#EDEDED` |

## 4. Back

| Aspecto | Definición |
|---|---|
| Runtime | Node.js 20 o superior, módulos ES (`"type": "module"`) |
| Framework | Express 4 (única dependencia) |
| Estilo de API | REST + JSON bajo el prefijo `/api` |
| Organización | `src/server.js` (rutas, validación y acceso a JSON) + `src/domain.js` (reglas puras) |
| Códigos HTTP | `200` OK · `201` registro creado · `400` entrada inválida · `404` no encontrado |
| Modelo de error | `{ "codigo": "VAL-01", "mensaje": "texto claro" }` |

## 5. Data source (archivos JSON)

| Aspecto | Definición |
|---|---|
| Ubicación | Carpeta `data/` en la raíz del proyecto |
| Tipos de archivo | **Parámetros** (política, solo lectura) · **Maestro** (clientes/solicitantes, solo lectura) · **Registro** (solicitudes/cotizaciones/evaluaciones, se agrega al final) |
| IDs | Secuenciales con prefijo y 4 dígitos (`SOL-0001`, `COT-0001`, `EVA-0001`), calculados a partir del último registro |
| Fechas | ISO 8601 (`2026-10-15T14:30:00.000Z`) |
| Semillas | Se crean **exactamente** con el contenido del anexo de este documento; no se modifican |

## 6. Estructura de proyecto

```
taller-sdd-<caso>/
├── CLAUDE.md          # reglas del proyecto para Claude Code
├── package.json       # scripts: start, test
├── kit/               # manifiestos entregados (Briefs)
├── specs/             # spec funcional, spec técnico, plan, UAT
├── data/              # archivos JSON (anexo de su caso)
├── public/            # index.html, styles.css, app.js
├── src/
│   ├── server.js      # Express: estáticos, rutas, lectura/escritura de JSON
│   └── domain.js      # reglas de negocio puras
└── test/
    └── domain.test.js # pruebas node:test del dominio
```

## 7. Calidad y pruebas

- Pruebas unitarias con el runner nativo **`node:test`** y `node:assert` (sin dependencias).
- Mínimo: **una prueba por cada escenario** del manifiesto funcional que involucre reglas, ejecutada contra las funciones del dominio.
- Para valores de cuota se acepta una **tolerancia de ±1%**; tasas, puntajes y estados deben ser exactos.
- Scripts obligatorios en `package.json`:
  - `npm start` → `node src/server.js`
  - `npm test` → `node --test test/`

## 8. Restricciones no funcionales

| Restricción | Valor |
|---|---|
| Ejecución | Ambiente web de Claude Code, puerto 3000 (configurable por variable `PORT`) |
| Seguridad | Sin autenticación (fuera de alcance). Solo datos ficticios; prohibido usar datos personales reales |
| Dependencias | Solo `express` |
| Persistencia | Solo archivos JSON; sin bases de datos |
| Idioma | Español en interfaz, mensajes, documentos y comentarios |

**Modo sin dependencias (contingencia):** si el registro npm no está disponible, se reemplaza Express por el módulo nativo `node:http` con un enrutador mínimo propio. El resto de la arquitectura no cambia.

## 9. API mínima sugerida por caso

> Es una guía para mantener el alcance; el Spec Técnico define el contrato final.

| Caso | Endpoints sugeridos |
|---|---|
| A · Originación | `POST /api/solicitudes` (evalúa y registra) |
| B · Pricing | `POST /api/cotizaciones` (calcula y registra) |
| C · Riesgo | `POST /api/evaluaciones` (evalúa y registra) |

## 10. Referencia de evolución a AWS (fuera del alcance del taller)

Cómo evolucionaría esta arquitectura hacia producción en AWS, manteniendo el spec como contrato:

| Capa del taller | Evolución en AWS |
|---|---|
| `public/` estático | Amazon S3 + Amazon CloudFront |
| Express `/api` | Amazon API Gateway + AWS Lambda (o Amazon ECS en AWS Fargate) |
| Registros JSON | Amazon DynamoDB |
| Parámetros JSON | AWS AppConfig o AWS Systems Manager Parameter Store |
| Logs de consola | Amazon CloudWatch |
| Asistente de desarrollo | Claude Code con modelos Claude en Amazon Bedrock |

---

# Anexos — Datos semilla por caso

> Cada participante usa **solo el anexo de su caso**. Los archivos se crean en `data/` con el nombre indicado y el contenido exacto.

## Anexo A — Caso A · Originación de Crédito de Libre Inversión

### `data/parametros-producto.json`
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

### `data/clientes.json`
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

### `data/solicitudes.json`
```json
[]
```

---

## Anexo B — Caso B · Pricing Engine

### `data/parametros-pricing.json`
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
  }
}
```

### `data/cotizaciones.json`
```json
[]
```

---

## Anexo C — Caso C · Motor de Riesgo de Perfil Crediticio

### `data/scorecard.json`
```json
{
  "version": "RSK-2026-10-v1",
  "reglasKO": [
    { "codigo": "KO-01", "descripcion": "Reportado en listas restrictivas" },
    { "codigo": "KO-02", "descripcion": "Mora activa mayor a 90 días", "umbralDias": 90 },
    { "codigo": "KO-03", "descripcion": "Score de buró inferior al mínimo", "scoreMinimo": 500 }
  ],
  "variables": [
    { "codigo": "SCORE_BURO", "nombre": "Score de buró", "maxPuntos": 50,
      "rangos": [ { "desde": 800, "hasta": 999, "puntos": 50 }, { "desde": 700, "hasta": 799, "puntos": 40 }, { "desde": 600, "hasta": 699, "puntos": 25 }, { "desde": 500, "hasta": 599, "puntos": 10 } ] },
    { "codigo": "ENDEUDAMIENTO", "nombre": "Endeudamiento (obligaciones / ingreso)", "maxPuntos": 30,
      "rangos": [ { "hastaIncluido": 0.20, "puntos": 30 }, { "hastaIncluido": 0.35, "puntos": 20 }, { "hastaIncluido": 0.50, "puntos": 10 }, { "hastaIncluido": null, "puntos": 0 } ],
      "nota": "Rangos evaluados en orden; null = sin límite superior. Ingreso 0 se trata como > 50%." },
    { "codigo": "MORA_12M", "nombre": "Máxima mora últimos 12 meses (días)", "maxPuntos": 20,
      "rangos": [ { "desde": 0, "hasta": 0, "puntos": 20 }, { "desde": 1, "hasta": 30, "puntos": 12 }, { "desde": 31, "hasta": 60, "puntos": 5 }, { "desde": 61, "hasta": 99999, "puntos": 0 } ] }
  ],
  "decision": [
    { "puntajeDesde": 70, "decision": "APROBAR" },
    { "puntajeDesde": 50, "decision": "REVISAR" },
    { "puntajeDesde": 0, "decision": "RECHAZAR" }
  ],
  "factoresExplicacion": 2
}
```

### `data/solicitantes.json`
```json
[
  { "cedula": "1001", "nombre": "Ana María Gómez", "scoreBuro": 820, "ingresoMensual": 6000000, "obligacionesMensuales": 500000, "moraMaxDias12m": 0, "moraActivaDias": 0, "listasRestrictivas": false },
  { "cedula": "1002", "nombre": "Carlos Andrés Ruiz", "scoreBuro": 720, "ingresoMensual": 4000000, "obligacionesMensuales": 1200000, "moraMaxDias12m": 15, "moraActivaDias": 0, "listasRestrictivas": false },
  { "cedula": "1003", "nombre": "Laura Pérez", "scoreBuro": 640, "ingresoMensual": 2500000, "obligacionesMensuales": 1000000, "moraMaxDias12m": 0, "moraActivaDias": 0, "listasRestrictivas": false },
  { "cedula": "1004", "nombre": "Jorge Díaz", "scoreBuro": 560, "ingresoMensual": 2000000, "obligacionesMensuales": 1200000, "moraMaxDias12m": 45, "moraActivaDias": 0, "listasRestrictivas": false },
  { "cedula": "1005", "nombre": "Marta León", "scoreBuro": 780, "ingresoMensual": 5000000, "obligacionesMensuales": 800000, "moraMaxDias12m": 0, "moraActivaDias": 0, "listasRestrictivas": true },
  { "cedula": "1006", "nombre": "Pedro Salas", "scoreBuro": 700, "ingresoMensual": 3500000, "obligacionesMensuales": 900000, "moraMaxDias12m": 120, "moraActivaDias": 120, "listasRestrictivas": false },
  { "cedula": "1008", "nombre": "Andrés Mejía", "scoreBuro": 450, "ingresoMensual": 2800000, "obligacionesMensuales": 600000, "moraMaxDias12m": 30, "moraActivaDias": 0, "listasRestrictivas": false }
]
```

### `data/evaluaciones.json`
```json
[]
```

---

*Manifiesto Técnico del taller SDD · GFT Technologies & AWS · Los datos son ficticios y de uso exclusivo para el taller.*
