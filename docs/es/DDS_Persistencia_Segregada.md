# Diseño de Persistencia Segregada en DDS: Abstracción vs. Duplicación

Este documento formaliza las directrices y decisiones de diseño sobre cómo estructurar la persistencia de datos (como `DbContext` de Entity Framework o conexiones a bases de datos) en la arquitectura **Data-Driven Slices (DDS)**, resolviendo la tensión entre la duplicación de código y el acoplamiento monolítico de datos.

> [!NOTE]
> **Esto corresponde a los perfiles DDS Standard / Strict.** En **DDS Lite** (CRUD simple, bajo riesgo), un `DbContext` compartido o un único repositorio es perfectamente aceptable mientras no genere acoplamiento doloroso. Se promueve a segregación **solo cuando aparecen los síntomas**: la IA debe leer demasiado, los cambios en un slice rompen otro, o el refactor pierde control.

---

## Decisiones Clave

| Concepto | Decisión de Diseño (Standard / Strict) | En Lite | Beneficio para la IA |
| :--- | :--- | :--- | :--- |
| **Monolito de Datos** | **Evitado.** No se recomienda un único `DbContext` centralizado con todas las tablas del sistema. | Aceptable mientras no duela. | Evita la contaminación del contexto mental de la IA. |
| **Persistencia Segregada** | Cada Slice define su propia persistencia y sus propias tablas (*Bounded Contexts* locales). | Opcional. | Limita el scope de refactorización a un solo directorio. |
| **Regla DRY (*Don't Repeat Yourself*)** | Se flexibiliza para la infraestructura. **La duplicación de esquemas es preferible al acoplamiento de datos.** | Igual. | Previene fallos colaterales e impide que cambios en un slice rompan otros. |
| **Transaccionalidad** | Se delega compartiendo la conexión física (`IDbTransaction` o punteros opacos) a través de contratos públicos. | Igual. | *Busca* preservar la atomicidad de SQL Server sin romper el aislamiento. |
| **Evolución de Contratos** | Los contratos públicos consumidos fuera del slice se **versionan / adaptan / migran**, no se cambian en silencio. | Igual. | Evita rupturas silenciosas entre slices y sistemas externos. |

---

## 1. Segregación de Layouts de Datos

En **DoD (Data-Oriented Design)**, los datos son la prioridad y el código es secundario. Los esquemas de base de datos representan la disposición física (*Layout*) de los datos.

Si creamos un único archivo de contexto (`DbContext`) compartido por todos los slices, introducimos un **acoplamiento en tiempo de compilación**. Si el slice de `Inventory` requiere añadir una columna a la tabla de stock, el proyecto entero debe recompilarse y la IA que asista en `Sales` estará expuesta a tipos contables y financieros innecesarios, aumentando la probabilidad de alucinaciones.

### La Solución: DbContexts Segregados
Cada Slice declara únicamente las tablas y campos que necesita para operar, compilando su contexto de datos de forma estrictamente privada (`internal`):

```csharp
// /slices/inventory/delivery.cs
// Este DbContext es privado y solo conoce la tabla de Stock
internal class InventoryDbContext : DbContext 
{
    public DbSet<Stock> Stocks { get; set; }
}
```

La lógica repetitiva de **infraestructura** (cargar el driver de base de datos, configurar los interceptores de logs o leer la cadena de conexión) **no se duplica**: se resuelve una sola vez en el ensamblado `Core` —que es de **infraestructura, no de dominio**— o se inyecta desde la UI al arrancar. Los IDs y montos de negocio **no** viven en `/core`; viven en el slice o contrato donde su semántica es explícita.

---

## 2. El Dilema DoD: Duplicación vs. Acoplamiento

La programación orientada a humanos tradicional nos enseñó a evitar la duplicación de código a toda costa. Sin embargo, esto suele producir abstracciones complejas y acopladas que resultan **difíciles** de refactorizar de forma automatizada por una IA.

La filosofía de **DoD puro** establece que:
> **Es preferible duplicar una estructura de datos o una consulta de lectura local antes que crear una dependencia compartida que acople dos sistemas independientes.**

Si dos Slices necesitan leer información del stock:
*   **Mal camino (POO):** Crear un servicio `SharedStockService` que ambos consuman. Si el Slice A necesita modificar el servicio, rompe al Slice B.
*   **Buen camino (DoD):** Cada Slice define su propia consulta de lectura optimizada en su capa `delivery`. Son independientes. Si un Slice cambia su forma de leer, el otro permanece inalterado.

---

## 3. Transacciones Cruzadas Compartidas

Para asegurar la atomicidad de SQL Server (que todo pase o todo falle en un solo bloque) sin romper el aislamiento del código, compartimos el manejador físico de la transacción (`IDbTransaction` / `DbTransaction*`), no las clases de base de datos.

El orquestador (`Sales`) abre la conexión física y delega las operaciones de escritura en los otros slices pasándoles la transacción. Cada slice mapea la transacción a su propio contexto privado de forma aislada.

```csharp
// En Slices.Sales (Orquestador)
using (var tx = connection.BeginTransaction())
{
    // Escribe la venta
    SalesDb.WriteSale(result.Sale, tx); 

    // Delega al contrato público de Inventario pasándole la transacción
    InventoryDb.UpdateStock(stockUpdate, tx); 

    tx.Commit();
}
```

---

## Checklist de Verificación para la IA

> Los ítems marcados **(Standard / Strict)** aplican a partir de esos perfiles. En **Lite** se evalúan como síntomas: si fallan y generan dolor, es señal de promoción.

*   [ ] **(Standard / Strict)** ¿El slice actual tiene dependencias de compilación con archivos de persistencia (`DbContext`, SQL) de otro slice? **(Debe ser NO)**
*   [ ] **(Standard / Strict)** ¿El `DbContext` local contiene tablas que pertenecen a la lógica de negocio de otros dominios? **(Debe ser NO)**
*   [ ] ¿El motor (`engine.cs` / `engine.hpp`) está completamente libre de código de persistencia, comandos SQL o librerías de base de datos? **(Debe ser SÍ)**
*   [ ] ¿Se comparte la transacción física a través de las firmas públicas de contratos para preservar la atomicidad en SQL? **(Debe ser SÍ)**
*   [ ] Si un contrato público se consume fuera de su slice, ¿está **versionado / adaptado / migrado** en lugar de cambiarse en silencio? **(Debe ser SÍ)**
*   [ ] ¿Los DTOs, modelos de negocio, IDs y montos (Dinero) viven en su slice/contrato y **no** en `/core`? **(Debe ser SÍ)**

---

## Próximo Paso

*   Para ver la implementación de referencia en código C++20 y C# (.NET 8+), consultá el estándar en el [Manual de Método de Trabajo DDS](DDS_Metodo_Trabajo_IA.md).
