# Casos de Uso: Registro de Discusión de la Matriz de Decisiones

> [!NOTE]
> **Estado: documento de TRABAJO — registro vivo, no criterio asentado.** Este documento registra la discusión caso por caso para no perderla. Cada caso resuelto **gradúa** a la matriz y workflows definitivos ([Workflows Operativos](DDS_Workflows.md)); este registro queda como trazabilidad de por qué se decidió cada cosa.

---

## 1. Corrección conceptual (asentada en la discusión): slice ≠ programa

**Un slice es una capacidad dentro de un programa — nunca el programa.** `contracts.Billing` es la cara pública *interna* del slice: lo que consumen otros slices y la composición. Puede respaldar endpoints, pero la API REST es el programa (host), no el slice.

```text
        MUNDO EXTERNO (consumidores, otras apps)
                    │
        ┌───────────▼───────────┐
        │   HOST / composition  │   <- el programa: endpoints, versionado público,
        │   (NO es un slice)    │      auth, middleware. Mapea superficie → slices.
        └───────────┬───────────┘
                    │ consume
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
 contracts.Billing contracts.Pricing ...   <- cara interna de cada slice
```

**Consecuencia clave:** el versionado externo, los consumidores de afuera y las ventanas de deprecación son problemas **del host**, no del slice. El slice solo le debe verdad a su charter y a sus contratos internos. El host es la frontera anticorrupción del programa.

**Pendiente de asentar al resolver el cluster del host (§4.2):** el concepto **host / composition root** como pieza del método (no es slice: no tiene capability propia; no es `/Core`: no es primitiva técnica) y el nodo candidato **D-8**: *"¿El cambio alcanza la superficie externa del programa?"* → si sí, sub-workflow del host (aditivo/ruptura/deprecación), separado del evento de contrato interno.

## 2. Catálogo de casos

| # | Caso | Grupo | Estado | Nota |
| :-: | :--- | :--- | :--- | :--- |
| C1 | Promoción de perfil (D-6 dispara, falta workflow de ejecución) | Estructura | 🔲 Pendiente | Cierra el entregable #1 del roadmap |
| C2 | Democión / fusión de slices | Estructura | 🔲 Pendiente | El método no tiene camino de vuelta |
| C3 | Muerte de slice / deprecación | Estructura | 🔲 Pendiente | **Reformulado por la corrección**: deprecación de dos capas (contrato interno + host retira endpoints) |
| C4 | ~~Contrato con consumidores externos~~ → **Propagación al exterior**: evento de contrato interno que altera la superficie del programa | Host | 🔲 Pendiente | **Reformulado**: evento de dos capas; el slice no versiona para el mundo — el host sí |
| C5 | Feature flag / cambio gradual | Host | 🔲 Pendiente | **Reformulado**: flag es dato de delivery/host, nunca del engine; charter describe comportamiento objetivo |
| C6 | Dato compartido transversal (`TenantId`, `Locale`) | Datos | 🔲 Pendiente | Routing interno; `TenantId` ≈ infraestructura (candidato legítimo a `/Core`) |
| C7 | Cambio de tipo transversal (`Money`: precisión/moneda) | Datos | 🔲 Pendiente | **Enriquecido**: la frontera del host aísla al mundo (DTOs externos); el ripple interno sigue |
| C8 | Bug fix: ¿la verdad es el código o el charter? | Corrección | ✅ Resuelto | Triage contra la pila de verdad (implementación/promesa/definición); RED primero; regla dura anti "arreglé el test" |
| C9 | Conflicto charter ↔ código descubierto (drift) | Corrección | ✅ Resuelto | Drift = sello/consistencia en rojo; clasificar 9a/9b/9c; consultar por defecto, autonomía solo en 9b trivial |
| C10 | Baseline rota (suite roja al empezar) | Corrección | 🔲 Pendiente | Regla de baseline limpia para el agente |
| C11 | Cambio no funcional (perf, layout AoS/SoA) | No-comportamiento | 🔲 Pendiente | Corazón del posture DoD sin workflow; el cierre verifica con benchmarks, no asserts |
| C12 | Refactor puro sin cambio de comportamiento | No-comportamiento | 🔲 Pendiente | ¿Qué exige el cierre sin RED→GREEN? |
| C13 | Cambio en `/Core` | Arquitectura | 🔲 Pendiente | Blast radius global; justificación obligatoria |
| C14 | Decisión mayor de arquitectura (DB, service bus) | Arquitectura | 🔲 Pendiente | Escalación al nivel host, no a un slice |
| C15 | Migración de schema de persistencia | Arquitectura | 🔲 Pendiente | Orden migration/contracts/engine por slice |

## 3. Orden de discusión acordado

1. **C8 + C9** — fuente de verdad: código vs. charter (principio que falta del diseño anti-pudrición).
2. **C4 + C3 + C5** — cluster del host (afianza el modelo corregido de §1).
3. **C6 + C7** — routing de datos transversales (la pregunta real de `/Core`).
4. **C12 + C10** — supervivencia diaria del agente.
5. **C1** — promoción (cierra el entregable #1 del roadmap).
6. C2, C11, C13, C14, C15 — según aparezcan en el proyecto real.

## 4. Resoluciones (se llenan al discutir; cada resolución graduada se refleja en [Workflows](DDS_Workflows.md))

### 4.1 C8 + C9 — Fuente de verdad: código vs. charter — ✅ RESUELTO

**Resolución (graduada a [Charter v2](DDS_Slice_Charter.md) y [Workflows](DDS_Workflows.md)):**

1. **Pila de verdad**: charter-data → contract → test → code; cambios top-down explícitos, nunca bottom-up silenciosos.
2. **Charter como datos** (front-matter) + decision log con porqués por ID; justificaciones fuera del charter.
3. **Tríada de definición**: `SLICE.md` + `[Slice].slice.test.cs` + `slice.shax` (sello del par: sha256 de hashes). Detección de drift instantánea por estado, sin tool.
4. **`consumes`**: referencias cross-slice declaradas vs. reales — detección determinística de creep sin escaneo de vocabulario.
5. **Protocolo P0–P3**: compuerta de impacto (definición/promesa/implementación); nivel 1 no toca la tríada (fricción cero); nivel 3 rota sello + log.
6. **Triage de bug**: compuerta P0 aplicada; regla dura anti-edición de aserciones.
7. **Agente ante drift**: consulta por defecto; autonomía solo en 9b trivial.

**Decisiones de diseño descartadas en la discusión** (trazabilidad): escaneo de vocabulario (heurístico, ruidoso); hashes mutuos md↔test (sin punto fijo); cadena test→md→yaml (fuerte pero con cascada y hash en doc humano); sello combinado elegido: mismo poder de detección con un solo valor y charter limpio.

### 4.2 C4 + C3 + C5 — Host y superficie externa
🔲 Pendiente de discusión. Al resolver: asentar el concepto host/composition root en el método y evaluar el nodo D-8.

### 4.3 C6 + C7 — Datos transversales
🔲 Pendiente de discusión.

### 4.4 C12 + C10 — Supervivencia diaria del agente
🔲 Pendiente de discusión.

### 4.5 C1 — Promoción de perfil
🔲 Pendiente de discusión.

### 4.6 Resto (C2, C11, C13, C14, C15)
🔲 Pendientes.

### 4.7 OpenDGI-1 — Primer piloto (pre-sello del slice RutValidation) — ✅ RESUELTO

**Resolución (graduada a [Charter v3](DDS_Slice_Charter.md) y [Workflows](DDS_Workflows.md)):**

1. **Q1 — Admisibilidad de campos**: se resuelve como **tiers de verificación por campo** (Charter §3.1.1); la admisibilidad sigue siendo Parte A — un campo no necesita ser verificable por máquina para ser admisible.
2. **Q2 — `consumes` vs. HARD RULE**: se resuelve como **hecho derivable vs. compromiso falseable**; `consumes` se queda en `SLICE.md` (Parte A del charter reformulada). El piloto detectó una contradicción real de redacción en la Parte A vieja.
3. **Q3 — Ciclo de vida de `excludes`**: se resuelve como **integridad referencial del owner por máquina** (Charter §6) + **regla de ripple de ciclo de vida** (Charter §7).
4. **Q4 — Re-stamp de `confirmed`**: el nivel 3 también re-estampa; P1 enmendado.

**Aprendizaje**: la lectura adversarial del piloto detectó la contradicción de la Parte A y la capacidad no explotada de integridad del owner **antes del primer sello** — exactamente la validación que pide el [prospecto](DDS_Metodo_Trabajo_IA.md#prospecto-medible-hipótesis-a-validar).

### 4.8 OpenDGI-2 — Primer sello bajo evaluación (post-RutValidation) — ✅ RESUELTO

**Resolución (graduada a [Charter v4](DDS_Slice_Charter.md) y [Workflows](DDS_Workflows.md)):**

1. **Evaluación del primer sello**: 5 defectos de aplicación (derivables embebidos en prosa, copia verbatim del archivo entero, capa 3 por Contains, binding por nombre, sello exact-bytes) — camino P1 en batch único cuando el owner autorice. El artefacto es válido: nació sellado, tríada íntegra.
2. **Copia embebida eliminada (decisión del owner)**: el test de definición es una **traducción** del charter, no un espejo; el charter se enforcementa como datos por tests genéricos (parseo del front-matter). Tiers reducidos a dos (machine / prose).
3. **`enforced_by`**: pin determinista charter→test; test que rota sin re-stamp = rojo mecánico. Cierra la ventana "test cambió sin su contraparte".
4. **Fila de log al nacer**: B1/B5 ahora exigen decision-log rows en el commit de nacimiento.
5. **Forma canónica LF** para sello y hash (rotaciones falsas cross-platform); migración de re-sello única.
6. **Anti-vacuidad** para arch tests; **vocabulario cruzado** entra a la ripple.
7. **Parte A completada**: promesas testables → contratos y tests de comportamiento; porqués → log; referir, nunca embeber.

**Aprendizaje**: la evaluación congelada del primer sello (preguntas sin cambios, respuestas con concesiones) produjo más método que la consulta previa al sellado — el piloto descubrió que "la cadena detecta deriva del trío, no del código respecto del charter", y la respuesta del owner fue eliminar el mecanismo especular en vez de afinarlo.
