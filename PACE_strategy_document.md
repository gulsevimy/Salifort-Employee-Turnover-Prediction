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
*(To be completed after EDA)*

## CONSTRUCT Stage
*(To be completed after model building)*

## EXECUTE Stage
*(To be completed after model evaluation, for the executive summary)*
