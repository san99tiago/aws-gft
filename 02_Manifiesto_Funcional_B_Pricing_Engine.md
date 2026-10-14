# Manifiesto Funcional — Caso B
## Pricing Engine de Crédito de Consumo

| Metadato | Valor |
|---|---|
| **Tipo de documento** | Brief Funcional (requerimiento de negocio) |
| **Prefijo de Spec ID** | `PRC-ENG` |
| **Dominio** | Producto y pricing |
| **Usuarios** | Asesor comercial · Gerente de producto |
| **Entidad** | Banco Demo (ficticio) |
| **Tiempo objetivo de construcción** | 35 minutos con Claude Code, desde cero |

> Todos los datos, tasas y umbrales de este documento son **ficticios e ilustrativos**. No corresponden a tarifas reales ni a la tasa de usura vigente.

---

## 1. Contexto de negocio

Hoy la tasa de cada crédito se calcula en una hoja de Excel que cada asesor tiene en una versión distinta. Resultado: tasas inconsistentes entre oficinas y ninguna trazabilidad de qué tabla se usó. El negocio quiere un **motor de pricing único** que calcule la tasa de forma transparente, explique cómo llegó a ella, ayude al asesor a comparar plazos con el cliente y registre cada cotización.

## 2. Objetivo y resultado esperado

- **Objetivo:** que cualquier asesor obtenga la misma tasa para las mismas condiciones, con su desglose y dentro de los límites de piso y tope del producto.
- **Resultado esperado:** una aplicación web que cotiza con una cascada visual de la tasa, compara la cuota en todos los plazos, guarda un historial de cotizaciones y muestra la tabla de parámetros vigente.

## 3. Actores

| Actor | Rol |
|---|---|
| Asesor comercial | Cotiza, compara plazos con el cliente y consulta su historial |
| Gerente de producto | Consulta el historial con indicadores y la tabla de parámetros vigente |

## 4. Vistas

La aplicación tiene **tres vistas** accesibles desde una barra de navegación:

1. **Cotizador:** el asesor ingresa las condiciones y obtiene la cotización con su **cascada visual** (barras que suman y restan hasta la tasa final), la tasa mensual, la cuota, el número de cotización y su vigencia. Debajo, un **comparador de plazos** muestra la tasa y la cuota para cada plazo permitido, resaltando el elegido.
2. **Historial:** indicadores (total de cotizaciones y tasa final promedio) y la lista de cotizaciones, con filtro por producto.
3. **Parámetros:** la tabla de parámetros vigente, en solo lectura, con su versión.

## 5. Alcance

**Dentro del alcance**
- Cotizador con validación de entradas y cascada visual de la tasa.
- Cálculo de tasa con descuentos, piso y tope; tasa mensual equivalente y cuota.
- Comparador de plazos.
- Registro e historial de cotizaciones con indicadores y filtro.
- Vista de solo lectura de los parámetros vigentes.

**Fuera del alcance**
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
| RN-10 | **Cascada visual:** cada componente (base, spread, ajuste por plazo, cada descuento y ajuste de piso o tope) se muestra como una barra con su signo y valor; la suma coincide con la tasa final. |
| RN-11 | **Comparador de plazos:** con las mismas entradas, calcula tasa final y cuota para cada plazo permitido y resalta el plazo elegido. El comparador **no registra** cotizaciones. |
| RN-12 | Cada cotización se registra con ID secuencial `COT-0001`, `COT-0002`…, fecha, entradas, desglose, tasa final, cuota, **versión de la tabla de parámetros** y **vigencia = fecha + 15 días**. |
| RN-13 | El **historial** muestra las cotizaciones de la más reciente a la más antigua, con filtro por producto, y los indicadores **total de cotizaciones** y **tasa final promedio** (2 decimales) sobre lo filtrado. |
| RN-14 | La vista de **parámetros** muestra la tabla vigente y su versión, sin permitir edición. |

## 8. Desglose esperado (cascada)

```
Tasa base del producto              20,00 %   ████████████████████
+ Spread de riesgo (banda B)        +2,50     ██▌
+ Ajuste por plazo (60 meses)       +1,00     █
− Descuentos (nómina)               −1,00     █
= TASA FINAL EA                     22,50 %
  Tasa mensual equivalente           1,7056 %
  Cuota mensual                     $1.337.700 aprox.
```

## 9. Criterios de aceptación de negocio

1. Dos cotizaciones con las mismas entradas producen exactamente la misma tasa.
2. La cascada muestra cada componente y su signo; la suma coincide con la tasa final.
3. Ninguna tasa final sale del rango piso–tope del producto.
4. Las bandas D y E y las entradas fuera de política no generan cotización.
5. El comparador muestra todos los plazos permitidos sin registrar cotizaciones.
6. Cada cotización queda en el historial con su ID, versión de tabla y vigencia.

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
| E7 | Comparador con las entradas de E1 | | | | | 12 m: 19,50% · ≈ $1.833.100 · 24 m: 19,50% · ≈ $998.000 · **36 m: 20,00% · ≈ $726.800 (resaltado)** · 48 m: 20,00% · ≈ $591.400 · 60 m: 20,50% · ≈ $516.500 · 72 m: 20,50% · ≈ $465.200 · no se registra |
| E8 | Historial | | | | | Tras E1–E6: 4 cotizaciones (E1–E4) con ID, versión y vigencia de 15 días · tasa promedio **20,38%** · la más reciente primero |
| E9 | Historial filtrado por VEHICULO | | | | | Solo la cotización de E4 · tasa promedio 14,00% |
| E10 | Parámetros | | | | | Vista de solo lectura con la tabla vigente y su versión `PRC-2026-10-v1` |

## 11. Supuestos y preguntas abiertas (a resolver en el Spec Funcional)

- ¿El orden es "aplicar descuentos y luego piso/tope"? (Supuesto: sí, según RN-04 a RN-07.)
- ¿Una cotización vencida se puede reutilizar? (Supuesto: no, solo se consulta.)
- ¿Se debe mostrar la tasa nominal mensual vencida además de la EA? (Supuesto: no en esta versión.)

## 12. Definición de terminado

- La app corre en el ambiente con las tres vistas: cotizador con cascada y comparador, historial y parámetros.
- Los escenarios E1 a E10 pasan.
- Cada regla RN tiene al menos un criterio de aceptación y una prueba trazable.
