# Benchmarking CSV vs Parquet: Predicate Pushdown y optimización columnar

Laboratorio de Fundamentos de Big Data (IDSD413), Escuela Politécnica Nacional.
Autora: **[Tu nombre y apellido]**

Compara un formato basado en filas (CSV) con uno columnar (Apache Parquet) sobre un dataset sintético de 15 millones de transacciones de retail, ejecutado en Google Colab.

## Contenido

| Elemento | Descripción |
|---|---|
| `S6_Preparación_de_Datos_y_Benchmarking.ipynb` | Notebook de Colab con los retos resueltos y sus salidas |
| `capturas/` | Capturas de la ejecución (Figuras 1 a 8 del informe) |

## Cómo reproducirlo

1. Abre el notebook en Google Colab (entorno de CPU).
2. Ejecuta **Entorno de ejecución → Ejecutar todo**.
3. El notebook genera `transacciones_epn.csv` y `transacciones_epn.parquet` dentro de la sesión. No se incluyen en el repositorio por su tamaño (el CSV pesa 740 MB).

## Retos resueltos

- **Reto 1, prueba de estrés de memoria:** volumen elevado a 15 millones de registros y medición aislada de la RAM por enfoque con `psutil` (base, pico y final) y `memory_usage(deep=True)`.
- **Reto 2, consulta analítica avanzada:** monto de las transacciones de la categoría "Juguetes" con estado "Completado", con dos filtros a nivel de archivo (Predicate Pushdown) y proyección de la columna `monto`.

## Resultados (15 millones de registros, corrida 3)

| Prueba | CSV | Parquet | Ventaja de Parquet |
|---|---:|---:|---:|
| Escritura (s) | 39.71 | 4.62 | 8.6x |
| Peso en disco (MB) | 740.53 | 225.39 | 69.56 % menos |
| Lectura completa (s) | 5.26 | 1.54 | 3.4x |
| Suma de `monto`, 1 columna (s) | 3.6784 | 0.1283 | 28.66x |
| Filtro "Electronica" (s) | 7.0473 | 1.2098 | 5.83x |
| Agrupación por `estado` (s) | 5.9997 | 1.3675 | 4.39x |
| Reto 2: Juguetes + Completado (s) | 6.8439 | 0.8145 | 8.40x |

### Memoria RAM al leer el dataset completo (MB)

| Enfoque | Tiempo (s) | Base | Pico | Final | DataFrame (`deep`) |
|---|---:|---:|---:|---:|---:|
| CSV completo | 5.87 | 2000 | 3018 | 2618 | 1991 |
| Parquet completo | 1.42 | 2046 | 2603 | 2603 | 1991 |

No hubo saturación de memoria: el pico máximo (3018 MB) es cerca del 23 % de los 12.7 GB del entorno. Una vez cargado en pandas, el DataFrame ocupa lo mismo con ambos formatos. La ventaja de Parquet en RAM está en el menor pico de lectura y en las lecturas selectivas.

### Reto 2: uso de columnas y memoria

| Métrica | CSV | Parquet |
|---|---:|---:|
| Filas devueltas | 750,894 | 750,894 |
| Columnas cargadas | 5 | 1 |
| RAM del resultado (MB) | 1991 | 5.7 |
| Suma de `monto` | 939,863,458.14 | 939,863,458.14 |

Ambos métodos devuelven las mismas filas y la misma suma.

## Capturas

![Figura 1: generación](capturas/fig1_generacion.png)
![Figura 2: escritura y pesos](capturas/fig2_escritura.png)
![Figura 3: lectura completa](capturas/fig3_lectura.png)
![Figura 4: RAM por enfoque](capturas/fig4_ram.png)
![Figura 5: column pruning](capturas/fig5_column_pruning.png)
![Figura 6: filtro Electronica](capturas/fig6_filtro_electronica.png)
![Figura 7: Reto 2](capturas/fig7_reto2.png)
![Figura 8: agrupación por estado](capturas/fig8_groupby.png)

## Nota sobre la variabilidad

Colab es una máquina virtual compartida, por lo que los tiempos cambian entre sesiones. En tres corridas con 15 millones de registros, la escritura del CSV tardó 74.79 s, 75.27 s y 39.71 s. Las razones entre formatos fueron más estables que los tiempos absolutos, pero deben leerse como órdenes de magnitud.

## Entorno

Google Colab (CPU, 12.7 GB de RAM), Python 3, pandas **[versión]** y PyArrow **[versión]**.
