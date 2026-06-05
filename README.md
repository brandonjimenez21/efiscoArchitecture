# Efisco ERP 🔧 - Automotive Management & Automation SaaS

> **Ingeniería de software de alta precisión aplicada a la rentabilidad y automatización del sector automotriz.**

Efisco es una plataforma SaaS diseñada para transformar talleres mecánicos en centros operativos inteligentes. A diferencia de un ERP genérico, Efisco integra un **Motor Financiero (Fiscal/Contable)** adaptado a la normativa colombiana de 2026, un **Clasificador de Vehículos** para tarificación dinámica y **OCR con IA** para el control de egresos.

![React 19](https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react)
![Express 5](https://img.shields.io/badge/Express-5-000000?style=for-the-badge&logo=express)
![Supabase](https://img.shields.io/badge/Supabase-Database-3ECF8E?style=for-the-badge&logo=supabase)
![Tailwind v4](https://img.shields.io/badge/Tailwind-v4-06B6D4?style=for-the-badge&logo=tailwind-css)
![Tesseract.js](https://img.shields.io/badge/OCR-Tesseract-blue?style=for-the-badge)
![Meta API](https://img.shields.io/badge/WhatsApp-Meta_API-25D366?style=for-the-badge&logo=whatsapp)

---

## 🏗️ Arquitectura y Blindaje Técnico

El sistema ha evolucionado hacia una arquitectura resiliente y multi-tenant de datos aislados.

### 1. Stack Tecnológico de Vanguardia
*   **Frontend (React 19 + Zustand):** Uso de Hooks concurrentes y gestión de estado ligero. Interfaz construida con **Tailwind v4** y componentes de **Shadcn/UI**.
*   **Backend (Node.js + Express 5):** API robusta con manejo nativo de errores y validaciones defensivas.
*   **Persistencia (Supabase/PostgreSQL):** Esquema físico optimizado con **CHECK CONSTRAINTS** para integridad de datos y disparadores atómicos para stock.

### 2. Resoluciones Críticas de Sincronización
*   **Mirror Mapping (Paridad de Esquema):** Implementación de una capa de compatibilidad en el controlador de taller para sincronizar automáticamente columnas duplicadas (`is_responsible_vat` ↔ `is_responsable_iva`, `iva_percentage` ↔ `vat_percentage`).
*   **Normalización Estricta de Tiers:** Clasificación forzada de servicios a **"Premium"** o **"Básico"**, garantizando compatibilidad absoluta con las restricciones de la base de datos.
*   **Blindaje WhatsApp (Meta Cloud API):** Sanitización extrema de teléfonos y manejo elegante de la ventana de 24h de Meta, evitando errores 500 y mejorando la UX del administrador.

---

## 📈 Inteligencia de Negocio: El "Financial Engine"

El núcleo de Efisco es su motor de cálculos puras (`financialEngine.js`):

### 1. Liquidación Fiscal (Normativa Colombia 2026)
*   **Régimen Ordinario vs Simple:** Desglose automático de IVA (19%), ReteIVA (15%), ReteFuente (basado en topes de 4 y 27 UVT) y ReteICA municipal.
*   **Saldo Real Bancario:** Cálculo de liquidez neta restando el efectivo en cuenta de la **Responsabilidad Fiscal (IVA Final)**.

### 2. Integración de Pasarelas de Pago
*   **Bold:** Soporte para tasas físicas (2.99% + 300) y online (3.49% + 900) con desglose de IVA plataforma.
*   **Addi:** Aplicación de comisiones en el rango 9-12%.

### 3. Flujo de Egresos y OCR
*   **Procesamiento OCR:** Integración de **Tesseract.js** para escanear facturas de proveedores, extrayendo NIT y valores totales automáticamente.
*   **Compras Automatizadas:** Registro de egresos con impacto inmediato en el **Libro Auxiliar (PUC)** y actualización de stock.

---

## 🛠️ Módulos Operativos Implementados

| Módulo | Estado | Funcionalidad Clave |
| :--- | :--- | :--- |
| **Recepción Express** | ✅ 100% | Registro rápido, envío de plantillas WhatsApp (Meta) y cola de espera. |
| **Bahía de Servicios** | ✅ 100% | Control de tiempos, asignación de mecánicos y selección de piezas de inventario. |
| **Clasificador IA** | ✅ 100% | Asignación automática de Tiers (Premium/Básico) según marca y antigüedad. |
| **Facturación (Settle)** | ✅ 100% | Cierre contable atómico, registro en Ledger y descuento de stock. |
| **Inventario Pro** | ✅ 90% | Stock en tiempo real, alertas de stock mínimo y trazabilidad de piezas. |
| **Dashboard Financiero** | ✅ 100% | Salud económica, margen operativo bruto y pasivos fiscales consolidados. |
| **OCR Proveedores** | ✅ 100% | Escaneo de facturas y automatización de cuentas por pagar. |

---

## 🚀 Lo que falta (Roadmap)

### Backend (Próximas Fases)
*   **Integración Dataico:** Envío de XML para Facturación Electrónica DIAN definitiva.
*   **Generación de PDFs:** Creación dinámica de Comprobantes de Ingreso y Ordenes de Trabajo impresas.
*   **Módulo de Nómina:** Cálculo automático de comisiones por mecánico basado en servicios finalizados.

### Frontend (Mejoras de UX)
*   **Visualización de Gráficas:** Implementación de gráficos de barras/torta en el Dashboard Financiero (Recharts).
*   **Historial Detallado:** Vista expandida de órdenes pasadas con filtros por fecha y placa.
*   **Notificaciones Push:** Alertas en tiempo real para mecánicos cuando se les asigna una nueva orden.

---

## 🛠️ Ejecución y Pruebas

```bash
# Servidor Backend (0.0.0.0:3000)
cd backend && pnpm dev

# Cliente Frontend (Vite)
cd frontend && pnpm dev

# Suite de Validación Financiera
cd backend && pnpm test tests/billing.test.js
cd backend && pnpm test tests/vehicleClassifier.test.js
cd backend && pnpm test tests/providerPurchase.test.js
```

---

**Efisco ERP** — *Impulsando la ingeniería automotriz a través del software.*
