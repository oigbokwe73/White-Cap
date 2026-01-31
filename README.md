# White-Cap


## 🏗️ Why Contractors Use White Cap — Detailed Breakdown

White Cap isn’t just a place to buy materials — contractors use White Cap because it fits **how construction actually operates in the field**. Below is a deeper, contractor-level explanation.

---

### 🔹 1. Job-Site Speed (Time = Money)

Construction runs on tight schedules. When materials don’t arrive, **crews sit idle — and that’s thousands of dollars lost per day**.

White Cap specializes in:

* Same-day and next-day delivery
* Early-morning job-site drop-offs
* Local branches stocking region-specific materials
* Emergency orders when something breaks or runs short

**Why this matters:**
Contractors don’t need to wait on centralized warehouses or 3–5 day shipping windows. Materials show up *when the crew shows up*.

---

### 🔹 2. Built Specifically for Construction (Not Retail)

Unlike big-box stores, White Cap is **contractor-first**, not consumer-first.

They support:

* Bulk quantities (pallets, bundles, truckloads)
* Construction-grade SKUs (not homeowner versions)
* Trade-specific materials (forming, rebar accessories, embeds, anchors)
* Project-based ordering instead of one-off purchases

**Why this matters:**
Contractors avoid mixing retail-grade products into commercial or DOT-regulated jobs.

---

### 🔹 3. Project-Based Purchasing & Cost Control

Most contractors operate with:

* Multiple active jobs
* Separate budgets per project
* Cost codes tied to accounting systems

White Cap supports:

* Job-number-based purchasing
* Project-level invoicing
* Monthly consolidated billing
* Spend tracking by job site

**Why this matters:**
Finance teams can reconcile invoices directly into systems like:

* Procore
* Viewpoint
* Sage 300 / 100
* CMiC

This drastically reduces accounting overhead.

---

### 🔹 4. Field Sales Reps Who Know Construction

White Cap reps are not retail clerks — many come from:

* Concrete crews
* Estimating teams
* Field supervision roles

They help with:

* Product substitutions during shortages
* Value engineering recommendations
* Code-compliant alternatives
* Quantity planning for pours or phases

**Why this matters:**
When materials are unavailable, experienced reps can **solve problems instead of just taking orders**.

---

### 🔹 5. Strong Safety & Compliance Support

Safety compliance is non-negotiable on modern job sites.

White Cap supplies:

* OSHA-compliant PPE
* Job-site safety kits
* Fall protection systems
* Fire protection supplies
* Site signage and barriers

They also help contractors standardize safety SKUs across all projects.

**Why this matters:**
Consistent safety inventory reduces:

* OSHA violations
* Job shutdowns
* Insurance exposure

---

### 🔹 6. Reliable Supply Chain During Shortages

During material shortages (concrete chemicals, anchors, fasteners), contractors rely on:

* Local branch inventory
* Regional redistribution
* Manufacturer relationships

White Cap maintains:

* Long-term supplier contracts
* Regional stocking strategies
* Backup product equivalents

**Why this matters:**
When national suppliers are out of stock, White Cap often still has local inventory.

---

### 🔹 7. Single Vendor for Multiple Trades

Instead of managing 10+ suppliers, contractors use White Cap as a **single-source distributor** for:

* Concrete accessories
* Tools & fasteners
* PPE
* Waterproofing
* Restoration materials

**Why this matters:**
Fewer vendors = fewer invoices, fewer delays, less admin work.

---

### 🔹 8. Strong Support for Large & Small Contractors

White Cap scales well for:

* Small contractors (5–50 employees)
* Mid-size GCs
* National infrastructure firms

They support:

* Credit accounts
* Dedicated account managers
* National pricing agreements
* Local branch flexibility

**Why this matters:**
Contractors can grow without changing suppliers every time they scale.

---

### 🔹 9. Trusted in High-Risk, High-Accountability Projects

White Cap is widely used on:

* DOT infrastructure projects
* Bridges and highways
* Airports
* Industrial plants
* Government-funded builds

**Why this matters:**
These projects demand:

* Certified materials
* Documentation
* Consistent supply
* Zero tolerance for mistakes

White Cap already operates in that environment.

---

### 🔹 10. Reduces Operational Friction

At the end of the day, contractors use White Cap because it reduces friction across:

* Field operations
* Procurement
* Accounting
* Safety
* Scheduling

Instead of chasing materials, teams can focus on **building**.

---

### 📌 Bottom Line

Contractors don’t choose White Cap because it’s cheap —
they choose it because it’s **reliable, predictable, and construction-native**.

> **“White Cap understands how job sites actually work.”**

---

## Azure Procurement Integration Architecture

### What this solves

* Ingest POs / receipts / invoices from ERP + suppliers (like White Cap)
* Normalize + validate (job, cost code, vendor, pricing)
* Persist to **Azure SQL** for finance/job costing
* Stream operational events + exceptions through **Service Bus**
* Drive **Power BI** spend + WIP + vendor performance dashboards

---

Below is a **battle-tested Azure supplier integration architecture** that covers **APIs + EDI + spend analytics**, designed for a contractor or distributor ecosystem (suppliers like White Cap, HD Supply, Grainger, etc.), with **SOX-friendly controls**, strong observability, and clean separation of concerns.

---

## Target Architecture

### What it supports

* **Supplier catalog & pricing** (contract pricing, regional availability)
* **Punchout / procurement** (optional)
* **PO creation + acknowledgement**
* **Ship notices / ASN**
* **Invoices + credits**
* **Payments status**
* **Returns / RMAs**
* **Spend analytics** (project / cost code / supplier / category)

---

## High-level Azure Architecture

```mermaid
flowchart LR
  %% Channels
  subgraph Channels[Consumers]
    A1[ERP / Accounting Sage, Viewpoint, CMiC]
    A2[Procurement Portal Web / Mobile]
    A3[Jobsite Apps Superintendent / Foreman]
  end

  %% Edge/API
  subgraph Edge[API & Security Edge]
    APIM[Azure API Management]
    WAF[App Gateway + WAF]
    IDP[Entra ID / B2C OIDC/OAuth2]
  end

  %% Integration
  subgraph Integration[Integration Layer]
    FN[Azure Functions\nOrchestration]
    LA[Logic Apps\nB2B / EDI workflows]
    SB[Service Bus\nQueues + Topics]
    EH[Event Hubs\nStreaming events]
    KV[Key Vault\nSecrets/Certs]
  end

  %% B2B/EDI
  subgraph B2B[EDI / B2B]
    IA[Integration Account\nSchemas/Maps/Agreements]
    X12[X12 / EDIFACT\n(850/855/856/810)]
    SFTP[SFTP Endpoint Blob/ADLS]
  end

  %% Data
  subgraph Data[Data Platform]
    SQL[Azure SQL Database\nOrders/Invoices Master]
    DL[ADLS Gen2 Raw/Curated]
    CG[Cosmos DB Supplier catalog cache]
    SR[Schema Registry Optional]
  end

  %% Analytics
  subgraph Analytics[Analytics & Reporting]
    DBX[Databricks / Synapse\nTransforms]
    PBI[Power BI\nSpend + Ops KPIs]
    KQL[Log Analytics / Kusto\nOperational analytics]
  end

  %% Observability
  subgraph Obs[Observability]
    AI[Application Insights]
    MON[Azure Monitor Alerts]
    SIEM[Microsoft Sentinel\n(optional)]
  end

  %% Suppliers
  subgraph Suppliers[Suppliers]
    S1[Supplier APIs\nCatalog/Orders/Invoices]
    S2[EDI VAN / AS2\nor Direct EDI]
  end

  %% Flows
  A1 -->|REST| WAF --> APIM
  A2 -->|REST| WAF --> APIM
  A3 -->|REST| WAF --> APIM
  APIM --> IDP
  APIM --> FN

  FN --> SB
  FN --> SQL
  FN --> CG
  FN --> AI

  SB --> LA
  LA --> IA
  IA --> X12
  X12 --> S2
  S2 --> X12 --> IA --> LA --> SB

  FN -->|Supplier REST| S1
  S1 --> FN

  LA --> SFTP
  SFTP --> DL
  SQL --> DBX --> PBI
  DL --> DBX --> PBI
  AI --> MON
  AI --> KQL
  KQL --> SIEM
  KV --> FN
  KV --> LA
```

---

## Core Patterns (so this stays reliable at scale)

### 1) API pattern: “Front Door + Policy + Contract”

**APIM** becomes your “single throat to choke”:

* OAuth2/OIDC enforcement, rate limits, IP allowlists
* Subscription keys per consumer (ERP vs portal vs jobsite)
* Request/response validation (schemas)
* Transform + normalize payloads (lightweight; keep heavy transforms out)

**Best practice:**
Use **canonical internal APIs** (your contract), and map each supplier’s quirks behind the scenes.

---

### 2) EDI pattern: “Events drive B2B, not the other way around”

Treat EDI documents as **messages** with strict lifecycle states:

* Received → Validated → Mapped → Posted → Acknowledged → Reconciled
* Every stage emits events into Service Bus

This makes EDI observable and retryable—like any other integration.

---

### 3) Messaging pattern: “Service Bus for reliability, Event Hubs for telemetry”

* **Service Bus Topics** for business events (PO.Created, Invoice.Received)
* **Service Bus Queues** for work items (per supplier or doc type)
* **Event Hubs** for high-volume “clickstream-like” integration telemetry (optional)

---

## Key Workflows (API + EDI together)

### Workflow A — Create PO (API-first, with EDI fallback)

```mermaid
sequenceDiagram
  autonumber
  participant ERP as ERP/Portal
  participant APIM as APIM
  participant FN as Functions (PO Orchestrator)
  participant SB as Service Bus
  participant SUP as Supplier API
  participant LA as Logic Apps (EDI)
  participant IA as Integration Account

  ERP->>APIM: POST /purchase-orders
  APIM->>FN: Validate + forward
  FN->>SB: Publish PO.Created event

  alt Supplier supports REST orders
    FN->>SUP: Create PO via API
    SUP-->>FN: PO Ack (id/status)
  else Supplier requires EDI
    SB->>LA: Trigger PO.Created
    LA->>IA: Map to X12 850
    IA-->>LA: 850 payload
    LA-->>SB: EDI.Sent(850)
  end

  FN-->>ERP: 202 Accepted + trackingId
```

### Workflow B — Invoice ingest + 3-way match (810 / API invoice)

```mermaid
sequenceDiagram
  autonumber
  participant SUP as Supplier (EDI/API)
  participant LA as Logic Apps
  participant IA as Integration Account
  participant SB as Service Bus
  participant FN as Functions (Invoice Processor)
  participant SQL as Azure SQL (PO/Receipt/Invoice)
  participant PBI as Power BI

  SUP->>LA: EDI 810 (Invoice) or API callback
  LA->>IA: Validate schema + agreement
  IA-->>LA: Normalized invoice JSON
  LA->>SB: Invoice.Received

  SB->>FN: Trigger processing
  FN->>SQL: Persist invoice + line items
  FN->>SQL: 3-way match (PO/Receipt/Invoice)
  FN-->>SB: Invoice.Matched or Invoice.Exception

  SQL-->>PBI: Spend + Exceptions reporting (refresh)
```

---

## Spend Analytics Model (what you should store)

### Canonical “Spend Fact” (curated table)

Minimum fields:

* SupplierId, SupplierName
* ProjectId, CostCode
* POId, InvoiceId, ReceiptId
* Category (tools, safety, concrete, fasteners)
* Amount, Tax, Freight, Discounts
* InvoiceDate, PostingDate
* Region/Branch (if relevant)
* MatchStatus (Matched / Partial / Exception)
* SLA metrics (days to ack, ship, invoice, pay)

**Storage approach**

* System-of-record: **Azure SQL** (for transactional truth + audit)
* Raw + historical + documents: **ADLS Gen2**
* Curated analytics: **Databricks/Synapse → Power BI**

---

## Security + Compliance (SOX-friendly)

### Identity & access

* APIM protected by **Entra ID** (users) and **Managed Identities** (services)
* Supplier integrations use **client certs / mTLS** where possible (stored in Key Vault)
* RBAC + least privilege everywhere

### Data protection

* TLS everywhere; private endpoints for SQL/Storage where possible
* Encryption at rest (default); consider CMK if required
* PII minimization: store only what you need for procurement/accounting

### Auditability

* Every document/message has:

  * correlationId, supplierId, documentType, status, timestamps
* Immutable raw payloads in ADLS “raw” zone (append-only)
* Logs in Log Analytics + long retention as needed

---

## Ops / Observability (you’ll thank yourself later)

**Application Insights**

* Track each integration transaction with a single correlationId
* Custom events: `PO.Created`, `EDI.Parsed`, `Invoice.Matched`, `Retry.Attempted`

**Dashboards worth building**

* Orders: created vs acked vs failed (by supplier)
* EDI: 850/855/856/810 counts + failure reasons
* Finance: invoice exceptions by category/cost code
* Latency: time from PO → ack, ship, invoice, paid

---

## Recommended “MVP → Scale” Build Plan

### MVP (2–4 weeks if you’re moving fast)

* APIM + Functions canonical endpoints: PO, Invoice
* Service Bus topics + dead-letter handling
* Azure SQL master tables + ADLS raw payload storage
* One supplier via API, one supplier via EDI (prove both patterns)
* Power BI initial spend dashboard

### Scale (next 6–12 weeks)

* EDI agreements per supplier + automated onboarding checklist
* Retry policies + idempotency keys everywhere
* Databricks/Synapse curated spend model
* Sentinel (optional) + alert tuning
* Supplier scorecards (OTIF, ack SLA, invoice accuracy)

---

## If you want, I can generate “starter assets” for this

Pick what you want and I’ll drop it in ready-to-run format:

1. **Azure SQL schema** (PO/Invoice/Receipt + SpendFact + Audit tables)
2. **APIM API design** (OpenAPI spec + policies for rate limit + JWT validate)
3. **Function App skeleton** (.NET or Node) with:

   * idempotency
   * Service Bus publish/subscribe
   * correlationId propagation
4. **Logic Apps EDI workflow outline** (850/855/810)
5. **Power BI KPI list + sample dashboard layout**

Tell me your preferred stack for Functions (**.NET** vs **Node**) and whether you want **EDI via Logic Apps** or an **EDI translator service** pattern—and I’ll tailor the templates.


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
