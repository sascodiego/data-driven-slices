# Charter de Slice, Tríada de Definición y Tests de Arquitectura

> [!NOTE]
> **Estado: propuesta asentada (v3), pendiente de validación.** Este documento incorpora al criterio DDS la **tríada de definición** por slice (charter + test de definición + sello) y los **tests de arquitectura**. Incluye la resolución del cluster C8+C9 del [registro de discusión](DDS_Casos_Discusion.md) y del cluster **OpenDGI-1** (primer piloto). Sigue la postura del [prospecto](DDS_Metodo_Trabajo_IA.md#prospecto-medible-hipótesis-a-validar): recomendaciones a poner a prueba, no resultados probados.

---

## 1. El problema

Un slice es una unidad de cambio — pero dentro del slice, el *propósito* queda implícito en el código. Eso produce dos fallos conocidos:

1. **Routing erróneo**: "¿esto va en `Billing` o en `Invoicing`?" se responde por intuición. El scope creep es el vector #1 de pudrición de slices.
2. **Doc muerto**: la respuesta clásica — un documento que describe lo que hace el código — diverge a las semanas y nadie lo mantiene.

## 2. El principio, en dos partes

**Parte A — Solo lo que el código no puede expresar:**

> El charter de un slice solo puede contener lo que el código no puede expresar: por qué existe, qué le está prohibido, qué NO es su problema, a quién consume. La distinción operativa es **hecho derivable vs. compromiso falseable**: un *hecho derivable* describe al repo y nadie lo tiene como promesa (inventarios de archivos, clases, flujos) — va banned. Un *compromiso falseable* es intención que el código puede violar, y por eso va en el charter: `consumes` declara a quién depende este slice y nada más, y se enforcea por igualdad en ambas direcciones (cruce no declarado = rojo; declaración sin consumo real = rojo).

**Parte B — La pila de verdad** (resolución C8/C9):

```text
1. CHARTER   — la intención      (por qué existe, límites, consumidores)
2. CONTRACT  — la promesa        (qué entra y sale por la frontera)
3. TEST      — la regla declarada (entrada→salida prometida; enforcement)
4. CODE      — la implementación  (una forma de cumplir 3)
```

Los cambios bajan gratis; **subir de capa exige artifacts explícitos en cada nivel**. Una contradicción que sube desde abajo es un defecto que se clasifica antes de resolverse, nunca algo que se acomoda. Regla dura: **jamás modificar una aserción para poner la suite en verde** — cambiar un test que codificaba comportamiento es un cambio de especificación y va en el mismo commit con la justificación de por qué el comportamiento viejo estaba mal.

## 3. La tríada de definición

Tres archivos en la raíz de cada slice. Un slice sin tríada no existe:

```text
Slices/Billing/
├── SLICE.md                  # charter: cláusulas en front-matter. Cero hashes.
├── Billing.slice.test.cs     # test de definición: reglas derivadas + copia embebida
├── slice.shax                # sello: sha256 del par (una línea hex)
├── contracts.Billing/ ...
```

### 3.1 `SLICE.md` — charter como datos

```markdown
---
name: Billing
purpose: own everything about charging money for completed sales
excludes:
  - { concept: pricing, owner: Pricing, decision: DDS-012 }
language:
  - { term: Charge, means: attempt to charge money }
consumes: [Pricing]
profile: { level: Standard, decision: DDS-011 }
surface: contracts.Billing
confirmed: 2026-10-22
---
```

- Prohibido: inventarios de archivos, clases, flujos, modelos de datos. Ley de tamaño: una pantalla; si necesita índice, documenta estructura y viola la regla.
- Los porqués **no viven acá**: viven en el decision log (tabla anclada del proyecto), referenciados por ID.

### 3.1.1 Tiers de verificación por campo

Cada campo del charter declara qué detección tiene. La admisibilidad de un campo no depende de su verificabilidad (el criterio de admisión sigue siendo Parte A); depende de su valor de decisión.

| Campo | Tier | Detección |
| :--- | :--- | :--- |
| `purpose` | prose | Sin detector de máquina: consumo A0 + re-confirmación anclada a eventos. Rot aceptada por diseño. |
| `excludes` | machine (estructural) + prose (semántico) | Integridad referencial del `owner` (§6) + ripple rule (§7). La atribución concepto→owner queda en revisión. |
| `language` | prose | Sin escaneo de vocabulario (decisión explícita): consumo A0 + re-confirmación. |
| `consumes` | machine | Igualdad bidireccional declarado ↔ real (test de definición). |
| `profile` | machine | Presencia y valores válidos según perfil. |
| `surface` | machine | Existencia de `contracts.X`. |

### 3.2 `[Slice].slice.test.cs` — el enforcement

- Reglas estructurales **derivadas de las cláusulas**, con **copia embebida** de la declaración que enforcementan (no confía en el `.md` en runtime).
- Chequeo de consistencia: copia embebida ↔ front-matter. Un charter editado sin re-derivar el test da rojo.
- Se compila vía glob en un único proyecto de tests de definiciones repo-level.

### 3.3 `slice.shax` — el sello del par

```
sha256( sha256(SLICE.md) + sha256([Slice].slice.test.cs) )
```

Una línea hex en `slice.shax`. **Un sello para el par**: cualquier modificación de cualquiera de los dos archivos rompe el sello → rojo instantáneo al correr la suite → obligación de explicar por qué cambió. Nadie hashea el `.shax`: es el ancla local del slice; su integridad es visibilidad de diff + revisión (+ guard futuro).

## 4. Protocolo de definición (P0–P3)

**P0 — Compuerta de impacto:** ¿hasta qué capa sube el cambio?

| Nivel | Cambia | Toca | NO toca |
| :-: | :--- | :--- | :--- |
| **3. Definición** | Qué es / para qué: purpose, excludes, consumes, language, profile | Charter + test de definición + `.shax` + fila de log | — |
| **2. Promesa** | La superficie (`contracts.X`) sin cambiar el concepto | Evento de evolución de contrato (D-4) + re-stamp `confirmed` | Tríada |
| **1. Implementación** | Lógica interna del engine, internals del delivery | Engine + tests de comportamiento | **Ni el `.md`, ni su test, ni el sello** |

**P1 — Camino de definición:** editar cláusulas → re-derivar el test (reglas + copia embebida) → recomputar el sello del par → `.shax` → **re-stamp `confirmed`** → fila en el decision log. La detección es de estado: pasos a medias = suite roja hasta completar. El re-stamp de nivel 3 no es redundancia: `confirmed` significa *última confirmación deliberada del charter* (P1 o P2) — frescura para el lector A0; el sello lleva integridad y la fila de log lleva provenancia.

**P2 — Camino de contrato:** evento D-4 existente + re-stamp. El log no se toca (dos ceremonias para lo mismo es burocracia).

**P3 — Contenido:** nada extra. Fricción cero para el caso común.

**P0 aplicado a bugs (C8):** el triage de bug es la compuerta — ¿el bug es de implementación (código ≠ su test), de promesa (contrato miente) o de definición (el charter mintiendo)? Se corrige en la capa clasificada y el cambio cae hacia abajo. RED primero, desde el reporte.

## 5. Matriz de detección

| Tocan | Resultado |
| :--- | :--- |
| `SLICE.md` o el test, solos o juntos, sin `.shax` | 🔴 sello del par roto |
| `slice.shax` solo | 🔴 declarado ≠ actual |
| Charter + `.shax` (re-definir sin re-derivar) | 🔴 consistencia: copia embebida ≠ front-matter |
| Test + `.shax` (reemplazar enforcement) | 🟡 verde de máquina, diff visible del anchor — la revisión exige el acta |
| Tríada completa + fila de log | 🟢 camino legítimo P1 |
| Referencia cross-slice no declarada en `consumes` | 🔴 test de definición: declarado ≠ real |
| `excludes.owner` apunta a un slice inexistente | 🔴 integridad referencial: owner no resuelve |
| Slice renombrado/disuelto sin re-confirmar los charters que lo nombran | 🟡 ripple rule: la revisión exige la re-confirmación en el mismo commit |

El residuo irreducible: la rotación completa sin fila de log es indistinguible de la legítima por máquina — lo cubren git history, revisión y el guard futuro (regla de diff: rotación de `.shax` ⇒ fila de log en el commit).

## 6. Tests de arquitectura (una regla por repo)

- Fronteras corren con la suite normal y fallan en rojo: ningún `Slices.X.engine` depende de `Slices.Y.*` (X≠Y); ningún engine depende de ningún delivery. Slices nuevos heredan la frontera gratis.
- Mecanismo: NetArchTest / ArchUnitNET / test de reflexión propio (~50 líneas).
- Más el **seal test genérico** (repo-level, ancla de confianza junto a los arch tests): por cada slice, hashea el par, compara contra `.shax`, y exige presencia de la tríada completa. Más el **test de integridad referencial**: todo `excludes.owner` de cada charter debe resolver a un slice existente con su tríada presente — el front-matter es datos, el owner es un nombre de slice, y la referencia rota es roja.

## 7. Mecánica anti-pudrición

1. **Lo ejecutable no se desactualiza en silencio**: sellos, consistencia y arch tests fallan ruidosamente.
2. **Re-confirmación anclada a eventos**: evolución de contrato ⇒ re-stamp del charter (P2). Contrato estable → charter estable.
3. **Consumo real**: el charter se lee al abrir cualquier tarea del slice (A0) y es seed natural de prompts (factory futura).
4. **`consumes` como detección de creep**: el cruce de frontera de namespace sin declaración = rojo — sin escaneo de vocabulario, igualdad exacta entre declarado y real.
5. **Ripple de ciclo de vida**: nacimiento, renombre, disolución o migración de concepto de un slice obliga a re-confirmar, en el mismo commit, todos los charters que lo nombren en `excludes`/`consumes` (el nacimiento del Workflow B1 muta dos charters como mínimo; el ripple los generaliza a N).

## 8. Límites honestos

- El sello protege **integridad**, no calidad: un test mal derivado, sellado, sigue mal derivado. La derivación es deliberada; su adecuación es revisión.
- El creep **sin firma estructural** (lógica ajena escrita desde cero sin cruzar namespaces) no lo detecta máquina: queda en lectura (A0/A1) y re-confirmación.
- Determinista para lo estructural y declarado; honesto para lo semántico.

## 9. Relación con el Tool CLI

El charter como datos (front-matter) y el `.shax` son consumibles directos del guard y la factory (criterio-como-datos, D5 del [registro del tool](DDS_Tool_CLI_Decisiones.md)). El guard file-level agrega: snapshot before/after de sellos y contratos, y la regla de diff "rotación de sello ⇒ fila de log". La verificación interim corre completa en la suite, sin tool.

## 10. Hipótesis a validar

| Hipótesis | Métrica |
| :--- | :--- |
| El charter reduce routing erróneo | Cambios re-asignados de slice tras revisión |
| `consumes` detecta creep antes que la revisión | Cruces detectados por suite vs. por revisión |
| El costo de mantenimiento de la tríada tiende a cero | Rotaciones de sello que no sean cambios de definición reales |
| La fricción del nivel 3 no desalienta documentar | Tríadas vigentes vs. slices totales |

## 11. Integración en el workflow del agente

Resuelta en [Workflows Operativos DDS](DDS_Workflows.md): matriz de entrada, workflows A/B con la compuerta P0 integrada, y el triage de bug. La regla operativa para el `AGENTS.md` vive ahí, con su rollout como decisión separada.
