# Docker

Dockerfiles para containerización de todos los servicios.

## Estructura

| Archivo | Descripción |
|---|---|
| `Dockerfile.core-api` | Multi-stage build para Spring Boot 3 + Java 21 (JRE slim). |
| `Dockerfile.generation-service` | Multi-stage build para FastAPI + Python 3.12 (distroless o slim). |
| `Dockerfile.frontend` | Multi-stage build para Angular (nginx:alpine). |

## Buenas Prácticas

- **Multi-stage builds**: Reducir tamaño final de imagen
- **Non-root user**: Ejecutar como usuario no privilegiado
- **Health checks**: Incluir HEALTHCHECK
- **Layer caching**: Optimizar orden de COPY para aprovechar cache
- **Distroless/slim**: Usar imágenes mínimas en producción
- **Secrets**: NUNCA copiar `.env` o claves al build

## Desarrollo

Usar `docker-compose.dev.yml` en raíz para levantar todo el stack localmente:

```bash
docker-compose -f docker-compose.dev.yml up --build
```

Servicios locales:
- PostgreSQL 16 (puerto 5432)
- RabbitMQ + Management UI (puertos 5672, 15672)
- Core API (puerto 8080)
- Generation Service (puerto 8000)
- Frontend (puerto 4200)
- Ollama (opcional, puerto 11434)
