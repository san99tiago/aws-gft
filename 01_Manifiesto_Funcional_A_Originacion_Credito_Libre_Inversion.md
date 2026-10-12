# Manifiesto Funcional — Caso A
## Originación digital de Crédito de Libre Inversión (de cara al cliente)

| Metadato | Valor |
|---|---|
| **Tipo de documento** | Brief Funcional (requerimiento de negocio) |
| **Prefijo de Spec ID** | `ORI-LI` |
| **Dominio** | Originación de crédito de consumo |
| **Usuario principal** | Cliente persona natural (canal web) |
| **Entidad** | Banco Demo (ficticio) |
| **Tiempo objetivo de construcción** | 35 minutos con Claude Code, desde cero |

> Todos los datos, tasas y umbrales de este documento son **ficticios e ilustrativos**. No corresponden a tarifas reales ni a la tasa de usura vigente.

---

## 1. Contexto de negocio

Banco Demo pierde clientes de crédito de consumo porque su solicitud de libre inversión exige ir a oficina y la respuesta tarda días. La competencia digital responde en minutos. El negocio quiere que el cliente pida su crédito en línea y reciba una **decisión inmediata**.

## 2. Objetivo y resultado esperado

- **Objetivo:** que un cliente obtenga una decisión de crédito de libre inversión en una sola pantalla, sin intervención humana.
- **Resultado esperado:** una página web donde el cliente ingresa sus datos y recibe una de tres respuestas: **aprobada** (con su oferta), **rechazada** o **no evaluable en línea**. Cada solicitud queda registrada con un número.

## 3. Actores

| Actor | Rol |
|---|---|
| Cliente | Ingresa sus datos, autoriza la consulta y recibe la decisión |
| Base de información crediticia (simulada) | Provee fecha de nacimiento, score, obligaciones vigentes y marca de listas restrictivas |

## 4. Flujo (una sola pantalla)

1. El cliente ingresa **cédula, ingreso mensual, monto y plazo**, y marca la **autorización de consulta**.
2. Presiona **Solicitar**.
3. El sistema evalúa y muestra la **oferta** (monto, plazo, tasa y cuota) o el mensaje de no aprobación, siempre con el número de solicitud.

## 5. Alcance

**Dentro del alcance**
- Formulario único con validación de entradas.
- Motor de decisión con las reglas de la sección 6.
- Registro de cada solicitud con su estado y motivo interno.

**Fuera del alcance**
- Simulador previo, aceptación o desistimiento de la oferta y ajuste automático del monto.
- Autenticación, OTP, biometría y firma electrónica.
- Desembolso, pagaré y plan de pagos.
- Integración con centrales de riesgo reales (se simulan con un archivo JSON).
- Historial de solicitudes y panel administrativo.

## 6. Reglas de negocio

| ID sugerido | Regla |
|---|---|
| RN-01 | **Monto** entre **$1.000.000 y $50.000.000**, múltiplo de **$100.000**. **Plazo** de **12, 24, 36, 48 o 60 meses**. Fuera de política no se puede enviar la solicitud. |
| RN-02 | La **autorización de consulta** es obligatoria para enviar la solicitud. |
| RN-03 | La cédula debe existir en la base de información crediticia. Si no existe → **NO_EVALUABLE** (motivo `NE-01`). |
| RN-04 | Si el cliente está en **listas restrictivas** → **RECHAZADA** (motivo `R-01`). **El mensaje al cliente nunca revela este motivo.** |
| RN-05 | La **edad** a la fecha de la solicitud debe estar entre **18 y 75 años** cumplidos (motivo de rechazo `R-02`). |
| RN-06 | El **score** debe ser **≥ 600** (motivo de rechazo `R-03`). La tasa se asigna por banda: **800–999 → 18% EA**, **700–799 → 22% EA**, **600–699 → 26% EA**. |
| RN-07 | **Cuota** por sistema francés: `cuota = P · i / (1 − (1 + i)^−n)`, con `i = (1 + EA)^(1/12) − 1`. Se redondea al peso. |
| RN-08 | **Capacidad de pago** = `40% × ingreso mensual − obligaciones mensuales vigentes`. Si la cuota ≤ capacidad → **APROBADA**; si no → **RECHAZADA** (motivo `R-04`). |
| RN-09 | **Orden de evaluación:** RN-03 → RN-04 → RN-05 → RN-06 → RN-08. La primera regla que no se cumple detiene la evaluación. |
| RN-10 | Toda evaluación genera una **solicitud** con ID secuencial `SOL-0001`, `SOL-0002`…, fecha, datos de entrada, estado, tasa, cuota y motivo interno. |

## 7. Mensajes al cliente

| Situación | Mensaje |
|---|---|
| Aprobada | "¡Tu crédito está aprobado!" + monto, plazo, tasa EA, cuota y número de solicitud |
| Rechazada (cualquier motivo R-xx) | "En este momento no podemos aprobar tu solicitud. Código de referencia: SOL-xxxx." |
| No evaluable | "No pudimos validar tu información en línea. Acércate a una oficina con el código SOL-xxxx." |
| Sin autorización | "Para continuar necesitamos tu autorización de consulta." |

## 8. Criterios de aceptación de negocio

1. No se puede enviar la solicitud con monto o plazo fuera de política, ni sin autorización.
2. Las decisiones respetan el orden de evaluación y producen el estado correcto para cada escenario de la sección 9.
3. La cuota mostrada coincide con la fórmula de RN-07 (tolerancia ±1%).
4. Un rechazo nunca revela al cliente el motivo interno.
5. Cada evaluación queda registrada con su número de solicitud.

## 9. Escenarios de prueba de usuario final

> Valores de cuota aproximados; **tolerancia ±1%**. Los datos de cada cédula están en el anexo de datos del Manifiesto Técnico.

| # | Cédula | Ingreso | Monto / Plazo | Resultado esperado |
|---|---|---|---|---|
| E1 | 1001 | $6.000.000 | $20.000.000 / 36 | **APROBADA** · 18% EA · cuota ≈ $709.700 |
| E2 | 1003 | $2.500.000 | $10.000.000 / 24 | **APROBADA** · 26% EA · cuota ≈ $525.400 |
| E3 | 1002 | $4.000.000 | $30.000.000 / 48 | **RECHAZADA** · motivo interno R-04 (capacidad de pago) |
| E4 | 1004 | $3.000.000 | $5.000.000 / 12 | **RECHAZADA** · motivo interno R-03 (score) |
| E5 | 1005 | $5.000.000 | $10.000.000 / 24 | **RECHAZADA** · motivo interno R-01 · mensaje genérico |
| E6 | 1006 | $4.000.000 | $8.000.000 / 24 | **RECHAZADA** · motivo interno R-02 (edad) |
| E7 | 9999 | $3.000.000 | $5.000.000 / 12 | **NO_EVALUABLE** · NE-01 |
| E8 | 1001 | $6.000.000 | $60.000.000 / 36 | No permite enviar (monto fuera de rango) |
| E9 | 1001 | $6.000.000 | $20.000.000 / 36 | Sin autorización: no permite enviar |

## 10. Supuestos y preguntas abiertas (a resolver en el Spec Funcional)

- ¿La edad se calcula con la fecha del sistema? (Supuesto: sí.)
- ¿Una solicitud no evaluable también recibe número? (Supuesto: sí, para que el cliente lo presente en oficina.)

## 11. Definición de terminado

- La app corre en el ambiente y el cliente obtiene su decisión en una sola pantalla.
- Los escenarios E1 a E9 pasan.
- Cada regla RN tiene al menos un criterio de aceptación y una prueba trazable.
