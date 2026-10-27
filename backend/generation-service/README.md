# Generation Service (FastAPI + Python 3.12)

Servicio aislado para generación y validación de reactivos matemáticos.

## Estructura

| Directorio | Descripción |
|---|---|
| `app/api/` | Routers FastAPI (generate, batch, health, internal). |
| `app/core/` | Configuración, logging estructurado, tracing OpenTelemetry, settings. |
| `app/schemas/` | Schemas Pydantic y JSON Schema estricto para `QuestionPayload`. |
| `app/services/llm/` | `LLMProvider` abstracto + implementaciones `GeminiProvider`, `OllamaProvider`. |
| `app/services/reflection/` | Lógica de reflexión con historial conversacional para corrección tras error de validación. |
| `app/validators/sympy/` | Validador simbólico seguro: `parse_expr`, allowlist, timeout, tipificación de errores. |
| `app/workers/` | Consumidores RabbitMQ, ejecución en ThreadPool/ProcessPool (evitar bloquear event loop). |
| `tests/` | Tests de seguridad, validación simbólica, reflexión y casos borde. |

## Flujo de Generación

1. **Solicitud**: Recibe petición con `micronode_id`, contexto situado, parámetros
2. **Prompt**: Construye prompt con objetivo, contexto GRA-06, ejemplos, restricciones
3. **LLM**: Llama vía `LLMProvider` con `structured_output` / validación JSON Schema
4. **Parseo**: Limpia markdown fences, valida contra esquema estricto
5. **Validación Simbólica**: `SymPyValidator` verifica solución correcta vs ecuación
6. **Reflexión**: Si falla validación matemática → reintenta con historial (sin backoff)
7. **Publicación**: Si pasa → guarda como `validated`/`draft`, emite evento

## Seguridad Crítica

- **NUNCA usar `sympify` con `evaluate=True`**: Usar `parse_expr` con `local_dict`/`global_dict` restringidos
- **Timeout obligatorio**: `signal.alarm` o `multiprocessing` con timeout (2s por defecto)
- **Allowlist**: Solo funciones/operadores matemáticos permitidos
- **Sin `eval/exec`**: Parsing con AST o gramática controlada
- **Sanitización**: Limpiar contenido antes de persistir
- **Tipificación**: Excepciones específicas (`SympyValidationError`, `SympyTimeoutError`, etc.)

## Reintentos

| Tipo de Error | Estrategia |
|---|---|
| Red / 429 / 5xx | Backoff exponencial + jitter, respeta `Retry-After` |
| Validación matemática | Reflexión inmediata con historial, sin backoff temporal |
| Schema inválido | Reflexión o descarte según contexto |

## Notas

- **No bloquear event loop**: Llamadas LLM y cómputo SymPy deben ir a workers/hilos
- **Determinismo en tests**: Permitir inyectar stub, fijar temperature=0, semilla
- **Trazabilidad**: Registrar `correlation_id`, `traceId`, `llm_model`, `cost`
