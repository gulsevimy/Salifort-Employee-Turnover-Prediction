# PACE Strategy Document
## Salifort Motors — Employee Turnover Prediction

---

## PLAN Stage

### 1. Business Problem
Salifort Motors is experiencing a high rate of employee turnover (both voluntary departures and terminations). High turnover is costly due to the investment the company makes in recruiting, training, and upskilling employees. Leadership wants to understand what factors are driving employees to leave and, ideally, predict which employees are at risk of leaving so the company can intervene proactively.

### 2. Goal of the Project
Design a model that predicts whether an employee will leave the company, using survey and HR data (e.g., job title, department, number of projects, average monthly hours, and other relevant variables). The end goal is to translate model findings into concrete recommendations that improve employee retention and job satisfaction.

### 3. Stakeholders
- Salifort's senior leadership team (primary audience for the executive summary)
- Human Resources department (who collected the survey data and will act on recommendations)

**Stakeholder needs:** Leadership is less interested in technical model details and more interested in *which factors drive turnover* and *what actions can be taken*. The executive summary should prioritize clear, actionable insights over statistical detail.

### 4. Data Overview
Dataset: `HR_capstone_dataset.csv`
- 14,999 rows (one row per employee, self-reported)
- 10 columns:

| Column | Type | Description |
|---|---|---|
| satisfaction_level | int64* | Employee's self-reported satisfaction [0–1] |
| last_evaluation | int64* | Score of employee's last performance review [0–1] |
| number_project | int64 | Number of projects employee contributes to |
| average_monthly_hours | int64 | Average hours worked per month |
| time_spend_company | int64 | Tenure in years |
| work_accident | int64 | Whether employee had a workplace accident (binary) |
| left | int64 | **Target variable** — whether employee left the company (binary) |
| promotion_last_5years | int64 | Whether employee was promoted in last 5 years (binary) |
| department | str | Employee's department |
| salary | str | Employee's salary level (low, medium, high) |

*Note: `satisfaction_level` and `last_evaluation` are documented as int64 but represent scores in a [0–1] range — this should be verified during EDA, as they are more likely stored as float64. Flagging data type mismatches early is a good habit before modeling.

### 5. Key Questions to Explore
- Which variables correlate most strongly with an employee leaving?
- Is turnover concentrated in specific departments or salary bands?
- Does workload (number_project, average_monthly_hours) relate to attrition?
- Does tenure (time_spend_company) show a pattern in who leaves?
- Do promotions (or lack thereof) affect retention?
- Is there a satisfaction/evaluation threshold below which turnover sharply increases?

### 6. Ethical Considerations
- The data is self-reported (satisfaction, evaluation), which can introduce reporting bias.
- A model predicting who is "likely to leave" carries risk of misuse — e.g., being used to deprioritize investment in employees flagged as flight risks, rather than to support them. Recommendations should frame predictions as a tool for *proactive support*, not employee scoring.
- Department and salary data should be handled carefully to avoid reinforcing existing inequities in how departments or pay bands are treated.

### 7. Success Criteria
- A model that reliably distinguishes employees likely to leave from those likely to stay.
- Given the business use case (informing HR interventions), **recall** on the "left" class is prioritized — missing an at-risk employee (false negative) is costlier than a false positive, since a false positive simply results in extra attention/support given to someone who wasn't at risk.
- Clear, defensible feature importance findings that can be explained to non-technical stakeholders.

---

## ANALYZE Stage
## PACE: Analyze Stage — Reflections

### What did you observe about the relationships between variables?
Several clear relationships emerged. `satisfaction_level` had the strongest 
correlation with `left` (-0.39) — lower satisfaction is associated with leaving. 
Two workload variables, `number_project` and `average_monthly_hours`, showed a 
**non-linear (U-shaped)** relationship with attrition: both very low workload 
(2 projects, ~130-160 hrs./mo.) and very high workload (6-7 projects, 
240-310 hrs./mo.) were associated with leaving, while moderate workload (3-4 
projects) was associated with staying. `promotion_last_5years` also stood out — 
employees who were promoted almost never left, while unpromoted employees who 
worked very long hours were disproportionately represented among leavers. 
`tenure` showed a similarly non-linear pattern: attrition risk rose in the 
mid-tenure range (5-6 years) before dropping to zero for employees with 7+ 
years of tenure. Department and last_evaluation, by contrast, showed weak 
relationships with attrition on their own.

### What do you observe about the distributions in the data?
Several distributions showed unusually sharp, almost rectangular cluster 
boundaries (e.g., in the `average_monthly_hours` vs. `satisfaction_level` and 
`vs. last_evaluation` scatterplots, and the tenure-4 satisfaction boxplot) 
rather than the smooth, organic spread expected from genuine self-reported 
data. This suggests parts of the dataset may be synthetic. Department sizes 
were also unevenly distributed (from 436 employees in Management to 3,239 in 
Sales), which affects how reliable department-level attrition rates are.

### What transformations did you make with your data? Why did you choose to make those decisions?
Duplicate rows were removed — it was highly unlikely that different employees 
would report identical values across all 10 columns, so these were treated as 
non-legitimate entries. Outliers in `tenure` were identified using the IQR 
method (1.5x IQR rule) to flag employees whose tenure fell unusually far from 
the typical range, which will inform decisions about how to handle them 
depending on which model is used (tree-based models are generally robust to 
outliers, while logistic regression is more sensitive).

### What are some purposes of EDA before constructing a predictive model?
EDA helps surface data quality issues (duplicates, outliers, possible synthetic 
patterns) before they silently affect model results. It also reveals the shape 
of relationships between variables — for example, discovering that workload's 
relationship with attrition is U-shaped rather than linear directly informs 
model choice, since linear models like logistic regression won't capture that 
pattern as well as tree-based models would without additional feature 
engineering. Finally, EDA helps identify which variables are likely to be 
strong predictors, guiding feature selection.


### Do you have any ethical considerations in this stage?
The dataset's self-reported nature (satisfaction, evaluation) may carry 
reporting bias. The possible presence of synthetic or manipulated data (based 
on the unusually sharp cluster boundaries observed) is worth flagging as a 
limitation — any conclusions drawn should be validated against real company 
data before being used to justify actual HR decisions. Additionally, care 
should be taken in how findings are communicated: framing predictions as a tool 
to proactively support at-risk employees, rather than to penalize or target 
them, is important to avoid misuse of the model's output.
## CONSTRUCT Stage
*(To be completed after model building)*

## EXECUTE Stage
*(To be completed after model evaluation, for the executive summary)*
