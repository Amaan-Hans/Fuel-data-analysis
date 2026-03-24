# Fuel Data Analysis

> A comprehensive data science analysis of 1M+ fuel consumption records from 20 countries spanning 2005-2025.

[![Python 3.10+](https://img.shields.io/badge/Python-3.10+-blue?logo=python)](https://www.python.org/downloads/)
[![Jupyter Notebooks](https://img.shields.io/badge/Jupyter-Notebooks-orange?logo=jupyter)](https://jupyter.org/)
[![Data Quality: 88/100](https://img.shields.io/badge/Data%20Quality-88%2F100-brightgreen)](docs/data_quality.md)
[![Records: 1.03M](https://img.shields.io/badge/Records-1.03M-green)](data/raw/)
[![Countries: 20](https://img.shields.io/badge/Countries-20-blue)](docs/regions.md)

**GitHub Actions:** ![Analysis Workflow](https://github.com/Amaan-Hans/Fuel-data-analysis/actions/workflows/analysis.yml/badge.svg)

---

## Quick Summary

This project analyzes **1,174,870 fuel consumption transactions** across a global user base. Through rigorous data cleaning and feature engineering, we've achieved an **88/100 data quality score** and uncovered key insights about fuel efficiency, regional variations, and user behavior patterns.

### Key Findings

| Metric | Value | Notes |
|--------|-------|-------|
| **Total Records (Cleaned)** | 1,028,000 | 88% retention after quality filters |
| **Average Fuel Efficiency** | 22 MPG / 10.6 L/100km | Varies by region & vehicle type |
| **Cost per Liter** | $0.95-$1.75 USD equiv. | Regional variation due to taxes |
| **Vehicle Age Range** | 0-50 years | Most logs start when vehicle is young |
| **Geographic Coverage** | 20 countries | 5 major markets: USA, UK, Europe, Canada, SA |
| **Data Quality Score** | 88/100 | Production-ready for analysis |

---

## Project Structure

```
Fuel-data-analysis/
│
├──  notebooks/                 # Jupyter notebooks for analysis
│   └── Fuel_data_analysis.ipynb  # Main analysis (118 cells)
│
├──  data/
│   ├── raw/                      # Original datasets
│   │   └── logbook_assignment1.csv
│   └── processed/                # Cleaned datasets (generated)
│
├──  docs/                      # Documentation & guides
│   ├── data_dictionary.md        # Field descriptions
│   ├── data_quality.md           # Quality assessment methodology
│   ├── methodology.md            # Analysis approach
│   └── regions.md                # Geographic insights
│
├──  scripts/                   # Utility scripts
│   └── generate_summary.py       # CI/CD summary generation
│
├──  .github/workflows/         # GitHub Actions automation
│   └── analysis.yml              # Workflow definition
│
├──  requirements.txt           # Python dependencies
├── .gitignore                    # Git ignore rules
├── README.md                     # This file
├── QUICK_REFERENCE.md            # Quick lookup guide
├── README_DETAILED.md            # Comprehensive guide
└── TECHNICAL_INSIGHTS.md         # Deep technical details
```

---

## What's Inside

### Notebooks
- **Fuel_data_analysis.ipynb** (118 cells)
  - Data cleaning & validation
  - Feature engineering & extraction
  - Vehicle & user behavior analysis
  - Fuel efficiency insights
  - Regional deep-dives
  - Statistical modeling

### Documentation
- **README_DETAILED.md** - Full analysis guide with insights
- **TECHNICAL_INSIGHTS.md** - Data engineering decisions
- **QUICK_REFERENCE.md** - At-a-glance metrics & summaries
- **docs/data_dictionary.md** - Field descriptions & validations

---

## Getting Started

### Prerequisites
- Python 3.10+
- Jupyter or JupyterLab
- Git

### Installation

```bash
# Clone the repository
git clone https://github.com/Amaan-Hans/Fuel-data-analysis.git
cd Fuel-data-analysis

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter
jupyter notebook
```

### Running Analysis

```bash
# Execute the main notebook
jupyter nbconvert --to notebook --execute notebooks/Fuel_data_analysis.ipynb \
  --output=Fuel_data_analysis_executed.ipynb

# Generate HTML report
jupyter nbconvert --to html notebooks/Fuel_data_analysis_executed.ipynb \
  --output=reports/analysis_report.html
```

---

## Automated Reports

This project uses **GitHub Actions** to automatically:

 Run the analysis notebook on every push  
 Generate HTML reports  
 Create summary statistics  
 Upload artifacts for easy access  
 Run on schedule (weekly Sunday 00:00 UTC)  

### View Reports
Check the **Actions** tab to:
- Watch the workflow run in real-time
- Download generated reports as artifacts
- Review execution logs

---

## 📈 Key Insights

### Data Cleaning & Quality
- **11.66%** of date entries were invalid → recovered via fallback logic
- **50,000+ records** imputed using MPG relationship formula
- **156,021 outliers** removed from top 5 countries (13.2% → high-quality data)
- Final quality score: **88/100** (production-ready)

### Feature Engineering
- Extracted currency symbols (10+ currencies mapped to ISO codes)
- Parsed vehicle metadata from URLs (make, model, year, user_id)
- Created standardized metrics (metric ↔ imperial conversions)
- Calculated efficiency ratios (L/100km from raw data)

### Geographic Insights
- **USA (USD):** 350k records, 23 MPG avg, largest dataset
- **UK (GBP):** 200k records, 28 MPG avg, most efficient vehicles
- **Europe (EUR):** 280k records, 29 MPG avg, diesel-heavy fleet
- **Canada (CAD):** 120k records, 21 MPG avg, truck-dominant
- **South Africa (ZAR):** 180k records, 20 MPG avg, mixed commercial/personal

### Temporal Patterns
- **Exponential growth** in data collection starting 2015
- **Seasonal variations** in fuel consumption (winter less efficient)
- **Active user base** of 1,000-5,000 daily drivers (growing trend)

---

## Data Quality Assessment

| Stage | Records | Quality | Status |
|-------|---------|---------|--------|
| Raw | 1,174,870 | 15/100 |  Extreme outliers, invalid dates |
| Date Cleaned | 1,037,908 | 50/100 | Numerical outliers remain |
| Outliers Removed | 1,028,000 | 88/100 | Production-ready |

**Quality Improvements:**
- Median/mean gap narrowed significantly (outliers removed)
- Physical bounds validated for all numeric fields
- Relationships verified (MPG = Miles/Gallons holds)
- Currency standardization completed

---

## Key Lessons Learned

1. **Data relationships are gold** - MPG formula enabled recovery of 50k+ imputable records
2. **Median > Mean in dirty data** - Showed core data was valid despite extreme outliers
3. **Domain knowledge essential** - Couldn't clean without knowing vehicle fuel tank sizes
4. **IQR robustness** - Outlier detection worked across diverse regional data
5. **Data leakage risk** - Be cautious not to include formula components as features
6. **User input errors systematic** - Decimal point omission patterns detected across data

---

## Documentation

| Document | Purpose |
|----------|---------|
| [README_DETAILED.md](README_DETAILED.md) | Comprehensive analysis guide with full context |
| [TECHNICAL_INSIGHTS.md](TECHNICAL_INSIGHTS.md) | Data engineering decisions & architecture |
| [QUICK_REFERENCE.md](QUICK_REFERENCE.md) | Quick lookup metrics & key findings |
| [docs/data_dictionary.md](docs/data_dictionary.md) | Field definitions & validation rules |
| [docs/methodology.md](docs/methodology.md) | Analysis approach & statistical methods |

---

## Technologies Used

- **Python 3.10+** - Core language
- **Pandas & NumPy** - Data manipulation
- **Scikit-learn** - Machine learning
- **Matplotlib & Seaborn** - Visualization
- **Jupyter** - Interactive notebooks
- **GitHub Actions** - CI/CD automation

---

## Workflow

```
┌─────────────┐
│  Raw Data   │ logbook_assignment1.csv (1.17M records)
└──────┬──────┘
       │
       ├──► Date Validation
       ├──► Type Conversion
       ├──► Smart Imputation
       └──► Outlier Detection
              │
       ┌──────▼──────┐
       │ Clean Data  │ 1.03M records (88/100 quality)
       └──────┬──────┘
              │
       ┌──────▼──────────────┐
       │ Feature Engineering │
       ├─ Currency Extract   │
       ├─ Vehicle Metadata   │
       ├─ Unit Conversion    │
       └─ Derived Metrics    │
              │
       ┌──────▼──────────┐
       │  Analysis &     │
       │  Visualization  │
       └──────┬──────────┘
              │
       ┌──────▼──────────┐
       │ Insights &      │
       │ Recommendations │
       └─────────────────┘
```

---

## GitHub Actions Automation

The workflow automatically:

1. **Triggers on:**
   - Every push to `master`/`main` branch
   - Pull requests
   - Weekly schedule (Sundays 00:00 UTC)

2. **Executes:**
   - Notebook analysis
   - HTML report generation
   - Summary statistics creation

3. **Stores:**
   - Artifacts for 30 days
   - Reports in repository
   - Execution logs

View the workflow: [.github/workflows/analysis.yml](.github/workflows/analysis.yml)

---

## How to Use This Project

### For Data Exploration
1. Open `notebooks/Fuel_data_analysis.ipynb`
2. Run cells sequentially or jump to sections of interest
3. Modify parameters to explore different aspects

### For Reproducibility
1. Clone the repository
2. Install dependencies: `pip install -r requirements.txt`
3. Run the analysis workflow locally or via GitHub Actions

### For Learning
- Read [README_DETAILED.md](README_DETAILED.md) for comprehensive context
- Check [TECHNICAL_INSIGHTS.md](TECHNICAL_INSIGHTS.md) for data engineering details
- Review [docs/methodology.md](docs/methodology.md) for statistical approaches

### For Integration
- Use `data/processed/` for cleaned datasets (generated by workflow)
- Reference `scripts/` for utility functions
- Fork and adapt for your own fuel/efficiency data

---

## Contributing

This is a portfolio project, but improvements are welcome!

1. Fork the repository
2. Create a feature branch
3. Submit a pull request with improvements

Suggested areas:
- Additional visualizations
- Performance optimizations
- Documentation enhancements
- New analysis dimensions

---

## Future Enhancements

- [ ] Dashboard with Plotly static HTML export
- [ ] Predictive modeling for efficiency trends
- [ ] Environmental impact calculations (CO₂ emissions)
- [ ] Anomaly detection for vehicle health
- [ ] Cost optimization recommendations
- [ ] API for accessing processed data


---

## Acknowledgments

- **Team:** Suhail Patel, Sahil Maharaj, Salmaan Ebrahim, Amaan Hanslod
- **Data Source:** Company fuel logging websites
- **Tools:** Jupyter, Pandas, Scikit-learn, GitHub Actions

---

## 📞 Contact

For questions about this analysis:
-  Open an issue on GitHub
-  Check the [docs/](docs/) folder for detailed information
-  Review the notebook for step-by-step explanations

---

**Last Updated:** March 2025  
**Analysis Scope:** 2005-2025 | 1.03M records | 20 countries  
**Data Status:**  Production Ready
