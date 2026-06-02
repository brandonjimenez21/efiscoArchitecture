# Efisco ERP 🔧 - Automotive Management & Automation SaaS

> **Ingeniería de software aplicada a la optimización operativa del sector automotriz.**

Efisco es una plataforma SaaS de alto rendimiento diseñada para transformar talleres mecánicos en centros operativos eficientes y automatizados. No es solo un gestor de órdenes; es un motor de inteligencia de negocio que automatiza la clasificación de servicios, optimiza márgenes de ganancia y asegura la trazabilidad total de la operación.

![React 19](https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react)
![Express 5](https://img.shields.io/badge/Express-5-000000?style=for-the-badge&logo=express)
![Supabase](https://img.shields.io/badge/Supabase-Database-3ECF8E?style=for-the-badge&logo=supabase)
![Tailwind v4](https://img.shields.io/badge/Tailwind-v4-06B6D4?style=for-the-badge&logo=tailwind-css)
![Jest](https://img.shields.io/badge/Jest-Testing-C21325?style=for-the-badge&logo=jest)

---

## 🏗️ Arquitectura y Decisiones Técnicas

El diseño sistémico de Efisco se basa en la **separación de preocupaciones (SoC)** y la **escalabilidad horizontal**.

### 1. Stack Moderno y Eficiente
*   **Frontend (React 19 + Vite):** Elegimos React 19 para aprovechar las últimas mejoras en concurrencia y gestión de estados. Vite actúa como el orquestador de compilación para garantizar tiempos de desarrollo mínimos y un bundle de producción optimizado.
*   **Backend (Node.js + Express 5):** Implementamos Express 5 por su manejo robusto de promesas y middleware, permitiendo una API RESTful limpia y altamente predecible.
*   **Persistencia (Supabase/PostgreSQL):** La decisión de usar Supabase responde a la necesidad de una infraestructura distribuida con capacidades de tiempo real nativas y un sistema de autenticación integrado que escala sin fricciones.

### 2. Diseño Multi-Tenant
La arquitectura es inherentemente multi-usuario. Cada petición es interceptada por un middleware de seguridad que inyecta un `workshop_id` basado en el contexto del usuario autenticado, garantizando el aislamiento total de datos entre diferentes talleres a nivel de aplicación.

---

## 📈 Ingeniería de Negocio: El "Pricing Engine"

A diferencia de los ERPs tradicionales, Efisco incluye un motor de lógica algorítmica para la toma de decisiones:

*   **Clasificador Dinámico:** El sistema clasifica vehículos (Gama Alta/Normal) y niveles de servicio (Premium/Básico) basándose en la marca, procedencia y antigüedad del vehículo.
*   **Estrategia de Márgenes Inversos:** Implementamos una lógica de márgenes dinámicos en inventario:
    - Costos bajos (500-1000): **125% de margen**.
    - Costos críticos (500k-1M): **10% de margen**.
    Esto asegura competitividad en el mercado mientras maximiza la utilidad en consumibles de alta rotación.

---

## 🛡️ Calidad de Código y Seguridad

La integridad operativa es nuestra prioridad número uno.

### 1. Validación Estricta de Datos
Implementamos una capa de validación en `backend/utils/validators.js` que utiliza expresiones regulares avanzadas para asegurar que los datos técnicos (medidas de frenos, lubricantes, sistemas eléctricos) sigan estándares industriales antes de tocar la base de datos.

### 2. Seguridad de Nivel Industrial
*   **Protección de Endpoints:** Middleware `requireAuth` que valida tokens JWT contra el servidor de Supabase en cada request.
*   **Aislamiento Operativo:** No se permite la manipulación de IDs de taller desde el cliente; el servidor determina la propiedad de los datos mediante el token de sesión.

### 3. Pruebas Automatizadas (Unit Testing)
Mantenemos una suite de pruebas con **Jest** y **Supertest** enfocada en los núcleos críticos del sistema:
- ✅ Cálculo exacto de utilidades y márgenes.
- ✅ Clasificación correcta de tiers de servicio.
- ✅ Integridad de las rutas protegidas.

```bash
# Ejecutar suite de pruebas de ingeniería
cd backend && pnpm test
```

---

## 🚀 Despliegue y Ejecución

Efisco está diseñado para ser "cloud-native".

### Requisitos
- Node.js v18 o superior.
- Una instancia de Supabase configurada con el esquema relacional de Efisco.

### Setup Rápido
1.  **Backend:** Instala dependencias y configura las variables de entorno (`SUPABASE_URL`, `SUPABASE_KEY`).
2.  **Frontend:** Inicia el servidor de desarrollo con `pnpm dev`.

---

**Efisco ERP** — *Impulsando la ingeniería automotriz a través del software.*

---

## 👥 Autoría y Desarrollo

Este ecosistema de software es un producto comercial cerrado, diseñado y desarrollado por:

*   **Brandon Jimenez** - *Co-Founder & Lead Full-Stack Developer* — [GitHub](https://github.com/brandonjimenez21) | [LinkedIn](https://www.linkedin.com/in/brandon-jimenez-1a124b215/)

> ⚠️ **Nota de Propiedad Intelectual:** El código fuente principal, los modelos de datos avanzados y los servicios de automatización de producción se mantienen en repositorios privados bajo la infraestructura de nuestra organización para proteger los activos comerciales de la empresa. Este repositorio funciona exclusivamente como una vitrina arquitectónica (*showcase*) para la demostración de competencias de ingeniería y diseño de sistemas.
