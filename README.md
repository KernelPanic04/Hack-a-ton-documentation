# Kernel Panic · NextWave 2026 Challenge 03

Repositorio documental y fuente compartida del equipo para **The Interface That
Builds Itself**.

La fuente operativa principal es [`AGENTS.md`](./AGENTS.md). Las decisiones
aceptadas se registran en [`DECISION_LOG.md`](./DECISION_LOG.md); los documentos
anteriores son contexto y no sustituyen esas dos fuentes.

## Phase 0 · contrato congelado

| Entregable | Fuente compartida | Implementación |
|---|---|---|
| 0.1 Contratos e IDs | [`docs/phase-0/CONTRACTS.md`](./docs/phase-0/CONTRACTS.md) | Backend `app/schemas/contracts.py`; frontend `src/runtime/contracts.ts` |
| 0.2 Registry de 9 componentes | [`docs/phase-0/COMPONENT_REGISTRY.md`](./docs/phase-0/COMPONENT_REGISTRY.md) | Props incluidas en los contratos v1 |
| 0.4 Design tokens | [`docs/phase-0/DESIGN_TOKENS.md`](./docs/phase-0/DESIGN_TOKENS.md) | Frontend `src/index.css` |
| 0.6 Entorno reproducible | [`docs/phase-0/ENVIRONMENT.md`](./docs/phase-0/ENVIRONMENT.md) | `.env.example` y Docker Compose de ambos repos |
| 0.7 Decision log | [`DECISION_LOG.md`](./DECISION_LOG.md) | Decisiones aceptadas y kill criteria |

Versión congelada: `schemaVersion = "1"`.

## Regla de cambio

Hasta H8, un cambio de contrato exige actualizar en el mismo PR:

1. `DECISION_LOG.md`, con aprobación de Lane D.
2. El modelo Pydantic del backend, autoridad ejecutable.
3. El espejo TypeScript escrito a mano.
4. La documentación de Phase 0 y las pruebas afectadas.

Después de H8, los contratos no cambian sin aprobación explícita de Lane D.
