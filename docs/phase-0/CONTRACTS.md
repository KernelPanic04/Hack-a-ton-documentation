# Contratos v1 e identificadores

Estado: **congelado para H1**. Responsable: Lane D. Versión literal de todos
los contratos: `"1"`.

La autoridad ejecutable es el modelo Pydantic del backend. El archivo
TypeScript es un espejo escrito a mano para no introducir generación de tipos en
la ruta crítica. Esta página define la semántica que ambos deben conservar.

## Convención de IDs

Todos los IDs son strings opacos, en minúsculas, sin espacios, con un prefijo
obligatorio. No se infiere información de negocio desde un ID.

| Campo | Prefijo | Lo crea | Estabilidad / ejemplo |
|---|---|---|---|
| `workflowId` | `wf_` | backend/fixture | Estable entre versiones: `wf_logistics_main` |
| `stepId` | `step_` | definición de workflow | Estable entre runs y versiones si el paso conserva identidad: `step_track_vessel` |
| `runId` | `run_` | backend | Una ejecución: `run_550e8400-e29b-41d4-a716-446655440000` |
| `decisionId` | `dec_` | backend | Una solicitud de decisión: `dec_550e8400-e29b-41d4-a716-446655440000` |
| `actionId` | `act_` | policy/workflow | Semántico y estable: `act_find_alternative` |
| `eventId` | `evt_` | **solo backend** | Un evento append-only: `evt_550e8400-e29b-41d4-a716-446655440000` |
| `idempotencyKey` | `idem_` | cliente | Un intento lógico de acción: `idem_550e8400-e29b-41d4-a716-446655440000` |
| `UINode.id` | `ui_` | composer | Estable para el mismo concepto entre snapshots: `ui_run_summary` |

Formato aceptado después del prefijo: `[a-z0-9][a-z0-9_-]{0,127}`. Para
instancias se recomienda UUID v4; para workflow, step, action y UI se recomienda
un slug semántico. Un reordenamiento visual no cambia un `UINode.id`.

`ActionEvent` nunca contiene `eventId`: el servidor lo asigna al aceptar o
rechazar la acción y la convierte en `RunEvent`. Reintentar una acción conserva
el mismo `idempotencyKey`; una acción nueva crea otro.

## Tipos compartidos

### `RunEvent`

Registro append-only, creado por backend. `sequence` es monotónico por run y
empieza en 1. `timestamp` es RFC 3339 con zona horaria.

| Campo | Tipo | Regla |
|---|---|---|
| `schemaVersion` | `"1"` | Obligatorio |
| `eventId` | `EventId` | Servidor |
| `runId` | `RunId` | Debe coincidir con el run |
| `workflowId` | `WorkflowId` | Estable entre versiones |
| `workflowVersion` | integer | `>= 1` |
| `sequence` | integer | `>= 1`, monotónico por run |
| `stateVersion` | integer | `>= 0`; solo aumenta al cambiar el estado |
| `type` | string | Nombre del evento |
| `stepId` | `StepId?` | Paso relacionado, si aplica |
| `payload` | JSON | Datos del evento, nunca objetos ejecutables |
| `timestamp` | datetime | RFC 3339, timezone-aware |

### `RunProjection`

Snapshot A → C. Es la vista materializada del run, no un evento.

| Campo | Tipo | Regla |
|---|---|---|
| `schemaVersion` | `"1"` | Obligatorio |
| `runId`, `workflowId` | IDs | Según la tabla anterior |
| `workflowVersion` | integer | `>= 1` |
| `stateVersion` | integer | `>= 0`; control de stale actions/UI |
| `lastSequence` | integer | Último `RunEvent.sequence`, `>= 0` |
| `status` | enum | `created`, `running`, `paused`, `completed`, `failed` |
| `currentStep` | `RunStepProjection?` | Ausente antes de iniciar o al terminar |
| `operation` | JSON object | Estado de negocio necesario para sintetizar UI |
| `recentEvents` | `RunEvent[]` | Máximo 50, orden ascendente |
| `pendingDecision` | `DecisionRequest?` | Solo la decisión actualmente pendiente |
| `availableActions` | `ActionDefinition[]` | Vacío sin decisión; no vacío si hay una pendiente |

### `UISpec`

Snapshot C → B. La raíz siempre es `page`; solo contiene los nueve tipos del
registry. No admite HTML, JSX, JavaScript, CSS ni URLs arbitrarias.

| Campo | Tipo | Regla |
|---|---|---|
| `schemaVersion` | `"1"` | Obligatorio |
| `runId`, `workflowId` | IDs | Deben corresponder a la proyección |
| `workflowVersion`, `stateVersion` | integer | Copia exacta de la proyección usada |
| `generatedBy` | enum | `deterministic`, `llm`, `fallback` |
| `reason` | string | Obligatorio, 1–500 caracteres; explica la estructura elegida |
| `layout` | `PageNode` | Árbol declarativo con IDs de nodo únicos |
| `allowedActions` | `ActionDefinition[]` | Subconjunto por ID de `RunProjection.availableActions` |

Una mejora determinista → LLM conserva `runId`, `workflowVersion` y
`stateVersion`; cambia `generatedBy`, `reason` y, cuando aporta valor, el árbol.

### `ActionEvent`

Comando B → A enviado por WebSocket. No es parte del event log hasta que el
backend lo valida y crea un `RunEvent`.

| Campo | Tipo | Regla |
|---|---|---|
| `schemaVersion` | `"1"` | Obligatorio |
| `idempotencyKey` | `IdempotencyKey` | Creado por cliente; se reutiliza en retries |
| `runId` | `RunId` | Run visible en el cliente |
| `workflowVersion`, `stateVersion` | integer | Valores visibles al hacer click |
| `decisionId`, `actionId` | IDs | Deben seguir pendientes/autorizados |
| `payload` | JSON | Validado contra `ActionDefinition.payloadSchema` |
| `timestamp` | datetime | Hora cliente, RFC 3339 con zona |

El backend valida token, acceso al run, versión, decisión pendiente, acción,
payload, policy e idempotencia. La hora cliente nunca decide orden ni validez.

## Envelope WebSocket

Todos los mensajes usan el mismo envelope:

```text
schemaVersion, type, runId, sequence, timestamp, payload
```

En mensajes de servidor, `sequence` es el `RunEvent.sequence` incluido en el
payload. En `ACTION_SUBMITTED`, es la última secuencia de servidor observada por
el cliente. El backend no confía en ese valor: reconcilia con `stateVersion`.

| Dirección | `type` | Payload |
|---|---|---|
| servidor → cliente | `RUN_STARTED`, `STEP_STARTED`, `STEP_COMPLETED`, `STATE_UPDATED`, `DECISION_REQUIRED`, `RUN_PAUSED`, `RUN_RESUMED`, `RUN_COMPLETED` | `{ event, projection }` |
| servidor → cliente | `UI_UPDATED` | `{ event, projection, uiSpec }` |
| servidor → cliente | `ACTION_ACCEPTED` | `{ event, projection, idempotencyKey, decisionId, actionId }` |
| servidor → cliente | `ACTION_REJECTED` | `{ event, code, message, idempotencyKey, currentStateVersion }` |
| servidor → cliente | `ERROR` | `{ event, code, message, retryable }` |
| cliente → servidor | `ACTION_SUBMITTED` | `ActionEvent` |

Los 12 tipos servidor → cliente son los mensajes P0 que debe manejar el
reducer. `ACTION_SUBMITTED` es un comando, no un estado del reducer.

## Invariantes de integración

- Campos desconocidos se rechazan en backend (`extra = forbid`).
- Los IDs de acciones y nodos son únicos dentro de su colección/árbol.
- Una `pendingDecision` exige al menos una `availableAction`; sin decisión la
  lista debe estar vacía.
- Los eventos recientes pertenecen al mismo run/workflow y no superan
  `lastSequence`.
- Props rotas o tipos desconocidos nunca producen pantalla blanca: el renderer
  aísla el nodo y muestra `GenericStepCard`/estado de error.
- La autenticación del WebSocket usa `DEMO_TOKEN`; el token no viaja dentro de
  `ActionEvent` ni se persiste en el event log.
