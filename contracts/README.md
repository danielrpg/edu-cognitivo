# Contratos

Contratos versionados entre servicios para garantizar compatibilidad.

## Estructura

| Directorio | Descripción |
|---|---|
| `openapi/` | Especificaciones OpenAPI 3.x para REST APIs (core-api). |
| `asyncapi/` | Especificaciones AsyncAPI para eventos asíncronos (RabbitMQ). |

## Contratos Requeridos

### OpenAPI (REST)
- `core-api.yaml`: Todos los endpoints de Core API
  - Autenticación/Autorización
  - Interacciones (`POST /api/v1/interactions`)
  - Navegación (`GET /api/v1/navigation/next`)
  - Dominio (`GET /api/v1/domain/me`)
  - Sesiones de práctica
  - Panel docente/admin
- Versionado en ruta: `/api/v1/...`
- Schemas compartidos con validación

### AsyncAPI (Eventos)
Eventos publicados por Core API, consumidos por Generation Service:

| Evento | Publisher | Consumer | Descripción |
|---|---|---|---|
| `QuestionGenerationRequested` | core-api | generation-service | Solicitud de generación para un micronodo |
| `QuestionGenerated` | generation-service | core-api | Ítem generado y validado, listo para publicar |
| `QuestionGenerationFailed` | generation-service | core-api | Generación fallida tras N intentos |

Campos obligatorios en todos los eventos:
- `eventId` (UUID)
- `eventType` (string)
- `correlationId` (UUID) - trazabilidad end-to-end
- `traceId` (para OpenTelemetry)
- `timestamp` (ISO 8601)
- `version` (schema version)

## Principios

- **Versionado**: Cambios breaking requieren nueva versión
- **Backward compatible**: Preferir adición de campos opcionales
- **Idempotencia**: Consumidores deben usar `inbox_messages` (consumer + message_id)
- **DLQ**: Mensajes fallidos van a Dead Letter Queue tras reintentos
- **Correlación**: Propagar `correlationId` y `traceId` en toda la cadena

## Generación de Código

Estos contratos pueden usarse para generar clientes/servidores con herramientas como:
- OpenAPI Generator
- AsyncAPI Generator
