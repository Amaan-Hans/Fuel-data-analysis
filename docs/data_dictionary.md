# 📋 Data Dictionary

Complete reference for all fields in the fuel consumption dataset.

## Core Temporal Fields

### `date_fueled` (DateTime)
- **Description:** Date when fuel was purchased/filled
- **Format:** YYYY-MM-DD (or various formats that pd.to_datetime can parse)
- **Validation:** 2005-01-01 to today
- **Missing:** 11.66% initially | Recovery method: Substituted with `date_captured` if valid
- **Notes:** Primary timeline reference for analysis

### `date_captured` (DateTime)
- **Description:** Date when entry was logged into system
- **Format:** YYYY-MM-DD
- **Validation:** Should be ≥ date_fueled (system entry after transaction)
- **Missing:** <1%
- **Notes:** Used as fallback when date_fueled invalid

---

## Vehicle Information Fields

### `odometer` (Float)
- **Description:** Vehicle's total mileage reading at time of refueling
- **Unit:** Miles (before conversion)
- **Range:** 0 - 10,000,000 (outliers >1M are likely commercial/logging errors)
- **Missing:** 2-3%
- **Imputation:** No direct imputation; used for age/vehicle health analysis
- **Notes:** Right-skewed distribution; most users start logging on newer vehicles

### `car_make` (String)
- **Description:** Vehicle manufacturer (extracted from URL)
- **Examples:** Toyota, Honda, Ford, BMW, Mercedes
- **Extraction:** Last 4th segment of user_url path
- **Missing:** ~5% (URL parsing failures)
- **Reliability:** High confidence (URL structure consistent)

### `car_model` (String)
- **Description:** Vehicle model (extracted from URL)
- **Examples:** Corolla, Civic, F-150, 3-Series
- **Extraction:** Last 3rd segment of user_url path
- **Missing:** ~5% (same URL parsing issues)
- **Reliability:** High confidence

### `car_year` (Integer)
- **Description:** Vehicle's manufacturing year (extracted from URL)
- **Format:** YYYY (e.g., 2015, 2020)
- **Range:** 1975-2025 (realistic vehicle ages)
- **Missing:** ~5%
- **Validation:** Coerced to numeric; invalid → NaN

---

## Fuel Consumption Fields

### `gallons` (Float) - *US Gallons*
- **Description:** Amount of fuel purchased
- **Unit:** US Gallons (before conversion to liters)
- **Range:** 0.1 - 200 (realistic: 5-20 typical)
- **Missing:** 3-7%
- **Imputation:** Calculated from miles/mpg if available
- **Formula:** gallons = miles / mpg
- **Type Issues:** Often stored as strings with commas (e.g., "15.5")

### `miles` (Float)
- **Description:** Distance driven since last fuel purchase
- **Unit:** Miles (US)
- **Range:** 0 - 2,000 (realistic for single tank)
- **Typical:** 200-800 (tank range varies by vehicle)
- **Missing:** 3-7%
- **Imputation:** Calculated from gallons × mpg if available
- **Formula:** miles = gallons × mpg
- **Outliers:** >40,000 miles = impossible, indicates data entry error

### `mpg` (Float) - *Miles Per Gallon*
- **Description:** Calculated fuel efficiency metric
- **Unit:** Miles per gallon (fuel consumed)
- **Range:** 3 - 300+ (realistic: 15-35 for most vehicles)
- **Typical:** 20-28 MPG (gasoline vehicles)
- **Missing:** 1-3%
- **Imputation:** Calculated from miles / gallons if available
- **Formula:** mpg = miles / gallons
- **Distribution:** Near-normal (best quality relative field)
- **Notes:** Highest fidelity of three metrics

---

## Cost Fields

### `total_spent` (String → Float)
- **Description:** Total amount spent on fuel (includes currency symbol)
- **Format:** String with currency symbol (e.g., "$45.50", "£35.00")
- **Extraction:** Regex to extract numeric value
- **Conversion:** String → Float (removes symbols, commas)
- **Missing:** ~2%
- **Validation:** Used but not primary (cost_per_gallon is more reliable)

### `cost_per_gallon` (String → Float)
- **Description:** Price per unit of fuel
- **Format:** String (e.g., "$3.45", "CHF 1.85")
- **Unit:** Currency per gallon
- **Extraction:** Regex to extract numeric portion
- **Conversion:** String → Float
- **Missing:** ~2%
- **Validation:** Must be positive; reasonable bounds by region

---

## Derived/Engineered Fields

### `currency` (String) - *Extracted Feature*
- **Description:** Currency code derived from cost fields
- **Extraction:** Regex pattern from `total_spent` or `cost_per_gallon`
- **Mapping:** Symbol → ISO code (e.g., $ → USD, £ → GBP)
- **Values:** USD, GBP, EUR, CAD, ZAR, etc.
- **Missing:** Marked as "Unknown" if extraction fails
- **Reliability:** 95%+ accurate for major symbols
- **Use Case:** Regional analysis, cost normalization

### `user_id` (String) - *Extracted Feature*
- **Description:** Unique identifier for data entry user (extracted from URL)
- **Format:** Last segment of user_url
- **Uniqueness:** Should be unique per user (but may have aliases)
- **Missing:** ~1%
- **Use Case:** User behavior analysis, tracking individual patterns

### `litres_filled` (Float) - *Converted Feature*
- **Description:** Fuel quantity in metric liters
- **Calculation:** gallons × 3.78541
- **Unit:** Liters
- **Purpose:** Standardization for international analysis
- **Precision:** Preserves decimal places from gallons conversion

### `km_driven` (Float) - *Converted Feature*
- **Description:** Distance in kilometers
- **Calculation:** miles × 1.60934
- **Unit:** Kilometers
- **Purpose:** Metric standardization for non-US audiences
- **Precision:** Preserves accuracy from miles conversion

### `litres_per_100km` (Float) - *Efficiency Metric*
- **Description:** Standardized fuel efficiency (metric standard)
- **Calculation:** (litres_filled / km_driven) × 100
- **Unit:** Liters per 100 kilometers
- **Interpretation:** Lower is better (more efficient)
- **Typical Range:** 5-25 L/100km (after outlier removal)
- **Relationship to MPG:** L/100km ≈ 235.214 / MPG

### `vehicle_age` (Integer) - *Derived Feature*
- **Description:** Age of vehicle at time of transaction
- **Calculation:** year(date_fueled) - car_year
- **Unit:** Years
- **Validation:** Filtered to 0-50 (negative ages removed as errors)
- **Use Case:** Efficiency trends by vehicle age

### `cost_per_litre` (Float) - *Derived Feature*
- **Description:** Regional fuel pricing in currency per liter
- **Calculation:** total_spent_float / litres_filled
- **Unit:** Currency per liter
- **Purpose:** Standardize fuel costs across regions
- **Validation:** Reasonable bounds by currency and region

---

## Data Quality Flags

### Cleaning Applied

| Issue | Records Affected | Action | Result |
|-------|-----------------|--------|--------|
| Invalid dates | 136,962 (11.66%) | Fallback to date_captured or drop | 11.66% recovery rate |
| Missing numeric values | ~120,000 | Imputed using field relationships | 50k+ records recovered |
| Extreme outliers | 156,021 (top 5 countries) | IQR-based removal | 15.2% → 88/100 quality |
| Type conversion errors | ~50,000 | Coerced to numeric or NaN | <0.5% loss |

---

## Validation Rules by Field

| Field | Type | Range | Null % | Notes |
|-------|------|-------|--------|-------|
| date_fueled | DateTime | 2005-2025 | 11.66% | Primary timeline |
| odometer | Float | 0-10M | 2-3% | Right-skewed |
| gallons | Float | 0.1-200 | 3-7% | Typical: 5-20 |
| miles | Float | 0-2000 | 3-7% | Typical: 200-800 |
| mpg | Float | 3-300 | 1-3% | Calc field, higher quality |
| total_spent | Float | 0-500 | 2% | Currency-dependent |
| cost_per_gallon | Float | 0-10 | 2% | Extracted from string |
| currency | String | ISO codes | <1% | ~10 major currencies |
| car_make | String | Various | 5% | Extracted from URL |
| car_year | Integer | 1975-2025 | 5% | Extracted from URL |

---

## Imputation Logic

### Three-Way Relationship (MPG Triangle)

```python
# Given: MPG = Miles / Gallons
# Can impute missing values in triangle:

IF mpg IS NULL AND miles IS NOT NULL AND gallons IS NOT NULL:
    mpg = miles / gallons

IF miles IS NULL AND mpg IS NOT NULL AND gallons IS NOT NULL:
    miles = mpg * gallons

IF gallons IS NULL AND miles IS NOT NULL AND mpg IS NOT NULL:
    gallons = miles / mpg
```

**Conditions for Imputation:**
✅ Both source values present and valid  
✅ No division by zero  
✅ Result within physical bounds  

---

## Regional Notes

### Currency Distribution
- **USD:** ~350k records (largest market)
- **GBP:** ~200k records (UK/Ireland)
- **EUR:** ~280k records (Western Europe)
- **CAD:** ~120k records (Canada)
- **ZAR:** ~180k records (South Africa)

### Typical Values by Region

| Region | Avg MPG | Avg L/100km | Cost/Gallon | Vehicle Type |
|--------|---------|------------|-------------|--------------|
| USA | 23 | 10.2 | $3.20 | Trucks, SUVs |
| UK | 28 | 8.4 | £1.35 | Compact, Efficient |
| Europe | 29 | 8.1 | €1.28 | Diesel, Sedans |
| Canada | 21 | 11.2 | $1.15 | Trucks, mixed |
| S. Africa | 20 | 11.8 | R22.50/L | Mixed |

---

## Data Quality Metadata

- **Last Updated:** March 2025
- **Total Records:** 1,028,000 (cleaned)
- **Data Quality Score:** 88/100
- **Missing Values:** <0.5%
- **Outliers Removed:** ~156,000
- **Relationships Validated:** MPG = Miles/Gallons ✅
- **Geographic Coverage:** 20 countries
- **Time Span:** 2005-2025 (22 years)
