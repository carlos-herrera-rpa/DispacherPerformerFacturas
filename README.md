# Enterprise Invoice Processing Automation (Dispatcher & Performer Architecture)
## Scalable UiPath RPA Solution Built on Robotic Enterprise Framework (REFramework)

## 📌 Project Overview
This enterprise-grade RPA solution automates the end-to-end ingestion, validation, and status-routing of pending corporate invoices. Operating under a decoupled **Dispatcher and Performer architecture**, the system utilizes **UiPath Orchestrator Queues** as a secure transactional buffer. This approach ensures maximum scalability, high availability, and isolated component maintenance.

The transactional business logic handles currency thresholds dynamically, routing high-value accounts for human validation while simulating instant clearance for standard operations.

## 🛠️ Technical Stack & Architecture
* **RPA Architecture:** Enhanced REFramework (Robotic Enterprise Framework)
* **Model Pattern:** Queue-Driven Transactional Processing (Dispatcher / Performer split)
* **Platform:** UiPath Studio (Windows / Modern Experience)
* **Data Sources:** Structured Excel Ledgers & Orchestrator Cloud Assets
* **Core Dependencies:**
  * `UiPath.Excel.Activities` (Modern Workbook & Excel Integration)
  * `UiPath.System.Activities` (Orchestrator Queue Handling)

## 🔄 Execution Architecture & Data Flow

### 1. The Dispatcher (Ingestion Phase)
* **Trigger:** Initialized within the production scheduler.
* **Mechanism:** Reads the source pending ledger (`FacturasPendientes.xlsx`), processes rows into atomic structured data points, and streams them securely into the cloud-hosted `FacturasQueue`.
* **Data Payload:** Maps fields cleanly into the transaction metadata (`FacturaID`, `Cliente`, `Monto`, `FechaVencimiento`) preventing any data corruption.

### 2. The Performer (Processing Phase - REFramework)
The Performer acts as the transactional core engine, strictly bounded by the standard REFramework states:
* **Initialization (Init):** Reads the system configuration dictionary (`Config.xlsx`), initializes environment states, and executes system kill-routines (`KillAllProcesses.xaml`) to ensure clean execution.
* **Get Transaction Data:** Fetches individual `QueueItem` instances directly from the Orchestrator Queue, enforcing transactional locking.
* **Process Transaction:** Executes the underlying business rules:
  * **Rule A (Approval Route):** If `Monto > 1,000`, flags the invoice metadata as `"Requiere aprobación"`.
  * **Rule B (Automated Route):** If `Monto <= 1,000`, flags the record as `"Pago automático simulado"`.
* **End Process:** Persists the results dynamically by compiling an analytical multi-column Excel tracking report (`ResultadosFacturas.xlsx`).

## 🛡️ Exception Management & Fault Tolerance
* **System Exceptions (SE):** Any infrastructure disruption (e.g., loss of cloud connectivity or file access locks) triggers an automatic application retry. The queue is configured for **up to 2 automated retries** before transitioning the item state to `Failed`.
* **Business Rule Exceptions (BRE):** Data anomalies are caught instantly, logged, and isolated without halting the global processing queue.

## 📂 Data Structure Model

### Input Data Profile (`FacturasPendientes.xlsx`)
| FacturaID | Cliente | Monto | FechaVencimiento |
| :--- | :--- | :--- | :--- |
| F002 | Santiago López | 1200 | 2025-12-15 |
| F003 | Laura Gómez | 450 | 2025-12-08 |

### Processed Output Structure (`ResultadosFacturas.xlsx`)
| FacturaID | Cliente | Monto | FechaVencimiento | TipoPago |
| :--- | :--- | :--- | :--- | :--- |
| F002 | Santiago López | 1200 | 2025-12-15 | Requiere aprobación |
| F003 | Laura Gómez | 450 | 2025-12-08 | Pago automático simulado |
