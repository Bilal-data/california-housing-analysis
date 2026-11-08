# California Housing Prices: Exploratory Analysis

A small exploratory data analysis on the California housing dataset bundled with Google Colab (`sample_data/california_housing_train.csv`). Done in Python (pandas, matplotlib) as part of my data-analyst learning path.

## Dataset
- 17,000 rows, 9 columns: `longitude`, `latitude`, `housing_median_age`, `total_rooms`, `total_bedrooms`, `population`, `households`, `median_income`, `median_house_value`.
- Target: `median_house_value` (price in dollars).

## Key findings

### 1. The price distribution is right-skewed
| Statistic | Value |
|---|---|
| Mean | 207,301 |
| Median | 180,400 |
| Std | 115,984 |
| Min / Max | 14,999 / 500,001 |

The mean is above the median by about 27,000: a few expensive areas pull it up.

### 2. The target is capped at 500,001
836 rows (4.9% of the data) have a price of 500,000 or 500,001. This is an artificial ceiling: any house above it is recorded as 500,001, so the real prices of these rows are unknown. A scatter plot shows it clearly as a flat horizontal line at the top.

### 3. Income is the strongest linear signal
Correlation with `median_house_value`:

| Column | Correlation |
|---|---|
| `median_income` | 0.69 |
| `total_rooms` | 0.13 |
| `housing_median_age` | 0.11 |
| `households` | 0.06 |
| `total_bedrooms` | 0.05 |
| `population` | -0.03 |
| `longitude` | -0.04 |
| `latitude` | -0.14 |

Location columns have weak linear correlation, but location is likely important in a non-linear way (for example, coast vs inland). Correlation only measures linear relationships.

### 4. Effect of removing the capped rows
| | With cap | Without cap |
|---|---|---|
| Rows | 17,000 | 16,164 |
| Correlation (income vs price) | 0.69 | 0.65 |
| Mean price | 207,301 | 192,163 |
| Median price | 180,400 | 173,800 |

The correlation dropped, but this does not mean the true relationship is weaker. Removing the top end shrinks the range of the data, and the capped rows hide the real high prices. The honest summary is: roughly 0.65 to 0.69, depending on how the cap is handled.

## Limitations
- Exploratory analysis followed by baseline and predictive models (see Model Comparison below).
- The cap means any model trained on this target will underestimate the highest prices.
- Correlation is not causation, and it misses non-linear relationships.
- One sample file of the dataset was used (the training split).

## Tools
Python, pandas, matplotlib, scikit-learn, Google Colab.

---

## ملخص بالعربي
- الداتا فيها 17,000 صف و9 أعمدة، والسعر الوسيط للمنطقة هو الهدف.
- المتوسط أكبر من الوسيط بحوالي 27 ألف، لأن البيوت الغالية بتشد المتوسط.
- اكتشفت **سقف مصطنع** عند 500,001 في 836 صف (4.9 في المئة).
- الدخل هو أقوى عامل: ارتباطه حوالي 0.69، ونزل لحوالي 0.65 بعد شيل الصفوف المسقوفة.
- بنينا نماذج تنبؤ: أفضلها `Random Forest` بمتوسط غلط حوالي 29,704 دولار (أحسن من النموذج الغبي بحوالي 61 بالمئة).    

## Model Comparison

Trained and tested on non-capped rows only (median_house_value < 500,000), 80/20 split, random_state=42.

| Model | MAE (USD) | vs. baseline |
|---|---|---|
| Baseline (always predict the mean) | 76,745 | - |
| Linear Regression (income only) | 55,906 | about 27 percent better |
| Linear Regression (8 features) | 45,355 | about 41 percent better |
| Random Forest (8 features) | 29,704 | about 61 percent better |

### Feature importance (Random Forest)

| Feature | Importance |
|---|---|
| median_income | 0.463 |
| longitude | 0.198 |
| latitude | 0.173 |
| housing_median_age | 0.058 |
| others (4 features) | each below 0.04 |

Location (latitude + longitude) accounts for about 37 percent, which explains why the linear model struggled: the effect of location is not a straight line.

### Limitations

- Rows at the artificial cap (500,001) were removed, so the model has not seen the most expensive houses and real-world error would be higher for them.
- A single train/test split was used, so the numbers can shift slightly with a different split.
- Feature importance shows what the model relied on, not what causes prices to change. The room, bedroom and household counts are correlated with each other, so their importance is shared between them.
- An error of about 16 percent of the average price is still too large for real decisions.
