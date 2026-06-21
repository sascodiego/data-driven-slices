# Estándar de Arquitectura: Data-Driven Slices (DDS)
## Guía de Diseño y Método de Trabajo Orientado a IA (DoD + Slices)

> [!NOTE]
> **Estado: prospecto, no un framework probado.** Este documento propone **Data-Driven Slices (DDS)** como un *criterio* arquitectónico y un conjunto de **hipótesis medibles** para el desarrollo con IA. El criterio aspira a ser ampliamente aplicable, pero sus beneficios **aún no están validados**; deben contrastarse mediante ejemplos y benchmarks. Ver [Prospecto Medible](#prospecto-medible-hipótesis-a-validar).

Este documento formaliza el estándar de arquitectura **Data-Driven Slices (DDS)** aplicando el paradigma de **Data-Oriented Design (DoD)** de forma pura. Como **hipótesis de trabajo**, su objetivo es proponer un entorno donde los agentes de Inteligencia Artificial (IA) tengan un contexto de lectura acotado y seguro (por Slices), sin acoplamiento físico de código y sin caer en vicios de la programación orientada a objetos (POO).

---

## Perfiles Progresivos: Lite → Standard → Strict

DDS se propone como un **criterio portable de decisión**, no como una ceremonia rígida. El mismo principio — *separar Datos (lo conocido) de Decisión (lo calculado) de Delivery (lo que modifica el mundo)* — se considera una hipótesis que vale la pena probar tanto en un CRUD simple como en una plataforma de facturación multi-equipo. Lo que cambia es **cuánta estructura física** exige esa separación, y eso lo determina el **riesgo, no el dogma**. (El alcance amplio de su aplicabilidad es una hipótesis a validar, no un hecho probado; ver [Prospecto Medible](#prospecto-medible-hipótesis-a-validar).)

> [!IMPORTANT]
> **Este documento describe el perfil DDS Strict por defecto** (la forma más fuerte: ensamblados separados, visibilidad `internal`, compilador como aduana). Los perfiles Lite y Standard son opciones legítimas de primera clase, no un "Strict incompleto".

| Perfil | Estructura | Cuándo usarlo | Frontera |
| :--- | :--- | :--- | :--- |
| **DDS Lite** | Separación conceptual `Data → Decision → Delivery`. Archivos mínimos, **sin** ensamblados separados. | CRUD simple, servicios pequeños, prototipos, dominios de bajo riesgo. | Solo convención. La extracción de `Decision` pura se aplica **solo cuando reduce riesgo**. |
| **DDS Standard** | Archivos explícitos `contracts / engine / delivery` con flujo de datos claro. | Lógica de negocio con reglas reales, suites de tests en crecimiento, o exploración de IA que se beneficia de contexto acotado. | Convención + archivos claros. Ensamblados separados son **recomendados, no obligatorios**. |
| **DDS Strict** | Fronteras físicas: un ensamblado por slice, visibilidad `internal`, aislamiento reforzado por compilador. | Proyectos de alto riesgo / extremos, trabajo multi-equipo o multi-agente donde la convención es insuficiente. | **Reforzada por compilador.** |

### Reglas de Promoción (por síntomas, no por calendario)

La promoción **solo** se justifica ante dolor o riesgo concretos. **Si no hay dolor, no se promueve.**

| Transición | Promover cuando se observe… |
| :--- | :--- |
| **Lite → Standard** | Decisión / datos / efectos ya no caben claramente en una unidad simple · los tests mezclan I/O con reglas · la IA debe leer demasiado para actuar · la delivery comienza a contaminar la lógica de decisión. |
| **Standard → Strict** | Aparecen imports cruzados o violaciones de frontera recurrentes · el radio de explosión del refactor crece sin control · la coordinación multi-agente o multi-equipo requiere refuerzo del compilador. |

### Nomenclatura y Tamaño de Slice

Un slice representa una **razón de cambio / capacidad**, no necesariamente una tabla o entidad.

| Contexto | Granularidad típica de slice |
| :--- | :--- |
| CRUD simple | `ProductManagement`, `CustomerManagement` son válidos. |
| Dominios de negocio | `Inventory`, `Sales`, `Billing` suelen ser mejores que `Customer` / `Product` / `Order` cuando esas entidades son solo datos compartidos dentro de una capacidad mayor. |
| Flujos muy complejos | Sub-slices / slices por caso de uso como `ApproveRefund`, `CancelSale` pueden justificarse. |

> Una entidad por slice **no es automáticamente incorrecta**; simplemente **no es automática**. Se dimensiona el slice por capacidad y radio de explosión, no por cantidad de tablas.

---

## Prospecto Medible: Hipótesis a Validar

DDS se publica como un **prospecto / manifiesto de criterios de diseño e hipótesis medibles**, no como un framework universal validado. El *criterio* aspira a ser aplicable en múltiples dominios; la *evidencia* de sus beneficios está pendiente. Esta sección declara qué propone DDS, qué **no** afirma todavía y qué debe medirse antes de promocionar una hipótesis a resultado. (Versión en inglés: [Measurable Prospectus](../../README.md#dds-as-a-measurable-prospectus-not-a-proven-framework).)

### Qué propone DDS
- Un **criterio de decisión**: separar **Datos** (lo conocido) de **Decisión** (lo calculado) de **Delivery** (lo que modifica el mundo).
- Una **escala de tres perfiles guiada por riesgo** (Lite / Standard / Strict) para que la estructura física sea proporcional al riesgo, no al dogma.
- Una **postura Zero-Mocks** para la capa `Decision` / `Engine`, con dobles de prueba de comportamiento observable permitidos para `Delivery`.
- **Inyección en SDD** como criterio arquitectónico dentro de Design / Apply / Verify — DDS **no agrega fases SDD**.

### Qué DDS NO afirma todavía
- **No** afirma reducciones universales y probadas en costo de tokens, costo de exploración de la IA, tasa de regresión o radio de explosión del refactor.
- **No** afirma que los perfiles sean prescripciones óptimas: son **recomendaciones a poner a prueba**, ordenadas por riesgo observado y no por resultados medidos.
- **No** afirma que Zero-Mocks sea universalmente superior a los mocks controlados; es solo la postura *propuesta* para los Engines puros, a validar empíricamente.
- **No** afirma que DDS reemplace a la POO o a Clean Architecture como regla general. Postula a DDS como un *candidato* mejor adaptado al desarrollo agéntico, a comparar con alternativas.

### Qué debe medirse
| Dimensión | Métrica a capturar | Hipótesis a contrastar |
| :--- | :--- | :--- |
| Costo de tokens / contexto | Tokens leídos y escritos por tarea | DDS *reduce* los tokens que el agente debe cargar. |
| Archivos explorados | Cantidad de archivos abiertos por cambio | El aislamiento por slice *acota* la exploración a un slice. |
| Tiempo de tests | Duración de la suite de tests del Engine | Los Engines Zero-Mocks corren *en milisegundos*. |
| Radio de explosión del refactor | Archivos tocados por cambio de comportamiento | El aislamiento *contiene* el radio de explosión. |
| Tasa de regresión | Defectos introducidos por cambio | Menos regresiones *entre slices*. |
| Sobrecarga de setup | Tiempo / archivos para iniciar un slice nuevo | Los perfiles mantienen el inicio *proporcional al riesgo*. |
| Sobrecarga cognitiva | Esfuerzo del revisor por cambio (heurística) | Los slices lineales son *más baratos de revisar*. |
| Promoción de perfil | Frecuencia de movimientos Lite → Standard → Strict | Las promociones siguen *dolor real*, no moda. |

### Cómo se validarán o falsarán las afirmaciones
Cada **ejemplo trabajado** y cada **benchmark** futuro publicará sus mediciones crudas contra la tabla anterior. Una hipótesis se considera **respaldada** solo cuando ejemplos repetidos reproducen la dirección y magnitud esperadas; se considera **falsada** cuando los ejemplos la contradicen, y la recomendación correspondiente se **degrada o elimina** de DDS.

---

### 1. Filosofía de Diseño: DoD Puro en Slices

Para evitar que la IA caiga en abstracciones innecesarias o modifique código colateral de otros módulos, estructuramos el sistema bajo los siguientes axiomas:

1.  **Aislamiento Termodinámico:** Un Slice nunca importa código de implementación de otro Slice. La comunicación es estrictamente a través de declaraciones públicas expuestas en el archivo de contratos del Slice destino.
2.  **Cero Vicios de la POO:**
    *   No hay herencia de clases de persistencia ni constructores para inyectar dependencias.
    *   No hay interfaces con tablas virtuales (`virtual` / `vtable`) que añadan costo de runtime o acoplamiento dinámico.
    *   Se prefieren funciones libres en espacios de nombres (namespaces) sobre clases con métodos estáticos.
3.  **Composición Secuencial de Pipelines:** Los motores (`engines`) son funciones estáticas puras y ciegas entre sí. La lógica de negocio no invoca dependencias de otros slices. El orquestador ejecuta los sistemas secuencialmente pasando estructuras de datos planas (PODs) de un paso al siguiente.
4.  **Lote de Datos y Persistencia:** El orquestador abre la transacción de base de datos e invoca a las funciones de persistencia de cada actor de forma procedimental, pasando un manejador de transacción física para preservar la atomicidad física.

> [!NOTE]
> Para una discusión profunda sobre la segregación de DbContexts, la duplicación controlada y el acoplamiento de base de datos en DoD, consultá el documento auxiliar de [Diseño de Persistencia Segregada](DDS_Persistencia_Segregada.md).
> 
> Para la implementación de este estándar en hardware con recursos severamente limitados (como microcontroladores ESP8266/ESP32, STM32) utilizando interrupciones físicas y cero heap allocation, consultá el documento auxiliar de [DDS en Sistemas Embebidos](DDS_Sistemas_Embebidos.md).

---

### 2. Distribución de Archivos y Reglas de Visibilidad

> [!NOTE]
> La estructura de **tres archivos obligatorios por slice** que se describe a continuación corresponde al perfil **DDS Standard / Strict**. En **DDS Lite**, basta con mantener la separación conceptual `Data → Decision → Delivery` (por ejemplo, una función pura de decisión y un adaptador de I/O en el mismo módulo); los archivos se separan solo cuando la legibilidad o el riesgo lo justifican.

Cada Slice se organiza en tres archivos. La visibilidad de los archivos limita qué información puede consumir la IA:

```text
/src
├── /core                        <-- Mínimo Core de Infraestructura (casi vacío: handles opacos de transacción, primitivas técnicas verdaderamente compartidas)
└── /slices
    ├── /inventory
    │   ├── contracts.XX         <-- PÚBLICO: Estructuras de datos (PODs) y envoltorios de contratos de persistencia/motores
    │   ├── engine.XX            <-- PRIVADO: Motor de lógica pura (estático, internal)
    │   └── delivery.XX          <-- PRIVADO: Código físico de base de datos (implementación, internal)
    ├── /finance
    │   ├── contracts.XX         <-- PÚBLICO: Estructuras de datos (PODs) y firmas/delegados de persistencia
    │   ├── engine.XX            <-- PRIVADO: Motor de lógica pura (estático, internal)
    │   └── delivery.XX          <-- PRIVADO: Código físico de base de datos (implementación, internal)
    └── /sales
        ├── contracts.XX         <-- PÚBLICO: Contratos de ventas
        ├── engine.XX            <-- PRIVADO: Motor de orquestación pura (estático, internal)
        └── delivery.XX          <-- PRIVADO: Punto de entrada, conexión SQL y transacciones
```

*   **Público (contracts.XX):** Solo contiene firmas de funciones, delegados y estructuras planas. Es lo único que otros Slices pueden leer.
*   **Privado (engine.XX y delivery.XX):** Contiene la lógica interna de cálculo y el SQL físico de escritura. Se declaran como componentes internos (`internal` / compilación separada) para que ningún otro Slice pueda instanciarlos o invocarlos directamente.

#### ⚠️ Regla de Compilación Crítica (Frontera Física — solo perfil Strict):
Para que la visibilidad `internal` de C# o la separación de unidades de compilación en C++ sea real, **cada Slice debe compilarse como un ensamblado físico independiente** (un proyecto `.csproj` de tipo Class Library en C#, o una librería estática/dinámica en C++). Si todos los slices compilan dentro del mismo proyecto monolítico, la IA podrá saltarse la frontera física y violar el aislamiento termodinámico.

> Esta regla **solo aplica al perfil Strict**. En **Lite** y **Standard**, el aislamiento se logra por **convención y layout de archivos**; no se requiere un ensamblado por slice. Se promueve a Strict únicamente cuando los síntomas de riesgo (imports cruzados, violaciones de frontera, radio de explosión del refactor) lo justifiquen.

#### Evolución de Contratos Públicos

Un contrato público (`contracts.XX`) que se consume fuera de su slice **no debe cambiarse de forma silenciosa**. Cuando un contrato es consumido por otros slices o por sistemas externos:

- **Versionar / adaptar / migrar** el contrato en lugar de romperlo.
- Mantener la retrocompatibilidad o coordinar el cambio explícito con los consumidores.
- Los DTOs y modelos de negocio o de flujo **no deben colocarse en `/core` por defecto** (ver regla de `/core` más abajo).

#### Regla de `/core` (Solo infraestructura)

> **`/core` está vacío hasta que se demuestre que es infraestructura.**

El directorio `/core` es de **infraestructura**, no de dominio. Debe ser **aburrido y casi vacío**. La regla por defecto es **no** poner nada ahí: nada de IDs, ni Dinero, ni modelos de negocio, ni tipos de contrato. Esos viven en el slice o grupo de contratos donde su semántica es explícita.

| Pertenece a `/core` (excepcional, solo infraestructura) | **No** pertenece a `/core` por defecto (vive en el slice/contrato) |
| :--- | :--- |
| Handles opacos de transacción (p. ej. `DbTransaction` forward-declared). | IDs de negocio (`CustomerId`, `ProductId`, `InvoiceId`, `ProductCatalogId`). |
| Primitivas técnicas verdaderamente neutrales y compartidas **solo si** su semántica es idéntica en todos los slices (p. ej. un `Result<T,E>` de error puramente técnico). | Dinero y montos (`Money`, `BillingAmount`, `LedgerAmount`, `CashRegisterBalance`, saldo de cuenta, total de factura). |
| Abstracciones de plataforma de bajo nivel (drivers, logging, conexión). | DTOs, modelos de producto/cliente/pedido, estructuras de flujo de un slice. |
| | Contratos de persistencia específicos, reglas y lógica de negocio. |

**Por qué IDs y Dinero no son de `/core`:** En un ERP, `Money` puede significar saldo de cuenta, monto de caja, asiento contable, total de factura o monto financiero — semánticas que pertenecen a slices, contratos públicos o grupos lógicos de features, no a un núcleo global. Lo mismo aplica a los identificadores: un `InvoiceId` y un `ProductCatalogId` son conceptos de slice. **Nada relacionado con un producto existe hasta que un slice lo usa y lo gestiona.**

**Cuándo promover un tipo a `/core`:** Es la excepción, no la regla. Exige prueba de que el tipo es **puramente técnico** (o semánticamente idéntico en todos los slices) **y** que compartirlo reduce el riesgo más de lo que aumenta el acoplamiento. Si un tipo de negocio "parece compartido", preferir **versionar/adaptar el contrato público** del slice dueño antes que moverlo a `/core`.

---

### 3. Implementación de Referencia en C++20

En C++, el aislamiento se resuelve con la separación clásica de cabeceras (`.hpp` públicas) y definiciones (`.cpp` privadas). El linker de C++ se encarga de unir las piezas en tiempo de compilación. Las variables de dinero se representan como enteros de 64 bits (`int64_t`) para evitar errores de precisión de punto flotante.

#### 3.1. Core de Infraestructura (Mínimo y Compartido)

Para mantener la seguridad de tipos sin acoplar los slices a un driver de base de datos específico (como SQLite o PostgreSQL), se define una estructura opaca para la transacción:

```cpp
// core/db_types.hpp (PÚBLICO)
#pragma once

namespace core {
    // Estructura opaca declarada hacia adelante (forward-declared)
    // Permite pasar punteros a la transacción sin revelar la base de datos subyacente
    struct DbTransaction; 
}
```

#### 3.2. Slice de Inventario (Satélite)

```cpp
// slices/inventory/contracts.hpp (PÚBLICO)
#pragma once
#include <cstdint>
#include "../../core/db_types.hpp"

namespace slices::inventory {

// Datos planos
struct StockUpdate {
    int64_t item_id; // Signed int64 para evitar desbordamientos y problemas de persistencia
    int32_t new_qty;
};

struct StockCheck {
    bool is_valid;
    int32_t next_qty;
};

// Declaración pública del cálculo puro (definido en engine.cpp)
StockCheck calculate_new_stock(int32_t current_qty, int32_t sell_qty);

// Declaración pública de la persistencia (definido en delivery.cpp)
void update_stock_in_db(const StockUpdate& update, core::DbTransaction* tx);

} // namespace slices::inventory
```

```cpp
// slices/inventory/engine.cpp (PRIVADO)
#include "contracts.hpp"

namespace slices::inventory {

StockCheck calculate_new_stock(int32_t current_qty, int32_t sell_qty) {
    if (current_qty < sell_qty) {
        return StockCheck{ false, current_qty };
    }
    return StockCheck{ true, current_qty - sell_qty };
}

} // namespace slices::inventory
```

```cpp
// slices/inventory/delivery.cpp (PRIVADO)
#include "contracts.hpp"
#include <iostream>

namespace slices::inventory {

void update_stock_in_db(const StockUpdate& update, core::DbTransaction* tx) {
    // El puntero opaco se utiliza de forma segura dentro del adaptador de infraestructura
    std::cout << "[SQL-INV] UPDATE Stocks SET Qty = " << update.new_qty 
              << " WHERE ItemId = " << update.item_id 
              << " (Tx: " << tx << ")\n";
}

} // namespace slices::inventory
```

#### 3.3. Slice de Finanzas (Satélite)

```cpp
// slices/finance/contracts.hpp (PÚBLICO)
#pragma once
#include <cstdint>
#include "../../core/db_types.hpp"

namespace slices::finance {

struct LedgerEntry {
    int32_t account_id;
    int64_t amount;
};

// Declaración pública del cálculo puro
LedgerEntry generate_entry(int32_t account_id, int64_t amount);

// Declaración pública de la persistencia
void write_ledger_to_db(const LedgerEntry& entry, core::DbTransaction* tx);

} // namespace slices::finance
```

```cpp
// slices/finance/engine.cpp (PRIVADO)
#include "contracts.hpp"

namespace slices::finance {

LedgerEntry generate_entry(int32_t account_id, int64_t amount) {
    return LedgerEntry{ account_id, amount };
}

} // namespace slices::finance
```

```cpp
// slices/finance/delivery.cpp (PRIVADO)
#include "contracts.hpp"
#include <iostream>

namespace slices::finance {

void write_ledger_to_db(const LedgerEntry& entry, core::DbTransaction* tx) {
    std::cout << "[SQL-FIN] INSERT INTO LedgerEntries (AccountId, Amount) VALUES (" 
              << entry.account_id << ", " << entry.amount << ") (Tx: " << tx << ")\n";
}

} // namespace slices::finance
```

#### 3.4. Slice de Ventas (Orquestador)

```cpp
// slices/sales/contracts.hpp (PÚBLICO)
#pragma once
#include <cstdint>

namespace slices::sales {

struct SaleRequest {
    int64_t item_id; // Signed int64 uniforme
    int32_t qty;
    int64_t price; // Precio en centavos
    int32_t account_id; // Cuenta contable para finanzas
};

struct SaleRecord {
    int64_t sale_id; // Identificador unificado como int64_t
    int64_t total; // Total en centavos
};

// Buffers de salida resultantes (exclusivos de ventas para evitar acoplamiento de de tipos)
struct SaleOutputData {
    SaleRecord sale;
    bool success;
};

// Declaración pública del punto de entrada del orquestador
void execute_sale(const SaleRequest& req);

} // namespace slices::sales
```

```cpp
// slices/sales/engine.hpp (PRIVADO)
#pragma once
#include "contracts.hpp"
#include "../inventory/contracts.hpp"

namespace slices::sales {
    // Composición procedural pura mediante funciones libres en namespace
    SaleOutputData process_sale(const SaleRequest& req, const slices::inventory::StockCheck& stock_check);
}
```

```cpp
// slices/sales/engine.cpp (PRIVADO)
#include "engine.hpp"

namespace slices::sales {

SaleOutputData process_sale(const SaleRequest& req, const slices::inventory::StockCheck& stock_check) {
    if (!stock_check.is_valid) {
        return SaleOutputData{ {}, false };
    }

    return SaleOutputData{
        SaleRecord{ 9999, req.qty * req.price },
        true
    };
}

} // namespace slices::sales
```

```cpp
// slices/sales/delivery.cpp (PRIVADO - Orquestador)
#include "contracts.hpp"
#include "engine.hpp"
#include "../inventory/contracts.hpp"
#include "../finance/contracts.hpp"
#include "../../core/db_types.hpp"
#include <iostream>
#include <stdexcept>

namespace slices::sales {

// En C++ de producción se usaría un objeto RAII para manejar el rollback automático de la transacción.
// Aquí se muestra un flujo procedimental simplificado con try-catch.
void execute_sale(const SaleRequest& req) {
    core::DbTransaction* fake_sql_transaction = nullptr; // Uso de nullptr para evitar warning de casteo en 64-bit
    std::cout << "[SQL-SALES] BEGIN TRANSACTION (Tx: " << fake_sql_transaction << ")\n";

    try {
        // 1. Obtener datos planos iniciales de la base de datos (I/O)
        int32_t current_stock = 100; 

        // 2. Ejecutar Sistema 1: Inventario (Lógica pura estática usando la firma pública)
        auto stock_check = slices::inventory::calculate_new_stock(current_stock, req.qty);

        // 3. Ejecutar Sistema 2: Finanzas (Lógica pura estática usando la firma pública)
        auto ledger = slices::finance::generate_entry(req.account_id, req.qty * req.price);

        // 4. Ejecutar Sistema 3: Ventas (Lógica pura estática)
        auto result = process_sale(req, stock_check);

        if (!result.success) {
            throw std::runtime_error("Lógica de negocio fallida (Stock insuficiente).");
        }

        // 5. Persistencia física atómica delegada sobre la misma transacción
        std::cout << "[SQL-SALES] INSERT INTO Sales (Total) VALUES (" << result.sale.total << ")\n";
        
        // Mapeo físico local de contratos en la capa delivery (DoD)
        slices::inventory::StockUpdate stock_update{ req.item_id, stock_check.next_qty };

        // Invocación directa a las funciones planas declaradas en los contratos públicos
        slices::inventory::update_stock_in_db(stock_update, fake_sql_transaction);
        slices::finance::write_ledger_to_db(ledger, fake_sql_transaction);

        std::cout << "[SQL-SALES] COMMIT TRANSACTION (Tx: " << fake_sql_transaction << ")\n";
    }
    catch (const std::exception& e) {
        // Rollback seguro ante fallos físicos o excepciones estándar
        std::cout << "[SQL-SALES] ROLLBACK TRANSACTION (Tx: " << fake_sql_transaction << "). Motivo: " << e.what() << "\n";
        throw; // Propagación
    }
    catch (...) {
        // Asegura que cualquier tipo de excepción desencadene el rollback
        std::cout << "[SQL-SALES] ROLLBACK TRANSACTION (Tx: " << fake_sql_transaction << "). Motivo: Excepción desconocida\n";
        throw;
    }
}

} // namespace slices::sales
```

---

### 4. Implementación de Referencia en C# (.NET 8+)

En C#, las declaraciones de firmas y los adaptadores internos se desacoplan de forma nativa a nivel de compilación mediante proyectos separados. El contrato expone métodos estáticos de redirección pública que llaman a clases internas (`internal`) del mismo ensamblado, ofreciendo validación en tiempo de compilación y eliminando el uso de delegados mutables de runtime.

#### 4.1. Slice de Inventario (Satélite)

```csharp
// slices/inventory/contracts.cs (PÚBLICO - Compila en Inventory.csproj)
using System;
using System.Data;

namespace Slices.Inventory;

// Datos planos (DoD)
public readonly record struct StockUpdate(long ItemId, int Qty);
public readonly record struct StockCheck(bool IsValid, int NextQty);

// Puntos de entrada del contrato público
public static class InventoryEngine
{
    // Redirección directa al motor interno. Es seguro porque compilan en el mismo ensamblado (Inventory.csproj).
    // Otros ensamblados (como Sales) no pueden ver ni omitir la capa interna porque InventoryEngineImpl es 'internal'.
    public static StockCheck CalculateStock(int currentQty, int sellQty) => 
        InventoryEngineImpl.CalculateNewStock(currentQty, sellQty);
}

public static class InventoryDb
{
    public static void UpdateStock(StockUpdate update, IDbTransaction transaction) => 
        InventoryDelivery.PersistStockToSql(update, transaction);
}
```

```csharp
// slices/inventory/engine.cs (PRIVADO - Compila en Inventory.csproj)
namespace Slices.Inventory;

// Marcado como internal para evitar acceso externo directo desde otros ensamblados
internal static class InventoryEngineImpl
{
    internal static StockCheck CalculateNewStock(int currentQty, int sellQty)
    {
        if (currentQty < sellQty)
        {
            return new StockCheck(false, currentQty);
        }
        return new StockCheck(true, currentQty - sellQty);
    }
}
```

```csharp
// slices/inventory/delivery.cs (PRIVADO - Compila en Inventory.csproj)
using System;
using System.Data;

namespace Slices.Inventory;

internal static class InventoryDelivery
{
    internal static void PersistStockToSql(StockUpdate update, IDbTransaction transaction)
    {
        if (transaction == null) throw new ArgumentNullException(nameof(transaction));
        if (transaction.Connection == null) throw new InvalidOperationException("La conexión es nula.");

        using (var command = transaction.Connection.CreateCommand())
        {
            command.Transaction = transaction;
            command.CommandText = "UPDATE Stocks SET Qty = @qty WHERE ItemId = @id";
            
            var pQty = command.CreateParameter();
            pQty.ParameterName = "@qty";
            pQty.Value = update.Qty;
            command.Parameters.Add(pQty);

            var pId = command.CreateParameter();
            pId.ParameterName = "@id";
            pId.Value = update.ItemId; 
            command.Parameters.Add(pId);

            command.ExecuteNonQuery();
            Console.WriteLine($"[SQL-INV] Actualizado stock del item {update.ItemId} a {update.Qty} en base de datos.");
        }
    }
}
```

#### 4.2. Slice de Finanzas (Satélite)

```csharp
// slices/finance/contracts.cs (PÚBLICO - Compila en Finance.csproj)
using System;
using System.Data;

namespace Slices.Finance;

// Datos planos
public readonly record struct LedgerEntry(int AccountId, long Amount); // Dinero en centavos

// Redirecciones directas a las implementaciones internas del slice
public static class FinanceEngine
{
    public static LedgerEntry GenerateEntry(int accountId, long amount) => 
        FinanceEngineImpl.GenerateEntryImpl(accountId, amount);
}

public static class FinanceDb
{
    public static void WriteLedger(LedgerEntry entry, IDbTransaction transaction) => 
        FinanceDelivery.PersistLedgerToSql(entry, transaction);
}
```

```csharp
// slices/finance/engine.cs (PRIVADO - Compila en Finance.csproj)
namespace Slices.Finance;

internal static class FinanceEngineImpl
{
    internal static LedgerEntry GenerateEntryImpl(int accountId, long amount)
    {
        return new LedgerEntry(accountId, amount);
    }
}
```

```csharp
// slices/finance/delivery.cs (PRIVADO - Compila en Finance.csproj)
using System;
using System.Data;

namespace Slices.Finance;

internal static class FinanceDelivery
{
    internal static void PersistLedgerToSql(LedgerEntry entry, IDbTransaction transaction)
    {
        if (transaction == null) throw new ArgumentNullException(nameof(transaction));
        if (transaction.Connection == null) throw new InvalidOperationException("La conexión es nula.");

        using (var command = transaction.Connection.CreateCommand())
        {
            command.Transaction = transaction;
            command.CommandText = "INSERT INTO LedgerEntries (AccountId, Amount) VALUES (@acc, @amount)";
            
            var pAcc = command.CreateParameter();
            pAcc.ParameterName = "@acc";
            pAcc.Value = entry.AccountId;
            command.Parameters.Add(pAcc);

            var pAmount = command.CreateParameter();
            pAmount.ParameterName = "@amount";
            pAmount.Value = entry.Amount;
            command.Parameters.Add(pAmount);

            command.ExecuteNonQuery();
            Console.WriteLine($"[SQL-FIN] Insertado registro contable bajo la transacción compartida.");
        }
    }
}
```

#### 4.3. Slice de Ventas (Orquestador)

```csharp
// slices/sales/contracts.cs (PÚBLICO - Compila en Sales.csproj)
namespace Slices.Sales;

public readonly record struct SaleRequest(long ItemId, int Qty, long Price, int AccountId);
public readonly record struct SaleRecord(long SaleId, long Total);

// Estructura de salida compuesta
public readonly record struct SaleOutputData(
    SaleRecord Sale,
    bool Success
);
```

```csharp
// slices/sales/engine.cs (PRIVADO - Compila en Sales.csproj)
using Slices.Inventory;

namespace Slices.Sales;

internal static class SalesEngine
{
    public static SaleOutputData ProcessSale(SaleRequest req, StockCheck stockCheck)
    {
        if (!stockCheck.IsValid)
        {
            return new SaleOutputData(default, false);
        }

        return new SaleOutputData(
            new SaleRecord(9999, req.Qty * req.Price), // ID y cálculos alineados
            true
        );
    }
}
```

```csharp
// slices/sales/delivery.cs (PÚBLICO - Compila en Sales.csproj)
using System;
using System.Data;
using Microsoft.Data.SqlClient;
using Slices.Inventory;
using Slices.Finance;

namespace Slices.Sales;

// Declarado como público para permitir al host principal invocar el caso de uso
public static class SalesDelivery
{
    public static void ExecuteSale(SaleRequest req, string connString)
    {
        using (var connection = new SqlConnection(connString))
        {
            connection.Open();
            using (var tx = connection.BeginTransaction())
            {
                try
                {
                    // 1. Obtener estado inicial de la base de datos (I/O)
                    int currentStock = 100; // Mismo mock inicial que C++

                    // 2. Ejecutar Sistema 1: Inventario (vía contrato público resuelto en compile-time)
                    StockCheck stockCheck = InventoryEngine.CalculateStock(currentStock, req.Qty);

                    // 3. Ejecutar Sistema 2: Finanzas (vía contrato público resuelto en compile-time)
                    LedgerEntry ledger = FinanceEngine.GenerateEntry(req.AccountId, req.Qty * req.Price);

                    // 4. Ejecutar Sistema 3: Ventas (DoD Puro, ciego a otros slices)
                    var result = SalesEngine.ProcessSale(req, stockCheck);

                    if (!result.Success)
                    {
                        throw new InvalidOperationException("Regla de negocio fallida (Stock insuficiente).");
                    }

                    // 5. Escribir datos locales (Ventas)
                    using (var cmd = connection.CreateCommand())
                    {
                        cmd.Transaction = tx;
                        cmd.CommandText = "INSERT INTO Sales (Total) VALUES (@total)";
                        
                        var pTotal = cmd.CreateParameter();
                        pTotal.ParameterName = "@total";
                        pTotal.Value = result.Sale.Total;
                        cmd.Parameters.Add(pTotal);

                        cmd.ExecuteNonQuery();
                    }

                    // 6. Mapear buffers físicos de cambios locales en delivery (DoD)
                    StockUpdate stockUpdate = new StockUpdate(req.ItemId, stockCheck.NextQty);

                    // 7. Delegar escrituras a otros slices mediante llamadas directas a contratos públicos
                    // Todo se ejecuta sobre la misma conexión y transacción de SQL
                    InventoryDb.UpdateStock(stockUpdate, tx); 
                    FinanceDb.WriteLedger(ledger, tx);        

                    tx.Commit();
                    Console.WriteLine("[SQL-SALES] Venta procesada y transaccionada de manera exitosa.");
                }
                catch (Exception ex)
                {
                    try
                    {
                        tx.Rollback();
                    }
                    catch (Exception rollbackEx)
                    {
                        Console.WriteLine($"[SQL-SALES] Error en rollback secundario: {rollbackEx.Message}");
                    }
                    throw; // Propaga el error original
                }
            }
        }
    }
}
```

---

### 5. Método de Trabajo para Agentes de IA (SDD + TDD)

DDS **no modifica el flujo SDD ni agrega pasos**. Se inyecta como **criterio arquitectónico** dentro de las fases existentes de SDD, una vez definida la idea:

| Fase SDD | Qué aporta DDS |
| :--- | :--- |
| **Design / Spec (planificación)** | Elegir el **perfil** (Lite / Standard / Strict), definir el **límite y nombre del slice**, fijar la **forma del contrato** y decidir la **estrategia TDD**. |
| **Apply** | Implementar con el perfil seleccionado y mantener la separación `Data / Decision / Delivery` **proporcionada** al riesgo del slice. |
| **Verify** | Verificar tests, fronteras, radio de explosión del refactor y confirmar que la **elección de perfil fue proporcional** al riesgo real. |

#### Contención de contexto durante Apply

Cuando el agente de IA trabaje en una tarea de desarrollo, se **busca** la contención de contexto del siguiente modo:

1.  **Contexto Acotado:** El agente solo tiene permitido leer y modificar los archivos dentro de la carpeta de su Slice destino (su ensamblado local en Strict, o su módulo/archivos en Lite/Standard).
2.  **Consumo de Dependencias:** Si requiere coordinarse con otro Slice, la IA solo lee el archivo público `contracts.XX` de dicho Slice (o, en Lite, la mínima firma pública necesaria).
3.  **Compilación como Aduana (Strict):** Si la IA modifica una firma de función o de datos en `contracts.XX`, el compilador nativo interrumpe la construcción del proyecto inmediatamente por inconsistencia de tipos, permitiendo resolver el impacto del contrato sin problemas en tiempo de ejecución. En Lite/Standard, esta aduana la cumple la **convención** hasta que el riesgo justifique promover a Strict.
4.  **Zero-Mocks Suite (TDD Puro):** Pruebas unitarias directas sobre las funciones puras de `engine.XX` sin mocks dinámicos de base de datos. Las pruebas de Delivery pueden usar test doubles / fakes / testcontainers controlados cuando validan **comportamiento observable** (no coreografía interna de llamadas). Ver [TDD Sin Mocks](DDS_TDD_Sin_Mocks.md).
