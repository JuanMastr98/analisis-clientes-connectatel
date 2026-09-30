# Análisis de datos de clientes de ConnectaTel

Proyecto de análisis exploratorio en Python sobre los usuarios y el uso del servicio de una empresa de telecomunicaciones (ConnectaTel). Incluye limpieza de datos, segmentación de clientes y recomendaciones de negocio.

## Objetivo

Entender cómo usan el servicio los clientes de ConnectaTel y cómo se relaciona ese uso con su edad y su plan, para:

- Detectar y corregir problemas de calidad en los datos.
- Segmentar a los clientes por nivel de uso y por grupo de edad.
- Identificar patrones de uso extremo (outliers) y qué implican para el negocio.
- Proponer mejoras a la oferta actual de planes o nuevos planes.

## Datasets utilizados

| Dataset | Contenido | Tamaño | Columnas |
|---|---|---|---|
| `users` | Un registro por cliente | 4,000 filas | `user_id`, `first_name`, `last_name`, `age`, `city`, `reg_date`, `plan`, `churn_date` |
| `usage` | Un registro por llamada o mensaje | 40,000 filas | `id`, `user_id`, `type`, `date`, `duration`, `length` |

- `user_id` es la llave que une ambas tablas.
- `type` indica si el registro es una llamada (`call`) o un mensaje (`text`). `duration` aplica solo a llamadas y `length` solo a mensajes.
- `plan` tiene dos categorías: Básico y Premium.
- Los registros de uso corresponden a 2024; los usuarios se registraron entre 2022 y 2024.

## Etapas del análisis

1. **Carga y diagnóstico de nulos.** Conteo y porcentaje de valores faltantes por columna en ambas tablas.
2. **Limpieza de valores centinela y fechas.**
   - `age`: el valor -999 se reemplazó por la mediana de las edades válidas.
   - `city`: el valor `"?"` se reemplazó por nulo.
   - `reg_date` y `date`: conversión a tipo fecha con `errors='coerce'`; las fechas fuera de rango (año 2026 en `reg_date`) se marcaron como nulas.
3. **Análisis de nulos en `duration` y `length`.** Se verificó que son MAR respecto a `type` (nulos estructurales) y se dejaron como nulos.
4. **Exploración de variables categóricas.** Valores únicos de `city`, `plan` y `type`.
5. **Agregación y unión.** Se agregó `usage` por `user_id` (`cant_mensajes`, `cant_llamadas`, `cant_minutos_llamada`) y se unió con `users` en el dataframe `user_profile`.
6. **Estadística descriptiva.** Resumen numérico y distribución porcentual por plan.
7. **Distribuciones por plan.** Histogramas de `age`, `cant_mensajes`, `cant_llamadas` y `cant_minutos_llamada`.
8. **Detección de outliers.** Boxplots y método IQR; se decidió conservar los valores extremos por ser usuarios con uso intensivo.
9. **Segmentación.** Se crearon dos columnas:
   - `grupo_uso`: Bajo uso, Uso medio o Alto uso.
   - `grupo_edad`: Joven (menos de 30), Adulto (menos de 60) o Adulto Mayor.
10. **Visualización y conclusiones.** Gráficos de los segmentos y análisis ejecutivo con recomendaciones.

## Cómo ejecutar el notebook en Google Colab

1. Entra a [Google Colab](https://colab.research.google.com) con tu cuenta de Google.
2. Abre el notebook:
   - **Desde tu computadora:** `Archivo` > `Subir cuaderno` y selecciona el archivo `.ipynb`.
   - **Desde GitHub:** `Archivo` > `Abrir cuaderno` > pestaña `GitHub` y pega la URL del repositorio.
3. Sube los datos:
   - Abre el panel de archivos (ícono de carpeta a la izquierda) y arrastra los archivos de `users` y `usage`, o
   - Monta Google Drive con este código y usa la ruta donde guardaste los archivos:

   ```python
   from google.colab import drive
   drive.mount('/content/drive')
   ```

4. Ajusta las rutas de lectura en las primeras celdas a los nombres reales de tus archivos, por ejemplo:

   ```python
   users = pd.read_csv('users.csv')
   usage = pd.read_csv('usage.csv')
   ```

5. Ejecuta las celdas en orden con `Entorno de ejecución` > `Ejecutar todo`.

Colab ya incluye las librerías necesarias, por lo que no hace falta instalar nada.

## Guía de reproducción

**Requisitos** (si ejecutas el notebook fuera de Colab):

- Python 3.9 o superior
- `pandas`, `numpy`, `matplotlib`, `seaborn`

```bash
pip install pandas numpy matplotlib seaborn
```

**Pasos:**

1. Coloca los archivos de datos en la misma carpeta que el notebook (o ajusta las rutas de lectura).
2. Abre el notebook en Jupyter Notebook, JupyterLab o VS Code.
3. Ejecuta las celdas **de arriba hacia abajo**. El orden importa: la limpieza (etapas 2 y 3) debe correr antes de la agregación y la segmentación (etapas 5 y 9), porque esas columnas dependen de los datos ya corregidos.
4. Verifica los resultados clave:
   - `users` tiene 4,000 filas y `usage` tiene 40,000.
   - `user_profile` tiene una fila por usuario, con las columnas `grupo_uso` y `grupo_edad`.
   - Después de la limpieza, `age` ya no contiene -999 y `reg_date` no contiene años posteriores a 2024.

## Resultados principales

- **Calidad de datos:** había valores centinela ocultos en `age` y `city`, 40 fechas de registro imposibles (1%) y nulos estructurales en `duration` y `length`.
- **Uso:** el 74% de los usuarios es de uso medio, el 19% de bajo uso y el 7% de alto uso.
- **Edad:** la base se reparte de forma casi uniforme entre 18 y 79 años; el grupo Adulto es el más grande (50%).
- **Plan:** Básico y Premium muestran patrones de uso muy similares, por lo que el plan no explica el nivel de uso.
- **Outliers:** solo aparecen en las variables de uso y del lado derecho; se conservaron por ser valores plausibles.

## Estructura sugerida del repositorio

```
.
├── README.md
├── notebook.ipynb      # ajusta al nombre real de tu notebook
└── data/
    ├── users.csv       # ajusta a los nombres reales
    └── usage.csv
```
