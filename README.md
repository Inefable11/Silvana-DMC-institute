# TP Integrador — Azure Data Engineering (Edición 3)

**Alumna:** Silvana Huayta Gómez
**Fecha de entrega:** 2026-06-22
**Instructor:** Miguel Balcazar
**Diploma:** Advanced Data Engineer — DMC

## Resumen del proyecto

Este TP implementa una arquitectura Medallion (Bronze → Silver → Gold) para
"TransAndino Express", una empresa de logística ficticia, siguiendo el
enunciado del módulo de Azure para Ingeniería de Datos.

A diferencia del entorno provisto por el curso, este trabajo se resolvió
utilizando una **suscripción propia de Microsoft Azure**, lo que permitió
construir la arquitectura completa con servicios reales (no limitado a
Databricks Community Edition):

- **Azure Data Factory** — orquestación de la ingesta raw → landing
- **Azure Data Lake Storage Gen2** — 5 containers (raw, landing, bronze, silver, gold) con permisos RBAC
- **Azure Databricks Premium + Unity Catalog** — transformación Bronze → Silver → Gold
- **Azure Key Vault** — gestión segura de secretos

## Alcance completado

| Parte | Descripción | Estado |
|---|---|---|
| 1 | Arquitectura y seguridad (diagrama + justificación teórica) | ✅ Completo |
| 2 | Ingesta con Azure Data Factory | ✅ Completo |
| 3 | ADLS Gen2 y Medallion Architecture | ✅ Completo |
| 4 | Procesamiento con Azure Databricks | ✅ Completo |
| 5 | Microsoft Fabric, OneLake y Power BI | ❌ No resuelto |

**Nota sobre la Parte 5:** Microsoft Fabric requiere una cuenta profesional o
educativa asociada a un tenant de Microsoft 365. La cuenta personal de Azure
utilizada para el resto del TP no es compatible con el inicio de sesión en
Fabric, por lo que esta parte no pudo completarse en el tiempo disponible.

## Estructura del repositorio

```
├── README.md
├── tp_notebook.ipynb          # Notebook completo Bronze → Silver → Gold
├── informe_final_tp.docx      # Informe con evidencias y respuestas teóricas
├── arquitectura/
│   └── diagrama_arquitectura.png
└── evidencias/
    ├── parte2_adf/            # Linked Services, pipeline, ejecución exitosa
    ├── parte3_adls/           # Containers, permisos RBAC
    └── parte4_databricks/     # Bronze, Silver, MERGE INTO, Gold, verificaciones
```


**Flujo de datos:**
`raw/` (6 CSV fuente) → **ADF** copia a `landing/` (parametrizado por fecha) →
**Databricks** ingesta a `bronze/` (schema explícito, todo String) →
limpieza y transformación a `silver/` (parseo de fechas, deduplicación,
carga incremental con MERGE INTO para incidencias) → agregación a `gold/`
(2 tablas de KPIs: envíos por zona y por tipo de ruta).

## Resultados del pipeline

- **Bronze:** 6 tablas ingestadas (envios: 600, transportistas: 20, rutas: 30,
  clientes_logistica: 60, incidencias 2023: 120, incidencias 2024: 80 filas)
- **Silver:** datos limpios y tipados; tabla `incidencias` con carga
  incremental vía MERGE INTO (170 filas finales: 120 base + 50 nuevas,
  30 actualizadas)
- **Gold:** `envios_zona` (151 filas), `envios_tipo_ruta` (93 filas)
- **Verificación de consistencia:** suma de envíos en Gold coincide
  exactamente con el total de Silver (diferencia = 0)

## Cómo ejecutar

1. El notebook `tp_notebook.ipynb` está parametrizado con un widget
   `fecha_proceso` que simula el parámetro que en el entorno del curso
   inyectaría Azure Data Factory.
2. Requiere acceso a un Storage Account ADLS Gen2 con los containers
   `raw`, `landing`, `bronze`, `silver`, `gold`, y un catálogo de Unity
   Catalog con External Locations configurados sobre cada container.
