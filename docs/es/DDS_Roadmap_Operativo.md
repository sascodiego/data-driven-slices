# DDS: Hoja de Ruta Operativa
## Lo que ya está definido y lo que falta para hacer DDS más robusto

> [!NOTE]
> **Estado: hoja de ruta / prospecto, no verdad validada.** Este documento resume el alcance actual del método **Data-Driven Slices (DDS)** y **Data-Oriented Design (DoD)** tal como están hoy en la documentación, y enumera lo que **aún falta** para que el método sea más robusto. Es una guía de trabajo, no un conjunto de resultados probados. Las hipótesis de beneficio se validarán con ejemplos y benchmarks (ver el [Prospecto Medible](DDS_Metodo_Trabajo_IA.md#prospecto-medible-hipótesis-a-validar)).

---

## Lectura rápida

1. **DoD** aporta la base de datos: structs planos, transformaciones sin estado, localidad de memoria. Es una **hipótesis** crítica de POO/DI, no un dogma.
2. **DDS** aporta la estructura: `Data / Decision / Delivery`, perfiles `Lite / Standard / Strict`, promoción por riesgo, integración en SDD y ODD sin fases nuevas y `/core` casi vacío (solo infraestructura).
3. **Lo que falta** son los mecanismos operativos que vuelven esos criterios accionables y medibles: playbooks de promoción, umbrales, ejemplos reales y un manual de evolución de contratos.

---

## 1. Lo que ya está definido para DoD

| Pilar | Qué significa | Dónde está |
| :--- | :--- | :--- |
| **Datos pasivos / structs / PODs / records** | Los datos son estructuras planas sin comportamiento oculto. El estado se transporta, no se esconde tras objetos. | [Introducción a DoD](DDS_Introduccion_DoD.md) |
| **Transformaciones sin estado** | La lógica vive en funciones puras que reciben datos y devuelven datos ($f(x) = y$). | [Método de Trabajo IA](DDS_Metodo_Trabajo_IA.md) |
| **Separación datos / comportamiento** | Lo conocido (Data), lo calculado (Decision) y lo que modifica el mundo (Delivery) viven separados. | [Método de Trabajo IA](DDS_Metodo_Trabajo_IA.md) |
| **Localidad de memoria / AoS / SoA** | El layout físico de los datos importa para la caché: Array of Structures vs. Structure of Arrays. | [Introducción a DoD](DDS_Introduccion_DoD.md) |
| **Crítica moderada de POO/DI** | POO, herencia, vtables y DI pesada se cuestionan como **hipótesis** (no como dogma) porque inflan el contexto de la IA. | [Método de Trabajo IA](DDS_Metodo_Trabajo_IA.md) · [De POO a Slices](DDS_De_POO_a_Slices.md) |
| **Testing por entrada/salida** | Los Engines puros se prueban comparando entrada y salida plana, sin mocks dinámicos. | [TDD Sin Mocks](DDS_TDD_Sin_Mocks.md) |
| **Hipótesis de IA: menos contexto implícito** | Separar datos de comportamiento debería reducir el contexto implícito que la IA debe reconstruir. **A validar.** | [Prospecto Medible](DDS_Metodo_Trabajo_IA.md#prospecto-medible-hipótesis-a-validar) |

---

## 2. Lo que ya está definido para DDS

| Pilar | Qué significa | Dónde está |
| :--- | :--- | :--- |
| **Data / Decision / Delivery** | El criterio central: separar lo conocido, lo calculado y lo que modifica el mundo. | [Método de Trabajo IA](DDS_Metodo_Trabajo_IA.md) |
| **Lite / Standard / Strict** | Tres perfiles de primera clase; la estructura física es proporcional al riesgo, no al dogma. | [Método de Trabajo IA](DDS_Metodo_Trabajo_IA.md) · [README](../../README.md) |
| **Promoción por dolor/riesgo** | Se sube de perfil **solo** ante síntomas concretos. Si no hay dolor, no se promueve. | [Método de Trabajo IA](DDS_Metodo_Trabajo_IA.md) |
| **Integración en SDD y ODD sin fases extra** | DDS se inyecta como criterio dentro de las fases existentes de cualquiera de las dos metodologías agénticas (SDD en Design / Apply / Verify; ODD en Explore / Classify-Track / Implement / Check); **no agrega fases** a ninguna. Ambas coexisten como opciones de primera clase. | [Método de Trabajo IA](DDS_Metodo_Trabajo_IA.md) · [README](../../README.md) |
| **Zero-Mocks acotado a Decision/Engine** | Los mocks dinámicos se prohíben para Decision/Engine; Delivery puede usar dobles de comportamiento observable. | [TDD Sin Mocks](DDS_TDD_Sin_Mocks.md) |
| **`/core` mínimo / solo infraestructura** | `/core` es aburrido y casi vacío: handles opacos de transacción y primitivas técnicas verdaderamente compartidas. **Los IDs y el Dinero NO viven en `/core` por defecto.** | [Método de Trabajo IA](DDS_Metodo_Trabajo_IA.md) |
| **Prospecto con dimensiones medibles** | Ocho dimensiones (tokens, archivos, runtime de tests, radio de explosión, regresión, etc.) con hipótesis a contrastar. | [Prospecto Medible](DDS_Metodo_Trabajo_IA.md#prospecto-medible-hipótesis-a-validar) |
| **Workflows operativos + matriz de decisiones** | Cómo entra un cambio (matriz D-1..D-7), modificar comportamiento (A) y nacer un slice (B), multi-slice y regla operativa para el agente. | [Workflows Operativos](DDS_Workflows.md) |

---

## 3. Lo que falta para hacer el método más robusto

Estos son los entregables pendientes. Cada uno convierte un criterio actual en un mecanismo accionable y verificable.

| # | Entregable faltante | Por qué importa | Estado |
| :-: | :--- | :--- | :--- |
| 1 | **Playbook de promoción/democión** `Lite ↔ Standard ↔ Strict` | Hoy existen disparadores de promoción, pero no el camino de vuelta (democión) ni los pasos concretos. | 🔲 Pendiente |
| 2 | **Umbrales medibles iniciales** | El prospecto nombra dimensiones pero no valores umbral que disparen una promoción. Sin números, la decisión sigue siendo subjetiva. | 🔲 Pendiente |
| 3 | **Ejemplo real de DDS Lite (CRUD)** | Falta un ejemplo terminal-a-terminal del perfil más liviano, para mostrar que DDS no exige ceremonia. | 🔲 Pendiente |
| 4 | **Ejemplo de evolución** `Lite → Standard → Strict` | Falta narrar el mismo slice creciendo de perfil a perfil, para mostrar que la promoción es natural y reversible. | 🔲 Pendiente |
| 5 | **Manual de evolución de contratos públicos** | Hoy se dice "versionar/adaptar/migrar", pero falta el procedimiento paso a paso y los criterios de retrocompatibilidad. | 🔲 Pendiente |
| 6 | **Guía de persistencia por perfil (con casos de cuándo NO segregar)** | La segregación está documentada para Standard/Strict, pero falta el ejemplo inverso: cuándo un `DbContext` compartido es correcto en Lite. | 🔲 Pendiente |
| 7 | **Definición operativa del radio de explosión del refactor** | Es una dimensión clave del prospecto, pero aún no tiene una definición operativa precisa (cómo medirlo de forma repetible). | 🔲 Pendiente |
| 8 | **Tool CLI `dds` (guard + factory)** | Verificación determinista de fronteras entre slices (el compilador solo cubre Strict) y generación sistemática de skills/prompts desde el criterio. Diseño decidido en el [Registro de Decisiones del Tool](DDS_Tool_CLI_Decisiones.md); implementación pendiente. | 🟡 Diseño decidido |
| 9 | **Charter de slice + tests de arquitectura** | Intención por slice sin doc dependiente del código (`SLICE.md`) y fronteras ejecutables que corren con la suite. Diseño decidido en el [doc de Charter](DDS_Slice_Charter.md); validación con el primer proyecto real pendiente. | 🟡 Diseño decidido |

### Próximo entregable consolidador

- 🔲 **Playbook Operativo DDS** — El siguiente gran entregable. Consolida los ítems 1 a 7 en un solo documento operacional: playbooks de promoción/democión, umbrales, ejemplos y manual de contratos. Es lo que transforma los criterios actuales en un método repetible.

---

## 4. Checklist de madurez del método

- [x] Criterio central definido (`Data / Decision / Delivery`).
- [x] Tres perfiles progresivos de primera clase.
- [x] Disparadores de promoción por síntomas.
- [x] Postura Zero-Mocks acotada a Decision/Engine.
- [x] Regla de `/core` corregida (infraestructura; IDs y Dinero fuera por defecto).
- [x] Integración en SDD y ODD sin fases nuevas.
- [x] Prospecto con dimensiones e hipótesis medibles.
- [x] Decisiones de diseño del tool CLI `dds` (guard + factory) asentadas.
- [x] Charter de slice (`SLICE.md`) + tests de arquitectura: propuesta asentada.
- [x] Workflows operativos (modificación / nacimiento de slice) con matriz de decisiones asentados.
- [ ] Charter + arch tests validados con el primer proyecto real.
- [ ] Tool CLI `dds` implementado (guard file-level + factory de skills/prompts).
- [ ] Playbook de promoción **y democión** entre perfiles.
- [ ] Umbrales numéricos iniciales por dimensión.
- [ ] Ejemplo real de DDS Lite (CRUD completo).
- [ ] Ejemplo de evolución `Lite → Standard → Strict`.
- [ ] Manual de evolución de contratos públicos.
- [ ] Guía de persistencia por perfil con casos de NO segregación.
- [ ] Definición operativa del radio de explosión.
- [ ] Playbook Operativo DDS consolidado.

---

## 5. Cómo leer esta hoja de ruta

- **No es verdad validada.** Es un mapa de lo definido y lo pendiente; cada afirmación de beneficio sigue siendo hipótesis hasta validarla con ejemplos.
- **El criterio es universal; la evidencia es local.** Cada ejemplo trabajado publicará sus mediciones crudas contra el [Prospecto Medible](DDS_Metodo_Trabajo_IA.md#prospecto-medible-hipótesis-a-validar).
- **El siguiente paso lógico** es cerrar los siete huecos y consolidarlos en el **Playbook Operativo DDS**.
