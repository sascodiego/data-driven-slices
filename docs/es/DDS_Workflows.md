# Workflows Operativos DDS

> [!NOTE]
> **Estado: referencia operativa asentada, a validar.** Este documento define la **matriz de decisiones** (por dónde entra un cambio) y el **procedimiento** de los dos workflows operativos: **modificar el comportamiento de una feature existente** y **nacer un slice nuevo**. Complementa al [Charter de Slice](DDS_Slice_Charter.md) y al [Método de Trabajo IA](DDS_Metodo_Trabajo_IA.md). Sigue la postura del [prospecto](DDS_Metodo_Trabajo_IA.md#prospecto-medible-hipótesis-a-validar): los workflows son la recomendación operativa a contrastar con el primer proyecto real.

---

## 1. Matriz de decisiones (por dónde entra un cambio)

Las preguntas se consultan **en orden**. Cada respuesta deriva a un workflow o a la siguiente pregunta.

| # | Pregunta | Respuesta → acción |
| :-: | :--- | :--- |
| **D-1** | ¿El cambio encaja en el `Purpose` de un slice existente? (se responde leyendo **charters**, no código) | **Sí** → [Workflow A](#2-workflow-a--modificar-comportamiento-de-una-feature). **No / parcial** → D-2. |
| **D-2** | ¿El nuevo capability tiene cluster propio de motivos de cambio, y un `Purpose` de una línea sin mencionar internals ajenos? | **Sí** → [Workflow B](#3-workflow-b--nacer-un-nuevo-slice). **No** → slice existente más cercano (Workflow A) y **anotar el dolor**: candidato a promoción de perfil o slice futuro. |
| **D-3** | ¿El cambio atraviesa varios slices? | **Sí** → ver [multi-slice](#4-cambios-multi-slice): descomponer por slice con el contrato como frontera de coordinación. **No** → continuar. |
| **D-4** | ¿La superficie pública (`contracts.X`) muta? | **Sí** → **evento de evolución de contrato**: paso explícito y visible + re-confirmar charter(s) + enumerar consumidores (usages del namespace). **No** → cambio contenido en el slice. |
| **D-5** | ¿Hay nuevo I/O (HTTP, DB, mensajería)? | **Sí** → toca `delivery.X`, con tests de comportamiento observable. **No** → engine only. |
| **D-6** | ¿El perfil del slice sigue siendo proporcional al riesgo? | Re-evaluar **solo con dolor observable** (promoción por síntomas). Sin dolor → no cambiar. |
| **D-7** | ¿La duda de routing no la resuelve ningún charter? | **Preguntar al humano.** 30 segundos de pregunta antes que un cambio en el slice equivocado. |

## 2. Workflow A — modificar comportamiento de una feature existente

Ejemplo conductor: *"Billing ahora incluye el impuesto X en los cargos"*.

| Paso | Qué se hace | Se lee / se escribe |
| :-: | :--- | :--- |
| **A0** | **Routing charter-first**: leer los `SLICE.md` de los slices candidatos; `Not here` decide el slice. Duda irresoluble → D-7 (preguntar). | Lee: charters. Nada de código todavía. |
| **A1** | **Ver la feature actual**, en este orden: `contracts.X` (superficie) → tests del engine (especificación ejecutable del comportamiento actual) → `engine.X` (recién ahora, el código). | Lee: contracts → tests → engine. |
| **A2** | **Bifurcación de contrato** (D-4): contrato intacto → cambio contenido. Contrato muta → evento de evolución: paso explícito, enumerar consumidores, **re-confirmar el charter en el mismo paso**. | Decide: radio de explosión = lista enumerable, no neblina. |
| **A3** | **Modificar test-first sobre el engine**: caso nuevo entrada→salida (**RED**), cambio en la función pura (**GREEN**). Caso B: adaptar contracts y cada delivery consumidor. | Escribe: tests del engine → engine → (si B) contracts + deliveries. |
| **A4** | **Fronteras**: suite completa — los [arch tests](DDS_Slice_Charter.md) garantizan que nada cruzó la frontera ni metió I/O en engine. Path nuevo → reclamable por el slice (huérfano = defecto). | Verifica: `dotnet test`. |
| **A5** | **Cierre**: commit de unidad de trabajo — engine + tests + (si hubo) contrato + charter re-confirmado, en el mismo commit. | Escribe: el historial del slice queda legible. |

## 3. Workflow B — nacer un nuevo slice

Ejemplo conductor: *"Pricing hoy vive dentro de Billing; será su propio slice"*.

| Paso | Qué se hace | Nota |
| :-: | :--- |
| **B0** | **Justificación** (D-2): `Purpose` de una línea sin internals ajenos + cluster de motivos de cambio. Anti-criterios: no por entidad, no por cambio one-off, si no se puede escribir `Not here` con dueños todavía no es slice. | Es decisión de diseño: el agente consulta ante la duda. |
| **B1** | **Charter-first, y mutan DOS charters**: nace `SLICE.md` del nuevo slice **antes del código**; el charter de origen gana su línea `Not here` y pierde lo que se fue. El nacimiento es una re-partición de responsabilidad — dejar el charter viejo sin actualizar es dejarlo mintiendo. | Primer artifact: charter, no código. |
| **B2** | **Perfil y scaffold mínimo vertical**: perfil al nacer (Lite por defecto; Standard con reglas reales; Strict solo con dolor demostrado). Scaffold = lo que el primer caso de uso necesita; `delivery.X` no nace si no hay I/O propio. | Estructura proporcional, no ceremonial. |
| **B3** | **Contratos antes que engine** (data-first): `contracts.X` se diseña primero — es la decisión más cara de revertir del nacimiento. El consumo desde/hacia otros slices se resuelve por contrato. | |
| **B4** | **Extracción test-first** (si proviene de otro slice): los tests entrada→salida planos **se mudan** al namespace nuevo (RED), la función pura se extrae (GREEN). La pureza del engine es lo que hace la extracción mecánica. | Cero mocks implicados. |
| **B5** | **Fronteras y commit de nacimiento**: los arch tests son una regla genérica por repo — el slice nuevo **hereda la frontera gratis** al existir. Commit único: charter nuevo + charter de origen actualizado + contracts + engine extraído + tests migrados. | El historial registra "Pricing nació de Billing" para siempre. |

## 4. Cambios multi-slice

Cuando un cambio atraviesa varios slices (D-3 = sí):

1. **Descomponer por slice** con la matriz (D-1 en adelante) para cada uno.
2. **El contrato es la frontera de coordinación**: cada slice expone lo que el otro necesita vía `contracts.X`; nunca se comparte engine ni delivery.
3. **Commit por slice o commit coordinado único** (paso de contrato compartido), nunca un commit que mezcle internals de dos slices.
4. Si la coordinación revela un contrato compartido estable, evaluar D-2: ¿el shared shape es un slice propio o vive en uno de los dos con el otro como consumidor? Regla: no crear slice solo para "compartir datos" — crearlo si tiene motivos de cambio propios.

## 5. Hoy vs. con el Tool CLI

| Momento | Hoy (DDS puro, sin tool) | Con el CLI (cuando exista) |
| :--- | :--- | :--- |
| Routing (A0/B0) | El agente lee charters y aplica la matriz por sí | Igual + `dds prompt` inyecta charter y forma del contrato como seed |
| Lectura (A1) | contracts → tests → engine | Igual |
| Modificación (A3) | Arch tests como aduana | + `guard before/after`: snapshot de contratos, exit 1 en path fuera del slice |
| Nacimiento (B1–B5) | Charter y scaffold manuales; arch tests cubren al nuevo gratis | + `dds scaffold slice`: esqueleto de charter + registro de globs en `.dds/config.yaml` |
| Cierre (A5/B5) | Commit convencional de unidad de trabajo | + fila de métricas registrada sin intervención |

El workflow completo funciona **sin** el tool: el tool elimina tipeo y automatiza verificación, no decisiones. Los pasos no acoplan con la metodología del agente: DDS viaja dentro de las fases de cualquier flujo, sin definirlas.

## 6. Regla operativa para el `AGENTS.md` (referencia, no aún en el template)

> Al modificar comportamiento: **charter-first** (routing con `SLICE.md`), lectura en orden `contracts → tests → engine`, bifurcación de contrato **explícita** (D-4), test-first sobre el engine. Al crear un slice: justificación (D-2), charter ×2 antes del código, scaffold mínimo vertical, commit de nacimiento único.

Esta regla aún **no** está en el template de `AGENTS.md` ni en el proyecto de prueba: su rollout es una decisión separada (actualizar template + notificar al agente instalado) para no divergir lo ya desplegado.
