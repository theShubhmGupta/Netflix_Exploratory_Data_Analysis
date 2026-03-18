# Netflix Content Catalogue — Exploratory Data Analysis

> Analysing 8,807 Netflix titles to uncover content distribution patterns, geographic trends, and audience rating behaviour using Python.

[![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-2.x-lightblue?logo=pandas)](https://pandas.pydata.org/)
[![Seaborn](https://img.shields.io/badge/Seaborn-Visualisation-teal)](https://seaborn.pydata.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)](https://jupyter.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Problem Context

Netflix's content catalogue has grown to 8,807 titles across 748 country entries, spanning 96 years of content (1925–2021). This project performs a structured EDA to identify patterns in content type, geography, ratings, and release timing.

> For business context and recommendations, see [`project_overview.md`](Netflix_Exploratory_Data_Analysis/project_overview.md) and [`business_reports/netflix_business_report.pdf`](Netflix_Exploratory_Data_Analysis/business_reports/netflix_business_report.pdf).

---

## Tech Stack

| Category | Tools |
|---|---|
| Language | Python 3.10 |
| Data Manipulation | Pandas, NumPy |
| Visualisation | Matplotlib, Seaborn, WordCloud |
| Environment | Jupyter Notebook / VS Code |
| Version Control | Git, GitHub |

---

## Data Understanding

### Dataset Structure

- **File:** `Netflix_Exploratory_Data_Analysis/data/raw/netflix_business_case.csv`
- **Shape:** 8,807 rows × 12 columns

| Column | Type | Description |
|---|---|---|
| `show_id` | object | Unique title identifier |
| `type` | object → category | Movie or TV Show |
| `title` | object | Title name |
| `director` | object | Director name(s) — high nulls |
| `cast` | object | Comma-separated actor list |
| `country` | object | Comma-separated country of production |
| `date_added` | object → datetime | Date added to Netflix |
| `release_year` | int64 | Original release year |
| `rating` | object → category | Content rating (TV-MA, PG, R, etc.) |
| `duration` | object | Mixed format: '90 min' or '2 Seasons' |
| `listed_in` | object | Comma-separated genre tags |
| `description` | object | Short synopsis |

### Data Quality Issues

| Column | Issue | Decision |
|---|---|---|
| `director` | 2,634 nulls (29.9%) | Not filled — excluded naturally by pandas `groupby` and `value_counts` |
| `cast` | 825 nulls (9.4%) | Not filled — dropped during `explode()` pre-processing |
| `country` | 831 nulls (9.4%) | Not filled — dropped during `explode()` pre-processing |
| `date_added` | 10 nulls (0.1%) | Converted to `NaT` via `pd.to_datetime(errors='coerce')` |
| `rating` | 4 nulls + 3 data entry errors | Nulls excluded via `dropna()` in group operations; `'66 min'`, `'74 min'`, `'84 min'` are misplaced duration values in the rating column |
| `duration` | 3 nulls + mixed format | Split into two derived columns — see Feature Engineering |

### Assumptions

- `release_year` is stored as an integer (year only). All date-gap calculations treat it as `YYYY-01-01` — actual gaps may be understated by up to 364 days.
- Content volume is used as a proxy for strategic intent — viewership data is not available in this dataset.
- No null filling was applied. Pandas handles null rows implicitly through `groupby`, `value_counts`, and `explode` operations — filling with placeholder strings (e.g. `'Unknown Director'`) would pollute frequency counts and ranking analyses.

---

## Methodology

### 1. Data Cleaning

```python
# Convert date_added to datetime — nulls become NaT automatically
df['date_added'] = pd.to_datetime(df['date_added'], format='mixed', errors='coerce')

# Convert low-cardinality columns to category dtype
# Done BEFORE any groupby operations — saves memory, enables category-aware methods
df['type']   = df['type'].astype('category')
df['rating'] = df['rating'].astype('category')
```

**Key decision:** Category conversion is done before null detection, not after. This avoids the `TypeError: Cannot setitem on a Categorical with a new category` that occurs when you try to `fillna()` on a category column with a value not already in its allowed set.

---

### 2. Feature Engineering

The `duration` column holds two incompatible formats in the same column — a design issue in the source data. It was split into two mutually exclusive numeric columns:

```python
# duration_minutes — populated only for Movies
df['duration_minutes'] = df.apply(
    lambda x: int(x['duration'].replace(' min', ''))
              if pd.notna(x['duration']) and 'min' in str(x['duration']) else np.nan, axis=1
)

# duration_seasons — populated only for TV Shows
df['duration_seasons'] = df.apply(
    lambda x: int(x['duration'].split(' ')[0])
              if pd.notna(x['duration']) and 'Season' in str(x['duration']) else np.nan, axis=1
)
```

**Important:** These two columns are **mutually exclusive by design** — a row is either a Movie or a TV Show, never both. Any analysis that drops rows based on one of these columns will silently remove an entire content type. The correct approach is `dropna(subset=['duration_minutes', 'duration_seasons'], how='all')` — only drop a row if **both** duration columns are null.

Date features extracted from `date_added`:

```python
df['month_added'] = df['date_added'].dt.month
df['month_name']  = df['date_added'].dt.strftime('%B')
df['year_added']  = df['date_added'].dt.year
df['week_added']  = df['date_added'].dt.isocalendar().week.astype('Int64')
```

---

### 3. Multi-Value Column Pre-processing (Un-nesting)

Three columns — `cast`, `country`, `listed_in` — store comma-separated values in a single cell. For accurate frequency analysis, each was exploded into a separate DataFrame:

```python
df_cast    = df.assign(cast=df['cast'].str.split(', ')).explode('cast')
df_country = df.assign(country=df['country'].str.split(', ')).explode('country')
df_genre   = df.assign(listed_in=df['listed_in'].str.split(', ')).explode('listed_in')

# Drop NaN rows produced by explode
df_cast    = df_cast.dropna(subset=['cast'])
df_country = df_country.dropna(subset=['country'])
df_genre   = df_genre.dropna(subset=['listed_in'])
```

**Why three separate DataFrames?**
Exploding all three on the same DataFrame creates a cartesian product. A title with 3 actors, 2 countries, and 4 genres would produce 24 rows instead of 1 — inflating every count downstream. Separate DataFrames ensure each dimension is counted independently and accurately.

---

### 4. Statistical Checks

- **Descriptive statistics:** `df.describe()` across numeric columns (`release_year`, `duration_minutes`, `duration_seasons`)
- **Missing value audit:** Count and percentage per column, sorted by severity
- **Outlier detection:** IQR method applied to `release_year`, `duration_minutes`, and `duration_seasons`
- **Correlation analysis:** Pearson correlation matrix on all numeric columns including engineered date features

```python
# IQR-based outlier detection
Q1, Q3 = df['release_year'].quantile([0.25, 0.75])
IQR = Q3 - Q1
outliers = df[(df['release_year'] < Q1 - 1.5*IQR) | (df['release_year'] > Q3 + 1.5*IQR)]
```

All identified outliers were **retained** — they represent valid content (pre-1960 classic films, stand-up specials, multi-season long-running shows) and their removal would distort distribution analysis.

---

### 5. Visualisation Strategy

Plots were chosen based on variable type and the specific question being answered:

| Question | Variable Type | Plot Chosen | Reason |
|---|---|---|---|
| How is release year distributed? | Continuous | Histogram + KDE | Show both shape and density |
| How long are movies? | Continuous | Histogram + KDE | Near-normal — both plots reinforce each other |
| How many seasons do TV Shows have? | Discrete | Countplot | Integer values — bar chart is more accurate than histogram |
| Does rating affect movie length? | Categorical vs Continuous | Boxplot | Shows median, spread, and outliers per group |
| How has content grown over time? | Categorical vs Time | Line plot | Trends are clearer than bars for time series |
| Which genres dominate? | High-cardinality text | Word Cloud | More readable than a 40-row bar chart |
| Are numeric variables correlated? | Multi-variable | Heatmap + Pairplot | Heatmap for magnitude; pairplot for per-type patterns |

---

## Key Analysis Areas

### Univariate Analysis
- Distribution of `release_year` — skewness, concentration period
- Distribution of `duration_minutes` for Movies — normality, outliers
- Distribution of `duration_seasons` for TV Shows — right-skew, dominant value
- Content addition volume by year — growth trajectory

### Bivariate Analysis
- `rating` vs `duration_minutes` — does audience rating correlate with film length?
- `type` vs `release_year` — are TV Shows a more recent catalogue addition than Movies?
- `country` vs `release_year` — do different production markets have different content age profiles?
- `type` vs `year_added` — how has the Movies-to-TV-Shows ratio shifted over time?
- `country` vs `type` — which markets produce more TV Shows vs Movies?
- `rating` vs `type` — how does the rating distribution differ between Movies and TV Shows?
- `month_name` vs `type` — is there a seasonal pattern in content additions?

### Correlation Analysis
- Pearson correlation across: `release_year`, `duration_minutes`, `duration_seasons`, `month_added`, `year_added`
- Pairplot with `type` as hue — to separate Movie and TV Show distributions visually

### Non-Graphical Analysis
- Value counts for: content type, rating, country, genre, director, actor, release year
- Unique attribute counts per column
- Top 10 rankings for countries, genres, directors, actors

---

## Notebook Breakdown

### `src/netflix_eda.ipynb`
The core analysis notebook. Runs end-to-end — data loading through outlier detection.

**Sections:**
1. Problem Statement
2. Import Libraries & Setup
3. Load Dataset + Basic Metrics
4. Shape, Data Types, Category Conversion
5. Missing Value Detection & Decision
6. Feature Engineering (datetime, duration split, date features)
7. Statistical Summary
8. Non-Graphical Analysis (value counts, un-nesting)
9. Visual Analysis — Univariate (Plots 01–04), Categorical Boxplots (Plots 05–07), Bivariate (Plots 08–12)
10. Correlation Heatmap (Plot 15) + Pairplot (Plot 16)
11. Outlier Check (Plots 13–14)
12. Insights 


Intended for non-technical stakeholders or anyone reviewing the project without running the code.

### `src/netflix_recommendations.ipynb`
A standalone notebook containing the 10 business recommendations. Each recommendation includes:
- Notebook reference (exact plot or section)
- Data finding
- Recommended action
- Expected outcome

---

## Visualisations

All 16 plots are saved automatically to `outputs/plots/` when `netflix_eda.py` is run.

| Plot | File | Type |
|---|---|---|
| Release Year Distribution | [`01_release_year_distribution.png`](outputs/plots/01_release_year_distribution.png) | Histogram + KDE |
| Movie Duration Distribution | [`02_movie_duration_distribution.png`](outputs/plots/02_movie_duration_distribution.png) | Histogram + KDE |
| TV Show Seasons | [`03_tvshow_seasons_countplot.png`](outputs/plots/03_tvshow_seasons_countplot.png) | Countplot |
| Content Added by Year | [`04_content_added_by_year.png`](outputs/plots/04_content_added_by_year.png) | Bar chart |
| Rating vs Movie Duration | [`05_boxplot_rating_vs_duration.png`](outputs/plots/05_boxplot_rating_vs_duration.png) | Boxplot |
| Type vs Release Year | [`06_boxplot_type_vs_releaseyear.png`](outputs/plots/06_boxplot_type_vs_releaseyear.png) | Boxplot |
| Countries vs Release Year | [`07_boxplot_country_vs_releaseyear.png`](outputs/plots/07_boxplot_country_vs_releaseyear.png) | Boxplot |
| Content Growth Over Time | [`08_bivariate_type_over_time.png`](outputs/plots/08_bivariate_type_over_time.png) | Line plot |
| Countries vs Content Type | [`09_bivariate_country_type.png`](outputs/plots/09_bivariate_country_type.png) | Grouped bar |
| Rating by Content Type | [`10_bivariate_rating_type.png`](outputs/plots/10_bivariate_rating_type.png) | Grouped bar |
| Monthly Content Addition | [`11_bivariate_monthly_addition.png`](outputs/plots/11_bivariate_monthly_addition.png) | Grouped bar |
| Genre Word Cloud | [`12_genre_wordcloud.png`](outputs/plots/12_genre_wordcloud.png) | Word Cloud |
| Outlier — Release Year | [`13_outlier_release_year.png`](outputs/plots/13_outlier_release_year.png) | Boxplot |
| Outlier — Movie Duration | [`14_outlier_movie_duration.png`](outputs/plots/14_outlier_movie_duration.png) | Boxplot |
| Correlation Heatmap | [`15_correlation_heatmap.png`](outputs/plots/15_correlation_heatmap.png) | Heatmap |
| Pairplot by Content Type | [`16_pairplot.png`](outputs/plots/16_pairplot.png) | Pairplot |

Plots are saved using:

```python
plt.savefig('outputs/plots/filename.png', dpi=150, bbox_inches='tight')
```

`bbox_inches='tight'` removes excess whitespace. `dpi=150` balances file size and quality.

---

## Project Structure

```
Netflix_Exploratory_Data_Analysis/
│
├── data/
│   └── raw/
│       └── netflix_business_case.csv       # Source dataset 
│
├── outputs/
│   └── plots/                             # 16 saved chart images (.png)
│
├── business_reports/
│   └── Netflix_Business_Report.pdf       
│
├── src/
│   └── netflix_eda.ipynb                  # full analysis
│   └── netflix_recommendations.ipynb      # recommendations                                       
│
├── .gitignore
├── requirements.txt
├── project_overview.md                    # Business-focused project summary
└── README.md                              
```

---

## How to Run

### 1. Clone the repository

```bash
git clone https://github.com/theShubhmGupta/Netflix_Exploratory_Data_Analysis.git
cd Netflix_Exploratory_Data_Analysis
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Place the dataset

Download `netflix_business_case.csv` and place it at:

```
data/raw/netflix_business_case.csv
```

### 4a. Run as Jupyter Notebook

```bash
jupyter notebook src/netflix_eda.ipynb
```

Run cells top to bottom using `Shift + Enter`. All plots render inline.

### 4b. Run as Python script

```bash
python src/netflix_eda.ipynb
```

Runs the full analysis from the project root. All 16 charts are saved to `outputs/plots/` automatically. Charts display one by one — close each window to proceed to the next.

> **Note:** Run from the project root directory, not from inside `src/`. The script uses relative paths (`data/raw/...` and `outputs/plots/...`) anchored to the project root.

---

## Reproducibility

| Item | Detail |
|---|---|
| Python version | 3.10 |
| Key library versions | See `requirements.txt` |
| Random seed | `random_state=42` used in pairplot sampling |
| Dataset | [netflix_business_case](data/raw/netflix_business_case.csv) |


The pairplot (Plot 16) uses a sample of 500 rows for performance:

```python
sample_df = filtered_df.sample(n=500, random_state=42)
```

`random_state=42` ensures the same 500 rows are selected every time — results are fully reproducible.

---

## Key Technical Learnings

### 1. Null handling strategy matters more than null filling
Filling nulls with placeholder strings (`'Unknown Director'`) introduces a new most-frequent value that pollutes `value_counts` rankings. The cleaner approach is to let pandas exclude nulls naturally — `groupby`, `value_counts`, and `explode` all handle this implicitly.

### 2. Category dtype conversion order is critical
Converting a column to `category` dtype locks its allowed values. If you then call `fillna()` with a new string, pandas throws a `TypeError`. Always fill nulls before converting to category — or skip filling entirely if nulls are handled implicitly.

### 3. Mutually exclusive columns require careful drop logic
`duration_minutes` and `duration_seasons` are NaN by design for the content type they don't apply to. Using `dropna(subset=['duration_minutes'])` silently drops all TV Shows. The correct approach is `dropna(how='all')` across both columns together — only drop a row if it has no duration information at all.

### 4. Exploding multi-value columns must stay in separate DataFrames
Exploding `cast`, `country`, and `listed_in` on the same DataFrame creates a cartesian product — one title with 3 actors and 4 genres becomes 12 rows, artificially inflating every count. Three separate DataFrames, one per dimension, is the correct approach.

### 5. Plot type selection should follow variable type, not preference
Countplot for discrete values (seasons), histogram + KDE for continuous values (duration), boxplot for categorical vs continuous comparisons. Using a histogram for season counts (which are integers 1–17) would misrepresent the data by implying continuity between discrete values.

---

## Author

**Shubham Gupta** — Aspiring Data Scientist

[![GitHub](https://img.shields.io/badge/GitHub-theShubhmGupta-black?logo=github)](https://github.com/theShubhmGupta)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-theshubhamguptaa-blue?logo=linkedin)](https://linkedin.com/in/theshubhamguptaa)
[![Tableau](https://img.shields.io/badge/Tableau-Public-blue?logo=tableau)](https://public.tableau.com/app/profile/shubham.gupta2025)

---

## License

This project is licensed under the MIT License. See [`LICENSE`](LICENSE) for details.
