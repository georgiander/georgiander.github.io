---
title: Classification Models on Churn
layout: page
menubar: projects_menu
---

# Various Classification Models on Customer Churn

[GitHub Repository](https://github.com/georgiander/Data-Analysis-Projects/blob/master/CustomerChurnAnalysis.ipynb)

This project I began while completing an IBM course in Machine Learning: Classification. This page is a log of all I have accomplished with this project and the code behind it is accessible via the link at the top of the page.

**Preface**

This report will outline the process taken for choosing a model regarding customer churn within Telco. The set uses various demographic information about the users and their service information. These factors can ultimately impact the rate at which customers are likely to leave the company. This metric is known as churn. Below is a data dictionary of all the features and their potential labels.

<details>
    <summary>Open to View Data Dictionary</summary>

| Feature | Labels |
|:--- | ---:|
|Count | Continous |
|Gender | male, female |
|Age|Continous|
|Senior Citizen| Yes, No|
|Married|Yes, No|
|Dependents|Yes, No|
|Partner|Yes, No|
|Tenure | Continous|
|Phone Service|Yes, No|
|Mulitple Lines|Yes, No|
|Internet Service|DSL, Fiber Optic, Cable, No|
|Online Security|Yes, No|
|Device Protection|Yes, No|
|Tech Support|Yes, No|
|StreamingTV|Yes, No|
|Streaming Movies|Yes, No|
|Contract|Month-to-Month, One-Year, Two-Yeat|
|Paperless Billing|Yes, No|
|Payment Method|Bank Withdrawal, Credit Card, Mailed Check|
|Monthly Charges|Continous|
|Total Charges|Continous|
|**Churn (Target Class)**|**Yes, No**|

</details>  

Since churn is a binary metric where the outputs are simply churned or didn't churn, the goal of this exploration is to predict who will churn. In addition, I will also explore the traits of those who churn as to better interpret the results.

## Data Cleaning
This specific dataset need several aspects to be cleaned before it can be put into a classification algorithm. This includes replacing missing data, encoding data and skewed classes.

### Missing Data
One issue I had that I didn't initially catch was several empty values within the 'TotalCharges' column. These did not come up when pulling null values because they were filled with spaces. Ultimately, I replaced all 11 empty spaces with '0 which fixed the dataset and removed the ValueError messages.

### Encoding Data
The majority of the preprocessing of this dataset is encoding the data in various ways. I organized the features into the three categories. First, features that already only had 2 choices. Second, features that have multiple labels and need to be label encoded. Finally, features that have multiple labels that boil down to two options. For example, the feature 'OnlineSecurity' has the potential to be 'Yes', 'No', or 'No Internet Service'. In cases like this, we will remove the excess option and leave just the binary. Below are the encoding categories and their respective features.

|Encoding Category | Features|
|---|---|
|One-Hot Encode (2 Options)|'gender', 'Senior Citizen', 'Dependents', 'PhoneService', 'PaperlessBilling|
|Consolidate (3 Options Into 2)|'MultipleLines', 'Online Security', 'Online Backup', 'Device Protection, 'Tech Support', 'StreamingTV', 'StreamingMovies'|
|Label Encode (Multiple Options)|'InternetService', 'Contract', 'PaymentMethod|

### Unbalanced Classes
The target class, 'Churn', has an unbalanced value count, as shown below. This is not relevant currently, but is important to note for the outcome of the models. Unbalanced data like this can often lead to underfitting the lower class.

![refimg](\refimgs\ChurnCount.png)

### Feature Correlation
After cleaning the data up, I created a following correlation graph. This graph show the relationship between all the aspects of the dataset. 

The deeper red the square, the higher the correlation. Most of the squares above are in the blue spectrum which means that the overall correlation of the features is low. With that being said, I expect the ability of future models to be limited.

![refimg](\refimgs\heatmap.png)


## Classification Models

### Inital Models
I began my observations by running generic models on the dataset to get a baseline. It is important to note here that all models are run with the same train-test split and, when applicable, cross-validation method. Initially, I ran a logistic regression and a decision tree model, both of which came out about 50/50 in practically all error metrics. There was no clear consensus and minor changes to the models did little to impact the dataset.

### Tuned Logisitic Regression Model

Next, I utilized a grid search with logistic regression to run a new model. This model performs significantly better than the initial models, as expected. The accuracy of this model is still relatively low, $51\%$, but the other metrics improved significantly. Specifically, the recall was $100\%$ and the F1-score was $96\%$. Since the model was built with the goal of predicting when people will churn, the most important metrics are recall and F1. 

||Initial| Tuned |
|:--|:--:|--:|
|Recall|0.51|1.0|
|F1-Score|0.52|0.96|

When you compare the model to the baseline, the performance increase is clear. The tuned model doubles the performance. With this model, it is easy to get an idea of who will churn, though there might be some incorrect labels. Furthermore, the coefficients of the model will give us the feature importance for the model, which is shown below.

![refimg](\refimgs\Imp_LR.png)

### Tuned Random Forest Model
The next model I fit was a Random Forest, which is an ensemble method and should perform similarly. This model's strength is in it's forest. Since it has the ability to cast such a wide net, it may also catch features that the logistic regression underrepresented. As expected, the tuned model performed just as well as the logistic regression model, as shown below.

||Initial| Tuned |LR Model |
|:--|:--:|:--:|--:|
|Recall|0.47|1.0| 1.0|
|F1-Score|0.47|0.95|0.96|

I would like to clarify that the grid search conditions are set to optimize the fscore, not accuracy, which is why they are performing poorly in that area.

The feature importances for this model are different than the first one and put much more weight on the MonthlyCharge than its counterpart.

![refimg](\refimgs\Imp_RF.png)

### SMOTE Random Forest

Finally, I performed a secondary random forest model on the data but this time I rebalanced the dataset. By upsampling with SMOTE the hope is to balance the data set and improve its F1-score.

||SMOTE| RF Model |LR Model |
|:--|:--:|:--:|--:|
|Recall|0.99|1.0| 1.0|
|F1-Score|0.96|0.95|0.96|


Ultimately, upsampling didn't heavily impact the model's capabilities. So this model performed almost the exact same as the original Random Forest. This makes sense as the coefficients of the decision trees shouldn't be influence by the distribution of labels within the target class.

## Current Takeaways

The best model to predict churn, in my opinion, is the logistic regression model. This model has fewer hyperparameters to set so tuning is much quicker. The slower methods do not perform any better than the logistic regression, so utilizing them for this purpose is simply a waste of time. Logistic Regression is also a much simpler model that is easy to interpret, or at least easier than a random forest model. 

The models also revealed how important each feature provided was to the models decision. The most important features to the models was 'TotalCharge'. The other features seemed to vary in importance to each model with 'PaymentMethod' and 'tenure' both appearing to have relevance throughout. I initially attributed these discrepancies to the lack of a scaler, but I reviewed my work and had, in fact, included a standard scaler in my pre-processing so that is not the issue. Seeing as the top features relate to the service side of a customer's profile not their socioeconomic profile, it is likely that the service provided and the way in which Telco interacts with its customers that is most important for customer retention.

## Planned Expansions
Moving forward with this dataset I plan to reevaluate features with high correlation with each other, like 'Senior Citizen' and 'Age'. Currently, I am mitigating redundancies by consolidating certain feature labels. This allows the dataset to be manipulated more easily but cuts out potentially important correlations. In future examinations of this dataset, I am going to reopen the unused labels to see if they are worth having.
