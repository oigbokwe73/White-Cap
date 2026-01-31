# White-Cap


Absolutely — here’s a **full Azure procurement integration architecture** that fits your pattern: **Azure Functions + Service Bus + Azure SQL + Power BI**, with proper **reliability, traceability, and job-cost reporting**.

---

## Azure Procurement Integration Architecture

### What this solves

* Ingest POs / receipts / invoices from ERP + suppliers (like White Cap)
* Normalize + validate (job, cost code, vendor, pricing)
* Persist to **Azure SQL** for finance/job costing
* Stream operational events + exceptions through **Service Bus**
* Drive **Power BI** spend + WIP + vendor performance dashboards

---

## Architecture Diagram

```mermaid
flowchart LR
  %% =========================
  %% Actors / Sources
  %% =========================
  ERP[Construction ERP\nProcore/Sage/CMiC/Viewpoint] -->|PO / Receipt / Vendor Master| API[ERP Export/API/Webhook]
  SUP[Supplier White Cap / Others] -->|Invoice PDF / EDI 810 / ASN| IN[Inbound Channel]

  %% =========================
  %% Inbound Channels
  %% =========================
  subgraph Inbound["Inbound Channels"]
    API --> AF1[Azure Function: Ingest-ERP]
    IN --> SA[Blob Storage\nInbound Docs]
    IN --> SBQ[Service Bus Queue\nsupplier-events]
  end

  %% =========================
  %% Orchestration / Validation
  %% =========================
  subgraph Orchestration["Orchestration + Validation"]
    AF1 --> SBPO[Service Bus Queue po-events]
    AF2[Azure Function: Ingest-Supplier] --> SBQ
    SBPO --> AF3[Azure Function: Validate+Normalize PO/Receipt]
    SBQ --> AF4[Azure Function: Validate+Normalize Invoice/ASN]
    AF3 --> SBT[(Service Bus Topic\nprocurement-domain)]
    AF4 --> SBT
  end

  %% =========================
  %% Persistence
  %% =========================
  subgraph Data["Data Layer"]
    SQL[Azure SQL Database\nProcurement + Job Cost]
    SA --> AF5[Azure Function: Doc Parser PDF/EDI Parser]
    AF5 --> SQL
    SBT --> AF6[Azure Function: Upsert Writer]
    AF6 --> SQL
  end

  %% =========================
  %% Exceptions / DLQ
  %% =========================
  subgraph Ops["Ops + Exceptions"]
    SBT -->|validation fail| DLQ[Service Bus DLQ]
    DLQ --> AF7[Azure Function: Exception Handler]
    AF7 --> SQLX[(Azure SQL Exceptions/Audit)]
    AF7 --> NOTIF[Teams/Email/Webhook Alert]
  end

  %% =========================
  %% Reporting
  %% =========================
  subgraph BI["BI + Analytics"]
    SQL --> PBI[Power BI\nSpend, WIP, Vendor SLA, Job Cost]
  end
```

---

## Key Flows

### 1) PO + Receipt flow (ERP → Azure)

1. ERP triggers webhook/export (PO created, PO changed, Receipt posted)
2. **Function Ingest-ERP** validates schema + adds correlation IDs
3. Publishes to **Service Bus `po-events`**
4. **Validate+Normalize** function maps:

   * Job number → Project ID
   * Cost code → GL / Phase
   * Items → standard item master / SKU mapping
5. Emits domain events to **Service Bus Topic `procurement-domain`**
6. **Upsert Writer** persists to **Azure SQL**

### 2) Supplier invoice flow (Supplier → Azure)

1. Supplier sends:

   * EDI 810 (invoice), ASN, packing slip, or PDF invoice
2. Land raw docs in **Blob Storage** + enqueue message to `supplier-events`
3. **Doc Parser** extracts line items, amounts, tax, freight (and metadata)
4. **Validate+Normalize** checks:

   * PO exists?
   * Receipt exists?
   * Vendor match?
5. Writes normalized invoice + lines into SQL, or pushes to DLQ

### 3) 3-way match (PO vs Receipt vs Invoice)

* Implement as:

  * A stored procedure in Azure SQL (fast, consistent), and/or
  * A function-based rules engine (more flexible for exceptions)

Result:

* Matched → “Approved for AP”
* Mismatch → exception record + workflow queue

---

## Core Azure Components

### Azure Functions (recommended split)

* **Ingest-ERP** (HTTP trigger)
* **Ingest-Supplier** (Service Bus trigger)
* **Doc Parser** (Blob trigger or SB trigger)
* **Validate+Normalize** (SB trigger)
* **Upsert Writer** (Topic subscription trigger)
* **Exception Handler** (DLQ trigger)

### Service Bus design

* Queues:

  * `po-events`
  * `supplier-events`
* Topic:

  * `procurement-domain`
* Subscriptions (examples):

  * `to-sql-writer`
  * `to-exceptions`
  * `to-notifications`
  * `to-analytics`

This keeps producers decoupled and lets you fan-out safely.

---

## Azure SQL Schema (practical + reporting-friendly)

**Master**

* `Vendor`
* `Project`
* `CostCode`
* `ItemCatalog` (optional)

**Transactional**

* `PurchaseOrderHeader`, `PurchaseOrderLine`
* `ReceiptHeader`, `ReceiptLine`
* `InvoiceHeader`, `InvoiceLine`

**Controls**

* `MatchResult` (status, tolerances, reason codes)
* `Exception` (type, severity, owner, SLA)
* `AuditEvent` (who/when/source/correlationId)
* `RawDocument` (Blob URI, checksum, doc type)

**Power BI Views**

* `vw_SpendByProjectCostCode`
* `vw_OpenPOCommitments`
* `vw_InvoiceAging`
* `vw_VendorOnTimeDelivery`
* `vw_ExceptionsByType`

---

## Reporting in Power BI

### Dashboards you’ll get “for free” once normalized:

* **Spend by Project / Cost Code / Vendor**
* **Committed cost** (Open POs) vs **Actuals** (Invoiced/Received)
* **3-way match rate**
* **Invoice aging** + exception backlog
* **Vendor SLA** (on-time delivery, fill rate)
* **Change order impact** (if ERP provides CO events)

---

## Security + Ops (the stuff that makes this production-grade)

### Security

* Managed Identity for Functions → SQL + Service Bus
* Private endpoints for SQL/Storage/Service Bus (if needed)
* Key Vault for secrets (only if unavoidable)
* Row Level Security in SQL for multi-entity reporting (optional)

### Observability

* Application Insights:

  * correlationId per transaction (PO → receipt → invoice)
  * dependency tracking for SQL/SB
* Log Analytics:

  * DLQ count by reason
  * processing latency and retries
* Alerts:

  * DLQ > threshold
  * function failures
  * invoice backlog SLA breach

---

## Message Contracts (simple, scalable)

Use a consistent envelope for all messages:

* `eventType` (POCreated, ReceiptPosted, InvoiceReceived, MatchCompleted)
* `sourceSystem` (ERP, WhiteCap, EDI)
* `correlationId` (same across PO/receipt/invoice chain)
* `projectId`, `poNumber`, `vendorId`
* `payloadVersion`
* `payload` (document body)

---

## Optional “Level Up” Enhancements (still fits your stack)

* Add **Azure API Management** in front of Functions if you want external partner APIs
* Add **Durable Functions** if you want long-running orchestration (e.g., wait for receipt then match invoice)
* Add **Synapse/ADF** only if you start doing heavy historical analytics or multi-ERP consolidation

Perfect — below is a **clean, enterprise-grade Mermaid sequence diagram** that mirrors how real construction procurement systems behave, while mapping **exactly to Azure Functions + Service Bus + Azure SQL**.

This is **slide-ready**, **architecture-review ready**, and matches the way finance + job-cost systems actually operate.

---

# 🧾 Procurement 3-Way Match — Sequence Diagram

```mermaid
sequenceDiagram
    autonumber

    participant ERP as Construction ERP
    participant AFI as Azure Function<br/>ERP Ingest
    participant SBP as Service Bus<br/>PO Queue
    participant AFR as Azure Function<br/>Receipt Processor
    participant SQL as Azure SQL<br/>Procurement DB
    participant SUP as Supplier<br/>(White Cap)
    participant AFS as Azure Function<br/>Supplier Ingest
    participant SBI as Service Bus<br/>Invoice Queue
    participant AFM as Azure Function<br/>3-Way Match Engine
    participant DLQ as Service Bus<br/>DLQ
    participant OPS as Exception Handler<br/>(Function)
    participant PBI as Power BI

    %% ----------------------------
    %% Purchase Order
    %% ----------------------------
    ERP->>AFI: Create Purchase Order (PO)
    AFI->>SBP: Publish PO Event
    SBP->>AFR: PO Message Trigger
    AFR->>SQL: Insert PO Header + Lines
    SQL-->>AFR: PO Stored

    %% ----------------------------
    %% Receipt
    %% ----------------------------
    ERP->>AFI: Post Receipt (Partial / Full)
    AFI->>SBP: Publish Receipt Event
    SBP->>AFR: Receipt Message Trigger
    AFR->>SQL: Insert Receipt Records
    SQL-->>AFR: Receipt Stored

    %% ----------------------------
    %% Supplier Invoice
    %% ----------------------------
    SUP->>AFS: Submit Invoice (EDI 810 / PDF)
    AFS->>SBI: Publish Invoice Event
    SBI->>AFM: Invoice Triggered
    AFM->>SQL: Load Invoice Header + Lines

    %% ----------------------------
    %% 3-Way Match
    %% ----------------------------
    AFM->>SQL: Fetch PO + Receipt + Invoice
    SQL-->>AFM: Matched Records

    alt All Values Within Tolerance
        AFM->>SQL: Update MatchStatus = Approved
        AFM->>SQL: Mark Ready for AP Posting
    else Mismatch Detected
        AFM->>SQL: Insert Exception Record
        AFM->>DLQ: Send Exception Message
    end

    %% ----------------------------
    %% Exception Handling
    %% ----------------------------
    DLQ->>OPS: Exception Trigger
    OPS->>SQL: Persist Exception Details
    OPS-->>ERP: Notify AP / Project Controls
    OPS-->>ERP: Hold Invoice from Posting

    %% ----------------------------
    %% Reporting
    %% ----------------------------
    SQL-->>PBI: Procurement + Exception Dataset
    PBI-->>PBI: Job Cost / Spend / Exception Dashboards
```

---

## 🔍 What This Diagram Clearly Shows

| Area      | What Happens                                |
| --------- | ------------------------------------------- |
| PO        | Created in ERP → persisted in Azure SQL     |
| Receipt   | Partial or full receipts tracked separately |
| Invoice   | Supplier submits via EDI/PDF                |
| Match     | Azure Function performs 3-way comparison    |
| Success   | Auto-approval → AP ready                    |
| Failure   | Exception raised → DLQ + workflow           |
| Reporting | Power BI reads single source of truth       |

---

## 🧠 Why This Model Works in Real Finance Systems

### ✔ Financial Control

* Enforces **SOX-friendly segregation**
* No invoice posts without matching PO + receipt

### ✔ Construction Reality

* Supports **partial receipts**
* Handles **split invoices**
* Tracks **freight / tax / price variance**

### ✔ Azure Reliability

* Service Bus guarantees delivery
* DLQ ensures nothing is lost
* Functions scale automatically

### ✔ Reporting Accuracy

* Job cost is not inflated prematurely
* Commitments vs actuals are always clear

---

## 🔐 Exception Scenarios Handled

| Scenario                        | Outcome         |
| ------------------------------- | --------------- |
| Invoice > PO amount             | Exception       |
| Invoice received before receipt | Pending         |
| Partial receipt mismatch        | Exception       |
| Unit price variance             | Tolerance check |
| Duplicate invoice               | Blocked         |
| Invalid cost code               | Exception       |

---

## 📊 Power BI Dashboards Enabled

* Open PO Commitments
* Actual vs Committed Cost
* Invoice Aging
* Exception Backlog
* Vendor Accuracy Rate
* Project Cost Overruns

---

## 💡 How This Aligns With Your Architecture Style

This diagram directly aligns with how you typically design:

* **Event-driven ingestion**
* **Service Bus fan-out**
* **Azure SQL as financial system of record**
* **Power BI operational + exec dashboards**
* **Clear control boundaries (ERP vs Azure vs Supplier)**

This is exactly how you’d explain it to:

* Finance
* Construction Ops
* Auditors
* Azure review boards

---

If you want next, I can:

✅ Add **tolerance logic table** (price %, quantity %, freight)
✅ Build **SQL stored procedure for 3-way matching**
✅ Create **Power BI semantic model (facts + dimensions)**
✅ Convert this into a **slide-ready architecture page**
✅ Add **Durable Function version** (wait-for-receipt orchestration)

Just tell me which one you want next 👌
