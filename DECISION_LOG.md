# Decision log · Kernel Panic

Las decisiones aceptadas son vinculantes para el roadmap. Una modificación
incluye fecha, decisión, alternativas, razón, responsable y consecuencias.

## 2026-08-29 · D-001 · `UISpec` declarativa como ruta principal

- **Estado:** aceptada.
- **Decisión:** el runtime renderiza un árbol declarativo limitado al registry.
- **Alternativas:** React/JS generado; HTML arbitrario; pantallas completas fijas.
- **Razón:** permite validación, seguridad, coherencia visual y composición real.
- **Responsable:** Lane D.
- **Consecuencias:** no se ejecuta código del LLM; un renderer de código solo
  podría existir como bonus aislado después del core.

## 2026-08-29 · D-002 · Contrato v1 y autoridad Pydantic

- **Estado:** aceptada y congelada para H1.
- **Decisión:** `schemaVersion = "1"`; Pydantic es la autoridad ejecutable y
  TypeScript es un espejo manual revisado en el mismo PR.
- **Alternativas:** TypeScript como autoridad; generación bidireccional; tipos
  sin una fuente explícita.
- **Razón:** el backend ya usa Pydantic y puede generar JSON Schema sin añadir
  infraestructura a la ruta crítica.
- **Responsable:** Lane D, por delegación explícita del usuario.
- **Consecuencias:** todo cambio actualiza backend, frontend, docs y pruebas;
  después de H8 exige aprobación explícita de D.

## 2026-08-29 · D-003 · IDs prefijados y `eventId` solo servidor

- **Estado:** aceptada.
- **Decisión:** se usan los prefijos `wf_`, `step_`, `run_`, `dec_`, `act_`,
  `evt_`, `idem_` y `ui_`. `ActionEvent` no contiene `eventId`.
- **Alternativas:** UUID desnudo; IDs sin convención; `eventId` creado por cliente.
- **Razón:** reduce confusiones entre entidades y mantiene al event log bajo
  autoridad del backend.
- **Responsable:** Lane D.
- **Consecuencias:** retries reutilizan `idempotencyKey`; aceptar o rechazar crea
  un `RunEvent` nuevo en servidor.

## 2026-08-29 · D-004 · Registry P0 de nueve componentes

- **Estado:** aceptada y congelada para H1.
- **Decisión:** `page`, `section`, `metric`, `alert`, `timeline`, `keyValue`,
  `compare`, `decisionPanel`, `step`.
- **Alternativas:** registry de 10 del brief; componentes por pantalla; primitive
  nueva para cada dato logístico.
- **Razón:** es el conjunto de nueve que el roadmap introduce de H1 a H13 y
  cubre estructura, estado, datos, comparación, decisión y fallback.
- **Responsable:** Lane B propone; Lane D congela. Validación de Lane C pendiente
  de firma, sin bloquear la implementación delegada por el usuario.
- **Consecuencias:** `routeMap`, `dataTable` y `documentChecklist` salen de P0;
  solo vuelven mediante una decisión aprobada.

## 2026-08-29 · D-005 · Síntesis progresiva con fallback determinista

- **Estado:** aceptada.
- **Decisión:** el composer determinista publica primero; el LLM puede mejorar
  jerarquía, selección permitida y contenido dentro del schema. Se envían
  snapshots completos de `UISpec`, no patches.
- **Alternativas:** esperar siempre al LLM; LLM solo para labels; JSON Patch.
- **Razón:** primera UI en menos de 50 ms, mejora visible y demo resistente.
- **Responsable:** Lane D.
- **Consecuencias:** timeout 5 s, un retry y fallback; `generatedBy` y `reason`
  prueban la ruta usada.

## 2026-08-29 · D-006 · WebSocket nativo con snapshot/polling

- **Estado:** aceptada.
- **Decisión:** WebSocket nativo para el loop vivo; al reconectar se obtiene el
  snapshot y, si WS falla, se activa polling.
- **Alternativas:** SSE; polling como ruta primaria; broker externo.
- **Razón:** el mismo canal admite updates y acciones sin nueva infraestructura.
- **Responsable:** Lane D.
- **Consecuencias:** 12 mensajes servidor P0, `ACTION_SUBMITTED` como comando y
  `DEMO_TOKEN` en el handshake.

## 2026-08-29 · D-007 · Event log append-only y estado versionado

- **Estado:** aceptada.
- **Decisión:** toda transición/aceptación/rechazo crea `RunEvent`; el reducer
  produce `RunProjection`. `stateVersion` detecta UI y decisiones stale.
- **Alternativas:** mutación sin log; usar el snapshot como evento.
- **Razón:** replay, depuración, auditoría y evidencia para la defensa.
- **Responsable:** Lane D.
- **Consecuencias:** `sequence` es monotónico por run y el export JSON es P0.

## 2026-08-29 · D-008 · Mantener monolito y stacks existentes

- **Estado:** aceptada.
- **Decisión:** React/Vite y FastAPI se extienden como runtime + monolito modular;
  workflow engine pequeño propio.
- **Alternativas:** Next.js; microservicios; LangGraph; Redis/Kafka.
- **Razón:** protege las ~60 horas útiles y el walking skeleton.
- **Responsable:** Lane D.
- **Consecuencias:** infraestructura adicional es P2 y no entra antes del core.

## 2026-08-29 · D-009 · Frontend state y validación

- **Estado:** aceptada.
- **Decisión:** `useReducer` + Context para estado; JSON Schema generado por
  Pydantic y AJV como validador cuando se implemente el socket/renderer.
- **Alternativas:** Zustand; Zod duplicando modelos; aceptar JSON sin validar.
- **Razón:** evita otro store y alinea la validación runtime con la autoridad.
- **Responsable:** Lane B/D.
- **Consecuencias:** no se mezclan AJV y Zod; error boundary por nodo sigue siendo
  obligatorio aun con validación previa.

## 2026-08-29 · D-010 · Trial-by-fire y efectos externos

- **Estado:** aceptada.
- **Decisión:** editor mínimo visible crea una versión y paso nuevo; las acciones
  P0 operan contra provider local/mock, sin side effects externos reales.
- **Alternativas:** YAML oculto; editor drag-and-drop; notificaciones reales.
- **Razón:** prueba generalidad sin comprometer tiempo ni seguridad.
- **Responsable:** Lane D.
- **Consecuencias:** el paso desconocido usa `step`/`compare`; el event log prueba
  que no hubo rebuild o edición de React.

## 2026-08-29 · D-011 · `dev` como base de código y `main` directo para documentación

- **Estado:** aceptada.
- **Decisión:** backend y frontend parten siempre de `dev`, crean una rama nueva
  antes de modificar archivos y vuelven a integrar mediante PR o merge hacia
  `dev`. La documentación se modifica directamente en `main` después de
  actualizarla por fast-forward. La promoción de código `dev` → `main` queda
  reservada para gates o autorización explícita de Lane D.
- **Alternativas:** trabajar directamente en `dev`; usar `main` como base diaria
  en código; crear ramas también para cada ajuste documental.
- **Razón:** evita repetir trabajo sobre arquitecturas antiguas, reduce
  divergencias entre colaboradores y mantiene una ruta de integración común sin
  añadir fricción innecesaria al repositorio documental.
- **Responsable:** Lane D, por instrucción explícita del usuario.
- **Consecuencias:** cada handoff de backend/frontend debe identificar rama base
  `dev`, rama de trabajo y merge objetivo; frontend establece su rama `dev` desde
  el `main` aprobado si aún no existe; no se permiten pushes forzados para
  reconciliar trabajo compartido.

## Kill criteria vigentes

| Gate | Si falla | Decisión obligatoria |
|---|---|---|
| H3 | Click no completa WS → renderer → action → log | Polling y cerrar skeleton en H4 |
| H8 | No hay golden path determinista de cinco pasos | Cortar composer LLM |
| H13 | No cierra loop humano/inspector/rechazo | Cortar editor visual; POST + JSON |
| H17 | Paso inventado no se ejecuta/renderiza | Mover trial-by-fire al video |
| H20 | Fallbacks no están probados | Recortar guion; freeze sin features |

## Firmas H1

| Lane | Estado | Evidencia |
|---|---|---|
| A | Pendiente | Revisar `RunProjection`, `RunEvent` e IDs con backend |
| B | Aprobada por el usuario | Registry, props y tokens implementados |
| C | Pendiente | Revisar `UISpec`, acciones y compatibilidad structured outputs |
| D | Aprobada por el usuario | Contratos, protocolo y decisiones congelados |
