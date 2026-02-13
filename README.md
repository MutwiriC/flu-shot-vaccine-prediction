# Flu Vaccine Prediction

## Project Overview

This project builds machine learning models to predict flu vaccination status using survey data from the 2009 National H1N1 Flu Survey. The goal is to identify individuals unlikely to receive vaccines so public health campaigns can be more targeted.

## Stakeholder

Centers for Disease Control and Prevention (CDC) - Public Health Division
The models and insights from this project are designed to help public health officials:

- Allocate limited outreach budgets more effectively
- Design personalized intervention strategies
- Understand the behavioral and demographic drivers of vaccine hesitancy

## Business Problem

Current vaccination campaigns are broad and population-wide. This means resources are spent on people who would vaccinate anyway, while vaccine-hesitant populations remain unengaged. Better targeting could improve both efficiency and outcomes.
<img width="295" height="202" alt="Seasonal Vaccine Distribution" src="https://github.com/user-attachments/assets/0044de47-f28a-48dd-9a8a-579c27d3821f" />

**_Two targets:_**

- H1N1 vaccine (pandemic strain)
- Seasonal flu vaccine (annual shot)


## Data

- Source: DrivenData Flu Shot Learning Competition
- Size: 26,707 respondents

Files:

- Training Features: `training_set_features.csv`
- Training Labels: `training_set_labels.csv`
- Test Features: `test_set_features.csv`

Features: 36 total

- 23 numerical (ratings, counts, binary flags)
- 12 categorical (age group, region, education, etc.)
- 1 ID column (respondent_id)

Targets flipped: We predict 1 = Did NOT vaccinate, 0 = Vaccinated.

## Approach

**_Model 1_**
* Algorithm - Logistic Regression
* Target - Seasonal vaccine
* Rationale - Interpretable baseline

**_Model 2_**
* Algorithm - Decision Tree
* Target - H1N1 vaccine
* Rationale - non-linear pandemic behavior

**_Primary Metric: Recall_**
We optimized for recall (true positives / (true positives + false negatives)).
Missing someone who needs an intervention is worse than occasionally contacting someone who doesn't need it. A false negative = missed opportunity to prevent illness.

## Data Processing

**_Cleaning Steps_**

1. Removed columns with >40% missing data
   _ health_insurance (54% missing)
   _ employment_industry (50% missing) \* employment_occupation (50% missing)
   Imputing half or more of a column is guessing, not estimating.

2. Numerical columns - Imputed with median
   - Median is robust to outliers
   - Better than mean for income, household size, etc.

3. Categorical columns - Imputed with mode (most frequent value)
4. Train-test split - 80/20 with stratification
   - Maintains same proportion of vaccinators/non-vaccinators in both sets

**_Preprocessing Pipelines_**

1. Pipeline A (with scaling) - for Logistic Regression:
   Logistic regression uses gradient descent and assumes features are on similar scales. Scaling prevents features with larger numeric ranges from dominating.
2. Pipeline B (without scaling) - for Decision Tree:
   Decision trees split on values

## Modeling Results

### Model 1: Logistic Regression (Seasonal Vaccine) Test set performance:

text Recall: 0.**8221** Precision: 0.**7916** F1-Score: 0.**8065** **ROC**-**AUC**: 0.**8560** Confusion matrix breakdown:

* True Negatives (correctly identified vaccinators): 1,**876**
* False Positives (wasted outreach): **611**
* False Negatives (missed opportunities): **508**
* True Positives (correctly identified non-vaccinators): 2,**347**

Out of **100** people who won't get the seasonal vaccine, we flag 82 for intervention. We miss 18.

### Model 2: Decision Tree (**H1N1** Vaccine) - Initial Test set performance
* text Recall: 0.**8393** 
* Precision: 0.**8453** 
* **ROC**-**AUC**: 0.**6351** 

Problem: **ROC**-**AUC** of 0.**635** is significantly lower than logistic regression. This suggests overfitting - the tree memorized training data patterns that didn't generalize.

### Hyperparameter Tuning (GridSearchCV) Parameters tuned:

- max_depth: [3, 5, 7, 10, None] - How deep the tree can grow
- min_samples_split: [2, 5, 10, 20] - Minimum samples required to split a node
- min_samples_leaf: [1, 2, 5, 10] - Minimum samples allowed in a leaf node
- criterion: ['gini', 'entropy'] - How split quality is measured

Best parameters found:

* text max_depth: 5 min_samples_split: 2 min_samples_leaf: 1 criterion: entropy Tuned model performance:
* text Recall: 0.**9513** (↑ from 0.**8393**) Precision: 0.**8493** F1-Score: 0.**8974** **ROC**-**AUC**: 0.**8124** (↑ from 0.**6351**) What changed: Limiting tree depth to 5 prevented memorization. The model now finds broader patterns rather than respondent-specific rules.

## Feature Importance

### Top 10 predictors of non-vaccination (**H1N1** model):
<img width="527" height="311" alt="Feature importance" src="https://github.com/user-attachments/assets/02d2d20d-c6d7-4469-9c8d-473c00eb9900" />


Observations:

- Medical recommendations and vaccine beliefs dominate the top spots
- Demographics matter but are secondary to trust and guidance
- People who perceive themselves at risk are more likely to vaccinate

## Business Implications

### 1. Prioritize Based on Risk Scores The final model outputs probabilities from 0.0-1.0. These can directly inform tiered outreach:

Score Range Priority Suggested Action
0.7 - 1.0 High Direct outreach (calls, mailers, home visits)
0.4 - 0.6 Medium Digital campaigns, reminders
0.0 - 0.3 Low General awareness only

### 2. Leverage Healthcare Providers
Doctor recommendation is the single strongest predictor. A 3-minute conversation during an existing appointment is more effective than any flyer.

Recommendation: Develop materials that help providers efficiently discuss vaccines with patients. Consider incentive programs for clinics with high recommendation rates.

### 3. Address Vaccine Confidence Systematically

Belief in vaccine effectiveness ranks consistently high. Different populations have different concerns - younger adults worry about side effects, older adults want efficacy data.

Recommendation: Segment messaging by age and previous vaccination history. One message does not fit all.

### 4. Age-Targeted Strategies

Current patterns show seniors already vaccinate at high rates. Young adults (18-34) have the lowest uptake.

Recommendation: Shift resources toward younger demographics. Consider workplace clinics, university partnerships, and social media campaigns.

## Limitations What this project doesn't address:

- Causation vs. correlation - We know what's associated with non-vaccination, not what causes it. Doctor recommendation predicts uptake; we don't know if increasing recommendations would cause the same effect.

- Data age - **2009** was 15+ years ago. **H1N1** was a unique pandemic; **COVID**-19 may have shifted attitudes further. These patterns should be validated on recent data.

- Self-report bias - People misremember or misrepresent. Someone might say they washed hands frequently because they know they should, not because they did.

- Missing structural variables - No questions about cost, transportation, time off work, or clinic access. We can identify hesitant individuals but not those who face practical barriers.

- Geographic granularity - Only **HHS** region available, not state or county. Can't target local campaigns.
