# PRD — Sistema de Aprendizaje Adaptativo Basado en Grafos de Micronodos y Macronodos Situados

| Campo | Valor |
|---|---|
| **Documento** | PRD (Product Requirements Document) |
| **Versión** | 1.0.0 |
| **Estado** | Aprobado para planificación (`/speckit.plan`) |
| **Fecha** | 2026-10-19 |
| **Constitución aplicable** | `.specify/memory/constitution.md` v1.0.0 (ratificada 2026-10-17) |
| **Fuentes** | `Revisión del plan de implementación y guía para Spec Kit.md`, Constitución v1.0.0, plantillas Spec Kit |
| **Documento relacionado** | `HISTORIAS-DE-USUARIO.md` v1.0.0 — 56 historias con escenarios Given/When/Then y su mapeo a los `FR-*` de este PRD |
| **Alcance de este documento** | Requerimientos funcionales y no funcionales por funcionalidad + orden de implementación |
| **Siguiente paso** | `/speckit.specify` por feature (7 features, ver Anexo A) → `/speckit.clarify` → `/speckit.plan` → `/speckit.tasks` |

### Fuentes del documento

| ID | Fuente | Qué aporta |
|---|---|---|
| SRC-01 | `.specify/memory/constitution.md` v1.0.0 | Principios I–V, restricciones técnicas, puertas de calidad y orden de referencia de fases |
| SRC-02 | `Revisión del plan de implementación y guía para Spec Kit.md` | Inconsistencias 1.1–1.15 (bugs a corregir), faltantes de la sección 2 y orden de fases sugerido |
| SRC-03 | `.specify/templates/*.md` (spec, plan, tasks, checklist) | Estructura y convenciones del flujo Spec Kit que el PRD alimenta |
| SRC-04 | `.specify/init-options.json` + `.specify/integration.json` | Numeración secuencial de features e integración con opencode |

---

## 1. Resumen ejecutivo

### 1.1 Problema

El estudiante avanza por una materia con un plan fijo (lineal). El sistema actual no conoce qué sabe el
estudiante, por lo que:

- repite ejercicios sobre conceptos ya dominados,
- avanza a conceptos sin los prerrequisitos suficientes,
- no puede explicar *por qué* un ejercicio aparece en ese momento,
- el docente no tiene visibilidad del dominio real por concepto, solo de la nota media.

### 1.2 Propuesta

Un motor adaptativo que modela la materia como un **grafo dirigido de micronodos** (conceptos atómicos)
agrupados en **macronodos** (unidades temáticas con contexto situado: rol, situación, herramienta y
artefacto del mundo real). Cada micronodo tiene:

- un modelo de dominio (*mastery*) estimado por **Bayesian Knowledge Tracing (BKT)** con parámetros
  calibrables por micronodo,
- un conjunto de prerrequisitos que el grafo respeta para no avanzar sobre huecos,
- un banco de reactivos validados simbólicamente.

El motor **registra cada interacción** del estudiante, **actualiza su dominio en una única fuente de
verdad (PostgreSQL)**, **selecciona el siguiente ejercicio según el grafo y el dominio**, y **genera
reactivos nuevos bajo demanda** con un LLM cuya salida nunca se trusts sin validación matemática.

### 1.3 Principio rector

> Todo el cálculo de dominio vive en una única fuente de verdad: la función BKT en PostgreSQL.
> Ningún reactivo llega al estudiante sin validación simbólica aprobada. Toda entrada de LLM es no
> confiable.

### 1.4 Qué entrega este PRD

1. Los **requerimientos funcionales y no funcionales** de las 7 funcionalidades del producto, con IDs
   estables (`FR-*`, `NFR-*`), criterios de aceptación y casos borde.
2. El **contrato de dominio** (modelo de datos, reglas del grafo, eventos, APIs) necesario para
   implementar sin ambigüedad.
3. El **orden de implementación explícito** (8 fases con entregables, dependencias y puertas de salida)
   con la matriz de trazabilidad requisito → fase.

---

## 2. Objetivos y no objetivos

### 2.1 Objetivos del MVP

| ID | Objetivo | Métrica de éxito |
|---|---|---|
| OBJ-01 | El estudiante siempre recibe un ejercicio cuyo concepto está en su "frente de aprendizaje" | 100% de las sesiones servidas cumplen la política de selección (FR-NAV-004) |
| OBJ-02 | El dominio del estudiante se mantiene actualizado y auditable | 100% de las interacciones registran una actualización de mastery (FR-INT-002) |
| OBJ-03 | Ningún reactivo incorrecto llega al estudiante | 0 reactivos sin validación simbólica aprobada en producción (NFR-SEC-002) |
| OBJ-04 | La primera pantalla de práctica es inmediata | p95 de entrega del primer ítem < 3 s con ítem cacheado (NFR-PERF-010) |
| OBJ-05 | El docente ve el dominio real por concepto | Panel con mastery por micronodo para 100% de los estudiantes de un curso (FR-DOC-002) |

### 2.2 No objetivos del MVP (fuera de alcance)

- Interfaz multi-idioma.
- Colaboración en tiempo real entre estudiantes.
- Integración con LMS externos (Moodle, Canvas) más allá de exportación CSV.
- Auto-calibración automática de parámetros BKT por machine learning (se permite calibración manual y
  ajuste por lotes; ver FR-BKT-009).
- gRPC entre servicios (excluido por la Constitución, principio IV).
- Motor de generación de reactivos multimodales (imágenes, audio).

---

## 3. Usuarios, roles y permisos

| Rol | Descripción | Capacidades |
|---|---|---|
| **Estudiante** (`student`) | Usuario final de la sesión de práctica | Resolver ítems, ver su propio dominio, consultar su historial, exportar su progreso |
| **Docente** (`teacher`) | Responsable de uno o varios cursos | Ver dominio agregado de sus estudiantes, ver métricas de su curso, aprobar/rechazar reactivos, reasignar prerrequisitos |
| **Administrador** (`admin`) | Operador del sistema | Gestionar grafos y bancos de ítems, calibrar parámetros BKT, configurar ítems por defecto, revisar la DLQ |
| **Servicio** (`service`) | Identidad máquina a máquina (FastAPI, workers) | Invocar endpoints internos, publicar/consumir eventos con idempotencia |

Regla transversal: **la identidad del estudiante se deriva siempre del token (JWT/OIDC)**, nunca de un
campo enviado por el cliente (FR-AUTH-002, Constitución III).

---

## 4. Conceptos de dominio

### 4.1 Glosario

| Concepto | Definición |
|---|---|
| **Macronodo** | Unidad temática de la materia (ej.: "Ecuaciones de segundo grado"). Aporta el **contexto situado**: rol, situación, herramienta y artefacto del mundo real en el que se aplican sus conceptos. |
| **Micronodo** | Concepto atómico evaluable de forma independiente (ej.: "producto de raíces"). Pertenece a un macronodo. Es la unidad de dominio y de estimación de *mastery*. |
| **Arista de prerrequisito** | Relación dirigida `A → B`: dominar A es condición para desbloquear (*front*) de B. El grafo es un DAG. |
| **Mastery (`p`)** | Probabilidad de que el estudiante domine un micronodo, estimada por BKT en [0,1]. |
| **Dominado** | `mastery ≥ 0.95` (umbral configurable por parámetro global `MASTERY_DOMINADO`). |
| **Ítem / reactivo** | Ejercicio evaluable vinculado a un micronodo, con enunciado, opciones, solución y validación simbólica. |
| **Banco de ítems** | Conjunto de reactivos validados y publicados por micronodo, reutilizables. |
| **Sesión de práctica** | Secuencia de ítems servidos a un estudiante en un momento dado, con objetivo y cierre. |
| **Telemetría** | Eventos de comportamiento del cliente (no solo respuestas): foco, visibilidad, abandono, pistas. |
| **Frente de aprendizaje** | Conjunto de micronodos no dominados cuyos prerrequisitos sí están dominados (candidatos a siguiente ejercicio). |

### 4.2 Reglas del grafo (decisiones de diseño asumidas)

Estas reglas no estaban disponibles en el material heredado; se adoptan como **supuestos explícitos**
(ver §15, decisión pendiente P-01..P-04) y son la base de las funcionalidades F2 (navegación) y F3
(generación).

| ID | Regla |
|---|---|
| GRA-01 | El grafo de prerrequisitos es un **DAG** validado en el momento de escritura (no se permiten ciclos). |
| GRA-02 | Un micronodo `B` es desbloqueable cuando **todos** sus prerrequisitos directos tienen `mastery ≥ 0.95`. |
| GRA-03 | El **mastery de un macronodo** es la media ponderada por dificultad de sus micronodos: `mastery_macronodo = Σ(d_i · p_i) / Σ(d_i)`. |
| GRA-04 | **Decaimiento (olvido)**: al leer el dominio, `p_efectiva = p · exp(-Δt / S)` con `S = 30 días` (vida media configurable `DECAY_HALF_LIFE_DAYS`). Se aplica **en lectura**, nunca en escritura, para que el trigger sea determinista y reproducible. |
| GRA-05 | Un micronodo se considera **dominado** con `p ≥ 0.95`; `0.80 ≤ p < 0.95` se considera *en refuerzo* y sigue siendo candidato prioritario a reaparecer. |
| GRA-06 | **Nodo situado**: cada macronodo declara `situated_context` (rol, situación, herramienta, artefacto). Ese contexto se inyecta en el prompt de generación y se muestra al estudiante en la cabecera del ejercicio. |
| GRA-07 | La selección de ítem usa la **política de frente de aprendizaje**: de los micronodos desbloqueables, se prioriza por (1) mayor déficit de mastery, (2) mayor peso de decaimiento, (3) mayor dificultad, (4) menor tiempo desde la última práctica. Desempate determinista por `micronode_id` para reproducibilidad. |

### 4.3 Modelo de datos (resumen normativo)

El DDL completo es entregable de la **Fase 1**. Este resumen fija los nombres y las relaciones que el
resto del PRD asume.

| Entidad | Campos clave | Notas |
|---|---|---|
| `students` | `id`, `external_auth_id` (único), `display_name`, `role`, `course_id`, `created_at` | `external_auth_id` = `sub` del JWT |
| `courses` | `id`, `name`, `subject_id`, `teacher_id` | Agrupa estudiantes y macronodos |
| `subjects` | `id`, `name` | Materia |
| `macronodes` | `id`, `course_id`, `code`, `title`, `order_index`, `situated_context` (JSONB) | `situated_context`: `{rol, situacion, herramienta, artefacto}` |
| `micronodes` | `id`, `macronode_id`, `code`, `title`, `learning_objective`, `difficulty` (1–5), `estimated_minutes` | `difficulty` es peso en GRA-03 |
| `micronode_edges` | `micronode_id`, `prereq_micronode_id`, `weight` | PK compuesta; validación DAG |
| `bkt_params` | `micronode_id` (PK), `p_L0`, `p_T`, `p_S`, `p_G`, `updated_at` | Parámetros **por micronodo** (Constitución I) |
| `items` | `id`, `micronode_id`, `kind`, `statement`, `options` (JSONB), `correct_option_id`, `symbolic_spec` (JSONB), `distractors` (JSONB), `hint`, `explanation`, `difficulty`, `status`, `validation_report` (JSONB), `source`, `llm_model`, `generation_cost`, `created_at` | `status ∈ {draft, validated, rejected, published, archived}` |
| `interaction_logs` | `id`, `student_id`, `item_id`, `micronode_id`, `correct`, `response_time_ms`, `hints_used`, `attempts`, `client_seq`, `client_ts_ms`, `server_ts` | **Particionada por rango mensual**; la PK incluye la clave de partición (Constitución, Restricciones) |
| `student_mastery` | `student_id`, `micronode_id` (PK), `p_known`, `n_attempts`, `n_correct`, `updated_at` | **Fuente única de verdad** |
| `student_mastery_macro` | `student_id`, `macronode_id` (PK), `p_known`, `updated_at` | Proyectado de GRA-03 |
| `practice_sessions` | `id`, `student_id`, `course_id`, `goal`, `started_at`, `ended_at`, `status`, `items_served`, `items_answered` | |
| `telemetry_events` | `id`, `student_id`, `session_id`, `event_type`, `payload` (JSONB), `client_ts_ms`, `server_ts` | Eventos no-respuesta |
| `outbox_events` | `id`, `event_type`, `aggregate_id`, `payload`, `correlation_id`, `created_at`, `published_at`, `attempts` | Patrón Outbox (FR-EVT-003) |
| `generation_jobs` | `id`, `micronode_id`, `status`, `requested_by`, `attempts`, `last_error`, `created_at`, `finished_at` | Estado de la generación asíncrona |
| `inbox_messages` | `consumer`, `message_id` (PK), `processed_at` | Idempotencia de consumidores |

### 4.4 Parámetros BKT (fórmulas normativas)

Con `p` = mastery actual, `c` = 1 si la respuesta es correcta y 0 si es incorrecta, y los parámetros del
micronodo (`p_L0` inicial, `p_T` transición, `p_S` slip, `p_G` guess):

```
POSTERIOR CON RESPUESTA CORRECTA (c = 1):
  numerador   = p · (1 - p_S) + (1 - p) · p_T
  denominador = p · (1 - p_S) + (1 - p) · p_T + (1 - p) · p_G
  p' = numerador / denominador

POSTERIOR CON RESPUESTA INCORRECTA (c = 0):
  numerador   = p · p_S
  denominador = p · p_S + (1 - p) · p_T
  p' = numerador / denominador
```

Restricciones normativas (Constitución I):

- `p_T ∈ (0, 1)`, `p_S ∈ (0, 1)` y `p_G ∈ (0, 1)` con **epsilon mínimo 1e-6**: nunca se permite
  `p_S = 0`, `p_G = 0` ni `p_T = 0`. Con `p_T ≥ 1e-6` el denominador es estrictamente positivo en
  ambos casos, y con `p_S ≥ 1e-6` se evita la degeneración `p = 1` (división por cero en la rama
  incorrecta). La validación del `CHECK` de la tabla lo garantiza.
- `p' ∈ [0, 1]`, calculado en `NUMERIC(12,8)` **sin truncamiento intermedio**; el redondeo a 8 decimales
  ocurre **solo al persistir**.
- La **inferencia** (`p'`) y el **decaimiento** (GRA-04) son operaciones separadas y auditables:
  el trigger solo infiere; el decaimiento se aplica en lectura.
- `p_L0` es la mastery inicial (por defecto 0.1) usada al crear la fila de `student_mastery` en la
  primera interacción.

---

## 5. Arquitectura de referencia (contexto de planificación)

> Este apartado es **contexto para `/speckit.plan`**, no un requisito de producto. Los requisitos
> son agnósticos al stack; la Constitución fija estas decisiones de infraestructura.

```
Angular (SPA)  ──HTTP/JSON──►  Spring Boot (Core API)  ──JDBC/JPA──►  PostgreSQL
                                      │                              ▲
                                      │ (contrato interno + cola)     │ trigger BKT
                                      ▼                              │
                               RabbitMQ  ◄── Outbox ────────────────┘
                                      │
                                      ▼
                               FastAPI (Generation Service) ──► LLMProvider (Gemini / Ollama)
                                      │                                    (structured output)
                                      ▼
                                 Validador SimPy seguro (sandbox, timeout)
```

| Decisión | Valor | Origen |
|---|---|---|
| Core API | Spring Boot 3 + Java 21 (virtual threads) | Revisión §1.10 |
| Servicio de generación | FastAPI + Python 3.12 | Revisión §1.1 |
| Base de datos | PostgreSQL 16 (+ `pgTAP` o Testcontainers) | Revisión §2 |
| Frontend | Angular 18+, Signals, zoneless, NgRx SignalStore | Revisión §1.9 |
| Broker | RabbitMQ (único broker en MVP) | Constitución IV |
| Streaming | SSE (sin gRPC en MVP) | Constitución IV |
| Trazas | OpenTelemetry con `traceId` propagado por la cola | Constitución, Observabilidad |
| Despliegue | Docker + Kubernetes, Ingress, probes, `NetworkPolicy`, PDB | Revisión §1.10 |

---

## 6. Funcionalidades y requerimientos

Cada funcionalidad sigue la misma plantilla:
**Descripción → Historias de usuario → Requerimientos funcionales → Requerimientos no funcionales →
Criterios de aceptación → Casos borde → Dependencias → Fuera de alcance**.

---

### F1 — Registro de interacciones y estimación del dominio (BKT)

**ID funcional:** `F1` · **Prioridad global:** P1 · **Riesgo:** Alto (núcleo pedagógico)

#### F1.1 Descripción

El sistema registra cada intento del estudiante sobre un ítem y actualiza, **en PostgreSQL y de forma
atómica con el registro**, la probabilidad de dominio (*mastery*) del micronodo evaluado. El cálculo se
realiza exclusivamente en la función BKT de base de datos, parametrizada por micronodo.

#### F1.2 Historias de usuario

| Historia | Prioridad | Testeable de forma independiente |
|---|---|---|
| **US-1.1** El estudiante responde un ítem y su dominio se actualiza al instante | P1 | Sí: enviar una interacción por API y consultar `student_mastery` |
| **US-1.2** El docente consulta el dominio real de un estudiante por concepto | P1 | Sí: endpoint de consulta con datos sembrados |
| **US-1.3** El sistema es auditable: cada cambio de mastery es trazable a una interacción | P2 | Sí: verificar FK y recálculo desde `interaction_logs` |
| **US-1.4** El admin calibra los parámetros BKT de un micronodo | P2 | Sí: endpoint de calibración + efecto observable en inferencias posteriores |

#### F1.3 Requerimientos funcionales

| ID | Requisito |
|---|---|
| FR-INT-001 | El sistema MUST registrar una interacción con: `student_id` (derivado del JWT), `item_id`, `micronode_id` (derivado del ítem, nunca del cliente), `correct`, `response_time_ms`, `hints_used`, `attempts`, `client_seq`, `client_ts_ms` y `server_ts` (timestamp del servidor). |
| FR-INT-002 | El registro de la interacción y la actualización de `student_mastery` MUST ocurrir en la **misma transacción**. Si la inferencia BKT falla, la interacción no se persiste. |
| FR-INT-003 | El sistema MUST mantener `student_mastery` como fuente **única** del dominio; ninguna otra tabla, servicio o cliente puede escribir `p_known` directamente. |
| FR-INT-004 | El trigger de inferencia MUST dispararse en la **inserción** de `interaction_logs` (AFTER INSERT) y MUST ser idempotente: reprocesar la misma interacción no puede alterar el resultado (garantizado por `client_seq` + `inbox_messages`/constraint de unicidad por `(student_id, client_seq)`). |
| FR-INT-005 | El sistema MUST derivar `micronode_id` a partir de `item_id` en servidor; un cliente MUST NOT poder enviar un `micronode_id` contradictorio. |
| FR-INT-006 | El sistema MUST aceptar reintentos del cliente con el mismo `client_seq` como idempotentes, devolviendo el resultado original sin recalcular. |
| FR-INT-007 | El sistema MUST exponer el dominio del estudiante por micronodo y por macronodo, aplicando el decaimiento GRA-04 en lectura, e indicar si cada micronodo está `dominado`, `en_reinforcement` o `pendiente` según GRA-05. |
| FR-INT-008 | El sistema MUST recalcular y persistir `student_mastery_macro` según GRA-03 tras cada inferencia, dentro de la misma transacción. |
| FR-INT-009 | El sistema MUST registrar en `interaction_logs` la interacción aunque el ítem esté `archived`, y MUST NOT computar dominio para ítems en estado `rejected` o `draft`. |
| FR-INT-010 | El sistema MUST exponer un endpoint de recalculado masivo (re-derivación de `student_mastery` desde `interaction_logs`) para auditoría y calibración, ejecutable solo por rol `admin` y sin impacto en el servicio en línea. |
| FR-BKT-001 | La función BKT MUST leer sus parámetros (`p_L0`, `p_T`, `p_S`, `p_G`) de `bkt_params` para el micronodo evaluado, con fallback a una fila de parámetros por defecto. MUST NOT usar constantes hardcodeadas en el código. |
| FR-BKT-002 | El cálculo MUST preservar precisión: `NUMERIC(12,8)` en almacenamiento, sin truncamiento en pasos intermedios, redondeo solo al persistir. |
| FR-BKT-003 | Las fórmulas MUST ser seguras ante casos límite: `p_S` y `p_G` con epsilon mínimo `1e-6` garantizado por `CHECK` en `bkt_params`; sin división por cero para ningún valor de `p` en `[0,1]`. |
| FR-BKT-004 | Los parámetros BKT MUST ser editables por rol `admin` con fecha, actor y valor anterior/posterior auditados. |
| FR-BKT-005 | La actualización de `bkt_params` MUST aplicarse solo a interacciones posteriores al cambio; las interacciones históricas NO se reinterpretan silenciosamente (el recálculo masivo FR-INT-010 es la única vía de reinterpretación). |
| FR-BKT-006 | El DDL MUST declarar explícitamente el `CREATE TRIGGER` (trigger, momento, función, condición `WHEN`), no solo la función. |
| FR-BKT-007 | Con `interaction_logs` particionada, la clave primaria y los índices únicos MUST incluir la clave de partición, y las claves foráneas MUST implementarse de forma compatible con el particionamiento (FK al micronodo/estudiante por id simple; unicidad por clave compuesta cuando aplique). |
| FR-BKT-008 | El sistema MUST exponer la métrica `mastery_delta` por interacción para observabilidad y diagnóstico. |
| FR-BKT-009 | El sistema MUST permitir calibración de parámetros por lotes (regla global o conjunto de micronodos) con validación de rangos y registro de auditoría. |
| FR-AUD-001 | El sistema MUST exponer al rol `teacher` (solo sobre sus cursos) y al rol `admin` la vista de dominio por estudiante y por concepto; un docente MUST NOT ver datos de cursos ajenos. |

#### F1.4 Requerimientos no funcionales

| ID | Requisito |
|---|---|
| NFR-PERF-001 | El registro de una interacción MUST completarse en p95 < 300 ms (excluyendo latencia de red del cliente). |
| NFR-PERF-002 | La lectura del dominio de un estudiante (grafo completo de su curso) MUST completarse en p95 < 200 ms. |
| NFR-CONS-001 | La inferencia MUST ser transaccional y consistente: si la transacción revierte, ni la interacción ni el mastery cambian. |
| NFR-DATA-001 | `interaction_logs` MUST particionarse por rango mensual con política de retención definida (ver P-05) y proceso de rotación de particiones automatizado. |
| NFR-SEC-001 | Los endpoints de dominio MUST exigir token válido y autorización por rol y por curso. |

#### F1.5 Criterios de aceptación

| ID | Criterio |
|---|---|
| AC-1.1 | Con un estudiante nuevo (`p = p_L0 = 0.1`) y una respuesta **incorrecta**, `p` disminuye según la fórmula BKT y `n_attempts = 1`, `n_correct = 0`. |
| AC-1.2 | Con una respuesta **correcta** repetida 30 veces y `p_T = 0.25`, `p_S = 0.1`, `p` converge a un valor > 0.95 y el micronodo pasa a estado `dominado`. |
| AC-1.3 | Con `p = 1.0` y respuesta incorrecta, el cálculo NO produce división por cero ni error; `p'` queda en `[0,1]`. |
| AC-1.4 | Enviar dos veces la misma interacción con el mismo `client_seq` deja **una** fila en `interaction_logs` y **un** único cambio de mastery. |
| AC-1.5 | Un token de un estudiante A no puede registrar una interacción para el estudiante B (el `student_id` del cuerpo se ignora o se rechaza con 403). |
| AC-1.6 | El dominio devuelto por la API aplica el decaimiento: tras 30 días sin práctica, la mastery efectiva es la mitad que en el momento de la última inferencia (±1 %). |
| AC-1.7 | El `mastery` de un macronodo coincide con la media ponderada por dificultad de sus micronodos (tolerancia 1e-6). |
| AC-1.8 | Un test automatizado verifica que `pg_trigger` existe con `AFTER INSERT` sobre `interaction_logs` y que su función está declarada en el esquema. |

#### F1.6 Casos borde

- La misma interacción llega con `client_seq` distinto (reintento no idempotente del cliente) → se
  registra como interacción nueva; se documenta que el cliente DEBE reutilizar `client_seq`.
- El ítem fue archivado entre que se sirvió y se respondió → se registra la interacción sin efecto en
  dominio (FR-INT-009) y se devuelve `204`.
- El `student_mastery` no existe todavía → se crea con `p = p_L0` y se infiere en el mismo paso.
- La partición del mes en curso no existe → el motor MUST crearla automáticamente (partición por defecto).
- El estudiante responde fuera de tiempo (abandono) → la interfaz registra el ítem como `skipped`, que
  **no** afecta al dominio pero sí a las métricas de sesión.

#### F1.7 Dependencias

- Fase 1 (esquema + trigger) es bloqueante para todo lo demás.
- No depende de LLM. Es la primera funcionalidad verificable end-to-end con un banco de ítems manual.

#### F1.8 Fuera de alcance

- Calibración automática por machine learning.
- Modelos de dominio alternativos (PFA, DKT) — la tabla `student_mastery` se dimensiona para permitir
  añadir una columna, pero no se implementan.

---

### F2 — Grafo de conocimiento, navegación y selección adaptativa

**ID funcional:** `F2` · **Prioridad global:** P1 · **Riesgo:** Medio

#### F2.1 Descripción

El sistema mantiene el grafo de micronodos y sus prerrequisitos, y decide **qué concepto trabaja el
estudiante a continuación** aplicando la política de frente de aprendizaje (GRA-07) y devolviendo un
ítem válido de ese concepto.

#### F2.2 Historias de usuario

| Historia | Prioridad | Testeable de forma independiente |
|---|---|---|
| **US-2.1** El estudiante avanza de concepto solo cuando domina los prerrequisitos | P1 | Sí: grafo sembrado + endpoint de navegación |
| **US-2.2** El estudiante ve en qué concepto está y por qué (contexto situado) | P1 | Sí: respuesta de navegación con contexto y razón |
| **US-2.3** El estudiante recibe un ejercicio de refuerzo si está "en refuerzo" (0.80 ≤ p < 0.95) | P2 | Sí: sembrar `p = 0.85` y verificar la selección |
| **US-2.4** El admin mantiene el grafo (micronodos, aristas, orden) | P2 | Sí: CRUD de grafo |

#### F2.3 Requerimientos funcionales

| ID | Requisito |
|---|---|
| FR-NAV-001 | El sistema MUST mantener micronodos con `code`, `title`, `learning_objective`, `difficulty` (1–5) y su macronodo. |
| FR-NAV-002 | El sistema MUST mantener aristas de prerrequisito entre micronodos y MUST **rechazar** cualquier arista que introduzca un ciclo (validación DAG, GRA-01). |
| FR-NAV-003 | El sistema MUST determinar el conjunto de micronodos desbloqueados: todos sus prerrequisitos directos con `mastery ≥ 0.95` (GRA-02). |
| FR-NAV-004 | La selección del siguiente micronodo MUST seguir la política GRA-07, de forma **determinista** (mismo estado → mismo resultado), y MUST ser auditable: la respuesta MUST incluir el `razonamiento` (política aplicada, valores de desempate). |
| FR-NAV-005 | El sistema MUST priorizar micronodos en estado *en refuerzo* (0.80 ≤ p < 0.95) por encima de micronodos nunca intentados cuando ambos están desbloqueados, salvo que el objetivo de la sesión indique lo contrario. |
| FR-NAV-006 | El sistema MUST devolver, junto con el ítem, el contexto situado del macronodo (rol, situación, herramienta, artefacto) y el objetivo de aprendizaje del micronodo (GRA-06). |
| FR-NAV-007 | Si no hay ítems publicados disponibles para el micronodo seleccionado, el sistema MUST iniciar la generación asíncrona del ítem y NOTIFICAR al cliente en lugar de bloquear la sesión (ver F3 y F5; FR-GEN-020). |
| FR-NAV-008 | Si no existe ningún micronodo desbloqueado (grafo saturado o grafo vacío), el sistema MUST devolver un estado explícito `sin_frente` con el macronodo dominado más reciente, para que la UI ofrezca repaso libre o repaso de macronodo dominados. |
| FR-NAV-009 | El sistema MUST excluir ítems ya respondidos correctamente en la misma sesión (variedad), salvo que la política de refuerzo los requiera. |
| FR-NAV-010 | El objetivo de sesión (`repaso`, `avance`, `diagnóstico`, `refuerzo`) MUST modificar la política de selección de forma explícita y documentada. |
| FR-GRAPH-001 | El rol `admin` MUST poder crear, editar y archivar micronodos, macronodos y aristas; el rol `teacher` MUST poder proponer aristas dentro de su curso, pendientes de aprobación. |
| FR-GRAPH-002 | El sistema MUST registrar auditoría de cambios del grafo (actor, fecha, antes/después). |
| FR-GRAPH-003 | El sistema MUST versionar el grafo por curso, de modo que un curso publicado no cambia retroactivamente al reordenar o editar micronodos. |

#### F2.4 Requerimientos no funcionales

| ID | Requisito |
|---|---|
| NFR-PERF-003 | El cálculo de la navegación MUST completarse en p95 < 150 ms con un curso de hasta 500 micronodos y 5 000 aristas. |
| NFR-PERF-004 | La política de selección MUST ser O(V + E) en el tamaño del grafo por curso; MUST NOT recalcular el dominio completo del estudiante en cada llamada. |
| NFR-DATA-002 | El grafo MUST estar cacheado en memoria con invalidación por evento de cambio (no re-consultar la base en cada navegación). |

#### F2.5 Criterios de aceptación

| ID | Criterio |
|---|---|
| AC-2.1 | Con micronodo `B` cuyo prerrequisito `A` tiene `p = 0.90`, `B` NO aparece en el frente de aprendizaje. |
| AC-2.2 | Con `A` en `p = 0.96`, `B` aparece en el frente y es el seleccionado si es el de mayor déficit. |
| AC-2.3 | Con dos candidatos empatados en déficit, dificultad y decaimiento, la selección es la misma en 100 ejecuciones consecutivas. |
| AC-2.4 | La respuesta de navegación incluye el contexto situado completo del macronodo y el objetivo de aprendizaje. |
| AC-2.5 | Intentar crear la arista `B → A` cuando existe `A → B` devuelve 409 con detalle del ciclo detectado. |
| AC-2.6 | Con todos los micronodos dominados, la navegación devuelve `sin_frente` con el macronodo dominado más reciente. |
| AC-2.7 | Con un micronodo desbloqueado sin ítems publicados, la navegación devuelve el estado "generando" y no un error. |

#### F2.6 Casos borde

- Grafo vacío (curso nuevo sin contenido) → `sin_frente` + mensaje accionable para el admin.
- Un micronodo con prerrequisitos en otro macronodo → el cruce de macronodos está permitido; el
  crecimiento del dominio del macronodo destino NO depende del origen (se evalúa por micronodo).
- Ciclo introducido por importación masiva → se rechaza toda la importación y se informa de las aristas
  conflictivas.
- El estudiante tiene `p = 0.95` exacto → dominado (umbral inclusivo, GRA-05).

#### F2.7 Dependencias

- F1 (dominio) es bloqueante: la navegación consume `student_mastery`.
- F3 (generación) es necesaria para la operación completa, pero no para verificar F2 con ítems sembrados.

---

### F3 — Generación y validación de reactivos

**ID funcional:** `F3` · **Prioridad global:** P1 · **Riesgo:** **Crítico** (correctitud matemática y
seguridad)

#### F3.1 Descripción

El sistema genera reactivos nuevos para un micronodo mediante un LLM, valida **simbólicamente** que el
reactivo y su solución son matemáticamente correctos, y solo entonces lo publica en el banco de ítems.
Si la validación falla, se reflexiona (reinyectando el error en el prompt) y se reintenta; si tras N
intentos no hay éxito, se usa el fallback del banco pregenerado.

#### F3.2 Historias de usuario

| Historia | Prioridad | Testeable de forma independiente |
|---|---|---|
| **US-3.1** El estudiante recibe un ejercicio nuevo para su concepto y puede responderlo | P1 | Sí: con un LLM stub determinista y un banco de ítems |
| **US-3.2** Un reactivo con solución matemática incorrecta nunca se publica | P1 | Sí: se inyecta un LLM stub que devuelve una solución errónea |
| **US-3.3** Un reactivo con prompt injection (`__import__('os')`) no ejecuta código | P1 | Sí: test de seguridad con payload real |
| **US-3.4** El docente/admin ve el motivo de rechazo de un reactivo | P2 | Sí: endpoint de revisión con el informe de validación |

#### F3.3 Requerimientos funcionales

| ID | Requisito |
|---|---|
| FR-GEN-001 | El sistema MUST exponer una abstracción `LLMProvider` con implementaciones intercambiables (Gemini, Ollama) y MUST NOT mezclar proveedores sin esa interfaz (Constitución III). |
| FR-GEN-002 | El sistema MUST obtener del LLM **structured output** validado contra un JSON Schema estricto. El parseo de texto libre MUST NOT aceptarse; si el proveedor no soporta structured output, el sistema MUST extraer y validar el bloque JSON contra el esquema, y ante fallo MUST descartar la respuesta. |
| FR-GEN-003 | El esquema del reactivo MUST incluir, como mínimo: `statement` (enunciado), `options` (opciones con `id` y `text`), `correct_option_id`, `distractors` (justificación de cada distractor), `symbolic_spec` (ecuación/expresión, símbolo, dominio, solución esperada), `hint`, `explanation`, `difficulty`. |
| FR-GEN-004 | El prompt MUST incluir: el micronodo (objetivo de aprendizaje, dificultad), el **contexto situado** del macronodo (GRA-06), los parámetros del ítem, 2–3 ítems de ejemplo del banco y las restricciones de formato. |
| FR-GEN-005 | El validador MUST usar parseo seguro: `parse_expr` con `local_dict`/`global_dict` restringidos y `transformations` controladas, o gramática/AST con allowlist. `sympify` con `evaluate=False` MUST NOT usarse como mecanismo de seguridad (Constitución II). |
| FR-GEN-006 | El validador MUST ejecutarse bajo un **timeout estricto** (por defecto 2 s) y MUST cancelar el cómputo al superarlo; una expresión patológica no puede consumir CPU indefinidamente. |
| FR-GEN-007 | El validador MUST derivar las **variables libres** de la expresión (nada de `x` hardcodeado) y MUST resolver la ecuación en todas las variables declaradas. |
| FR-GEN-008 | El validador MUST comparar contra el **conjunto completo de soluciones** (cuadráticas y superiores), no solo la primera; una respuesta correcta alternativa MUST aceptarse. |
| FR-GEN-009 | El validador MUST normalizar el enunciado antes de parsear: admitir **un único** `=` como igualdad; tratar `<=`, `>=`, `==`, `≠` de forma explícita; rechazar entradas ambiguas con motivo tipificado. |
| FR-GEN-010 | El validador MUST tipificar sus excepciones (`SympyValidationError`, `SympyTimeoutError`, `SympyParseError`, `SympyUnsupportedExprError`) y MUST NOT capturar `Exception` de forma genérica que oculte la causa (Constitución II). |
| FR-GEN-011 | El sistema MUST distinguir la causa del fallo: **red / rate limit / 5xx** → reintento con backoff exponencial y jitter; **validación matemática** → bucle explícito de reflexión que reinyecta el error y el contexto en el prompt siguiente, **sin** backoff temporal. |
| FR-GEN-012 | El bucle de reflexión MUST mantener historial de conversación (prompt, respuesta, error, correcciones) y MUST limitarse a `N` intentos (por defecto 3). |
| FR-GEN-013 | Todo reactivo MUST registrar su `validation_report` (checks ejecutados, resultado, motivo de rechazo, soluciones encontradas, tiempo de cómputo) para auditoría y para el panel docente. |
| FR-GEN-014 | Un reactivo MUST permanecer en `draft` hasta superar la validación; solo entonces pasa a `validated` y puede publicarse (`published`) para servirse a estudiantes. |
| FR-GEN-015 | El sistema MUST aplicar límites de coste y cuota: coste máximo por ítem, coste máximo por curso/día y cuota de peticiones al proveedor; al superar el límite MUST detenerse y reportar en vez de degradar silenciosamente. |
| FR-GEN-016 | La calidad pedagógica MUST evaluarse **separadamente** de la corrección matemática: el sistema MUST registrar señales de calidad (claridad del enunciado, plausibilidad de los distractores, duplicidad textual) y exponerlas al docente; SymPy MUST NOT considerarse evidencia de calidad didáctica. |
| FR-GEN-017 | El sistema MUST detectar ítems duplicados o semánticamente redundantes (hash de enunciado normalizado + similitud con ítems publicados del mismo micronodo). |
| FR-GEN-018 | El sistema MUST almacenar el ítem con su `llm_model`, `prompt_version` y `generation_cost` para trazabilidad y análisis de coste. |
| FR-GEN-019 | El banco de ítems MUST soportar **pre-generación** por lotes (por micronodo, por curso) como camino principal de aprovisionamiento. |
| FR-GEN-020 | Tras `N` intentos fallidos, el sistema MUST aplicar el **fallback**: servir un ítem pregenerado del mismo micronodo con dificultad equivalente; si no existe, MUST notificar al cliente con un estado de error explícito y accionable (nunca un ítem no validado). |
| FR-GEN-021 | El sistema MUST registrar todo reactivo descartado con su motivo (reject reason codes: `sympy_invalid`, `sympy_timeout`, `schema_invalid`, `duplicate`, `quality_flags`, `quota_exceeded`, `provider_error`). |
| FR-GEN-022 | El sistema MUST ser **determinista en las pruebas**: los tests MUST poder inyectar un proveedor stub y fijar `temperature = 0`, semilla y versión de prompt. |
| FR-GEN-023 | El servicio de generación MUST NOT ejecutarse en el event loop de forma bloqueante: las llamadas de red y el cómputo simbólico MUST delegarse en hilos o procesos dedicados (Constitución IV). |
| FR-GEN-024 | El sistema MUST sanitizar el `statement` y el `explanation` antes de persistirlos: sin HTML ejecutable ni scripts, longitud máxima acotada, saneo de prompt injection evidente en el contenido de salida. |
| FR-GEN-025 | El sistema MUST registrar la `correlation_id` y el `traceId` en toda generación, y propagarlos al evento emitido y a los logs del proveedor. |

#### F3.4 Requerimientos no funcionales

| ID | Requisito |
|---|---|
| NFR-SEC-002 | El servicio de validación MUST ejecutarse con las mitigaciones de la Constitución II: parseo con allowlist, timeout, límites de tamaño de entrada y aislamiento del proceso. |
| NFR-SEC-003 | El servicio de generación MUST exigir autenticación de servicio; sus endpoints MUST NOT ser alcanzables desde Internet sin la gateway. |
| NFR-SEC-004 | La API key del proveedor MUST almacenarse como secreto (Kubernetes `Secret` / gestor de secretos) y MUST NOT aparecer en logs, repositorio ni respuestas de error. |
| NFR-PERF-005 | Una generación en línea MUST completarse en p95 < 15 s con proveedor ágil; MUST reportarse el p95 de entrega del primer ítem. |
| NFR-COST-001 | El coste medio por ítem generado MUST ser medido y comparado contra un umbral configurable; el coste acumulado por curso/día MUST ser visible para `admin`. |
| NFR-QUAL-001 | La tasa de reactivos descartados MUST ser < 15% en el banco pregenerado y < 25% en generación en línea (ver SLO-003). |

#### F3.5 Criterios de aceptación

| ID | Criterio |
|---|---|
| AC-3.1 | Un ítem con `correct_option_id` cuya solución NO satisface la ecuación del enunciado es rechazado con `sympy_invalid` y NO aparece en el banco publicado. |
| AC-3.2 | Un ítem con `2x² - 3x - 2 = 0` y soluciones `{2, -0.5}` acepta CUALQUIERA de las dos respuestas del estudiante. |
| AC-3.3 | Un ítem con `y = x² + 1` (variable distinta de `x`) se valida correctamente y no produce soluciones erróneas por asunción de variable. |
| AC-3.4 | Una expresión patológica (producto de matrices symbolicamente explosivo) termina en < 2.5 s con `SympyTimeoutError` registrado y CPU liberada. |
| AC-3.5 | Un payload con `__import__('os').system('...')` dentro del enunciado es rechazado en parseo sin ejecutar nada y sin efectos observables. |
| AC-3.6 | Una respuesta del LLM con ```json fences ``` se procesa correctamente; una respuesta que no cumple el JSON Schema se descarta con `schema_invalid`. |
| AC-3.7 | Un error de validación MUST reinyectar el motivo en el siguiente prompt (verificable en el historial del proveedor stub) y MUST NOT esperar backoff temporal. |
| AC-3.8 | Un 429 del proveedor MUST reintentar con backoff exponencial y jitter y MUST respetar el header `Retry-After`. |
| AC-3.9 | Tras 3 intentos fallidos y sin ítems de fallback, el cliente recibe un estado explícito de indisponibilidad; no se sirve ningún ítem no validado. |
| AC-3.10 | Un ítem ya publicado con el mismo enunciado normalizado no se vuelve a publicar (código `duplicate`). |
| AC-3.11 | El `validation_report` de cada ítem contiene la lista de checks ejecutados, su resultado y el motivo; es consultable por el docente. |

#### F3.6 Casos borde

- El LLM devuelve JSON válido pero con `options` de 2 elementos y `correct_option_id` inexistente → rechazo `schema_invalid` con mensaje específico.
- El enunciado contiene dos `=` (`x = 2 = 3`) → rechazo `sympy_parse_error` (motivo: igualdad ambigua).
- La ecuación no tiene solución real (`x² + 1 = 0`) → rechazo con motivo `no_real_solution` (o `complex_solution` si el dominio declarado es complejo).
- La solución declarada usa decimales (`0.5`) frente a fracciones (`1/2`) → MUST aceptarse tras normalización.
- El proveedor devuelve 200 con cuerpo vacío → `provider_error`, reintento con backoff.
- El presupuesto de coste se agota a mitad de un lote → el lote se detiene en un ítem, no se corta a mitad de ítem; los ítems ya validados se conservan.

#### F3.7 Dependencias

- F2 (grafo) aporta el micronodo y el contexto situado al prompt.
- F7 (asincronía y operación) es necesaria para el modo en línea; en el MVP inicial la generación puede ser
  síncrona desde un worker de lotes (fuera del event loop).

#### F3.8 Fuera de alcance

- Validación semántica en lenguaje natural ("¿el enunciado está bien redactado?") más allá de heurísticas
  de calidad y revisión docente.
- Generación de reactivos con imágenes o gráficos.

---

### F4 — Sesión de práctica y experiencia del estudiante (frontend)

**ID funcional:** `F4` · **Prioridad global:** P1 · **Riesgo:** Medio

#### F4.1 Descripción

El estudiante practica en una sesión guiada por el motor adaptativo: recibe un ítem con su contexto,
responde, recibe feedback inmediato y su dominio se actualiza. La sesión es telemetrada de forma fiable.

#### F4.2 Historias de usuario

| Historia | Prioridad | Testeable de forma independiente |
|---|---|---|
| **US-4.1** El estudiante inicia una sesión y recibe su primer ejercicio en < 3 s | P1 | Sí: e2e con backend sembrado |
| **US-4.2** El estudiante ve feedback inmediato (correcto/incorrecto + explicación) | P1 | Sí: e2e |
| **US-4.3** El estudiante ve su progreso por concepto | P2 | Sí: e2e sobre vista de dominio |
| **US-4.4** El estudiante puede usar pistas sin penalización injusta | P2 | Sí: e2e |

#### F4.3 Requerimientos funcionales

| ID | Requisito |
|---|---|
| FR-UX-001 | La sesión de práctica MUST consistir en: inicio de sesión → objetivo → ítems servidos → cierre con resumen. |
| FR-UX-002 | La pantalla del ítem MUST mostrar: enunciado, opciones, contexto situado (rol/situación/herramienta), objetivo de aprendizaje, indicador de nivel del concepto y contador de progreso de sesión. |
| FR-UX-003 | Al responder, la UI MUST mostrar feedback inmediato (correcto/incorrecto), la explicación y, si aplica, la pista. |
| FR-UX-004 | La UI MUST indicar el estado de dominio del concepto (pendiente / en refuerzo / dominado) sin revelar el valor numérico de *mastery* al estudiante (decisión de producto, ver P-06). |
| FR-UX-005 | El cliente MUST enviar las interacciones con `client_seq` monótono por sesión para idempotencia y MUST permitir reintento seguro. |
| FR-UX-006 | El cliente MUST NOT enviar `student_id` como fuente de autoridad; el servidor MUST derivarlo del token y el cliente MUST ignorarlo si aparece en la respuesta de la API. |
| FR-UX-007 | El cliente MUST medir tiempos con alta resolución (`performance.now()`) y enviar además un `client_ts_ms` de pared; el servidor MUST usar su propio timestamp como referencia autoritativa. |
| FR-UX-008 | El cliente MUST mantener una **cola local persistente** de eventos pendientes (almacenamiento local) y reintentarlos con backoff hasta su confirmación, ante fallo de red o cierre de la pestaña. |
| FR-UX-009 | Al cerrar o ocultar la pestaña, el cliente MUST enviar la telemetría pendiente mediante `navigator.sendBeacon` (o equivalente) para evitar la pérdida de datos del `bufferTime`. |
| FR-UX-010 | El cliente MUST NOT perder eventos por el uso de buffers temporales; la estrategia de agrupación MUST garantizar entrega o reintento persistente. |
| FR-UX-011 | El cliente MUST ofrecer una acción de "saltar" que registra el ítem como `skipped` (sin efecto en el dominio). |
| FR-UX-012 | El cliente MUST mostrar un estado de carga y un estado de error accionable si el ítem no está disponible (generación en curso o indisponibilidad del servicio). |
| FR-UX-013 | El cliente MUST ser accesible: navegación por teclado, contraste AA, etiquetas ARIA en controles de respuesta, y feedback no exclusivamente cromático. |
| FR-UX-014 | El cliente MUST funcionar en modo responsive (móvil y escritorio) y MUST ser usable con lector de pantalla. |
| FR-UX-015 | La UI MUST ser resiliente a respuestas lentas: mientras espera, el estudiante puede cancelar la solicitud en curso sin bloquear la sesión. |
| FR-UX-016 | El cierre de sesión MUST mostrar: ítems respondidos, aciertos, conceptos trabajados, conceptos dominados ganados y tiempo. |

#### F4.4 Requerimientos no funcionales

| ID | Requisito |
|---|---|
| NFR-PERF-006 | La UI MUST renderizar el primer ítem en menos de 1.5 s tras la respuesta del backend (LCP de la vista de práctica). |
| NFR-PERF-007 | El bundle inicial MUST ser < 300 KB gzip; el resto MUST cargarse de forma diferida. |
| NFR-PERF-008 | El cliente MUST usar Signals y evitar `runOutsideAngular` salvo para listeners de eventos DOM o timers (uso incorrecto en el diseño original, revisión §1.9). |
| NFR-REL-001 | La telemetría MUST tener una tasa de pérdida efectiva < 0.5% medida por comparación entre eventos enviados y confirmados. |
| NFR-A11Y-001 | La vista de práctica MUST cumplir WCAG 2.1 AA en sus flujos críticos. |
| NFR-SEC-005 | El token MUST almacenarse de forma que no sea accesible desde JavaScript no confiable (preferible cookie `HttpOnly` + CSRF, o almacenamiento con TTL corto y rotación). |

#### F4.5 Criterios de aceptación

| ID | Criterio |
|---|---|
| AC-4.1 | Con ítem cacheado, el estudiante ve el primer ejercicio en menos de 3 s desde el inicio de sesión (p95). |
| AC-4.2 | Al responder correctamente, el feedback correcto aparece en < 300 ms. |
| AC-4.3 | Con la red caída, la interacción queda en cola local y se entrega al recuperar la conexión, sin duplicados (idempotencia por `client_seq`). |
| AC-4.4 | Al cerrar la pestaña con eventos pendientes, el evento llega al servidor (verificable con `sendBeacon` en test de navegador). |
| AC-4.5 | La navegación completa del flujo de práctica es realizable solo con teclado. |
| AC-4.6 | Un token de estudiante A no puede registrar respuestas en nombre de B desde el cliente. |

#### F4.6 Casos borde

- El ítem llega pero el cliente se queda sin red → cola local + indicador "sin conexión".
- El servidor devuelve `409` por `client_seq` duplicado → el cliente interpreta como idempotente OK, no como error.
- El estudiante responde el mismo ítem 3 veces (reintentos) → `attempts` se incrementa, solo la última cuenta para el dominio.
- El ítem no tiene opciones (respuesta abierta numérica) → la UI MUST permitir entrada numérica con validación en cliente.

#### F4.7 Dependencias

- F1 (interacciones), F2 (navegación), F3 (ítems) y F7 (streaming) son todas precondición para la
  experiencia completa; el flujo es verificable en la Fase 3 con ítems sembrados.

---

### F5 — Telemetría del estudiante

**ID funcional:** `F5` · **Prioridad global:** P2 · **Riesgo:** Medio

#### F5.1 Descripción

El cliente emite eventos de comportamiento (no solo respuestas): foco en la ventana, cambio de pestaña,
abandono, uso de pistas, tiempo por ítem. El backend los persiste de forma fiable y los usa para
métricas de producto y de calidad de la experiencia.

#### F5.2 Historias de usuario

| Historia | Prioridad | Testeable de forma independiente |
|---|---|---|
| **US-5.1** El admin ve la tasa de abandono por ítem y por concepto | P2 | Sí: con eventos sembrados |
| **US-5.2** El sistema no pierde eventos al cerrar la pestaña | P1 | Sí: test de navegador con `sendBeacon` |
| **US-5.3** El docente ve señales de ítems confusos (muchos intentos, muchas pistas) | P3 | Sí: agregación sobre `interaction_logs` |

#### F5.3 Requerimientos funcionales

| ID | Requisito |
|---|---|
| FR-TEL-001 | El cliente MUST emitir al menos: `session_start`, `session_end`, `item_presented`, `item_answered`, `item_skipped`, `hint_requested`, `tab_hidden`, `window_blurred`, `copy_paste_detected` (si procede). |
| FR-TEL-002 | Cada evento MUST incluir `client_seq`, `session_id`, `event_type`, `client_ts_ms`, `payload` y `schema_version`. |
| FR-TEL-003 | El cliente MUST usar una cola persistente local con reintento exponencial y, ante `visibilitychange`, MUST drenar por `sendBeacon`. |
| FR-TEL-004 | El servidor MUST aceptar eventos de forma **idempotente** por `(student_id, client_seq)`. |
| FR-TEL-005 | El servidor MUST validar el `event_type` contra una allowlist y rechazar eventos desconocidos con código `unknown_event_type` (sin romper el flujo del cliente). |
| FR-TEL-006 | El servidor MUST registrar `server_ts` de recepción, de modo que el análisis de latencia use tiempo de servidor. |
| FR-TEL-007 | El backend MUST exponer métricas agregadas: abandono por ítem/concepto, tiempo medio por ítem, ratio de pistas, ratio de intentos múltiples. |
| FR-TEL-008 | La telemetría MUST NOT influir en la evaluación: los eventos no alteran el dominio ni la nota. |
| FR-TEL-009 | El sistema MUST respetar la minimización de datos: la telemetría MUST NOT incluir texto libre del estudiante ni contenido de las respuestas. |
| FR-TEL-010 | El cliente MUST enviar `studentId` únicamente como dato decorativo; el servidor MUST usar siempre el del token (FR-UX-006). |

#### F5.4 Requerimientos no funcionales

| ID | Requisito |
|---|---|
| NFR-REL-002 | La pérdida efectiva de telemetría MUST ser < 0.5% (NFR-REL-001 compartido). |
| NFR-PRIV-001 | La telemetría MUST tener una política de retención configurable (por defecto 180 días) y un proceso de borrado automatizado. |
| NFR-PRIV-002 | El acceso a telemetría en bruto MUST limitarse a `admin`; el docente accede solo a agregados. |

#### F5.5 Criterios de aceptación

| ID | Criterio |
|---|---|
| AC-5.1 | 100 % de los eventos enviados con `client_seq` quedan persistidos exactamente una vez. |
| AC-5.2 | Un evento con `event_type` no permitido se rechaza con 422 y no afecta a la sesión del estudiante. |
| AC-5.3 | Cerrar la pestaña con 3 eventos pendientes produce 3 eventos en el servidor (vía `sendBeacon`). |
| AC-5.4 | El panel de métricas muestra la tasa de abandono por concepto y es consistente con los datos brutos (±1 %). |

---

### F6 — Panel docente y administración del banco de ítems

**ID funcional:** `F6` · **Prioridad global:** P2/P3 · **Riesgo:** Bajo

#### F6.1 Descripción

El docente ve el dominio real de sus estudiantes por concepto, revisa reactivos generados y gestiona el
banco. El administrador gestiona el grafo, calibra parámetros y opera el sistema.

#### F6.2 Historias de usuario

| Historia | Prioridad | Testeable de forma independiente |
|---|---|---|
| **US-6.1** El docente ve qué conceptos domina cada estudiante de su curso | P2 | Sí: vista de curso |
| **US-6.2** El docente revisa y aprueba reactivos generados | P2 | Sí: flujo de revisión con `validation_report` |
| **US-6.3** El admin calibra los parámetros BKT de un concepto | P2 | Sí: efecto observable en inferencias posteriores |
| **US-6.4** El docente detecta conceptos con alto abandono | P3 | Sí: agregado sobre métricas |
| **US-6.5** El admin ve la salud del sistema (errores, descartes, coste) | P3 | Sí: panel con métricas |

#### F6.3 Requerimientos funcionales

| ID | Requisito |
|---|---|
| FR-DOC-001 | El panel MUST mostrar, por estudiante y por micronodo: mastery (valor y etiqueta), nº de intentos, aciertos, última actividad y estado. |
| FR-DOC-002 | El panel MUST mostrar, por curso: distribución de dominio por macronodo, media del curso y listado de estudiantes con alertas (bajo dominio, inactividad). |
| FR-DOC-003 | El docente MUST poder filtrar y exportar el dominio del curso (CSV) respetando la autorización por curso. |
| FR-DOC-004 | El panel de revisión de ítems MUST mostrar el enunciado, las opciones, el `validation_report`, los checks superados/fallidos, el modelo que lo generó, su coste y los motivos de rechazo de los descartados. |
| FR-DOC-005 | El docente/admin MUST poder aprobar (`validated` → `published`), editar, rechazar (`rejected`) o archivar (`archived`) un ítem. La edición del enunciado o de la solución MUST exigir una nueva validación simbólica antes de publicar. |
| FR-DOC-006 | El admin MUST poder editar los parámetros BKT por micronodo o en lote, viendo el efecto simulado sobre datos históricos antes de aplicar (simulación no destructiva). |
| FR-DOC-007 | El admin MUST poder generar ítems en lote por micronodo (pre-generación), viendo el coste estimado antes de ejecutar y el resultado por ítem al terminar. |
| FR-DOC-008 | El admin MUST poder disparar el recálculo masivo de dominio (FR-INT-010) y monitorizar su progreso. |
| FR-DOC-009 | El admin MUST poder consultar la cola de mensajes fallidos (DLQ) y reprocesar o descartar cada mensaje, con auditoría. |
| FR-DOC-010 | El panel MUST mostrar el coste de generación acumulado por curso y por periodo. |
| FR-DOC-011 | Toda acción de escritura del panel MUST quedar auditada (actor, fecha, objeto, cambios). |

#### F6.4 Requerimientos no funcionales

| ID | Requisito |
|---|---|
| NFR-PERF-009 | La vista de curso con 500 estudiantes MUST renderizar en < 2 s (con paginación o agregación precalculada). |
| NFR-SEC-006 | La autorización MUST verificarse **en servidor** en cada endpoint del panel; ocultar botones en la UI no es una medida de seguridad. |

#### F6.5 Criterios de aceptación

| ID | Criterio |
|---|---|
| AC-6.1 | Un docente autenticado solo ve los cursos que tiene asignados (verificado también por intento directo a la API de otro curso → 403). |
| AC-6.2 | Editar la solución de un ítem publicado y publicarlo sin revalidar MUST ser imposible (el sistema lo bloquea). |
| AC-6.3 | La simulación de calibración muestra la distribución de mastery antes/después sin escribir en base de datos. |
| AC-6.4 | La exportación CSV contiene una fila por estudiante y micronodo con los campos de FR-DOC-001. |

---

### F7 — Confianza, asincronía, despliegue y operación

**ID funcional:** `F7` · **Prioridad global:** transversal · **Riesgo:** Alto (operación)

#### F7.1 Descripción

Capacidades transversales que hacen el sistema mantenible y operable: autenticación y autorización,
protección de datos de menores, comunicación asíncrona versionada, observabilidad, despliegue y CI/CD.

#### F7.2 Requerimientos funcionales

| ID | Requisito |
|---|---|
| FR-AUTH-001 | El sistema MUST autenticar estudiantes, docentes y administradores mediante JWT/OIDC y MUST validar firma, emisor, audiencia y expiración. |
| FR-AUTH-002 | La identidad MUST derivarse del token en todos los endpoints; los campos de identidad del cuerpo de la petición MUST ignorarse o rechazarse. |
| FR-AUTH-003 | El sistema MUST aplicar autorización por rol y por recurso (curso) en servidor para todas las operaciones de lectura y escritura. |
| FR-AUTH-004 | El sistema MUST requerir autenticación de servicio (mTLS o credencial de servicio) para los endpoints internos de generación y para publicar/consumir eventos. |
| FR-AUTH-005 | El sistema MUST soportar revocación de sesión (logout) y expiración por inactividad. |
| FR-PRIV-001 | El sistema MUST registrar el consentimiento (padre/tutor o maior de edad según normativa aplicable) antes de almacenar datos de menores, y MUST bloquear el uso sin consentimiento registrado. |
| FR-PRIV-002 | El sistema MUST aplicar minimización de datos: solo se almacenan los datos necesarios para la adaptación y la evaluación. |
| FR-PRIV-003 | El sistema MUST ofrecer exportación y borrado de los datos del estudiante (derecho de acceso y supresión) con ejecución efectiva en < 30 días y evidencia auditable. |
| FR-PRIV-004 | El sistema MUST definir y automatizar la retención por tipo de dato (interacciones, telemetría, ítems) y su borrado. |
| FR-PRIV-005 | El sistema MUST registrar los accesos a datos de estudiantes en un log de auditoría de solo lectura. |
| FR-EVT-001 | Los eventos MUST tener nombre versionado (`{Entity}.{Event}.v{n}`), esquema JSON versionado y campo `correlationId` obligatorio. |
| FR-EVT-002 | Los consumidores MUST ser idempotentes, usando `inbox_messages` para deduplicar por `(consumer, message_id)`. |
| FR-EVT-003 | La publicación de eventos MUST usar el patrón **Outbox**: el evento se escribe en `outbox_events` en la misma transacción que el cambio de estado, y un publicador lo entrega al broker con reintento y marca `published_at`. |
| FR-EVT-004 | El sistema MUST definir una **DLQ** por cola, con reintegro manual o automático tras N fallos, y MUST alertar cuando hay mensajes en la DLQ. |
| FR-EVT-005 | El trabajo con duración esperada > 1 s MUST ejecutarse de forma asíncrona (cola + streaming SSE); MUST NOT bloquear el event loop ni el hilo de la petición HTTP. |
| FR-EVT-006 | La entrega de reactivos generados MUST poder consumirse en streaming (SSE) con eventos de progreso y estado final, y MUST ser recuperable por el cliente tras desconexión (reconexión por `Last-Event-ID` o consulta de estado). |
| FR-OBS-001 | Todos los servicios MUST instrumentar trazas OpenTelemetry, propagando `traceId` a través de la cola (headers del mensaje). |
| FR-OBS-002 | El sistema MUST exponer métricas: p95 de entrega del primer ítem, tasa de reactivos descartados, tasa de error por proveedor, profundidad de cola, latencia de inferencia, coste acumulado. |
| FR-OBS-003 | El sistema MUST definir alertas sobre: DLQ no vacía, tasa de error del proveedor por encima del umbral, p95 de entrega por encima del SLO, presupuesto de coste por encima del 80 % y particiones sin rotar. |
| FR-OBS-004 | El sistema MUST tener logs estructurados con `correlationId`, `traceId`, `student_id` (pseudonimizado) y nivel de log. |
| FR-OBS-005 | Los secretos MUST gestionarse fuera del repositorio (Kubernetes `Secret` + gestor externo) y MUST rotarse sin redeploy manual del código. |
| FR-DEP-001 | Cada servicio MUST tener su Dockerfile multi-stage, no-root, con imagen mínima y etiquetas (versión, commit, fecha). |
| FR-DEP-002 | Los manifiestos de Kubernetes MUST incluir `Deployment`, `Service`, `Secret`, `NetworkPolicy`, `PodDisruptionBudget`, `Ingress`, `ConfigMap`, `ServiceAccount` y `HorizontalPodAutoscaler`. |
| FR-DEP-003 | Los `Deployment` MUST definir `livenessProbe`, `readinessProbe` y `startupProbe` coherentes con el tiempo de arranque real de cada servicio. |
| FR-DEP-004 | El autoescalado MUST basarse en la métrica que refleje la carga real: profundidad de cola (KEDA sobre RabbitMQ) para el trabajo dominado por espera de red; CPU como métrica secundaria. |
| FR-DEP-005 | El entorno de desarrollo local MUST proveerse con `docker compose` (PostgreSQL, RabbitMQ, ambos servicios, proveedor LLM local, migraciones y datos semilla). |
| FR-CICD-001 | La CI MUST ejecutar: lint, formato, análisis estático, pruebas unitarias, pruebas de integración (Testcontainers), pruebas de contrato y verificación de migración de esquema. |
| FR-CICD-002 | La CD MUST construir y publicar imágenes por commit, desplegar en entorno de integración y requerir aprobación explícita para producción. |
| FR-CICD-003 | La CI MUST bloquear el merge si falla la suite de pruebas del núcleo BKT o del validador simbólico (puerta de calidad de la Constitución). |
| FR-DEP-006 | El escalado por CPU MUST tener en cuenta la relación `requests`/`limits`: la carga útil MUST dimensionarse para la latencia de proveedor real, no solo para CPU. |

#### F7.3 Requerimientos no funcionales

| ID | Requisito |
|---|---|
| NFR-REL-003 | El servicio principal MUST alcanzar una disponibilidad del 99,5 % mensual (SLO). |
| NFR-SEC-007 | Ningún servicio MUST ejecutarse como `root`; MUST usar `runAsNonRoot` y `readOnlyRootFilesystem`. |
| NFR-SEC-008 | Los datos en tránsito MUST usar TLS y los datos en reposo MUST estar cifrados (PostgreSQL: cifrado de disco gestionado por el proveedor). |
| NFR-REL-004 | El broker MUST garantizar entrega de eventos al menos una vez; la deduplicación es responsabilidad del consumidor (FR-EVT-002). |
| NFR-IA-001 | Los tiempos de inactividad MUST ser superiores a la ventana de mantenimiento sin pérdida de datos (actualizaciones con estrategia `RollingUpdate` y `maxUnavailable = 0`). |

#### F7.4 Criterios de aceptación

| ID | Criterio |
|---|---|
| AC-7.1 | Un token expirado o con firma inválida recibe 401 en todos los endpoints (verificado con suite de contrato). |
| AC-7.2 | El `traceId` de una petición del cliente aparece en los logs del core, en el mensaje publicado y en los logs del servicio de generación. |
| AC-7.3 | Un evento fallido 3 veces acaba en la DLQ y genera alerta. |
| AC-7.4 | Con la cola con 500 mensajes pendientes, el autoscaler aumenta réplicas (verificado con prueba de escalado). |
| AC-7.5 | `kubectl apply` de los manifiestos deja los pods en `Ready` con las probes configuradas. |
| AC-7.6 | La CI falla si el test del trigger BKT o el test property-based del validador fallan. |
| AC-7.7 | Una desconexión del cliente durante el streaming del ítem NO pierde la generación: al reconectar, el cliente recupera el estado. |
| AC-7.8 | El consentimiento no registrado bloquea la creación de sesión de práctica. |

---

## 7. Contratos (resumen normativo)

> Los esquemas completos (OpenAPI y AsyncAPI/JSON Schema) son entregables de la Fase 1/5 y se versionan en
> `specs/<feature>/contracts/`. Aquí se fija la **superficie** y las reglas de contrato.

### 7.1 API del core (Spring Boot) — prefijo `/api/v1`

| Método | Ruta | Rol | Idempotencia | Descripción |
|---|---|---|---|---|
| `POST` | `/interactions` | student | `Idempotency-Key` + `client_seq` | Registrar interacción y actualizar dominio |
| `POST` | `/sessions` | student | `Idempotency-Key` | Crear sesión de práctica |
| `POST` | `/sessions/{id}/close` | student | — | Cerrar sesión y obtener resumen |
| `GET` | `/navigation/next` | student | — | Siguiente ítem recomendado (+ razonamiento) |
| `GET` | `/me/mastery` | student | — | Dominio del estudiante (micronodos y macronodos) |
| `GET` | `/me/progress` | student | — | Resumen de progreso de sesión/curso |
| `GET` | `/items/{id}` | student | — | Detalle de ítem (sin la solución, salvo ítems ya respondidos) |
| `POST` | `/telemetry/events` | student | `client_seq` | Ingesta de telemetría (lote) |
| `GET` | `/courses/{id}/mastery` | teacher/admin | — | Dominio agregado del curso |
| `GET` | `/courses/{id}/mastery/export` | teacher/admin | — | Exportación CSV |
| `GET` | `/items?status=&micronodeId=` | teacher/admin | — | Listado/revisión de ítems |
| `PATCH` | `/items/{id}` | teacher/admin | — | Editar/aprobar/rechazar/archivar ítem |
| `POST` | `/items/validate` | teacher/admin | — | Revalidar simbólicamente un ítem editado |
| `POST` | `/graph/micronodes` | admin | — | Crear micronodo |
| `POST` | `/graph/macronodes` | admin | — | Crear macronodo con contexto situado |
| `POST` | `/graph/edges` | admin | — | Crear arista (rechaza ciclos) |
| `GET` | `/bkt-params/{micronodeId}` | admin | — | Leer parámetros BKT |
| `PUT` | `/bkt-params/{micronodeId}` | admin | — | Actualizar parámetros BKT (auditado) |
| `POST` | `/bkt-params/simulate` | admin | — | Simulación no destructiva de calibración |
| `POST` | `/admin/recompute-mastery` | admin | `Idempotency-Key` | Recálculo masivo de dominio |
| `GET` | `/admin/queues` | admin | — | Estado de colas y DLQ |
| `POST` | `/admin/queues/{queue}/redrive` | admin | — | Reprocesar mensajes de la DLQ |
| `GET` | `/admin/cost` | admin | — | Coste de generación por curso/periodo |

### 7.2 API del servicio de generación (FastAPI) — interna

| Método | Ruta | Descripción |
|---|---|---|
| `POST` | `/v1/generate` | Solicitar generación de ítem para un micronodo (body: `micronode_id`, `difficulty`, `session_id`) → devuelve `job_id` |
| `GET` | `/v1/jobs/{job_id}` | Estado del trabajo (`queued`, `running`, `succeeded`, `failed`, `fallback_used`) |
| `GET` | `/v1/jobs/{job_id}/stream` | SSE con progreso y resultado |
| `POST` | `/v1/validate` | Validar simbólicamente un ítem (usado por el panel docente tras editar) |
| `GET` | `/v1/providers` | Proveedores configurados y su estado (sin exponer secretos) |

### 7.3 Eventos (AsyncAPI / JSON Schema)

| Evento | Productor | Consumidores | Payload mínimo |
|---|---|---|---|
| `InteractionLogged.v1` | Core | Analítica, auditoría | `interaction_id`, `student_id`, `micronode_id`, `item_id`, `correct`, `mastery_delta`, `server_ts` |
| `MasteryUpdated.v1` | Core | Notificaciones, panel | `student_id`, `micronode_id`, `p_known`, `macronode_mastery`, `updated_at` |
| `QuestionGenerationRequested.v1` | Core | Generación | `job_id`, `micronode_id`, `macronode_id`, `situated_context`, `difficulty`, `correlation_id` |
| `QuestionGenerated.v1` | Generación | Core, analítica | `job_id`, `item_id`, `validation_report`, `generation_cost`, `correlation_id` |
| `QuestionRejected.v1` | Generación | Core, analítica | `job_id`, `reason_code`, `validation_report`, `attempts` |

Reglas: `correlationId` obligatorio en todos; `event_id` único; versión de esquema en el nombre del
evento; el `traceparent` se propaga en los headers del mensaje.

### 7.4 Esquema del reactivo (JSON Schema, extracto normativo)

```json
{
  "type": "object",
  "required": ["statement", "options", "correct_option_id", "symbolic_spec", "hint", "explanation", "difficulty"],
  "additionalProperties": false,
  "properties": {
    "statement": { "type": "string", "minLength": 10, "maxLength": 1200 },
    "options": {
      "type": "array", "minItems": 3, "maxItems": 6,
      "items": {
        "type": "object",
        "required": ["id", "text", "is_correct"],
        "properties": {
          "id": { "type": "string" },
          "text": { "type": "string", "minLength": 1, "maxLength": 400 },
          "is_correct": { "type": "boolean" },
          "rationale": { "type": "string", "maxLength": 600 }
        }
      }
    },
    "correct_option_id": { "type": "string" },
    "symbolic_spec": {
      "type": "object",
      "required": ["equation", "variables", "expected_solutions"],
      "properties": {
        "equation": { "type": "string", "maxLength": 500 },
        "variables": { "type": "array", "items": { "type": "string" }, "minItems": 1, "maxItems": 4 },
        "domain": { "type": "string", "enum": ["real", "complex", "natural", "integer"] },
        "expected_solutions": { "type": "array", "items": { "type": "string" }, "minItems": 1 }
      }
    },
    "hint": { "type": "string", "maxLength": 400 },
    "explanation": { "type": "string", "maxLength": 1200 },
    "difficulty": { "type": "integer", "minimum": 1, "maximum": 5 }
  }
}
```

---

## 8. Requerimientos no funcionales transversales

| ID | Categoría | Requisito |
|---|---|---|
| NFR-ARCH-001 | Arquitectura | La lógica de dominio, la infraestructura y la capa HTTP residirán en módulos separados (Clean Architecture / SOLID). Un archivo que las mezcle no se acepta (Constitución). |
| NFR-ARCH-002 | Arquitectura | El MVP usa **un único broker**; incorporar gRPC u otro protocolo sin métrica que lo justifique no se permite. |
| NFR-ARCH-003 | Arquitectura | La complejidad añadida se justifica con una métrica real, documentada en el `plan.md` de la feature. |
| NFR-PERF-010 | Rendimiento | SLO-001: p95 de entrega del primer ítem < 3 s (cacheado) y < 15 s (generado en línea). |
| NFR-PERF-011 | Rendimiento | SLO-002: p95 de registro de interacción < 300 ms; p95 de navegación < 150 ms. |
| NFR-QUAL-002 | Calidad | SLO-003: tasa de reactivos descartados < 15 % (banco pregenerado). |
| NFR-QUAL-003 | Calidad | SLO-004: 0 reactivos servidos sin validación simbólica aprobada (auditable). |
| NFR-QUAL-004 | Calidad | SLO-005: p95 de pérdida de telemetría < 0,5 %. |
| NFR-REL-005 | Fiabilidad | SLO-006: disponibilidad ≥ 99,5 % del core en horario laboral escolar. |
| NFR-COST-002 | Coste | SLO-007: coste por ítem generado y por estudiante-mes monitorizados, con presupuesto por curso y alerta al 80 %. |
| NFR-TEST-001 | Pruebas | El trigger BKT se prueba con pgTAP o Testcontainers, incluyendo casos de borde numéricos (p=0, p=1, p_S mínimo). |
| NFR-TEST-002 | Pruebas | El validador simbólico se prueba con *property-based tests* (Hypothesis) sobre un corpus de ecuaciones y respuestas esperadas. |
| NFR-TEST-003 | Pruebas | Los contratos entre servicios se prueban con *contract tests* (OpenAPI/AsyncAPI) en CI. |
| NFR-TEST-004 | Pruebas | Cobertura mínima del núcleo BKT y del validador: ≥ 90 % de líneas en esos módulos. |
| NFR-TEST-005 | Pruebas | Los flujos críticos del frontend tienen pruebas e2e (Playwright) y unitarias (Karma/Jest según stack). |
| NFR-OBS-002 | Observabilidad | Trazas OpenTelemetry con propagación de `traceId` por la cola; logs estructurados; métricas y alertas definidos en FR-OBS-002/003. |
| NFR-I18N-001 | Producto | La interfaz estará en español (es-ES) en el MVP; la architectura no impide añadir idiomas. |
| NFR-BROWSER-001 | Producto | Soporte de los 2 navegadores principales y 2 versiones anteriores; móvil responsive. |

---

## 9. SLOs y criterios de aceptación globales

| ID | SLO | Medición | Objetivo |
|---|---|---|---|
| SLO-001 | Entrega del primer ítem | p95 desde el inicio de sesión hasta la primera pregunta | < 3 s cacheado / < 15 s generado |
| SLO-002 | Latencia de dominio | p95 de `POST /interactions` | < 300 ms |
| SLO-003 | Tasa de reactivos descartados | descartados / generados | < 15 % (pregenerado) |
| SLO-004 | Integridad matemática | ítems servidos sin validación | 0 |
| SLO-005 | Pérdida de telemetría | (enviados − persistidos) / enviados | < 0,5 % |
| SLO-006 | Disponibilidad del core | uptime mensual en horario escolar | ≥ 99,5 % |
| SLO-007 | Coste | coste/ítem y coste/estudiante-mes | dentro del presupuesto; alerta al 80 % |
| SLO-008 | Profundidad de cola | mensajes pendientes | p95 < 100 mensajes |

---

## 10. Orden de implementación

### 10.1 Principios de ordenación

1. **Núcleo antes que comprehensibilidad**: la verdad del dominio (F1) va primero, porque todo lo demás la
   consume.
2. **Riesgo máximo antes que funcionalidad máxima**: la generación y validación (F3) es la parte con más
   incertidumbre técnica; se aísla pronto, pero **después** de tener el ciclo end-to-end mínimo.
3. **Vertical antes que horizontal**: cada fase entrega un recorrido completo usable (aunque con
   simplificaciones), no una capa completa sin consumidor.
4. **Cada fase es demostrable y verificable**: sin la puerta de salida de la fase anterior no se avanza.
5. **La complejidad se pospone**: asincronía, streaming, autoescalado y panel docente llegan cuando la
   métrica los justifica.

### 10.2 Fases

| Fase | Nombre | Entregable verificable | Funcionalidades | Requisitos cubiertos (extracto) | Depende de | Puerta de salida |
|---|---|---|---|---|---|---|
| **F0** | Cimientos del repositorio | Monorepo con 3 módulos (`core/`, `generation/`, `web/`), `docker compose` local, CI con lint+tests+build, migraciones versionadas | transversal | NFR-ARCH-001, NFR-TEST-001 (base), FR-DEP-005, FR-CICD-001 | — | `docker compose up` levanta PostgreSQL + RabbitMQ + ambos servicios; CI en verde en el primer commit |
| **F1** | Modelo de datos + BKT en PostgreSQL | DDL completo (8+ tablas), función BKT parametrizada por micronodo, `CREATE TRIGGER` explícito, particionamiento de `interaction_logs`, suite de tests del trigger (pgTAP/Testcontainers) | F1 (parcial) | FR-INT-002/003/004, FR-BKT-001..009, NFR-DATA-001, NFR-PERF-002 | F0 | Test del trigger en verde: casos p=0, p=1, p_S→epsilon, idempotencia, DAG no afectado; partición del mes actual creada automáticamente |
| **F2** | API de dominio e interacciones | API core: `POST /interactions`, `GET /me/mastery`, `GET /navigation/next` (con ítems semilla), auditoría de cambios | F1, F2 | FR-INT-001..010, FR-NAV-001..008, FR-AUD-001, NFR-PERF-001/003 | F1 | Flujo completo "registrar interacción → mastery actualizado → navegación cambia" verificado con test de integración; permisos por rol probados |
| **F3** | Sesión de práctica en el cliente | Front Angular: sesión, render del ítem, feedback, resumen de sesión; cola local + `sendBeacon` | F4, F5 | FR-UX-001..016, FR-TEL-001..010, NFR-PERF-006/007/008, NFR-REL-001, NFR-A11Y-001 | F2 | e2e verde: el estudiante completa una sesión de 5 ítems; al cerrar la pestaña con eventos pendientes, no se pierden eventos; teclado accesible |
| **F4** | Generación y validación de reactivos | Servicio FastAPI: `LLMProvider`, structured output, validador SimPy seguro (sandbox + timeout + allowlist), reflexión con reinyección de errores, `validation_report`, pre-generación por lotes | F3 (parcial), F6 (revisión) | FR-GEN-001..025, NFR-SEC-002/004, NFR-QUAL-001, NFR-TEST-002 | F2, F3 | Un ítem generado por LLM llega a `validated` y se sirve; los tests de payload malicioso, timeout, variable distinta de `x` y soluciones múltiples pasan |
| **F5** | Asincronía, outbox, streaming y pre-generación | Outbox transaccional + publicador, consumidores idempotentes, DLQ, SSE para generación, modo asíncrono con `fallback` al banco | F3, F7 | FR-EVT-001..006, FR-NAV-007, FR-GEN-020, FR-OBS-001/002/003, NFR-REL-004 | F4 | Un `QuestionGenerationRequested` se procesa de extremo a extremo por la cola; el `traceId` se propaga; un fallo 3 veces va a la DLQ y alerta |
| **F6** | Panel docente y operación del banco | Panel de dominio por curso, revisión de ítems con `validation_report`, calibración BKT con simulación, recálculo masivo, gestión de DLQ y coste | F6 | FR-DOC-001..011, FR-BKT-004/009, FR-INT-010, NFR-SEC-006 | F2, F4, F5 | Un docente ve el dominio de su curso y exporta CSV; un admin calibra parámetros y ve el efecto simulado; la autorización por curso está probada |
| **F7** | Despliegue, escalado y observabilidad | Dockerfiles multi-stage, manifiestos K8s completos, HPA/KEDA por profundidad de cola, OpenTelemetry + dashboards + alertas, CI/CD con entornos y secretos | F7 | FR-DEP-001..006, FR-OBS-001..005, FR-CICD-001..003, FR-AUTH-001..005, FR-PRIV-001..005, NFR-REL-003, NFR-SEC-007/008 | F5, F6 | Manifiestos aplicados → pods `Ready`; el autoscaler escala con 500 mensajes en cola; la suite completa en CI bloquea el merge si falla el núcleo; auditoría de acceso a datos activa |

### 10.3 Grafo de dependencias entre fases

```
F0 ──▶ F1 ──▶ F2 ──▶ F3 ──▶ F4 ──▶ F5 ──▶ F7
              │              │              │
              └──────────────┴──────────────┴──▶ F6 ──▶ F7
```

- Ruta crítica: **F0 → F1 → F2 → F3 → F4 → F5 → F7**.
- F6 (panel) puede arrancar en paralelo con F4/F5 a partir de los endpoints ya disponibles de F2.
- F7 (despliegue) puede adelantarse en su parte de **manifiestos y CI** (F0) sin esperar al producto.

### 10.4 Dentro de cada fase: orden interno de tareas

Cada fase se descompone en los siguientes bloques, en este orden:

1. **Contrato primero**: esquema de datos / endpoint / evento definido y versionado **antes** del código.
2. **Prueba primero** para el núcleo (BKT, validador): test rojo → implementación → verde.
3. **Implementación**: modelos → servicios → endpoints (nunca al revés).
4. **Observabilidad**: logs, métricas y trazas del bloque desde el primer día (no al final).
5. **Documentación**: OpenAPI/AsyncAPI y `README` del módulo actualizados en el mismo commit.
6. **Checkpoint**: la fase se cierra con la puerta de salida verificada y el suite en verde.

### 10.5 Reglas de Definition of Done (aplicables a todas las fases)

- [ ] Test(s) nuevos en verde; suite completa en verde.
- [ ] Cobertura del núcleo ≥ 90 % (NFR-TEST-004).
- [ ] Sin desviaciones de la Constitución sin justificación documentada en `Complexity Tracking`.
- [ ] Contratos (OpenAPI/AsyncAPI) actualizados y versionados.
- [ ] Migraciones de esquema aplicadas y reversibles en entorno de integración.
- [ ] Trazas, logs y métricas presentes para el flujo nuevo.
- [ ] Sin secretos ni datos de estudiante en el repositorio ni en los logs.
- [ ] La puerta de salida de la fase está verificada con evidencia.

---

## 11. Matriz de trazabilidad (resumen)

| Funcionalidad | Requisitos | Fase principal | Verificación principal |
|---|---|---|---|
| F1 Dominio (BKT) | FR-INT-001..010, FR-BKT-001..009, FR-AUD-001 | F1 → F2 | Test del trigger (pgTAP/Testcontainers) + test de integración |
| F2 Grafo y navegación | FR-NAV-001..010, FR-GRAPH-001..003 | F2 | Test de la política de selección (determinismo) + contrato de API |
| F3 Generación y validación | FR-GEN-001..025 | F4 → F5 | Property-based tests del validador + tests de seguridad + stub de LLM |
| F4 Sesión de práctica | FR-UX-001..016 | F3 | e2e + a11y + test de `sendBeacon` |
| F5 Telemetría | FR-TEL-001..010 | F3 | e2e de entrega + reconciliación de conteos |
| F6 Panel docente | FR-DOC-001..011 | F6 | Test de autorización por curso + test de flujo de revisión |
| F7 Confianza y operación | FR-AUTH-*, FR-PRIV-*, FR-EVT-*, FR-OBS-*, FR-DEP-*, FR-CICD-* | F0 → F5 → F7 | Contract tests + pruebas de despliegue + CI |

Cobertura: **125 requisitos funcionales** (`FR-*`), **47 requisitos no funcionales** (`NFR-*`),
**48 criterios de aceptación** (`AC-*`), **8 SLOs**, **7 reglas de grafo** (`GRA-*`), **10 decisiones
pendientes** (`P-*`) y **12 riesgos** (`RSK-*`). No hay huecos en la numeración: todos los identificadores
son correlativos y estables, pensados para ser referenciados desde los `spec.md` de cada feature.

---

## 12. Dependencias y supuestos

### 12.1 Dependencias externas

| ID | Dependencia | Uso | Riesgo |
|---|---|---|---|
| DEP-01 | PostgreSQL 16+ | Fuente de verdad del dominio | Alta (fuente única) |
| DEP-02 | RabbitMQ | Bus de eventos | Media |
| DEP-03 | Proveedor LLM (Gemini/Ollama) | Generación de reactivos | Media (coste, cuota, latencia) |
| DEP-04 | IdP con OIDC/JWT | Autenticación | Media |
| DEP-05 | Clúster Kubernetes | Despliegue | Media |
| DEP-06 | Registro de imágenes + CI/CD | Entrega | Baja |

### 12.2 Supuestos del proyecto

- El banco de ítems inicial se crea **manualmente** (semilla) para habilitar las fases F2 y F3 sin depender
  del LLM.
- El cálculo de dominio no necesita datos de otros sistemas (no hay SIS/LMS integraciones en el MVP).
- Un curso tiene un único profesor responsable; la co-docencia se resuelve con varios `teacher_id` en
  `courses` y autorización "any-of".
- La evaluación del curso (notas) es responsabilidad del LMS institucional; el sistema expone dominio y
  exportación, no califica.

---

## 13. Riesgos

| ID | Riesgo | Prob. | Impacto | Mitigación | Riesgo residual |
|---|---|---|---|---|---|
| RSK-01 | El LLM genera ítems con soluciones incorrectas | Alta | Crítico | Validación simbólica obligatoria + banco pregenerado + revisión docente (F3) | Bajo |
| RSK-02 | RCE vía payload del LLM | Media | Crítico | Parseo allowlist + sandbox + timeout + tests de seguridad (FR-GEN-005/006) | Bajo |
| RSK-03 | El BKT mal calibrado produce saltos al vacío | Media | Alto | Parámetros por micronodo + simulación de calibración + métricas de progresión (FR-DOC-006) | Medio |
| RSK-04 | Coste y latencia del LLM en el camino crítico | Media | Medio | Pre-generación por lotes + caché + modo asíncrono + límites de cuota (FR-GEN-015/019) | Bajo |
| RSK-05 | Pérdida de telemetría por cierre de pestaña | Alta (sin corregir) | Medio | `sendBeacon` + cola local persistente (FR-TEL-003) | Bajo |
| RSK-06 | Divergencia entre dominio calculado en cliente y servidor | Media | Alto | Fuente única de verdad + derivación del `micronode_id` en servidor (FR-INT-003/005) | Bajo |
| RSK-07 | Cuello de botella en el core por trabajo de larga duración | Media | Medio | Cola + SSE + escalado por profundidad de cola (FR-EVT-005, FR-DEP-004) | Bajo |
| RSK-08 | Filtración de datos de menores o acceso indebido | Baja | Crítico | RBAC por curso + minimización + auditoría + consentimiento (F7) | Medio |
| RSK-09 | Los datos de `interaction_logs` crecen sin control | Media | Medio | Particionado mensual + retención + archivado (NFR-DATA-001) | Bajo |
| RSK-10 | Sobreingeniería prematura (gRPC, multi-broker) | Media | Medio | Restricción explícita en la Constitución y gate de justificación (NFR-ARCH-002/003) | Bajo |
| RSK-11 | La especificación original de los "5 pilares" no está disponible | Alta | Medio | Se adopta el conjunto de supuestos §15; pendiente de validación con el responsable | Medio |
| RSK-12 | Deterioro del dominio por decaimiento mal calculado | Media | Medio | Decaimiento en lectura, aislado y testeado; parámetro configurable (GRA-04) | Bajo |

---

## 14. Fuera de alcance (declarado)

1. Integraciones con LMS/SIS y proveedor institucional de identidad específico (solo OIDC genérico).
2. Modelos de dominio alternativos (PFA, DKT, bandits).
3. Calibración automática de BKT por machine learning.
4. Generación de contenido multimodal.
5. Colaboración, mensajería y contenido social entre estudiantes.
6. Personalización por estilos de aprendizaje con evidencia científica (fuera del MVP y sin respaldo en
   el material disponible).
7. Exportación a estándares externos de preguntas (Questionmark, Moodle XML) más allá de CSV.
8. Aplicación móvil nativa (la web responsive es el MVP).

---

## 15. Decisiones pendientes de validación (supuestos adoptados)

Estas lagunas vienen del material heredado (falta la especificación original de los "5 pilares"). Cada una
tiene un valor por defecto **adoptado en este PRD** para no bloquear la implementación, y un disparador
claro para revisarla.

| ID | Pregunta | Valor por defecto adoptado | Impacto si cambia | Disparador |
|---|---|---|---|---|
| P-01 | ¿Cuál es la granularidad del grafo y cómo se define un micronodo? | Micronodo = concepto atómico evaluable; un macronodo agrupa 4–12 | Alto: cambia el modelo de datos y el trabajo de poblamiento | Confirmación del responsable del proyecto |
| P-02 | ¿Cómo se propaga el dominio de micronodo a macronodo? | Media ponderada por dificultad (GRA-03) | Medio: afecta a la vista docente y a las alertas | Revisión con el equipo docente |
| P-03 | ¿Umbral de "dominado" y decaimiento? | 0.95 y vida media 30 días (GRA-04/05) | Alto: cambia la política de selección y el ritmo de avance | Piloto con datos reales |
| P-04 | ¿Qué es exactamente un "nodo situado"? | Macronodo con contexto situado `{rol, situacion, herramienta, artefacto}` inyectado en el prompt y visible en la UI (GRA-06) | Alto: define el diferencial pedagógico del producto | Definición del responsable de producto |
| P-05 | ¿Retención de datos de menores? | Interacciones 24 meses, telemetría 180 días, ítems indefinidos | Medio: obligaciones legales y de almacenamiento | Revisión legal |
| P-06 | ¿El estudiante ve el valor numérico de mastery? | No; solo etiqueta de estado (FR-UX-004) | Bajo | Validación UX |
| P-07 | ¿Materia y curso predeterminados para el piloto? | 1 asignatura piloto, 1 curso, ~120 micronodos, 1 docente, ~50 estudiantes | Medio: dimensiona el trabajo de poblamiento | Confirmación del piloto |
| P-08 | ¿Proveedor LLM inicial? | Gemini con structured output + Ollama en local para desarrollo (FR-GEN-001) | Medio: condiciona costes y capacidad de structured output | Evaluación de coste y capacidad |
| P-09 | ¿IdP? | OIDC genérico con roles en el token, emisor y audiencia configurables | Medio: condiciona el login | Alineación con el equipo de IT |
| P-10 | ¿Volumen objetivo de estudiantes concurrentes? | 500 concurrentes, 5 000 estudiantes totales | Alto: condiciona escalado, particionado y coste | Confirmación de escalabilidad |

> **Nota Spec Kit:** al derivar cada feature con `/speckit.specify`, los requisitos afectados por estas
> decisiones deben mantener la etiqueta `[NEEDS CLARIFICATION]` solo si la decisión afecta al alcance de
> esa feature; en caso contrario, se aplica el valor por defecto y se registra como supuesto.

---

## 16. Anexo A — Reglas de derivación a features Spec Kit

Este PRD **no reemplaza** al flujo Spec Kit; lo alimenta. Cada funcionalidad debe convertirse en una
feature con su `spec.md` independiente:

| # | Feature (directorio `specs/NNN-<short-name>`) | Alcance | Depende de |
|---|---|---|---|
| 001 | `001-bkt-mastery-core` | Esquema, trigger BKT, parámetros, recálculo | — |
| 002 | `002-interaction-and-navigation-api` | API de interacciones, dominio, navegación, grafo | 001 |
| 003 | `003-practice-session-frontend` | Sesión de práctica Angular + telemetría | 002 |
| 004 | `004-item-generation-validation` | FastAPI, `LLMProvider`, validador SimPy, reportes | 002 |
| 005 | `005-async-events-and-streaming` | Outbox, broker, consumidores idempotentes, DLQ, SSE | 004 |
| 006 | `006-teacher-admin-console` | Panel docente/admin, calibración, coste, DLQ | 002, 004 |
| 007 | `007-security-observability-deployment` | Auth/RBAC, privacidad, OTel, K8s, CI/CD | 002..006 |

Cada feature debe producir, según el flujo: `spec.md` → `clarify` → `plan.md` (+ `data-model.md`,
`research.md`, `contracts/`) → `tasks.md` → `analyze` → `implement`.

---

## 17. Anexo B — Checklist de calidad del PRD (autovalidación)

- [x] Todos los roles identificados con sus permisos (SRC-01, §3).
- [x] 7 funcionalidades con historias priorizadas, FR, NFR, criterios de aceptación y casos borde (SRC-01, SRC-02).
- [x] Cobertura de las correcciones críticas del documento de revisión: 1.1–1.15 (asincronía, reflexión real, BKT parametrizado, validador seguro, esquema del reactivo, telemetría fiable, K8s, etc.).
- [x] Cobertura de los faltantes de la sección 2 del documento de revisión: modelo de datos, reglas del grafo, contratos, autenticación, outbox, fallback, pre-generación, coste, estructura de repositorios, estrategia de pruebas, CI/CD, observabilidad, SLOs.
- [x] Orden de implementación explícito con entregables, dependencias, puertas de salida y DoD (SRC-01 §Flujo de Desarrollo).
- [x] Trazabilidad requisito → fase → verificación (§11).
- [x] Riesgos con mitigación (faltante señalado en la revisión, §13).
- [x] Decisiones pendientes explícitas con valor por defecto (evita propagar ambigüedad a las tareas, §15).
- [x] Coherencia con la Constitución v1.0.0 en los 5 principios y las restricciones técnicas.

---

**Versión del documento:** 1.0.0 · **Fecha:** 2026-10-19 · **Siguiente revisión:** tras la validación de
P-01..P-10 y la primera derivación a features Spec Kit.
