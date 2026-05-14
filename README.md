# Kenya Lending Portfolio — End-to-End Data Analysis Project

**Tools used:** Microsoft Excel · Julius AI · Claude AI  
**Dataset size:** 1,506 loans (after cleaning)  
**Author:** Lilian Cheuno · [LinkedIn](https://www.linkedin.com/in/lilian-cheuno) 

---

## Project overview

This is a full end-to-end data analysis project on a Kenyan lending portfolio dataset, from raw, messy data, all the way to executive-level insights. The goal was to simulate the kind of work a data analyst would do for a credit manager or C-suite audience.

The project was done in three stages:

1. **Data profiling** — spotting what needs cleaning before touching the data
2. **Data cleaning** — fixing issues in Excel using formulas, with every decision documented
3. **Analysis** — pulling strategic insights framed for business decision-makers

---

## Files in this repository

| File | Description |
|---|---|
| `KenyaLending_DirtyDataset.xlsx` | Original raw dataset with all issues intact |
| `KenyaLending_Cleaned.xlsx` | Fully cleaned dataset ready for analysis |

---

## Stage 1 — Data profiling

Before cleaning anything, I documented every issue found in the raw dataset on a separate sheet. Here is what I found:

| Issue | Count |
|---|---|
| Blank Loan IDs (primary key) | 10 |
| Impossible ages (below 18 or above 100) | 10 |
| Negative annual incomes | 14 |
| Blank annual income | 49 |
| Blank credit scores | 55 |
| Blank gender | 38 |
| Blank employment status | 31 |
| Blank loan purpose | 58 |
| Mixed date formats | 60 |
| Invalid credit score (-50, outside 0–999 range) | 1 |

---

## Stage 2 — Data cleaning (Excel)

All cleaning was done in Excel using formulas. A helper column was created for each issue so the original data was never overwritten. Every decision was documented with the business reason behind it.

### Decisions made

| Issue | Decision | Reason |
|---|---|---|
| Blank Loan IDs | Deleted all 10 rows | Primary key — records cannot be identified or analysed without it |
| Impossible ages (below 18, above 100) | Replaced with median of valid ages only | Biologically implausible values; median excludes the outliers being corrected |
| Negative annual incomes | Replaced with median of valid (positive) annual income | Income cannot logically be negative |
| Blank annual income | Imputed with median of valid (positive) incomes | Median is robust to outlier distortion in skewed income data |
| Blank credit scores | Imputed with median of valid scores (0–999) | Median avoids distortion from extreme values |
| Invalid credit score (-50) | Replaced with median | Outside the valid Kenyan CRB range of 0–999; treated as missing |
| Inconsistent gender casing | Standardised to Female / Male | Inconsistent casing causes wrong groupings in analysis |
| Blank gender | Labelled "Unknown" | Cannot infer gender; imputation would distort gender distribution |
| Blank employment status | Labelled "Unknown" | Critical creditworthiness field; wrong imputation distorts results |
| Blank loan purpose | Labelled "Unknown" | High-cardinality field; safe imputation not possible |
| Mixed date formats | Standardised to YYYY-MM-DD | Ensures consistent sorting, filtering, and cross-tool compatibility |

### Key formulas used

**Flagging blank Loan IDs:**
```excel
=IF(A2="","MISSING","OK")
```

**Replacing outlier ages with median of valid ages:**
```excel
=IF(OR(C2<18,C2>100),$K$1,C2)
```
Where `$K$1` = `=MEDIAN(IF((C2:C1517>=18)*(C2:C1517<=100),C2:C1517))` *(array formula — Ctrl+Shift+Enter)*

**Standardising gender:**
```excel
=IF(D2="","Unknown",IF(OR(D2="F",D2="FEMALE",D2="Female"),"Female",IF(OR(D2="M",D2="MALE",D2="Male"),"Male","Other")))
```

**Standardising mixed date formats:**
```excel
=IFERROR(IF(ISNUMBER(H2),H2,DATEVALUE(H2)),DATE(RIGHT(H2,4),MID(H2,4,2),LEFT(H2,2)))
```

**Imputing credit score (excluding out-of-range values):**
```excel
=IF(OR(I2="",I2<0,I2>999),$L$1,I2)
```
Where `$L$1` = `=MEDIAN(IF((I2:I1517>=0)*(I2:I1517<=999),I2:I1517))` *(array formula — Ctrl+Shift+Enter)*

---

## Stage 3 — Analysis (insights for C-suite)

### 1. Portfolio health

The portfolio is in a critical state — only 32.4% of loans are healthy.

| Bucket | Loans | Percentage |
|---|---|---|
| Healthy (Current + Fully Paid) | 488 | 32.4% |
| At Risk (Late 31–60 days + Late 61–90 days) | 499 | 33.1% |
| Non-Performing (Default + Charged Off) | 519 | 34.5% |

> Nearly 68% of the portfolio is either at risk or already non-performing.

---

### 2. Default rate by county

| County | Total Loans | Default Rate |
|---|---|---|
| Meru | 138 | 37.0% |
| Thika | 148 | 36.5% |
| Kakamega | 143 | 36.4% |
| Eldoret | 142 | 35.9% |
| Nairobi | 154 | 35.1% |
| Kisumu | 149 | 34.9% |
| Nyeri | 153 | 33.3% |
| Machakos | 132 | 33.3% |
| Mombasa | 161 | 32.3% |
| Nakuru | 137 | 28.5% |

> Meru, Thika, and Kakamega carry the highest geographic risk. Nakuru has the lowest default rate among all counties.

---

### 3. Credit score vs loan performance

| Loan status | Avg credit score |
|---|---|
| Late (61–90 days) | 553 |
| Default | 557 |
| Late (31–60 days) | 561 |
| Charged Off | 571 |
| Fully Paid | 579 |
| Current | 585 |

Borrowers with scores below 500 default at a rate of 36.1% vs 33.6% for those above 500. The difference is modest, suggesting the current credit score threshold alone is not a strong enough predictor of default.

---

### 4. Loan purpose risk

| Loan purpose | Total disbursed (KES) | Avg interest rate | Default rate |
|---|---|---|---|
| Education | 96,680,313 | 17.2% | 36.6% |
| Personal | 103,719,046 | 18.1% | 36.6% |
| Medical | 102,187,380 | 17.7% | 35.7% |
| Business | 103,873,456 | 18.2% | 34.4% |
| Home Improvement | 88,849,551 | 17.9% | 34.2% |
| Agriculture | 100,324,294 | 18.0% | 31.0% |

> Agriculture is the safest loan purpose at 31% default rate. Education and Personal loans carry the highest risk.

---

### 5. Employment status vs default

| Employment status | Total loans | Default rate |
|---|---|---|
| Self-Employed | 387 | 36.7% |
| Business Owner | 373 | 36.2% |
| Employed | 347 | 34.6% |
| Unemployed | 368 | 30.2% |

> Counterintuitively, unemployed borrowers have the lowest default rate — likely because they receive smaller loan amounts. Self-employed and business owners carry the most risk, possibly due to income volatility.

---

### 6. Debt-to-income ratio exposure

| DTI band | Borrowers | Avg loan (KES) | Default rate |
|---|---|---|---|
| Low risk (DTI < 0.35) | 816 | 296,757 | 33.6% |
| Moderate risk (0.35–0.50) | 257 | 539,614 | 39.3% |
| High risk (DTI > 0.50) | 433 | 553,166 | 33.3% |

> The moderate DTI band (0.35–0.50) has the highest default rate at 39.3%, and these borrowers also have significantly higher average loan amounts. This suggests mid-range leveraged borrowers are the most vulnerable segment.

---

## Three recommendations for the credit manager

1. **Introduce county-level risk tiers.** Meru, Thika, and Kakamega have default rates above 36% — consider tighter approval criteria or lower loan ceilings in these regions until the pattern is understood.

2. **Review the credit score threshold.** The difference in default rates between borrowers above and below 500 is only 2.5 percentage points — the current scoring model may not be calibrated well enough. A review of the scoring model or a composite risk score (incorporating DTI and late payment history) is recommended.

3. **Flag moderate-DTI borrowers (0.35–0.50) for enhanced review.** This segment has the highest default rate (39.3%) and the largest average loan size, making them the highest-loss-value risk in the portfolio.

---

## Tools and skills demonstrated

- Data profiling and issue documentation
- Excel formula-based cleaning (IF, OR, MEDIAN, IFERROR, DATEVALUE, DATE, ABS, ISNUMBER)
- Array formulas with conditional logic
- Domain knowledge application (Kenyan CRB credit score range 0–999)
- Business-framed analysis for non-technical stakeholders
- Portfolio segmentation (healthy / at risk / non-performing)

---
