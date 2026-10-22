# Charter de Slice y Tests de Arquitectura

> [!NOTE]
> **Estado: propuesta asentada, pendiente de validación.** Este documento incorpora al criterio DDS dos mecanismos complementarios: el **charter de slice** (intención) y los **tests de arquitectura** (fronteras ejecutables). Sigue la postura del [prospecto](DDS_Metodo_Trabajo_IA.md#prospecto-medible-hipótesis-a-validar): son recomendaciones a poner a prueba con el primer proyecto real, no resultados probados.

---

## 1. El problema

Un slice es una unidad de cambio — pero dentro del slice, el *propósito* queda implícito en el código. Eso produce dos fallos conocidos:

1. **Routing erróneo**: "¿esto va en `Billing` o en `Invoicing`?" se responde por intuición. El scope creep es el vector #1 de pudrición de slices.
2. **Doc muerto**: la respuesta clásica — un documento que describe lo que hace el código — diverge a las semanas y nadie lo mantiene.

## 2. El principio

> **El charter de un slice solo puede contener lo que el código no puede expresar: por qué existe, qué le está prohibido hacer, y qué NO es su problema. Si una máquina puede derivarlo del repo, va contra la regla.**

Con esa regla, el documento no se mantiene porque no describe nada que cambie seguido: el *propósito* de un slice cambia una vez por año; su estructura cambia todos los días. Se documenta lo lento, jamás lo rápido.

## 3. Los dos artifacts (separados por naturaleza)

### 3.1 `SLICE.md` — charter de intención

10 a 25 líneas, colocado **dentro del slice** (docs colocalizados se leen; docs en carpetas centrales se pudren sin testigos):

```markdown
# Slice: Billing
Purpose: own everything about charging money for completed sales.
Not here: pricing rules (Pricing), refunds (ApproveRefund), payment retries (Payments).
Language: Charge (attempt), Invoice (document), Settlement (money moved).
Profile: Standard — real business rules, no compiler walls needed yet.
Surface: contracts.Billing is the only public face. Confirmed: YYYY-MM-DD.
```

**Contenido permitido (solo intención):**

| Campo | Para qué sirve |
| :--- | :--- |
| `Purpose` (una línea) | El capability: los motivos de cambio viven acá |
| `Not here` (exclusiones explícitas con dueño) | **Anti scope-creep** — la línea más valiosa del charter |
| `Language` (3–5 términos del dominio) | Lenguaje ubicuo mínimo, definiciones crisp |
| `Profile` + por qué (una línea) | Elección registrada al crear el slice |
| `Surface` + fecha de confirmación | La regla ("contracts.X es la única cara pública"), no un inventario |

**Prohibido:** inventarios de archivos, listas de clases, flujos, modelos de datos (eso es trabajo de `contracts.X`). Ley de tamaño: si el charter necesita índice, ya está documentando estructura y viola la regla. Máximo una pantalla.

### 3.2 Tests de arquitectura — fronteras ejecutables

La parte exigible no vive en prosa: vive como **tests que corren con la suite normal** y fallan en rojo como cualquier test.

| Perfil | Mecanismo | Nota |
| :--- | :--- | :--- |
| **Strict** | El compilador ya lo hace: un assembly por slice + `internal` | No se necesita test; no compila. Es la aduana nativa. |
| **Standard / Lite** | **NetArchTest** o **ArchUnitNET**: reglas de dependencia como tests xunit normales | Alternativa sin dependencia: test de reflexión propio (~50 líneas) que escanea dependencias entre namespaces de slices. Determinista, corre en ms, alineado con Zero-Mocks. |

La regla es **una por repo, no por slice**: "ningún namespace `Slices.X.engine` depende de `Slices.Y.*` para X≠Y; ningún engine depende de ningún delivery". Los slices nuevos heredan la frontera gratis.

## 4. Mecánica anti-pudrición

1. **Lo ejecutable no se desactualiza en silencio**: un arch test vencido *falla* ruidosamente. La exigencia no depende de que nadie actualice nada.
2. **Re-confirmación anclada a eventos, no al calendario**: cuando un contrato (`contracts.X`) evoluciona, el charter se re-valida en el mismo paso. Contrato estable → propósito casi seguro estable; y el momento del cambio de contrato es el único momento con contexto fresco para corregir divergencia.
3. **Consumo real, no adorno**: el charter se lee al abrir cualquier tarea del slice y es el seed natural del prompt de implementación (ver [Tool CLI](DDS_Tool_CLI_Decisiones.md)). Un doc con tres consumidores (routing del cambio, contexto del agente, checklist de revisión) no muere; uno que nadie lee, sí.

## 5. Relación con el guard del Tool CLI

Esta propuesta **re-encuadra el alcance v2 del guard**: la verificación de imports no se hará con AST en Go (Roslyn-in-Go); vive en el propio lenguaje objetivo como tests de arquitectura. El guard queda file-level (paths, contratos, huérfanos, core) — ver la decisión actualizada en el [Registro de Decisiones del Tool](DDS_Tool_CLI_Decisiones.md).

## 6. Hipótesis a validar

| Hipótesis | Métrica |
| :--- | :--- |
| El charter reduce decisiones de routing erróneas | Cambios re-asignados de slice tras revisión |
| Los arch tests contienen cruces antes que la revisión | Violaciones detectadas por la suite vs. por revisión humana |
| El costo de mantenimiento del charter tiende a cero | Ediciones de charter que no sean re-confirmaciones por evolución de contrato |

## 7. Integración en el workflow del agente

Resuelta en [Workflows Operativos DDS](DDS_Workflows.md): la matriz de decisiones define por dónde entra cada cambio, y los workflows A (modificar comportamiento) y B (nacer un slice) fijan el procedimiento con el charter como primera lectura (charter-first) y las reglas de bifurcación de contrato y nacimiento. La regla operativa para el `AGENTS.md` está referenciada ahí como pendiente de rollout (actualizar template + notificar al agente ya instalado, decisión separada).
