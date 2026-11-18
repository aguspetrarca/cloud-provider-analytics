# Cloud Provider Analytics

Proyecto Integrador de la materia **Minería de Datos II** de ISTEA – Segundo Cuatrimestre 2026.
Integrantes: Agustina Petrarca y Matías Rodríguez.

## Objetivo

Diseñar e implementar progresivamente una solución de datos para el caso Cloud Provider Analytics, que permita integrar y analizar información de clientes, usuarios, recursos cloud, soporte, facturación, marketing, NPS y eventos de uso.

El proyecto combina ingesta batch de archivos CSV con streaming de eventos JSONL. Spark procesa los datos, Parquet guarda los resultados intermedios y Cassandra, mediante AstraDB, permite consultarlos con CQL.

## Tecnologías principales

- Python y Pandas para exploración de datos.
- PySpark para procesamiento batch.
- Spark Structured Streaming para procesamiento de eventos.
- Parquet para almacenamiento de datos procesados.
- Cassandra / AstraDB para consultas sobre datos preparados.

## Requisitos y configuración

- Google Colab y acceso a Google Drive.
- PySpark 4.0.5 y cassandra-driver. El notebook 06 incluye las celdas de instalación.
- Dataset cloud_provider_challenge_dataset_v1.zip provisto por la materia.
- Una base AstraDB activa con el keyspace finops, su Secure Connect Bundle y un token de aplicación.

En Drive, creá la carpeta MyDrive/cloud-provider-analytics-data/landing/. Copiá ahí el contenido de datalake/landing/ del ZIP, de modo que customers_orgs.csv, users.csv, billing_monthly.csv y usage_events_stream/ queden directamente dentro de landing/.
Para ejecutar 06, subí el ZIP de conexión al entorno temporal de Colab, copiá su ruta en SCB_PATH y guardá el token en Secretos de Colab con el nombre ASTRA_DB_TOKEN. Habilitá el acceso al secreto para ese notebook. Guardá las credenciales y el ZIP de conexión fuera del repositorio.

## Arquitectura propuesta

Se definió una arquitectura híbrida que combina procesamiento batch para las fuentes CSV y streaming para los eventos de uso en formato JSONL.

El Data Lake se organiza en cinco capas:

- **Landing:** conserva los archivos originales sin modificaciones.
- **Bronze:** incorpora los datos con tipos definidos y metadatos de trazabilidad.
- **Silver:** realiza validaciones, limpieza e integración de datos.
- **Gold:** prepara información para análisis de costos, soporte y uso de servicios.
- **Cassandra:** publica el mart para consultas por rango de fechas o por día y servicio

La arquitectura fue diseñada durante el primer parcial. Para la segunda etapa trabajamos en su implementación y validación.

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

## Orden de ejecución
Los notebooks están en notebooks/. Si Colab inicia una sesión nueva, ejecutá primero las celdas de configuración de cada notebook.

- 01 · Exploración: Revisar las fuentes y los problemas de calidad detectados. Es la referencia para entender el dataset.
- 02 · Batch Bronze: Generar los Parquet de organizaciones, usuarios y facturación. Verificar 80 organizaciones, 800 usuarios y 240 facturas.
- 03 · Streaming Bronze: Ejecutar la configuración, la definición del esquema y el bloque de reprocesamiento con conversión de value, watermark y deduplicación por event_id. Verificar 43.200 eventos en bronze/usage_events_reprocesado. Para probar idempotencia, repetir la ejecución con la misma ruta y el mismo checkpoint.
- 04 · Silver: Leer bronze/usage_events_reprocesado, aplicar los controles y verificar 2.244 eventos en quarantine/usage_events. Guardar los 43.200 eventos enriquecidos en silver/usage_events.
- 05 · Gold FinOps: Ejecutar el bloque del mart diario hasta la relectura de gold/org_daily_usage_by_service. Verificar 11.050 filas y los totales del cuadro de resultados.
- 06 · Cassandra Serving: Leer el mart actualizado, conectar a finops, preparar la tabla y cargar los datos. Ejecutar la verificación de totales y las dos consultas CQL. Repetir la carga y la verificación para comprobar que se conservan los conteos y los totales.

## Resultados verificados

Los costos bajo el umbral de −0,01 USD quedan marcados y se conservan en el cálculo financiero para conciliar el costo con Bronze. Los valores de uso sin unidad quedan en cuarentena y se excluyen del cálculo de solicitudes. El carbono se agrega cuando está disponible en los eventos.

## Consultas de Cassandra

La tabla finops.org_daily_usage_by_service usa la organización como clave de partición y la fecha y el servicio para ordenar y distinguir las filas. El notebook 06 contiene el CQL y las salidas de las dos consultas verificadas:
Por rango de fechas. Devuelve costo, eventos, solicitudes y carbono de una organización, desglosados por día y servicio.
Por día y servicio. Recupera una fila específica y compara sus métricas con Gold. En la prueba, el servicio analytics devolvió 238 solicitudes y 0,051855 kg de carbono, los mismos valores del Parquet.

## Estructura del repositorio

- `notebooks/`: notebooks de exploración y análisis.
- `docs/`: documentación y diagramas del proyecto.
- `DECISIONS.md`: registro de decisiones técnicas.
- `README.md`: descripción general y estado del proyecto.
