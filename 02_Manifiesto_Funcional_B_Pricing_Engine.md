# Manifiesto Funcional — Caso B
## Pricing Engine de Crédito de Consumo

| Metadato | Valor |
|---|---|
| **Tipo de documento** | Brief Funcional (requerimiento de negocio) |
| **Prefijo de Spec ID** | `PRC-ENG` |
| **Dominio** | Producto y pricing |
| **Usuario principal** | Asesor comercial |
| **Entidad** | Banco Demo (ficticio) |
| **Tiempo objetivo de construcción** | 35 minutos con Claude Code, desde cero |

> Todos los datos, tasas y umbrales de este documento son **ficticios e ilustrativos**. No corresponden a tarifas reales ni a la tasa de usura vigente.

---

## 1. Contexto de negocio

Hoy la tasa de cada crédito se calcula en una hoja de Excel que cada asesor tiene en una versión distinta. Resultado: tasas inconsistentes entre oficinas y ninguna trazabilidad de qué tabla se usó. El negocio quiere un **motor de pricing único** que calcule la tasa de forma transparente, explique cómo llegó a ella y registre cada cotización.

## 2. Objetivo y resultado esperado

- **Objetivo:** que cualquier asesor obtenga la misma tasa para las mismas condiciones, con su desglose y dentro de los límites de piso y tope del producto.
- **Resultado esperado:** una página web que cotiza, muestra la cascada de construcción de la tasa, calcula la cuota y registra la cotización con un número y la versión de la tabla de parámetros usada.

## 3. Actores

| Actor | Rol |
|---|---|
| Asesor comercial | Ingresa las condiciones del cliente y obtiene la cotización |

## 4. Flujo (una sola pantalla)

1. El asesor ingresa **producto, banda de riesgo, monto, plazo, canal** y si el cliente tiene **nómina** en el banco.
2. Presiona **Cotizar**.
3. El motor muestra la cascada de la tasa, la tasa final EA, la tasa mensual equivalente, la cuota y el número de cotización.

## 5. Alcance

**Dentro del alcance**
- Cotizador con validación de entradas.
- Cálculo de tasa en cascada con descuentos, piso y tope.
- Cálculo de tasa mensual equivalente y cuota.
- Registro de cada cotización.

**Fuera del alcance**
- Historial de cotizaciones, vigencia y vista de parámetros.
- Edición de parámetros desde la interfaz.
- Segmentos de cliente y aprobación de excepciones de tasa.
- Cálculo de la banda de riesgo (llega como dato de entrada).
- Autenticación y perfiles de usuario.

## 6. Entradas del cotizador

| Campo | Valores permitidos |
|---|---|
| Producto | `LIBRE_INVERSION`, `VEHICULO` |
| Banda de riesgo | `A`, `B`, `C`, `D`, `E` |
| Monto | Según rango del producto, múltiplo de $100.000 |
| Plazo | 12, 24, 36, 48, 60 o 72 meses |
| Canal | `DIGITAL`, `OFICINA` |
| Nómina en el banco | Sí / No |

## 7. Reglas de negocio

| ID sugerido | Regla |
|---|---|
| RN-01 | **Rangos de monto por producto:** Libre Inversión $1.000.000 – $80.000.000; Vehículo $10.000.000 – $200.000.000; múltiplo de $100.000. Fuera de rango → error de validación. |
| RN-02 | **Plazos permitidos:** 12, 24, 36, 48, 60 y 72 meses. |
| RN-03 | Las bandas **D y E no son cotizables**: el motor responde "Banda de riesgo no cotizable" sin calcular tasa ni registrar. |
| RN-04 | **Tasa calculada** = tasa base del producto + spread de riesgo + ajuste por plazo − descuentos. |
| RN-05 | **Tasa base:** Libre Inversión 20,00% EA; Vehículo 15,00% EA. **Spread de riesgo:** A +0,00; B +2,50; C +5,00 puntos. **Ajuste por plazo:** 12–24 meses +0,00; 36–48 meses +0,50; 60–72 meses +1,00 puntos. |
| RN-06 | **Descuentos acumulables:** nómina −1,00; canal Digital −0,50 puntos. |
| RN-07 | **Piso y tope:** Libre Inversión piso 14,00% / tope 25,00% EA; Vehículo piso 14,00% / tope 22,00% EA. Si la tasa calculada queda fuera, se lleva al límite y se marca "ajustada a piso" o "ajustada a tope". |
| RN-08 | **Tasa mensual equivalente** `i = (1 + EA)^(1/12) − 1`. **Cuota** por sistema francés: `cuota = P · i / (1 − (1 + i)^−n)`. |
| RN-09 | **Redondeo:** tasa EA a 2 decimales, tasa mensual a 4 decimales (en %), cuota al peso. |
| RN-10 | Cada cotización se registra con ID secuencial `COT-0001`, `COT-0002`…, fecha, entradas, desglose, tasa final, cuota y **versión de la tabla de parámetros**. |

## 8. Desglose esperado (cascada)

```
Tasa base del producto              20,00 %
+ Spread de riesgo (banda B)        +2,50
+ Ajuste por plazo (60 meses)       +1,00
− Descuentos (nómina)               −1,00
= Tasa calculada                    22,50 %
  Ajuste piso / tope                 —
= TASA FINAL EA                     22,50 %
  Tasa mensual equivalente           1,7056 %
  Cuota mensual                     $1.337.700 aprox.
```

## 9. Criterios de aceptación de negocio

1. Dos cotizaciones con las mismas entradas producen exactamente la misma tasa.
2. El desglose muestra cada componente y su signo; la suma coincide con la tasa calculada.
3. Ninguna tasa final sale del rango piso–tope del producto.
4. Las bandas D y E y las entradas fuera de política no generan cotización.
5. Cada cotización queda registrada con su número y la versión de la tabla.

## 10. Escenarios de prueba de usuario final

> Valores aproximados; **tolerancia ±1% en la cuota**. Tasas exactas a 2 decimales.

| # | Producto | Banda | Monto / Plazo | Canal | Nómina | Resultado esperado |
|---|---|---|---|---|---|---|
| E1 | LIBRE_INVERSION | A | $20.000.000 / 36 | DIGITAL | No | Tasa final **20,00%** · mensual 1,5309% · cuota ≈ $726.800 |
| E2 | LIBRE_INVERSION | B | $50.000.000 / 60 | OFICINA | Sí | Tasa final **22,50%** · mensual 1,7056% · cuota ≈ $1.337.700 |
| E3 | LIBRE_INVERSION | C | $10.000.000 / 72 | OFICINA | No | Calculada 26,00% → **ajustada a tope 25,00%** · mensual 1,8769% · cuota ≈ $254.400 |
| E4 | VEHICULO | A | $60.000.000 / 24 | DIGITAL | Sí | Calculada 13,50% → **ajustada a piso 14,00%** · mensual 1,0979% · cuota ≈ $2.857.400 |
| E5 | LIBRE_INVERSION | D | $10.000.000 / 24 | DIGITAL | No | **No cotizable** (banda D) · no se registra cotización |
| E6 | VEHICULO | A | $5.000.000 / 24 | DIGITAL | No | **Error de validación** (monto fuera de rango) |

## 11. Supuestos y preguntas abiertas (a resolver en el Spec Funcional)

- ¿El orden es "aplicar descuentos y luego piso/tope"? (Supuesto: sí, según RN-04 a RN-07.)
- ¿Se debe mostrar la tasa nominal mensual vencida además de la EA? (Supuesto: no en esta versión.)

## 12. Definición de terminado

- La app corre en el ambiente con el cotizador y su desglose en una sola pantalla.
- Los escenarios E1 a E6 pasan.
- Cada regla RN tiene al menos un criterio de aceptación y una prueba trazable.
