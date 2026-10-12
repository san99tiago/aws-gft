# Manifiesto Funcional — Caso C
## Motor de Riesgo para Perfil Crediticio

| Metadato | Valor |
|---|---|
| **Tipo de documento** | Brief Funcional (requerimiento de negocio) |
| **Prefijo de Spec ID** | `RSK-PER` |
| **Dominio** | Gestión de riesgo de crédito |
| **Usuario principal** | Analista de riesgo |
| **Entidad** | Banco Demo (ficticio) |
| **Tiempo objetivo de construcción** | 35 minutos con Claude Code, desde cero |

> Todos los datos, puntajes y umbrales de este documento son **ficticios e ilustrativos**. No representan una política de riesgo real.

---

## 1. Contexto de negocio

Los analistas de riesgo de Banco Demo evalúan perfiles con criterios distintos y sus decisiones no se pueden explicar ni auditar. El regulador exige decisiones **consistentes, explicables y trazables** a la versión de la política aplicada. El negocio quiere un **motor de riesgo** que aplique reglas excluyentes (knock-out), calcule un puntaje con una scorecard parametrizada y explique qué factores pesaron.

## 2. Objetivo y resultado esperado

- **Objetivo:** que todo perfil se evalúe con la misma política y que cada decisión pueda explicarse en una frase.
- **Resultado esperado:** una página web donde el analista ingresa una cédula y obtiene decisión, puntaje por variable y los 2 factores que más restaron puntos, quedando la evaluación registrada.

## 3. Actores

| Actor | Rol |
|---|---|
| Analista de riesgo | Ingresa la cédula y ejecuta la evaluación |
| Base de solicitantes (simulada) | Provee la información del perfil en un archivo JSON |

## 4. Flujo (una sola pantalla)

1. El analista ingresa una **cédula** y presiona **Evaluar**.
2. El motor aplica las reglas knock-out y, si ninguna se activa, calcula la scorecard.
3. Se muestran nombre del solicitante, decisión, puntaje total, puntaje por variable y los 2 factores principales (o los KO activados), con el número de evaluación.

## 5. Alcance

**Dentro del alcance**
- Reglas knock-out y scorecard de 3 variables.
- Decisión por puntaje.
- Explicabilidad (top 2 factores).
- Registro de cada evaluación con la versión de la scorecard.

**Fuera del alcance**
- Historial de evaluaciones y ficha detallada del solicitante.
- Bandas de riesgo, modelos estadísticos o de machine learning.
- Edición de la scorecard desde la interfaz.
- Flujo de revisión manual (la decisión REVISAR solo se informa).

## 6. Reglas de negocio

### 6.1 Reglas knock-out (se evalúan primero)

| ID sugerido | Código | Regla |
|---|---|---|
| RN-01 | KO-01 | Solicitante reportado en **listas restrictivas** → RECHAZAR. |
| RN-02 | KO-02 | **Mora activa mayor a 90 días** → RECHAZAR. |
| RN-03 | KO-03 | **Score de buró menor a 500** → RECHAZAR. |
| RN-04 | — | Si se activa cualquier KO, la decisión es **RECHAZAR**, **no se calcula la scorecard** y se listan **todas** las reglas KO activadas. |

### 6.2 Scorecard (máximo 100 puntos)

| ID sugerido | Variable | Rangos y puntos | Máx. |
|---|---|---|---|
| RN-05 | **Score de buró** | 800–999 → 50 · 700–799 → 40 · 600–699 → 25 · 500–599 → 10 | 50 |
| RN-06 | **Endeudamiento** (obligaciones mensuales / ingreso mensual) | ≤ 20% → 30 · > 20% y ≤ 35% → 20 · > 35% y ≤ 50% → 10 · > 50% → 0. Si el ingreso es 0, se asume > 50%. | 30 |
| RN-07 | **Máxima mora últimos 12 meses** (días) | 0 → 20 · 1–30 → 12 · 31–60 → 5 · > 60 → 0 | 20 |

### 6.3 Decisión, explicabilidad y registro

| ID sugerido | Regla |
|---|---|
| RN-08 | **Decisión por puntaje total:** ≥ 70 → **APROBAR** · 50–69 → **REVISAR** · < 50 → **RECHAZAR**. |
| RN-09 | **Explicabilidad:** para cada variable, puntos perdidos = máximo − obtenido. Se muestran los **2 factores con más puntos perdidos** (solo si perdieron > 0). En empate, se respeta el orden de la tabla 6.2. |
| RN-10 | Cada evaluación se registra con ID secuencial `EVA-0001`, `EVA-0002`…, fecha, cédula, KO activados, puntaje por variable, total, decisión, factores y **versión de la scorecard**. |
| RN-11 | Si la cédula no existe, se informa "Solicitante no encontrado" y no se registra evaluación. |

## 7. Información que se muestra

| Elemento | Contenido |
|---|---|
| Resultado | Nombre del solicitante, decisión (con color), puntaje total, tabla de puntaje por variable, KO activados o top 2 factores, ID de evaluación |

## 8. Criterios de aceptación de negocio

1. Las reglas KO se evalúan antes que la scorecard y, si se activan, la scorecard no se calcula.
2. El puntaje total es la suma exacta de los puntajes por variable.
3. La decisión corresponde exactamente a la tabla de RN-08.
4. La explicación muestra los factores correctos en el orden correcto.
5. Toda evaluación queda registrada con la versión de la scorecard.

## 9. Escenarios de prueba de usuario final

> Los datos de cada cédula están en el anexo de datos del Manifiesto Técnico. Orden de puntos: Buró / Endeudamiento / Mora 12m.

| # | Cédula | Puntos por variable | Total | Decisión esperada | Top factores / KO |
|---|---|---|---|---|---|
| E1 | 1001 | 50 / 30 / 20 | 100 | **APROBAR** | Sin factores (no perdió puntos) |
| E2 | 1002 | 40 / 20 / 12 | 72 | **APROBAR** | Buró (−10), Endeudamiento (−10) — empatan con orden de tabla |
| E3 | 1003 | 25 / 10 / 20 | 55 | **REVISAR** | Buró (−25), Endeudamiento (−20) |
| E4 | 1004 | 10 / 0 / 5 | 15 | **RECHAZAR** | Buró (−40), Endeudamiento (−30) |
| E5 | 1005 | — | — | **RECHAZAR** | KO-01 (listas restrictivas) |
| E6 | 1006 | — | — | **RECHAZAR** | KO-02 (mora activa 120 días) |
| E7 | 1008 | — | — | **RECHAZAR** | KO-03 (score de buró 450) |
| E8 | 9999 | — | — | "Solicitante no encontrado" | No se registra evaluación |

## 10. Supuestos y preguntas abiertas (a resolver en el Spec Funcional)

- ¿Se puede reevaluar la misma cédula? (Supuesto: sí, genera una nueva evaluación.)
- ¿La decisión REVISAR debe asignarse a un analista? (Supuesto: no en esta versión, solo se informa.)

## 11. Definición de terminado

- La app corre en el ambiente con la evaluación y su explicación en una sola pantalla.
- Los escenarios E1 a E8 pasan.
- Cada regla RN tiene al menos un criterio de aceptación y una prueba trazable.
