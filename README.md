# Predicción de reapertura de bugs

Repositorio del trabajo **Predicción de reapertura de bugs en proyectos de desarrollo de software**.

El objetivo es estimar, a partir de la información disponible hasta la primera resolución válida de un bug, el riesgo de que el ticket sea reabierto dentro de los 180 días posteriores.

El análisis utiliza proyectos de código abierto gestionados con Jira e incluidos en el dataset TAWOS. Se comparan una regresión logística, Random Forest y LightGBM mediante una evaluación temporal diseñada para evitar fuga de información.

## Pregunta de investigación

> ¿En qué medida es posible estimar, utilizando únicamente la información disponible hasta el momento de la primera resolución válida de un bug, el riesgo de que sea reabierto dentro de los 180 días posteriores?

## Metodología

El flujo de trabajo comprende:

1. reconstrucción de la primera resolución válida a partir del historial de cambios;
2. normalización de estados según el workflow de cada proyecto;
3. generación de atributos disponibles hasta la primera resolución;
4. construcción de la variable objetivo con un horizonte fijo de 180 días;
5. exclusión de observaciones sin seguimiento completo;
6. partición y validación temporal expansiva con una brecha de 180 días;
7. entrenamiento de regresión logística, Random Forest y LightGBM;
8. evaluación mediante PR-AUC, ROC-AUC, precision, recall, F1, balanced accuracy y lift;
9. análisis de escenarios operativos y de heterogeneidad entre proyectos.

## Resultados principales

La cohorte final contiene 66.865 bugs, de los cuales 7.094 fueron reabiertos dentro de los 180 días posteriores a su primera resolución.

En el conjunto temporal de prueba, LightGBM obtuvo el mejor desempeño:

- PR-AUC: 0,137;
- ROC-AUC: 0,719;
- prevalencia positiva: 0,055.

Al priorizar el 20 % de los bugs con mayor score, el modelo recuperó el 46,06 % de las reaperturas, con una precision de 12,67 % y un lift de 2,30.

Los scores se interpretan como medidas relativas de riesgo. El modelo se propone como herramienta de apoyo para priorizar revisiones y pruebas adicionales, no como mecanismo de decisión automática.

## Estructura del repositorio

```text
bug-reopen-prediction/
├── notebooks/
│   ├── 01_build_analytical_dataset.ipynb
│   ├── 02_global_temporal_modeling.ipynb
│   └── 03_project_specific_models.ipynb
├── database/
│   ├── tawos_auxiliary_tables.sql
│   └── README.md
├── README.md
├── requirements.txt
├── requirements-lock.txt
├── .env.example
└── .gitignore
```

La carpeta `output/` se crea automáticamente durante la ejecución y no se versiona.

## Requisitos

Para reproducir el análisis se necesita:

- Git;
- Python 3.12;
- MySQL Server 8 o superior;
- acceso local a la base TAWOS.

El entorno de referencia fue ejecutado con Python 3.12.13 y MySQL Server 9.2.0.

## Instalación

### 1. Clonar el repositorio

```bash
git clone https://github.com/DanielaVol/bug-reopen-prediction.git
cd bug-reopen-prediction
```

### 2. Crear un entorno virtual

#### Windows PowerShell

```powershell
py -3.12 -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

#### Linux o macOS

```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

### 3. Instalar el entorno utilizado en el análisis

Para reproducir las versiones exactas del entorno:

```bash
python -m pip install -r requirements-lock.txt
```

Registrar el entorno como kernel de Jupyter:

```bash
python -m ipykernel install --user --name bug-reopen-prediction --display-name "Python (bug-reopen-prediction)"
```

`requirements-lock.txt` contiene las versiones exactas utilizadas en la ejecución de referencia.

`requirements.txt` contiene únicamente las dependencias principales con rangos de versiones compatibles. Se conserva como referencia para mantenimiento o actualización del proyecto, pero la reproducción del trabajo debe realizarse con `requirements-lock.txt`.

## Preparación de la base de datos

### 1. Descargar TAWOS

La base completa no se incluye en este repositorio debido a su tamaño. Puede obtenerse desde las fuentes oficiales:

- [TAWOS en UCL Research Data Repository](https://rdr.ucl.ac.uk/articles/dataset/The_TAWOS_dataset/21308124)
- [Sitio del proyecto TAWOS](https://solar.cs.ucl.ac.uk/os/tawos)
- [Repositorio oficial de TAWOS](https://github.com/SOLAR-group/TAWOS)

### 2. Importar TAWOS en MySQL

Importar el dump descargado en una base local. El nombre utilizado por defecto en este proyecto es:

```text
tawos
```

Las instrucciones detalladas se encuentran en [`database/README.md`](database/README.md).

### 3. Cargar las tablas auxiliares

Después de importar TAWOS, ejecutar sobre la misma base:

```text
database/tawos_auxiliary_tables.sql
```

El archivo incorpora las tablas:

- `analysis_project_scope`;
- `status_group`;
- `status_map_project`.

Estas tablas documentan los proyectos seleccionados y la normalización de los estados utilizada para reconstruir resoluciones y reaperturas.

## Configuración de la conexión

Copiar el archivo de ejemplo.

### Windows PowerShell

```powershell
Copy-Item .env.example .env
```

### Linux o macOS

```bash
cp .env.example .env
```

Editar únicamente el archivo `.env` con las credenciales de la base local:

```env
DB_HOST=localhost
DB_PORT=3306
DB_NAME=tawos
DB_USER=usuario_mysql
DB_PASSWORD=contraseña_mysql
```

El archivo `.env` no se versiona.

No es necesario modificar rutas ni código dentro de los notebooks.

## Ejecución

Desde la raíz del repositorio, con el entorno virtual activado:

```bash
jupyter lab
```

Seleccionar el kernel:

```text
Python (bug-reopen-prediction)
```

Ejecutar los notebooks en este orden:

### 1. Construcción del dataset analítico

```text
notebooks/01_build_analytical_dataset.ipynb
```

Este notebook:

- se conecta a MySQL;
- valida la existencia de las tablas requeridas;
- identifica la primera resolución válida;
- reconstruye los atributos disponibles hasta esa fecha;
- identifica la primera reapertura posterior;
- genera los datasets analíticos.

### 2. Modelado temporal global

```text
notebooks/02_global_temporal_modeling.ipynb
```

Este notebook:

- construye el target a 180 días;
- aplica la cohorte con seguimiento completo;
- realiza el análisis exploratorio;
- genera la partición temporal;
- entrena y optimiza los modelos;
- evalúa el conjunto temporal de prueba;
- exporta métricas y resultados.

### 3. Modelos específicos por proyecto

```text
notebooks/03_project_specific_models.ipynb
```

Este notebook entrena modelos separados por proyecto y compara su desempeño con el modelo global.

## Archivos generados

El primer notebook crea, entre otros:

```text
output/bug_reopen_dataset_full.parquet
output/bug_reopen_modeling_dataset.parquet
output/bug_reopen_modeling_dataset_with_users.parquet
output/bug_reopen_text_dataset.parquet
```

Los notebooks de modelado almacenan sus resultados en:

```text
output/modeling_results/
```

La carpeta `output/` está excluida del repositorio porque contiene datos y artefactos generados durante la ejecución.

## Controles de reproducción

Al finalizar la construcción del dataset deberían obtenerse los siguientes valores:

| Control | Valor esperado |
|---|---:|
| Bugs con primera resolución válida | 67.806 |
| Filas del dataset completo | 67.806 |
| Columnas del dataset completo | 198 |
| Filas del dataset de modelado base | 67.806 |
| Columnas del dataset de modelado base | 175 |

Después de aplicar el horizonte de 180 días:

| Control | Valor esperado |
|---|---:|
| Bugs elegibles | 66.865 |
| Reabiertos dentro de 180 días | 7.094 |
| No reabiertos dentro de 180 días | 59.771 |
| Prevalencia positiva | 10,61 % |

Diferencias en estos conteos pueden indicar que se utilizó otra versión de TAWOS, que la importación está incompleta o que no se cargaron correctamente las tablas auxiliares.

## Datos no incluidos

El repositorio no contiene:

- el dump completo de TAWOS;
- credenciales de acceso;
- datasets analíticos generados;
- estudios de Optuna;
- modelos entrenados;
- archivos temporales de Jupyter.

## Referencia del dataset

Tawosi, V., Al-Subaihin, A., Moussa, R. y Sarro, F. (2022). *A versatile dataset of agile open source software projects*. Proceedings of the 19th International Conference on Mining Software Repositories, 707-711. https://doi.org/10.1145/3524842.3528029

