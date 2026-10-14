# Guía del Participante
## Del manifiesto a la app, desde cero: Spec Funcional, Spec Técnico y Plan con Claude Code
### Taller SDD · GFT Technologies & AWS

> **Regla de oro:** ninguna línea de código nace sin un spec aprobado. Tú eres el dueño de cada gate; Claude Code es tu acelerador.

---

## Tu ruta en 75 minutos

| Paso | Duración | Salida | Gate |
|---|---|---|---|
| 0. Arranque desde cero | 5' | Proyecto + `CLAUDE.md` | Claude Code respondiendo |
| 1. Spec Funcional | 7' | `specs/01-spec-funcional.md` | **Gate 1:** APROBADO |
| 2. Spec Técnico | 5' | `specs/02-spec-tecnico.md` | **Gate 2:** APROBADO |
| 3. Plan | 3' | `specs/03-plan.md` | Plan revisado |
| 4. Implementación | 35' | Código + pruebas | `npm test` verde · app arriba |
| 5. Pruebas de usuario | 10' | `specs/04-uat.md` | ≥ 80% escenarios OK |
| 6. Demo y cierre | 10' | — | — |

**Cómo funciona la trazabilidad:** el manifiesto trae reglas → el spec funcional las numera como **RN-xx** y crea criterios **CA-xx** → el spec técnico ubica cada CA en un endpoint, función y prueba → el plan crea tareas **T-xx** que citan RN/CA → el código y las pruebas citan los mismos IDs.

---

## Paso 0 · Arranque desde cero (5 min)

Tienes tu propia **cuenta AWS efímera** con Amazon Bedrock y un ambiente web de Claude Code que arranca **vacío**. Todo lo que vas a ver al final lo construyes hoy.

1. Abre el enlace del ambiente con las credenciales de tu cuenta e inicia sesión.
2. Sube tus dos documentos a la raíz del ambiente:
   - Manifiesto funcional de tu caso
   - Manifiesto técnico
3. Pega este prompt en Claude Code:

```text
Prepara el proyecto desde cero:
1. Crea las carpetas kit/ y specs/.
2. Mueve el manifiesto funcional a kit/manifiesto-funcional.md y el técnico a kit/manifiesto-tecnico.md.
3. Crea CLAUDE.md en la raíz con exactamente este contenido:

# Reglas del proyecto — Taller SDD (GFT & AWS)

## Fuentes de verdad (en este orden)
1. specs/01-spec-funcional.md y specs/02-spec-tecnico.md (cuando estén APROBADOS)
2. kit/manifiesto-funcional.md (Brief Funcional)
3. kit/manifiesto-tecnico.md (Brief Técnico y datos semilla)

## Principios Spec-Driven Development
1. No se escribe código sin specs en estado APROBADO.
2. Trazabilidad: reglas RN-xx, criterios CA-xx, tareas T-xx. Código y pruebas citan los IDs en comentarios.
3. No inventar: lo que no esté en el manifiesto o el spec se marca como [SUPUESTO].
4. Alcance cerrado: no agregar funcionalidades, pantallas ni dependencias fuera del spec.
5. Spec vivo: si la implementación revela un vacío, primero se actualiza el spec y se registra en "Control de cambios".
6. Los datos semilla del manifiesto técnico se usan exactamente como están.
7. KISS: la solución más simple que cumpla el spec.
8. Buenas prácticas obligatorias: las de la sección 8 de kit/manifiesto-tecnico.md (SOLID, microservicio autocontenido, cero código espagueti).

## Stack
Node.js 20+, Express (única dependencia), front en frontend/ (HTML/CSS/JS vanilla), back en backend/ (API, reglas, datos en backend/data/*.json y pruebas con node:test).
Front y back solo se comunican por la API /api.

## Comandos
- npm install
- npm test
- npm start  (puerto 3000)

## Idioma
Español en documentos, interfaz, mensajes y comentarios.

No crees nada más. Confírmame con la lista de archivos.
```

> **Tip de tiempo:** durante la implementación activa el modo de aceptar ediciones automáticamente para no aprobar archivo por archivo. En los pasos de specs no es necesario.

---

## Paso 1 · Spec Funcional (7 min)

**Qué es:** el contrato de negocio. Describe el comportamiento esperado sin hablar de tecnología.

### 1.1 Prompt

```text
Actúa como Analista Funcional aplicando Spec-Driven Development.
Lee CLAUDE.md y kit/manifiesto-funcional.md.
Crea specs/01-spec-funcional.md. NO escribas código ni crees otros archivos.

Estructura (máximo 2 páginas):
0. Encabezado: Spec ID (prefijo del manifiesto + "-F-001"), versión 0.1, Estado: BORRADOR, fecha.
1. Objetivo y alcance (máximo 5 líneas).
2. Reglas de negocio: tabla RN-xx | regla con valores exactos.
3. Criterios de aceptación CA-xx en Gherkin (Dado / Cuando / Entonces), cada uno con sus RN-xx.
4. Escenarios: la tabla de escenarios del manifiesto con su CA-xx.
5. Supuestos [SUPUESTO].
6. Matriz de trazabilidad: RN-xx → CA-xx → Escenario.
7. Control de cambios (tabla vacía: versión | fecha | cambio).

No inventes reglas ni cambies valores. Nada de tecnología.
Al terminar, dame un resumen de 3 líneas.
```

### 1.2 Revisa tú (Gate 1)

- [ ] Todas las reglas del manifiesto aparecen como RN-xx con sus valores exactos.
- [ ] Cada RN tiene al menos un CA-xx y cada escenario tiene su CA.
- [ ] No hay tecnología en el documento.

### 1.3 Aprueba

```text
Revisé el spec funcional. Ajustes: <tus ajustes o "ninguno">. Acepto los supuestos.
Cambia el estado a APROBADO, versión 1.0, y registra la aprobación en Control de cambios.
```

---

## Paso 2 · Spec Técnico (5 min)

**Qué es:** el *cómo* gobernado. Dice dónde vive cada criterio de aceptación: en qué endpoint, función y prueba.

### 2.1 Prompt

```text
Actúa como Arquitecto de Software aplicando Spec-Driven Development.
Lee CLAUDE.md, specs/01-spec-funcional.md (APROBADO) y kit/manifiesto-tecnico.md
(solo el anexo de datos de mi caso).
Crea specs/02-spec-tecnico.md. NO escribas código.

Estructura (máximo 2 páginas):
0. Encabezado: Spec ID (el del funcional cambiando "-F-" por "-T-"), versión 0.1, Estado: BORRADOR.
1. Estructura de archivos (la del manifiesto técnico).
2. Contratos de API: por endpoint, método, ruta, request, response y errores.
3. Funciones de dominio: firma y RN-xx que implementa cada una.
4. Vistas: por cada vista, componentes (tarjetas, KPI, tablas, gráficos), campos, validaciones y estados,
   siguiendo los lineamientos de experiencia de usuario del manifiesto técnico.
5. Pruebas: una por escenario, con entrada y resultado esperado.
6. Matriz de trazabilidad: CA-xx → función → prueba.
7. Control de cambios.

Solo el stack del manifiesto técnico; única dependencia express.
Al terminar, dame un resumen de 3 líneas.
```

### 2.2 Revisa tú (Gate 2)

- [ ] Cada CA aparece en la matriz con función y prueba.
- [ ] Las reglas viven en funciones de dominio puras y los valores de política salen de `backend/data/*.json`.
- [ ] No hay dependencias ni funcionalidades fuera del spec.
- [ ] La estructura respeta las buenas prácticas: las reglas de negocio (`domain.js`) reciben los datos como parámetros, sin depender del servidor web ni de los archivos; `routes.js` sin reglas de negocio, `GET /api/health` definido; front en `frontend/` y back en `backend/`.

### 2.3 Aprueba

```text
Revisé el spec técnico. Ajustes: <tus ajustes o "ninguno">.
Cambia el estado a APROBADO, versión 1.0, y registra en Control de cambios.
```

---

## Paso 3 · Plan (3 min)

**Qué es:** la secuencia de trabajo que Claude Code va a ejecutar, en tareas pequeñas y verificables.

### 3.1 Prompt

```text
Actúa como Tech Lead. Lee CLAUDE.md y los dos specs APROBADOS.
Crea specs/03-plan.md. NO escribas código todavía.

Máximo 8 tareas, en este orden:
T-01 package.json, backend/src/server.js mínimo y datos semilla en backend/data/
T-02 dominio + pruebas unitarias (checkpoint: npm test en verde)
T-03 repositorio, rutas y endpoints de la API (incluye GET /api/health)
T-04 front base en frontend/: index.html, styles.css con los componentes visuales, navegación, api.js y formato.js
T-05 vista principal del caso
T-06 vistas secundarias del caso
T-07 arranque de la app y prueba del flujo principal
T-08 specs/04-uat.md con la tabla de escenarios
     (columnas: Escenario | CA | Resultado esperado | Resultado obtenido | OK/NOK)
Por tarea: casilla [ ], archivos, IDs RN/CA que cubre y criterio de verificación.
Muéstrame el plan en una tabla corta.
```

### 3.2 Revisa tú

- [ ] Toda RN y todo CA aparece en al menos una tarea.
- [ ] El dominio y sus pruebas van antes que la API y las vistas.

---

## Paso 4 · Implementación desde cero (35 min)

### 4.1 Prompt

```text
Implementa specs/03-plan.md tarea por tarea, en orden.
- Antes de cada tarea escribe "T-xx: <nombre>"; al terminarla marca [x] en el plan.
- Respeta los specs aprobados. En código y pruebas cita los IDs (// RN-04, // CA-07).
- Corre npm install cuando corresponda y npm test al cerrar T-02; corrige hasta que esté en verde.
- Si encuentras un vacío o contradicción en un spec: actualiza primero el spec
  (Control de cambios) y luego continúa.
- No agregues funcionalidades ni dependencias fuera del spec.
- Aplica las buenas prácticas de la sección 8 del manifiesto técnico y, al final, verifica su checklist.
- No me pidas confirmación entre tareas salvo que estés bloqueado.
- Al final ejecuta npm start y dime cómo abrir la app y probar el flujo principal.
```

### 4.2 Checkpoint a mitad de la implementación

Si a mitad del bloque el dominio no tiene `npm test` en verde, pide:

```text
Prioriza: deja el dominio con pruebas en verde, luego la API y la vista principal.
Mueve las vistas secundarias al final del plan como pendiente.
```

---

## Paso 5 · Pruebas de usuario final (10 min)

1. Con la app corriendo, ábrela en el navegador del ambiente (puerto 3000).
2. Abre `specs/04-uat.md` y ejecuta cada escenario **como lo haría el usuario de negocio**, desde las vistas de la app.
3. Registra el resultado obtenido y marca OK/NOK.

Si un escenario falla:

```text
Falló el escenario <Ex> (CA-xx). Esperado: <resultado esperado>. Obtenido: <resultado obtenido>.
Identifica la causa raíz contra el spec, corrige el código y ajusta la prueba unitaria.
Si el defecto está en el spec, actualízalo primero y regístralo en Control de cambios.
```

---

## Paso 6 · Demo (3 min por voluntario, uno por caso)

1. Muestra tu `specs/01-spec-funcional.md` y un CA.
2. Ejecuta ese escenario en la app.
3. Demuestra la trazabilidad: busca el ID en el código, por ejemplo `grep -rn "CA-03" src test`.
4. Cuenta un aprendizaje: ¿qué tuviste que corregir en el spec durante la construcción?

---

## Plan B · Prompt exprés (si al cerrar el Spec Funcional vas atrasado)

Genera Spec Técnico y Plan en un solo paso:

```text
Con el spec funcional APROBADO y kit/manifiesto-tecnico.md (solo el anexo de mi caso),
crea specs/02-spec-tecnico.md compacto (funciones de dominio, endpoints, vistas y matriz CA → función → prueba)
y specs/03-plan.md con las 8 tareas estándar: package.json+data, dominio+pruebas, API, front base,
vista principal, vistas secundarias, arranque, 04-uat.md. Márcalos ambos como APROBADO versión 1.0. No escribas código todavía.
```

---

## Solución de problemas rápida

| Síntoma | Qué hacer |
|---|---|
| `npm install` falla por red | Pide a Claude: "Aplica el modo sin dependencias del manifiesto técnico: reemplaza Express por node:http y actualiza el spec técnico". |
| Puerto 3000 ocupado | Ejecuta con otro puerto: `PORT=3001 npm start`. |
| No puedes abrir la app en el navegador | Avisa a un facilitador. Mientras tanto, prueba los escenarios con `curl` contra la API y con `npm test`. |
| La cuota no coincide | Verifica la conversión de tasa EA a mensual: `i = (1 + EA/100)^(1/12) − 1`. |
| Claude agrega cosas que no pediste | Recuérdale: "Aplica la regla 4 de CLAUDE.md: alcance cerrado. Revierte lo que no esté en el spec". |
| Claude pierde el hilo | Pídele: "Relee specs/03-plan.md y continúa desde la primera tarea sin marcar". Los specs son la memoria del proyecto. |

---

## Lo que te llevas

- Un **Spec Funcional** y un **Spec Técnico** versionados y aprobados.
- Un **Plan** trazable y una **evidencia de pruebas de usuario**.
- Una **aplicación construida desde cero y funcionando**, en la que cada línea de código se puede rastrear hasta una regla de negocio.

> **Importante:** tu cuenta AWS es temporal y se elimina al terminar el taller. Si quieres conservar tu proyecto (specs y código), descárgalo antes de salir.

*Taller SDD · GFT Technologies & AWS*
