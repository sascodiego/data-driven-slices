# DDS Tool CLI: Guard + Factory — Registro de Decisiones

> [!NOTE]
> **Estado: decisión de diseño asentada, implementación pendiente.** Este documento registra las decisiones de diseño tomadas para el tool CLI auxiliar del método DDS. No hay código aún; este registro existe para que la implementación futura tenga una fuente de decisiones explícita y auditable. Sigue siendo parte del [prospecto](DDS_Metodo_Trabajo_IA.md#prospecto-medible-hipótesis-a-validar): el tool codifica criterios no validados y evoluciona con ellos.

---

## 1. Contexto

Vamos a iniciar un proyecto nuevo desarrollado con agentes de IA que usará **DDS** como criterio arquitectónico. Se identificaron tres necesidades:

1. **Verificación de fronteras**: el compilador es la aduana en Strict, pero en Lite/Standard la frontera es convención — y la convención sin verificador es solo un deseo.
2. **Asistencia sistemática al agente**: prompts y skills generados de manera determinista a partir del criterio, no escritos a mano cada vez.
3. **Métricas**: datos observables para validar (o falsar) las hipótesis del prospecto.

La respuesta a las tres es **un solo tool CLI en Go** con dos subsistemas sobre un núcleo común.

## 2. Forma del tool

```text
                    ┌─────────────────────────────┐
                    │   criterio (datos, YAML)    │   <- única fuente de verdad operativa
                    │   perfiles, reglas,         │
                    │   checklists, contratos     │
                    └──────────┬──────────────────┘
                 ┌─────────────┼──────────────────┐
                 ▼             ▼                  ▼
        ┌────────────┐  ┌─────────────┐  ┌──────────────────┐
        │  guard     │  │  factory    │  │  config default  │
        │  before/   │  │  skills,    │  │  (.dds/config)   │
        │  after     │  │  prompts,   │  │                  │
        │  verifica  │  │  AGENTS.md  │  │                  │
        └────────────┘  └─────────────┘  └──────────────────┘
```

El principio que unifica los dos subsistemas: **generar y verificar usan la misma definición de frontera**. `dds prompt implement --slice Billing` produce un prompt con los paths permitidos del slice (los mismos globs que valida el guard), de modo que el agente recibe contexto acotado y después el guard verifica que lo respetó. No pueden divergir porque leen la misma config.

## 3. Decisiones cerradas

| # | Decisión | Alcance | Racional |
| :-: | :--- | :--- | :--- |
| D1 | El tool vive **en este repo**, como Go module anidado en `tools/dds-guard/` (con `go.mod` propio; la raíz sigue siendo un repo de documentación). | Repositorio | El criterio y su enforcement versionan juntos: si una hipótesis se degrada o se remueve del prospecto, el tool cambia en el mismo commit. No puede quedar un binario publicado ejecutando una versión vieja del método. |
| D2 | El workspace del proyecto objetivo es **`.dds/` oculto**: `config.yaml` (mapa slice → globs, rutas de contratos, paths de `/core`), snapshots del guard (gitignored) y `metrics.csv`. No se usan directorios `/odd` ni equivalentes en el proyecto objetivo. | Proyecto objetivo | El workspace es del método DDS, separado del tracking de tareas del agente. |
| D3 | El proyecto objetivo es **C#**. | Proyecto objetivo | Define el formato de los ejemplos y, en v2, el análisis de imports (Roslyn). |
| D4 | **Métricas solo observables**: blast radius (`git show --name-only`), commits por tarea, runtime de tests, retrabajos, regresiones (post-hoc), promociones de perfil. **Sin** tokens ni costo operativo, **sin** archivos explorados. Todo dato manual se etiqueta como tal; nada se estima ni se inventa. | Métricas | Solo se publica lo respaldable. Los tokens no se obtienen de forma confiable; mejor ausencia de dato que número inflado. |
| D5 | El criterio operativo vive como **datos estructurados (YAML)** citando el doc de `docs/es/` del que proviene cada regla. La prosa queda para humanos; los datos, para el tool. No se generan los docs desde YAML. | Núcleo | Evita prompts hardcodeados en código y mantiene auditoría criterio ↔ dato. |
| D6 | Guard v1 **a nivel archivo**: cruces por path, hash de contratos, paths no mapeados, `/core` tocado. **Sin AST — definitivo, no solo v1**: la verificación de imports vive en el lenguaje objetivo como tests de arquitectura (ver [Charter de Slice](DDS_Slice_Charter.md)); en Strict el compilador ya cubre ese hueco. | Guard | Alcance honesto de lo detectable de forma determinista; re-encuadre de la vieja idea de v2 con Roslyn-in-Go. |

### Guard v1: contrato operativo

```text
dds guard before --slice X   # HEAD, warning si working tree sucio, hashes de contracts.XX
dds guard after              # clasifica cambios: in-slice / contrato mutado /
                             # path no mapeado / core tocado; exit 1 en violación;
                             # agrega fila a .dds/metrics.csv
```

| Detección | Severidad |
| :--- | :--- |
| Archivo modificado fuera del slice objetivo | Violación (exit 1) |
| Contrato (`contracts.XX`) mutado | Warning fuerte: exige paso explícito de evolución de contrato |
| Path nuevo que ningún slice reclama | Warning: mapear o es basura |
| `/core` tocado | Warning: solo infraestructura |

### Factory: patrón sin ceremonia

Factory en Go idiomático: registry de artifacts + constructores + `text/template` con `go:embed`. Cada artifact = template + datos de contexto. Si el diseño crece más que eso, está mal diseñado.

## 4. Decisiones abiertas (a resolver antes de implementar)

| # | Decisión | Estado | Recomendación |
| :-: | :--- | :--- | :--- |
| A1 | Orden de implementación: guard primero o factory primero | Abierta | **Guard primero**: es más chico e independiente; el núcleo de config es común. |
| A2 | Idioma de los artifacts generados (skills, prompts) | **Decidido: inglés** | Inglés confirmado al redactar el prototipo de `AGENTS.md` ([template seed](../../tools/dds-guard/templates/AGENTS.md)). |
| A3 | Nombre del binario: `dds` vs `ddsctl` | Abierta | `ddsctl` evita colisiones en PATH; `dds` es más corto. |
| A4 | Set de skills v1 | Abierta | Propuesta: `dds-slice` (crear slice con perfil), `dds-task` (tarea con guard before/after), `dds-review` (verificar proporcionalidad al cerrar). |

## 5. Instrucción al agente

El `AGENTS.md` del proyecto objetivo (repo-level, nunca global) contiene **instrucciones DDS puras**: criterio, layout de slices, perfiles, reglas de frontera, Zero-Mocks, convenciones. **Sin referencias al CLI mientras el tool no exista** — instruir verificación inexistente es prometer lo que no hay. Cuando el guard se implemente, la factory agregará la línea operativa ("toda tarea de código corre `dds guard before` al abrir y `dds guard after` al cerrar") y la disciplina de métricas como una sección generada, no escrita a mano. ODD no se instruye (es parte del ecosistema) y **no se documenta integración ODD↔DDS hasta que el criterio esté probado y medido**; lo que se instruye es DDS como criterio arquitectónico.

> **Prototipo:** el template vive en [`tools/dds-guard/templates/AGENTS.md`](../../tools/dds-guard/templates/AGENTS.md) (inglés, C#/.NET 8+, DDS puro: sin guard, sin métricas, sin referencias al CLI). **v1** = instalación base + charter + arch tests + matriz de decisiones + workflows A/B, en sincronía con lo desplegado al agente del proyecto satélite (update #2). Cuando la factory se implemente, ese template se regenera desde los datos del criterio y esta versión manual queda de referencia.

## 6. Consecuencias

- Este repo pasa a ser multi-lenguaje: `docs/es/` = criterio (prosa), `tools/dds-guard/` = mecanismo (Go). La frontera se mantiene explícita.
- Distribución en el proyecto objetivo: binarios por plataforma en GitHub Releases (mientras no existan, `go run ./tools/dds-guard` desde una clona de este repo).
- Los skills y prompts generados quedan marcados con la versión del criterio; regenerarlos tras un cambio del criterio produce un diff visible.
