# Sistema de Aprendizaje Adaptativo Basado en Grafos de Micronodos y Macronodos Situados

Este repositorio contiene la implementación del sistema adaptativo para el aprendizaje de matemáticas, basado en un grafo de conocimiento con **micronodos** (conceptos atómicos) y **macronodos** (unidades temáticas situadas). El motor utiliza **Bayesian Knowledge Tracing (BKT)** para estimar el dominio del estudiante, con cálculo realizado exclusivamente en PostgreSQL.

## Estructura del Proyecto

| Directorio | Descripción |
|---|---|
| `backend/` | Servicios backend (Core API y Generation Service). Ver `backend/README.md`. |
| `db/` | Migraciones, semillas y esquemas de base de datos PostgreSQL. Ver `db/README.md`. |
| `contracts/` | Contratos versionados entre servicios (OpenAPI para REST, AsyncAPI para eventos). Ver `contracts/README.md`. |
| `docker/` | Dockerfiles para todos los servicios. Ver `docker/README.md`. |
| `k8s/` | Manifiestos de Kubernetes (despliegue, servicios, HPA/KEDA, políticas). Ver `k8s/README.md`. |
| `specs/` | Especificaciones generadas por Spec Kit por feature (F1-F7). Ver `specs/README.md`. |
| `.specify/` | Plantillas, memoria de constitución y configuración de Spec Kit. |
| `frontend/` | Aplicación Angular 18+ (ya existente). Ver `frontend/README.md`. |
| `PRD.md` | Product Requirements Document con requerimientos funcionales y no funcionales. |
| `HISTORIAS-DE-USUARIO.md` | Catálogo completo de historias de usuario con escenarios Given/When/Then. |
| `Revisión del plan de implementación y guía para Spec Kit.md` | Guía para implementar con correcciones identificadas. |

## Arquitectura

```
Angular (SPA)  ──HTTP/JSON──►  Spring Boot Core API  ──JDBC/JPA──►  PostgreSQL
                                       │                              ▲
                                       │ (Outbox)                     │ trigger BKT
                                       ▼                              │
                                     RabbitMQ  ◄─────────────────────────┘
                                       │
                                       ▼
                                FastAPI Generation Service ──► LLM (Gemini/Ollama)
                                       │
                                       ▼
                                  SymPy Validator (sandboxed)
```

## Principios Fundamentales

- **Dominio en una sola fuente de verdad**: Todo cálculo BKT vive en PostgreSQL (función + trigger).
- **Validación simbólica obligatoria**: Ningún reactivo llega al estudiante sin validación con SymPy.
- **Entrada LLM no confiable**: Todo contenido generado debe validarse y sanearse.
- **Identidad desde token**: `student_id` siempre se deriva del JWT, nunca del cliente.
- **Asincronía para trabajo largo**: Generación > 1s se maneja con colas (RabbitMQ) y SSE.
- **Tests obligatorios**: Críticos para trigger BKT y validador simbólico.

## Tecnologías

| Componente | Tecnología | Versión |
|---|---|---|
| Core API | Spring Boot + Java | 3.x + 21 |
| Generation Service | FastAPI + Python | + Python 3.12 |
| Base de Datos | PostgreSQL | 16 |
| Broker | RabbitMQ | Latest |
| Frontend | Angular | 18+ |
| Validación Matemática | SymPy | Latest |
| Tracing | OpenTelemetry | Latest |

## Inicio Rápido

### Desarrollo Local
```bash
# Levantar stack completo
docker-compose -f docker-compose.dev.yml up --build
```

### Migraciones de BD
```bash
# Ejecutar migraciones (ver db/README.md)
```

### Pruebas
```bash
# Core API
cd backend/core-api && ./mvnw test

# Generation Service
cd backend/generation-service && pytest

# Frontend (ya configurado)
cd frontend && npm test
```

## Documentación Relacionada

- [PRD](./PRD.md) - Requerimientos completos
- [Historias de Usuario](./HISTORIAS-DE-USUARIO.md) - 56 historias con criterios de aceptación
- [Guía Spec Kit](./Revisión%20del%20plan%20de%20implementación%20y%20guía%20para%20Spec%20Kit.md) - Flujo de implementación
