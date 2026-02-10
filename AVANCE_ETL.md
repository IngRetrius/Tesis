# Avance del Proceso ETL

## Proyecto

**Titulo**: "Diseno e Implementacion de un Sistema Web de Analisis Financiero con Modelo de Machine Learning para la Evaluacion de Riesgo en PYMES del Municipio de Ibague, Tolima"

---

## Fuentes de datos

El proyecto utiliza dos fuentes de datos principales:

1. **SIREM** (Superintendencia de Sociedades) - 4 archivos CSV (~9.17 GB en total) con estados financieros bajo NIIF.
2. **Camara de Comercio de Ibague** - 1 archivo CSV (23.58 MB) con las empresas activas en la jurisdiccion de Ibague.

La estrategia central consiste en realizar un JOIN entre ambas fuentes usando el NIT como llave, para identificar las PYMES de Ibague que cuentan con informacion financiera reportada ante Supersociedades.

---

## Trabajo realizado

### Fase 1: Exploracion y ETL inicial (etl_camara_comercio.ipynb)

#### 1.1. Analisis exploratorio de los datasets SIREM

- Se cargaron muestras de los 4 datasets del SIREM.
- Se verifico la estructura: los 4 comparten 11 columnas (`CODIGO_INSTANCIA`, `NIT`, `NUMERO_RADICADO`, `ID_PUNTO_ENTRADA`, `PUNTO_ENTRADA`, `ID_TAXONOMIA`, `TAXONOMIA`, `FECHA_CORTE`, `CONCEPTO`, `PERIODO`, `VALOR`).
- Los datos estan en formato vertical/largo (una fila por cada par concepto-valor).
- La taxonomia NIIF Pymes representa aproximadamente el **80-85%** de los registros.
- Cobertura temporal: **2015-2024**.
- Los valores son numericos en todos los datasets, excepto en Caratula (mixto).

#### 1.2. ETL del dataset de Camara de Comercio

- Se cargo el dataset completo: **20,280 registros** y **85 columnas**.
- Analisis de calidad de datos:
  - 84 de 85 columnas presentan valores NaN, pero solo entre 5 y 9 registros cada una (0.02%, despreciable).
  - Uso extensivo de "No aplica" y "No reporta" como nulos funcionales (hasta el 100% en algunas columnas).
  - Columna NIT: **9,146 registros (45.1%)** contienen "No aplica". Estos corresponden principalmente a **ESTABLECIMIENTOS DE COMERCIO** (9,000 sucursales comerciales) que no poseen NIT propio.
  - **11,129 registros** tienen NITs validos (**11,126 unicos**).
- Analisis del formato de NIT:
  - Los NITs en Camara de Comercio son digitos limpios (sin separadores).
  - Distribucion por longitud: 7 digitos (3), 8 digitos (332), 9 digitos (3,154), 10 digitos (3,823), 11 digitos (3,817).

#### 1.3. Carga completa de los datasets SIREM

Se cargaron los 4 datasets del SIREM en su totalidad (sin muestreo):

| Dataset | Registros |
|---|---|
| Caratula | 8,685,453 |
| Estado de Situacion Financiera | 17,851,220 |
| Estado de Resultado Integral | 7,320,752 |
| Estado de Flujo de Efectivo | 5,769,293 |
| **Total** | **39,626,718** |

- El SIREM contiene aproximadamente **43,041 NITs unicos** (todos de 9 digitos, con comas como separador de miles).
- Se realizo la limpieza de NITs eliminando las comas.

#### 1.4. Descubrimiento critico: incompatibilidad de formato de NIT

- El JOIN directo entre ambas fuentes produjo **0 coincidencias**.
- Se identifico la causa raiz: el **digito de verificacion (DV)**.
  - SIREM almacena el NIT base de 9 digitos (ejemplo: `830010665`).
  - Camara de Comercio almacena el NIT de 10 digitos con el DV concatenado al final (ejemplo: `8300106651`).
- **Solucion implementada**: eliminar el ultimo digito (DV) de los NITs de 10 digitos para obtener el NIT base de 9 digitos.

#### 1.5. Resultados de la validacion del JOIN

Despues de la normalizacion del DV, se encontraron **66 empresas de Ibague presentes en el SIREM**:

- Las 66 pertenecen al municipio **73001 - IBAGUE** (verificado).
- Composicion por tipo de sociedad: SAS (38), Ltda (12), SA (12), Sociedad en Comandita (2), Otras (2).
- **61 de las 66** reportan bajo taxonomia **NIIF Pymes** (relevante para la tesis).

**Nota sobre el tamano de la muestra**: de las 1,733 sociedades comerciales identificadas en Camara de Comercio, solo 66 aparecen en el SIREM. Esto se debe a que el SIREM unicamente contiene empresas vigiladas por la Superintendencia de Sociedades. La mayoria de las PYMES no estan obligadas a reportar. Esta es una limitacion conocida y documentada en la propuesta de tesis.

---

### Fase 2: JOIN definitivo y pivoteo (procesamiento_datos_financieros.ipynb)

#### 2.1. Extraccion de datos financieros

Se procesaron los 3 datasets financieros del SIREM mediante lectura por chunks (500,000 filas por bloque) con doble filtro simultaneo: NIT de Ibague + taxonomia NIIF Pymes.

| Dataset | Filas leidas | Filas extraidas | Tasa | Empresas |
|---|---|---|---|---|
| Estado de Situacion Financiera | 17,851,220 | 27,567 | 0.15% | 61 |
| Estado de Resultado Integral | 7,320,752 | 10,720 | 0.15% | 61 |
| Estado de Flujo de Efectivo | 5,769,293 | 8,510 | 0.15% | 61 |
| **Total** | **30,941,265** | **46,797** | **0.15%** | **61** |

Las 61 empresas extraidas corresponden a las que reportan bajo NIIF Pymes. Se excluyo el dataset de Caratula por contener metadatos (no datos financieros numericos); la informacion de las empresas se obtiene de la Camara de Comercio.

#### 2.2. Estructura temporal descubierta

El analisis revelo una estructura temporal compleja en los datasets SIREM:

**Campo TAXONOMIA** (6 valores unicos por dataset):

| Valor de TAXONOMIA | Filas (Sit. Financiera) |
|---|---|
| `01- Estados Financieros de Fin de Ejercicio - Corte 2016` | 4,542 |
| `01- Estados Financieros de Fin de Ejercicio - Corte 2017` | 2,239 |
| `01- Estados Financieros de Fin de Ejercicio - Corte 2018` | 2,094 |
| `01- Estados Financieros de Fin de Ejercicio - Corte 2019` | 2,440 |
| `01- Estados Financieros de Fin de Ejercicio - Corte 2020` | 2,657 |
| `01 - Estados Financieros de Fin de Ejercicio - Corte 202` | 13,595 |

> **Hallazgo critico**: La taxonomia mas reciente ("Corte 202") agrupa los datos de **2021 a 2024** bajo un mismo valor truncado. Esto dificulta la extraccion del anio fiscal directamente desde TAXONOMIA. La columna `FECHA_CORTE` si contiene el anio completo (2021, 2022, 2023, 2024) y deberia usarse como fuente alternativa para el anio fiscal.

**Campo PERIODO** (comportamiento variable por dataset):

| Dataset | Valores de PERIODO | Observacion |
|---|---|---|
| Situacion Financiera | "Periodo Actual", "Periodo Anterior", "2016-dic-31", "2015-dic-31", "2015-ene-01", "2017-dic-31" | Formato mixto: etiquetas para 2018+ y fechas para 2016-2017 |
| Resultado Integral | Similar a Situacion Financiera | Mismo comportamiento mixto |
| Flujo de Efectivo | "Periodo Actual", "2017", y **valores de NIT como periodo** (800077198, 809004045, etc.) | **Anomalia de calidad de datos** |

> **Hallazgo critico**: El campo PERIODO del Flujo de Efectivo contiene **NITs de empresas** como valores de periodo (46 valores unicos, la mayoria son NITs). Esto parece ser un error en el dataset fuente del SIREM.

#### 2.3. Filtrado por periodo actual

Para evitar duplicacion de datos (el "Periodo Anterior" de un anio fiscal es el mismo dato que el "Periodo Actual" del anio previo), se filtro conservando solo registros del periodo actual:

| Dataset | Antes | Despues | Removidas |
|---|---|---|---|
| Situacion Financiera | 27,567 | **10,378** | 17,189 (periodo anterior + fechas 2016-2017) |
| Resultado Integral | 10,720 | **4,370** | 6,350 (periodo anterior + fechas 2016-2017) |
| Flujo de Efectivo | 8,510 | **8,510** (sin filtro) | 0 (no se detecto patron estandar) |

> **Consecuencia**: Los datos de Situacion Financiera y Resultado Integral para los anios **2016 y 2017** se perdieron porque su campo PERIODO usa formato de fecha ("2016-dic-31") en lugar de la etiqueta "Periodo Actual". Estos anios requieren un tratamiento especial.

#### 2.4. Cobertura temporal resultante

Despues del filtrado, la cobertura temporal (usando Estado de Situacion Financiera como referencia) es:

| Anio | Empresas con datos |
|---|---|
| 2018 | 30 |
| 2019 | 36 |
| 2020 | 41 |

**Estadisticas de profundidad temporal por empresa**:
- Minimo: 0 anios (4 empresas sin datos en rango 2018-2020)
- Maximo: 3 anios
- Promedio: 2.2 anios
- Mediana: 3 anios

**Distribucion**:

| Anios de datos | Numero de empresas |
|---|---|
| 0 anios | 4 |
| 1 anio | 10 |
| 2 anios | 8 |
| 3 anios | 27 |

> **Total combinaciones empresa-anio**: 107 (para Balance y Resultados), 180 (para Flujo de Efectivo que conservo 2016-2020).

#### 2.5. Conceptos financieros identificados

| Estado Financiero | Conceptos unicos |
|---|---|
| Situacion Financiera (Balance) | **66 conceptos** |
| Resultado Integral | **19 conceptos** |
| Flujo de Efectivo | **101 conceptos** |
| **Total** | **186 conceptos** |

Los conceptos clave para el calculo de indicadores estan presentes:

**Balance**: Activos corrientes totales, Pasivos corrientes totales, Total pasivos, Patrimonio total, Inventarios corrientes, Efectivo y equivalentes al efectivo, Cuentas comerciales por cobrar corrientes, Propiedades planta y equipo, Total de activos.

**Resultados**: Ingresos de actividades ordinarias, Costo de ventas, Ganancia bruta, Gastos de ventas, Gastos de administracion, Ganancia (perdida) por actividades de operacion, Ganancia (perdida), Costos financieros, Ingresos financieros.

> **Nota sobre codificacion**: Los nombres de CONCEPTO presentan artefactos de codificacion en caracteres acentuados (ejemplo: "Ganancia (p\u00e9rdida)" aparece como "Ganancia (p�rdida)"). Los datos son funcionales pero requieren limpieza de encoding para presentacion.

#### 2.6. Verificacion de duplicados

Se encontraron combinaciones NIT + ANIO + CONCEPTO con registros multiples:

| Dataset | Duplicados | Total combinaciones | Porcentaje |
|---|---|---|---|
| Situacion Financiera | 128 | 3,463 | 3.7% |
| Resultado Integral | 47 | 1,402 | 3.4% |
| Flujo de Efectivo | 271 | 3,777 | 7.2% |

Los duplicados se concentran en la empresa NIT `800135342` en el anio 2018 y en el Flujo de Efectivo en general. Se resolvieron usando `aggfunc='first'` en el pivoteo (se conserva el primer valor encontrado).

#### 2.7. Resultados del pivoteo

Se transformaron los datos del formato vertical (concepto-valor) al formato horizontal (una columna por concepto):

| Dataset | Filas (empresa-anio) | Columnas | Empresas | Anios | Completitud |
|---|---|---|---|---|---|
| Situacion Financiera | 107 | 67 | 45 | 2018-2020 | **49.8%** |
| Resultado Integral | 107 | 20 | 45 | 2018-2020 | **72.8%** |
| Flujo de Efectivo | 180 | 103 | 57 | 2016-2020 | **20.8%** |

> **Interpretacion de la completitud**: Un 49.8% en Situacion Financiera significa que la mitad de las celdas posibles (empresa x anio x concepto) tienen valor. Esto es esperable porque muchos conceptos contables no aplican a todas las empresas (ej: "Activos biologicos" solo aplica a empresas agropecuarias, "Plusvalia" solo a empresas que han adquirido otras).

#### 2.8. Consolidacion del dataset final

Se unieron los 3 estados financieros pivotados en un unico dataset y se agrego informacion de la Camara de Comercio:

| Caracteristica | Valor |
|---|---|
| **Dimensiones** | **180 filas x 192 columnas** |
| Empresas unicas | 57 |
| Rango temporal | 2016 - 2020 |
| Columnas financieras | 148 |
| Columnas de metadatos | 7 (NIT, Razon Social, Organizacion, Categoria, CIIU, Actividad Economica, Grupo NIIF) |
| Conceptos compartidos entre estados | 1 ("Ganancia (perdida)" aparece en Resultado y Flujo → renombrada con sufijo) |

**Distribucion por tipo de organizacion** (57 empresas):

| Tipo de Sociedad | Cantidad |
|---|---|
| S.A.S | 34 |
| Sociedad Limitada | 10 |
| Sociedad Anonima | 10 |
| Sociedad en Comandita | 2 |
| Establecimiento de Comercio | 1 |

**Empresas con mayor cobertura temporal** (5 anios de datos, 2016-2020):

| NIT | Razon Social |
|---|---|
| 890700040 | Ferreteria Godoy S.A. |
| 890702292 | Sociedad Contreras Cajiao S.A.S |
| 800251407 | Agropecuaria La Pilar SAS |
| 809004045 | Fibratela S.A. |
| 890706489 | Distribuciones Zubieta y Cia S.A.S. |
| 890701920 | Agropecuaria La Ceiba Gonella Hnos Ltda |
| 809005111 | Inversiones y Construcciones Duran Cuellar S.A.S |
| 800186960 | Altipal S.A.S - Agencia Ibague |
| 890701760 | Jose I. Diaz M. y Cia. Diagrotol S. en C. |
| 890706754 | Garcia Varela Limitada |

#### 2.9. Archivos generados

| Archivo | Filas | Tamano | Descripcion |
|---|---|---|---|
| `pymes_ibague_situacion_financiera_vertical.csv` | 10,378 | 2.42 MB | Balance - formato vertical (periodo actual) |
| `pymes_ibague_resultado_integral_vertical.csv` | 4,370 | 0.95 MB | Resultados - formato vertical (periodo actual) |
| `pymes_ibague_flujo_efectivo_vertical.csv` | 8,510 | 2.15 MB | Flujo de Efectivo - formato vertical (completo) |
| `pymes_ibague_consolidado.csv` | 180 | 0.13 MB | **Dataset consolidado horizontal** |
| **Total** | | **5.65 MB** | |

Todos los archivos guardados en: `data/`

---

## Hallazgos criticos y problemas identificados

### Problema 1: Perdida de datos 2021-2024 (ALTA PRIORIDAD)

**Descripcion**: La columna TAXONOMIA para los anios 2021 en adelante contiene el valor truncado `"01 - Estados Financieros de Fin de Ejercicio - Corte 202"` (sin el anio completo). El regex de extraccion de anio (`[0-9]{4}`) no puede capturar un anio de este valor, resultando en ANIO = NaN para esos registros.

**Evidencia**: FECHA_CORTE muestra datos de 2021, 2022, 2023 y 2024 disponibles, pero el rango temporal del dataset final solo cubre 2016-2020.

**Impacto**: Se perdieron aproximadamente **13,595 filas** del Estado de Situacion Financiera y cantidades proporcionales de los otros datasets. Esto representa potencialmente **4 anios adicionales de datos**.

**Solucion propuesta**: Usar `FECHA_CORTE` en lugar de (o como complemento de) `TAXONOMIA` para extraer el anio fiscal. FECHA_CORTE contiene fechas completas como `"2023 Dec 31 12:00:00 AM"`.

### Problema 2: Perdida de datos 2016-2017 en Balance y Resultados (MEDIA PRIORIDAD)

**Descripcion**: El filtro de PERIODO (`str.contains('actual|corriente')`) elimino los datos de 2016 y 2017 porque estos anios usan formato de fecha en el campo PERIODO (`"2016-dic-31"`, `"2015-ene-01"`) en lugar de la etiqueta `"Periodo Actual"`.

**Impacto**: Se perdieron datos de 2 anios para Balance y Resultado Integral. El Flujo de Efectivo no se vio afectado porque no se le aplico el filtro.

**Solucion propuesta**: Para los anios 2016-2017, identificar el periodo "actual" comparando las fechas en el campo PERIODO: `"2016-dic-31"` es el cierre del periodo para la taxonomia 2016 (equivalente a "Periodo Actual"), mientras que `"2015-dic-31"` y `"2015-ene-01"` son periodos comparativos.

### Problema 3: Anomalia en PERIODO del Flujo de Efectivo (BAJA PRIORIDAD)

**Descripcion**: El campo PERIODO del dataset de Flujo de Efectivo contiene 46 valores unicos, de los cuales la mayoria son **NITs de empresas** (ej: `800077198`, `809004045`, `890700179`). Solo `"Periodo Actual"` y `"2017"` son valores validos de periodo.

**Impacto**: No afecto directamente el procesamiento (no se aplico filtro de PERIODO al Flujo de Efectivo), pero indica un problema de calidad en el dataset fuente del SIREM. Los NITs en PERIODO podrian causar duplicados si representan filas adicionales para las mismas empresas.

### Problema 4: Discrepancia en numero de empresas entre datasets

**Descripcion**: Tras el procesamiento, el numero de empresas varia entre datasets:

| Etapa | Empresas |
|---|---|
| Match inicial (notebook 1) | 66 (61 NIIF Pymes) |
| Extraccion SIREM (los 3 datasets) | 61 |
| Pivoteo Situacion Financiera | 45 |
| Pivoteo Resultado Integral | 45 |
| Pivoteo Flujo de Efectivo | 57 |
| Dataset consolidado | 57 |

**Causa**: Las 61 empresas fueron extraidas, pero despues del filtro de PERIODO y la extraccion de ANIO, solo 49 tienen datos en Balance/Resultados (45 con ANIO valido + 4 con ANIO=NaN). Las 12 empresas faltantes probablemente solo tienen datos en 2021-2024 (taxonomia truncada) o en 2016-2017 (PERIODO en formato fecha).

---

## Decisiones tecnicas clave

| Aspecto | Decision | Justificacion |
|---|---|---|
| Tipo de dato del NIT | STRING (no numerico) | Preservar ceros iniciales y formato |
| Campo NIT_PARA_JOIN | Quitar ultimo digito a NITs de 10 digitos | Eliminar digito de verificacion para compatibilidad con SIREM |
| Estrategia de carga | Chunks de 500K filas con filtrado simultaneo | Eficiencia de memoria (~7 GB procesados sin cargar todo en RAM) |
| Liberacion de memoria | `gc.collect()` entre cargas | Prevenir agotamiento de los 16 GB de RAM disponibles |
| Filtro de PERIODO | Solo "Periodo Actual" (cuando aplica) | Evitar duplicacion entre anios |
| Extraccion de ANIO | Regex `[0-9]{4}` sobre TAXONOMIA | Limitacion identificada: no funciona para taxonomia 2021+ |
| Manejo de duplicados | `pivot_table` con `aggfunc='first'` | Robusto ante duplicados; conserva primer valor |
| Merge de estados financieros | `outer` join sobre NIT + ANIO | Conservar todas las empresa-anio aunque falte un estado |
| Codificacion de salida | UTF-8 | Compatibilidad con el resto del sistema |

---

## Analisis de viabilidad del dataset para Machine Learning

### Fortalezas

1. **Datos reales y oficiales**: Provienen de fuentes gubernamentales verificables (Supersociedades + Camara de Comercio).
2. **Diversidad de empresas**: 57 empresas de distintos tipos de sociedad y sectores economicos.
3. **Riqueza de features**: 148 columnas financieras disponibles para calculo de indicadores.
4. **Consistencia NIIF**: Todos los datos siguen la misma normativa contable (NIIF Pymes Grupo 2).

### Limitaciones y riesgos

1. **Muestra pequena**: 180 observaciones (empresa-anio) es un dataset reducido para ML. Con la correccion de los problemas 1 y 2, podria ampliarse significativamente.
2. **Desbalance temporal**: Mas empresas reportan en anios recientes (41 en 2020 vs 30 en 2018).
3. **Completitud variable**: Desde 20.8% (Flujo de Efectivo) hasta 72.8% (Resultado Integral). Se requerira estrategia de imputacion o seleccion de features con alta completitud.
4. **Panel desbalanceado**: No todas las empresas tienen datos para todos los anios. Esto limita el analisis de tendencias temporales.

### Estimacion del dataset corregido (si se resuelven los problemas 1 y 2)

| Escenario | Empresa-anio estimadas | Empresas | Anios |
|---|---|---|---|
| Actual (solo 2018-2020 para Balance) | 107 | 45 | 3 |
| Con correccion de 2016-2017 | ~170 | ~55 | 5 |
| Con correccion de 2021-2024 | ~350-450 | ~61 | 9 |

> Con la correccion de ambos problemas, el dataset podria cuadruplicar su tamano, lo cual mejoraria sustancialmente la viabilidad del modelo de ML.

---

## Proximos pasos

### Inmediatos (correccion de datos)

1. **Corregir extraccion de anio**: Usar `FECHA_CORTE` en lugar de `TAXONOMIA` para obtener el anio fiscal completo (2015-2024).
2. **Recuperar datos 2016-2017**: Ajustar el filtro de PERIODO para incluir las fechas de cierre ("2016-dic-31") como equivalentes a "Periodo Actual".
3. **Investigar anomalia de PERIODO en Flujo de Efectivo**: Verificar si los NITs en el campo PERIODO generan duplicados.
4. **Limpiar encoding de CONCEPTO**: Normalizar los caracteres acentuados para presentacion limpia.

### Siguientes fases del proyecto

5. **Calcular indicadores financieros**: liquidez (razon corriente, prueba acida, capital de trabajo), rentabilidad (ROA, ROE, margenes), endeudamiento (razon de deuda, apalancamiento, cobertura de intereses), eficiencia (rotacion de activos, inventarios, cartera).
6. **Crear etiquetas de riesgo** para el entrenamiento del modelo de Machine Learning.
7. **Iniciar la arquitectura del sistema web** (React + Node.js + PostgreSQL).

---

*Documento actualizado: Febrero 2026*
*Notebooks del proyecto: `etl_camara_comercio.ipynb`, `procesamiento_datos_financieros.ipynb`*
*Datos procesados: carpeta `data/`*
