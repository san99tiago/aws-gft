# Taller "Menos Código, Más Negocio: SDD en Acción"
### De la especificación al producto con IA · GFT Technologies & AWS

| Ficha | |
|---|---|
| **Duración** | 1 hora 45 minutos (105 min) |
| **Formato** | 30 min de teoría + 75 min de práctica individual guiada |
| **Organizan** | GFT Technologies & AWS |
| **Audiencia** | Desarrolladores, arquitectos, analistas funcionales, líderes técnicos y de producto |
| **Herramienta** | Claude Code en un ambiente web en el navegador, sobre una **cuenta AWS efímera por asistente** con su propio Amazon Bedrock |
| **Punto de partida** | **Desde cero (from scratch):** el ambiente arranca vacío; no hay nada técnico creado antes del taller |
| **Resultado** | Cada participante sale con una aplicación funcionando, construida desde cero a partir de un Spec Funcional, un Spec Técnico y un Plan trazables |

---

## 1. Objetivo del taller

Dar una **base conceptual y práctica de Spec-Driven Development (SDD)**: entender por qué el spec es la fuente única de verdad y vivirlo construyendo **desde cero**, en menos de una hora, una aplicación probada contra criterios de aceptación de negocio.

El taller también busca **mostrar la velocidad**: de un ambiente vacío a una app funcionando y trazable en una sola sesión.

**Principio rector (Marco SDD GFT):** ninguna línea de código nace sin un spec aprobado.

**Principio de diseño del taller:** KISS (*Keep It Simple, Stupid*). Casos pequeños, una pantalla por caso, pocas reglas y pocos escenarios.

---

## 2. Cómo se mapea el taller al Marco SDD de GFT

| Marco SDD GFT | En el taller | Artefacto |
|---|---|---|
| ① Brief Funcional (qué y por qué) | Se entrega al participante | `Manifiesto Funcional` del caso (A, B o C) |
| ① Brief Técnico (contexto, restricciones) | Se entrega al participante | `Manifiesto Técnico` común + datos semilla JSON |
| ② Spec Funcional | Lo construye el participante con Claude Code | `specs/01-spec-funcional.md` |
| ② Spec Técnico | Lo construye el participante con Claude Code | `specs/02-spec-tecnico.md` |
| ③ Ejecución acelerada | Plan + implementación desde cero con Claude Code | `specs/03-plan.md` + código |
| ④ Validación | Pruebas de usuario final contra escenarios | `specs/04-uat.md` |
| Gobierno transversal | Spec ID, IDs RN/CA/T, gates de aprobación | Trazabilidad en specs, código y pruebas |

---

## 3. Agenda

> Cada bloque incluye qué se explica o se hace, en máximo 3 líneas.

### Parte 1 — Teoría (30 min)

> El contenido slide por slide, con notas para los facilitadores, está en `06_Presentacion_Teoria_SDD.md`.

| Bloque | Dur. | Lidera | Qué se explica |
|---|---|---|---|
| **Apertura y propósito** | 5' | GFT & AWS | Presentación de GFT y AWS y por qué hacemos este taller juntos. Objetivo del día y lo que cada persona se lleva: una app construida desde cero a partir de un spec. |
| **La evolución del desarrollador** | 7' | GFT | Del desarrollo tradicional al copiloto, al vibe coding, al SDD y a la Smart Factory: un requerimiento orquestado de punta a punta por agentes, con *harness engineering*. Por qué el vibe coding no escala en entornos regulados. El valor se mueve de escribir sintaxis a especificar y orquestar. |
| **Anatomía del SDD — visión general** | 5' | GFT | Marco SDD de GFT: Brief → Spec Funcional → Spec Técnico → Plan → Código → Validación. Qué rol cumple cada etapa y por qué el spec es la fuente única de verdad. |
| **Anatomía del SDD — detalle y profundidad** | 5' | AWS | Spec ID y trazabilidad de punta a punta (RN → CA → tarea → código → prueba); gates de aprobación entre etapas. El spec como artefacto vivo: lo aprendido al construir se devuelve al spec. |
| **SDD con Claude Code sobre AWS** | 5' | AWS | Cómo Claude Code consume specs como contexto (CLAUDE.md, archivos del repo) y ejecuta planes. Claude en Amazon Bedrock: seguridad, gobierno y camino del laptop a la nube. |
| **Reglas del juego** | 3' | GFT | Los 3 casos de uso, el kit que recibe cada participante y los gates por etapa. Duración de cada etapa, checkpoints y cómo pedir ayuda a los facilitadores. |

### Parte 2 — Práctica guiada (75 min)

| Bloque | Dur. | Lidera | Qué se hace |
|---|---|---|---|
| **Arranque desde cero** | 5' | Facilitadores | Cada participante entra a su ambiente web vacío y confirma su caso. Sube los manifiestos a `kit/` y, con un prompt, Claude Code crea la estructura del proyecto y el `CLAUDE.md` con las reglas SDD. |
| **Spec Funcional** | 7' | Facilitadores | Claude Code transforma el Manifiesto Funcional en reglas RN-xx y criterios CA-xx en Gherkin, con matriz de trazabilidad. El participante revisa con el checklist y aprueba (**Gate 1**). |
| **Spec Técnico** | 5' | Facilitadores | Claude Code deriva del spec funcional y del Manifiesto Técnico: contratos API, funciones de dominio, pantalla y pruebas. El participante valida que cada CA tenga dónde vivir y aprueba (**Gate 2**). |
| **Plan** | 3' | Facilitadores | Claude Code genera el plan de tareas T-xx (máx. 6) con IDs trazados y criterio de verificación. Incluye la plantilla de pruebas de usuario `04-uat.md`. |
| **Implementación desde cero** | 35' | Facilitadores | Claude Code ejecuta el plan sin pausas: proyecto, datos, dominio con pruebas, API y pantalla. Cita IDs en código y pruebas. **Checkpoint a mitad del bloque:** dominio con `npm test` en verde; si aparece un vacío, se actualiza primero el spec. |
| **Pruebas de usuario final** | 10' | Facilitadores | Con la app corriendo en el ambiente, el participante ejecuta los escenarios del manifiesto y registra OK/NOK en `04-uat.md`. Los defectos se corrigen citando el CA afectado. |
| **Demo y cierre** | 10' | GFT & AWS | Tres demos de 2 minutos (una por caso): spec → app → un escenario → rastreo de un CA hasta el código. Cierre con aprendizajes, próximos pasos y encuesta; recordar descargar el proyecto antes de que se elimine la cuenta. |

---

## 4. Checkpoints de control (semáforo del facilitador)

| Al cierre de | Hito esperado | Si un participante va atrasado |
|---|---|---|
| Arranque | Proyecto creado desde cero, manifiestos en `kit/`, `CLAUDE.md` creado y Claude Code respondiendo | Facilitador AWS resuelve el acceso al ambiente o se forma pareja |
| Spec Funcional | Spec Funcional en estado APROBADO | Aprobar con preguntas abiertas como [SUPUESTO] |
| Spec Técnico | Spec Técnico en estado APROBADO | Usar el **prompt exprés** (Spec Técnico + Plan en uno) de la guía |
| Plan | Plan generado (máx. 6 tareas) | Aprobar el plan tal cual; los estilos visuales van al final |
| Mitad de la implementación | Dominio implementado, `npm test` en verde | Priorizar API + pantalla del flujo principal |
| Implementación | App corriendo en el ambiente | Probar escenarios vía `curl` contra la API |
| Pruebas de usuario final | ≥ 80% de escenarios OK | Registrar NOK y defecto como aprendizaje del spec |

---

## 5. Los tres casos de uso

Los casos están acotados con criterio **KISS** para que quepan, desde cero, en los 35 minutos de implementación.

| Caso | Nombre | Usuario | Foco de aprendizaje | Pantallas | Reglas | Escenarios |
|---|---|---|---|---|---|---|
| **A** | Originación digital de crédito libre inversión | Cliente final | Reglas en cadena con orden de evaluación y capacidad de pago | 1 | 10 | 9 |
| **B** | Pricing Engine | Asesor comercial | Tasa en cascada, descuentos, piso y tope | 1 | 10 | 6 |
| **C** | Motor de riesgo de perfil crediticio | Analista de riesgo | Reglas knock-out, scorecard de 3 variables y explicación de la decisión | 1 | 11 | 8 |

Los tres usan la misma arquitectura de referencia: front estático de una sola página, back Node.js/Express y datos en archivos JSON. Cada evaluación se registra con un ID; no hay vistas de historial ni de administración. Los datos, tasas y umbrales son **ficticios e ilustrativos**.

**Asignación recomendada:** repartir en tercios. Si se conoce el perfil, orientar A a perfiles de experiencia/front, B a perfiles de producto/cálculo y C a perfiles de reglas/analítica.

---

## 6. Kit que recibe cada participante

1. **Cuenta AWS efímera con ambiente de Claude Code** — cada asistente recibe su propia cuenta AWS temporal, con Amazon Bedrock y un ambiente web de Claude Code listo para usar en el navegador. Arranca **vacía**: todo el proyecto se crea desde cero en la sesión. La cuenta se elimina al terminar el taller.
2. `Manifiesto Funcional` de su caso (A, B o C) — el requerimiento de negocio.
3. `Manifiesto Técnico` — arquitectura de referencia común + anexo de datos JSON de su caso.
4. `Guía del Participante` — paso a paso, prompts listos para pegar y checklists de cada gate.

Distribución sugerida: los tres documentos se comparten por enlace o ZIP al inicio de la práctica, y el participante los sube a su ambiente en el Arranque.

---

## 7. Roles del equipo facilitador

El taller lo conducen **dos facilitadores, uno de GFT y uno de AWS**, en igualdad de rol. Entre los dos cubren la teoría, la conducción de la práctica y el acompañamiento en sala.

| Facilitador | Énfasis | Responsabilidad compartida |
|---|---|---|
| GFT | Marco SDD, specs, gates y trazabilidad | Conducir las etapas de la práctica, recorrer la sala, desbloquear prompts y errores |
| AWS | Claude Code, Bedrock y ambiente web | Conducir las etapas de la práctica, recorrer la sala, resolver acceso y problemas del ambiente |

**Control del tiempo:** cualquiera de los dos anuncia los checkpoints y activa el plan B (prompt exprés).

---

## 8. Prerrequisitos técnicos (enviar a participantes con 1 semana de anticipación)

AWS entrega a cada asistente una **cuenta AWS efímera** con su propio Amazon Bedrock y un **ambiente web en el navegador con Claude Code listo para usar**. El ambiente arranca vacío y el participante no instala nada.

- **Laptop en buen estado**, con batería cargada y cargador.
- **Acceso a internet estable**, sin bloqueos de proxy o firewall corporativo hacia el ambiente web de AWS.
- **Navegador moderno** actualizado (Chrome, Edge o Firefox).

**Verificación rápida antes del taller:** abrir el enlace del ambiente que comparte AWS e iniciar sesión con las credenciales entregadas.

---

## 9. Preparación del equipo facilitador (T-7 días)

- [ ] **Ensayo completo de los 3 casos desde cero en el ambiente de AWS**, cronometrando cada etapa contra la agenda.
- [ ] Guardar una **solución de referencia** por caso (specs + código) como respaldo para demos o rescates.
- [ ] Confirmar con AWS las cuentas efímeras: una por asistente (más cuentas de respaldo), cómo se entregan las credenciales y Claude Code autenticado contra el Bedrock de cada cuenta.
- [ ] Confirmar con AWS que el ambiente trae Node.js 20+ y acceso al registro npm, y que la app del participante (puerto 3000) se puede abrir en el navegador.
- [ ] Confirmar con AWS que en cada cuenta efímera el modelo Claude está habilitado en Bedrock y que sus cuotas por defecto aguantan una sesión intensa de Claude Code.
- [ ] Confirmar con AWS cuánto tiempo vive cada cuenta (que cubra los 105 minutos con margen) y cuándo se elimina.
- [ ] Validar que la red del recinto permite acceder al ambiente web.
- [ ] Diseñar las slides del bloque teórico a partir de `06_Presentacion_Teoria_SDD.md` y completar el texto marcado [COMPLETAR] (slide 17, AWS) y los logos de la slide 2.
- [ ] Preparar encuesta de cierre.

---

## 10. Criterios de éxito del taller

| Indicador | Meta |
|---|---|
| Participantes con app funcionando al cierre de la implementación | ≥ 80% |
| Escenarios de usuario final aprobados por participante | ≥ 80% |
| Specs con IDs y matriz de trazabilidad completa | 100% de quienes llegan a implementación |
| Satisfacción (encuesta de cierre) | ≥ 4,5 / 5 |

---

## 11. Riesgos y mitigaciones

| Riesgo | Mitigación |
|---|---|
| Falla de acceso al ambiente web | Cuentas efímeras de respaldo preparadas por AWS; trabajo en parejas |
| Cuotas por defecto de Bedrock bajas en una cuenta nueva | AWS valida modelo habilitado y cuotas en el ensayo; cuentas de respaldo |
| El participante pierde su trabajo al eliminarse la cuenta | En el cierre, quien quiera conservar su proyecto lo descarga antes de salir |
| Solo dos facilitadores para toda la sala | Limitar el cupo a 15–20 participantes; parejas espontáneas; prompt exprés y soluciones de referencia listas |
| `npm install` falla en el ambiente | Modo sin dependencias (`node:http`) del Manifiesto Técnico |
| La app no se puede abrir en el navegador del ambiente | Validarlo con AWS en el ensayo; como contingencia, probar escenarios vía `curl` y `npm test` |
| Participante atrasado | Prompt exprés de la guía; priorizar flujo principal sobre estética |
| Expansión de alcance ("le agrego esto") | Regla del `CLAUDE.md`: si no está en el spec, no se construye |
| Claude inventa reglas | Instrucción de marcar [SUPUESTO] y gate de revisión humana antes de aprobar |

---

*Taller co-dictado por GFT Technologies & AWS. Documento de estructura y agenda.*
