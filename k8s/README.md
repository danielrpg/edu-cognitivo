# Kubernetes

Manifiestos para despliegue en Kubernetes.

## Estructura Sugerida

| Archivo/Directorio | Descripción |
|---|---|
| `namespace.yaml` | Namespace del proyecto |
| `configmap.yaml` | Configuración no sensible |
| `secrets.yaml` | Secrets (template - usar gestor externo en prod) |
| `postgres/` | StatefulSet o referencia a managed Postgres |
| `rabbitmq/` | Deployment/StatefulSet para RabbitMQ |
| `core-api/` | Deployment, Service, Ingress, HPA |
| `generation-service/` | Deployment, Service, HPA/KEDA (recomendado) |
| `frontend/` | Deployment, Service, Ingress |
| `network-policies/` | NetworkPolicy para aislamiento |
| `pdb/` | PodDisruptionBudget |
| `monitoring/` | ServiceMonitor, PrometheusRules (opcional) |

## Requerimientos NFR

| Recurso | Propósito |
|---|---|
| **Probes** | Liveness, Readiness y Startup probes para todos los servicios |
| **Resources** | `requests`/`limits` definidos correctamente |
| **HPA** | HorizontalPodAutoscaler para core-api y generation-service |
| **KEDA** | Scaler por profundidad de cola RabbitMQ (mejor que CPU para workload I/O bound) |
| **NetworkPolicy** | Restringir tráfico entre servicios |
| **PodDisruptionBudget** | Asegurar disponibilidad mínima |
| **Ingress** | Con TLS (cert-manager) |
| **Secrets** | API keys de LLM en Kubernetes Secrets o External Secrets Operator |

## Notas Importantes

- **HPA vs CPU**: Para generation-service, el cuello de botella suele ser espera de red al LLM. Escalar por `rabbitmq_queue_length` con KEDA es más efectivo.
- **Virtual Threads**: Core API con Java 21 puede manejar más concurrencia con menos pods.
- **Tracing**: Propagar `traceId` via headers y mensajes RabbitMQ para OpenTelemetry.
- **Graceful shutdown**: Configurar `terminationGracePeriodSeconds` y shutdown hooks.
