# Efisco ERP 🔧 - Automotive Management & Automation SaaS

> **Ingeniería de software de alta precisión aplicada a la rentabilidad y automatización del sector automotriz.**

Efisco es una plataforma SaaS diseñada para transformar talleres mecánicos en centros operativos inteligentes. A diferencia de un ERP genérico, Efisco integra un **Motor Financiero (Fiscal/Contable)** adaptado a la normativa colombiana de 2026, un **Clasificador de Vehículos** para tarificación dinámica y **OCR con IA** para el control de egresos.

![React 19](https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react)
![Express 5](https://img.shields.io/badge/Express-5-000000?style=for-the-badge&logo=express)
![Supabase](https://img.shields.io/badge/Supabase-Database-3ECF8E?style=for-the-badge&logo=supabase)
![Tailwind v4](https://img.shields.io/badge/Tailwind-v4-06B6D4?style=for-the-badge&logo=tailwind-css)
![AWS Textract](https://img.shields.io/badge/OCR-AWS_Textract-orange?style=for-the-badge)
![Meta API](https://img.shields.io/badge/WhatsApp-Meta_API-25D366?style=for-the-badge&logo=whatsapp)

---

## 🏗️ Arquitectura de Sistema y Blindaje Técnico

El sistema utiliza una arquitectura multi-tenant con aislamiento de datos a nivel de fila (RLS) y un núcleo de cálculo financiero inmutable.

### 1. Mapa de Componentes y Capas

```mermaid
graph LR
    %% Estilos Globales Minimalistas (Fondos Transparentes)
    classDef default stroke:#455a64,stroke-width:1px,fill:none;
    classDef highlight stroke:#0052cc,stroke-width:2px,fill:none;
    classDef engine stroke:#d9480f,stroke-width:1.5px,stroke-dasharray: 3 3,fill:none;
    classDef database stroke:#212529,stroke-width:2px,fill:none;
    classDef external stroke:#5f3dc4,stroke-width:1.5px,fill:none;
    classDef user stroke:#90a4ae,stroke-width:2px,stroke-dasharray: 5 5,fill:none;

    %% Flujo Inicial de Acceso
    User((Usuario)) --> FE[Frontend React 19]
    FE --> Auth[Middleware: JWT/RLS]

    %% Capa Central: Backend y sus Motores Relacionados
    subgraph Core [Backend Core - Express 5]
        Auth --> Ops[Módulo Operativo]
        Auth --> Inv[Módulo Logístico]
        Auth --> Fin[Módulo Financiero]
        
        %% Conexiones verticales directas con sus respectivos motores
        Ops --- VClass[Vehicle Classifier]
        Inv --- OCR[AWS Textract]
        Fin --- FEng[Financial Engine]
    end

    %% Capa de Datos y Persistencia
    subgraph Data [Persistencia - Supabase]
        DB[(PostgreSQL Master)] --> Kardex[[Ledger Inmutable]]
    end

    %% Capa de Servicios Externos
    subgraph Externals [Servicios Externos]
        WA[WhatsApp API]
        Pay[Pasarelas Bold/Addi]
        DIAN[Dataico DIAN]
    end

    %% Enlaces Estructurados entre Capas (Evita cruces caóticos)
    Ops & Inv & Fin ----> DB
    Ops --> WA
    Fin --> Pay
    Fin --> DIAN

    %% Asignación de Estilos
    class User user;
    class FE,Auth,Ops,Inv,Fin highlight;
    class VClass,OCR,FEng engine;
    class DB,Kardex database;
    class WA,Pay,DIAN external;
```

---

## 🔄 Ciclo de Vida Operativo (End-to-End)

Flujo de ejecución con gestión de estados y activaciones síncronas. La proximidad de servicios externos evita cruces visuales.

```mermaid
sequenceDiagram
    autonumber
    participant C as Cliente
    participant R as Recepción
    participant WA as WhatsApp (Meta)
    participant B as Bahía (Ops)
    participant I as Inventario
    participant F as Finanzas
    participant EXT as Externos (DIAN/Pay)
    participant DB as Persistencia (DB)

    C->>R: Ingreso (Reporte de Fallo)
    activate R
    R->>WA: Notificar al Cliente
    R->>B: Convertir a Orden de Trabajo
    activate B
    Note over B: El Classifier asigna Tier (Básico/Premium)
    deactivate R
    
    loop Durante el Servicio
        B->>I: Solicitar Repuesto
        activate I
        I->>DB: Validar Stock & TX
        DB-->>I: Confirmación
        I-->>B: Item Entregado
        deactivate I
        B->>B: Acumular Costos
    end
    
    B->>F: Finalizar y Liquidar
    activate F
    deactivate B
    
    rect rgb(0, 0, 0)
        Note over F, DB: Proceso de Cierre Contable (Fiscal 2026)
        F->>F: Calcular FinancialEngine
        F->>DB: Insertar Movimientos Ledger
        F->>EXT: Emitir Factura & Procesar Pago
    end
    
    F->>C: Vehículo Listo + Factura Digital
    deactivate F
```

---

## 📦 Lógica de Inventario y Kardex Inmutable

Trazabilidad total: cada movimiento físico genera un reflejo contable obligatorio en la base de datos.

```mermaid
graph LR
    subgraph "Entrada (Abastecimiento)"
        Purchase[Compra a Proveedor] --> OCR_P[OCR: Extraer Factura]
        OCR_P --> Inv_Up[Update: current_stock]
    end

    subgraph "Persistencia (Base de Datos)"
        Inv_Up --> Master[(Inventario Maestro)]
        Inv_Up --> Kardex[[Historial Kardex Inmutable]]
        Master --> Alerts{Stock < Min?}
    end

    subgraph "Salida (Operación)"
        WO[Work Order] --> Add_Item[Añadir Repuesto]
        Add_Item --> Pricing[PricingEngine: IA Margin]
        Pricing --> Inv_Down[Update: current_stock]
        Inv_Down --> Kardex
    end

    Alerts --> Dashboard[Notificación Low Stock]
```

---

## 📊 Motor Financiero (FinancialEngine.js)

### 1. Matriz de Decisión de Liquidación

```mermaid
flowchart TD
    Start([Inicio Liquidación]) --> Data[Cargar: Labor + Parts + Config]
    Data --> Tier{Tier del Servicio?}
    
    Tier -- Premium --> PM[Aplicar Margen Premium: ~10%]
    Tier -- Básico --> BM[Aplicar Margen Básico: ~5%]
    
    PM & BM --> Base[Base Impositiva]
    Base --> IVA[Cálculo IVA: 19% si aplica]
    IVA --> Total[Total Factura]
    
    Total --> Gateway{Usa Pasarela?}
    Gateway -- Bold/Addi --> Comm[Calcular Comisión + IVA]
    Gateway -- Efectivo --> NoComm[Cero Comisión]
    
    Comm & NoComm --> Rets{Agente Retenedor?}
    Rets -- Sí --> CalcRets[ReteIVA 15% / ReteFuente / ReteICA]
    Rets -- No --> ZeroRets[Sin Retenciones]
    
    CalcRets & ZeroRets --> DB[(Persistencia Ledger Atómico)]
    DB --> Result[Net Cash Inflow + Real Bank Balance]
```

---

## 🛠️ Stack Tecnológico de Alto Rendimiento

| Capa | Tecnología | Propósito |
| :--- | :--- | :--- |
| **UI Framework** | React 19 (Beta) | Reactividad ultra-rápida y concurrencia. |
| **Styles** | Tailwind CSS v4 | Diseño atómico y optimización de bundle. |
| **State** | Zustand | Gestión de estado ligero y escalable. |
| **Backend** | Express 5 + Node.js | API robusta con soporte nativo para promesas. |
| **Database** | Supabase (PostgreSQL) | Persistencia, RLS y Webhooks en tiempo real. |
| **AI/OCR** | AWS Textract | Extracción de datos de facturas de proveedores. |
| **Comms** | Meta WhatsApp Cloud API | Comunicación automatizada con el cliente. |

---

## 🚀 Ejecución y Pruebas

```bash
# Servidor Backend (0.0.0.0:3000)
cd backend && pnpm dev

# Cliente Frontend (Vite)
cd frontend && pnpm dev

# Suite de Pruebas Unitarias (Lógica Financiera y Clasificación)
cd backend && pnpm test
```

---

**Efisco ERP** — *Impulsando la ingeniería automotriz a través de software de alto rendimiento.*
