# Backend

Servicios backend del Sistema de Aprendizaje Adaptativo.

## Estructura

| Directorio | Descripción |
|---|---|
| `core-api/` | API principal en Spring Boot 3 + Java 21. Gestiona autenticación, sesiones, interacciones, navegación adaptativa, dominio BKT y panel docente/admin. |
| `generation-service/` | Servicio de generación en FastAPI + Python 3.12. Genera reactivos con LLM, valida simbólicamente con SymPy, implementa reflexión y pre-generación. |

## Responsabilidades

### core-api
- Autenticación JWT/OIDC con derivación obligatoria de identidad
- Registro atómico de interacciones con actualización de `student_mastery`
- Navegación adaptativa según grafo de prerrequisitos (política GRA-07)
- Consulta de dominio con decaimiento aplicado en lectura
- Panel docente y administración
- Publicación de eventos via patrón Outbox
- Integración con RabbitMQ

### generation-service
- Abstracción LLMProvider (Gemini/Ollama)
- Structured output con validación JSON Schema
- Validación simbólica segura con SymPy (`parse_expr`, allowlist, timeout)
- Bucle de reflexión con historial para errores de validación
- Pre-generación por lotes
- Detección de duplicados
- Trazabilidad de costes y auditoría

## Comunicación

- **REST**: Core API expone endpoints para frontend
- **Eventos**: Comunicación asíncrona via RabbitMQ (QuestionGenerationRequested, QuestionGenerated)
- **SSE**: Notificaciones en tiempo real al frontend para estado de generación

## Desarrollo

Ver README de cada submódulo para instrucciones específicas de desarrollo.
