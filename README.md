# Cloud Provider Analytics

Proyecto Integrador de la materia **Minería de Datos II** de ISTEA – Segundo Cuatrimestre 2026.

## Objetivo

Diseñar e implementar progresivamente una solución de datos para el caso Cloud Provider Analytics, que permita integrar y analizar información de clientes, usuarios, recursos cloud, soporte, facturación, marketing, NPS y eventos de uso.

El proyecto contempla tanto procesamiento batch como streaming, utilizando una arquitectura de Data Lake.

## Tecnologías principales

- Python y Pandas para exploración de datos.
- PySpark para procesamiento batch.
- Spark Structured Streaming para procesamiento de eventos.
- Parquet para almacenamiento de datos procesados.
- Cassandra / AstraDB para consultas sobre datos preparados.

## Arquitectura propuesta

Se definió una arquitectura híbrida que combina procesamiento batch para las fuentes CSV y streaming para los eventos de uso en formato JSONL.

El Data Lake se organiza en cuatro capas:

- **Landing:** conserva los archivos originales sin modificaciones.
- **Bronze:** incorpora los datos con tipos definidos y metadatos de trazabilidad.
- **Silver:** realiza validaciones, limpieza e integración de datos.
- **Gold:** prepara información para análisis de costos, soporte y uso de servicios.

Los datos procesados en Gold podrán utilizarse para alimentar Cassandra/AstraDB y herramientas de análisis.

La arquitectura fue diseñada durante el primer parcial. Su implementación y validación se desarrollarán progresivamente en las siguientes etapas.

## Fuentes de datos

El proyecto trabaja con ocho fuentes de información:

- `customers_orgs.csv`
- `users.csv`
- `resources.csv`
- `support_tickets.csv`
- `marketing_touches.csv`
- `nps_surveys.csv`
- `billing_monthly.csv`
- `usage_events_stream` (JSONL)

Durante la exploración se analizaron estructuras, tipos de datos, valores nulos, duplicados y posibles problemas de calidad.

Se identificaron 43.200 eventos de uso distribuidos en 120 archivos JSONL, con dos versiones de esquema.

También se corrigió la interpretación de `customers_orgs.nps_score`: este campo representa un NPS agregado cuyo rango válido es de -100 a 100. Se detectó un único valor fuera de rango (101).

## Estructura del repositorio

- `notebooks/`: notebooks de exploración y análisis.
- `docs/`: documentación y diagramas del proyecto.
- `DECISIONS.md`: registro de decisiones técnicas.
- `README.md`: descripción general y estado del proyecto.

A medida que avance la implementación se incorporarán las carpetas necesarias para código, configuraciones, pruebas y evidencias.

## Estado del proyecto

**Primer parcial:** diseño inicial y exploración de fuentes completados.

**Correcciones posteriores:** actualización de la validación NPS y revisión de documentación.

**Próxima etapa:** implementación de procesos batch y streaming, controles de calidad y preparación de datos para análisis.
