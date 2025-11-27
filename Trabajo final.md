# 📘 Proyecto Final – Modelo Dimensional & Dashboard de Bienestar Laboral

Este proyecto implementa un modelo analítico tipo Estrella a partir de un dataset público/CSV de bienestar laboral (STATUSWELLBEING.csv).
Incluye la creación de tablas Dimensionales y una tabla de Hechos en Power BI, el desarrollo de métricas DAX, y un Dashboard interactivo publicado en Power BI Service.

## 🚀 Objetivos del Proyecto

Transformar un CSV Raw en un modelo analítico estructurado.

Construir las tablas Dim_Persona, Dim_Tiempo y Fact_Respuestas.

Crear relaciones según un modelo Estrella clásico.

Generar métricas DAX específicas para análisis de bienestar.

Diseñar un dashboard profesional en Power BI.

Documentar el proceso completo (este README).

### 📂 Estructura del Repositorio
/data
   STATUSWELLBEING.csv

/powerbi
   Dashboard_Wellbeing.pbix

/docs
   Modelo_Estrella.png
   Diccionario_Datos.pdf

README.md

### 🧱 Modelo Dimensional (Esquema Estrella)

El modelo está compuesto por:

Fact_Respuestas – tabla de hechos

Dim_Persona – dimensión legible del colaborador

Dim_Tiempo – calendario propio para análisis temporal

#### 📐 Diagrama del Modelo Estrella
                   ┌──────────────────────┐
                   │     Dim_Persona      │
                   ├──────────────────────┤
                   │ PersonaID (PK)       │
                   │ Edad                 │
                   │ Genero               │
                   │ Area                 │
                   │ Puesto               │
                   │ Seniority            │
                   └───────────┬──────────┘
                               │ 1
                               │
                               │ *
                      ┌────────▼─────────┐
                      │  Fact_Respuestas │
                      ├──────────────────┤
                      │ ResponseID (PK)  │
                      │ PersonaID (FK)   │
                      │ Fecha (FK)       │
                      │ StressLevel      │
                      │ WorkLifeBalance  │
                      │ MoodToday        │
                      │ SleepQuality     │
                      │ Productivity     │
                      │ ... otras        │
                      └─────────┬────────┘
                                │ *
                                │
                                │ 1
                   ┌────────────▼────────────┐
                   │       Dim_Tiempo        │
                   ├──────────────────────────┤
                   │ Fecha (PK)               │
                   │ Año                      │
                   │ Mes                      │
                   │ MesNumero                │
                   │ DiaMes                   │
                   │ DiaSemana                │
                   │ SemanaISO                │
                   └──────────────────────────┘

### 🛠️ Desarrollo Paso a Paso
#### 1. Ingesta del dataset (Bronze)

Se carga STATUSWELLBEING.csv en Power BI.

Se selecciona el separador correcto y se cambian tipos de datos.

No se realizan transformaciones profundas en esta capa.

#### 2. Construcción de Dimensiones (Silver)
Dim_Persona

Se duplica la tabla original.

Se conservan:
PersonaID, Edad, Genero, Area, Puesto, Seniority.

Se eliminan duplicados.

Resultado: una fila por colaborador.

Dim_Tiempo

Se crea un calendario dinámico en Power Query:

let
    FechaMin = List.Min(Fact_Respuestas[Fecha]),
    FechaMax = List.Max(Fact_Respuestas[Fecha]),
    Calendar = List.Dates(FechaMin, Duration.Days(FechaMax - FechaMin) + 1, #duration(1,0,0,0)),
    #"ToTable" = Table.FromList(Calendar, Splitter.SplitByNothing(), {"Fecha"}),
    #"AddYear" = Table.AddColumn(#"ToTable", "Año", each Date.Year([Fecha])),
    #"AddMonth" = Table.AddColumn(#"AddYear", "Mes", each Date.MonthName([Fecha])),
    #"AddSemana" = Table.AddColumn(#"AddMonth", "SemanaISO", each Date.WeekOfYear([Fecha]))
in
    #"AddSemana"

Fact_Respuestas

Se renombra la tabla original.

Se agrega clave surrogate:

Add Column → Index Column → From 1 → ResponseID

#### 3. Modelado de Relaciones

En Model View:

From	To	Tipo
Dim_Persona[PersonaID]	Fact_Respuestas[PersonaID]	1:*
Dim_Tiempo[Fecha]	Fact_Respuestas[Fecha]	1:*
🔢 Medidas DAX del Proyecto
Cantidad de respuestas
Cantidad Respuestas =
COUNT(Fact_Respuestas[ResponseID])

Promedio de Estrés
Promedio Estrés =
AVERAGE(Fact_Respuestas[StressLevel])

Promedio Work-Life Balance
Promedio WLB =
AVERAGE(Fact_Respuestas[WorkLifeBalance])

Promedio de Calidad de Sueño
Promedio Sueño =
AVERAGE(Fact_Respuestas[SleepQuality])

Promedio de Productividad
Promedio Productividad =
AVERAGE(Fact_Respuestas[Productivity])

Índice de Bienestar (Wellbeing Index)
Wellbeing Index =
AVERAGEX(
    Fact_Respuestas,
    (Fact_Respuestas[WorkLifeBalance] +
     Fact_Respuestas[SleepQuality] +
     Fact_Respuestas[Productivity] -
     Fact_Respuestas[StressLevel])
)

% Estrés Alto (≥4)
% Estrés Alto =
VAR Total = COUNTROWS(Fact_Respuestas)
VAR Altos =
    CALCULATE(
        COUNTROWS(Fact_Respuestas),
        Fact_Respuestas[StressLevel] >= 4
    )
RETURN DIVIDE(Altos, Total)

Variación Mes a Mes del Estrés
Variación Estrés MoM =
VAR Actual = [Promedio Estrés]
VAR Anterior =
    CALCULATE(
        [Promedio Estrés],
        DATEADD(Dim_Tiempo[Fecha], -1, MONTH)
    )
RETURN
Actual - Anterior

### 📊 Visualizaciones recomendadas
KPI Cards

Wellbeing Index

Promedio Estrés

Promedio Productividad

Cantidad total de respuestas

Line Chart

Estrés por mes

Productividad por mes

Barras

Estrés por Área

Productividad por Puesto

Scatter Plot

SleepQuality vs Productivity

Pie Chart

MoodToday (si aplica)

### 🚀 Publicación & Resultados

El dashboard final se publica en Power BI Service.

Las métricas permiten identificar áreas críticas, tendencias de estrés, productividad y bienestar.

El modelo es escalable y permite incorporar nuevas encuestas sin modificar la estructura.

👤 Autor

Mariano Ezequiel Barcia
Especialista Senior de Recursos Humanos – Desarrollo del Talento

