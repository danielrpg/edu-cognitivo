<!--
Sync Impact Report
==================
Amendment type: Initial ratification (template scaffold -> first adopted version)
Version change: N/A (unfilled template) -> 1.0.0
Bump rationale: First adoption of project-specific principles and governance. Because the
  document had no prior ratified version, this is treated as a MAJOR-style initial adoption
  (1.0.0) rather than an amendment.

Modified / added principles:
  - [PRINCIPLE_1_NAME] -> I. Fuente Única de Verdad del Dominio
  - [PRINCIPLE_2_NAME] -> II. Validación Simbólica Obligatoria (NO NEGOCIABLE)
  - [PRINCIPLE_3_NAME] -> III. Entrada de LLM No Confiable
  - [PRINCIPLE_4_NAME] -> IV. Comunicación Asíncrona y Contratos Versionados
  - [PRINCIPLE_5_NAME] -> V. Testing Obligatorio del Núcleo (BKT y Validador)

Added sections:
  - [SECTION_2_NAME] -> Restricciones Técnicas y de Seguridad
  - [SECTION_3_NAME] -> Flujo de Desarrollo y Puertas de Calidad

Removed sections: none

Deferred TODOs:
  - TODO(RATIFICATION_DATE): No existe fecha de adopción previa; se registra 2026-10-17 como
    fecha de primera ratificación. Ajustar si existiera una aprobación formal anterior.
  - Las reglas del grafo (prerrequisitos, propagación de dominio, criterio de "dominado",
    decaimiento) y la definición de "nodos situados" no están en el material disponible. No se
    codifican como principios; se difieren a /clarify y /specify.

Deferred non-governance intents (see Next Actions in the final summary; NOT executed here):
  - Reescribir el documento de arquitectura corregido (trigger, validador SymPy seguro,
    motor de generación con reflexión real).
  - Diseñar el modelo de datos completo (DDL) del grafo de micro/macronodos con BKT parametrizado.
  - Generar los specs por feature.
-->

# Sistema de Aprendizaje Adaptativo Constitution

## Core Principles

### I. Fuente Única de Verdad del Dominio

Todo cálculo de dominio (*mastery*) vive en una única fuente de verdad: la función BKT
implementada en PostgreSQL. Ningún servicio (Spring Boot, FastAPI, Angular) MUST recalcular,
duplicar ni sobreescribir el estado de dominio de forma paralela; los servicios consumen el
valor derivado, no una estimación propia.

- Los parámetros BKT MUST parametrizarse por micronodo (por ejemplo `p_L0`, `p_T`, `p_S`,
  `p_G`); prohibido usar constantes globales o hardcodeadas que impidan la calibración.
- La precisión numérica del dominio MUST preservarse a lo largo de los pasos de inferencia
  (sin redondeos que acumulen error) y las fórmulas MUST ser seguras ante casos límite
  (por ejemplo, evitar división por cero cuando `p_slip` o `p_guess` alcanzan valores extremos).
- Siguiendo la recomendación del documento de revisión, el BKT puede refactorizarse en su lugar,
  pero el criterio expuesto en el texto y el código MUST coincidir; la discrepancia entre
  documentación e implementación se considera deuda no admitida.

Razón: un único punto de cálculo es la condición para que la adaptación sea auditable,
reproducible y calibrable con datos reales. Duplicar la lógica garantiza divergencia silenciosa.

### II. Validación Simbólica Obligatoria (NO NEGOCIABLE)

Ningún reactivo (*item*) llega al estudiante sin validación simbólica aprobada.

- La validación MUST usar parseo seguro: `parse_expr` con `local_dict`/`global_dict`
  restringidos y `transformations` controladas, o una gramática/AST allowlist. `sympify` con
  `eval` (aun con `evaluate=False`) MUST NOT aceptarse como mecanismo de seguridad.
- El validador MUST ejecutar bajo un *timeout* estricto; una expresión patológica no puede
  consumir CPU indefinidamente.
- El validador MUST soportar variables distintas de la asumida por defecto y comparar contra
  el conjunto completo de soluciones válidas (no solo la primera), para no rechazar respuestas
  correctas alternativas.
- Las excepciones MUST tipificarse de forma específica; capturar genéricamente y ocultar la
  causa MUST NOT ocurrir.

Razón: la corrección matemática es la garantía pedagógica mínima. Un validador frágil o evadible
introduce respuestas erróneas en el flujo de aprendizaje y, con entrada de LLM, abre una vía de
ejecución no confiable.

### III. Entrada de LLM No Confiable

Toda entrada y salida de LLM se trata como no confiable y MUST NOT ejecutarse ni confiarse
directamente.

- La respuesta del modelo MUST validarse contra un esquema/structured output antes de usarse;
  el parseo directo de texto libre (por ejemplo, markdown con fences) MUST NOT darse por válido.
- Los datos del cliente no se confían: la identidad del estudiante MUST derivarse del token
  (JWT/OIDC) en el backend, nunca de un campo enviado por el cliente.
- El acceso a proveedores de LLM MUST realizarse a través de una abstracción (por ejemplo
  `LLMProvider`); mezclar proveedores sin interfaz común MUST NOT ocurrir.
- El parseo matemático de artefactos generados por el modelo MUST permanecer aislado como
  defensa frente a *prompt injection* y ejecución remota de código.

Razón: el LLM es una fuente externa susceptible de manipulación. Tratarla como confiable
convierte una alucinación en un fallo de seguridad o de integridad de datos.

### IV. Comunicación Asíncrona y Contratos Versionados

El trabajo cuya duración esperada supere 1 segundo MUST ejecutarse de forma asíncrona; los
contratos entre componentes MUST estar versionados.

- La generación de reactivos y demás tareas de larga duración MUST NOT bloquear el event loop
  ni el hilo de la petición HTTP; se emiten mediante cola de eventos y se exponen por streaming
  (por ejemplo SSE).
- Las llamadas bloqueantes (red, LLM) MUST NOT invocarse desde contextos `async` sin delegación
  adecuada.
- Los reintentos MUST separarse por causa: red/*rate limiting* usa backoff exponencial; errores
  de validación matemática usan un bucle explícito de reflexión que reinyecta el historial en el
  siguiente prompt. Aplicar backoff a errores de validación MUST NOT ocurrir.
- Los contratos (OpenAPI, AsyncAPI/JSON Schema) MUST estar versionados e incluir `correlationId`,
  idempotencia en consumidores, patrón Outbox para consistencia transaccional y política de DLQ.

Razón: el trabajo costoso y dependiente de terceros es inherentemente lento e inestable; hacerlo
síncrono degrada la disponibilidad y reproduce el problema que la arquitectura busca evitar.

### V. Testing Obligatorio del Núcleo (BKT y Validador)

El núcleo del sistema MUST estar cubierto por pruebas automatizadas antes de considerarse
entregado.

- El *trigger* BKT MUST probarse (por ejemplo, pgTAP o Testcontainers).
- El validador simbólico MUST probarse con *property-based tests*.
- Los contratos entre servicios MUST probarse con *contract tests*.
- El ordenador de la lógica de dominio y el validador de reactivos son componentes críticos:
  un cambio en ellos sin pruebas que lo respalden MUST NOT fusionarse.

Razón: estos dos componentes determinan la corrección pedagógica y son los de mayor riesgo; las
pruebas son la única evidencia objetiva de que sus invariantes se mantienen.

## Restricciones Técnicas y de Seguridad

La arquitectura MUST preservar la separación de responsabilidades y justificar cada componente
con una métrica real.

- **Arquitectura por eventos:** la comunicación para trabajo no trivial MUST ser asíncrona. En el
  MVP MUST usarse un único broker; incorporar múltiples protocolos (eventos + SSE + gRPC) sin
  justificación cuantificada MUST NOT ocurrir. gRPC queda fuera del MVP.
- **Código limpio:** principios SOLID y Clean Architecture. Lógica de dominio, infraestructura y
  capa HTTP MUST residir en módulos separados; un único archivo que las mezcle MUST NOT aceptarse.
- **Generación de reactivos:** se favorece la pre-generación y caché (banco de ítems) sobre la
  generación en línea; ante fallo tras N intentos MUST existir un *fallback* a banco pregenerado
  y un estado de error visible al estudiante.
- **Coste y cuota del LLM:** MUST existir límites de coste/cuota, y la calidad pedagógica se
  evalúa por separado de la corrección matemática (SymPy no valida ambigüedad ni didáctica).
- **Modelo de datos:** MUST existir el modelo completo (`students`, `macronodes`, `micronodes`,
  `micronode_edges`, `items`/`questions`, `interaction_logs`, `student_mastery`, `bkt_params`).
  Con particionado de `interaction_logs`, las claves primarias e índices únicos MUST incluir la
  clave de partición.
- **Seguridad y privacidad:** autenticación/autorización JWT/OIDC con roles
  (estudiante/docente/admin). La API key del proveedor de LLM MUST gestionarse como secreto.
  La protección de datos de menores (privacidad y consentimiento) MUST ser requisito explícito.
- **Despliegue:** los manifiestos MUST incluir probes, `Service`, `Secret`, `NetworkPolicy`,
  `PodDisruptionBudget`, Ingress y Dockerfiles. El autoescalado MUST basarse en la métrica que
  refleje la carga real (profundidad de cola para trabajo dominado por espera de red).

## Flujo de Desarrollo y Puertas de Calidad

El desarrollo avanza por fases con entregables verificables; cada fase habilita la siguiente.

- **Orden de implementación de referencia:**
  1. Esquema PostgreSQL + trigger BKT + tests.
  2. Registro de interacción y consulta de dominio (sin LLM).
  3. Sesión de práctica y telemetría básica de extremo a extremo.
  4. Generación + validación simbólica (síncrona al inicio, aislada por su riesgo).
  5. Migración de la generación a asíncrona (cola + SSE) con patrón Outbox.
  6. Despliegue, autoescalado y observabilidad.
- **Puertas de calidad:** toda PR/revisión MUST verificar el cumplimiento de esta constitución.
  La complejidad añadida MUST justificarse con una métrica real.
- **Observabilidad:** MUST instrumentarse con trazas distribuidas (OpenTelemetry) propagando
  `traceId` a través de la cola, junto con dashboards y alertas. MUST definirse SLOs medibles
  (por ejemplo p95 de entrega del primer ítem y tasa de reactivos descartados).
- **Entrega:** MUST existir CI/CD, gestión de secretos y separación de entornos.

## Governance

Esta constitución prevalece sobre cualquier otra práctica o convención del proyecto.

- **Enmiendas:** cualquier cambio MUST documentarse, aprobarse por el responsable del proyecto y
  acompañarse de un plan de migración cuando afecte a artefactos existentes.
- **Versionado:** la versión sigue versionado semántico (MAJOR.MINOR.PATCH). MAJOR para cambios
  incompatibles o redefiniciones de principios; MINOR para principios o secciones nuevas y
  ampliaciones materiales; PATCH para aclaraciones y correcciones no semánticas.
- **Cumplimiento:** toda PR/revisión MUST verificar el cumplimiento de los principios. Las
  desviaciones MUST justificarse y documentarse; las violaciones no justificadas MUST bloquear la
  fusión.
- **Resolución de conflictos:** ante ambigüedad de interpretación, el responsable del proyecto
  resuelve y la decisión se registra como aclaración (PATCH) o enmienda.

**Version**: 1.0.0 | **Ratified**: 2026-10-17 | **Last Amended**: 2026-10-17
