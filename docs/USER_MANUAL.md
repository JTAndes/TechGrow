# Manual de Usuario - TechGrow

## 1. Introducción
El sistema **TechGrow** es una plataforma automatizada basada en serverless, analítica en la nube y seguridad perimetral en AWS, orientada a predecir las transacciones en restaurantes. Este manual guía tanto al equipo técnico como al personal de negocio en la operación, supervisión y consumo de los resultados del sistema.

---

## 2. Roles de Usuario y Accesos
* **Administrador / Ingeniero de Datos:** Responsable de desplegar la infraestructura (`template.yaml`), monitorear el aislamiento de red (VPC/WAF), las ejecuciones en Step Functions y verificar la integridad de los datos en el Data Lake.
* **Analista de Negocio / Tomador de Decisiones:** Usuario final encargado de visualizar los tableros ejecutivos en Amazon QuickSight para la planificación operativa y de inventarios.

---

## 3. Guía para el Administrador / Operador Técnico

### 3.1 Verificación del Pipeline Automatizado
El sistema opera de forma autónoma todos los lunes a las 6:00 AM gracias a **Amazon EventBridge Scheduler** y su rol dedicado, activando la máquina de estados en **AWS Step Functions**.

1. **Monitorear ejecuciones:**
   * Ingresa a la consola de AWS y busca **Step Functions**.
   * Selecciona la máquina de estados llamada `techgrow-pipeline-orquestador`.
   * En la sección de *Executions*, podrás verificar si las ejecuciones semanales concluyeron de forma exitosa (`Succeeded`) o si ocurrieron fallos (`Failed`).

2. **Entendiendo el Flujo Paralelo:**
   * La máquina de estados ejecuta de forma concurrente el procesamiento de datos del **Glue Job** (`techgrow-glue-medallon-etl`) y la recolección de contexto de APIs externas mediante la **Lambda** (`techgrow-api-collector`) dentro de la subred privada.
   * Una vez ambas tareas finalizan, se dispara automáticamente la **Lambda de Inferencia** (`techgrow-prediction-model`), la cual deposita el resultado limpio en el bucket S3 final (`techgrow-modelo`).

### 3.2 Despliegue de la Infraestructura
Si necesitas actualizar o desplegar la infraestructura de red aislada (VPC), seguridad perimetral (WAF), roles de gobierno (IAM) y los recursos de cómputo, utiliza AWS SAM:
bash
sam build
sam deploy --guided

---

### 3.3 Validación de Resultados y Logs del Pipeline
Si necesitas depurar una ejecución fallida o verificar que los datos llegaron correctamente:
1. **Inspecciona los Logs en CloudWatch:** Busca los grupos de logs correspondientes a las funciones Lambda (`/aws/lambda/techgrow-api-collector` o `/aws/lambda/techgrow-prediction-model`) para rastrear errores de ejecución o de conexión.
2. **Verifica el Data Lake (S3):** Revisa que el bucket `techgrow-medallon` contenga las particiones actualizadas en las carpetas `bronze/`, `silver/` y `gold/`.

---

## 4. Guía para el Usuario de Negocio (Consumo de Reportes)

Los tomadores de decisiones y analistas de negocio requieren interactuar a través del panel visual.

### 4.1 Acceso al Dashboard Ejecutivo
1. Ingresa a **Amazon QuickSight** utilizando tus credenciales corporativas asignadas.
2. Dirígete al menú lateral de **Dashboards** (Tableros) y selecciona el reporte gerencial titulado **TechGrow - Predicción de Demanda de Platos**.

### 4.2 Interpretación de Métricas Clave
* **Volumen Estimado de Platos:** Visualiza de forma gráfica la proyección de la demanda de cada plato para la semana en curso, desglosada por sucursales.
* **Variables Contextuales de Impacto:** Analiza cómo las variables exógenas (como el clima pronosticado, días festivos y tendencias de movilidad) afectaron directamente la predicción generada por el modelo.
* **Filtros Temporales y Geográficos:** Utiliza los selectores superiores del reporte para comparar las corridas actuales frente al comportamiento histórico de semanas anteriores.

---

## 5. Soporte y Resolución de Problemas (Troubleshooting)

¿Qué sucede si el dashboard no muestra datos nuevos los lunes por la mañana?

1. **Paso 1:** Ingresa a **AWS Step Functions** y revisa el historial de la máquina de estados `techgrow-pipeline-orquestador`. Si el estado final es `Failed`, haz clic sobre la ejecución para identificar cuál de las tareas (Glue Job o Lambda de APIs) falló.
2. **Paso 2:** Si el pipeline terminó en `Succeeded` pero el dashboard sigue sin actualizarse, verifica que el **Glue Crawler** se haya ejecutado correctamente para refrescar el catálogo de datos (`techgrow_medallon_db`) y que la capa `gold/` tenga la partición del día actual.
