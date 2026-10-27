# Base de Datos (PostgreSQL 16)

Migraciones, esquemas y datos de prueba para el sistema adaptativo.

## Estructura

| Directorio | Descripción |
|---|---|
| `migrations/` | Migraciones versionadas (Flyway recomendado). Orden cronológico V001, V002, ... |
| `seeds/` | Datos semilla para desarrollo (cursos, macronodos, micronodos, ítems de prueba). |

## Migraciones Requeridas (Orden Crítico)

| Migración | Propósito | Prioridad |
|---|---|---|
| `V001__init_schema.sql` | Crear todas las entidades: `students`, `courses`, `subjects`, `macronodes`, `micronodes`, `micronode_edges`, `bkt_params`, `items`, `interaction_logs`, `student_mastery`, `student_mastery_macro`, `practice_sessions`, `telemetry_events`, `outbox_events`, `generation_jobs`, `inbox_messages`. | P0 |
| `V002__bkt_trigger.sql` | Función BKT parametrizada por micronodo + `CREATE TRIGGER AFTER INSERT ON interaction_logs`. Debe ser idempotente, usar `NUMERIC(12,8)`, fórmulas seguras ante división por cero. | P0 |
| `V003__partitions.sql` | Particionado mensual de `interaction_logs`. Crear particiones futuras automáticamente. Incluir clave de partición en PK/índices únicos. | P0 |
| `V004__constraints_checks.sql` | `CHECK` para `bkt_params`: `p_T,p_S,p_G >= 1e-6`, `p_L0,p_S,p_G,p_T BETWEEN 0 AND 1`. Validación DAG en `micronode_edges`. | P0 |
| `V005__outbox_indexes.sql` | Índices para patrón outbox e inbox_messages. | P1 |

## Modelo Crítico

### Fuente Única de Verdad
- `student_mastery`: Calculada **únicamente** por trigger BKT. Nunca escrita directamente por aplicación.
- `interaction_logs`: Particionada por rango mensual. PK compuesta debe incluir `server_ts` o fecha de partición.

### Parámetros BKT (por micronodo)
- `bkt_params`: `p_L0`, `p_T`, `p_S`, `p_G` - **no usar constantes hardcodeadas**
- Fórmulas con epsilon 1e-6 para evitar degeneración
- Precisión `NUMERIC(12,8)`, redondeo solo al persistir

### Grafo
- `micronode_edges`: DAG validado (sin ciclos)
- Umbral dominado: `mastery >= 0.95` (configurable)

## Comandos Útiles

```sql
-- Verificar trigger BKT existe
SELECT * FROM pg_trigger WHERE tgname LIKE '%bkt%';

-- Ver particiones
SELECT schemaname, tablename FROM pg_tables WHERE tablename LIKE 'interaction_logs%';

-- Verificar función
SELECT proname, prosrc FROM pg_proc WHERE proname LIKE '%bkt%';
```

## Notas

- Usar `NUMERIC` para todos los cálculos de probabilidad
- FK con particionado: considerar restricciones cuidadosamente
- Rotación de particiones según política de retención (P-05)
