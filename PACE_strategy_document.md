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
## PACE: Construct Stage — Reflections

### Do you notice anything odd?
Yes — a key anomaly was the very high initial performance of the tree-based 
models (Round 1 AUC ~0.97-0.98), which raised suspicion of data leakage. 
Investigation identified two likely culprits: `satisfaction_level` (a survey 
value that may not be reliably available for every employee in production) 
and `average_monthly_hours` (which may reflect the *consequence* of an 
employee's departure decision rather than a cause of it — employees who've 
already decided to leave, or been marked for termination, may work fewer 
hours as a result). Another odd finding was that `last_evaluation` showed 
almost no linear correlation with `left` (0.0066) in the correlation heatmap, 
yet emerged as the single most important feature in both tree-based models — 
this reflects a non-linear/interaction effect that simple correlation missed.

### Which independent variables did you choose for the model and why?
The final (champion) models used all available variables except 
`satisfaction_level` and `average_monthly_hours`, which were dropped for the 
data leakage reasons above. `average_monthly_hours` was replaced with a 
coarser binary feature, `overworked` (>175 hrs./mo.), to retain a workload 
signal without the leakage risk. Categorical variables (`department`, 
`salary`) were encoded — `salary` ordinally (given its natural low/medium/high 
ordering) and `department` via one-hot encoding (no natural ordering).

### Are each of the assumptions met?
For logistic regression, several assumptions were violated: the linearity 
assumption between predictors and the logit of the outcome did not hold for 
`number_project` and `average_monthly_hours`, both of which showed non-linear 
(U-shaped) relationships with attrition during EDA. This was reflected in the 
model's poor recall (0.26). Multicollinearity was checked and found not to be 
severe among the continuous predictors (max correlation 0.33, well below the 
~0.7-0.8 concern threshold). For the tree-based models, no linearity or 
distributional assumptions apply, which is part of why they performed 
substantially better.

### How well does your model fit the data?
The champion random forest model achieved strong, consistent performance: 
test-set recall of 0.90, precision of 0.87, F1 of 0.89, accuracy of 0.96, and 
AUC of 0.94. Test-set scores closely matched (and in some cases slightly 
exceeded) cross-validation scores, indicating the model generalizes well and 
is not overfitting.

### Can you improve it? Is there anything you would change about the model?
Possible next steps: (1) explore additional feature engineering, such as 
interaction terms between `number_project` and `tenure`, given their combined 
importance; (2) test alternative overwork thresholds beyond 175 hrs./mo. to 
see if recall/precision can be further balanced; (3) evaluate XGBoost as a 
third model type, as instructed in the original activity, for a fuller 
comparison; (4) consider whether decision tree (slightly higher recall, much 
faster training, more interpretable) might be preferable to random forest 
(slightly higher AUC) depending on how the model will actually be deployed 
and explained to stakeholders.

### Do you have any ethical considerations in this stage?
The data leakage investigation itself was an ethical consideration — deploying 
a model that relies on `satisfaction_level` could produce misleadingly 
optimistic performance claims if that data isn't consistently available in 
practice. More broadly, this model should be framed to stakeholders as a tool 
for proactively supporting employees flagged as at-risk (e.g., workload 
review, career development conversations), not for penalizing them or 
treating the prediction as a certainty. Given `last_evaluation` and 
`number_project` are the top drivers, any resulting interventions should focus 
on manager-level workload and performance-review practices, rather than 
individual employee blame.

## EXECUTE Stage

### What key insights emerged from your model(s)?
Four variables — `last_evaluation`, `number_project`, `tenure`, and 
`overworked` — account for ~99.6% of the model's predictive power, while 
department and salary contribute almost nothing. The relationship between 
workload (project count, hours) and attrition is U-shaped, not linear: both 
underutilized employees (2 projects) and overworked employees (6-7 projects) 
show elevated attrition, while a moderate workload (3-4 projects) is 
associated with retention. Employees who were promoted in the last 5 years 
almost never left. Tenure showed a non-linear pattern too — risk rose in the 
mid-tenure range (5-6 years) before dropping to zero for employees with 7+ 
years.

### What business recommendations do you propose based on the models built?
- Cap the number of projects employees can be assigned to.
- Investigate promotion and career-growth pathways, especially for 
  mid-tenure (5-6 year) employees.
- Reward or reduce excessive working hours rather than requiring them without 
  proportionate compensation.
- Clarify overtime pay policies and workload/time-off expectations.
- Hold company-wide and team-level discussions on work culture.
- Avoid reserving high evaluation scores only for employees working 200+ 
  hours/month — use a more proportionate scale for rewarding effort.
- Since department and salary are not meaningful drivers, prioritize 
  retention interventions company-wide rather than targeting specific 
  departments or pay bands.

### What potential recommendations would you make to your manager/company?
Deploy the model as a proactive support tool — flagging at-risk employees for 
manager check-ins, workload review, or career development conversations — 
rather than as a punitive tool. Since the model relies on data the company 
already reliably collects (`last_evaluation`, `number_project`, `tenure`, 
hours), it can be run on the full active employee population without 
additional data collection like satisfaction surveys.

### Do you think your model could be improved? Why or why not? How?
Yes: (1) test XGBoost as a third model type for a fuller comparison; (2) 
explore interaction features (e.g., `number_project` × `tenure`); (3) test 
alternative `overworked` thresholds; (4) investigate whether `last_evaluation` 
— the single most important feature — carries a similar data leakage risk to 
`satisfaction_level`, since evaluations may not be conducted frequently enough 
to be reliably available for all employees, and/or the causal direction may 
run the other way (dissatisfaction could lower evaluation scores rather than 
low scores causing departure).

### Given what you know about the data and the models you were using, what other questions could you address for the team?
- Is the mid-tenure attrition spike (5-6 years) linked to a specific policy 
  (e.g., promotion cycles, benefit vesting)?
- What's driving the unusually narrow, low satisfaction scores seen in the 
  4-year tenure group during EDA?
- Would predicting `last_evaluation` or `satisfaction_level` directly (rather 
  than `left`) yield more actionable insight into their own drivers?
- Could K-means clustering reveal natural employee segments worth exploring 
  separately?


### Do you have any ethical considerations in this stage?
Recommendations should be framed to support flagged employees (workload 
review, career conversations) rather than penalize them. Given the unusually 
sharp cluster boundaries observed during EDA (suggesting possible synthetic 
data), findings should be validated against real company data before being 
used to justify actual HR policy changes.
