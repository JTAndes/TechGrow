# TechGrow - Requerimientos y Dependencias 📦

Este documento detalla las dependencias de software, librerías de Machine Learning y herramientas de infraestructura necesarias para ejecutar correctamente el proyecto **TechGrow** en un entorno local (macOS / Linux / Windows).

## 🐍 Requisitos del Sistema
* **Python**: `3.10` o superior (Recomendado gestionar mediante Anaconda/Miniconda).
* **Git LFS**: Obligatorio para la descarga y control de versiones de datasets y modelos pesados (`.csv`, `.joblib`).

## 📚 Dependencias de Python (`requirements.txt`)

Copia y guarda las siguientes librerías en un archivo `requirements.txt` o instálalas directamente:

```text
# Análisis de datos y manipulación
pandas>=2.0.0
numpy>=1.24.0

# Modelado y Machine Learning
scikit-learn>=1.2.0
joblib>=1.2.0

# Visualización
matplotlib>=3.7.0
seaborn>=0.12.0

# Entorno de desarrollo interactivo
jupyter>=1.0.0
notebook>=6.5.0
ipykernel>=6.20.0
```

## 🚀 Instalación Rápida del Entorno

Si estás configurando tu entorno local desde cero en la terminal, ejecuta los siguientes comandos:

```bash
# 1. Crear y activar un entorno virtual con Conda (Recomendado)
conda create -n techgrow-env python=3.10 -y
conda activate techgrow-env

# 2. Instalar las dependencias del proyecto
pip install -r requirements.txt
```