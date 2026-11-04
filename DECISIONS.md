# Registro de Decisiones Técnicas

## Proyecto Cloud Provider Analytics

Este documento reúne las principales decisiones tomadas durante el diseño del proyecto integrador de Minería de Datos II.

El objetivo es explicar qué tecnologías y estructuras elegimos, por qué las consideramos adecuadas y qué aspectos tendremos que validar durante la implementación.

## 1. Arquitectura híbrida: batch y streaming

**Decisión:** utilizar procesamiento batch para las fuentes CSV y streaming para los eventos de uso en JSONL.

**Justificación:** los archivos de clientes, usuarios, facturación y soporte pueden procesarse por lotes, mientras que los eventos de uso permiten simular información que llega continuamente.

**Alternativa considerada:** procesar todas las fuentes mediante batch. Sería más sencillo de implementar, pero no permitiría trabajar con las características del procesamiento en tiempo real.

**Estado:** diseño definido; implementación pendiente.

## 2. Organización del Data Lake

**Decisión:** organizar los datos en cuatro capas: Landing, Bronze, Silver y Gold.

**Justificación:** esta separación permite conservar los archivos originales, procesarlos progresivamente y mantener diferenciados los datos sin transformar de aquellos preparados para análisis.

**Alternativa considerada:** almacenar directamente los datos limpios. Esto reduciría la cantidad de capas, pero dificultaría recuperar información original o repetir transformaciones.

**Estado:** diseño definido; implementación pendiente.

## 3. Formato de almacenamiento

**Decisión:** mantener los archivos originales en Landing y utilizar Parquet para las capas procesadas.

**Justificación:** Parquet permite almacenar los datos por columnas y facilita las consultas y transformaciones con Spark.

**Alternativa considerada:** utilizar CSV en todas las capas. Es un formato sencillo, pero presenta limitaciones para conservar tipos de datos y realizar consultas analíticas eficientes.

**Estado:** decisión de diseño.

## 4. Particionamiento de datos

**Decisión:** no particionar inicialmente las fuentes maestras pequeñas. Para los eventos de uso se propone particionar por fecha.

**Justificación:** las fuentes pequeñas no necesitan divisiones adicionales. En cambio, los eventos pueden crecer y las particiones por fecha permitirían consultar períodos específicos sin recorrer toda la información.

**Alternativa considerada:** particionar todas las fuentes. Esto agregaría complejidad y podría generar muchos archivos pequeños.

**Estado:** pendiente de validar durante la implementación.

## 5. Calidad de datos

**Decisión:** conservar los registros originales en Landing y aplicar las validaciones en las capas de procesamiento.

Los controles incluirán valores nulos, duplicados, tipos de datos, rangos válidos y referencias entre fuentes.

En `customers_orgs.nps_score` se utilizará el rango de -100 a 100, porque representa un índice NPS agregado. Los valores negativos son válidos y el valor 101 detectado deberá marcarse como fuera de rango.

**Alternativa considerada:** eliminar directamente los registros inválidos. Se descarta porque podría provocar pérdida de información y dificultar el análisis de los errores.

**Estado:** exploración realizada; reglas de procesamiento pendientes.

## 6. Trazabilidad y tratamiento de errores

**Decisión propuesta:** incorporar metadatos como `ingest_ts`, `source_file` y `schema_version` en Bronze.

También se propone separar los registros que no cumplan las validaciones en una zona de quarantine, conservando el motivo del rechazo y el archivo de origen.

**Justificación:** esto permitirá identificar de dónde proviene cada registro y revisar errores sin modificar los archivos originales.

**Alternativa considerada:** descartar los registros inválidos durante la carga. No se considera conveniente porque impediría investigar los problemas posteriormente.

**Estado:** pendiente de implementación.

## 7. Procesamiento de eventos

**Decisión propuesta:** utilizar Spark Structured Streaming con micro-batches para procesar los archivos JSONL.

Durante la implementación se evaluarán mecanismos de checkpoint, deduplicación por identificador de evento y manejo de eventos tardíos.

**Justificación:** buscamos evitar procesamientos duplicados y poder recuperar la ejecución cuando se interrumpa.

**Alternativa considerada:** procesar los JSONL como un único lote. Sería más simple, pero no permitiría demostrar el comportamiento de streaming.

**Estado:** diseño preliminar; parámetros y pruebas pendientes.

## 8. Capa de consulta

**Decisión:** preparar datasets analíticos en Gold y utilizar Cassandra/AstraDB como tecnología de serving.

**Justificación:** se busca separar el procesamiento de datos de su consulta posterior.

**Alternativa considerada:** consultar directamente los archivos Parquet. Es útil para exploración, pero no representa la misma arquitectura de serving prevista para el proyecto.

**Estado:** diseño definido; implementación pendiente.

## 9. Retención y reprocesamiento

**Decisión propuesta para el proyecto académico:** conservar los archivos originales de Landing durante todo el desarrollo y permitir regenerar las capas Bronze, Silver y Gold.

La política de retención para un entorno productivo todavía no está definida y requeriría considerar volumen, costos y necesidades del negocio.

**Justificación:** mantener los datos originales permite repetir procesos y corregir transformaciones sin perder información.

**Estado:** política académica propuesta; pendiente de validación.

## 10. Próximos pasos

Durante el segundo parcial se buscará:

- Implementar los procesos batch con PySpark.
- Desarrollar el procesamiento de eventos con Structured Streaming.
- Incorporar validaciones de calidad y quarantine.
- Probar la deduplicación, los checkpoints y el reprocesamiento.
- Construir los datasets analíticos de Gold.
- Documentar resultados, pruebas y decisiones adicionales.

Las decisiones podrán revisarse si durante la implementación encontramos limitaciones técnicas o alternativas más adecuadas.
