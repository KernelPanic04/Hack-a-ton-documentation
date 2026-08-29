# Design tokens P0

Estado: **congelado para H1**. Fuente ejecutable: `src/index.css` del frontend,
dentro de `@theme`. Los componentes consumen tokens; no introducen colores,
espaciados o sombras ad hoc.

## Espaciado semántico

| Token | Valor | Uso |
|---|---:|---|
| `--spacing-ui-1` | `0.25rem` | Separación interna mínima |
| `--spacing-ui-2` | `0.5rem` | Icono ↔ texto |
| `--spacing-ui-3` | `0.75rem` | Controles compactos |
| `--spacing-ui-4` | `1rem` | Padding móvil / gap de card |
| `--spacing-ui-6` | `1.5rem` | Padding de card |
| `--spacing-ui-8` | `2rem` | Gap de sección |
| `--spacing-ui-12` | `3rem` | Separación entre regiones de página |

La página usa inline padding `ui-4 / ui-6 / ui-8` por breakpoint. Una card
usa `ui-4` en móvil y `ui-6` desde `sm`. No se agregan escalas paralelas.

## Jerarquía tipográfica

| Rol | Size / line-height | Peso/fuente |
|---|---|---|
| display | `2rem / 1.15` | Alfa Slab One 400 |
| section | `1.5rem / 1.25` | Alfa Slab One 400 |
| card title | `1rem / 1.4` | Roboto 700 |
| body | `0.9375rem / 1.6` | Roboto 400 |
| label | `0.8125rem / 1.4` | Roboto 600 |
| caption | `0.75rem / 1.4` | Roboto 500 |

## Énfasis obligatorio

| Emphasis | Fondo | Borde | Texto/acento | Semántica |
|---|---|---|---|---|
| `normal` | `#eef6f6` | `#a8dadc` | `#1d3557` | Información estable o activa |
| `warning` | `#fff6e8` | `#f6c177` | `#8a3d00` | Requiere atención, aún recuperable |
| `critical` | `#fff1f0` | `#fda29b` | `#8f1d18` | Fallo, riesgo alto o acción bloqueada |

El significado nunca depende solo del color: icono/label/status acompañan
warning y critical. Texto normal mantiene contraste oscuro sobre superficie.

## Forma, elevación y movimiento

- Control radius: `0.75rem`.
- Card radius: `1.5rem`.
- Card shadow: la sombra existente `--shadow-card`.
- Transición UI: `200ms`.
- `prefers-reduced-motion: reduce` elimina animación no esencial.
- Focus visible: outline de 3px, nunca se elimina sin reemplazo.
