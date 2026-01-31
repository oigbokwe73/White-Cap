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
  ERP[Construction ERP\n(Procore/Sage/CMiC/Viewpoint)] -->|PO / Receipt / Vendor Master| API[ERP Export/API/Webhook]
  SUP[Supplier\n(White Cap / Others)] -->|Invoice PDF / EDI 810 / ASN| IN[Inbound Channel]

  %% =========================
  %% Inbound Channels
  %% =========================
  subgraph Inbound["Inbound Channels"]
    API --> AF1[Azure Function: Ingest-ERP]
    IN --> SA[(Blob Storage\nInbound Docs)]
    IN --> SBQ[Service Bus Queue\nsupplier-events]
  end

  %% =========================
  %% Orchestration / Validation
  %% =========================
  subgraph Orchestration["Orchestration + Validation"]
    AF1 --> SBPO[(Service Bus Queue\npo-events)]
    AF2[Azure Function: Ingest-Supplier] --> SBQ
    SBPO --> AF3[Azure Function: Validate+Normalize (PO/Receipt)]
    SBQ --> AF4[Azure Function: Validate+Normalize (Invoice/ASN)]
    AF3 --> SBT[(Service Bus Topic\nprocurement-domain)]
    AF4 --> SBT
  end

  %% =========================
  %% Persistence
  %% =========================
  subgraph Data["Data Layer"]
    SQL[(Azure SQL Database\nProcurement + Job Cost)]
    SA --> AF5[Azure Function: Doc Parser\n(PDF/EDI Parser)]
    AF5 --> SQL
    SBT --> AF6[Azure Function: Upsert Writer]
    AF6 --> SQL
  end

  %% =========================
  %% Exceptions / DLQ
  %% =========================
  subgraph Ops["Ops + Exceptions"]
    SBT -->|validation fail| DLQ[(Service Bus DLQ)]
    DLQ --> AF7[Azure Function: Exception Handler]
    AF7 --> SQLX[(Azure SQL\nExceptions/Audit)]
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

---

If you want, I can generate the next artifacts in the same style you’ve been building lately:

1. **Mermaid sequence diagram** for PO → Receipt → Invoice → 3-way match → Exception
2. **Azure SQL DDL** for the full schema + views + match stored procedure
3. **Function App skeleton** (C# or Python) with Service Bus triggers + SQL upserts
4. **Power BI dataset model** (tables/measures) for job-cost + vendor KPI dashboards

Tell me which one you want first.
