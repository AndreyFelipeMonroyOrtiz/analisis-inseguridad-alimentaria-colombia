![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-ETL_Pipeline-150458?style=flat&logo=pandas&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-Dashboard-F2C811?style=flat&logo=powerbi&logoColor=black)
![DANE Data](https://img.shields.io/badge/Fuente-DANE_ECV-008080?style=flat)
# Análisis de Inseguridad Alimentaria en Colombia (2022–2025)

Análisis de la evolución y distribución geográfica de la inseguridad alimentaria en los hogares colombianos, usando microdatos oficiales del DANE. El proyecto conecta con mi experiencia previa trabajando en el Banco de Alimentos de Bogotá, donde vi de primera mano la operación de un programa de seguridad alimentaria — este proyecto extiende esa mirada a una escala nacional usando datos públicos.

## Pregunta que responde

¿Cómo ha evolucionado la inseguridad alimentaria grave en Colombia entre 2022 y 2025, y qué departamentos concentran los niveles más altos?

## Fuente de datos

- **Encuesta Nacional de Calidad de Vida (ECV) – DANE**, microdatos anuales 2022–2025 ([dane.gov.co](https://www.dane.gov.co)).
- **Cobertura**: 345.895 encuestas acumuladas en el periodo (86.848 hogares en el corte de 2025).
- **Módulos integrados**: Vivienda (ubicación geográfica), Composición del Hogar (`P6020`, `P6040` — jefatura, sexo, edad), Educación (`P6210`/`P8587`), Seguridad Alimentaria (`P3516S1`–`P3516S8`, escala FIES).

## Metodología

**Limpieza y consolidación (ETL)**
- Normalización de códigos de departamento con `.str.zfill(2)`, corrigiendo años donde el código venía sin cero a la izquierda (ej. `'5'` en vez de `'05'` para Antioquia), e incluyendo Santander (`'68'`), ausente del diccionario DIVIPOLA original.
- Homologación del separador decimal del factor de expansión (`FEX_C`) y configuración de la importación en Power BI bajo la región *Inglés (Estados Unidos)*, para evitar que el decimal se interpretara como separador de miles.
- Eliminación de duplicados por llave compuesta (`DIRECTORIO` + `ANIO`).

**Escala FIES**
La Escala de Experiencia de Inseguridad Alimentaria (FIES, FAO) mide 8 experiencias del hogar en el último año — desde preocupación por quedarse sin alimentos hasta pasar un día entero sin comer. Cada ítem es binario (1 = Sí, 0 = No) y se suman en un puntaje bruto (`PUNTAJE_FIES`, escala 0–8).

**Calibración del umbral de severidad**
El DANE clasifica la severidad con un modelo estadístico Rasch/IRT que no se replica en este proyecto. Como aproximación transparente, se calibra el punto de corte de `PUNTAJE_FIES` que minimiza el error cuadrático frente a las tasas nacionales oficiales del DANE (2022–2025):

$$\text{Error} = \sum_{t} \left(\text{Tasa calculada}_t(\text{umbral}) - \text{Tasa oficial DANE}_t\right)^2$$

Esto define las variables `PIAMG_DUMMY` (moderada o grave) y `PIAG_DUMMY` (grave). Es un proxy documentado, no el cálculo oficial exacto del DANE.

**Ponderación estadística**
Todas las tasas usan el factor de expansión de la encuesta (`FEX_C`), no conteos simples de filas — indispensable al trabajar con datos de encuesta, no de censo. Medida DAX usada en las tarjetas, rankings y el mapa de Power BI:

```dax
Tasa_Inseguridad_Grave_% = 
DIVIDE(
    SUMX('ECV_Hambre_Colombia_2022_2025', 'ECV_Hambre_Colombia_2022_2025'[PIAG_DUMMY] * 'ECV_Hambre_Colombia_2022_2025'[FEX_C]),
    SUM('ECV_Hambre_Colombia_2022_2025'[FEX_C]),
    0
) * 100
```

## Retos técnicos resueltos

- **Reproducibilidad**: el pipeline original dependía de variables creadas en celdas ejecutadas fuera de orden en Google Colab, lo que impedía correrlo de principio a fin en un entorno nuevo. Se reestructuró en una sola pasada lineal.
- **Validación cruzada**: comparar los resultados de Python contra el dashboard de Power BI — en vez de confiar en uno solo — fue lo que permitió detectar el problema del factor de expansión antes de publicar el proyecto. Un departamento con muestra pequeña (Vichada) mostraba tasas muy distintas entre ambas herramientas; rastrear la diferencia llevó al hallazgo del decimal mal interpretado.

## Herramientas

- **Python (Pandas)**: limpieza, consolidación multi-año, cálculo de indicadores.
- **Power BI**: dashboard interactivo de 2 páginas (panorama nacional + mapa por departamento).
- **SQL**: consultas de exploración sobre los datos consolidados.

## Hallazgos principales

- Los departamentos con mayor inseguridad alimentaria grave en 2025 son **La Guajira (13,8%)**, **Chocó (9,8%)** y **Vichada (7,7%)**, muy por encima del resto del país.
- La inseguridad alimentaria grave muestra una tendencia decreciente a nivel nacional entre 2022 y 2025, consistente con la publicada oficialmente por el DANE.
- Existe una relación visible entre nivel educativo del jefe de hogar y la tasa de inseguridad alimentaria.

## Dashboard

<img width="1279" height="720" alt="image" src="https://github.com/user-attachments/assets/f4be16db-c6ee-4701-bf3a-9a2436536591" />
<img width="1280" height="721" alt="image" src="https://github.com/user-attachments/assets/4b4f723b-4cde-4fc6-a7de-ec9d86fa890b" />



## Estructura del repositorio

```
├── proyecto_hambre_colombia.py             # Pipeline de limpieza y cálculo de indicadores
├── ECV_Hambre_Colombia_2022_2025.csv       # Dataset consolidado, listo para Power BI
├── Proyecto_hambre_colombia.pbix           # Dashboard interactivo de Power BI
└── README.md
```

## Autor

Andrey Felipe Monroy Ortiz — Economista | Analista de Datos | [LinkedIn](https://www.linkedin.com/in/andrey-felipe-monroy-ortiz-806417348)
