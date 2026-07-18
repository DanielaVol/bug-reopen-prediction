# Preparación de la base de datos

Este directorio contiene las tablas auxiliares necesarias para reproducir la selección de proyectos y la normalización de estados utilizada en el análisis.

La base completa de TAWOS no se incluye en el repositorio debido a su tamaño.

## 1. Obtener TAWOS

El dataset puede obtenerse desde sus fuentes oficiales:

- https://rdr.ucl.ac.uk/articles/dataset/The_TAWOS_dataset/21308124
- https://solar.cs.ucl.ac.uk/os/tawos
- https://github.com/SOLAR-group/TAWOS

La publicación asociada es:

Tawosi, V., Al-Subaihin, A., Moussa, R. y Sarro, F. (2022). *A versatile dataset of agile open source software projects*. Proceedings of the 19th International Conference on Mining Software Repositories, 707-711. https://doi.org/10.1145/3524842.3528029

## 2. Importar la base en MySQL

Se recomienda MySQL Server 8.x.

### Opción A: MySQL Workbench

1. Abrir MySQL Workbench.
2. Conectarse a la instancia local.
3. Crear un schema para TAWOS, si el dump no lo crea automáticamente.
4. Ingresar en **Server → Data Import**.
5. Seleccionar el dump descargado.
6. Elegir el schema de destino.
7. Ejecutar **Start Import**.

### Opción B: línea de comandos

Cuando el dataset se distribuye como un dump SQL:

```bash
mysql -u USUARIO -p NOMBRE_BASE < tawos.sql
```

El nombre exacto del archivo depende de la distribución descargada.

## 3. Verificar las tablas originales

El primer notebook requiere las siguientes tablas de TAWOS:

```text
project
repository
sprint
version
affected_version
fix_version
issue
issue_link
user
comment
change_log
issue_component
component
```

La consulta siguiente permite comprobar su existencia:

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = DATABASE()
ORDER BY table_name;
```

## 4. Importar las tablas auxiliares

Después de importar TAWOS, ejecutar:

```text
database/tawos_auxiliary_tables.sql
```

En MySQL Workbench:

1. abrir el archivo SQL;
2. seleccionar como schema activo la base de TAWOS;
3. ejecutar todo el script;
4. confirmar que no se produzcan errores.

Las tablas incorporadas son:

### `analysis_project_scope`

Define los proyectos incluidos en el análisis.

### `status_group`

Define grupos funcionales de estados, por ejemplo actividad, validación, resolución, cierre sin corrección y reapertura explícita.

### `status_map_project`

Asigna cada estado observado en cada proyecto a un grupo funcional. Esta tabla contiene la clasificación necesaria para interpretar workflows heterogéneos sin asumir que una misma etiqueta posee el mismo significado en todos los proyectos.

## 5. Verificar las tablas auxiliares

```sql
SELECT COUNT(*) AS projects
FROM analysis_project_scope;

SELECT COUNT(*) AS status_groups
FROM status_group;

SELECT COUNT(*) AS mapped_statuses
FROM status_map_project;
```

También puede verificarse la integridad referencial de los proyectos:

```sql
SELECT s.Project_ID
FROM analysis_project_scope AS s
LEFT JOIN project AS p
    ON p.ID = s.Project_ID
WHERE p.ID IS NULL;
```

La consulta no debería devolver filas.

## 6. Configurar la conexión

En la raíz del repositorio, copiar `.env.example` como `.env` y completar:

```env
DB_HOST=localhost
DB_PORT=3306
DB_NAME=tawos
DB_USER=usuario_mysql
DB_PASSWORD=contraseña_mysql
```

El nombre indicado en `DB_NAME` debe coincidir con el schema donde se encuentran tanto las tablas originales como las auxiliares.

## 7. Ejecutar la construcción del dataset

Desde la raíz del repositorio:

```bash
jupyter lab
```

Abrir y ejecutar:

```text
notebooks/01_build_analytical_dataset.ipynb
```

El notebook valida las tablas antes de continuar. Si falta alguna, detiene la ejecución e informa su nombre.

## 8. Controles esperados

Con la versión de TAWOS utilizada en el trabajo:

- se identifican 67.806 bugs con primera resolución válida;
- el dataset completo contiene 67.806 filas y 198 columnas;
- el dataset de modelado base contiene 67.806 filas y 175 columnas;
- después de exigir 180 días completos de seguimiento quedan 66.865 bugs;
- 7.094 de esos bugs fueron reabiertos dentro del horizonte.

Una diferencia en estos valores puede indicar:

- otra versión de TAWOS;
- tablas auxiliares distintas;
- un schema incorrecto;
- una importación incompleta;
- cambios en las reglas de reconstrucción.

## Seguridad

No incorporar al repositorio:

- dumps completos de la base;
- archivos `.env`;
- contraseñas;
- usuarios administrativos;
- datos generados de gran tamaño.
