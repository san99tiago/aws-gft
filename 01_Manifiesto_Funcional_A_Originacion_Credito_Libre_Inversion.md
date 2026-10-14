# Manifiesto Funcional — Caso A
## Originación digital de Crédito de Libre Inversión (de cara al cliente)

| Metadato | Valor |
|---|---|
| **Tipo de documento** | Brief Funcional (requerimiento de negocio) |
| **Prefijo de Spec ID** | `ORI-LI` |
| **Dominio** | Originación de crédito de consumo |
| **Usuarios** | Cliente persona natural (canal web) · Gerente comercial (panel) |
| **Entidad** | Banco Demo (ficticio) |
| **Tiempo objetivo de construcción** | 35 minutos con Claude Code, desde cero |

> Todos los datos, tasas y umbrales de este documento son **ficticios e ilustrativos**. No corresponden a tarifas reales ni a la tasa de usura vigente.

---

## 1. Contexto de negocio

Banco Demo pierde clientes de crédito de consumo porque su solicitud de libre inversión exige ir a oficina y la respuesta tarda días. La competencia digital responde en minutos. El negocio quiere un **journey 100% digital** donde el cliente simule, se identifique, reciba una decisión inmediata con su plan de pagos y acepte la oferta en una sola sesión. La gerencia comercial quiere ver en un panel cómo se comportan las solicitudes.

## 2. Objetivo y resultado esperado

- **Objetivo:** que un cliente obtenga una decisión de crédito de libre inversión en menos de 3 minutos, sin intervención humana.
- **Resultado esperado:** una aplicación web con un journey de 4 pasos para el cliente (simular, identificarse, recibir la oferta con plan de pagos, aceptar) y un panel de solicitudes con indicadores para la gerencia comercial.

## 3. Actores

| Actor | Rol |
|---|---|
| Cliente | Simula, entrega datos, autoriza la consulta y acepta o desiste de la oferta |
| Gerente comercial | Consulta el panel de solicitudes y sus indicadores |
| Base de información crediticia (simulada) | Provee fecha de nacimiento, score, obligaciones vigentes y marca de listas restrictivas |

## 4. Journey del cliente y vistas

La aplicación tiene **dos vistas** accesibles desde una barra de navegación: **Solicitar crédito** (journey del cliente) y **Panel de solicitudes**.

**Journey del cliente**, con indicador de progreso de 4 pasos:

1. **Simular:** el cliente elige monto y plazo y ve una cuota referencial que se actualiza al cambiar los valores.
2. **Identificarse y autorizar:** ingresa cédula e ingreso mensual, y autoriza la consulta de su información crediticia.
3. **Recibir la oferta:** el sistema evalúa y muestra la oferta (monto, plazo, tasa, cuota, total a pagar e intereses totales) con su **plan de pagos**, o el mensaje de no aprobación.
4. **Aceptar:** el cliente acepta o desiste; recibe la confirmación con su número de solicitud y el estado final.

## 5. Alcance

**Dentro del alcance**
- Simulador de cuota referencial.
- Formulario de identificación con autorización de consulta.
- Motor de decisión con las reglas de la sección 6.
- Oferta con plan de pagos (tabla de amortización).
- Aceptación o desistimiento de la oferta.
- Panel de solicitudes con indicadores.

**Fuera del alcance**
- Ajuste automático del monto cuando no alcanza la capacidad de pago.
- Autenticación, OTP, biometría y firma electrónica.
- Desembolso y generación de pagaré.
- Integración con centrales de riesgo reales (se simulan con un archivo JSON).
- Edición de parámetros desde la interfaz.

## 6. Reglas de negocio

| ID sugerido | Regla |
|---|---|
| RN-01 | **Monto** entre **$1.000.000 y $50.000.000**, múltiplo de **$100.000**. **Plazo** de **12, 24, 36, 48 o 60 meses**. Fuera de política el cliente no puede avanzar. |
| RN-02 | El **simulador** usa la **tasa referencial de 26% EA** y muestra el aviso "Valor estimado, sujeto a tu perfil crediticio". |
| RN-03 | La **autorización de consulta** es obligatoria para continuar (referencia ilustrativa: Ley 1266 de 2008 y Ley 1581 de 2012). |
| RN-04 | La cédula debe existir en la base de información crediticia. Si no existe → **NO_EVALUABLE** (motivo `NE-01`). |
| RN-05 | Si el cliente está en **listas restrictivas** → **RECHAZADA** (motivo `R-01`). **El mensaje al cliente nunca revela este motivo.** |
| RN-06 | La **edad** a la fecha de la solicitud debe estar entre **18 y 75 años** cumplidos (motivo de rechazo `R-02`). |
| RN-07 | El **score** debe ser **≥ 600** (motivo de rechazo `R-03`). La tasa se asigna por banda: **800–999 → 18% EA**, **700–799 → 22% EA**, **600–699 → 26% EA**. |
| RN-08 | **Cuota** por sistema francés: `cuota = P · i / (1 − (1 + i)^−n)`, con `i = (1 + EA)^(1/12) − 1`. Se redondea al peso. |
| RN-09 | **Capacidad de pago** = `40% × ingreso mensual − obligaciones mensuales vigentes`. Si la cuota ≤ capacidad → **APROBADA**; si no → **RECHAZADA** (motivo `R-04`). |
| RN-10 | **Orden de evaluación:** RN-04 → RN-05 → RN-06 → RN-07 → RN-09. La primera regla que no se cumple detiene la evaluación. |
| RN-11 | **Plan de pagos:** por cada mes, `interés = saldo × i`, `abono a capital = cuota − interés`, `saldo nuevo = saldo − abono`. La última cuota se ajusta para que el saldo final sea exactamente 0. **Total a pagar** = suma de cuotas; **intereses totales** = total a pagar − monto. |
| RN-12 | Toda evaluación genera una **solicitud** con ID secuencial `SOL-0001`, `SOL-0002`…, fecha, datos de entrada, estado, tasa, cuota y motivo interno. |
| RN-13 | Solo una solicitud **APROBADA** puede pasar a **ACEPTADA** (si el cliente acepta) o **DESISTIDA** (si desiste). |
| RN-14 | El **panel** muestra indicadores (total de solicitudes y conteo por estado) y la lista de solicitudes de la más reciente a la más antigua, con el estado destacado por color. El motivo interno solo se ve en el panel. |

## 7. Mensajes al cliente

| Situación | Mensaje |
|---|---|
| Aprobada | "¡Tu crédito está aprobado! Revisa las condiciones de tu oferta." |
| Rechazada (cualquier motivo R-xx) | "En este momento no podemos aprobar tu solicitud. Código de referencia: SOL-xxxx." |
| No evaluable | "No pudimos validar tu información en línea. Acércate a una oficina con el código SOL-xxxx." |
| Sin autorización | "Para continuar necesitamos tu autorización de consulta." |
| Aceptada | "¡Listo! Tu solicitud SOL-xxxx fue aceptada. Pronto te contactaremos para el desembolso." |

## 8. Criterios de aceptación de negocio

1. El cliente no puede avanzar con monto o plazo fuera de política, ni sin autorización.
2. Las decisiones respetan el orden de evaluación y producen el estado correcto para cada escenario de la sección 9.
3. La cuota coincide con la fórmula de RN-08 (tolerancia ±1%) y el plan de pagos termina con saldo 0.
4. Un rechazo nunca revela al cliente el motivo interno.
5. Cada evaluación queda registrada; la aceptación o el desistimiento cambian su estado.
6. El panel refleja todas las solicitudes con indicadores consistentes.

## 9. Escenarios de prueba de usuario final

> Valores aproximados; **tolerancia ±1%**. Los datos de cada cédula están en el anexo de datos del Manifiesto Técnico.

| # | Cédula | Ingreso | Monto / Plazo | Resultado esperado |
|---|---|---|---|---|
| E1 | 1001 | $6.000.000 | $20.000.000 / 36 | **APROBADA** · 18% EA · cuota ≈ $709.700 |
| E2 | 1003 | $2.500.000 | $10.000.000 / 24 | **APROBADA** · 26% EA · cuota ≈ $525.400 |
| E3 | 1002 | $4.000.000 | $30.000.000 / 48 | **RECHAZADA** · motivo interno R-04 (capacidad de pago) |
| E4 | 1004 | $3.000.000 | $5.000.000 / 12 | **RECHAZADA** · motivo interno R-03 (score) |
| E5 | 1005 | $5.000.000 | $10.000.000 / 24 | **RECHAZADA** · motivo interno R-01 · mensaje genérico |
| E6 | 1006 | $4.000.000 | $8.000.000 / 24 | **RECHAZADA** · motivo interno R-02 (edad) |
| E7 | 9999 | $3.000.000 | $5.000.000 / 12 | **NO_EVALUABLE** · NE-01 |
| E8 | — | — | $60.000.000 / 36 | No permite avanzar (monto fuera de rango) |
| E9 | 1001 | $6.000.000 | $20.000.000 / 36 | Sin autorización: no permite avanzar |
| E10 | Simulador | — | $10.000.000 / 24 | Cuota referencial ≈ $525.400 con aviso de estimación |
| E11 | Plan de E1 | — | — | 36 filas · primera fila: interés ≈ $277.800, abono ≈ $432.000, saldo ≈ $19.568.000 · saldo final $0 · total a pagar ≈ $25.550.500 · intereses ≈ $5.550.500 |
| E12 | E1 aceptada | — | — | Solicitud pasa a **ACEPTADA** y se muestra su número |
| E13 | Panel | — | — | Tras E1–E7 y E12: 7 solicitudes · 1 aceptada, 1 aprobada, 4 rechazadas, 1 no evaluable · la más reciente primero |

## 10. Supuestos y preguntas abiertas (a resolver en el Spec Funcional)

- ¿La edad se calcula con la fecha del sistema? (Supuesto: sí.)
- ¿La oferta tiene vigencia? (Supuesto: solo durante la sesión.)
- ¿Puede el cliente cambiar monto o plazo después de la decisión? (Supuesto: no; inicia una nueva solicitud.)

## 11. Definición de terminado

- La app corre en el ambiente con el journey de 4 pasos y el panel de solicitudes.
- Los escenarios E1 a E13 pasan.
- Cada regla RN tiene al menos un criterio de aceptación y una prueba trazable.
