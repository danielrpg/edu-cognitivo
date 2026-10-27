# Historias de Usuario — Sistema de Aprendizaje Adaptativo Basado en Grafos de Micronodos y Macronodos Situados

| Campo | Valor |
|---|---|
| **Documento** | Catálogo de historias de usuario |
| **Versión** | 1.0.0 |
| **Fecha** | 2026-10-19 |
| **Derivado de** | `PRD.md` v1.0.0 |
| **Constitución aplicable** | `.specify/memory/constitution.md` v1.0.0 |
| **Total de historias** | 56 (14 P1 · 25 P2 · 17 P3) |
| **Uso previsto** | Copiar cada bloque de historias a `specs/<feature>/spec.md` (formato compatible con `.specify/templates/spec-template.md`) |

---

## 1. Convenciones

### 1.1 Formato de cada historia

Cada historia sigue la plantilla de `spec-template.md`, ampliada con el mapeo a requisitos:

```
### US-<F>.<N> — <Título> (Prioridad: P<n>)

**Actor**: <rol>
**Por qué esta prioridad**: <valor de negocio y rationale>
**Prueba independiente**: <cómo se verifica sola>
**Cobre**: FR-xxx, AC-x.x
**Escenarios de aceptación**:
1. **Dado** … **Cuando** … **Entonces** …
```

### 1.2 Criterios de prioridad

| Nivel | Significado |
|---|---|
| **P1** | Sin esta historia el sistema **no cumple su propósito** o **no es seguro**. Bloquea el MVP. |
| **P2** | Valor importante, pero el producto es utilizable y demostrable sin ella. |
| **P3** | Mejora operativa, de experiencia o de analítica. Entra al final del roadmap. |

### 1.3 Reglas de construcción (heredadas de la Constitución)

- Cada historia es **independientemente testeable y demostrable**: si se implementa sola, deja valor.
- Ninguna historia puede exigir que el dominio se calcule fuera de PostgreSQL (Principio I).
- Ninguna historia puede entregar un reactivo sin validación simbólica (Principio II).
- Ninguna historia confía en la identidad enviada por el cliente (Principio III).
- Las historias de trabajo largo (> 1 s) se resuelven de forma asíncrona (Principio IV).
- Toda historia que toque el núcleo BKT o el validador simbólico exige pruebas automatizadas (Principio V).

### 1.4 Índice global

| ID | Título | Actor | P | Fases | Requisitos clave |
|---|---|---|---|---|---|
| US-1.1 | Registrar respuesta y actualizar mi dominio | Estudiante | P1 | F1–F2 | FR-INT-001/002, FR-BKT-001/003 |
| US-1.2 | Ver mi dominio por concepto | Estudiante | P1 | F2 | FR-INT-007, GRA-04/05 |
| US-1.3 | Ver el dominio real de mis estudiantes | Docente | P1 | F2 | FR-AUD-001 |
| US-1.4 | Reintentar una respuesta sin duplicar el conteo | Estudiante | P2 | F2 | FR-INT-004/006 |
| US-1.5 | Auditar de dónde sale cada cambio de dominio | Admin | P2 | F1–F2 | FR-INT-003, FR-BKT-008 |
| US-1.6 | Calibrar el modelo de un concepto | Admin | P2 | F6 | FR-BKT-004/009 |
| US-1.7 | Simular una calibración antes de aplicarla | Admin | P3 | F6 | FR-BKT-009, FR-DOC-006 |
| US-1.8 | Recalcular el dominio de forma masiva | Admin | P3 | F6 | FR-INT-010 |
| US-2.1 | No avanzar sobre conceptos sin prerrequisitos | Estudiante | P1 | F2 | FR-NAV-002/003, GRA-01/02 |
| US-2.2 | Saber por qué me toca este concepto | Estudiante | P1 | F2 | FR-NAV-004/006, GRA-06/07 |
| US-2.3 | Practicar en refuerzo cuando estoy cerca del dominio | Estudiante | P2 | F2 | FR-NAV-005, GRA-05 |
| US-2.4 | Elegir el objetivo de mi sesión | Estudiante | P2 | F2 | FR-NAV-010 |
| US-2.5 | Saber que ya no queda nada por delante | Estudiante | P2 | F2 | FR-NAV-008 |
| US-2.6 | Administrar el grafo de conceptos | Admin | P2 | F2 | FR-GRAPH-001/002/003 |
| US-2.7 | Proponer un prerrequisito a mi curso | Docente | P3 | F6 | FR-GRAPH-001/002 |
| US-2.8 | Ver el mapa de conceptos del curso | Estudiante | P3 | F6 | FR-NAV-001, FR-NAV-006 |
| US-3.1 | Recibir un ejercicio nuevo de mi concepto | Estudiante | P1 | F4 | FR-GEN-003/004/014 |
| US-3.2 | Que ningún ejercicio con error matemático llegue | Estudiante | P1 | F4 | FR-GEN-005/007/008/014 |
| US-3.3 | Que un contenido malicioso no me afecte | Estudiante | P1 | F4 | FR-GEN-005/006/024 |
| US-3.4 | Ver por qué se rechazó un ejercicio generado | Docente | P2 | F4, F6 | FR-GEN-013/021, FR-DOC-004 |
| US-3.5 | Aceptar cualquier respuesta correcta válida | Estudiante | P2 | F4 | FR-GEN-008/009 |
| US-3.6 | Que el generador se corrija solo tras un error | Admin | P2 | F4 | FR-GEN-011/012 |
| US-3.7 | Que el sistema no se caiga si el proveedor falla | Admin | P2 | F4–F5 | FR-GEN-011, FR-GEN-020 |
| US-3.8 | Pre-generar el banco de ejercicios del curso | Admin | P2 | F4 | FR-GEN-019, FR-DOC-007 |
| US-3.9 | Conocer el coste de generar ejercicios | Admin | P3 | F6 | FR-GEN-015/018, FR-DOC-010 |
| US-3.10 | Que no se repitan ejercicios equivalentes | Docente | P3 | F4 | FR-GEN-016/017 |
| US-4.1 | Empezar a practicar y ver mi primer ejercicio | Estudiante | P1 | F3 | FR-UX-001/002, NFR-PERF-006 |
| US-4.2 | Responder y recibir feedback inmediato | Estudiante | P1 | F3 | FR-UX-003, FR-INT-001 |
| US-4.3 | Ver el contexto real en el que aplico el concepto | Estudiante | P1 | F3 | FR-UX-002/003, GRA-06 |
| US-4.4 | Ver en qué estado está mi dominio | Estudiante | P2 | F3 | FR-UX-004 |
| US-4.5 | Pedir una pista sin perder mi progreso | Estudiante | P2 | F3 | FR-UX-003, FR-INT-001 |
| US-4.6 | Saltar un ejercicio que no me sale | Estudiante | P2 | F3 | FR-UX-011 |
| US-4.7 | Practicar aunque se caiga la conexión | Estudiante | P2 | F3 | FR-UX-008/010 |
| US-4.8 | Cerrar la pestaña sin perder lo que hice | Estudiante | P2 | F3 | FR-UX-009, FR-TEL-003 |
| US-4.9 | Responder con el mismo ítem más de una vez | Estudiante | P2 | F3 | FR-UX-005, FR-INT-001 |
| US-4.10 | Usar la plataforma solo con teclado | Estudiante | P3 | F3 | FR-UX-013/014, NFR-A11Y-001 |
| US-4.11 | Ver el resumen de mi sesión | Estudiante | P3 | F3 | FR-UX-016 |
| US-5.1 | Saber qué conceptos se me están resistiendo | Docente | P3 | F6 | FR-TEL-007, FR-DOC-002 |
| US-5.2 | Saber dónde se abandona la práctica | Admin | P3 | F6 | FR-TEL-007 |
| US-5.3 | Que un evento inválido no rompa mi sesión | Estudiante | P3 | F3 | FR-TEL-005 |
| US-6.1 | Ver el dominio de cada estudiante de mi curso | Docente | P1 | F6 | FR-DOC-001/002, FR-AUD-001 |
| US-6.2 | Exportar el dominio de mi curso | Docente | P2 | F6 | FR-DOC-003 |
| US-6.3 | Revisar y aprobar un ejercicio generado | Docente | P2 | F6 | FR-DOC-004/005, FR-GEN-014 |
| US-6.4 | Corregir un ejercicio sin publicar un error | Docente | P2 | F6 | FR-DOC-005, FR-GEN-005 |
| US-6.5 | No ver datos de cursos que no son míos | Docente | P1 | F6 | FR-AUTH-003, NFR-SEC-006 |
| US-6.6 | Operar la cola de mensajes fallidos | Admin | P3 | F7 | FR-DOC-009, FR-EVT-004 |
| US-6.7 | Ver la salud del sistema y su coste | Admin | P3 | F7 | FR-DOC-010, FR-OBS-002 |
| US-7.1 | Entrar al sistema con mi identidad | Estudiante/Docente/Admin | P1 | F7 | FR-AUTH-001/002 |
| US-7.2 | Consentimiento antes de usar el sistema si soy menor | Estudiante | P2 | F7 | FR-PRIV-001 |
| US-7.3 | Pedir la eliminación de mis datos | Estudiante | P3 | F7 | FR-PRIV-003/004 |
| US-7.4 | Que el acceso a datos de estudiantes quede registrado | Admin | P3 | F7 | FR-PRIV-005, FR-OBS-004 |
| US-7.5 | Seguir una petición de extremo a extremo | Operador | P2 | F5–F7 | FR-OBS-001, FR-EVT-001 |
| US-7.6 | Ser avisado cuando algo se degrada | Operador | P3 | F7 | FR-OBS-003, FR-EVT-004 |
| US-7.7 | Que el servicio aguante más carga de la prevista | Operador | P3 | F7 | FR-DEP-004, NFR-REL-003 |
| US-7.8 | Que un cambio con el núcleo roto no llegue a producción | Dev | P2 | F0, F7 | FR-CICD-003, NFR-TEST-004 |
| US-7.9 | Recibir el ejercicio generado sin quedarme esperando en silencio | Estudiante | P2 | F5 | FR-EVT-005/006, FR-NAV-007 |

---

## 2. F1 — Registro de interacciones y estimación del dominio (BKT)

> **Objetivo de la funcionalidad**: que el dominio de cada estudiante se mantenga actualizado, auditable y
> calculado en un único lugar (PostgreSQL).

### US-1.1 — Registrar respuesta y actualizar mi dominio (Prioridad: P1)

**Actor**: Estudiante
**Por qué esta prioridad**: es el corazón del producto. Sin la actualización del dominio, el motor
adaptativo no sabe qué hacer y toda la adaptación es una simulación.
**Prueba independiente**: enviar una interacción por la API y comprobar que `student_mastery` cambia según
la fórmula BKT, sin necesidad de frontend ni de LLM.
**Cobre**: FR-INT-001, FR-INT-002, FR-INT-005, FR-BKT-001, FR-BKT-002, FR-BKT-003, AC-1.1, AC-1.2, AC-1.3

**Escenarios de aceptación**:

1. **Dado** un estudiante sin historial y un ítem publicado de un micronodo,
   **Cuando** responde correctamente,
   **Entonces** se registra una interacción con `student_id` derivado del token, `micronode_id` derivado
   del ítem, `server_ts` del servidor y la mastery del micronodo aumenta.
2. **Dado** un estudiante con `p = 0.9` en un micronodo,
   **When** responde incorrectamente,
   **Entonces** la mastery disminuye y la fila `student_mastery` queda con `n_attempts` incrementado y
   `n_correct` sin cambios.
3. **Dado** un token válido de estudiante A,
   **When** envía un cuerpo con `student_id` de estudiante B,
   **Entonces** la interacción se registra para A (el campo del cuerpo se ignora) y la respuesta no expone
   datos de B.
4. **Dado** un ítem en estado `rejected`,
   **When** se registra una interacción,
   **Entonces** la interacción se guarda pero **no** se computa dominio y la respuesta es `204`.
5. **Dado** un fallo en la inferencia BKT,
   **When** se confirma la transacción,
   **Entonces** ni la interacción ni el mastery quedan persistidos (atomicidad).

---

### US-1.2 — Ver mi dominio por concepto (Prioridad: P1)

**Actor**: Estudiante
**Por qué esta prioridad**: el estudiante necesita ver su avance; es el retorno de valor inmediato y lo
valida con sus propios datos.
**Prueba independiente**: sembrar `student_mastery` y consultar el endpoint de dominio propio.
**Cobre**: FR-INT-007, FR-INT-008, GRA-03, GRA-04, GRA-05, AC-1.6, AC-1.7

**Escenarios de aceptación**:

1. **Dado** un estudiante con interacciones en varios micronodos,
   **When** consulta su dominio,
   **Entonces** recibe, por micronodo, la mastery, el estado (`pendiente` / `en_reinforcement` / `dominado`)
   y el número de intentos.
2. **Dado** un estudiante con `p = 0.95` exacto en un micronodo,
   **When** consulta su dominio,
   **Entonces** ese micronodo aparece como `dominado` (umbral inclusivo).
3. **Dado** un micronodo sin actividad hace 30 días,
   **When** el estudiante consulta su dominio,
   **Entonces** la mastery mostrada aplica el decaimiento (≈ la mitad) y el sistema informa de que es una
   mastery efectiva, no almacenada.
4. **Dado** un macronodo con micronodos de distinta dificultad,
   **When** se consulta el dominio,
   **Entonces** la mastery del macronodo coincide con la media ponderada por dificultad (±1e-6).
5. **Dado** un estudiante autenticado,
   **When** consulta el dominio de otro estudiante,
   **Entonces** la API responde `403` y no filtra información por `404` vs `403`.

---

### US-1.3 — Ver el dominio real de mis estudiantes (Prioridad: P1)

**Actor**: Docente
**Por qué esta prioridad**: es la propuesta de valor para el docente (dejar de mirar notas medias) y
valida que el modelo de dominio sirve para algo más que para el algoritmo.
**Prueba independiente**: endpoint de consulta con datos sembrados y un token de docente.
**Cobre**: FR-AUD-001, NFR-SEC-001, AC-1.5

**Escenarios de aceptación**:

1. **Dado** un docente con cursos asignados,
   **When** consulta el dominio de un estudiante de uno de sus cursos,
   **Entonces** recibe la vista por concepto con mastery, intentos, aciertos y última actividad.
2. **Dado** un docente de otro curso,
   **When** consulta el mismo dominio por API directa,
   **Entonces** la respuesta es `403` y el acceso queda registrado en auditoría.

---

### US-1.4 — Reintentar una respuesta sin duplicar el conteo (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: la telemetría y las redes móviles producen reintentos; sin idempotencia, el
dominio del estudiante se infla y el sistema deja de ser fiable.
**Prueba independiente**: enviar dos veces la misma interacción con el mismo `client_seq` y comparar el
estado del dominio.
**Cobre**: FR-INT-004, FR-INT-006, AC-1.4

**Escenarios de aceptación**:

1. **Dado** una interacción registrada con `client_seq = 42`,
   **When** se reenvía con el mismo `client_seq`,
   **Entonces** queda **una** fila en `interaction_logs` y el dominio cambia **una** sola vez.
2. **Dado** una interacción ya registrada,
   **When** el cliente reintenta y recibe la respuesta original,
   **Entonces** el cliente no muestra error y no duplica el ítem en la sesión.

---

### US-1.5 — Auditar de dónde sale cada cambio de dominio (Prioridad: P2)

**Actor**: Administrador
**Por qué esta prioridad**: sin trazabilidad, cualquier disputa ("¿por qué el sistema me dio esto?") es
imposible de resolver y la calibración se vuelve a ciegas.
**Prueba independiente**: consultar la serie de interacciones de un estudiante y recomputar el mastery.
**Cobre**: FR-INT-003, FR-BKT-008, NFR-CONS-001

**Escenarios de aceptación**:

1. **Dado** un historial de interacciones,
   **When** el admin consulta el detalle de una interacción,
   **Entonces** ve el `mastery_delta` y el mastery antes/después.
2. **Dado** el historial completo de un estudiante y un micronodo,
   **When** se recalcula el mastery de forma independiente,
   **Entonces** el resultado coincide (±1e-8) con el valor almacenado.
3. **Dado** un parámetro BKT que cambió en el pasado,
   **When** se audita una interacción anterior al cambio,
   **Entonces** el sistema indica qué parámetros se aplicaron en ese momento.

---

### US-1.6 — Calibrar el modelo de un concepto (Prioridad: P2)

**Actor**: Administrador
**Por qué esta prioridad**: los parámetros por defecto no son universalmente válidos; sin calibración, el
motor puede bloquear a un estudiante o dejar huecos de conocimiento.
**Prueba independiente**: cambiar un parámetro y comprobar el efecto en una inferencia posterior.
**Cobre**: FR-BKT-004, FR-BKT-005, FR-BKT-009, AC-1.2

**Escenarios de aceptación**:

1. **Dado** un micronodo con parámetros `p_T`, `p_S`, `p_G`,
   **When** el admin actualiza `p_T`,
   **Then** el cambio queda auditado (actor, fecha, valor anterior y nuevo).
2. **Dado** una actualización de parámetros,
   **When** el admin emite un valor fuera de rango o `p_S = 0`,
   **Entonces** la API lo rechaza con `422` explicando la restricción (epsilon mínimo 1e-6).
3. **Dado** un cambio de parámetros,
   **When** se registra una interacción posterior al cambio,
   **Entonces** se aplica el valor nuevo; las interacciones históricas no se reinterpretan.

---

### US-1.7 — Simular una calibración antes de aplicarla (Prioridad: P3)

**Actor**: Administrador
**Por qué esta prioridad**: evita cambios de parámetros que degradan el aprendizaje de un curso entero;
es una herramienta de apoyo, no un flujo crítico.
**Prueba independiente**: ejecutar la simulación y verificar que la base de datos no cambia.
**Cobre**: FR-BKT-009, FR-DOC-006, AC-6.3

**Escenarios de aceptación**:

1. **Dado** un conjunto de parámetros nuevos,
   **When** el admin lanza la simulación,
   **Entonces** se muestra la distribución de mastery antes/después por curso y **no** se escribe nada.
2. **Dado** una simulación con parámetros inválidos,
   **When** se solicita,
   **Entonces** se devuelve el error de validación sin ejecutar la simulación.

---

### US-1.8 — Recalcular el dominio de forma masiva (Prioridad: P3)

**Actor**: Administrador
**Por qué esta prioridad**: es una operación de mantenimiento; solo es necesaria cuando cambian
parámetros o se corrigen datos históricos.
**Prueba independiente**: ejecutar el recálculo sobre un subconjunto de estudiantes y comparar.
**Cobre**: FR-INT-010, FR-BKT-005, NFR-PERF-002

**Escenarios de aceptación**:

1. **Dado** un conjunto de estudiantes con interacciones,
   **When** el admin lanza un recálculo,
   **Entonces** el trabajo se ejecuta en segundo plano, reporta progreso y no degrada el servicio en línea.
2. **Dado** un estudiante sin datos,
   **When** el recálculo lo incluye,
   **Entonces** no se crea fila de mastery (no hay qué derivar).
3. **Dado** un rol `teacher`,
   **When** intenta lanzar el recálculo,
   **Entonces** la API responde `403`.

---

## 3. F2 — Grafo de conocimiento, navegación y selección adaptativa

> **Objetivo de la funcionalidad**: decidir, de forma determinista y auditable, qué concepto trabaja el
> estudiante a continuación y por qué.

### US-2.1 — No avanzar sobre conceptos sin prerrequisitos (Prioridad: P1)

**Actor**: Estudiante
**Por qué esta prioridad**: es la garantía pedagógica central del grafo. Si el estudiante avanza sobre
huecos, el sistema deja de ser adaptativo.
**Prueba independiente**: grafo sembrado con dos micronodos encadenados y consulta de navegación.
**Cobre**: FR-NAV-002, FR-NAV-003, GRA-01, GRA-02, AC-2.1, AC-2.2, AC-2.5

**Escenarios de aceptación**:

1. **Dado** el micronodo `B` con prerrequisito `A` y `A` en `p = 0.90`,
   **When** el sistema calcula el frente de aprendizaje,
   **Entonces** `B` no aparece entre los desbloqueados.
2. **Dado** `A` en `p = 0.96`,
   **When** el sistema calcula el frente,
   **Entonces** `B` aparece desbloqueado y es candidato a ser el siguiente.
3. **Dado** un grafo con una arista que cerraría un ciclo,
   **When** el admin intenta crearla,
   **Entonces** la API responde `409` indicando el ciclo detectado y el grafo queda intacto.

---

### US-2.2 — Saber por qué me toca este concepto (Prioridad: P1)

**Actor**: Estudiante
**Por qué esta prioridad**: la adaptador opaca genera desconfianza; el razonamiento es lo que convierte el
sistema en un aliado pedagógico.
**Prueba independiente**: consultar la navegación y verificar que incluye razonamiento y contexto.
**Cobre**: FR-NAV-004, FR-NAV-006, GRA-06, GRA-07, AC-2.3, AC-2.4

**Escenarios de aceptación**:

1. **Dado** un estudiante con varios candidatos desbloqueados,
   **When** el sistema elige el siguiente,
   **Entonces** la respuesta incluye el micronodo, el ítem, el objetivo de aprendizaje y el razonamiento
   (política aplicada y valores de desempate).
2. **Dado** dos candidatos empatados en déficit, dificultad y decaimiento,
   **When** se repite la selección 100 veces con el mismo estado,
   **Entonces** la selección es idéntica en las 100 ejecuciones.
3. **Dado** un micronodo de un macronodo con contexto situado,
   **When** se sirve el ejercicio,
   **Entonces** el contexto situado (rol, situación, herramienta, artefacto) viaja con la respuesta.

---

### US-2.3 — Practicar en refuerzo cuando estoy cerca del dominio (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: es lo que convierte el modelo en un refuerzo real y evita que el estudiante
avance con un 85 % de dominio en un concepto clave.
**Prueba independiente**: sembrar `p = 0.85` y verificar la prioridad de la selección.
**Cobre**: FR-NAV-005, GRA-05, AC-2.2

**Escenarios de aceptación**:

1. **Dado** un micronodo en refuerzo (`0.80 ≤ p < 0.95`) y otro nunca intentado, ambos desbloqueados,
   **When** el sistema selecciona,
   **Entonces** elige el micronodo en refuerzo.
2. **Dado** una sesión con objetivo `repaso`,
   **When** el sistema selecciona,
   **Entonces** el objetivo de sesión prevalece sobre la prioridad por defecto y el razonamiento lo indica.

---

### US-2.4 — Elegir el objetivo de mi sesión (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: da control al usuario y permite demostrar el valor del motor con
intenciones distintas (diagnóstico, repaso, avance).
**Prueba independiente**: crear sesiones con cada objetivo y verificar políticas distintas.
**Cobre**: FR-NAV-010

**Escenarios de aceptación**:

1. **Dado** los objetivos `avance`, `repaso`, `diagnóstico` y `refuerzo`,
   **When** el estudiante inicia una sesión con cada uno,
   **Entonces** la política de selección cambia de forma explícita y documentada, y el razonamiento de la
   navegación indica qué objetivo se aplicó.
2. **Dado** una sesión con objetivo `diagnóstico`,
   **When** el sistema selecciona,
   **Entonces** prioriza conceptos nunca intentados aunque estén bloqueados por prerrequisitos
   (con la excepción de la validación matemática, que siempre se aplica).

---

### US-2.5 — Saber que ya no queda nada por delante (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: sin este estado, el estudiante ve un spinner infinito en la última sesión del
curso; es un fallo de experiencia fácil de evitar.
**Prueba independiente**: grafo con todos los micronodos dominados y consulta de navegación.
**Cobre**: FR-NAV-008, AC-2.6

**Escenarios de aceptación**:

1. **Dado** un estudiante que domina todos los micronodos desbloqueables,
   **When** pide la navegación,
   **Entonces** recibe el estado `sin_frente` con el macronodo dominado más reciente y opciones de repaso.
2. **Dado** un curso sin grafo publicado,
   **When** el estudiante entra a practicar,
   **Entonces** la UI comunica que el contenido está en preparación en lugar de mostrar un error técnico.

---

### US-2.6 — Administrar el grafo de conceptos (Prioridad: P2)

**Actor**: Administrador
**Por qué esta prioridad**: sin gestión del grafo el sistema no se puede ampliar con contenido nuevo; es
la operación de día a día del admin.
**Prueba independiente**: CRUD de micronodos, macronodos y aristas sobre un curso de prueba.
**Cobre**: FR-GRAPH-001, FR-GRAPH-002, FR-GRAPH-003, FR-NAV-001

**Escenarios de aceptación**:

1. **Dado** un curso en producción,
   **When** el admin edita el orden o el título de un micronodo,
   **Entonces** el grafo publicado no cambia retroactivamente para los estudiantes que ya lo usaron.
2. **Dado** un cambio en el grafo,
   **When** se guarda,
   **Entonces** queda auditado con actor, fecha y valores anteriores/nuevos.
3. **Dado** un micronodo nuevo sin ítems,
   **When** se crea,
   **Entonces** el sistema lo marca como pendiente de ítems y lo incluye en el flujo de pre-generación.

---

### US-2.7 — Proponer un prerrequisito a mi curso (Prioridad: P3)

**Actor**: Docente
**Por qué esta prioridad**: el docente conoce el aula y puede detectar relaciones que el diseño inicial no
contempló; es una mejora colaborativa.
**Prueba independiente**: crear una propuesta y verificar que queda pendiente de aprobación.
**Cobre**: FR-GRAPH-001, FR-GRAPH-002

**Escenarios de aceptación**:

1. **Dado** un docente,
   **When** propone una arista dentro de su curso,
   **Entonces** la arista queda en estado pendiente y no altera la navegación de los estudiantes.
2. **Dado** una propuesta que introduce un ciclo,
   **When** el admin la aprueba,
   **Entonces** la aprobación se rechaza y se informa del ciclo.

---

### US-2.8 — Ver el mapa de conceptos del curso (Prioridad: P3)

**Actor**: Estudiante
**Por qué esta prioridad**: ayuda a entender la progresión y a tener sensación de avance; no es necesario
para la adaptación.
**Prueba independiente**: vista del grafo con datos sembrados.
**Cobre**: FR-NAV-001, FR-NAV-006, GRA-03

**Escenarios de aceptación**:

1. **Dado** un curso con grafo publicado,
   **When** el estudiante abre el mapa,
   **Entonces** ve los macronodos, los micronodos y su estado de dominio sin exponer el valor numérico de
   la mastery.
2. **Dado** un micronodo bloqueado,
   **When** el estudiante lo visualiza,
   **Entonces** se indica qué concepto debe dominar antes, sin revelar la solución.

---

## 4. F3 — Generación y validación de reactivos

> **Objetivo de la funcionalidad**: producir ejercicios novos sin que ningún error matemático ni contenido
> malicioso llegue al estudiante.

### US-3.1 — Recibir un ejercicio nuevo de mi concepto (Prioridad: P1)

**Actor**: Estudiante
**Por qué esta prioridad**: sin generación, el banco de ítems es finito y la práctica se agota; es la
diferencial de valor frente a un cuestionario estático.
**Prueba independiente**: con un proveedor LLM stub determinista, generar un ítem y comprobar que llega a
estado `validated` y servible.
**Cobre**: FR-GEN-003, FR-GEN-004, FR-GEN-014, FR-GEN-018, AC-3.6

**Escenarios de aceptación**:

1. **Dado** un micronodo sin ítems publicados,
   **When** el sistema genera un ejercicio,
   **Entonces** el enunciado, las opciones, la pista, la explicación y la dificultad llegan al cliente.
2. **Dado** un proveedor que devuelve JSON con fences markdown,
   **When** se procesa la respuesta,
   **Entonces** se extrae, se valida contra el esquema y el ítem se guarda con su modelo, versión de prompt
   y coste.
3. **Dado** un proveedor que devuelve una respuesta que no cumple el esquema,
   **When** se procesa,
   **Entonces** la respuesta se descarta con motivo `schema_invalid` y no se crea ningún ítem.

---

### US-3.2 — Que ningún ejercicio con error matemático llegue (Prioridad: P1)

**Actor**: Estudiante
**Por qué esta prioridad**: es la garantía de corrección matemática del producto; sin ella, el sistema
enseña cosas falsas.
**Prueba independiente**: generar ítems con soluciones erróneas mediante un LLM stub y comprobar que se
rechazan.
**Cobre**: FR-GEN-005, FR-GEN-007, FR-GEN-008, FR-GEN-010, FR-GEN-014, NFR-SEC-002, AC-3.1, AC-3.2, AC-3.3

**Escenarios de aceptación**:

1. **Dado** un ítem cuya opción correcta no satisface la ecuación del enunciado,
   **When** se valida,
   **Entonces** se rechaza con motivo `sympy_invalid` y nunca se publica.
2. **Dado** una ecuación cuadrática con dos soluciones,
   **When** un estudiante responde con cualquiera de las dos,
   **Entonces** la respuesta se considera correcta.
3. **Dado** un enunciado con una variable distinta de `x`,
   **When** se valida,
   **Entonces** el validador resuelve en la variable declarada y no devuelve soluciones erróneas.
4. **Dado** un fallo interno del validador,
   **When** se produce,
   **Entonces** se registra una excepción tipificada (no genérica) y el ítem no se publica.

---

### US-3.3 — Que un contenido malicioso no me afecte (Prioridad: P1)

**Actor**: Estudiante
**Por qué esta prioridad**: la entrada del LLM es no confiable y podría ejecutar código en el servidor; es
un riesgo de seguridad, no de calidad.
**Prueba independiente**: enviar un ítem con `__import__('os')` en el enunciado y comprobar que se rechaza
sin efectos observables.
**Cobre**: FR-GEN-005, FR-GEN-006, FR-GEN-024, NFR-SEC-002, AC-3.4, AC-3.5

**Escenarios de aceptación**:

1. **Dado** un enunciado con una expresión tipo `__import__('os').system('...')`,
   **When** se valida,
   **Entonces** se rechaza en el parseo, no se ejecuta nada y no se registran efectos secundarios.
2. **Dado** una expresión patológica que consume CPU,
   **When** se valida,
   **Entonces** termina dentro del timeout (2 s por defecto), se registra `SympyTimeoutError` y el proceso
   libera la CPU.
3. **Dado** un enunciado con HTML/script ejecutable,
   **When** se persiste,
   **Entonces** se sanea antes de almacenarse y no se renderiza como código en el cliente.

---

### US-3.4 — Ver por qué se rechazó un ejercicio generado (Prioridad: P2)

**Actor**: Docente
**Por qué esta prioridad**: sin visibilidad del motivo de rechazo, el docente no puede confiar en el
sistema ni ajustar la generación; es requisito del panel de revisión.
**Prueba independiente**: consultar el `validation_report` de un ítem rechazado.
**Cobre**: FR-GEN-013, FR-GEN-021, FR-DOC-004, AC-3.11

**Escenarios de aceptación**:

1. **Dado** un ítem rechazado,
   **When** el docente lo abre en el panel,
   **Entonces** ve los checks ejecutados, su resultado, el motivo (`sympy_invalid`, `sympy_timeout`,
   `schema_invalid`, `duplicate`, `quality_flags`, `quota_exceeded`, `provider_error`) y las soluciones
   encontradas.
2. **Dado** un ítem validado,
   **When** el docente lo abre,
   **Entonces** ve el informe, el modelo que lo generó y su coste.

---

### US-3.5 — Aceptar cualquier respuesta correcta válida (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: marcar como incorrecta una respuesta correcta alternativa destruye la confianza
en el sistema y falsea el dominio.
**Prueba independiente**: suite de ecuaciones con soluciones múltiples contra respuestas alternativas.
**Cobre**: FR-GEN-008, FR-GEN-009, AC-3.2

**Escenarios de aceptación**:

1. **Dado** un ítem con soluciones `{2, -1/2}`,
   **When** el estudiante responde `-0.5`,
   **Entonces** se acepta tras normalizar decimal y fracción.
2. **Dado** un enunciado con una igualdad ambigua (`x = 2 = 3`),
   **When** se valida,
   **Entonces** se rechaza con un motivo específico, sin convertir el problema en otra expresión.

---

### US-3.6 — Que el generador se corrija solo tras un error (Prioridad: P2)

**Actor**: Administrador
**Por qué esta prioridad**: la reflexión real multiplica la tasa de ítems válidos y reduce el coste
desperdiciado; es la diferencia entre un generador usable y uno que falla la mitad de las veces.
**Prueba independiente**: proveedor stub que falla la primera validación y acierta la segunda; verificar
el historial del prompt.
**Cobre**: FR-GEN-011, FR-GEN-012, NFR-TEST-002, AC-3.7

**Escenarios de aceptación**:

1. **Dado** un ítem que falla la validación matemática,
   **When** el motor reintenta,
   **Entonces** el siguiente prompt **incluye** el error de validación y el historial de la conversación.
2. **Dado** un error de validación (no de red),
   **When** se reintenta,
   **Entonces** el reintento es inmediato (sin backoff temporal).
3. **Dado** un error de red o `429`,
   **When** se reintenta,
   **Entonces** se aplica backoff exponencial con jitter y se respeta `Retry-After`.

---

### US-3.7 — Que el sistema no se caiga si el proveedor falla (Prioridad: P2)

**Actor**: Administrador
**Por qué esta prioridad**: la disponibilidad del servicio depende de un tercero; sin degradación elegante,
un corte del proveedor tumba la práctica.
**Prueba independiente**: simular el proveedor caído y verificar que la sesión continúa con el banco
pregenerado.
**Cobre**: FR-GEN-011, FR-GEN-020, FR-GEN-021, FR-NAV-007, AC-3.9

**Escenarios de aceptación**:

1. **Dado** un proveedor caído y un ítem pregenerado equivalente,
   **When** el estudiante necesita un ejercicio,
   **Entonces** se sirve el ítem del banco y la sesión continúa.
2. **Dado** un proveedor caído y sin ítems de fallback,
   **When** el estudiante pide un ejercicio,
   **Entonces** recibe un estado de indisponibilidad explícito y accionable, nunca un ítem no validado.
3. **Dado** un fallo del proveedor,
   **When** se registra,
   **Entonces** queda en la DLQ o en el histórico de descartes con su código de motivo.

---

### US-3.8 — Pre-generar el banco de ejercicios del curso (Prioridad: P2)

**Actor**: Administrador
**Por qué esta prioridad**: la pre-generación es la vía principal de aprovisionamiento (elimina la latencia
de 5–15 s del camino crítico del estudiante).
**Prueba independiente**: ejecutar un lote sobre un micronodo y verificar el resultado por ítem.
**Cobre**: FR-GEN-019, FR-GEN-022, FR-DOC-007

**Escenarios de aceptación**:

1. **Dado** un curso con micronodos sin cobertura,
   **When** el admin lanza la pre-generación,
   **Entonces** se genera un lote con el coste estimado visible antes de empezar y el resultado por ítem al
   terminar.
2. **Dado** un lote que agota el presupuesto de coste,
   **When** ocurre,
   **Entonces** el lote se detiene en un ítem (no a mitad) y los ya validados se conservan.

---

### US-3.9 — Conocer el coste de generar ejercicios (Prioridad: P3)

**Actor**: Administrador
**Por qué esta prioridad**: sin control de coste, la generación puede multiplicar la factura sin que nadie
lo note.
**Prueba independiente**: generar en modo stub con coste simulado y consultar los acumulados.
**Cobre**: FR-GEN-015, FR-GEN-018, FR-DOC-010, NFR-COST-001

**Escenarios de aceptación**:

1. **Dado** un curso con generaciones,
   **When** el admin consulta el coste,
   **Entonces** ve el acumulado por curso y por periodo y el coste medio por ítem.
2. **Dado** el presupuesto alcanzado,
   **When** se intenta una generación,
   **Entonces** el sistema se detiene y reporta, sin degradar silenciosamente ni exceder el límite.

---

### US-3.10 — Que no se repitan ejercicios equivalentes (Prioridad: P3)

**Actor**: Docente
**Por qué esta prioridad**: la repetición percibida hace que el banco parezca más pequeño de lo que es y
degrada la confianza en el generador.
**Prueba independiente**: generar dos veces el mismo micronodo y comparar.
**Cobre**: FR-GEN-016, FR-GEN-017, AC-3.10

**Escenarios de aceptación**:

1. **Dado** un ítem ya publicado con el mismo enunciado normalizado,
   **When** se genera otro equivalente,
   **Entonces** se rechaza con motivo `duplicate`.
2. **Dado** un ítem con señales de calidad dudosas (enunciado ambiguo, distractores implausibles),
   **When** se valida,
   **Entonces** se registra la señal de calidad y el docente la ve en el panel (la validación matemática no
   se presenta como evidencia de calidad didáctica).

---

## 5. F4 — Sesión de práctica y experiencia del estudiante (frontend)

> **Objetivo de la funcionalidad**: que el estudiante practique sin fricción y con feedback inmediato.

### US-4.1 — Empezar a practicar y ver mi primer ejercicio (Prioridad: P1)

**Actor**: Estudiante
**Por qué esta prioridad**: es el primer contacto con el producto; si tarda o falla, no hay sesión.
**Prueba independiente**: e2e con backend sembrado midiendo el tiempo hasta el primer ítem.
**Cobre**: FR-UX-001, FR-UX-002, FR-UX-012, NFR-PERF-006, AC-4.1

**Escenarios de aceptación**:

1. **Dado** un estudiante autenticado y un ítem cacheado,
   **When** inicia una sesión,
   **Entonces** ve el primer ejercicio en menos de 3 s (p95) y el render del ítem en menos de 1.5 s tras la
   respuesta del backend.
2. **Dado** un ítem que aún se está generando,
   **When** el cliente lo solicita,
   **Entonces** ve un estado de carga informative, no un error, y el ítem llega al terminar.
3. **Dado** un objetivo de sesión,
   **When** el estudiante lo elige,
   **Entonces** la sesión se crea con ese objetivo y se refleja en el razonamiento de la navegación.

---

### US-4.2 — Responder y recibir feedback inmediato (Prioridad: P1)

**Actor**: Estudiante
**Por qué esta prioridad**: el bucle de retroalimentación es el corazón de la práctica; sin él no hay
aprendizaje.
**Prueba independiente**: e2e respondiendo un ítem sembrado y comprobando el feedback y la actualización de
dominio.
**Cobre**: FR-UX-003, FR-INT-001, AC-4.2

**Escenarios de aceptación**:

1. **Dado** un ítem servido,
   **When** el estudiante responde,
   **Entonces** ve en menos de 300 ms si acertó, la explicación y (si aplica) la pista.
2. **Dado** una respuesta enviada,
   **When** el servidor la registra,
   **Entonces** la mastery del concepto se actualiza antes de la siguiente pregunta.
3. **Dado** una respuesta incorrecta,
   **When** se muestra el feedback,
   **Entonces** se muestra la explicación y el sistema ofrece continuar con el siguiente ejercicio.

---

### US-4.3 — Ver el contexto real en el que aplico el concepto (Prioridad: P1)

**Actor**: Estudiante
**Por qué esta prioridad**: el "nodo situado" es el diferencial pedagógico del producto; sin contexto, el
sistema es un cuestionario más.
**Prueba independiente**: e2e verificando que el contexto situado del macronodo aparece en la pantalla del
ítem.
**Cobre**: FR-UX-002, FR-UX-003, GRA-06, FR-GEN-004, AC-2.4

**Escenarios de aceptación**:

1. **Dado** un ítem de un macronodo situado,
   **When** se muestra,
   **Entonces** el estudiante ve rol, situación, herramienta y objetivo de aprendizaje del concepto.
2. **Dado** un ejercicio generado con ese contexto,
   **When** se revisa el ítem,
   **Entonces** el contexto aparece en el prompt de generación y en el enunciado.

---

### US-4.4 — Ver en qué estado está mi dominio (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: da sensación de progreso y orienta; no es requisito para la adaptación.
**Prueba independiente**: vista de progreso con datos sembrados.
**Cobre**: FR-UX-004, FR-UX-016, P-06

**Escenarios de aceptación**:

1. **Dado** un estudiante con progreso,
   **When** abre su vista de progreso,
   **Entonces** ve el estado por concepto (`pendiente`, `en refuerzo`, `dominado`) y los conceptos
   dominados ganados.
2. **Dado** la misma vista,
   **When** se inspecciona la respuesta de la API,
   **Entonces** no se expone el valor numérico de la mastery (decisión de producto P-06).

---

### US-4.5 — Pedir una pista sin perder mi progreso (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: el uso de pistas es una señal pedagógica importante y su mal diseño sesga el
dominio.
**Prueba independiente**: e2e pidiendo una pista y verificando la interacción registrada.
**Cobre**: FR-UX-003, FR-INT-001

**Escenarios de aceptación**:

1. **Dado** un ítem con pista,
   **When** el estudiante la pide,
   **Entonces** ve la pista y la interacción se registra con `hints_used` incrementado.
2. **Dado** una respuesta tras usar pistas,
   **When** se registra,
   **Entonces** el dominio se actualiza de forma coherente y la auditoría refleja el uso de pistas.

---

### US-4.6 — Saltar un ejercicio que no me sale (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: obligar a responder bloquea al estudiante; el ítem saltado no debe contar como
fallo.
**Prueba independiente**: e2e saltando un ítem y verificando que el dominio no cambia.
**Cobre**: FR-UX-011, FR-INT-009

**Escenarios de aceptación**:

1. **Dado** un ítem servido,
   **When** el estudiante lo salta,
   **Entonces** se registra como `skipped`, **no** afecta al dominio y aparece en las métricas de sesión.
2. **Dado** un ítem saltado,
   **When** el sistema selecciona el siguiente,
   **Entonces** el ítem saltado no se vuelve a servir en esa misma sesión.

---

### US-4.7 — Practicar aunque se caiga la conexión (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: en la realidad de las aulas y la conectividad móvil, perder el progreso por
un corte de red destruye la confianza en el sistema.
**Prueba independiente**: e2e con red interceptada y caída, restaurando la conexión después.
**Cobre**: FR-UX-008, FR-UX-010, FR-UX-015, NFR-REL-001, AC-4.3

**Escenarios de aceptación**:

1. **Dado** una conexión caída al responder,
   **When** el estudiante responde,
   **Entonces** la interacción queda en una cola local persistente, se muestra el aviso "sin conexión" y se
   entrega al recuperar la red.
2. **Dado** una cola local con eventos pendientes,
   **When** se recarga la página,
   **Entonces** los eventos pendientes se reintentan sin duplicarse (idempotencia por `client_seq`).
3. **Dado** una petición en vuelo lenta,
   **When** el estudiante navega a otra pantalla,
   **Entonces** puede cancelar la solicitud sin bloquear la sesión.

---

### US-4.8 — Cerrar la pestaña sin perder lo que hice (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: el buffering temporal pierde eventos al cerrar el navegador, lo que destruye
las métricas y puede perder la respuesta enviada.
**Prueba independiente**: test de navegador que cierra la pestaña con eventos pendientes y verifica su
llegada al servidor.
**Cobre**: FR-UX-009, FR-TEL-003, AC-4.4, AC-5.3

**Escenarios de aceptación**:

1. **Dado** 3 eventos pendientes en la cola local,
   **When** el estudiante cierra la pestaña,
   **Entonces** los 3 llegan al servidor vía `sendBeacon` (o el mecanismo equivalente).
2. **Dado** un evento oculto por `visibilitychange`,
   **When** cambia la visibilidad de la pestaña,
   **Entonces** la cola se drena de inmediato y el resto se entrega por reintento.

---

### US-4.9 — Responder con el mismo ítem más de una vez (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: los reintentos son normales; el sistema debe contar los intentos sin inflar el
dominio.
**Prueba independiente**: e2e respondiendo dos veces al mismo ítem.
**Cobre**: FR-UX-005, FR-INT-001, AC-1.4

**Escenarios de aceptación**:

1. **Dado** un ítem respondido una vez,
   **When** el estudiante vuelve a responderlo,
   **Entonces** `attempts` se incrementa y solo la última respuesta cuenta para el dominio.
2. **Dado** un reintento de red con el mismo `client_seq`,
   **When** el servidor lo recibe,
   **Entonces** responde con el resultado original y no vuelve a aplicar la inferencia.

---

### US-4.10 — Usar la plataforma solo con teclado (Prioridad: P3)

**Actor**: Estudiante
**Por qué esta prioridad**: requisito de accesibilidad y de inclusión; no bloquea la entrega del MVP, pero
es un compromiso de producto y, con datos de menores, una obligación de diseño.
**Prueba independiente**: auditoría de teclado y lector de pantalla sobre el flujo de práctica.
**Cobre**: FR-UX-013, FR-UX-014, NFR-A11Y-001, AC-4.5

**Escenarios de aceptación**:

1. **Dado** un estudiante que navega solo con teclado,
   **When** completa una sesión,
   **Entonces** puede responder, pedir pista, saltar y cerrar sin ratón.
2. **Dado** un usuario con baja visión,
   **When** usa el flujo,
   **Entonces** el feedback no depende solo del color y los controles tienen etiquetas accesibles.

---

### US-4.11 — Ver el resumen de mi sesión (Prioridad: P3)

**Actor**: Estudiante
**Por qué esta prioridad**: da cierre y reforzar el avance; es la puerta de entrada al resto de la vista de
progreso.
**Prueba independiente**: e2e cerrando una sesión de 5 ítems.
**Cobre**: FR-UX-016

**Escenarios de aceptación**:

1. **Dado** una sesión con 5 ítems respondidos,
   **When** el estudiante la cierra,
   **Entonces** ve ítems respondidos, aciertos, conceptos trabajados, conceptos dominados ganados y tiempo
   total.

---

## 6. F5 — Telemetría del estudiante

> **Objetivo de la funcionalidad**: entender cómo se usa el producto sin que la telemetría influya en la
> evaluación.

### US-5.1 — Saber qué conceptos se me están resistiendo (Prioridad: P3)

**Actor**: Docente
**Por qué esta prioridad**: la analítica de abandono por concepto es un insumo de diseño del curso; útil,
pero no imprescindible para la adaptación.
**Prueba independiente**: eventos sembrados y agregación sobre `interaction_logs`.
**Cobre**: FR-TEL-007, FR-DOC-002

**Escenarios de aceptación**:

1. **Dado** eventos de telemetría de varios estudiantes,
   **When** el docente consulta las métricas de su curso,
   **Entonces** ve abandono por ítem y por concepto, tiempo medio por ítem, ratio de pistas y ratio de
   intentos múltiples.
2. **Dado** un curso con pocos datos,
   **When** se consultan las métricas,
   **Entonces** se indica el tamaño de la muestra para no interpretar ruido como tendencia.

---

### US-5.2 — Saber dónde se abandona la práctica (Prioridad: P3)

**Actor**: Administrador
**Por qué esta prioridad**: alimenta la mejora del producto y del contenido; no es un flujo de usuario.
**Prueba independiente**: consulta de métricas agregadas con eventos sembrados.
**Cobre**: FR-TEL-007, FR-DOC-010

**Escenarios de aceptación**:

1. **Dado** eventos de `tab_hidden` y `session_end` incompletos,
   **When** se calcula la tasa de abandono,
   **Entonces** el resultado es consistente con los datos brutos (±1 %).

---

### US-5.3 — Que un evento inválido no rompa mi sesión (Prioridad: P3)

**Actor**: Estudiante
**Por qué esta prioridad**: robustez del cliente ante eventos que el servidor no conoce; evita que una
mejora futura del cliente rompa la versión desplegada.
**Prueba independiente**: enviar un `event_type` fuera de la allowlist.
**Cobre**: FR-TEL-005

**Escenarios de aceptación**:

1. **Dado** un cliente que emite un `event_type` desconocido,
   **When** lo envía,
   **Entonces** el servidor responde `422` con el código `unknown_event_type` y la sesión del estudiante
   continúa sin errores.

---

## 7. F6 — Panel docente y administración del banco de ítems

> **Objetivo de la funcionalidad**: dar control al docente y al administrador sobre contenido, calibración y
> operación.

### US-6.1 — Ver el dominio de cada estudiante de mi curso (Prioridad: P1)

**Actor**: Docente
**Por qué esta prioridad**: es la vista que justifica el proyecto desde el lado del docente; sin ella, el
sistema solo sirve al estudiante.
**Prueba independiente**: vista de curso con datos sembrados y token de docente.
**Cobre**: FR-DOC-001, FR-DOC-002, FR-AUD-001, AC-6.4

**Escenarios de aceptación**:

1. **Dado** un curso con estudiantes,
   **When** el docente abre la vista de curso,
   **Entonces** ve la distribución de dominio por macronodo, la media del curso y el detalle por estudiante
   y concepto.
2. **Dado** estudiantes con inactividad o dominio bajo,
   **When** se genera la lista de alertas,
   **Entonces** aparecen destacados para la revisión del docente.

---

### US-6.2 — Exportar el dominio de mi curso (Prioridad: P2)

**Actor**: Docente
**Por qué esta prioridad**: necesario para compartir resultados con el resto del equipo o integrarlos en
procesos del centro; no es crítico en el día a día.
**Prueba independiente**: descarga del CSV y comparación con la vista.
**Cobre**: FR-DOC-003, NFR-SEC-006

**Escenarios de aceptación**:

1. **Dado** un curso,
   **When** el docente exporta,
   **Entonces** el CSV contiene una fila por estudiante y micronodo con mastery, estado, intentos, aciertos y
   última actividad.
2. **Dado** un docente sin permiso sobre el curso,
   **When** pide la exportación por API,
   **Entonces** la respuesta es `403`.

---

### US-6.3 — Revisar y aprobar un ejercicio generado (Prioridad: P2)

**Actor**: Docente
**Por qué esta prioridad**: cierra el ciclo de calidad del banco; sin revisión humana, el banco generado no
es publicable en un contexto real.
**Prueba independiente**: flujo de revisión sobre un ítem validado.
**Cobre**: FR-DOC-004, FR-DOC-005, FR-GEN-014, AC-3.11

**Escenarios de aceptación**:

1. **Dado** un ítem en estado `validated`,
   **When** el docente lo aprueba,
   **Entonces** pasa a `published` y queda disponible para servirse.
2. **Dado** un ítem generado,
   **When** el docente lo revisa,
   **Entonces** ve enunciado, opciones, informe de validación, modelo, coste y motivos de rechazo de los
   descartados.
3. **Dado** un ítem `rejected`,
   **When** el docente lo archiva,
   **Entonces** deja de ofrecerse y queda registrado en el histórico.

---

### US-6.4 — Corregir un ejercicio sin publicar un error (Prioridad: P2)

**Actor**: Docente
**Por qué esta prioridad**: los reactivos generados siempre requieren retoques; sin revalidación obligatoria,
una edición manual introduce errores que el sistema había detectado.
**Prueba independiente**: editar la solución de un ítem y verificar que no se puede publicar sin revalidar.
**Cobre**: FR-DOC-005, FR-GEN-005, AC-6.2

**Escenarios de aceptación**:

1. **Dado** un ítem `published` cuya solución se edita,
   **When** se intenta publicar,
   **Entonces** el sistema exige una validación simbólica nueva y bloquea la publicación hasta que pase.
2. **Dado** una edición que introduce una solución incorrecta,
   **When** se revalida,
   **Entonces** la validación falla y el ítem no vuelve a `published`.

---

### US-6.5 — No ver datos de cursos que no son míos (Prioridad: P1)

**Actor**: Docente
**Por qué esta prioridad**: es un requisito de aislamiento de datos de menores y dearios; una fuga aquí
es un incidente de privacidad, no un fallo de UX.
**Prueba independiente**: intento directo a la API de un curso ajeno.
**Cobre**: FR-AUTH-003, NFR-SEC-006, AC-6.1

**Escenarios de aceptación**:

1. **Dado** un docente del curso A,
   **When** intenta leer el dominio del curso B por API,
   **Entonces** la respuesta es `403` y el acceso queda auditado.
2. **Dado** un panel con controles de navegación,
   **When** el docente navega,
   **Entonces** solo aparecen sus cursos; la restricción se aplica **en servidor**, no solo ocultando
   botones.

---

### US-6.6 — Operar la cola de mensajes fallidos (Prioridad: P3)

**Actor**: Administrador
**Por qué esta prioridad**: es la herramienta de recuperación ante fallos de procesamiento; necesaria para
operar, no para la entrega.
**Prueba independiente**: provocar un fallo, verificar la DLQ y reprocesar.
**Cobre**: FR-DOC-009, FR-EVT-004, AC-7.3

**Escenarios de aceptación**:

1. **Dado** un mensaje fallido 3 veces,
   **When** el admin consulta las colas,
   **Entonces** el mensaje aparece en la DLQ con su motivo y su `correlationId`.
2. **Dado** un mensaje en la DLQ,
   **When** el admin lo reprocesa,
   **Entonces** se reencola y la operación queda auditada.

---

### US-6.7 — Ver la salud del sistema y su coste (Prioridad: P3)

**Actor**: Administrador
**Por qué esta prioridad**: el panel operativo es necesario cuando el sistema está en producción; puede
vivir en el dashboard de observabilidad durante el MVP.
**Prueba independiente**: populate de métricas y consulta del panel.
**Cobre**: FR-DOC-010, FR-OBS-002, FR-OBS-003, NFR-COST-001

**Escenarios de aceptación**:

1. **Dado** el sistema en operación,
   **When** el admin abre el panel,
   **Entonces** ve p95 de entrega del primer ítem, tasa de reactivos descartados, errores por proveedor,
   profundidad de cola y coste acumulado.
2. **Dado** un presupuesto de coste al 80 %,
   **When** se consulta,
   **Entonces** el panel lo destaca como alerta.

---

## 8. F7 — Confianza, asincronía, despliegue y operación

> **Objetivo de la funcionalidad**: que el sistema sea seguro, operable y resiliente. Las historias de esta
> sección son transversales y se implementan de forma incremental en cada fase.

### US-7.1 — Entrar al sistema con mi identidad (Prioridad: P1)

**Actor**: Estudiante / Docente / Administrador
**Por qué esta prioridad**: sin autenticación y con identidad enviada por el cliente, un estudiante podría
registrar el progreso de otro; es un requisito de integridad y de privacidad.
**Prueba independiente**: suite de contrato con tokens válidos, expirados y manipulados.
**Cobre**: FR-AUTH-001, FR-AUTH-002, FR-AUTH-003, FR-AUTH-004, FR-AUTH-005, NFR-SEC-001, AC-7.1

**Escenarios de aceptación**:

1. **Dado** un token válido con rol `student`,
   **When** llama a un endpoint protegido,
   **Entonces** la petición se atiende con la identidad del token.
2. **Dado** un token expirado o con firma inválida,
   **When** llama a cualquier endpoint protegido,
   **Entonces** la respuesta es `401` en todos los servicios.
3. **Dado** un token de rol `student`,
   **When** llama a un endpoint de administración,
   **Entonces** la respuesta es `403` y el intento se audita.
4. **Dado** un cliente que envía `studentId` en el cuerpo,
   **When** el servidor procesa la petición,
   **Entonces** el valor del cuerpo se ignora y el del token prevalece.

---

### US-7.2 — Consentimiento antes de usar el sistema si soy menor (Prioridad: P2)

**Actor**: Estudiante (menor de edad) / Tutor
**Por qué esta prioridad**: es una obligación legal y de política del producto con datos de menores; bloquea
el uso real, aunque no el desarrollo técnico.
**Prueba independiente**: intentar crear una sesión sin consentimiento registrado.
**Cobre**: FR-PRIV-001, FR-PRIV-002, AC-7.8

**Escenarios de aceptación**:

1. **Dado** un estudiante sin consentimiento registrado,
   **When** intenta iniciar una sesión de práctica,
   **Entonces** la acción queda bloqueada y se indica el motivo y el proceso de registro.
2. **Dado** un consentimiento registrado,
   **When** el estudiante inicia sesión,
   **Entonces** el sistema funciona con normalidad y registra la versión del consentimiento aceptado.

---

### US-7.3 — Pedir la eliminación de mis datos (Prioridad: P3)

**Actor**: Estudiante / Tutor
**Por qué esta prioridad**: derecho de supresión; se implementa como proceso operativo y puede llegar tras
el MVP.
**Prueba independiente**: solicitud de borrado sobre un usuario sembrado y verificación del estado final.
**Cobre**: FR-PRIV-003, FR-PRIV-004, NFR-PRIV-001

**Escenarios de aceptación**:

1. **Dado** una solicitud de acceso y supresión,
   **When** se ejecuta,
   **Entonces** se exportan los datos, se borran las interacciones y la telemetría, y se conserva evidencia
   auditable de la ejecución.
2. **Dado** datos con política de retención vigente,
   **When** se ejecuta el borrado,
   **Entonces** solo se eliminan los datos que la política permite eliminar.

---

### US-7.4 — Que el acceso a datos de estudiantes quede registrado (Prioridad: P3)

**Actor**: Administrador
**Por qué esta prioridad**: requisito de auditoría y de cumplimiento; se resuelve con el log de auditoría de
solo lectura, ya previsto en el logging estructurado.
**Prueba independiente**: consultar el log tras una operación de lectura sensible.
**Cobre**: FR-PRIV-005, FR-OBS-004

**Escenarios de aceptación**:

1. **Dado** un docente que consulta el dominio de un estudiante,
   **When** la operación se completa,
   **Entonces** queda registrada con actor, recurso, fecha y `correlationId`.
2. **Dado** el log de auditoría,
   **When** se intenta modificar,
   **Entonces** el sistema lo rechaza (solo lectura).

---

### US-7.5 — Seguir una petición de extremo a extremo (Prioridad: P2)

**Actor**: Operador
**Por qué esta prioridad**: sin trazas distribuidas, el trabajo asíncrono es una caja negra; es la base del
resto de la observabilidad.
**Prueba independiente**: trazabilidad de una petición que atraviesa core → cola → generación.
**Cobre**: FR-OBS-001, FR-EVT-001, FR-OBS-004, AC-7.2

**Escenarios de aceptación**:

1. **Dado** una petición del cliente,
   **When** se emite un evento a la cola,
   **Entonces** el `traceId` aparece en los logs del core, en los headers del mensaje y en los logs del
   servicio de generación.
2. **Dado** un evento con `correlationId`,
   **When** se busca en el panel de trazas,
   **Entonces** se recuperan todas las operaciones relacionadas.

---

### US-7.6 — Ser avisado cuando algo se degrada (Prioridad: P3)

**Actor**: Operador
**Por qué esta prioridad**: las alertas cierran el ciclo de operación; se montan junto con el despliegue.
**Prueba independiente**: provocar cada condición y verificar la alerta.
**Cobre**: FR-OBS-003, FR-EVT-004, AC-7.3

**Escenarios de aceptación**:

1. **Dado** un mensaje en la DLQ,
   **When** se crea la condición,
   **Entonces** se emite una alerta con el motivo y el `correlationId` del mensaje.
2. **Dado** un p95 de entrega por encima del SLO,
   **When** se evalúa la alerta,
   **Entonces** se notifica al equipo con la métrica y la ventana temporal.

---

### US-7.7 — Que el servicio aguante más carga de la prevista (Prioridad: P3)

**Actor**: Operador
**Por qué esta prioridad**: la escalabilidad se demuestra con carga real; es la última fase del roadmap.
**Prueba independiente**: prueba de escalado con carga en la cola.
**Cobre**: FR-DEP-004, FR-DEP-006, FR-DEP-002, FR-DEP-003, NFR-REL-003, NFR-IA-001, AC-7.4, AC-7.5

**Escenarios de aceptación**:

1. **Dado** 500 mensajes pendientes en la cola,
   **When** se aplica la política de escalado,
   **Entonces** el número de réplicas aumenta por profundidad de cola (no solo por CPU).
2. **Dado** los manifiestos de Kubernetes,
   **When** se aplican,
   **Entonces** los pods quedan `Ready` con `livenessProbe`, `readinessProbe` y `startupProbe` configuradas.
3. **Dado** una actualización de versión,
   **When** se despliega,
   **Entonces** no hay pérdida de servicio (rolling update sin instancias indisponibles).

---

### US-7.8 — Que un cambio con el núcleo roto no llegue a producción (Prioridad: P2)

**Actor**: Equipo de desarrollo
**Por qué esta prioridad**: la Constitución la fija como puerta de calidad; un BKT o un validador sin
pruebas puede corromper el dominio o enseñar matemáticas falsas.
**Prueba independiente**: suite de la CI con un fallo inyectado en el test del trigger.
**Cobre**: FR-CICD-001, FR-CICD-002, FR-CICD-003, NFR-TEST-001, NFR-TEST-002, NFR-TEST-003, NFR-TEST-004, AC-7.6

**Escenarios de aceptación**:

1. **Dado** un cambio que rompe la función BKT,
   **When** se ejecuta la CI,
   **Entonces** la CI falla en el test del trigger y el merge queda bloqueado.
2. **Dado** un cambio que rompe el validador simbólico,
   **When** se ejecuta la CI,
   **Entonces** fallan los property-based tests y el merge queda bloqueado.
3. **Dado** una migración de esquema incompatible,
   **When** se ejecuta la CI,
   **Entonces** la verificación de migración falla antes del despliegue.

---

### US-7.9 — Recibir el ejercicio generado sin quedarme esperando en silencio (Prioridad: P2)

**Actor**: Estudiante
**Por qué esta prioridad**: la generación tarda 5–15 s; sin streaming ni estado, la sesión parece colgada y
el estudiante abandona.
**Prueba independiente**: e2e con proveedor lento y cliente que se desconecta y reconecta.
**Cobre**: FR-EVT-005, FR-EVT-006, FR-NAV-007, FR-UX-012, AC-7.7

**Escenarios de aceptación**:

1. **Dado** una solicitud de generación en curso,
   **When** el cliente se conecta al stream,
   **Entonces** recibe eventos de progreso y el estado final con el ítem.
2. **Dado** una desconexión durante el streaming,
   **When** el cliente se reconecta,
   **Entonces** recupera el estado del trabajo y el ítem no se pierde ni se genera dos veces.
3. **Dado** un trabajo de larga duración en el core,
   **When** se ejecuta,
   **Entonces** no bloquea el hilo de la petición HTTP ni el event loop.

---

## 9. Recortes MVP (slices verticales demostrables)

Cada slice es un incremento entregable y demostrable que agrupa historias. El orden respeta la ruta
crítica definida en el PRD (§10.3).

| Slice | Objetivo demo | Historias | Fase PRD | Puerta de salida |
|---|---|---|---|---|
| **M0 — Dominio verificable** | "Respondo un ítem y mi dominio se recalcula en PostgreSQL" | US-1.1, US-1.2, US-1.5, US-2.1, US-2.2 | F1–F2 | Test del trigger en verde + API de interacciones y navegación operativas |
| **M1 — Sesión de práctica end-to-end** | "Estudiante completa una sesión real de 5 ítems" | US-4.1, US-4.2, US-4.3, US-4.9, US-1.4, US-7.1 | F3 | e2e verde; identidad desde token; sin doble conteo |
| **M2 — Generación validada** | "El sistema genera un ejercicio nuevo y lo publica solo si es correcto" | US-3.1, US-3.2, US-3.3, US-3.6, US-3.8 | F4 | Tests de seguridad y property-based en verde |
| **M3 — Resiliencia y asincronía** | "Si el proveedor falla, el estudiante sigue practicando" | US-3.7, US-7.9, US-7.5, US-4.8, US-4.7 | F5 | Outbox + SSE + DLQ operando; pérdida de telemetría < 0,5 % |
| **M4 — Panel docente** | "El docente ve el dominio de su curso y aprueba reactivos" | US-6.1, US-6.5, US-6.3, US-6.2, US-2.6, US-1.6 | F6 | Aislamiento por curso probado; flujo de aprobación completo |
| **M5 — Operación y cumplimiento** | "El sistema se opera, se audita y se despliega" | US-7.2, US-7.4, US-7.6, US-7.7, US-7.8, US-6.6 | F7 | Manifiestos aplicados, alertas activas, CI bloqueando el núcleo roto |

**M0 + M1 = MVP demostrable.** M2 es el primer salto de valor diferencial. M3–M5 endurecen el producto.

---

## 10. Matriz de cobertura

### 10.1 Historias → requisitos del PRD

| Historia | FR | NFR | AC | Fase |
|---|---|---|---|---|
| US-1.1 | FR-INT-001/002/005, FR-BKT-001/002/003 | NFR-CONS-001 | AC-1.1/1.2/1.3 | F1–F2 |
| US-1.2 | FR-INT-007/008 | NFR-PERF-002 | AC-1.6/1.7 | F2 |
| US-1.3 | FR-AUD-001 | NFR-SEC-001 | AC-1.5 | F2 |
| US-1.4 | FR-INT-004/006 | — | AC-1.4 | F2 |
| US-1.5 | FR-INT-003, FR-BKT-008 | NFR-CONS-001 | AC-1.7 | F1–F2 |
| US-1.6 | FR-BKT-004/005/009 | — | AC-1.2 | F6 |
| US-1.7 | FR-BKT-009, FR-DOC-006 | — | AC-6.3 | F6 |
| US-1.8 | FR-INT-010 | NFR-PERF-002 | — | F6 |
| US-2.1 | FR-NAV-002/003 | — | AC-2.1/2.2/2.5 | F2 |
| US-2.2 | FR-NAV-004/006 | NFR-PERF-003/004 | AC-2.3/2.4 | F2 |
| US-2.3 | FR-NAV-005 | — | AC-2.2 | F2 |
| US-2.4 | FR-NAV-010 | — | — | F2 |
| US-2.5 | FR-NAV-008 | — | AC-2.6 | F2 |
| US-2.6 | FR-GRAPH-001/002/003, FR-NAV-001 | — | — | F2 |
| US-2.7 | FR-GRAPH-001/002 | — | — | F6 |
| US-2.8 | FR-NAV-001/006 | NFR-DATA-002 | — | F6 |
| US-3.1 | FR-GEN-003/004/014/018 | NFR-SEC-002 | AC-3.6 | F4 |
| US-3.2 | FR-GEN-005/007/008/010/014 | NFR-SEC-002, NFR-TEST-002 | AC-3.1/3.2/3.3 | F4 |
| US-3.3 | FR-GEN-005/006/024 | NFR-SEC-002 | AC-3.4/3.5 | F4 |
| US-3.4 | FR-GEN-013/021, FR-DOC-004 | — | AC-3.11 | F4, F6 |
| US-3.5 | FR-GEN-008/009 | — | AC-3.2 | F4 |
| US-3.6 | FR-GEN-011/012 | NFR-TEST-002 | AC-3.7 | F4 |
| US-3.7 | FR-GEN-011/020/021, FR-NAV-007 | — | AC-3.9 | F4–F5 |
| US-3.8 | FR-GEN-019/022, FR-DOC-007 | — | — | F4 |
| US-3.9 | FR-GEN-015/018, FR-DOC-010 | NFR-COST-001 | — | F6 |
| US-3.10 | FR-GEN-016/017 | — | AC-3.10 | F4 |
| US-4.1 | FR-UX-001/002/012 | NFR-PERF-006/007 | AC-4.1 | F3 |
| US-4.2 | FR-UX-003, FR-INT-001 | — | AC-4.2 | F3 |
| US-4.3 | FR-UX-002/003, FR-GEN-004 | — | AC-2.4 | F3 |
| US-4.4 | FR-UX-004/016 | — | — | F3 |
| US-4.5 | FR-UX-003, FR-INT-001 | — | — | F3 |
| US-4.6 | FR-UX-011, FR-INT-009 | — | — | F3 |
| US-4.7 | FR-UX-008/010/015 | NFR-REL-001 | AC-4.3 | F3 |
| US-4.8 | FR-UX-009, FR-TEL-003 | NFR-REL-001 | AC-4.4, AC-5.3 | F3 |
| US-4.9 | FR-UX-005, FR-INT-001 | — | AC-1.4 | F3 |
| US-4.10 | FR-UX-013/014 | NFR-A11Y-001 | AC-4.5 | F3 |
| US-4.11 | FR-UX-016 | — | — | F3 |
| US-5.1 | FR-TEL-007, FR-DOC-002 | NFR-PRIV-002 | — | F6 |
| US-5.2 | FR-TEL-007 | NFR-PRIV-002 | — | F6 |
| US-5.3 | FR-TEL-005 | — | — | F3 |
| US-6.1 | FR-DOC-001/002, FR-AUD-001 | NFR-PERF-009 | AC-6.4 | F6 |
| US-6.2 | FR-DOC-003 | NFR-SEC-006 | — | F6 |
| US-6.3 | FR-DOC-004/005, FR-GEN-014 | — | AC-3.11 | F6 |
| US-6.4 | FR-DOC-005, FR-GEN-005 | — | AC-6.2 | F6 |
| US-6.5 | FR-AUTH-003 | NFR-SEC-006 | AC-6.1 | F6 |
| US-6.6 | FR-DOC-009, FR-EVT-004 | — | AC-7.3 | F7 |
| US-6.7 | FR-DOC-010, FR-OBS-002/003 | NFR-COST-001 | — | F7 |
| US-7.1 | FR-AUTH-001..005 | NFR-SEC-001 | AC-7.1 | F7 |
| US-7.2 | FR-PRIV-001/002 | — | AC-7.8 | F7 |
| US-7.3 | FR-PRIV-003/004 | NFR-PRIV-001 | — | F7 |
| US-7.4 | FR-PRIV-005, FR-OBS-004 | — | — | F7 |
| US-7.5 | FR-OBS-001/004, FR-EVT-001 | — | AC-7.2 | F5–F7 |
| US-7.6 | FR-OBS-003, FR-EVT-004 | — | AC-7.3 | F7 |
| US-7.7 | FR-DEP-002/003/004/006 | NFR-REL-003, NFR-IA-001 | AC-7.4/7.5 | F7 |
| US-7.8 | FR-CICD-001/002/003 | NFR-TEST-001..004 | AC-7.6 | F0, F7 |
| US-7.9 | FR-EVT-005/006, FR-NAV-007, FR-UX-012 | NFR-REL-004 | AC-7.7 | F5 |

### 10.2 Cobertura inversa: FR sin historia asociada

| Requisito | Historia(s) | Observación |
|---|---|---|
| FR-INT-009 | US-4.6, US-1.1 (escenario 4) | Cubierto en el escenario de ítem no publicable |
| FR-BKT-006 | US-1.1 | El DDL del trigger se verifica en la suite de la Fase F1, no en una historia de usuario |
| FR-BKT-007 | US-1.1 | Particionamiento verificado por pruebas de esquema de la Fase F1 |
| FR-NAV-009 | US-2.2 | Variedad de ítems dentro de la sesión; cubierto por el criterio de selección |
| FR-TEL-001/002/004/006/008/009/010 | US-4.8, US-5.3 | Contrato de telemetría; cubierto por US-4.8 y el contrato de la API |
| FR-GEN-002 | US-3.1 (escenario 2 y 3) | Structured output y descarte de JSON inválido |
| FR-GEN-023 | US-7.9 | El core no bloquea el event loop durante el trabajo largo |
| FR-PRIV-004 | US-7.3 | Retención y borrado automatizado |
| FR-DEP-001/005 | US-7.7, US-7.8 | Dockerfile, `docker compose` local y CI/CD |
| FR-OBS-005 | US-7.7 | Gestión de secretos fuera del repositorio |
| NFR-DATA-001/002 | US-1.2 | Particionamiento, retención y caché del grafo |

---

## 11. Guion de derivación a features Spec Kit

Para cada feature, copiar el bloque de historias correspondiente y mantener el formato de `spec-template.md`:

| Feature Spec Kit | Historias a copiar | Prioridades |
|---|---|---|
| `001-bkt-mastery-core` | US-1.1, US-1.2, US-1.5, US-1.6 | P1, P1, P2, P2 |
| `002-interaction-and-navigation-api` | US-1.3, US-1.4, US-2.1, US-2.2, US-2.3, US-2.4, US-2.5, US-2.6 | P1, P2, P1, P1, P2, P2, P2, P2 |
| `003-practice-session-frontend` | US-4.1 … US-4.11, US-5.3, US-7.1 | P1 ×3, P2 ×5, P3 ×3 + P3 + P1 |
| `004-item-generation-validation` | US-3.1 … US-3.10 | P1 ×3, P2 ×5, P3 ×2 |
| `005-async-events-and-streaming` | US-3.7, US-7.5, US-7.9 | P2, P2, P2 |
| `006-teacher-admin-console` | US-6.1 … US-6.7, US-1.7, US-1.8, US-2.7, US-2.8, US-5.1, US-5.2 | P1 ×2, P2 ×4, P3 ×7 + P3 + P3 + P3 + P3 + P3 |
| `007-security-observability-deployment` | US-7.2, US-7.3, US-7.4, US-7.6, US-7.7, US-7.8, US-6.6 | P2, P3, P3, P3, P3, P2, P3 |

**Orden de ejecución recomendado**: M0 → M1 → M2 → M3 → M4 → M5 (§9).

---

**Versión:** 1.0.0 · **Fecha:** 2026-10-19 · **Derivado de:** `PRD.md` v1.0.0 · **Siguiente paso:**
`/speckit.specify` para la feature `001-bkt-mastery-core`.
