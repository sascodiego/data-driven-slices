# Charter de Slice, Tríada de Definición y Tests de Arquitectura

> [!NOTE]
> **Estado: propuesta asentada (v4), pendiente de validación.** Este documento incorpora al criterio DDS la **tríada de definición** por slice (charter + test de definición + sello) y los **tests de arquitectura**. Incluye la resolución del cluster C8+C9 del [registro de discusión](DDS_Casos_Discusion.md) y de los clusters **OpenDGI-1** (primer piloto) y **OpenDGI-2** (primer sello bajo evaluación). Sigue la postura del [prospecto](DDS_Metodo_Trabajo_IA.md#prospecto-medible-hipótesis-a-validar): recomendaciones a poner a prueba, no resultados probados.

---

## 1. El problema

Un slice es una unidad de cambio — pero dentro del slice, el *propósito* queda implícito en el código. Eso produce dos fallos conocidos:

1. **Routing erróneo**: "¿esto va en `Billing` o en `Invoicing`?" se responde por intuición. El scope creep es el vector #1 de pudrición de slices.
2. **Doc muerto**: la respuesta clásica — un documento que describe lo que hace el código — diverge a las semanas y nadie lo mantiene.

## 2. El principio, en dos partes

**Parte A — Solo lo que el código no puede expresar:**

> El charter de un slice solo puede contener lo que el código no puede expresar: por qué existe, qué le está prohibido, qué NO es su problema, a quién consume. La distinción operativa es **hecho derivable vs. compromiso falseable**: un *hecho derivable* describe al repo y nadie lo tiene como promesa (inventarios de archivos, clases, flujos) — va banned. Un *compromiso falseable* es intención que el código puede violar, y por eso va en el charter: `consumes` declara a quién depende este slice y nada más, y se enforcea por igualdad en ambas direcciones (cruce no declarado = rojo; declaración sin consumo real = rojo).

La frontera se sostiene en dos direcciones. Las promesas **testables** (precondiciones, forma de salida, casos de rechazo) viven en `contracts.X` y en los tests de comportamiento; el charter no las repite. El charter lleva la intención y los datos de frontera; el *porqué* de cada decisión va al decision log. Referir, nunca embeber: un valor que un test puede expresar no entra en el charter ni como referencia congelada.

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
├── Billing.slice.test.cs     # test de definición: la traducción de las cláusulas
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
enforced_by: <sha256 of [Slice].slice.test.cs>
confirmed: 2026-10-22
---
```

- Prohibido: inventarios de archivos, clases, flujos, modelos de datos. Ley de tamaño: una pantalla; si necesita índice, documenta estructura y viola la regla.
- Los porqués **no viven acá**: viven en el decision log (tabla anclada del proyecto), referenciados por ID.
- `enforced_by` es el pin del enforcer: el charter declara qué versión del test lo enforcementa. Un test que cambia sin re-stamp en el charter es rojo determinista (§3.3).

### 3.1.1 Tiers de verificación por campo

Cada campo del charter declara qué detección tiene. La admisibilidad de un campo no depende de su verificabilidad (el criterio de admisión sigue siendo Parte A); depende de su valor de decisión. Desde v4 hay dos tiers: el charter se enforcementa **como datos** (lectura parseada del front-matter por tests genéricos repo-level), y lo que ningún dato expresa queda en prosa.

| Campo | Tier | Detección |
| :--- | :--- | :--- |
| `purpose` | prose | Sin detector de máquina: consumo A0 + re-confirmación anclada a eventos. Rot aceptada por diseño. |
| `excludes` | machine (estructural) + prose (semántico) | Integridad referencial del `owner` (§6) + ripple rule (§7). La atribución concepto→owner queda en revisión. |
| `language` | prose (local) + machine (cruzado) | Sin escaneo de vocabulario dentro del slice; referencias de vocabulario entre charters entran a la ripple (§7). |
| `consumes` | machine | Igualdad bidireccional declarado ↔ real, por lectura parseada (§6). |
| `profile` | machine | Presencia y valores válidos según perfil (lectura parseada). |
| `surface` | machine | Existencia de `contracts.X` (lectura parseada). |
| `enforced_by` | machine | Igualdad determinista con el hash canónico del test de definición (§3.3). |

### 3.2 `[Slice].slice.test.cs` — la traducción

El test de definición es una **traducción a código de lo que el charter declara** — una especificación, no un espejo. Prohibida la copia embebida del texto del charter (ni completa ni por cláusulas): el `.md` se usa como datos a verificar, nunca como verdad a confiar, y la verificación de datos vive en los tests genéricos (§6).

El test de definición traduce a mano, en el nacimiento y en cada rotación P1:
- La forma de la promesa: operaciones de contrato con su firma (parámetros y retorno, no solo nombres) y sus estados de salida exactos.
- Reglas estructurales específicas del slice que los tests genéricos no cubren.

Cambiar el test sin que el charter lo acompañe da rojo por `enforced_by` (§3.3); cambiar el charter sin re-traducir el test queda cubierto por sello + tests genéricos + revisión (§8).

### 3.3 `slice.shax` — el sello del par, en forma canónica

```
sha256( sha256(canonical(SLICE.md)) + sha256(canonical([Slice].slice.test.cs)) )
```

Una línea hex en `slice.shax`. `canonical()` es la forma canónica del método: los bytes se decodifican como **UTF-8 con BOM removido** y luego se normaliza a **LF** — así ni el encoding ni un checkout en otra plataforma con conversión de fin de línea producen rotaciones falsas. Hashing y comparaciones corren siempre sobre la forma canónica; los digests existentes se migran una sola vez.

**Un sello para el par**: cualquier modificación de cualquiera de los dos archivos rompe el sello → rojo al correr la suite → obligación de explicar por qué cambió. Y la dirección inversa queda determinista: `enforced_by` en el front-matter fija el hash canónico del test; un test que rota sin re-stamp en el charter es rojo, aunque el sello se recompute. Nadie hashea el `.shax`: es el ancla local del slice; su integridad es visibilidad de diff + revisión (+ guard futuro).

## 4. Protocolo de definición (P0–P3)

**P0 — Compuerta de impacto:** ¿hasta qué capa sube el cambio?

| Nivel | Cambia | Toca | NO toca |
| :-: | :--- | :--- | :--- |
| **3. Definición** | Qué es / para qué: purpose, excludes, consumes, language, profile | Charter + test de definición + `.shax` + fila de log | — |
| **2. Promesa** | La superficie (`contracts.X`) sin cambiar el concepto | Evento de evolución de contrato (D-4) + re-stamp `confirmed` | Tríada |
| **1. Implementación** | Lógica interna del engine, internals del delivery | Engine + tests de comportamiento | **Ni el `.md`, ni su test, ni el sello** |

**P1 — Camino de definición:** editar cláusulas → re-traducir el test → recomputar el hash canónico del test y re-stamp `enforced_by` en el front-matter → recomputar el sello del par → `.shax` → re-stamp `confirmed` → fila en el decision log. La detección es de estado: pasos a medias = suite roja hasta completar. El re-stamp de nivel 3 no es redundancia: `confirmed` significa *última confirmación deliberada del charter* (P1 o P2) — frescura para el lector A0; el sello lleva integridad, `enforced_by` lleva el vínculo charter→test, y la fila de log lleva provenancia.

**P2 — Camino de contrato:** evento D-4 existente + re-stamp. El log no se toca (dos ceremonias para lo mismo es burocracia).

**P3 — Contenido:** nada extra. Fricción cero para el caso común.

**P0 aplicado a bugs (C8):** el triage de bug es la compuerta — ¿el bug es de implementación (código ≠ su test), de promesa (contrato miente) o de definición (el charter mintiendo)? Se corrige en la capa clasificada y el cambio cae hacia abajo. RED primero, desde el reporte.

## 5. Matriz de detección

| Tocan | Resultado |
| :--- | :--- |
| `SLICE.md` o el test, solos o juntos, sin `.shax` | 🔴 sello del par roto |
| `slice.shax` solo | 🔴 declarado ≠ actual |
| Test + `.shax` sin re-stamp `enforced_by` | 🔴 enforced_by ≠ hash canónico del test |
| Tríada completa rotada, sin fila de log | 🟡 verde de máquina, diff visible del anchor — la revisión exige el acta |
| Tríada completa + fila de log | 🟢 camino legítimo P1 |
| Referencia cross-slice no declarada en `consumes` | 🔴 test genérico de charters: declarado ≠ real |
| `excludes.owner` apunta a un slice inexistente | 🔴 integridad referencial: owner no resuelve |
| Slice renombrado/disuelto sin re-confirmar los charters que lo nombran | 🟡 ripple rule: la revisión exige la re-confirmación en el mismo commit |

El residuo irreducible: la rotación completa sin fila de log es indistinguible de la legítima por máquina — lo cubren git history, revisión y el guard futuro (regla de diff: rotación de `.shax` ⇒ fila de log en el commit).

## 6. Tests de arquitectura (una regla por repo)

- Fronteras corren con la suite normal y fallan en rojo: ningún `Slices.X.engine` depende de `Slices.Y.*` (X≠Y); ningún engine depende de ningún delivery. Slices nuevos heredan la frontera gratis. **Guardia de anti-vacuidad**: un arch test que no cargó ningún slice/assemblies es rojo, no verde vacuo.
- Mecanismo: NetArchTest / ArchUnitNET / test de reflexión propio (~50 líneas).
- El **test genérico de charters como datos** (repo-level, cero deps, un solo test para todos los slices): parsea el front-matter de cada charter (parser mínimo propio — nunca matching de strings sobre el texto) y verifica: tríada presente, sello vigente, `enforced_by` igual al hash canónico del test, integridad referencial de `excludes.owner` (todo owner resuelve a un slice existente con su tríada), `consumes` declarado ↔ cruces reales, `profile` y `surface` válidos. Un front-matter que no parsea es rojo.

## 7. Mecánica anti-pudrición

1. **Lo ejecutable no se desactualiza en silencio**: sellos, consistencia y arch tests fallan ruidosamente.
2. **Re-confirmación anclada a eventos**: evolución de contrato ⇒ re-stamp del charter (P2). Contrato estable → charter estable.
3. **Consumo real**: el charter se lee al abrir cualquier tarea del slice (A0) y es seed natural de prompts (factory futura).
4. **`consumes` como detección de creep**: el cruce de frontera de namespace sin declaración = rojo — sin escaneo de vocabulario, igualdad exacta entre declarado y real.
5. **Ripple de ciclo de vida**: nacimiento, renombre, disolución o migración de concepto de un slice obliga a re-confirmar, en el mismo commit, todos los charters que lo nombren en `excludes`/`consumes` (el nacimiento del Workflow B1 muta dos charters como mínimo; el ripple los generaliza a N).
6. **Ripple de vocabulario**: cuando un charter referencia términos `language` de OTRO charter, esas referencias entran al conjunto de la ripple — la rotación del slice dueño del término re-confirma los charters que lo citan, en el mismo commit.

## 8. Límites honestos

- El sello protege **integridad**, no calidad: un test mal derivado, sellado, sigue mal derivado. La derivación es deliberada; su adecuación es revisión.
- El creep **sin firma estructural** (lógica ajena escrita desde cero sin cruzar namespaces) no lo detecta máquina: queda en lectura (A0/A1) y re-confirmación.
- Determinista para lo estructural y declarado; honesto para lo semántico.
- El residuo de la rotación doble (charter + test juntos, re-sellados) sigue siendo indistinguible por máquina: lo cubren la fila de log, la revisión y el guard futuro — igual que antes de v4.
- Un cambio de cláusula que requería una regla traducida nueva y no la trae da verde: la traducción es deliberada; su adecuación es revisión. El método no re-embebe derivables para cerrar esa ventana.

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
