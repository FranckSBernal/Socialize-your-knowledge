# Socialize-your-knowledge
Desempeño de los colaboradores de Socialize your knowledge
import streamlit as st
import pandas as pd
import plotly.express as px
from PIL import Image

# 1. Configuración de la página y título/descripción
st.set_page_config(
    page_title="Desempeño de Colaboradores",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Título
st.title("📊 Desempeño de los colaboradores de Socialize your knowledge")

# Breve descripción
st.markdown("""
La siguiente web tiene la finalidad de mostrar, de manera práctica y sencilla, los **KPI de desempeño**, 
lo que permitirá identificar tus fortalezas y áreas de oportunidad, y así lograr mejorar tu rendimiento y obtener mayor calidad en tus servicios.
""")

st.markdown("---")

# 2. Carga de datos y logotipo

# Cargar el archivo CSV
try:
    df = pd.read_csv("Employee_data.csv")
    # Limpieza básica de nombres de columnas y datos
    df.columns = df.columns.str.strip()
    df['gender'] = df['gender'].str.strip()
    df['marital_status'] = df['marital_status'].str.strip()
    df['performance_score'] = pd.to_numeric(df['performance_score'], errors='coerce', downcast='integer')
except FileNotFoundError:
    st.error("Error: El archivo 'Employee_data.csv' no fue encontrado. Asegúrate de que esté en el mismo directorio.")
    st.stop()

# Cargar y desplegar el logotipo
try:
    logo = Image.open("Logo Socialize your knowledge.PNG")
    # Usar un contenedor para el logo y alinearlo a la derecha (opcional)
    with st.container():
        col1, col2 = st.columns([4, 1])
        with col2:
            st.image(logo, width=150)
except FileNotFoundError:
    st.warning("Advertencia: La imagen 'Logo Socialize your knowledge.PNG' no fue encontrada. No se mostrará el logotipo.")

st.sidebar.header("Filtros de Análisis")

# 3. Controles para selección de filtros
# Control para seleccionar el género
genero_options = ['Todos'] + list(df['gender'].unique())
selected_gender = st.sidebar.selectbox(
    'Selecciona el **Género**:',
    genero_options
)

# Control para seleccionar el rango del puntaje de desempeño
min_score = int(df['performance_score'].min())
max_score = int(df['performance_score'].max())
selected_score_range = st.sidebar.slider(
    'Selecciona el **Puntaje de Desempeño** (1-5):',
    min_value=min_score,
    max_value=max_score,
    value=(min_score, max_score)
)

# Control para seleccionar el estado civil
estado_civil_options = ['Todos'] + list(df['marital_status'].unique())
selected_marital_status = st.sidebar.selectbox(
    'Selecciona el **Estado Civil**:',
    estado_civil_options
)

# 4. Aplicar filtros al DataFrame
df_filtered = df.copy()

if selected_gender != 'Todos':
    df_filtered = df_filtered[df_filtered['gender'] == selected_gender]

df_filtered = df_filtered[
    (df_filtered['performance_score'] >= selected_score_range[0]) & 
    (df_filtered['performance_score'] <= selected_score_range[1])
]

if selected_marital_status != 'Todos':
    df_filtered = df_filtered[df_filtered['marital_status'] == selected_marital_status]

# Mostrar un mensaje si no hay datos después de aplicar filtros
if df_filtered.empty:
    st.warning("No hay datos para mostrar con los filtros seleccionados.")
    st.stop()

st.header("Gráficos de Indicadores Clave")
st.markdown("---")

# 5. Despliegue de Gráficos

# --- Fila 1 de gráficos ---
col_perf_dist, col_avg_hours = st.columns(2)

with col_perf_dist:
    st.subheader("Distribución de Puntajes de Desempeño")
    # Gráfico 1: Distribución de los puntajes de desempeño (Histograma/Barras)
    fig_perf = px.histogram(
        df_filtered, 
        x='performance_score', 
        title='Frecuencia de Puntajes de Desempeño',
        color='gender',
        labels={'performance_score': 'Puntaje de Desempeño (1-5)'},
        nbins=len(df_filtered['performance_score'].unique())
    ).update_layout(xaxis={'categoryorder':'total descending', 'dtick': 1})
    st.plotly_chart(fig_perf, use_container_width=True)

with col_avg_hours:
    st.subheader("Promedio de Horas Trabajadas por Género")
    # Gráfico 2: Promedio de horas trabajadas por el género del empleado (Gráfico de barras)
    df_avg_hours = df_filtered.groupby('gender')['average_work_hours'].mean().reset_index()
    fig_hours = px.bar(
        df_avg_hours, 
        x='gender', 
        y='average_work_hours',
        title='Promedio de Horas Mensuales por Género',
        labels={'average_work_hours': 'Promedio de Horas', 'gender': 'Género'},
        color='gender'
    )
    st.plotly_chart(fig_hours, use_container_width=True)

st.markdown("---")

# --- Fila 2 de gráficos ---
col_age_salary, col_hours_perf = st.columns(2)

with col_age_salary:
    st.subheader("Edad vs. Salario")
    # Gráfico 3: Edad de los empleados con respecto al salario (Gráfico de dispersión)
    fig_salary = px.scatter(
        df_filtered, 
        x='age', 
        y='salary', 
        title='Relación entre Edad y Salario',
        color='gender',
        hover_data=['name_employee', 'position', 'performance_score'],
        labels={'age': 'Edad', 'salary': 'Salario'}
    )
    st.plotly_chart(fig_salary, use_container_width=True)

with col_hours_perf:
    st.subheader("Horas Trabajadas vs. Puntaje de Desempeño")
    # Gráfico 4: Relación del promedio de horas trabajadas versus el puntaje de desempeño (Gráfico de dispersión)
    # Agrupar por puntaje para mostrar el promedio de horas para ese grupo
    df_avg_hours_perf = df_filtered.groupby('performance_score')['average_work_hours'].mean().reset_index()
    
    fig_hours_perf = px.scatter(
        df_avg_hours_perf, 
        x='average_work_hours', 
        y='performance_score', 
        size='average_work_hours', # Tamaño del punto basado en las horas
        color='performance_score',
        title='Relación: Horas Trabajadas (Promedio) vs. Puntaje de Desempeño',
        labels={'average_work_hours': 'Promedio de Horas Trabajadas', 'performance_score': 'Puntaje de Desempeño (1-5)'}
    ).update_layout(yaxis={'dtick': 1})
    
    st.plotly_chart(fig_hours_perf, use_container_width=True)

st.markdown("---")

# 6. Conclusión
st.header("Conclusión del Análisis")
st.info("""
**El análisis de desempeño de los colaboradores es fundamental para la gestión del talento.**

Los gráficos presentados permiten una visualización rápida de tendencias clave:

* **Puntaje de Desempeño:** Identifica qué puntajes son los más comunes. Un alto volumen en los puntajes superiores sugiere un buen rendimiento general, mientras que picos en puntajes bajos pueden indicar áreas que necesitan capacitación o planes de mejora (**PIP**).
* **Horas/Género:** Muestra si existen diferencias significativas en el promedio de horas trabajadas entre géneros, lo cual podría ser un punto de partida para analizar la distribución de la carga laboral.
* **Edad/Salario:** La dispersión en este gráfico suele indicar que el salario está más relacionado con el **puesto** y el **puntaje de desempeño** que con la edad.
* **Horas/Desempeño:** Si el promedio de horas está positivamente correlacionado con el puntaje, sugiere que la dedicación se traduce en mejores resultados. Si no hay correlación o es negativa, podría indicar problemas de **eficiencia** o **burnout**.

*Para mejorar el rendimiento y la calidad del servicio, es crucial enfocarse en los colaboradores con puntajes bajos (1 y 2), revisar sus niveles de satisfacción y carga horaria, y aplicar planes de desarrollo personalizados.*
""")

# Comando para ejecutar la aplicación:
# Ejecuta el script con: streamlit run tu_archivo.py
