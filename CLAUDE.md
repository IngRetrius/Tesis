# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Thesis project: **Sistema Web de Analisis Financiero para PYMES** (Web-based Financial Analysis System for SMEs). The goal is to build a web system that analyzes Colombian SME financial data using official SIREM datasets from the Superintendencia de Sociedades.

The project is in its early/data-exploration phase. Currently contains a single Jupyter notebook for exploratory data analysis and documentation about the datasets.

## Environment Setup

```bash
# Activate virtual environment (Windows)
.\venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter
jupyter notebook
```

Python virtual environment is at `venv/`. Key dependencies: pandas 2.2.0, numpy 1.26.3, matplotlib 3.8.2, seaborn 0.13.1, scikit-learn 1.4.0.

## Data Architecture

### Source: SIREM Datasets (~9.17 GB total)

Four CSV files in `dataset/`, all sharing the same 11-column **vertical/long format** (one row per concept-value pair, NOT one row per company):

| Column | Purpose |
|--------|---------|
| `NIT` | Company tax ID (may contain commas like `"900,152,104"` - must be cleaned) |
| `PUNTO_ENTRADA` | Company type: "NIIF Pymes" (SMEs, thesis focus) vs "NIIF Plenas" (large companies) |
| `TAXONOMIA` | Fiscal year cutoff (2015-2023+) |
| `CONCEPTO` | Financial account name (e.g., "Activos corrientes totales") |
| `PERIODO` | Reporting period |
| `VALOR` | Numeric value (but stored as text in Caratula dataset) |

### The four datasets:

1. **Caratula** (2.07 GB) - Company metadata. `VALOR` column is mixed type (text + numbers).
2. **Estado de Situacion Financiera** (4.10 GB) - Balance sheet. 85 unique account concepts. `VALOR` is float64.
3. **Estado de Resultado Integral** (1.57 GB) - Income statement. 28 unique account concepts. `VALOR` is float64.
4. **Estado de Flujo de Efectivo** (1.43 GB) - Cash flow statement. 59 unique account concepts. `VALOR` is int64.

### Critical data handling notes

- **Files are large**: use chunked reading (`chunksize=` in `pd.read_csv`) when memory is a concern. Load full datasets for production analysis
- **Always use `low_memory=False`** when reading these CSVs
- **Encoding**: try UTF-8 first, fall back to Latin-1. Some CONCEPTO values have encoding artifacts (e.g., `�` for accented characters)
- **Pivoting required**: data is vertical (concept-value rows). To analyze a single company, filter by NIT then pivot CONCEPTO into columns
- **Filter by `PUNTO_ENTRADA`** containing "Pymes" to isolate SME data (thesis scope)
- The concept names for "Gastos de administracion" appear with encoding issues - search case-insensitively and account for encoding variants

### Key financial concepts mapping (for indicator calculations)

**From Situacion Financiera:** "Activos corrientes totales", "Pasivos corrientes totales", "Total pasivos", "Patrimonio total", "Inventarios corrientes", "Efectivo y equivalentes al efectivo", "Cuentas comerciales por cobrar y otras cuentas por cobrar corrientes"

**From Resultado Integral:** "Ingresos de actividades ordinarias", "Costo de ventas", "Ganancia bruta", "Gastos de ventas", "Ganancia (perdida) por actividades de operacion", "Ganancia (perdida)", "Costos financieros"

## Reference Documentation

See `DATASETS_SIREM.md` for full dataset documentation including all financial indicator formulas (liquidity, solvency, profitability, activity ratios).
