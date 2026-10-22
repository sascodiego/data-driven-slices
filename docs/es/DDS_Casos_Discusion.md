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
| C8 | Bug fix: ¿la verdad es el código o el charter? | Corrección | 🔲 Pendiente | El caso más frecuente; define el principio de fuente de verdad |
| C9 | Conflicto charter ↔ código descubierto (drift) | Corrección | 🔲 Pendiente | Quién gana; el drift en sí es defecto |
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

### 4.1 C8 + C9 — Fuente de verdad: código vs. charter
🔲 Pendiente de discusión.

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
