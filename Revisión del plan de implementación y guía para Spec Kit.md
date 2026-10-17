# Revisión del plan de implementación: inconsistencias, riesgos y guía para Spec Kit

*Sistema de Aprendizaje Adaptativo Basado en Grafos de Micronodos y Macronodos Situados*

El documento revisado tiene buena dirección arquitectónica (EDA, upsert, batching, HPA), pero **el código y el texto se contradicen en varios puntos** y faltan artefactos para que sea implementable. Este informe se divide en tres partes: inconsistencias, lo que falta y cómo usarlo con Spec Kit.

## 1. Inconsistencias y errores (texto vs. código y bugs)

### Críticos (rompen el sistema o su corrección)

**1.1 La arquitectura propuesta no coincide con el código.** El texto recomienda eventos (RabbitMQ/Kafka + SSE) para evitar bloquear Tomcat, pero el Paso 3 expone `POST /generate-item` **síncrono**, y `generate_and_audit` es una función síncrona dentro de un endpoint `async def`. Esto bloquea el event loop de FastAPI (tenacity usa `time.sleep` y la llamada al LLM es bloqueante). Resultado: exactamente el problema que el texto dice evitar.

**1.2 El "self-reflection" no funciona.** En `generate_and_audit`, el `error_context` se crea y se lanza dentro de una excepción, pero **nunca se reinyecta en el prompt** del siguiente intento. Cada reintento llama al LLM con el mismo `prompt` (`prompt_context` siempre es `""`). Además, el backoff exponencial de 2-10 s de tenacity es innecesario para errores de validación (no es rate limiting): se pierde tiempo sin ganar nada. Hay que separar:

- Reintentos de **red/rate limit**: tenacity con backoff.
- Reintentos de **validación matemática**: bucle explícito con historial de conversación.

**1.3 El trigger BKT tiene varios problemas.**

- `v_p_transit` y los demás parámetros son constantes globales; BKT estándar los parametriza **por micronodo** (p\_L0, p\_T, p\_S, p\_G). Hardcodearlos impide calibrar.
- El texto dice que se refactoriza con `INSERT ... ON CONFLICT DO UPDATE`, pero el código usa `DO NOTHING` seguido de `SELECT ... FOR UPDATE`. Texto y código no coinciden (el código es en realidad el patrón más razonable aquí, pero el texto debe corregirse).
- **Precisión numérica:** `v_current_p NUMERIC(4,3)`, el inicial `0.100` y el clamp `0.010`–`1.000` redondean en cada paso y acumulan error. Con `p=1` y respuesta incorrecta, la fórmula de Bayes puede dar división por cero si `p_slip` se parametriza a 0.
- No se especifica si el trigger es `AFTER INSERT` o `BEFORE INSERT`: la función hace `RETURN NEW` pero el `CREATE TRIGGER` **no aparece**. Falta el DDL del trigger.
- Con particionado de `interaction_logs`, las PK e índices únicos deben incluir la clave de partición; el documento no lo menciona.

**1.4 SymPy: la "seguridad" propuesta es falsa.** `sympify(..., evaluate=False)` **no previene RCE**: `sympify` usa `eval` internamente, y `evaluate=False` solo evita la simplificación. La mitigación correcta es `parse_expr` con `local_dict`/`global_dict` restringidos y `transformations` controladas, o validar con una gramática/AST allowlist antes de parsear. Además, `payload.correct_solution` se pasa a `sp.sympify(...)` **sin ningún aislamiento**, y el input viene de un LLM (no confiable, susceptible a prompt injection).

**1.5 Validación matemática frágil.**

- `payload.equation_str.split("=")` falla con más de un `=` o con `==`, `<=`, etc.
- `sp.symbols('x')` está hardcodeado: ecuaciones con otra variable devuelven soluciones erróneas.
- `solutions[0]` ignora ecuaciones con múltiples soluciones (cuadráticas): una respuesta correcta alternativa se marcaría como incorrecta.
- No hay *timeout* en `solve`/`simplify`: una expresión patológica consume CPU indefinidamente (DoS); el HPA solo escala, no limita.
- `except Exception` captura todo y lo convierte en `SympyValidationError`, escondiendo bugs.

### Moderados

**1.6** `QuestionPayload` no contiene lo que el reactivo necesita: enunciado, distractores, opciones, nivel de dificultad, pista, explicación. Solo tiene `equation_str` y `correct_solution`. Faltan campos que el frontend y el BKT requieren.

**1.7** `create_item(micronode_id: str)` en un `POST` recibe el parámetro como query param, sin schema de request, sin autenticación y sin idempotency key.

**1.8** `response.text` se parsea con `model_validate_json` directamente: el LLM suele devolver markdown con fences. Falta structured output (`response_schema` de Gemini) o limpieza. Además, "Gemini (Ollama)" mezcla dos proveedores distintos sin abstracción; no hay interfaz `LLMProvider`.

**1.9 Angular / telemetría.**

- `runOutsideAngular` solo tiene sentido para **listeners de eventos DOM** o timers; aquí envuelve un `Subject.next()`, que no dispara change detection de todos modos. El patrón no cumple su propósito. Además, `bufferTime` crea su timer en el constructor (dentro de la zona) y `HttpClient.subscribe` re-entra a la zona. Con Signals/zoneless no hace falta.
- **Pérdida de datos:** `bufferTime(5000)` pierde eventos al cerrar la pestaña. Falta `visibilitychange` + `navigator.sendBeacon`.
- Sin reintento ni cola persistente si el POST falla (solo `console.error`).
- `studentId` en el payload del cliente: **no debe confiarse**; el backend debe tomarlo del JWT.
- "Milisegundos de resolución": `Date.now()` no es de alta precisión; debería usarse `performance.now()` más un timestamp de servidor.
- El texto menciona NgRx SignalStore y "Backtracking", pero no hay código ni definición de qué es.

**1.10 Kubernetes.**

- HPA al 70% de CPU con `requests: 250m` escala sobre \~175m, pero el límite es 1000m: la relación request/limit de 1:4 hace que el HPA reaccione tarde respecto a la saturación real. Con un workload dominado por **espera de red al LLM**, la CPU no es la métrica correcta; conviene escalar por **profundidad de cola** (KEDA con RabbitMQ).
- Faltan: probes de FastAPI, `Service`, `Secret` para la API key de Gemini, `NetworkPolicy`, `PodDisruptionBudget`, Ingress y Dockerfiles. El texto describe Spring Boot, Angular y Postgres, pero **solo hay YAML del motor cognitivo**.
- `image: tu-registro/...:v1` sin pipeline de CI/CD.

### Menores / de consistencia

- Texto: "Spring Boot … alta concurrencia" y "Tomcat thread starvation". Con Java 21 se resuelve parcialmente con **virtual threads**, alternativa que no se evalúa.
- Se propone gRPC para "validación rápida de un grafo", pero no hay `.proto` ni caso concreto. Introducir tres protocolos (eventos + SSE + gRPC) en un MVP es sobreingeniería sin justificación.
- Se piden "principios SOLID y Clean Architecture", pero el código entregado es un único archivo con lógica, infraestructura y HTTP mezclados.
- El documento se refiere a "los 5 pilares que solicitaste" y a una especificación que **no está adjunta**. Todo lo relativo al grafo (prerrequisitos, macronodos, propagación de dominio) está ausente.

## 2. Lo que falta para poder implementarlo

### Dominio y datos

1. Modelo de dominio completo: `students`, `macronodes`, `micronodes`, `micronode_edges` (prerrequisitos), `items/questions`, `interaction_logs`, `student_mastery`, `bkt_params`. Solo se infieren dos tablas.
2. Reglas del grafo: cómo el dominio de un micronodo afecta a su macronodo; criterio de "dominado" (umbral, p. ej. ≥ 0.95); lógica de selección del siguiente ítem (política adaptativa); manejo de olvido/decaimiento.
3. Calibración de parámetros BKT (valores iniciales y cómo se ajustan con datos reales).
4. Estrategia de "situados": qué contexto define un macronodo situado y cómo se usa en el prompt.

### Contratos

5. OpenAPI para Spring Boot y esquemas de eventos (AsyncAPI o JSON Schema) para `QuestionGenerationRequested` / `QuestionGenerated`, con versionado, `correlationId` y política de DLQ.
6. Esquema JSON completo del reactivo con structured output.
7. Autenticación/autorización (JWT/OIDC, roles estudiante/docente/admin) y protección de datos de menores (privacidad, consentimiento).

### Fiabilidad

8. Patrón **Outbox** para publicar eventos de forma consistente con la transacción de Postgres; idempotencia en consumidores.
9. Qué ocurre cuando falla la generación tras N intentos (fallback a banco de ítems pregenerados, estado de error visible al estudiante).
10. **Pre-generación y caché de ítems** (casi siempre mejor que generar en línea con 5–15 s de latencia).
11. Límites de coste y cuota del LLM, y evaluación de calidad pedagógica (SymPy valida corrección matemática, no calidad didáctica ni ambigüedad del enunciado).

### Ingeniería

12. Estructura de repositorios/módulos, Dockerfiles y `docker-compose` para desarrollo local (Postgres, RabbitMQ, ambos servicios, Ollama).
13. Estrategia de pruebas: tests del trigger (pgTAP o Testcontainers), property-based tests de la validación simbólica, contract tests entre servicios y tests del frontend.
14. CI/CD, gestión de secretos y entornos.
15. Observabilidad concreta: dashboards, alertas y tracing distribuido (OpenTelemetry con propagación de `traceId` a través de la cola).
16. Criterios de aceptación y SLOs (p. ej. p95 de entrega del primer ítem, tasa de reactivos descartados).

## 3. Cómo usarlo con Spec Kit para avanzar rápido

Spec Kit funciona con el flujo `constitution → specify → clarify → plan → tasks → analyze → implement`. El error más común es pasarle este documento tal cual como `/specify`: mezcla el *qué* con el *cómo* y contiene errores que Spec Kit propagaría a las tareas.

**Paso 0: corregir primero el documento** con la sección 1 (al menos 1.1 a 1.5), o tratarlo solo como borrador de `plan`, no de `spec`.

**Paso 1: `/constitution`.** Fija principios no negociables derivados del documento:

- Todo cálculo de dominio vive en una única fuente de verdad (la función BKT en Postgres).
- Ningún reactivo llega al estudiante sin validación simbólica aprobada.
- Toda entrada de LLM se trata como no confiable.
- Comunicación asíncrona para trabajo de más de 1 s; contratos versionados.
- Tests obligatorios para el trigger y el validador.

**Paso 2: `/specify`** (solo requisitos de usuario, **sin tecnología**). Divídelo en features pequeñas, no en un solo spec gigante:

1. Registro de interacciones y estimación de dominio por micronodo.
2. Generación y validación de reactivos.
3. Navegación por el grafo y selección del siguiente ejercicio.
4. Telemetría del estudiante.
5. Panel docente (si aplica).

**Paso 3: `/clarify`.** Aquí se resuelven las preguntas abiertas de la sección 2 (umbral de dominio, política adaptativa, qué pasa ante fallo del LLM, privacidad).

**Paso 4: `/plan`.** Aquí sí va la tecnología (Spring Boot, FastAPI, Postgres, Angular, RabbitMQ, K8s). Pega la arquitectura **ya corregida**. Spec Kit generará `data-model.md`, `contracts/` y `research.md`: pídele explícitamente que incluya OpenAPI, AsyncAPI y el DDL del trigger.

**Paso 5: `/tasks` → `/analyze` → `/implement`.** Ejecuta feature por feature. Orden sugerido para llegar rápido a algo funcional:

| Fase | Entregable | Por qué primero |
| --- | --- | --- |
| 1 | Esquema Postgres + trigger BKT + tests | Núcleo; todo depende de él |
| 2 | Spring Boot: registrar interacción y consultar dominio (sin LLM) | Valor demostrable con banco de ítems manual |
| 3 | Angular: sesión de práctica + telemetría básica | Ciclo de extremo a extremo |
| 4 | FastAPI: generación + validación SymPy (síncrono al inicio) | Aislar el riesgo más alto |
| 5 | Pasar generación a asíncrona (cola + SSE) y outbox | Solo cuando el flujo básico ya funcione |
| 6 | K8s, HPA/KEDA, observabilidad | Al final |

**Para ir rápido:** en el MVP elimina gRPC, usa un banco de ítems pregenerado con generación en segundo plano y un solo broker. Añade complejidad cuando una métrica real la justifique.

## Siguientes pasos posibles

- Reescribir el documento corregido (arquitectura + trigger, validador SymPy seguro y motor de generación con reflexión real).
- Generar el borrador de `constitution.md` y los prompts `/specify` por feature para Spec Kit.
- Diseñar el modelo de datos completo (DDL) del grafo de micro/macronodos con BKT parametrizado.

Si se dispone de la especificación original del proyecto, conviene compartirla: varias de las lagunas (reglas del grafo y "nodos situados") dependen de ella.
