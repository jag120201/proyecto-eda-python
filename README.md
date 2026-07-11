# 📊 Análisis Exploratorio de Datos — Campaña de Marketing Bancario

**Autor:** José Antonio Gálvez Beneroso

Proyecto de **Análisis Exploratorio de Datos (EDA)** realizado íntegramente en **Python**, sobre datos reales de campañas de marketing directo (telefónico) de una institución bancaria portuguesa, cuyo objetivo era la contratación de un depósito a plazo fijo.

---

## 🎯 Objetivo del proyecto

Realizar un análisis exploratorio completo de los datos aportados, cubriendo:

- Transformación y limpieza de los datos.
- Análisis descriptivo de los datos.
- Visualización de los datos.
- Informe explicativo del análisis.

## 🛠️ Herramientas utilizadas

- **Python**
- **Pandas** (manipulación y limpieza de datos)
- **NumPy** (operaciones numéricas)
- **Matplotlib / Seaborn** (visualización)
- **Visual Studio Code** como entorno de desarrollo

> No se ha utilizado SQL: todo el procesamiento se realiza con Python y Pandas.

---

## 🗂️ Estructura del repositorio

```
├── data/
│   ├── raw/                    # Datos originales, sin modificar
│   │   ├── bank-additional.csv
│   │   └── customer-details.xlsx
│   └── processed/              # Datos ya limpios y transformados
│       ├── bank_additional_limpio.csv
│       └── dataset_completo.csv
├── notebooks/
│   └── EDA_marketing_bancario.ipynb   # Notebook con todo el análisis
├── img/                        # Gráficos exportados del análisis
└── README.md
```

## 📁 Datos utilizados

### `bank-additional.csv`
Un registro por cada contacto telefónico de la campaña. Incluye datos del cliente (edad, profesión, estado civil, educación...), datos de la llamada (duración, número de contactos, resultado de la campaña anterior...) e indicadores macroeconómicos del momento del contacto (euríbor, índice de confianza del consumidor, etc.), además de la variable objetivo `y` (si el cliente suscribió o no el depósito).

### `customer-details.xlsx`
Información demográfica y de comportamiento del cliente (ingresos, hijos/adolescentes en el hogar, visitas mensuales a la web, fecha de alta como cliente), repartida en **3 hojas** según el año en que el cliente entró en el banco (2012, 2013, 2014).

Ambos ficheros se cruzan a través del identificador único del cliente (`id_` / `ID`), que coincide al 100% entre ambas fuentes.

---

## 🧹 1. Transformación y limpieza de los datos

Principales problemas detectados y solución aplicada:

| Problema detectado | Solución aplicada |
|---|---|
| Columna `Unnamed: 0` residual en ambos ficheros | Eliminada |
| Columnas numéricas (`euribor3m`, `cons.price.idx`...) con coma como separador decimal, leídas como texto | Convertidas a `float` reemplazando `,` por `.` |
| `marital` y `poutcome` con formato de texto inconsistente (mayúsculas) | Normalizadas a minúsculas |
| `default`, `housing`, `loan` codificadas como 0/1 | Traducidas a categorías legibles (`si` / `no`) |
| `date` en texto con el mes en español (`"2-agosto-2019"`) | Parseada a `datetime`, generando además `contact_month` y `contact_year` |
| Valores nulos en `age` | Imputados con la mediana |
| Valores nulos en variables categóricas (`job`, `marital`, `education`, etc.) | Imputados como `"desconocido"` |
| Valores nulos en `cons.price.idx` y `euribor3m` | Imputados con la mediana |
| `pdays = 999` (valor centinela de "nunca contactado") | Traducido a una variable auxiliar `contactado_antes` (sí/no) |
| Filas duplicadas | Revisadas y eliminadas |
| Datasets de campaña y de cliente en ficheros separados | Cruzados mediante `id_` / `ID` en un único `dataset_completo.csv` |

Todo el proceso está documentado paso a paso en el notebook, con explicación de **por qué** se tomó cada decisión de limpieza.

## 📈 2. Análisis descriptivo

Se han calculado, entre otros:

- Estadísticos generales (media, mediana, desviación estándar) de las variables numéricas más relevantes (edad, duración de llamada, ingresos, visitas web...).
- Tasa de suscripción global de la campaña, y por profesión, estado civil, nivel educativo, tipo de contacto y resultado de campañas anteriores.
- Matriz de correlación entre variables numéricas, incluyendo los indicadores macroeconómicos.
- Estadísticos agrupados (por ejemplo, ingreso medio según número de hijos/adolescentes en el hogar).

## 🖼️ 3. Visualización de los datos

El notebook incluye gráficos como:

- Distribución de la edad de los clientes.
- Tasa de suscripción por profesión.
- Duración de la llamada según el resultado de la suscripción (boxplot).
- Mapa de calor de correlaciones entre variables numéricas.
- Evolución del número de contactos por año y mes.
- Distribución del ingreso según si el cliente tiene hijos en casa.
- Relación entre el resultado de la campaña anterior y la suscripción actual.

Todos los gráficos se guardan automáticamente en la carpeta `img/`.

---

## 📝 4. Informe explicativo y conclusiones

**Calidad de los datos.** El dataset original tenía varios problemas típicos de un caso real (separadores decimales europeos, fechas en español, columnas binarias como 0/1, nulos en variables demográficas). Tras la limpieza, queda un dataset coherente y listo para análisis o modelado.

**Perfil de los clientes.** La mayoría de los clientes contactados tiene entre 30 y 50 años, está casada y trabaja como administrativo, obrero cualificado o técnico, con estudios universitarios o de secundaria como niveles más frecuentes.

**Resultados de la campaña.**
- La tasa de suscripción global es baja: el dataset está claramente desbalanceado, algo habitual en campañas de telemarketing.
- La **duración de la llamada** está asociada con la suscripción: llamadas más largas tienden a terminar en "sí". En un caso real esta variable no estaría disponible antes de llamar, por lo que no serviría para decidir a quién contactar, aunque sí es útil para entender qué pasó durante la campaña.
- Los clientes con un resultado **positivo en la campaña anterior** (`poutcome = success`) tienen una probabilidad de suscripción claramente más alta: son el público prioritario para futuras campañas.
- Hay diferencias relevantes por **profesión**: perfiles como estudiantes o jubilados muestran tasas de conversión superiores a la media.
- Las variables macroeconómicas (`euribor3m`, `emp.var.rate`, `nr.employed`) están muy correlacionadas entre sí, por reflejar el mismo ciclo económico. Es importante tenerlo en cuenta para evitar problemas de multicolinealidad en un futuro modelo predictivo.

**Cruce con datos demográficos.** Se pudo enlazar el 100% de los contactos con su información demográfica. Se observan diferencias de ingresos según si el cliente tiene o no hijos/adolescentes en casa, lo que abre la puerta a un análisis de segmentación más detallado.

**Posibles próximos pasos:**
- Construir un modelo predictivo (regresión logística, árbol de decisión...) que estime la probabilidad de suscripción **sin usar `duration`**, ya que no se conoce antes de realizar la llamada.
- Segmentar clientes combinando variables demográficas y de comportamiento (clustering).
- Explorar el componente geográfico (`latitude`/`longitude`), no contemplado en el enunciado original del proyecto.

---

## ▶️ Cómo ejecutar el proyecto

1. Clonar este repositorio.
2. Instalar las dependencias:
   ```bash
   pip install pandas numpy matplotlib seaborn openpyxl jupyter
   ```
3. Abrir el notebook `notebooks/EDA_marketing_bancario.ipynb` en Visual Studio Code o Jupyter y ejecutarlo de principio a fin.
