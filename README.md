# Sistema Web de Analisis Financiero para PYMES de Ibague

**Tesis de Grado**: "Diseno e Implementacion de un Sistema Web de Analisis Financiero con Modelo de Machine Learning para la Evaluacion de Riesgo en PYMES del Municipio de Ibague, Tolima"

## Descripcion

Sistema web que evalua la salud financiera y el nivel de riesgo de PYMES de Ibague (Colombia) a partir de datos oficiales del SIREM (Superintendencia de Sociedades). Combina indicadores financieros tradicionales con un modelo de Machine Learning para clasificar el riesgo empresarial.

## Fuentes de Datos

| Fuente | Descripcion | Tamano |
|--------|-------------|--------|
| **SIREM** | Estados financieros bajo NIIF (Supersociedades) | ~9.17 GB (4 CSVs) |
| **Camara de Comercio de Ibague** | Empresas activas, corte diciembre 2025 | 23.58 MB |

- **66 empresas** de Ibague identificadas en SIREM, **61 son NIIF Pymes**
- Cobertura temporal: **2016 - 2024**

## Estructura del Proyecto

```
Tesis/
├── dataset/                  # Datos fuente (no incluidos en el repo)
├── data/                     # Datos procesados (no incluidos en el repo)
├── documents/                # Documentos de la tesis (propuesta, guia)
├── etl_camara_comercio.ipynb # Notebook 1: ETL inicial y validacion JOIN
├── procesamiento_datos_financieros.ipynb # Notebook 2: JOIN + pivot
├── AVANCE_ETL.md             # Reporte de avance del ETL
├── DICCIONARIO_DATOS_ML.md   # Diccionario de datos e indicadores
├── PROTOTIPO_PRODUCTO.md     # Especificacion del producto web
├── requirements.txt          # Dependencias Python
└── .gitignore
```

> Los directorios `dataset/` y `data/` contienen archivos grandes y estan excluidos del repositorio via `.gitignore`.

## Indicadores Financieros

El sistema calcula 18 indicadores agrupados en 4 categorias:

- **Liquidez**: Razon corriente, prueba acida, capital de trabajo
- **Solvencia**: Endeudamiento total, apalancamiento, cobertura de intereses
- **Rentabilidad**: Margen bruto, margen operacional, margen neto, ROA, ROE
- **Actividad**: Rotacion de cartera, rotacion de inventarios, rotacion de activos

## Tecnologias

### Fase actual (Analisis de datos)
- Python 3.x
- pandas, numpy, matplotlib, seaborn
- scikit-learn
- Jupyter Notebook

### Fase siguiente (Sistema web)
- Frontend: React
- Backend: Node.js
- Base de datos: PostgreSQL

## Instalacion

```bash
# Clonar el repositorio
git clone https://github.com/IngRetrius/Tesis.git
cd Tesis

# Crear y activar entorno virtual
python -m venv venv
.\venv\Scripts\activate  # Windows

# Instalar dependencias
pip install -r requirements.txt

# Ejecutar Jupyter
jupyter notebook
```

## Autor

Proyecto de tesis - Ingenieria de Sistemas, Ibague, Tolima, Colombia.
