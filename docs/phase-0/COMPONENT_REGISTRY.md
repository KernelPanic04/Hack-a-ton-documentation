# Component registry P0 · 9 primitivas

Estado: **congelado para H1**. Los nombres son case-sensitive. No se añade una
décima primitiva sin entrada en `DECISION_LOG.md` aprobada por Lane D.

Todos los objetos rechazan props desconocidas. `emphasis` usa exactamente
`normal | warning | critical`.

| `type` | Props exactas | Hijos | Propósito |
|---|---|---|---|
| `page` | `title: string`, `subtitle?: string`, `eyebrow?: string` | 1+ nodos | Raíz única de una `UISpec` |
| `section` | `title?: string`, `description?: string`, `columns?: 1\|2\|3`, `emphasis?: Emphasis` | 0+ nodos | Agrupación y estructura; no contiene lógica de negocio |
| `metric` | `label: string`, `value: string\|number`, `supportingText?: string`, `trend?: up\|down\|flat`, `emphasis?: Emphasis` | No | KPI o dato destacado |
| `alert` | `title: string`, `message: string`, `emphasis: Emphasis` | No | Hallazgo que requiere atención visual |
| `timeline` | `title?: string`, `items: TimelineItem[1+]` | No | Estado de pasos del workflow |
| `keyValue` | `title?: string`, `items: KeyValueItem[1+]`, `columns?: 1\|2` | No | Datos genéricos de operación/documento |
| `compare` | `title: string`, `leftLabel: string`, `rightLabel: string`, `rows: CompareRow[1+]` | No | Comparación genérica, incluido el paso inventado |
| `decisionPanel` | `decisionId`, `title`, `message?`, `actions[1+]`, `status?`, `errorMessage?`, `emphasis?` | No | Human-in-the-loop; las acciones solo referencian IDs permitidos |
| `step` | `stepId`, `title`, `objective?`, `status`, `summary?`, `verdict?`, `emphasis?` | No | Fallback `GenericStepCard` para cualquier tipo de paso |

## Objetos anidados

### `TimelineItem`

`id: StepId`, `title: string`, `status: pending | active | completed | attention | failed`,
`detail?: string`, `timestamp?: RFC3339 datetime`.

### `KeyValueItem`

`key: string`, `label: string`, `value: string | number | boolean`,
`emphasis?: Emphasis`.

### `CompareRow`

`key: string`, `label: string`, `before: string | number | boolean | null`,
`after: string | number | boolean | null`,
`outcome: same | changed | improved | worse | attention`.

### `DecisionAction`

`actionId: ActionId`, `label: string`,
`style?: primary | secondary | danger`, `requiresConfirmation?: boolean`.

### `DecisionPanel.status`

`idle | submitting | accepted | rejected`. En estado `rejected`,
`errorMessage` es obligatorio.

## Reglas del renderer

- Solo `page` y `section` aceptan `children`.
- La raíz es siempre `page`; un `page` no puede anidarse.
- Un ID de nodo no se repite en el árbol.
- Un `decisionPanel.actions[].actionId` debe existir tanto en
  `UISpec.allowedActions` como en `RunProjection.availableActions`.
- Un tipo de step desconocido se representa con `step`; un tipo de componente
  desconocido o props inválidas se aísla mediante error boundary por nodo.
- `routeMap`, `dataTable` y `documentChecklist` pertenecían al brief anterior y
  quedan fuera de P0. Se representan con `timeline`, `keyValue`, `compare` o
  `step` hasta que exista una necesidad real aprobada.
