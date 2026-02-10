# Prototipo Funcional del Sistema Web de Analisis Financiero para PYMES

## Documento de Especificacion del Producto

**Proyecto**: "Diseno e Implementacion de un Sistema Web de Analisis Financiero con Modelo de Machine Learning para la Evaluacion de Riesgo en PYMES del Municipio de Ibague, Tolima"

**Version**: 1.0
**Fecha**: Febrero 2026

---

## 1. Vision General del Producto

### 1.1 Que es

Un sistema web que permite evaluar la salud financiera y el nivel de riesgo de PYMES de Ibague a partir de datos oficiales del SIREM (Superintendencia de Sociedades). El sistema combina indicadores financieros tradicionales con un modelo de Machine Learning para clasificar el riesgo empresarial.

### 1.2 Para quien es

| Perfil de Usuario | Descripcion | Objetivo Principal |
|---|---|---|
| **Analista financiero** | Profesional que evalua empresas para creditos, inversiones o consultoria | Obtener un diagnostico rapido y confiable de la salud financiera de una PYME |
| **Gerente/propietario PYME** | Dueno o administrador de una pequena o mediana empresa | Entender la situacion financiera de su empresa y compararla con el sector |
| **Entidad bancaria** | Analista de riesgo crediticio | Evaluar el riesgo de otorgar credito a una PYME |
| **Investigador academico** | Estudiantes o docentes de finanzas | Analizar patrones financieros del sector PYME en Ibague |

### 1.3 Propuesta de valor

- **Diagnostico automatizado**: En lugar de calcular indicadores manualmente, el sistema los genera en segundos
- **Clasificacion de riesgo por ML**: No solo muestra numeros, sino que interpreta y clasifica el riesgo (bajo, medio, alto)
- **Contexto sectorial**: Los indicadores se comparan contra promedios del sector para dar sentido a los numeros
- **Historico temporal**: Permite ver la evolucion financiera de una empresa en el tiempo (2016-2024)
- **Datos oficiales verificados**: Toda la informacion proviene de fuentes oficiales (Supersociedades y Camara de Comercio)

---

## 2. Datos del Sistema

### 2.1 Datos precargados (no los ingresa el usuario)

El sistema viene con datos procesados de **61 PYMES de Ibague** registradas en el SIREM, cubriendo el periodo **2016-2024**. Estos datos ya estan limpios, transformados y listos para consulta.

| Dato Precargado | Fuente | Detalle |
|---|---|---|
| Informacion general de la empresa | Camara de Comercio de Ibague | Razon social, NIT, tipo de organizacion (S.A.S., Ltda., S.A.), sector economico (CIIU) |
| Estado de Situacion Financiera | SIREM - Supersociedades | Activos, pasivos, patrimonio (82 conceptos contables) |
| Estado de Resultado Integral | SIREM - Supersociedades | Ingresos, costos, gastos, utilidades (24 conceptos contables) |
| Estado de Flujo de Efectivo | SIREM - Supersociedades | Flujos de operacion, inversion y financiacion (103 conceptos contables) |
| 18 indicadores financieros calculados | Derivado de estados financieros | Liquidez, rentabilidad, endeudamiento, eficiencia |
| Clasificacion de riesgo | Modelo ML (Z-Score Altman + heuristica) | Riesgo bajo, medio o alto por empresa-anio |

### 2.2 Datos que ingresa el usuario (consultas manuales)

El sistema permite al usuario ingresar datos financieros de una empresa **externa** (que no este en la base de datos) para obtener un diagnostico. Los campos que deberia llenar son:

#### Formulario de Ingreso Manual

**Seccion 1: Identificacion de la Empresa**

| Campo | Tipo | Obligatorio | Ejemplo |
|---|---|---|---|
| Nombre de la empresa | Texto | Si | "Mi Empresa S.A.S." |
| NIT | Numerico (9 digitos) | No | 901234567 |
| Tipo de organizacion | Selector | Si | S.A.S. / Ltda. / S.A. / Otra |
| Sector economico | Selector (CIIU) | Si | Comercio / Manufactura / Servicios / Construccion / Agro / Otro |
| Anio del reporte | Selector | Si | 2024 |

**Seccion 2: Balance General (cifras en pesos colombianos)**

| Campo | Obligatorio | Indicadores que alimenta |
|---|---|---|
| Total de activos | Si | ROA, Rotacion activos, Razon deuda, Apalancamiento, Z-Score |
| Activos corrientes totales | Si | Razon corriente, Prueba acida, Capital trabajo, Z-Score |
| Efectivo y equivalentes | No | Razon de efectivo |
| Cuentas por cobrar corrientes | No | Rotacion de cartera, Dias de cartera |
| Inventarios corrientes | No | Prueba acida, Rotacion inventarios, Dias inventario |
| Total pasivos | Si | Razon deuda, Deuda/Patrimonio, Z-Score |
| Pasivos corrientes totales | Si | Razon corriente, Prueba acida, Capital trabajo |
| Patrimonio total | Si | ROE, Deuda/Patrimonio, Apalancamiento, Z-Score |
| Ganancias acumuladas | No | Z-Score (X2) |

**Seccion 3: Estado de Resultados (cifras del periodo)**

| Campo | Obligatorio | Indicadores que alimenta |
|---|---|---|
| Ingresos de actividades ordinarias (ventas) | Si | Todos los margenes, Rotacion activos, Z-Score |
| Costo de ventas | No | Margen bruto, Rotacion inventarios |
| Ganancia bruta | No (se calcula si no se ingresa) | Margen bruto |
| Ganancia (perdida) operacional | Si | Margen operacional, Cobertura intereses, Z-Score |
| Costos financieros (intereses) | No | Cobertura de intereses |
| Ganancia (perdida) neta | Si | Margen neto, ROA, ROE |

> **Nota**: Los campos marcados como obligatorios son el minimo para calcular los indicadores principales y el Z-Score. Los campos opcionales permiten calcular indicadores adicionales y obtener un diagnostico mas completo.

#### Validaciones del Formulario

| Regla | Mensaje de Error |
|---|---|
| Total de activos > 0 | "Los activos totales deben ser positivos" |
| Total activos = Total pasivos + Patrimonio (+/- 1% tolerancia) | "La ecuacion contable no cuadra. Verifique los datos." |
| Activos corrientes <= Total activos | "Los activos corrientes no pueden superar los activos totales" |
| Pasivos corrientes <= Total pasivos | "Los pasivos corrientes no pueden superar los pasivos totales" |
| Ingresos >= 0 | "Los ingresos no pueden ser negativos" |
| Anio entre 2016 y anio actual | "Seleccione un anio valido" |

---

## 3. Funcionalidades del Sistema

### 3.1 Modulo 1: Busqueda y Consulta de Empresa

**Que hace**: El usuario busca una empresa de la base de datos y obtiene su perfil financiero completo.

**Flujo del usuario**:

```
1. Usuario ingresa a la pagina principal
2. Ve un campo de busqueda: "Buscar empresa por nombre o NIT"
3. Escribe el nombre (ej: "Ferreteria") o NIT (ej: "890702298")
4. El sistema muestra resultados coincidentes en una lista desplegable
5. Usuario selecciona la empresa deseada
6. Se carga el perfil completo de la empresa
```

**Pantalla de resultados - Perfil de Empresa**:

```
+---------------------------------------------------------------+
|  FERRETERIA GODOY S.A.                                        |
|  NIT: 890702298 | S.A.S. | Sector: Comercio                  |
|  Datos disponibles: 2016 - 2024 (9 anios)                     |
|                                                                |
|  [2024]  [2023]  [2022]  [2021]  ... (pestanas por anio)      |
+---------------------------------------------------------------+
|                                                                |
|  CLASIFICACION DE RIESGO (2024)                                |
|  +---------------------------------------------------+        |
|  |   [=====] RIESGO BAJO  |  Z-Score: 3.42           |        |
|  |   (semaforo verde)      |  Confianza: 87%          |        |
|  +---------------------------------------------------+        |
|                                                                |
|  INDICADORES CLAVE (2024)                                      |
|  +-------------+  +-------------+  +-------------+            |
|  | Razon       |  | Margen      |  | Razon de    |            |
|  | Corriente   |  | Neto        |  | Deuda       |            |
|  |   2.15      |  |   8.3%      |  |   45%       |            |
|  |  (Bueno)    |  |  (Bueno)    |  |  (Bueno)    |            |
|  +-------------+  +-------------+  +-------------+            |
|  +-------------+  +-------------+  +-------------+            |
|  | ROA         |  | ROE         |  | Cobertura   |            |
|  |   6.2%      |  |   11.3%     |  | Intereses   |            |
|  |  (Bueno)    |  |  (Bueno)    |  |   4.5x      |            |
|  +-------------+  +-------------+  +-------------+            |
|                                                                |
+---------------------------------------------------------------+
```

**Interpretacion visual de cada indicador**: El sistema usa un semaforo de colores:
- **Verde**: Indicador en rango saludable
- **Amarillo**: Indicador en zona de atencion
- **Rojo**: Indicador en zona de riesgo

| Indicador | Verde | Amarillo | Rojo |
|---|---|---|---|
| Razon Corriente | > 1.5 | 1.0 - 1.5 | < 1.0 |
| Prueba Acida | > 1.0 | 0.5 - 1.0 | < 0.5 |
| Capital de Trabajo | > 0 (holgado) | > 0 (ajustado) | < 0 |
| Razon de Efectivo | > 0.3 | 0.1 - 0.3 | < 0.1 |
| Margen Bruto | > 30% | 15% - 30% | < 15% |
| Margen Operacional | > 10% | 0% - 10% | < 0% |
| Margen Neto | > 5% | 0% - 5% | < 0% |
| ROA | > 5% | 0% - 5% | < 0% |
| ROE | > 10% | 0% - 10% | < 0% |
| Razon de Deuda | < 50% | 50% - 70% | > 70% |
| Deuda/Patrimonio | < 1.5 | 1.5 - 3.0 | > 3.0 |
| Apalancamiento | < 2.5 | 2.5 - 4.0 | > 4.0 |
| Cobertura Intereses | > 3.0 | 1.5 - 3.0 | < 1.5 |
| Rotacion Activos | > 1.0 | 0.5 - 1.0 | < 0.5 |
| Dias de Inventario | < 60 | 60 - 120 | > 120 |
| Dias de Cartera | < 60 | 60 - 90 | > 90 |

---

### 3.2 Modulo 2: Dashboard de Evolucion Temporal

**Que hace**: Muestra la evolucion de los indicadores financieros de una empresa a lo largo de los anios con graficos interactivos.

**Graficos disponibles**:

#### Grafico 1: Evolucion del Riesgo (linea temporal)

```
  Riesgo
  Alto  |          *
        |     *         *
  Medio |  *     *         *    *
        |                          *
  Bajo  |                               *
        +--+-----+-----+-----+-----+-----+---
          2016  2017  2018  2019  2020  2021  2022  2023  2024

  [La linea muestra como ha cambiado la clasificacion de riesgo]
  [El punto de 2020 podria subir por efectos COVID-19]
```

#### Grafico 2: Indicadores de Liquidez (lineas multiples)

```
  Valor
  3.0 |
  2.5 |  ---  Razon Corriente
  2.0 |  - -  Prueba Acida
  1.5 |  ...  Razon Efectivo
  1.0 |  ___  Umbral minimo deseable
  0.5 |
  0.0 +--+-----+-----+-----+-----+-----+---
        2016  2017  2018  2019  2020  2021 ...
```

#### Grafico 3: Estructura Financiera (barras apiladas por anio)

```
  100% |  [Patrimonio]  [Pasivos Corrientes]  [Pasivos No Corrientes]
       |
   75% |  ########  ########  ########  ########
       |  ########  ########  ########  ########
   50% |  --------  --------  --------  --------
       |  ::::::::  ::::::::  ::::::::  ::::::::
   25% |  ::::::::  ::::::::  ::::::::  ::::::::
       |  ::::::::  ::::::::  ::::::::  ::::::::
    0% +----2021------2022------2023------2024---
```

#### Grafico 4: Rentabilidad Comparada (barras agrupadas)

```
  Muestra Margen Bruto, Margen Operacional y Margen Neto
  agrupados por anio, permitiendo ver la compresion
  de margenes a lo largo del tiempo.
```

#### Grafico 5: Flujo de Efectivo (cascada/waterfall)

```
  Muestra los flujos de operacion (+), inversion (-) y
  financiacion (+/-) y el cambio neto en efectivo,
  visualizado como barras que suben y bajan.
```

**Interactividad**:
- El usuario puede seleccionar/deseleccionar indicadores del grafico
- Tooltip al pasar el mouse sobre un punto: muestra el valor exacto y su interpretacion
- Selector de rango de anios (filtrar por periodo)
- Boton para descargar el grafico como imagen PNG

---

### 3.3 Modulo 3: Evaluacion de Riesgo con Machine Learning

**Que hace**: Toma los datos financieros (de la base de datos o ingresados manualmente) y ejecuta el modelo de ML para clasificar el riesgo.

#### Flujo A: Evaluacion de empresa de la base de datos

```
1. Usuario busca y selecciona una empresa
2. Selecciona el anio a evaluar
3. Hace clic en "Evaluar Riesgo"
4. El sistema calcula los 18 indicadores automaticamente
5. Ejecuta el modelo de ML (XGBoost)
6. Muestra el resultado con la clasificacion y explicacion
```

#### Flujo B: Evaluacion de empresa externa (datos manuales)

```
1. Usuario hace clic en "Evaluar Nueva Empresa"
2. Completa el formulario de ingreso manual (Seccion 2.2)
3. El sistema valida los datos (ecuacion contable, rangos)
4. Calcula los indicadores financieros
5. Ejecuta el modelo de ML
6. Muestra resultado con clasificacion, explicacion y comparacion sectorial
```

#### Pantalla de Resultado de Evaluacion

```
+---------------------------------------------------------------+
|  RESULTADO DE EVALUACION DE RIESGO                            |
+---------------------------------------------------------------+
|                                                                |
|  Empresa: FERRETERIA GODOY S.A.S. | Anio: 2024               |
|                                                                |
|  +---------------------------------------------------+        |
|  |                                                   |        |
|  |     RIESGO: ██████ BAJO ██████                    |        |
|  |                                                   |        |
|  |     Z-Score de Altman: 3.42                       |        |
|  |     Zona: Segura (> 2.90)                         |        |
|  |                                                   |        |
|  |     Modelo ML (XGBoost):                          |        |
|  |       Probabilidad Riesgo Bajo:  82%              |        |
|  |       Probabilidad Riesgo Medio: 15%              |        |
|  |       Probabilidad Riesgo Alto:   3%              |        |
|  |                                                   |        |
|  +---------------------------------------------------+        |
|                                                                |
|  FACTORES PRINCIPALES DE LA CLASIFICACION                     |
|  (Que indicadores pesaron mas en la decision del modelo)      |
|                                                                |
|  1. Razon de deuda: 0.45 (45%) .............. (+) Favorable   |
|  2. Margen operacional: 12.3% ............... (+) Favorable   |
|  3. Cobertura de intereses: 4.5x ............ (+) Favorable   |
|  4. Razon corriente: 2.15 ................... (+) Favorable   |
|  5. Rotacion de activos: 0.85 ............... (!) Mejorable   |
|                                                                |
|  DIAGNOSTICO NARRATIVO                                        |
|  "La empresa presenta una situacion financiera saludable.     |
|   Su nivel de endeudamiento es moderado (45%), con buena      |
|   capacidad para cubrir sus obligaciones de corto plazo       |
|   (razon corriente de 2.15). La rentabilidad operacional      |
|   es positiva (12.3%) y puede cubrir sus costos financieros   |
|   4.5 veces. El area de mejora identificada es la eficiencia  |
|   en el uso de activos (rotacion de 0.85, por debajo del      |
|   promedio sectorial de 1.2)."                                |
|                                                                |
+---------------------------------------------------------------+
```

**Componentes del diagnostico narrativo** (generado automaticamente segun los valores):

| Aspecto | Que dice | Cuando |
|---|---|---|
| Liquidez | "La empresa tiene capacidad/dificultad para cubrir obligaciones de corto plazo" | Siempre |
| Rentabilidad | "La operacion genera/no genera utilidades suficientes" | Siempre |
| Endeudamiento | "El nivel de deuda es bajo/moderado/alto" | Siempre |
| Eficiencia | "La empresa usa sus activos de forma eficiente/ineficiente" | Si hay datos |
| Tendencia | "La situacion ha mejorado/empeorado respecto al anio anterior" | Si hay datos historicos |
| Comparacion | "Comparado con empresas similares del sector, se encuentra por encima/debajo del promedio" | Siempre |

---

### 3.4 Modulo 4: Comparacion Sectorial

**Que hace**: Permite comparar una empresa contra el promedio del sector o contra otras empresas de la base de datos.

#### Vista 1: Empresa vs Promedio del Sector

```
+---------------------------------------------------------------+
|  COMPARACION SECTORIAL                                        |
|  Empresa: FERRETERIA GODOY S.A.S. vs Sector Comercio (2024)  |
+---------------------------------------------------------------+
|                                                                |
|  Grafico de Radar (Arana)                                     |
|                                                                |
|              Razon Corriente                                   |
|                   *                                            |
|                 / | \                                          |
|    Rotacion   /  |  \   Margen                                |
|    Activos --*   |   *-- Operacional                          |
|               \  |  /                                          |
|                \ | /                                           |
|    Cobertura    \|/    ROA                                     |
|    Intereses ----*---- (centro)                                |
|                                                                |
|    --- Empresa (azul)                                          |
|    --- Promedio sector (gris punteado)                         |
|                                                                |
|  Tabla Comparativa:                                            |
|  +-------------------+-----------+----------+---------+       |
|  | Indicador         | Empresa   | Sector   | Estado  |       |
|  +-------------------+-----------+----------+---------+       |
|  | Razon Corriente   |    2.15   |   1.65   |  Mejor  |       |
|  | Margen Neto       |    8.3%   |   5.1%   |  Mejor  |       |
|  | Razon Deuda       |   45.0%   |  52.3%   |  Mejor  |       |
|  | Rotacion Activos  |    0.85   |   1.20   |  Peor   |       |
|  | ROE               |   11.3%   |   8.7%   |  Mejor  |       |
|  | Dias Cartera      |     72    |    58    |  Peor   |       |
|  +-------------------+-----------+----------+---------+       |
|                                                                |
|  INTERPRETACION:                                               |
|  "La empresa supera al promedio del sector en 4 de 6          |
|   indicadores clave. Las areas de mejora frente al sector     |
|   son: rotacion de activos y dias de cartera."                |
+---------------------------------------------------------------+
```

#### Vista 2: Ranking de Empresas

```
+---------------------------------------------------------------+
|  RANKING DE PYMES - IBAGUE (2024)                             |
|  Ordenar por: [Selector: Riesgo / Rentabilidad / Liquidez]    |
+---------------------------------------------------------------+
|                                                                |
|  #  | Empresa                    | Riesgo | Z-Score | ROA     |
|  ---+----------------------------+--------+---------+---------+
|  1  | Empresa ABC S.A.S.         | Bajo   |  4.12   | 12.3%   |
|  2  | Empresa DEF Ltda.          | Bajo   |  3.87   |  9.8%   |
|  3  | FERRETERIA GODOY S.A.S.    | Bajo   |  3.42   |  6.2%   |
|  4  | Empresa GHI S.A.           | Medio  |  2.45   |  3.1%   |
|  5  | Empresa JKL Ltda.          | Medio  |  1.98   |  1.5%   |
|  ...                                                           |
|  58 | Empresa XYZ S.A.S.         | Alto   |  0.87   | -5.2%   |
|                                                                |
|  [Filtros: Sector | Tipo Organizacion | Rango de Anios]       |
|                                                                |
+---------------------------------------------------------------+
```

#### Vista 3: Comparacion Entre Dos Empresas

```
El usuario selecciona dos empresas de la base de datos y ve
una comparacion lado a lado con todos los indicadores,
graficos de radar superpuestos y tendencias temporales comparadas.
```

---

### 3.5 Modulo 5: Dashboard General (Panorama del Sector)

**Que hace**: Muestra estadisticas agregadas de todas las 61 PYMES de Ibague, dando un panorama general del sector empresarial.

#### Tarjetas Resumen (parte superior del dashboard)

```
+-------------+  +-------------+  +-------------+  +-------------+
| 61          |  | 35          |  | 18          |  | 8           |
| Empresas    |  | Riesgo Bajo |  | Riesgo Medio|  | Riesgo Alto |
| analizadas  |  | (57%)       |  | (30%)       |  | (13%)       |
+-------------+  +-------------+  +-------------+  +-------------+
```

#### Grafico 1: Distribucion de Riesgo (dona/pie chart)

```
Muestra la proporcion de empresas en cada nivel de riesgo
para el anio seleccionado, con comparacion contra el anio anterior.
```

#### Grafico 2: Evolucion del Riesgo Agregado (area apilada)

```
  100% |  ████████████████████████████████████
       |  ████████ Riesgo Alto ████████████████
   75% |  ████████████████████████████████████
       |  -------- Riesgo Medio ---------------
   50% |  ::::::::::::::::::::::::::::::::::::
       |  ::::::::::::::::::::::::::::::::::::
   25% |          Riesgo Bajo
       |  ::::::::::::::::::::::::::::::::::::
    0% +----2016---2018---2020---2022---2024---

  Muestra como ha cambiado la proporcion de riesgo en el sector
  a lo largo de los anios (se espera ver el efecto COVID en 2020-2021).
```

#### Grafico 3: Promedios Sectoriales por Tipo de Organizacion

```
  Barras agrupadas que muestran el ROA, Margen Neto y Razon
  de Deuda promedio, agrupados por tipo de organizacion
  (S.A.S. vs Ltda. vs S.A.).
```

#### Grafico 4: Mapa de Calor de Indicadores

```
  Heatmap que muestra todos los indicadores (filas) por anio (columnas),
  coloreados de verde (saludable) a rojo (critico), para el promedio
  del sector. Permite identificar rapidamente que indicadores se han
  deteriorado y en que anios.

                2016  2017  2018  2019  2020  2021  2022  2023  2024
  Razon Corr.   ██    ██    ██    ██    ░░    ██    ██    ██    ██
  Margen Neto   ██    ██    ██    ██    ░░    ░░    ██    ██    ██
  Razon Deuda   ██    ██    ░░    ░░    ░░    ░░    ░░    ██    ██
  ROA           ██    ██    ██    ██    ░░    ░░    ██    ██    ██

  ██ = saludable (verde)  ░░ = atencion (amarillo)  XX = critico (rojo)
```

#### Grafico 5: Distribucion de Indicadores (box plots)

```
  Para el anio seleccionado, muestra la distribucion estadistica
  de cada indicador (mediana, cuartiles, outliers) de las 61 empresas.
  Permite ver la dispersion del sector y detectar empresas atipicas.
```

#### Filtros Disponibles en el Dashboard

| Filtro | Opciones | Efecto |
|---|---|---|
| Anio | 2016 - 2024 | Filtra todos los graficos al anio seleccionado |
| Tipo de organizacion | S.A.S. / Ltda. / S.A. / Todas | Filtra por forma societaria |
| Sector economico | Comercio / Manufactura / Servicios / Todos | Filtra por actividad CIIU |
| Nivel de riesgo | Bajo / Medio / Alto / Todos | Filtra por clasificacion ML |

---

### 3.6 Modulo 6: Reportes Descargables

**Que hace**: Genera informes en PDF que el usuario puede descargar para uso externo (presentaciones, solicitudes de credito, informes academicos).

#### Reporte 1: Informe Individual de Empresa (PDF)

Contenido del PDF generado:

```
INFORME DE ANALISIS FINANCIERO
=================================
Empresa: [Nombre]
NIT: [NIT]
Periodo analizado: [Anio o rango de anios]
Fecha de generacion: [Fecha actual]

1. INFORMACION GENERAL
   - Tipo de organizacion, sector, anios de datos disponibles

2. CLASIFICACION DE RIESGO
   - Resultado del modelo ML (bajo/medio/alto)
   - Z-Score de Altman y probabilidades
   - Factores determinantes

3. INDICADORES FINANCIEROS
   - Tabla con los 18 indicadores del ultimo anio
   - Semaforo de estado de cada uno
   - Comparacion con promedio del sector

4. EVOLUCION TEMPORAL
   - Graficos de tendencia de indicadores clave
   - Resumen narrativo de la evolucion

5. DIAGNOSTICO Y RECOMENDACIONES
   - Fortalezas identificadas
   - Areas de mejora
   - Recomendaciones basadas en los datos

6. NOTA METODOLOGICA
   - Fuentes de datos utilizadas
   - Modelo ML empleado
   - Limitaciones del analisis
```

#### Reporte 2: Informe Sectorial (PDF)

```
INFORME PANORAMA SECTORIAL - PYMES IBAGUE
============================================

1. Resumen ejecutivo del sector
2. Distribucion de riesgo
3. Indicadores promedio y su evolucion
4. Ranking de empresas
5. Tendencias identificadas
6. Conclusiones
```

#### Reporte 3: Exportacion de Datos (CSV/Excel)

```
El usuario puede exportar:
- Indicadores financieros de una empresa (todos los anios)
- Indicadores de todas las empresas (un anio)
- Dataset completo con indicadores y clasificaciones
```

---

## 4. Arquitectura Tecnica Propuesta

### 4.1 Stack Tecnologico

| Componente | Tecnologia | Justificacion |
|---|---|---|
| **Frontend** | React.js | Componentes reutilizables, ecosistema amplio de librerias de graficos |
| **Graficos** | Recharts o Chart.js | Graficos interactivos (radar, lineas, barras, heatmaps) |
| **Backend** | Node.js + Express | API REST para servir datos al frontend |
| **Base de datos** | PostgreSQL | Datos estructurados, consultas complejas, buen soporte para series temporales |
| **Motor ML** | Python (scikit-learn / XGBoost) | Modelo entrenado, se expone via API interna |
| **Generacion PDF** | jsPDF o Puppeteer | Para los reportes descargables |
| **Autenticacion** | JWT (JSON Web Tokens) | Sesiones seguras sin estado |

### 4.2 Diagrama de Arquitectura

```
+-------------------+        +-------------------+        +------------------+
|                   |  HTTP  |                   |  SQL   |                  |
|   Frontend        |<------>|   Backend         |<------>|   PostgreSQL     |
|   (React.js)      |  REST  |   (Node.js +      |        |                  |
|                   |  API   |    Express)        |        |  - Empresas      |
|   - Dashboard     |        |                   |        |  - Estados Fin.  |
|   - Busqueda      |        |   - API REST      |        |  - Indicadores   |
|   - Evaluacion    |        |   - Autenticacion |        |  - Clasificacion |
|   - Reportes      |        |   - Validaciones  |        |  - Usuarios      |
|   - Graficos      |        |                   |        |                  |
+-------------------+        +--------+----------+        +------------------+
                                      |
                                      | HTTP (interno)
                                      |
                              +-------v----------+
                              |                  |
                              |  Motor ML        |
                              |  (Python API)    |
                              |                  |
                              |  - XGBoost       |
                              |  - Calculo       |
                              |    indicadores   |
                              |  - Z-Score       |
                              |  - Prediccion    |
                              |                  |
                              +------------------+
```

### 4.3 Estructura de la Base de Datos (esquema simplificado)

```sql
-- Empresas registradas
empresas (
    id, nit, razon_social, organizacion, sector_ciiu, grupo_niif
)

-- Estados financieros anuales (formato horizontal)
estados_financieros (
    id, empresa_id, anio, tipo_estado,
    -- Campos varian segun tipo_estado
    total_activos, activos_corrientes, pasivos_corrientes, ...
)

-- Indicadores calculados
indicadores (
    id, empresa_id, anio,
    razon_corriente, prueba_acida, capital_trabajo, razon_efectivo,
    margen_bruto, margen_operacional, margen_neto, roa, roe,
    razon_deuda, deuda_patrimonio, apalancamiento, cobertura_intereses,
    rotacion_activos, rotacion_inventarios, rotacion_cartera,
    dias_inventario, dias_cartera
)

-- Clasificaciones de riesgo
clasificaciones (
    id, empresa_id, anio,
    z_score, clasificacion_zscore,
    probabilidad_bajo, probabilidad_medio, probabilidad_alto,
    clasificacion_ml, modelo_version
)

-- Usuarios del sistema
usuarios (
    id, nombre, email, password_hash, rol
)

-- Evaluaciones manuales (empresas externas)
evaluaciones_manuales (
    id, usuario_id, fecha,
    nombre_empresa, nit, sector,
    datos_ingresados (JSON),
    indicadores_calculados (JSON),
    clasificacion_resultado, z_score
)
```

---

## 5. Flujos de Usuario Completos

### 5.1 Flujo: "Quiero saber como esta mi empresa"

```
Pagina principal
    |
    v
Buscar empresa por nombre o NIT
    |
    v
Seleccionar empresa de los resultados
    |
    v
Ver perfil de empresa (indicadores + riesgo del ultimo anio)
    |
    +---> Ver evolucion temporal (graficos historicos)
    |
    +---> Comparar con el sector (grafico radar + tabla)
    |
    +---> Descargar informe PDF
```

### 5.2 Flujo: "Quiero evaluar una empresa que no esta en el sistema"

```
Pagina principal
    |
    v
Clic en "Evaluar Nueva Empresa"
    |
    v
Completar formulario de datos financieros
    |
    v
El sistema valida los datos (ecuacion contable, rangos logicos)
    |
    +--> Error de validacion --> Corregir datos
    |
    v
El sistema calcula indicadores + ejecuta modelo ML
    |
    v
Resultado: clasificacion de riesgo + diagnostico + comparacion sectorial
    |
    +---> Descargar informe PDF con resultado
```

### 5.3 Flujo: "Quiero ver el panorama general del sector PYME en Ibague"

```
Menu principal --> Dashboard General
    |
    v
Ver tarjetas resumen (total empresas, distribucion de riesgo)
    |
    v
Explorar graficos:
    +---> Distribucion de riesgo (pie chart)
    +---> Evolucion temporal del riesgo (area chart)
    +---> Promedios sectoriales (barras)
    +---> Mapa de calor de indicadores (heatmap)
    +---> Distribucion estadistica (box plots)
    |
    v
Aplicar filtros (anio, sector, tipo organizacion)
    |
    v
Ver ranking de empresas
    |
    +---> Clic en empresa del ranking --> Ir a perfil de empresa
    |
    +---> Descargar informe sectorial PDF
```

### 5.4 Flujo: "Quiero comparar dos empresas"

```
Menu principal --> Comparar Empresas
    |
    v
Seleccionar Empresa A (busqueda)
    |
    v
Seleccionar Empresa B (busqueda)
    |
    v
Seleccionar anio de comparacion
    |
    v
Ver comparacion lado a lado:
    +---> Tabla de indicadores comparados
    +---> Grafico radar superpuesto
    +---> Clasificacion de riesgo de cada una
    +---> Tendencias temporales comparadas
```

---

## 6. Indicadores Calculados: Detalle Completo

### 6.1 Los 18 Indicadores del Sistema

Para cada indicador, el sistema muestra:

| Elemento | Descripcion |
|---|---|
| **Valor numerico** | El resultado del calculo (ej: 2.15) |
| **Unidad** | Veces (x), porcentaje (%), pesos ($), dias |
| **Semaforo** | Verde / Amarillo / Rojo segun umbrales |
| **Etiqueta interpretativa** | "Bueno", "Aceptable", "Atencion", "Critico" |
| **Valor del sector** | Promedio de las 61 PYMES para comparacion |
| **Tendencia** | Flecha arriba/abajo comparando con anio anterior |
| **Explicacion** | Texto corto explicando que significa el indicador en lenguaje simple |

### 6.2 Tabla Completa de Indicadores

#### Liquidez

| # | Indicador | Formula | Unidad | Que responde |
|---|---|---|---|---|
| 1 | Razon Corriente | Activos corrientes / Pasivos corrientes | Veces | Puede pagar sus deudas de corto plazo? |
| 2 | Prueba Acida | (Activos corrientes - Inventarios) / Pasivos corrientes | Veces | Puede pagar sin depender de vender inventario? |
| 3 | Capital de Trabajo | Activos corrientes - Pasivos corrientes | Pesos ($) | Cuanto le sobra (o falta) para operar dia a dia? |
| 4 | Razon de Efectivo | Efectivo / Pasivos corrientes | Veces | Puede pagar solo con el dinero en caja? |

#### Rentabilidad

| # | Indicador | Formula | Unidad | Que responde |
|---|---|---|---|---|
| 5 | Margen Bruto | Ganancia bruta / Ingresos | Porcentaje | Cuanto gana por cada peso vendido antes de gastos? |
| 6 | Margen Operacional | Ganancia operacional / Ingresos | Porcentaje | Cuanto gana por cada peso vendido despues de operar? |
| 7 | Margen Neto | Ganancia neta / Ingresos | Porcentaje | Cuanto gana realmente por cada peso vendido? |
| 8 | ROA | Ganancia neta / Total activos | Porcentaje | Que tan bien usa sus activos para generar ganancias? |
| 9 | ROE | Ganancia neta / Patrimonio | Porcentaje | Que rendimiento obtienen los socios por su inversion? |

#### Endeudamiento

| # | Indicador | Formula | Unidad | Que responde |
|---|---|---|---|---|
| 10 | Razon de Deuda | Total pasivos / Total activos | Porcentaje | Que proporcion de la empresa esta financiada con deuda? |
| 11 | Deuda/Patrimonio | Total pasivos / Patrimonio | Veces | Cuantos pesos debe por cada peso propio? |
| 12 | Apalancamiento | Total activos / Patrimonio | Veces | Cuanto se multiplican los recursos propios con deuda? |
| 13 | Cobertura de Intereses | Ganancia operacional / Costos financieros | Veces | Puede pagar los intereses de sus prestamos? |

#### Eficiencia

| # | Indicador | Formula | Unidad | Que responde |
|---|---|---|---|---|
| 14 | Rotacion de Activos | Ingresos / Total activos | Veces | Cuantos pesos de venta genera cada peso de activo? |
| 15 | Rotacion de Inventarios | Costo de ventas / Inventarios | Veces | Cuantas veces al anio renueva su inventario? |
| 16 | Rotacion de Cartera | Ingresos / Cuentas por cobrar | Veces | Cuantas veces al anio cobra a sus clientes? |
| 17 | Dias de Inventario | 365 / Rotacion inventarios | Dias | Cuantos dias tarda en vender su inventario? |
| 18 | Dias de Cartera | 365 / Rotacion cartera | Dias | Cuantos dias tarda en cobrar? |

### 6.3 Indicador Especial: Z-Score de Altman

Ademas de los 18 indicadores, el sistema calcula el Z-Score de Altman adaptado para empresas no cotizadas:

```
Z' = 0.717 * (Capital Trabajo / Total Activos)
   + 0.847 * (Ganancias Acumuladas / Total Activos)
   + 3.107 * (Ganancia Operacional / Total Activos)
   + 0.420 * (Patrimonio / Total Pasivos)
   + 0.998 * (Ingresos / Total Activos)
```

| Resultado | Clasificacion | Significado |
|---|---|---|
| Z' > 2.90 | **Zona Segura** (Riesgo Bajo) | La empresa tiene baja probabilidad de quiebra |
| 1.23 < Z' < 2.90 | **Zona Gris** (Riesgo Medio) | Situacion incierta, requiere atencion |
| Z' < 1.23 | **Zona de Peligro** (Riesgo Alto) | Alta probabilidad de dificultades financieras |

---

## 7. Mapa de Navegacion del Sistema

```
+---------------------------+
|     PAGINA PRINCIPAL      |
|  (Busqueda + Acceso       |
|   rapido a modulos)       |
+---------------------------+
    |          |          |           |
    v          v          v           v
+--------+ +--------+ +----------+ +----------+
| Perfil | | Evaluar| | Dashboard| | Comparar |
| Empresa| | Nueva  | | General  | | Empresas |
+--------+ +--------+ +----------+ +----------+
    |          |          |           |
    v          v          v           v
+--------+ +--------+ +----------+ +----------+
| Evolu- | |Resultado| | Ranking  | | Lado a  |
| cion   | |de Riesgo| | Empresas | | Lado    |
+--------+ +--------+ +----------+ +----------+
    |          |          |           |
    v          v          v           v
+--------------------------------------------------+
|              DESCARGAR REPORTE (PDF / CSV)        |
+--------------------------------------------------+
```

### Barra de Navegacion Principal

```
+---------------------------------------------------------------+
| [Logo]  Inicio | Dashboard | Evaluar | Comparar | Reportes    |
+---------------------------------------------------------------+
```

---

## 8. Consideraciones de Diseno (UX/UI)

### 8.1 Principios de Diseno

| Principio | Aplicacion |
|---|---|
| **Simplicidad** | Los indicadores se muestran con lenguaje simple, no solo numeros. Cada indicador tiene una frase explicativa en lenguaje cotidiano |
| **Semaforo visual** | Verde/amarillo/rojo para que el usuario entienda de un vistazo sin necesidad de interpretar numeros |
| **Progresividad** | Se muestra primero el resumen (riesgo general), y el usuario profundiza si quiere ver el detalle |
| **Responsividad** | El sistema se adapta a pantallas de escritorio, tablet y movil |
| **Accesibilidad** | Contraste de colores adecuado, textos alternativos en graficos, fuentes legibles |

### 8.2 Paleta de Colores Sugerida

| Uso | Color | Codigo |
|---|---|---|
| Riesgo Bajo / Indicador saludable | Verde | #22C55E |
| Riesgo Medio / Indicador atencion | Amarillo/Naranja | #F59E0B |
| Riesgo Alto / Indicador critico | Rojo | #EF4444 |
| Fondo principal | Blanco/Gris claro | #F8FAFC |
| Texto principal | Gris oscuro | #1E293B |
| Acentos / Botones principales | Azul institucional | #3B82F6 |

### 8.3 Componentes Reutilizables

| Componente | Donde se usa |
|---|---|
| **Tarjeta de indicador** (valor + semaforo + tendencia) | Perfil empresa, Dashboard, Comparacion |
| **Grafico de radar** | Comparacion sectorial, Comparacion entre empresas |
| **Grafico de linea temporal** | Evolucion empresa, Dashboard |
| **Tabla ordenable/filtrable** | Ranking, Indicadores detallados |
| **Badge de riesgo** (bajo/medio/alto con color) | Perfil empresa, Ranking, Resultado evaluacion |
| **Selector de anio** | Todas las vistas |
| **Buscador de empresa** (autocomplete) | Busqueda, Comparacion |

---

## 9. Limitaciones Conocidas del Sistema

| Limitacion | Detalle | Impacto |
|---|---|---|
| **Cobertura limitada** | Solo 61 PYMES de Ibague con datos en SIREM. No todas las PYMES de Ibague estan en la base de datos | Los promedios sectoriales no representan a todas las PYMES de la ciudad |
| **Datos historicos estaticos** | Los datos van de 2016 a 2024 y no se actualizan automaticamente | El sistema no tiene datos en tiempo real; se actualiza cuando Supersociedades publica nuevos datos |
| **Z-Score calibrado para EE.UU.** | El modelo Z de Altman fue calibrado con empresas estadounidenses | Los umbrales pueden no ser exactos para el contexto colombiano, aunque la formula es ampliamente usada en Latinoamerica |
| **Dataset pequeno para ML** | 359 observaciones empresa-anio pueden no ser suficientes para modelos complejos | Se usan modelos simples con regularizacion fuerte; las predicciones tienen un margen de incertidumbre |
| **Completitud variable** | No todos los conceptos financieros estan disponibles para todas las empresas | Algunos indicadores no se pueden calcular para ciertas empresas (aparecen como "No disponible") |
| **Solo NIIF Pymes** | Excluye empresas que reportan bajo NIIF Plenas (Grupo 1) | No aplica para grandes empresas |

---

## 10. Resumen de Capacidades

### Lo que el sistema SI hace

- Consultar los estados financieros de 61 PYMES de Ibague (2016-2024)
- Calcular 18 indicadores financieros automaticamente
- Clasificar el riesgo financiero usando ML (bajo/medio/alto)
- Mostrar la evolucion temporal de indicadores en graficos interactivos
- Comparar una empresa contra el promedio del sector
- Comparar dos empresas entre si
- Evaluar empresas externas ingresando datos manualmente
- Generar informes descargables en PDF
- Mostrar un dashboard panoramico del sector PYME de Ibague
- Explicar cada indicador en lenguaje simple con semaforos visuales

### Lo que el sistema NO hace

- No reemplaza el juicio de un analista financiero profesional
- No predice el futuro con certeza (el modelo tiene limitaciones)
- No se conecta en tiempo real a bases de datos de Supersociedades
- No incluye todas las PYMES de Ibague, solo las registradas en SIREM
- No realiza proyecciones financieras ni simulaciones de escenarios
- No almacena datos sensibles de usuarios (solo datos financieros publicos del SIREM)

---

*Documento de prototipo funcional - Version 1.0*
*Proyecto de Tesis - Universidad del Tolima*
*Febrero 2026*
