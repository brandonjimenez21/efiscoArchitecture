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
    %% Definición de Estilos Minimalistas (Fondo Transparente)
    classDef user stroke:#90a4ae,stroke-width:2px,stroke-dasharray: 5 5,fill:none;
    classDef client stroke:#0288d1,stroke-width:2px,fill:none;
    classDef logic stroke:#fbc02d,stroke-width:2px,fill:none;
    classDef storage stroke:#2e7d32,stroke-width:2px,fill:none;
    classDef external stroke:#7b1fa2,stroke-width:2px,fill:none;

    User((Usuario Taller)) --> Capa1

    subgraph Capa1 ["1. Capa de Cliente & Acceso"]
        Frontend[Frontend: React 19 + Zustand] --> Auth[JWT Auth + Workshop Isolation]
    end
    class Frontend,Auth client;

    subgraph Capa2 ["2. Núcleo de Aplicación (Express 5)"]
        BackendRouter{API Gateway Router}
        
        subgraph Modulos ["Módulos de Negocio"]
            Ops[Operaciones: Recepción/Bahía]
            Inv[Logística: Inventario/Kardex]
            Fin[Contabilidad: Liquidación/PUC]
        end
        
        subgraph Motores ["Motores de Inteligencia & Reglas"]
            VClass[VehicleClassifier: Tiering]
            OCR[OCR IA: AWS Textract]
            FE[FinancialEngine: DIAN 2026]
        end

        BackendRouter --> Ops & Inv & Fin
        Ops --> VClass
        Inv --> OCR
        Fin --> FE
    end
    class BackendRouter,Ops,Inv,Fin,VClass,OCR,FE logic;

    subgraph Capa3 ["3. Capa de Datos (Supabase/PostgreSQL)"]
        DB[(Database: Multi-tenant Schema)] --> Trig[Atomic Triggers: Stock/Audit]
        DB --> Ledger[Cash Flow Ledger: Inmutable]
    end
    class DB,Trig,Ledger storage;

    subgraph Capa4 ["4. Ecosistema Externo e Integraciones"]
        Pay[Pasarelas: Bold/Addi]
        DIAN[Facturación: Dataico]
        WA[Notificaciones: Meta API]
    end
    class Pay,DIAN,WA external;

    %% --- FLUJO ESTRUCTURADO ENTRE CAPAS ---
    Capa1 --> Capa2
    Capa2 --> Capa3
    Capa2 --> Capa4

    %% Aplicación de Estilo Especial a Usuario
    class User user;
```

---

## 🔄 Ciclo de Vida Operativo (End-to-End)

Desde la recepción del vehículo hasta el cierre contable, cada paso está sincronizado con el motor financiero.

```mermaid
sequenceDiagram
    autonumber
    participant C as Cliente/Vehículo
    participant R as Recepción (QuickIntake)
    participant B as Bahía (WorkOrder)
    participant I as Inventario (Kardex)
    participant F as Finanzas (Settlement)

    C->>R: Ingreso (Reporte de Fallo)
    R->>WA: Notificar al Cliente (Meta API)
    R->>B: Convertir a Orden de Trabajo
    Note over B: El VehicleClassifier asigna Tier (Básico/Premium)
    
    loop Durante el Servicio
        B->>I: Solicitar Repuesto
        I->>I: Validar Stock & Categoría
        I-->>B: Descontar Stock (Atomic Sync)
        B->>B: Acumular Costo de Mano de Obra e Inventario
    end
    
    B->>F: Finalizar y Liquidar Orden
    F->>FE: Procesar FinancialEngine (Impuestos/Comisiones)
    F->>Ledger: Insertar Movimientos PUC (INC_GROSS, TAX_IVA)
    F->>Dataico: Generar Factura Electrónica
    F->>C: Vehículo Entregado + Factura WhatsApp
```

---

## 📦 Lógica de Inventario y Kardex Inmutable

Trazabilidad total: cada tornillo está vinculado a un proveedor y a una orden de trabajo.

```mermaid
graph LR
    subgraph "Entrada (Abastecimiento)"
        Purchase[Compra a Proveedor] --> OCR_P[OCR: Extraer Factura]
        OCR_P --> Inv_Up[Update: current_stock]
        Inv_Up --> Tx_In[Transaction: type=invoice]
    end

    subgraph "Estado del Almacén"
        Inv_Up --> Master[(Inventario Maestro)]
        Master --> Alerts{Stock < Min?}
        Alerts --> Dashboard[Notificación Low Stock]
    end

    subgraph "Salida (Operación)"
        WO[Work Order] --> Add_Item[Añadir Repuesto]
        Add_Item --> Pricing[PricingEngine: IA Margin]
        Pricing --> Inv_Down[Update: current_stock]
        Inv_Down --> Tx_Out[Transaction: type=service_out]
    end

    Tx_In & Tx_Out --> Kardex[[Historial Kardex por Ítem]]
```

---

## 📊 Motor Financiero (FinancialEngine.js)

El cerebro de Efisco implementa la lógica fiscal colombiana 2026.

### 1. Matriz de Decisión de Liquidación

```mermaid
flowchart TD
    Start([Inicio Liquidación]) --> Data[Cargar: Labor + Parts + WorkshopConfig]
    Data --> Tier{Tier del Servicio?}
    
    Tier -- Premium --> PM[Aplicar Margen Premium: ~10%]
    Tier -- Básico --> BM[Aplicar Margen Básico: ~5%]
    
    PM & BM --> Base[Base Impositiva]
    Base --> IVA[Cálculo IVA: 19% si aplica]
    IVA --> Total[Total Factura]
    
    Total --> Gateway{Usa Pasarela?}
    Gateway -- Bold/Addi --> Comm[Calcular Comisión + IVA Plataforma]
    Gateway -- Efectivo --> NoComm[Cero Comisión]
    
    Comm & NoComm --> Rets{Agente Retenedor?}
    Rets -- Sí --> CalcRets[ReteIVA 15% / ReteFuente / ReteICA]
    Rets -- No --> ZeroRets[Sin Retenciones]
    
    CalcRets & ZeroRets --> Ledger[(Registro Ledger Atómico)]
    Ledger --> Result[Net Cash Inflow + Real Bank Balance]
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
