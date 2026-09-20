# Documento de Arquitectura de Solución - TechGrow

## 1. Visión General del Proyecto
Este proyecto implementa una arquitectura serverless, basada en eventos y orientada a microservicios en AWS diseñada específicamente para la predicción de demanda de restaurantes (volumen de platos a vender). La solución automatiza la ingesta de datos externos (clima, tráfico, festivos), el procesamiento masivo con una arquitectura de datos en capas (Medallón), la inferencia de modelos de Machine Learning, y la persistencia de resultados optimizados para la visualización ejecutiva en un dashboard de negocio.

---

## 2. Diagrama de Arquitectura de Solución
<img width="815" height="507" alt="image" src="https://github.com/user-attachments/assets/873cc84c-9f6e-4fe2-ac88-dc42c2f96035" />



---

## 3. Flujo de Datos y Orquestación (Extremo a Extremo)
El pipeline opera de manera totalmente automatizada bajo el siguiente flujo:
1. **Disparador (EventBridge Scheduler):** Un evento programado mediante expresión cron (`cron(0 6 ? * MON *)`) activa semanalmente la ejecución de la máquina de estados.
2. **Orquestación Paralela (Step Functions):** Coordina de forma concurrente dos tareas críticas:
   - **Rama de Datos (Glue ETL):** Ejecuta el Glue Job para procesar y transformar los datos a través de la arquitectura medallón (`Bronze -> Silver -> Gold`) dentro del bucket S3 central.
   - **Rama de Contexto Externo (Lambda APIs):** Consulta los servicios externos (Meteo, Festivos, Waze, etc.) y persiste los datos contextuales en el Data Lake.
3. **Sincronización:** Step Functions espera obligatoriamente a que ambas ramas concluyan de forma exitosa antes de avanzar.
4. **Inferencia del Modelo (Lambda Modelo):** Una vez sincronizada la información de la capa Gold y el contexto de las APIs, se ejecuta la función Lambda encargada de aplicar el modelo predictivo.
5. **Persistencia Final (S3 Modelo):** Los resultados estructurados y limpios son depositados en el bucket S3 dedicado (`techgrow-modelo`).
6. **Consumo de Negocio (QuickSight Dashboard):** El servicio de Business Intelligence se conecta directamente a este bucket final para renderizar las métricas y predicciones de forma limpia y directa para los tomadores de decisiones.

---

## 4. Desglose Técnico de la Infraestructura (`template.yaml`)

El archivo de infraestructura como código (IaC) utiliza **AWS SAM / CloudFormation** y está compuesto por los siguientes recursos principales:

### A. Capas de Almacenamiento (S3 Buckets)
* **`DataLakeBucket` (`techgrow-medallon`):** Almacena todo el flujo de datos estructurado en la arquitectura medallón (carpetas `bronze/`, `silver/`, `gold/`) y el contexto recolectado de APIs (`apis-context/`). Cuenta con bloqueo total de acceso público por seguridad.
* **`ModelPredictionsBucket` (`techgrow-modelo`):** Bucket aislado exclusivamente para almacenar las salidas procesadas del modelo predictivo, sirviendo como fuente de datos optimizada para QuickSight.

### B. Procesamiento de Datos (AWS Glue & Data Catalog)
* **`GlueMedallionJob` (`techgrow-glue-medallon-etl`):** Script ETL en Python ejecutado en entornos serverless de Glue para la limpieza y modelado de los datos tabulares.
* **`DataLakeGlueDatabase` & `GoldTableCrawler`:** Base de datos metastore de Glue y un Crawler automatizado programado para escanear periódicamente la capa `gold/`, manteniendo actualizado el catálogo de datos de manera automática.

### C. Funciones de Computación (AWS Lambda)
* **`ApiCollectorLambda` (`techgrow-api-collector`):** Función optimizada para realizar peticiones concurrentes a APIs externas e ingerir los datos climáticos y de movilidad.
* **`PredictionModelLambda` (`techgrow-prediction-model`):** Función con mayor capacidad de memoria (1024 MB) y timeout extendido para cargar y ejecutar la lógica de inferencia sobre la data consolidada.

### D. Orquestación y Automatización
* **`PredictionStateMachine` (Step Functions):** Define la lógica de control mediante el archivo `workflow.asl.json`, gestionando paralelismos, reintentos y control de errores.
* **`WeeklyEventBridgeSchedule`:** Automatiza el despertar del pipeline en horarios estratégicos sin requerir servidores encendidos 24/7.

---

## 5. Estructura de Orquestación (`workflow.asl.json`)
La máquina de estados implementa el siguiente comportamiento lógico:
* **`Parallel` State:** Dispara simultáneamente el Glue Job de transformación de datos y la recolección de APIs.
* **Sincronización `.sync`:** Utiliza la integración nativa de SDK para esperar la finalización exacta del job de Glue.
* **Task final:** Desencadena la Lambda de inferencia del modelo una vez las dependencias de datos están listas y validadas.

---

## 6. Consumo y Visualización de Resultados
El componente encargado de mostrar los resultados finales a los usuarios de negocio es el **Dashboard de QuickSight**. Este se conecta al bucket S3 `techgrow-modelo` mediante el catálogo de Glue, presentando una interfaz ejecutiva enfocada exclusivamente en los valores operativos clave (como la cantidad de platos estimados a vender), abstraída por completo de la complejidad técnica del pipeline subyacente.
