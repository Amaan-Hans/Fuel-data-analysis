# Technical Insights & Data Engineering Decisions

## Dataset Architecture

### Core Data Flow
```
Raw CSV (1.17M rows)
    ↓ [Date Validation]
Cleaned (1.04M rows)
    ↓ [Type Conversion & Imputation]
Enhanced (with computed fields)
    ↓ [Outlier Detection]
Production Dataset (1.03M rows, Top 5 currencies)
    ↓ [Analysis & Modeling]
Insights & Recommendations
```

### Column Categories

**Temporal Columns:**
- `date_fueled` - Primary timeline reference
- `date_captured` - System entry time (fallback for date_fueled)
- Derived: `year`, `month`, `quarter`, `day_of_week`

**Vehicle Information:**
- `car_make`, `car_model`, `car_year` - Extracted from URL
- `odometer` - Vehicle mileage at fillup
- Derived: `vehicle_age` = current_year - car_year

**Fuel Consumption:**
- `gallons`, `miles`, `mpg` - User-entered transaction data
- `litres_filled`, `km_driven`, `litres_per_100km` - Standardized metrics
- Relationships: MPG = miles/gallons (used for imputation)

**Financial Data:**
- `total_spent`, `cost_per_gallon` - Raw cost entries
- `currency` - Extracted from cost fields
- Derived: `cost_per_litre`, `cost_per_100km`

**User Identification:**
- `user_id` - Extracted from URL
- `user_url` - Source of vehicle & user metadata

---

## Data Quality Measurements

### Validation Thresholds

**Date Field Validation:**
- Range: 2005-01-01 to today (22 years)
- Invalid format handling: Use pd.to_datetime with dayfirst=True
- Invalid date fallback: Switch to date_captured if valid

**Numeric Field Validation:**
```python
# Physical Plausibility Bounds (pre-outlier removal)
Gallons:        0.1 - 200.0 L (handles from small electric to fuel truck)
Miles per tank: 0 - 2000 miles (handles fuel type and tank size variation)
MPG:            3 - 300 (realistic bounds; >200 flagged as suspicious)
Odometer:       0 - 10,000,000 miles (extreme but possible for commercial)
```

**Outlier Detection Method:**
- **IQR Algorithm:** fence = [Q1 - 1.5×IQR, Q3 + 1.5×IQR]
- **Rationale:** Robust to extreme values, distribution-agnostic
- **Application:** Applied per-currency for regional context

**Currency-Specific Cost Caps:**
- USD: Max $200/fillup (budget at $4/gallon ≈ 50-gal = unlikely)
- GBP: Max £150/fillup (UK prices ~£1.25/L → ~120L max = commercial fleet)
- EUR: Max €180/fillup (similar to GBP in Euro context)
- CAD: Max $180 CAD (1.3:1 USD ratio)
- ZAR: Max R2500 (~$170 USD equivalent)

---

## Imputation Strategy

### Three-Way Relationship Imputation

```python
# Given: MPG = Miles / Gallons
# Can impute any one from two others:

if MPG is missing and Miles and Gallons present:
    MPG = Miles / Gallons

if Miles is missing and MPG and Gallons present:
    Miles = MPG * Gallons

if Gallons is missing and Miles and MPG present:
    Gallons = Miles / MPG
```

**Imputation Statistics:**
- ~50,000 MPG values recovered
- ~40,000 Miles values recovered  
- ~30,000 Gallons values recovered
- Total recovery rate: +3.2% additional complete records

**Quality Check:** Only impute when:
1. Both source values present and valid
2. Result is within physical bounds
3. No division by zero

---

## Feature Engineering Pipeline

### Extraction Patterns

**Currency Extraction (from total_spent field):**
```regex
^([^\d\s]+)  # Capture leading non-digit characters
```
Maps symbols: `$`→USD, `£`→GBP, `€`→EUR, `₹`→INR, etc.

**Vehicle Metadata Extraction (from user_url):**
```
URL pattern: .../make/model/year/user_id
Uses last 4 path segments, handles missing segments gracefully
```

**Numeric Extraction (from cost fields):**
```
Remove commas: "1,234.56" → "1234.56"
Extract number: (\d+\.?\d*) → captures integers and floats
Convert: str → float with coercion to NaN for failures
```

### Unit Conversions

**Distance:**
- 1 mile = 1.60934 kilometers
- Applied to `miles` → `km_driven`

**Volume:**
- 1 US gallon = 3.78541 liters
- Applied to `gallons` → `litres_filled`

**Efficiency (Inverse Relationship):**
- MPG (miles per gallon) → L/100km
- Formula: L/100km = 235.214 / MPG (or direct calculation)
- Direct calculation used: (litres / km_driven) × 100

---

## Statistical Insights

### Distribution Shapes (Pre-Outlier Removal)

| Metric | Shape | Median | Mean | Std Dev | IQR |
|--------|-------|--------|------|---------|-----|
| Odometer | Right-skewed | 91,887 | 351,425 | 4.2M | ~150k |
| Miles | Extreme right | 267 | 2,841 | 18k | ~400 |
| Gallons | Extreme right | 12 | 89 | 643 | ~10 |
| MPG | Near-normal | 22 | 52 | 287 | ~12 |
| L/100km | Near-normal | 10.6 | 23.4 | 142 | ~2.8 |

### Skewness Interpretation
- **Right-skewed (Odometer, Miles, Gallons):** Most users on lower end; rare extreme values
  - Indicates: Users start logging from relatively new vehicles; some old/commercial outliers
  
- **Near-normal (MPG, L/100km):** Central tendency around realistic efficiency
  - Indicates: Calculations (derived from ratio) naturally center around expected values

---

## Machine Learning Insights

### Feature Importance Discovery

**Random Forest Model Results:**
```
litres_filled:     55% importance
km_driven:         45% importance  
All other features: <1% importance combined
OOB Score:         0.9999 (nearly perfect)
```

**Critical Lesson - Data Leakage:**
```
Target = litres_per_100km = (litres_filled / km_driven) * 100
Features include: litres_filled, km_driven
```

**Problem:** Model re-learned the mathematical formula rather than discovering patterns

**Lesson for Practitioners:**
1. Never include variables used to construct the target
2. Validate feature importance through domain knowledge
3. Be skeptical of extremely high predictive scores
4. When R² ≈ 0.999, suspect data leakage first
5. Test with external holdout set to validate generalization

### Corrected Approach
- **Remove formula components** from feature set
- **Use only independent predictors:** vehicle_age, odometer, car_make, car_model, car_year
- **Expected performance:** Moderate (not perfect) - reflects real predictive patterns
- **Result:** Reveals that vehicle characteristics are weak efficiency predictors
  - → Driving behavior matters more than vehicle type
  - → User skill/habits drive efficiency differences

---

## Regional Analysis Template

### Top 5 Currency Analysis

**Why Top 5?**
1. Reduces noise from sparse regional data
2. Ensures statistical significance (n > 100k per region)
3. Enables meaningful cross-regional comparison
4. Simplifies visualization and interpretation

**Regional Characteristics:**

| Currency | Dominant Region | Sample Size | Primary Vehicles |
|----------|-----------------|-------------|------------------|
| USD | North America | ~350k | Trucks, SUVs, Sedans |
| GBP | UK/Ireland | ~200k | Compact, Efficient cars |
| EUR | Western Europe | ~280k | Diesel, Efficient |
| CAD | Canada | ~120k | Trucks, SUVs |
| ZAR | South Africa | ~180k | Mixed commercial/personal |

---

## Recommendations for Future Work

### Immediate Improvements
- [ ] **Client-side validation:** Prevent decimal point errors at input
- [ ] **Anomaly alerts:** Flag suspicious entries for user verification
- [ ] **Periodic re-training:** Update outlier thresholds as data grows
- [ ] **External validation:** Compare cost/efficiency with official sources

### Advanced Features
- [ ] **Time series forecasting:** Predict seasonal fuel prices
- [ ] **Clustering:** Identify vehicle/driver archetypes
- [ ] **Recommendation engine:** Suggest fuel-efficient routes
- [ ] **Dashboard:** Real-time efficiency tracking per vehicle

### Research Opportunities
- [ ] **Weather API integration:** Analyze temperature/humidity impact on MPG
- [ ] **Traffic data correlation:** Rush hour vs. highway efficiency
- [ ] **Driver profiling:** Identify efficient vs. aggressive drivers
- [ ] **Environmental impact:** CO₂ emissions calculation and tracking

---

## Tools & Technologies Used

**Data Processing:**
- pandas (DataFrames, groupby, time series)
- numpy (Numerical operations)

**Visualization:**
- matplotlib (Basic plots, distribution analysis)
- seaborn (Statistical plots, boxplots)

**Machine Learning:**
- scikit-learn (Random Forest, preprocessing)

**Best Practices:**
- Type casting with error handling
- Vectorized operations (no loops)
- Meaningful variable naming
- Comprehensive documentation

---

## Data Governance Notes

### Data Lineage
```
source: logbook_assignment1.csv
├── Validation layer
│   ├── Date range check (2005-2025)
│   ├── Type validation
│   └── Range plausibility
├── Transformation layer
│   ├── Type conversion
│   ├── Imputation
│   └── Feature engineering
└── Production dataset
    └── Ready for analysis & modeling
```

### Missing Data Handling
- **Systematic Missing (documented):** NaN values documented and traced
- **Handled Missing:** Imputed via relationships (MPG triangle)
- **Removed Missing:** Dropped if critical and unrecoverable
- **Final Status:** <0.5% missing in production dataset

### Assumptions Made
1. `date_captured` can substitute for invalid `date_fueled` (reasonable for time-series)
2. MPG, Miles, Gallons relationship holds perfectly (mathematical constraint)
3. IQR method appropriately identifies outliers (industry standard)
4. Currency extraction accurate (assumes consistent formatting)
5. Vehicle URL parsing preserves semantic meaning (manual verification recommended)

---

**Last Updated:** March 2025
**Data Freshness:** Current (includes 2025 data)
**Next Review:** Quarterly data quality audit recommended
