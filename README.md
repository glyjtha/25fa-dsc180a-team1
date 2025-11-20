# Sepsis Multimorbidity Analysis

**DSC 180A Quarter 1 Project - Team 1**

Analysis of sepsis patients with multimorbidities using the MIMIC-III database, including latent class analysis (LCA) and network visualizations.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [Running the Analysis](#running-the-analysis)
- [Reproducibility](#reproducibility)
- [Development](#development)
- [Documentation](#documentation)

## Overview

This project analyzes sepsis patients with multimorbidities from the MIMIC-III database. The analysis includes:

- **Data Processing**: Extraction and processing of patient data with comorbidity classifications
- **Latent Class Analysis (LCA)**: Clustering of patients into subgroups based on multimorbidity patterns
- **Network Visualizations**: Network graphs showing relationships between comorbidities
- **Statistical Analysis**: Age-stratified analysis and mortality patterns

## Prerequisites

Before starting, ensure you have:

- **Git** installed
- **Pixi** package manager installed (see installation instructions below)
- **PostgreSQL database** with MIMIC-III data loaded
- **R** installed (for LCA analysis) - version 4.0 or higher recommended

### Installing Pixi

**macOS/Linux:**
```bash
curl -fsSL https://pixi.sh/install.sh | sh
```

**Windows (PowerShell - Run as Administrator):**
```powershell
iwr -useb https://pixi.sh/install.ps1 | iex
```

After installation, restart your terminal or run:
```bash
source ~/.bashrc  # or ~/.zshrc on macOS
```

**Verify installation:**
```bash
pixi --version
```

### Installing R (for LCA Analysis)

**macOS:**
```bash
brew install r
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get update
sudo apt-get install r-base
```

**Windows:**
Download and install from [CRAN](https://cran.r-project.org/bin/windows/base/)

**Required R packages:**
The R script will automatically install required packages, but you can pre-install them:
```r
install.packages(c("poLCA", "dplyr", "readr"))
```

## Installation

1. **Clone the repository:**
```bash
git clone <REPO_URL>
cd 25fa-dsc180a-team1
```

2. **Install dependencies:**
```bash
pixi install
```

This will install all Python dependencies defined in `pyproject.toml`, including:
- Database connectors (SQLAlchemy, psycopg)
- Data analysis libraries (pandas, scikit-learn)
- Visualization libraries (matplotlib, seaborn, networkx)
- Jupyter Lab for interactive analysis

3. **Verify installation:**
```bash
pixi run python --version  # Should show Python 3.11
pixi run pytest --version   # Verify pytest is available
```

## Configuration

### Environment Variables

Create a `.env` file in the project root with the following variables:

```bash
# Database Configuration
DATABASE_URL=postgresql://username:password@host:port/database_name

# Table Names (adjust if using custom table names)
ADMISSION_COMORBIDITY_TABLE=mimiciii.admission_comorbidity
TARGET_PATIENT=mimiciii.target_patients
MORBIDITY_COUNTS_TABLE=mimiciii.morbidity_counts
```

**Important Notes:**
- Replace `username`, `password`, `host`, `port`, and `database_name` with your actual database credentials
- The `DATABASE_URL` should point to your MIMIC-III database
- Table names should match your database schema (default assumes `mimiciii` schema)
- Never commit your `.env` file to version control

**Example `.env` file:**
```bash
DATABASE_URL=postgresql://mimic_user:secure_password@localhost:5432/mimic
ADMISSION_COMORBIDITY_TABLE=mimiciii.admission_comorbidity
TARGET_PATIENT=mimiciii.target_patients
MORBIDITY_COUNTS_TABLE=mimiciii.morbidity_counts
```

### Database Setup

Ensure your MIMIC-III database is properly set up with:
- All required MIMIC-III tables loaded
- Materialized views created (see `src/mimiciii_db/queries/` for query scripts)
- Proper schema permissions

## Project Structure

```
25fa-dsc180a-team1/
├── src/                          # Source code modules
│   ├── mimiciii_db/              # Database utilities and queries
│   │   ├── db.py                 # Database connection class
│   │   ├── config.py             # Database configuration
│   │   └── queries/              # SQL query modules
│   │       ├── filtered_patients.py
│   │       ├── elixhauser_quan.py
│   │       ├── morbidity_counts.py
│   │       ├── multimorbidity_by_age_bracket_1a.py
│   │       └── illness_score_queries/  # Severity score calculations
│   │           ├── 01_echo_data.py
│   │           ├── 11_sofa.py
│   │           ├── 12_oasis.py
│   │           └── ...
│   ├── mathmatical_analysis/     # Statistical analysis modules
│   │   ├── lca_clustering.py     # LCA clustering wrapper
│   │   ├── lca_model_fitting.R   # R script for LCA analysis
│   │   └── config.py             # Analysis configuration
│   └── visualizations/           # Visualization modules
│       ├── network_viz.py        # Network graph visualizations
│       ├── subgroup_network_viz.py
│       ├── subgroup_multimorbidity_bubble.py
│       └── config.py
├── notebooks/                    # Jupyter analysis notebooks
│   ├── 1a.ipynb                  # Age-stratified multimorbidity analysis
│   ├── 1b.ipynb                  # Additional analysis
│   ├── 1b_Full.ipynb             # Full analysis notebook
│   └── 4.ipynb                   # Additional visualizations
├── data/                         # Output data files (generated)
│   ├── lca_results_summary.csv
│   ├── lca_all_subgroups.csv
│   └── lca_best_model_stats.csv
├── assets/                       # Generated figures and outputs
│   ├── eda/                      # Exploratory data analysis plots
│   ├── fig_1/                    # Figure 1 outputs
│   ├── fig_2/                    # Figure 2 outputs
│   ├── fig_4/                    # Figure 4 outputs
│   ├── fig_5/                    # Figure 5 outputs
│   └── subgroup_multimorbidity_bubble.png
├── tests/                        # Test suite
│   ├── test_db_functional.py     # Database functionality tests
│   └── README.md
├── logs/                         # Log files (generated)
├── pyproject.toml                # Project configuration and dependencies
├── pixi.lock                     # Locked dependency versions
└── README.md                     # This file
```

## Running the Analysis

### Interactive Analysis with Jupyter

1. **Start Jupyter Lab:**
```bash
pixi run jupyter lab
```

This will open Jupyter Lab in your browser. Navigate to the `notebooks/` directory.

2. **Run notebooks in order:**
   - Start with `1a.ipynb` for initial data exploration
   - Continue with `1b.ipynb` or `1b_Full.ipynb` for detailed analysis
   - Run `4.ipynb` for additional visualizations

3. **Notebook execution:**
   - Each notebook should be run from top to bottom
   - Ensure database connection is established before running queries
   - Check that required data tables exist before executing queries

### Running Individual Components

#### Database Queries

Execute database queries using Python:

```bash
pixi shell
python -c "from mimiciii_db import DB; from mimiciii_db.config import db_url; db = DB.from_url(db_url()); print('Connected!')"
```

#### LCA Analysis

Run the Latent Class Analysis:

```bash
pixi shell
python -c "from mathmatical_analysis.lca_clustering import lca_analysis; results = lca_analysis(); print(results)"
```

Or from Python:
```python
from mathmatical_analysis.lca_clustering import lca_analysis

# Run LCA analysis (exports data, runs R script, loads results)
results = lca_analysis()

# Access results
print(results['results_summary'])      # Model comparison
print(results['subgroup_assignments']) # Patient assignments
print(results['best_model_stats'])     # Best model statistics
```

#### Visualizations

Generate network visualizations:

```bash
pixi shell
python -c "from visualizations.network_viz import *; # Run visualization functions"
```

### Available Pixi Tasks

Run development and testing tasks:

```bash
# Run tests
pixi run test

# Run tests with coverage
pixi run test-cov

# Lint code
pixi run lint

# Format code
pixi run fmt

# Enter interactive shell with environment activated
pixi shell
```

## Reproducibility

### Step-by-Step Reproduction Guide

To reproduce the complete analysis:

1. **Set up environment:**
```bash
pixi install
```

2. **Configure database:**
   - Create `.env` file with database credentials
   - Ensure MIMIC-III database is accessible
   - Verify required tables and views exist

3. **Run data preparation queries:**
   - Execute queries in `src/mimiciii_db/queries/` to create materialized views
   - Ensure all illness score queries are run in order (01-13)

4. **Execute notebooks in sequence:**
   - `notebooks/1a.ipynb` - Initial analysis
   - `notebooks/1b_Full.ipynb` - Full analysis pipeline
   - `notebooks/4.ipynb` - Visualizations

5. **Verify outputs:**
   - Check `data/` directory for CSV outputs
   - Check `assets/` directory for generated figures
   - Compare outputs with expected results

### Data Dependencies

The analysis requires the following data to be available in the database:

- **Patient demographics**: `mimiciii.patients`, `mimiciii.admissions`
- **Comorbidity data**: Elixhauser comorbidity classifications
- **Clinical scores**: SOFA, OASIS, SAPSII (generated by illness score queries)
- **Vital signs and lab values**: First-day ICU measurements

### Output Files

After running the analysis, you should have:

**Data files (`data/`):**
- `lca_results_summary.csv` - LCA model comparison results
- `lca_all_subgroups.csv` - Patient subgroup assignments
- `lca_best_model_stats.csv` - Best model statistics

**Visualizations (`assets/`):**
- `eda/` - Exploratory data analysis plots
- `fig_1/`, `fig_2/`, `fig_4/`, `fig_5/` - Publication figures
- Network visualizations for each subgroup

### Version Information

- **Python**: 3.11.*
- **R**: 4.0+ (for LCA analysis)
- **Key dependencies**: See `pyproject.toml` for complete list
- **Database**: PostgreSQL with MIMIC-III v1.4

## Development

### Adding New Queries

1. Create a new Python file in `src/mimiciii_db/queries/`
2. Define query functions that return SQL strings and parameters
3. Register queries using the `@db.register()` decorator
4. Document query purpose and output schema

### Adding New Visualizations

1. Create visualization functions in `src/visualizations/`
2. Use configuration from `visualizations/config.py` for database access
3. Save outputs to `assets/` directory with descriptive names
4. Document figure generation in notebook or script

### Testing

Run the test suite:

```bash
# Run all tests
pixi run test

# Run with coverage report
pixi run test-cov

# Run specific test file
pixi shell
pytest tests/test_db_functional.py -v
```

### Code Quality

```bash
# Check code style
pixi run lint

# Format code
pixi run fmt
```

## Documentation

### Additional Resources

- **Module Documentation**: See README files in each `src/` subdirectory
  - `src/mimiciii_db/README.md` - Database utilities documentation
  - `src/mimiciii_db/queries/README.md` - Query documentation
  - `src/mimiciii_db/queries/illness_score_queries/README.md` - Illness score queries
  - `tests/README.md` - Testing guide

### Troubleshooting

**Database Connection Issues:**
- Verify `DATABASE_URL` in `.env` is correct
- Check database is running and accessible
- Ensure network/firewall allows connection
- Verify user has necessary permissions

**R Script Execution Errors:**
- Ensure R is installed and in PATH
- Check required R packages are installed: `poLCA`, `dplyr`, `readr`
- Verify R script path is correct: `src/mathmatical_analysis/lca_model_fitting.R`

**Import Errors:**
- Ensure you're in the Pixi environment: `pixi shell`
- Verify package is installed: `pixi install`
- Check Python path includes `src/` directory

**Missing Data Tables:**
- Run database setup queries in `src/mimiciii_db/queries/`
- Verify materialized views are created
- Check table names match `.env` configuration

### Getting Help

For issues or questions:
1. Check the troubleshooting section above
2. Review module-specific README files
3. Check test files for usage examples
4. Contact the project maintainers

---

