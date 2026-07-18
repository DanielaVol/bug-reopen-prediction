# Predicción de reapertura de bugs

Repositorio del trabajo **Predicción de reapertura de bugs en proyectos de desarrollo de software**.

El objetivo es estimar, utilizando únicamente información disponible hasta la primera resolución válida de un bug, el riesgo de que el ticket sea reabierto dentro de los 180 días posteriores.

El análisis utiliza proyectos de código abierto gestionados con Jira e incluidos en el dataset TAWOS. Se comparan una regresión logística, Random Forest y LightGBM mediante una evaluación temporal orientada a evitar fuga de información.

## Pregunta de investigación

> ¿En qué medida es posible estimar, utilizando únicamente la información disponible hasta el momento de la primera resolución válida de un bug, el riesgo de que sea reabierto dentro de los 180 días posteriores?

## Diseño del análisis

El procedimiento incluye:

1. reconstrucción de la primera resolución válida a partir del historial de cambios;
2. normalización de estados según el workflow de cada proyecto;
3. generación de atributos disponibles hasta la primera resolución;
4. construcción de un target con horizonte fijo de 180 días;
5. exclusión de observaciones sin seguimiento completo;
6. validación temporal expansiva con una brecha de 180 días;
7. comparación de regresión logística, Random Forest y LightGBM;
8. evaluación mediante PR-AUC, ROC-AUC, precision, recall, F1, balanced accuracy y lift;
9. análisis de escenarios operativos y heterogeneidad entre proyectos.

## Resultados principales

La cohorte final contiene 66.865 bugs, de los cuales 7.094 fueron reabiertos dentro de los 180 días posteriores a su primera resolución.

LightGBM obtuvo el mejor desempeño en el conjunto temporal de prueba:

- PR-AUC: 0,137;
- ROC-AUC: 0,719;
- prevalencia positiva en prueba: 0,055;
- lift aproximado de PR-AUC respecto de la referencia aleatoria: 2,49.

En el escenario que prioriza el 20 % de los bugs con mayor score, el modelo recuperó el 46,06 % de las reaperturas, con una precision de 12,67 % y un lift de 2,30.

Los scores se interpretan como medidas relativas de riesgo y no como probabilidades calibradas. El modelo se propone como una herramienta de apoyo para priorizar revisiones, no como un mecanismo de decisión automática.

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
├── output/                  # generado localmente; no se versiona
├── README.md
├── requirements.txt
├── .env.example
└── .gitignore
```

## Requisitos del sistema

El análisis fue ejecutado con:

- Python 3.12.13;
- MySQL Server 8.x;
- JupyterLab;
- las bibliotecas declaradas en `requirements.txt`.

También se requiere Git para clonar el repositorio.

## Instalación local

### 1. Clonar el repositorio

```bash
git clone https://github.com/DanielaVol/bug-reopen-prediction.git
cd bug-reopen-prediction
```

### 2. Crear un entorno virtual

En Windows PowerShell:

```powershell
py -3.12 -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
python -m ipykernel install --user --name bug-reopen-prediction --display-name "Python (bug-reopen-prediction)"
```

En Linux o macOS:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
python -m ipykernel install --user --name bug-reopen-prediction --display-name "Python (bug-reopen-prediction)"
```

### 3. Configurar MySQL

La base completa de TAWOS no se incluye en el repositorio debido a su tamaño.

Debe descargarse desde una fuente oficial:

- https://rdr.ucl.ac.uk/articles/dataset/The_TAWOS_dataset/21308124
- https://solar.cs.ucl.ac.uk/os/tawos
- https://github.com/SOLAR-group/TAWOS

Una vez importada la base en MySQL, deben agregarse las tablas auxiliares incluidas en:

```text
database/tawos_auxiliary_tables.sql
```

Las instrucciones detalladas se encuentran en [`database/README.md`](database/README.md).

### 4. Configurar las credenciales

Copiar el archivo de ejemplo:

En Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

En Linux o macOS:

```bash
cp .env.example .env
```

Editar `.env` con los datos de la instancia local:

```env
DB_HOST=localhost
DB_PORT=3306
DB_NAME=tawos
DB_USER=usuario_mysql
DB_PASSWORD=contraseña_mysql
```

El archivo `.env` contiene credenciales y no debe incorporarse al repositorio.

### 5. Iniciar JupyterLab

Ejecutar JupyterLab desde la raíz del repositorio:

```bash
jupyter lab
```

Abrir los notebooks con el kernel `Python (bug-reopen-prediction)`.

## Orden de ejecución

Los notebooks deben ejecutarse en este orden:

### 1. `notebooks/01_build_analytical_dataset.ipynb`

Construye el dataset analítico desde MySQL. Entre otras tareas:

- valida la existencia de las tablas requeridas;
- selecciona los proyectos;
- normaliza los estados;
- identifica la primera resolución válida;
- reconstruye atributos disponibles hasta esa fecha;
- identifica la primera reapertura posterior;
- genera atributos temporales e históricos;
- exporta los datasets analíticos y las auditorías.

Principales archivos generados:

```text
output/bug_reopen_dataset_full.parquet
output/bug_reopen_modeling_dataset.parquet
output/bug_reopen_modeling_dataset_with_users.parquet
output/bug_reopen_text_dataset.parquet
```

También se generan versiones CSV y archivos de auditoría.

### 2. `notebooks/02_global_temporal_modeling.ipynb`

Realiza:

- construcción del target con horizonte de 180 días;
- exclusión de bugs sin seguimiento completo;
- análisis exploratorio;
- partición temporal;
- validación temporal expansiva;
- entrenamiento de regresión logística, Random Forest y LightGBM;
- optimización con Optuna;
- selección de umbrales y escenarios por capacidad;
- evaluación final y exportación de resultados.

Los resultados se guardan en:

```text
output/modeling_results/
```

### 3. `notebooks/03_project_specific_models.ipynb`

Entrena y evalúa modelos separados por proyecto y compara su desempeño con el modelo global.

Debe ejecutarse después del notebook principal porque utiliza datasets y resultados generados previamente.

## Controles de reproducción

Después de ejecutar el primer notebook deberían obtenerse, entre otros, los siguientes valores:

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

Pequeñas diferencias en tiempos de ejecución son esperables. Los conteos y la composición de la cohorte no deberían variar cuando se utiliza la misma versión de TAWOS y las mismas tablas auxiliares.

## Configuración de rutas

Las versiones iniciales de los notebooks fueron ejecutadas en un entorno cuyo directorio de trabajo era `/app`. Para una ejecución local sin Docker, los notebooks deben utilizar la raíz del repositorio como directorio base.

La celda de configuración del primer notebook debe definir las rutas de manera portable, por ejemplo:

```python
from pathlib import Path

PROJECT_ROOT = Path.cwd().resolve()
if PROJECT_ROOT.name == "notebooks":
    PROJECT_ROOT = PROJECT_ROOT.parent

OUTPUT_DIR = PROJECT_ROOT / "output"
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)

load_dotenv(PROJECT_ROOT / ".env")
```

Los notebooks de modelado deben buscar los datos en:

```python
PROJECT_ROOT / "output"
```

Antes de considerar finalizada la publicación del repositorio, se recomienda aplicar este cambio directamente en los tres notebooks para que ninguna ejecución dependa de `/app` ni de `/mnt/data`.

## Datos y archivos no versionados

No se incluyen:

- el dump completo de TAWOS;
- los datasets analíticos generados;
- estudios Optuna;
- modelos entrenados;
- credenciales;
- archivos temporales de Jupyter.

Estos elementos deben permanecer excluidos mediante `.gitignore`.

## Reproducibilidad de versiones

`requirements.txt` declara un conjunto compatible de dependencias para Python 3.12. Para registrar las versiones exactas del entorno utilizado en la ejecución final, puede generarse un archivo de bloqueo con:

```bash
python -m pip freeze > requirements-lock.txt
```

Ese archivo debe generarse después de ejecutar satisfactoriamente los tres notebooks y puede incorporarse al repositorio junto con `requirements.txt`.

## Referencia del dataset

Tawosi, V., Al-Subaihin, A., Moussa, R. y Sarro, F. (2022). *A versatile dataset of agile open source software projects*. Proceedings of the 19th International Conference on Mining Software Repositories, 707-711. https://doi.org/10.1145/3524842.3528029

