[README MOLINETES 2024.md](https://github.com/user-attachments/files/35053959/README.MOLINETES.2024.md)

# Demanda del subte de CABA y cambios de tarifa, 2024

Trabajo práctico de preparación, análisis y visualización de datos sobre los molinetes del subte de la Ciudad de Buenos Aires durante 2024.

**Objetivos del análisis**
- Medir la demanda del subte en 2024 considerando los cambios de tarifa (febrero, mayo, junio, agosto y diciembre).
- Calcular el promedio de interacciones por día en hora pico.

---

## Organización del trabajo

El trabajo se divide en tres etapas:

| Etapa | Qué se hace | Herramienta | Estado |
|---|---|---|---|
| 1. Limpieza y curaduría | Carga y unión de los 24 archivos, control de calidad, estandarización de estaciones, conversión de fechas, variables derivadas y marcado de días anómalos. Produce `molinetes_2024.csv`, la base lista para analizar. | Python (Google Colab) | Finalizada |
| 2. Análisis | Demanda por franja horaria y hora pico, promedio de interacciones por día, comparación antes y después de cada cambio de tarifa. Parte de `molinetes_2024.csv`. | R | Pendiente |
| 3. Informe final | Integra la metodología, los hallazgos del análisis y las conclusiones. | A definir | Pendiente |

El CSV exportado en la etapa 1 es el punto de unión entre la limpieza y el análisis.

---

## Datos

- **Fuente:** [Subte - Viajes en molinetes](https://data.buenosaires.gob.ar/dataset/subte-viajes-molinetes), Portal de Datos Abiertos de la Ciudad de Buenos Aires.
- **Organismo:** Jefatura de Gabinete de Ministros, Secretaría de Transporte y Obras Públicas, Subterráneos de Buenos Aires (SBASE).
- **Contenido:** cantidad de pasajeros por molinete en cada estación, en intervalos de 15 minutos y discriminando por tipo de pasaje. Año 2024.
- **Última actualización de la fuente:** 13 de octubre de 2026. **Fecha de descarga:** 06/11/2026.
- **Licencia de los datos:** CC-BY-2.5-AR.
- **Tamaño:** 24 archivos CSV mensuales (grupos de líneas ABC y DEH), 11.440.440 filas y 10 columnas antes de la limpieza.

> Los datos no se incluyen en este repositorio por su tamaño. Para reproducir el trabajo, descargar `molinetes_2024.zip` desde la fuente y seguir el notebook.

### Variables originales

| Variable | Descripción |
|---|---|
| `fecha` | Día del registro |
| `desde`, `hasta` | Inicio y fin del intervalo de 15 minutos |
| `linea` | Línea de subte (A, B, C, D, E, H) |
| `molinete` | Código identificador del molinete |
| `estacion` | Estación |
| `pax_pagos` | Pasajeros con pasaje pago |
| `pax_pases_pagos` | Pasajeros con pase pago |
| `pax_franq` | Pasajeros con franquicia |
| `pax_total` | Total de pasajeros |

### Variables derivadas

| Variable | Descripción |
|---|---|
| `hora` | Hora de inicio del intervalo (0 a 23) |
| `mes` | Mes (1 a 12) |
| `dia_semana` | Día de la semana, **0 = lunes** (convención de Python; en R con `lubridate::wday(fecha, week_start = 1)` el lunes es 1) |
| `franja` | Madrugada (0 a 5), Pico mañana (6 a 9), Valle (10 a 15), Pico tarde (16 a 19), Noche (20 a 23) |
| `dia_anomalo` | 1 para el 31/10/2024 (ver más abajo), 0 para el resto |

---

## Proceso de limpieza

1. **Carga:** se descomprime el ZIP, se lee cada CSV (detectando codificación, BOM y separador), se normalizan los nombres de columnas y se unen los 24 archivos. Se controla que las filas del resultado sean la suma de las filas de cada archivo.
2. **Duplicados:** se eliminaron 3 filas duplicadas (3 pares idénticos en todas las columnas), el 0,00% de la base.
3. **Valores faltantes:** solo `molinete` tiene vacíos (158.033 registros, aprox. 1,4%). No se eliminaron, porque el resto de las variables está completo en esas filas y borrarlas implicaría perder demanda real.
4. **Estandarización de estaciones:**
   - Se quitaron espacios sobrantes (`Saenz Peña `).
   - Las estaciones de combinación con sufijo de línea con punto (`Callao.B`, `Pueyrredon.D`, `Retiro.C`) pasan a nombre con espacio (`Callao B`). Se mantienen separadas por línea.
   - `Retiro E` y `Retiro.E` se unifican; `Independencia.H`, registrada dentro de la línea E, se unifica con `Independencia E`.
   - Abreviaturas y mayúsculas: `Pza. de los Virreyes` pasa a `Plaza de los Virreyes`, `Leandro N. Alem` a `Leandro N Alem` y `9 de julio` a `9 de Julio`.
   - Control: ninguna estación aparece en más de una línea. Resultado: A 18, B 17, C 9, D 16, E 18 y H 12 estaciones.
5. **Fechas:** conversión a formato fecha con día primero (`dayfirst=True`). Se verificó que no se hayan cruzado día y mes.
6. **Variables derivadas:** `hora`, `mes`, `dia_semana`, `franja` y `dia_anomalo`. Controles: 12 meses, horas de 0 a 23, ninguna franja nula, ningún pasajero negativo y `pax_total` igual a la suma de los tres tipos de pasaje.

---

## Días con registros anómalos

- **9/05/2024 y 30/10/2024: sin registros.** Son paros totales de subte (paro general de la CGT y paro de la Mesa Nacional del Transporte). La base tiene **364 días** y no 366.
- **31/10/2024: registros anormalmente bajos.** Tiene 7.072 filas (el promedio diario es de unas 31.400) y 38.772 pasajeros. Hay datos en todas las horas y líneas, pero sin picos de mañana ni de tarde, y nueve estaciones no tienen ninguna fila ese día. **Causa no determinada.** La fecha queda marcada con `dia_anomalo = 1` y se excluye de los promedios por día y de hora pico.

---

## Cómo reproducirlo

1. Descargar `molinetes_2024.zip` desde la fuente.
2. **Etapa 1:** abrir el notebook en Google Colab y ejecutarlo completo (la lectura de los 24 archivos tarda unos 4 minutos). Genera `molinetes_2024.csv`.
3. **Etapa 2:** analisis en R Studio ( en proceso )

---

## Autor

Tomás Galvez Vera - [GitHub](https://github.com/tomasgalvezvera) - [LinkedIn](https://linkedin.com/in/tomasggalvezvera)
