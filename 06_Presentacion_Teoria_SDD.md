# Presentación de Teoría — Taller "Menos Código, Más Negocio: SDD en Acción"
### Contenido slide por slide con notas para los facilitadores · GFT Technologies & AWS

| Ficha | |
|---|---|
| **Duración** | 30 minutos |
| **Presentan** | Facilitador GFT y facilitador AWS |
| **Slides** | 21 |
| **Objetivo** | Que cada participante entienda por qué el spec es la fuente única de verdad y llegue a la práctica sabiendo qué va a construir, con qué reglas y en cuánto tiempo |

> Cada slide trae **En pantalla** (lo que ve la audiencia, máximo 5 ideas) y **Notas** (lo que dice quien presenta). El texto marcado con **[COMPLETAR]** depende de mensajes que debe aportar AWS.

| Bloque | Dur. | Lidera | Slides |
|---|---|---|---|
| 1. Apertura y propósito | 5' | GFT & AWS | 1 – 3 |
| 2. La evolución del desarrollador | 7' | GFT | 4 – 8 |
| 3. Anatomía del SDD — visión general | 5' | GFT | 9 – 11 |
| 4. Anatomía del SDD — detalle y profundidad | 5' | AWS | 12 – 15 |
| 5. SDD con Claude Code sobre AWS | 5' | AWS | 16 – 19 |
| 6. Reglas del juego | 3' | GFT | 20 – 21 |

---

## Bloque 1 · Apertura y propósito (5') — GFT & AWS

### Slide 1 · Portada

**En pantalla**
- **Menos Código, Más Negocio: SDD en Acción**
- De la especificación al producto con IA

**Notas:** Bienvenida breve. Anunciar el formato: 30 minutos de teoría y 75 de práctica, en la que cada persona construye su propia aplicación.

### Slide 2 · GFT & AWS

**En pantalla**
- Logos de GFT Technologies y AWS (sin texto adicional)

**Notas:** Santiago y Daniel presentan de viva voz a cada compañía y por qué hacen este taller juntos.

### Slide 3 · Qué te llevas hoy

**En pantalla**
- Una **aplicación funcionando**, construida **desde cero** en menos de una hora
- Un **Spec Funcional** y un **Spec Técnico** aprobados por ti
- **Trazabilidad** de cada línea de código hasta una regla de negocio

**Notas:** Poner la vara alta desde el inicio: el ambiente arranca vacío y al final habrá una app probada contra escenarios de negocio. Lo importante no es la velocidad por sí sola, sino la velocidad con control.

---

## Bloque 2 · La evolución del desarrollador (7') — GFT

### Slide 4 · Cinco momentos de cómo construimos software

**En pantalla**
1. **Tradicional:** la persona escribe cada línea
2. **Copiloto:** la IA autocompleta mientras la persona escribe
3. **Vibe coding:** la persona conversa y la IA genera
4. **SDD:** la persona especifica y aprueba; la IA construye contra el spec
5. **Smart Factory / FDE:** un requerimiento orquestado de punta a punta por agentes, dentro de un *harness*

**Notas:** Recorrer la línea de tiempo rápido. Preguntar a la sala en qué momento está su equipo hoy (mano alzada). La mayoría estará entre 2 y 3.

### Slide 5 · El vibe coding seduce, pero no escala

**En pantalla**
- Un prompt y aparece una app en minutos: ideal para explorar
- **Sin trazabilidad:** no hay forma de ligar código con regla de negocio
- **Sin gobierno:** decisiones implícitas que nadie aprobó
- **Con retrabajo:** cada cambio de política es regenerar y revisar todo
- En banca: auditoría, regulador y explicabilidad no son opcionales

**Notas:** Reconocer el valor real del vibe coding para prototipos. Luego el ejemplo concreto: el regulador pregunta por qué se rechazó un crédito. Con vibe coding la respuesta es "lo decidió el código que generó la IA". Con SDD la respuesta es "regla RN-06, aprobada en el spec versión 1.0, verificada por la prueba E4".

### Slide 6 · El valor se mueve

**En pantalla**

| Antes | Ahora |
|---|---|
| Escribir sintaxis | Especificar comportamiento |
| Conocer el framework | Conocer el negocio |
| Revisar código línea a línea | Aprobar specs y validar escenarios |
| Productividad = líneas escritas | Productividad = valor entregado y trazable |

**Notas:** El rol técnico no desaparece, se eleva. Quien mejor especifica obtiene mejores resultados de la IA. Por eso en la sala hay analistas, arquitectos y líderes de producto, no solo desarrolladores.

### Slide 7 · Smart Factory: un requerimiento, de punta a punta

**En pantalla**

```
Requerimiento → Brief → Specs → Plan → Código → Pruebas → Despliegue
                  ▲        ▲               ▲          ▲
               agentes especializados ejecutan cada etapa
                  ●        ●                          ●
               personas aprueban en los gates
```

- Un solo hilo de trazabilidad, del requerimiento a producción
- La persona **dirige y aprueba**; los agentes **ejecutan**
- El FDE vive cerca del negocio y orquesta el flujo completo

**Notas:** Esta es la visión de la Smart Factory: no se trata de que un desarrollador use un asistente, sino de que un requerimiento entre por un extremo y salga por el otro como software probado, con agentes en cada etapa y personas en los puntos de decisión. Lo que hace posible que ese flujo sea confiable es el harness.

### Slide 8 · Harness engineering: el arnés del agente

**En pantalla**
- **Agente = modelo + harness**
- El *harness* es todo lo que rodea al modelo para que trabaje bien y de forma predecible:
  - **Contexto:** specs, `CLAUDE.md`, datos de referencia
  - **Guardarraíles:** gates, alcance cerrado, reglas del proyecto
  - **Retroalimentación:** pruebas y verificaciones automáticas
  - **Herramientas:** terminal, archivos, APIs
- Hoy vas a construir un harness mínimo: `CLAUDE.md` + specs + pruebas + gates

**Notas:** El resultado de un agente depende tanto de su harness como del modelo. La ingeniería se desplaza a diseñar ese arnés. Cuando el agente se equivoca, no se corrige el código a mano: se mejora el harness (un spec más claro, una regla nueva, una prueba que faltaba) para que no vuelva a pasar. SDD es la forma de construir ese harness alrededor de un requerimiento.

---

## Bloque 3 · Anatomía del SDD — visión general (5') — GFT

### Slide 9 · Principio rector

**En pantalla**
- **Ninguna línea de código nace sin un spec aprobado**
- El spec es la **fuente única de verdad**
- La IA es el acelerador; la persona es la dueña de cada aprobación

**Notas:** Dejar esta frase en pantalla unos segundos. Es la regla que van a aplicar en toda la práctica y la que está escrita en el `CLAUDE.md` que van a crear.

### Slide 10 · El Marco SDD de GFT

**En pantalla**

```
① Brief          →  ② Spec              →  ③ Ejecución acelerada  →  ④ Validación
Funcional          Funcional               Plan                      Pruebas de
Técnico            Técnico                 Código                    usuario final
───────────────── Gobierno transversal: Spec ID · IDs · gates ─────────────────
```

**Notas:** Recorrer las cuatro etapas de izquierda a derecha. Remarcar que el gobierno no es una etapa al final: atraviesa todo el flujo.

### Slide 11 · Qué responde cada artefacto

**En pantalla**

| Artefacto | Responde | En el taller |
|---|---|---|
| Brief Funcional | ¿Qué necesita el negocio y por qué? | Se entrega (Manifiesto Funcional) |
| Brief Técnico | ¿Con qué restricciones construimos? | Se entrega (Manifiesto Técnico) |
| Spec Funcional | ¿Cómo se comporta? ¿Cómo lo verifico? | Lo creas tú con Claude Code |
| Spec Técnico | ¿Dónde vive cada criterio? | Lo creas tú con Claude Code |
| Plan | ¿En qué orden se construye? | Lo creas tú con Claude Code |
| Validación | ¿Funciona como pidió el negocio? | Lo pruebas tú |

**Notas:** Este es el puente con el facilitador de AWS, que va a mostrar cómo se conectan estos artefactos entre sí.

---

## Bloque 4 · Anatomía del SDD — detalle y profundidad (5') — AWS

### Slide 12 · Todo tiene un ID

**En pantalla**
- **Spec ID:** `ORI-LI-F-001` (funcional) · `ORI-LI-T-001` (técnico)
- **RN-xx:** reglas de negocio
- **CA-xx:** criterios de aceptación
- **T-xx:** tareas del plan
- Los mismos IDs aparecen en specs, código y pruebas

**Notas:** Los IDs son lo que convierte un documento en un contrato verificable. Sin ID no hay trazabilidad posible.

### Slide 13 · Trazabilidad de punta a punta

**En pantalla**

```
RN-06  Score ≥ 600 o se rechaza (R-03)
  └─ CA-08  Dado un cliente con score 550
            Cuando solicita $5.000.000 a 12 meses con autorización
            Entonces la solicitud queda RECHAZADA con motivo interno R-03
            Y el mensaje al cliente no revela el motivo
       └─ T-02  Dominio + pruebas
            └─ src/domain.js        // RN-06
                 └─ test/domain.test.js   // CA-08 · Escenario E4
```

**Notas:** Ejemplo real del Caso A. Contar que en la demo final van a hacer justo esto: tomar un CA y buscarlo en el código con `grep`. Esa es la respuesta al regulador de la slide 5.

### Slide 14 · Gates de aprobación

**En pantalla**
- **Gate 1:** Spec Funcional APROBADO → recién ahí se diseña
- **Gate 2:** Spec Técnico APROBADO → recién ahí se planea y se construye
- Cada gate tiene un checklist corto
- Quien aprueba es la persona, no la IA

**Notas:** Los gates son el punto de control humano. En la práctica, cada participante es el dueño de sus gates: revisa con el checklist de la guía y aprueba con un prompt que cambia el estado y la versión del spec.

### Slide 15 · El spec es un artefacto vivo

**En pantalla**
1. Al construir aparece un vacío o una contradicción
2. **Primero** se actualiza el spec
3. Se registra en **Control de cambios**
4. **Después** se ajusta el código

**Notas:** El error clásico es corregir el código y dejar el spec desactualizado: a partir de ahí el spec deja de ser fuente de verdad. Es el mismo principio del harness: se corrige la fuente, no solo el síntoma.

---

## Bloque 5 · SDD con Claude Code sobre AWS (5') — AWS

### Slide 16 · Cómo Claude Code trabaja con specs

**En pantalla**
- **`CLAUDE.md`:** las reglas permanentes del proyecto
- **Los specs:** el contexto y la memoria del proyecto
- **Agente:** lee archivos, planea, escribe código, corre pruebas y corrige
- Si pierde el hilo: "relee el plan y continúa"

**Notas:** Claude Code no es un autocompletado: es un agente que trabaja sobre los archivos del proyecto. Por eso un buen spec en el repositorio rinde más que un buen prompt suelto.

### Slide 17 · Claude en Amazon Bedrock

**En pantalla**
- Modelos Claude dentro de la cuenta AWS
- Control de acceso con IAM y auditoría con AWS CloudTrail
- Los datos no se usan para entrenar los modelos
- [COMPLETAR: mensajes de seguridad y cumplimiento que AWS quiera destacar]

**Notas:** Este es el argumento para entornos regulados: la misma experiencia de desarrollo, con los controles de seguridad y gobierno de AWS.

### Slide 18 · Del laptop a la nube

**En pantalla**

| Hoy en el taller | En producción sobre AWS |
|---|---|
| `public/` estático | Amazon S3 + Amazon CloudFront |
| Express `/api` | Amazon API Gateway + AWS Lambda |
| Registros JSON | Amazon DynamoDB |
| Parámetros JSON | AWS AppConfig |
| Asistente de desarrollo | Claude Code con Claude en Amazon Bedrock |

**Notas:** La arquitectura del taller es deliberadamente simple, pero el spec no cambia al llevarla a producción: lo que cambia es dónde vive cada pieza.

### Slide 19 · Tu ambiente hoy

**En pantalla**
- **Claude Code en el navegador**, sin instalar nada
- Arranca **vacío**: todo se construye desde cero

**Notas:** Mostrar en vivo, en 30 segundos, cómo se entra al ambiente. Explicar cómo pedir ayuda si falla el acceso.

---

## Bloque 6 · Reglas del juego (3') — GFT

### Slide 20 · Elige tu caso

**En pantalla**

| Caso | Construyes | Para quién |
|---|---|---|
| **A · Originación** | Decisión de crédito en una pantalla | Perfiles de experiencia y front |
| **B · Pricing** | Tasa en cascada con piso y tope | Perfiles de producto y cálculo |
| **C · Riesgo** | Knock-out, scorecard y explicación | Perfiles de reglas y analítica |

Datos ficticios · una pantalla por caso · mismo tiempo de construcción

**Notas:** Asignar los casos por tercios. Recordar que el kit trae el Manifiesto Funcional del caso, el Manifiesto Técnico y la Guía del Participante con los prompts listos.

### Slide 21 · Tu ruta en 75 minutos

**En pantalla**

| Paso | Dur. | Gate |
|---|---|---|
| Arranque desde cero | 5' | Claude Code respondiendo |
| Spec Funcional | 7' | **Gate 1** |
| Spec Técnico | 5' | **Gate 2** |
| Plan | 3' | Plan revisado |
| Implementación | 35' | `npm test` en verde |
| Pruebas de usuario | 10' | ≥ 80% escenarios OK |
| Demo y cierre | 10' | — |

- Si no está en el spec, no se construye
- Si vas atrasado: prompt exprés de la guía
- Ayuda: levanta la mano, un facilitador va a tu puesto

**Notas:** Anunciar que el tiempo se va a marcar en voz alta al cierre de cada etapa. Cerrar con "abran el ambiente: empezamos".

---

*Presentación de teoría del taller SDD · GFT Technologies & AWS · Los ejemplos usan datos ficticios del Caso A.*
