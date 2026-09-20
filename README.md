# TechGrow 

Sistema de analítica predictiva y modelos de transaccionalidad por estación, diseñado bajo estándares de arquitectura de software, infraestructura segura y gestión eficiente de grandes volúmenes de datos.

## 📂 Estructura del Repositorio

El proyecto se encuentra organizado de manera modular para separar la ingesta de datos, la documentación, los artefactos de machine learning, la experimentación y el código fuente de infraestructura:

```text
TechGrow/
├── datos/                  # Datasets del proyecto (gestionados con Git LFS)
├── docs/                   # Documentación técnica y arquitectura de la solución
├── modelo/                 # Artefactos y modelos serializados (.joblib)
├── notebooks/              # Jupyter Notebooks de análisis exploratorio y modelado
├── src/                    # Código fuente, definiciones de infraestructura y flujos
├── .gitattributes          # Configuración de seguimiento para Git LFS
├── README.md               # Documentación principal del repositorio
└── REQUIREMENTS.md         # Dependencias y requerimientos del entorno
```

## Componentes

* **Analítica y Modelado (`/notebooks` & `/modelo`)**: Contiene los notebooks de entrenamiento y evaluación, junto con los modelos serializados (`.joblib`) orientados a la predicción y transaccionalidad por estación y sucursal.
* **Infraestructura y Orquestación (`/src`)**: Incluye la definición de la máquina de estados y plantillas de despliegue (`template.yaml` y `statemachine_workflow.asl.json`).
* **Documentación (`/docs`)**: Manuales de usuario y diagramas de arquitectura detallados (`SOLUTION_ARCHITECTURE.md`, `USER_MANUAL.md`).

## Configuración y Despliegue Local

1. **Clonar el repositorio y configurar Git LFS:**
   ```bash
   git clone https://github.com/JTAndes/TechGrow.git
   cd TechGrow
   git lfs install
   git lfs pull
   ```

2. **Instalar dependencias:**
   Consulta el archivo [REQUIREMENTS.md](REQUIREMENTS.md) para configurar tu entorno virtual e instalar las librerías necesarias.

3. **Ejecución:**
   Explora los notebooks en la carpeta `/notebooks` utilizando Jupyter Lab o VS Code para replicar las ejecuciones y análisis.
