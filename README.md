# Efisco ERP 🔧 - Automotive Management & Automation SaaS

> **High-precision software engineering applied to profitability and automation in the automotive sector.**

Efisco is a SaaS platform designed to transform mechanical workshops into intelligent operational centers. Unlike a generic ERP, Efisco integrates a **Financial Engine (Fiscal/Accounting)** adapted to Colombian regulations (2026), a **Vehicle Classifier** for dynamic pricing, and **AI-powered OCR** for expense control.

![React 19](https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react)
![Express 5](https://img.shields.io/badge/Express-5-000000?style=for-the-badge&logo=express)
![Supabase](https://img.shields.io/badge/Supabase-Database-3ECF8E?style=for-the-badge&logo=supabase)
![Tailwind v4](https://img.shields.io/badge/Tailwind-v4-06B6D4?style=for-the-badge&logo=tailwind-css)
![AWS Textract](https://img.shields.io/badge/OCR-AWS_Textract-orange?style=for-the-badge)
![Meta API](https://img.shields.io/badge/WhatsApp-Meta_API-25D366?style=for-the-badge&logo=whatsapp)

> 🌐 [Leer en Español](./README.es.md)

---

## 🏗️ System Architecture & Technical Design

The system uses a **multi-tenant architecture** with row-level data isolation (RLS) and an immutable financial calculation core.

### 1. Component & Layer Map

```mermaid
graph LR
    classDef default stroke:#455a64,stroke-width:1px,fill:none;
    classDef highlight stroke:#0052cc,stroke-width:2px,fill:none;
    classDef engine stroke:#d9480f,stroke-width:1.5px,stroke-dasharray: 3 3,fill:none;
    classDef database stroke:#212529,stroke-width:2px,fill:none;
    classDef external stroke:#5f3dc4,stroke-width:1.5px,fill:none;
    classDef user stroke:#90a4ae,stroke-width:2px,stroke-dasharray: 5 5,fill:none;

    User((User)) --> FE[Frontend React 19]
    FE --> Auth[Middleware: JWT/RLS]

    subgraph Core [Backend Core - Express 5]
        Auth --> Ops[Operations Module]
        Auth --> Inv[Logistics Module]
        Auth --> Fin[Finance Module]

        Ops --- VClass[Vehicle Classifier]
        Inv --- OCR[AWS Textract]
        Fin --- FEng[Financial Engine]
    end

    subgraph Data [Persistence - Supabase]
        DB[(PostgreSQL Master)] --> Kardex[[Immutable Ledger]]
    end

    subgraph Externals [External Services]
        WA[WhatsApp API]
        Pay[Bold/Addi Gateways]
        DIAN[Dataico DIAN]
    end

    Ops & Inv & Fin ----> DB
    Ops --> WA
    Fin --> Pay
    Fin --> DIAN

    class User user;
    class FE,Auth,Ops,Inv,Fin highlight;
    class VClass,OCR,FEng engine;
    class DB,Kardex database;
    class WA,Pay,DIAN external;
```

---

## 🔄 Operational Lifecycle (End-to-End)

Full execution flow with state management and synchronous activations.

```mermaid
sequenceDiagram
    autonumber
    participant C as Client
    participant R as Reception
    participant WA as WhatsApp (Meta)
    participant B as Bay (Ops)
    participant I as Inventory
    participant F as Finance
    participant EXT as External (DIAN/Pay)
    participant DB as Persistence (DB)

    C->>R: Arrival (Fault Report)
    activate R
    R->>WA: Notify Client
    R->>B: Convert to Work Order
    activate B
    Note over B: Classifier assigns Tier (Basic/Premium)
    deactivate R

    loop During Service
        B->>I: Request Part
        activate I
        I->>DB: Validate Stock & TX
        DB-->>I: Confirmation
        I-->>B: Item Delivered
        deactivate I
        B->>B: Accumulate Costs
    end

    B->>F: Finalize & Settle
    activate F
    deactivate B

    rect rgb(0, 0, 0)
        Note over F, DB: Accounting Close Process (Fiscal 2026)
        F->>F: Run FinancialEngine
        F->>DB: Insert Ledger Entries
        F->>EXT: Issue Invoice & Process Payment
    end

    F->>C: Vehicle Ready + Digital Invoice
    deactivate F
```

---

## 📦 Inventory Logic & Immutable Kardex

Full traceability: every physical movement generates a mandatory accounting entry in the database.

```mermaid
graph LR
    subgraph "Input (Supply)"
        Purchase[Supplier Purchase] --> OCR_P[OCR: Extract Invoice]
        OCR_P --> Inv_Up[Update: current_stock]
    end

    subgraph "Persistence (Database)"
        Inv_Up --> Master[(Master Inventory)]
        Inv_Up --> Kardex[[Immutable Kardex History]]
        Master --> Alerts{Stock < Min?}
    end

    subgraph "Output (Operations)"
        WO[Work Order] --> Add_Item[Add Part]
        Add_Item --> Pricing[PricingEngine: AI Margin]
        Pricing --> Inv_Down[Update: current_stock]
        Inv_Down --> Kardex
    end

    Alerts --> Dashboard[Low Stock Notification]
```

---

## 📊 Financial Engine (FinancialEngine.js)

### Settlement Decision Matrix

```mermaid
flowchart TD
    Start([Start Settlement]) --> Data[Load: Labor + Parts + Config]
    Data --> Tier{Service Tier?}

    Tier -- Premium --> PM[Apply Premium Margin: ~10%]
    Tier -- Basic --> BM[Apply Basic Margin: ~5%]

    PM & BM --> Base[Tax Base]
    Base --> IVA[VAT Calculation: 19% if applicable]
    IVA --> Total[Invoice Total]

    Total --> Gateway{Payment Gateway?}
    Gateway -- Bold/Addi --> Comm[Calculate Commission + VAT]
    Gateway -- Cash --> NoComm[Zero Commission]

    Comm & NoComm --> Rets{Withholding Agent?}
    Rets -- Yes --> CalcRets[ReteIVA 15% / ReteFuente / ReteICA]
    Rets -- No --> ZeroRets[No Withholdings]

    CalcRets & ZeroRets --> DB[(Atomic Ledger Persistence)]
    DB --> Result[Net Cash Inflow + Real Bank Balance]
```

---

## 🛠️ High-Performance Tech Stack

| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **UI Framework** | React 19 (Beta) | Ultra-fast reactivity and concurrency |
| **Styles** | Tailwind CSS v4 | Atomic design and bundle optimization |
| **State** | Zustand | Lightweight and scalable state management |
| **Backend** | Express 5 + Node.js | Robust API with native promise support |
| **Database** | Supabase (PostgreSQL) | Persistence, RLS and real-time Webhooks |
| **AI/OCR** | AWS Textract | Data extraction from supplier invoices |
| **Comms** | Meta WhatsApp Cloud API | Automated client communication |

---

## 🔑 Key Technical Decisions

- **Multi-tenant with RLS** — Row-level security in PostgreSQL ensures complete data isolation between workshops without separate databases.
- **Immutable Ledger** — Every financial movement is append-only. No record is ever updated or deleted, guaranteeing full accounting auditability.
- **Async OCR pipeline** — Supplier invoice processing runs in the background via AWS Textract, keeping the UI responsive.
- **Dynamic tier pricing** — The Vehicle Classifier automatically assigns service tiers, enabling margin control without manual configuration per service.

---

## 🚀 Getting Started

```bash
# Backend server (0.0.0.0:3000)
cd backend && pnpm dev

# Frontend client (Vite)
cd frontend && pnpm dev

# Unit test suite (Financial logic & Classification)
cd backend && pnpm test
```

---

## 📬 Contact
efiscosas@gmail.com

Built and maintained by a single developer. Open to feedback, contributions and collaboration.
---

**Efisco ERP** — *Driving automotive engineering through high-performance software.*
