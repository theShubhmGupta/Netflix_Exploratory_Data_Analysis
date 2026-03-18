# Netflix Content Strategy — Project Overview

> **Domain:** Media & Streaming | **Tool:** Python | **Dataset:** Netflix Shows (8,807 records, mid-2021)

---

## 1. Problem Statement

Netflix has over 200 million subscribers globally and a catalogue of 8,807 titles. As content growth slowed post-2019, the business question became:

> *Where should Netflix invest its content budget to maximise subscriber acquisition and minimise churn?*

This project analyses Netflix's content catalogue to uncover patterns in content type, geography, ratings, genre, and release timing — and translates those patterns into clear, data-backed business recommendations.

---

## 2. Dataset Overview

| Attribute | Detail |
|---|---|
| Source | [`data/raw/netflix_business_case.csv`](data/raw/netflix_business_case.csv) |
| Records | 8,807 titles |
| Columns | 12 (show_id, type, title, director, cast, country, date_added, release_year, rating, duration, listed_in, description) |
| Content Types | 6,131 Movies (70%) + 2,676 TV Shows (30%) |
| Release Year Range | 1925 – 2021 |
| Countries | 748 unique entries |
| Ratings | 17 categories |

---

## 3. Key Insights

- **TV Shows are growing faster than Movies** — Since 2015, TV Show additions have outpaced Movies in growth rate, signalling a platform shift towards serialised content.

- **60%+ of TV Shows have only 1 season** — The median season count is 1 (mean: 1.76). Netflix either cancels early or heavily acquires limited series — both create audience trust risk.

- **The US dominates but Asia is the real growth frontier** — The US contributes 42% of all titles. India is second (12%) but almost entirely Movies. South Korea and Japan punch above their weight in TV Shows, driven by K-dramas and anime.

- **Netflix is an adult-first platform with a family content gap** — TV-MA accounts for 36% of all titles. Family-friendly content (TV-Y through PG combined) is only ~10% of the catalogue — a structural retention risk for household plans.

- **January and July are peak content months — February and August are dead zones** — Content additions consistently spike in January and July, and dip in February and August. This creates predictable engagement gaps mid-quarter.

- **Licenced content arrives stale** — A ~0.5 correlation between `release_year` and `year_added` confirms licenced titles reach Netflix long after their original release. Originals land fresh on day one.

- **Niche genres are underserved relative to their retention value** — International Movies, Dramas, and Comedies dominate. Anime, stand-up comedy, and investigative documentaries are significantly underrepresented despite their high-engagement fan bases.

---

## 4. Business Recommendations

| Priority | Recommendation |
|---|---|
| High | Commit to 2–3 season guarantees for new TV Show commissions |
| High | Commission TV Show originals in South Korea and Japan |
| High | Grow family content to 20% of new acquisitions |
| High | Shift budget from licencing to Netflix originals |
| Medium | Expand India's output from Movies into TV Shows |
| Medium | Schedule flagship launches in January and July |
| Medium | Fill February and August with mid-tier content drops |
| Medium | Build quarterly content pipelines for anime, stand-up, docs |
| Low | Deploy mature content to acquire; family content to retain |
| Low | Accelerate acquisition from France, Spain, Mexico, Brazil |

> Full findings and references in [`business_report/netflix_business_report.pdf`](business_report/netflix_business_report.pdf).

---

## 5. Impact / Value

| Area | Value Delivered |
|---|---|
| **Subscriber Retention** | Identifying the 1-season cancellation pattern and family content gap directly addresses the two biggest structural churn drivers |
| **Acquisition Strategy** | Geographic analysis pinpoints where the next wave of subscribers will come from — Asia-Pacific and LatAm |
| **Content Investment** | Correlation analysis proves licenced content underperforms originals on freshness — a direct input for budget allocation decisions |
| **Release Timing** | Monthly pattern analysis gives the content calendar team a data-backed framework for scheduling decisions |
| **Genre Strategy** | Word cloud and genre distribution analysis identifies underserved high-retention niches that competitors have not fully captured |

---

## 6. Project Structure

```
Netflix_Exploratory_Data_Analysis/
│
├── data/
│   └── raw/
│       └── netflix_business_case.csv          # Original dataset 
│
│
├── outputs/
│   └── plots/                                 # All charts saved as .png files
│       ├── 01_release_year_distribution.png
│       ├── 02_movie_duration_distribution.png
│       ├── 03_tvshow_seasons_countplot.png
│       ├── 04_content_added_by_year.png
│       ├── 05_boxplot_rating_vs_duration.png
│       ├── 06_boxplot_type_vs_releaseyear.png
│       ├── 07_boxplot_country_vs_releaseyear.png
│       ├── 08_bivariate_type_over_time.png
│       ├── 09_bivariate_country_type.png
│       ├── 10_bivariate_rating_type.png
│       ├── 11_bivariate_monthly_addition.png
│       ├── 12_genre_wordcloud.png
│       ├── 13_outlier_release_year.png
│       ├── 14_outlier_movie_duration.png
│       ├── 15_correlation_heatmap.png
│       └── 16_pairplot.png
│
├── reports/        
│   └── netflix_business_report.pdf          
│
├── src/
│   └── netflix_eda.ipynb                         # Detailed Analysis file
│   └── netflix_recommendations.ipynb             # Recommendation file
│
├── .gitignore                                
├── requirements.txt                        
└── project_overview.md                   
```

### File Descriptions

| File | What it does |
|---|---|
| `netflix_eda.ipynb` | Core analysis notebook — data cleaning, 16 visualisations, statistical summary, outlier checks |
| `netflix_recommendations.ipynb` | Standalone recommendations file — each of the 10 recommendations referenced back to specific plots |
| `Netflix_Business_Report.docx` | Business report — executive summary, findings, SWOT, roadmap |
| `outputs/plots/` | 16 chart images used in the report and embeddable in the README |

---

## Tools Used

![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-EDA-lightblue?logo=pandas)
![Seaborn](https://img.shields.io/badge/Seaborn-Visualisation-teal)
![Matplotlib](https://img.shields.io/badge/Matplotlib-Plots-orange)
![WordCloud](https://img.shields.io/badge/WordCloud-Genre_Analysis-purple)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)
