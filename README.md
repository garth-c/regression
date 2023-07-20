# Regression
Project Description:
In this project, I will develop a regression model to insurance costs based on several predictor variables. The response variable is numeric and it represents the claim costs associated with customers by an insurance company and it is in dollars. The predictor variables are both numeric and categorical for the person associated with the insurance claim cost. 

I will use a regression model for this demo and the overall goal is to develop an explanatorey model with the highest R^2 value. This is an exploratory project so the emphasis will be in iteration and discovery. This project will use R and RStudio to develop the models. 

Data description:
source: Machine Learning with R dataset
data title: Inurance cost modeling data set
dimensions of the data (RxC): 1,338 x 7

I modified some of the numeric values by rounding them to two decimal places in Excel. A screen shot of the first few records is shown below.

<img width="257" alt="image" src="https://github.com/garth-c/regression/assets/138831938/9c0e1db4-e212-4dd0-92e5-392ac8558c8f">

------------------------------------------------------------------------------------------------

# road map for this demo
+ data ingest and data prep
+ explore the data
+ feature selection
+ regression model
+ interpret model results

-------------------------------------------------------------------------------------------------

#data ingest and data prep

Set up the computing environment in RStudio.

```
###~~~
#set up the computing environment
###~~~

#load the needed libs
library(readxl) #read in excel files
library(tidyverse) #data wrangling
library(skimr) #data quality eval
```

Read in the data and validate it against the source data file. Change character data to factors to aid in the modeling process. 

```
###~~~
#read in source data set
###~~~

#insurance data
insurance <- read_excel('insurance.xlsx', 
                         sheet = 'insurance')

#validate the input
View(insurance)
glimpse(insurance)
dim(insurance) #must match the input file dimensions
str(insurance)
```

The output from skimr, which was used for the data quality asessment is below. There were no conerns with this data.

<img width="673" alt="image" src="https://github.com/garth-c/regression/assets/138831938/ab4d11db-64f9-47ed-b32e-2d810fb40907">


--------------------------------------------------------------------------------------------------------

# explore the data

Next, the source data and the relationships between the variables are explored. The first thing to do is to load the needed libraries to help with this process.

```
#load the needed libs
library(PerformanceAnalytics) #numeric data exploration
library(rcompanion) #categorical data exploration
library(pwr) #power testing
```

A quick summary of the source data is shown below.

```
#quick overall summary
summary(insurance)
```

<img width="671" alt="image" src="https://github.com/garth-c/regression/assets/138831938/3c5ad378-f597-44d0-9b59-423f420efd14">

The numeric predictors are: age, bmi, and children. All of the other predictors are categoricals. The next step is to review the response data at a high level. I will use a histogram for this that uses 30 groups to display the data.

```
#quick plot of the response variable
windows()
hist(insurance$charges,
     breaks = 30,
     main = 'Histogram of claims insurance costs')
```

The histogram is shown below. As can be seen from the plot, the response data is very right skewed (long declining tail to the right). This is a clue that one of the regressions models to try is a Gamma regression model. I will fit multiple models to see which one works the best for this project goal.

![image](https://github.com/garth-c/regression/assets/138831938/704f75cc-08b2-4625-9c2a-8d3d2bef491b)


Now, the numeric variables are put into a separate data frame and analysed. The idea is that there shouldn't be too much correlation between the numeric predictor variables otherwise this would increase the risk of multicollinearity with the model. For this asssessment, I will focus on correlation between the variables.

```
###~~~
#explore the numeric predictor variables
###~~~

#put numeric values into a holding data frame
numerics <- insurance[, c(2,4,5)]

#correlation matrix chart
windows()
PerformanceAnalytics::chart.Correlation(numerics, 
                                        histogram = TRUE, 
                                        method = 'spearman')

#look for outliers in the numeric values
windows()
boxplot(numerics,
        main = 'Boxplot to find outliers')
```

The numeric predictor correlations are shown below. There isn't much correlation between the them which is what I want for numeric predictor variables. 

![image](https://github.com/garth-c/regression/assets/138831938/445cf6d8-bea2-4f5e-a816-fa5411990128)

Looking for outliers in the numeric predictors, there are some in the bmi, but not enough to be concerned with, which is also a good sign for the value of these variables as predictors. 

![image](https://github.com/garth-c/regression/assets/138831938/1c9fa156-a022-4cdf-8f11-2bd61c8b9c4a)

Now I will analyze the categorical variables. The function that I will use for this a Cramer's V which incorporates additional factors into a Chi-Square test such as sample size and then put the effect size into a 0 -> 1 scale which is very convenient. The lower the Cramer's V score, the less correlation there is between the categorical factors which is what I want for categorical predictors.  

```
###~~~
#explore the categoricals
###~~~

#put numeric values into a holding data frame
categoricals <- insurance[, -c(1,2,4,5)]

#use cramer's V for the test of association between suspect variables
rcompanion::cramerV(x = categoricals$smoker,
                    y = categoricals$region)
```

An example output is shown below. After reviewing all of the categorical combinations, there were no highly correlated categorical values to be concerned with. 

<img width="313" alt="image" src="https://github.com/garth-c/regression/assets/138831938/1b1e7b1b-72d0-492c-a975-5ec3af92a0ca">

Next, I will compare the numeric predictors to the categorical predictors to see if there are any highly correlation combinations. If there are any significant correlations, I will test the power of the output and then evaluate the effect size to determine if a removal should be made. The idea is to compare the means and mean ranks of the numeric values over the groups in the categoricals to see if there are any significant differences which indicate the a strong correlation. 

After reviewing all combinations of the numeric to categorical values, the only combination with a significant correlation was bmi over region.

The first part is a visual inspection for any obvious signs of a difference. In this specific example, there is definitely visual differences in bmi between the regions. 

```
###~~~
#explore numeric to categoricals
###~~~

#box plot for visual inspection
windows()
boxplot(bmi ~ sex,
        data = insurance,
        main = 'Compare numeric distribution between groups')
```


![image](https://github.com/garth-c/regression/assets/138831938/6ff99f81-e4a1-4220-bcf0-a2d5bfcd9d1e)


The next part is a computation test to see if the visual differences detected above are statistically significant. This is accomplished with a Kruskal test which is a non parametric one way ANOVA test that uses mean ranks as the evaluation data.

```
#one way ANOVA test - non parametric version
#Ho: mean ranks of the groups are statistically the same
#Ha: at least one sample mean rank differs from the rest
#a non-significant result means the two are weakly correlated
kruskal.test(bmi ~ sex,
             data = insurance)
```

The output from this test for bmi over region is shown below. The test result is significant which indicates that there is significant correlation for this combination. This significant result now needs more context to properly evaluate it.

<img width="410" alt="image" src="https://github.com/garth-c/regression/assets/138831938/70542598-13a4-4a32-af14-d2c92e7588e6">

The first part of this context is a power test to determine of there is enough source data to detect our desired effect size (small in this case) and if the test results are based on enough data that they would be considered reliable and repeatable. There isn't a specific power test for a Kruskal function that I am aware of so I will substitute a Chi-Square power test instead as it is probably close enough for an exploratory project. 

```
#chi-square power testing - probably the closest one to a kruskal test
#set the desired effect size to detect
effect_size_chsq <- pwr::cohen.ES(test = 'chisq',
                                  size = 'small')
#conduct the power test
pwr::pwr.chisq.test(w = effect_size_chsq$effect.size,
                    N = nrow(insurance), #sample size is the row count
                    df = (4-1), #four regions
                    #power = 0.80,
                    sig.level = 0.05)
```

The power calculation output is shown below. The power metric is ~88% and a typical acceptable metric for the input is 80%. Thus, there are more than enough samples to produce reliable results for this test. 

<img width="247" alt="image" src="https://github.com/garth-c/regression/assets/138831938/b411127a-9221-4459-ac00-fdcb8b8c24ce">


The next step to is evaluate the effect size of this correlation

```
#effect size testing for a significant result
rstatix::kruskal_effsize(data = insurance,
                         bmi ~ sex, #formula input
                         ci = TRUE, #need confidence intervals
                         conf.level = 0.95, #set the confidence level
                         ci.type = 'basic', #just roll with the basics
                         nboot = 1000) #number of bootstrap reps for the c.i.
```

The results are shown below. Since the effect size is small in magnitude, any adjustements or removals for this specific correlation combination is not warranted and I will leave this data alone. 

<img width="382" alt="image" src="https://github.com/garth-c/regression/assets/138831938/df4fb51b-dac7-4820-8de8-6194ed7759b9">

Now to evaluate the numeric predictors to the numeric response variable. This will informative as to what kind of regression is needed based on these relationships. For example if there is a polynomial relationship between one of the predictors and the response, then this could be accounted for in the regression model definition. So for this section, I want to evaluate the bend in the relationship line between all combinations of these variable. The relationship line in this case is a 'loess' line. 

Relationship between charges and children is below. There isn't enough bend in the loess line to be compelling of a model definition adjustment.

![image](https://github.com/garth-c/regression/assets/138831938/d825be5c-5c79-4f43-b9aa-77463ad71226)

Relationship between charges and age is below. There isn't enough bend in the loess line to be compelling of a model definition adjustment.

![image](https://github.com/garth-c/regression/assets/138831938/9255008f-86ca-4884-acb8-c5deaab1f1a6)

Relationship between charges and bmi is below. There isn't enough bend in the loess line to be compelling of a model definition adjustment.

![image](https://github.com/garth-c/regression/assets/138831938/f8495245-b964-4b00-9285-264b46713ead)


The last part of the exploration phase is to compare the numeric response means/mean ranks over the predictor categoricals for all combinations. A strong correlation is expected since these are predictor variables. After performing this analysis using the same code as above for the cateorical to numeric predictors, only 'smoker' was a strong correlation. The modeling process will evaluate all of these relationships. 


# feature selection



















  
