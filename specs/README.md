# Specs (Spec Kit)

Especificaciones generadas por el flujo de Spec Kit: `specify → clarify → plan → tasks → analyze → implement`.

## Estructura por Feature

Cada feature tendrá su propio directorio:

```
specs/
├── 001-interacciones-bkt/      # F1: Registro de interacciones y estimación BKT
├── 002-grafo-navegacion/       # F2: Grafo de conocimiento y navegación adaptativa
├── 003-generacion-validacion/  # F3: Generación y validación de reactivos
├── 004-sesion-practica/        # F4: Sesión de práctica y UX
├── 005-telemetria/             # F5: Telemetría del estudiante
├── 006-panel-docente/          # F6: Panel docente y administración
└── 007-confianza-operacion/    # F7: Confianza, asincronía, despliegue
```

## Archivos Generados

| Archivo | Descripción |
|---|---|
| `spec.md` | Requerimientos funcionales, historias de usuario, criterios de aceptación |
| `plan.md` | Plan técnico con arquitectura, decisiones, riesgos |
| `data-model.md` | Modelo de datos detallado |
| `contracts/` | OpenAPI/AsyncAPI específicos de la feature |
| `research.md` | Investigación y decisiones técnicas |
| `tasks.md` | Lista de tareas desagregadas para implementación |
| `checklist.md` | Checklist de verificación |

## Flujo Recomendado

1. **/constitution** - Definir principios no negociables (ya referenciado)
2. **/specify** - Crear spec por feature (solo qué, sin cómo)
3. **/clarify** - Resolver ambigüedades
4. **/plan** - Plan técnico (cómo con stack definido)
5. **/tasks** - Generar tareas ejecutables
6. **/analyze** - Verificar consistencia
7. **/implement** - Implementar feature por feature

## Orden de Implementación (MVP)

Seguir el orden sugerido en la revisión:
1. Esquema + Trigger BKT
2. Core API: interacciones y dominio (sin LLM)
3. Angular: sesión de práctica con ítems sembrados
4. Generation Service: generación + validación SymPy
5. Asincronía (RabbitMQ + outbox + SSE)
6. Operación (K8s, observabilidad)
