# Regression
Project Description:
In this exploratory regression modeling project, I delve into the world of predictive analysis using regression techniques. The primary objective is to gain insights into the relationships between various independent variables and a target variable of interest. Through comprehensive data exploration and visualization, I want to identify significant patterns and correlations within the dataset.

Utilizing popular regression algorithms, I will construct predictive models to forecast the target variable's values based on the input features. Throughout the project, I emphasize thorough documentation and explanatory visualizations, allowing users to comprehend the modeling process and results effectively.

Specificallyt, in this project I will develop multiple simple regression models to analyze insurance claims costs based on several predictor variables. The response variable is numeric and it represents the claim costs associated with customers by an insurance company and it is in dollars. The predictor variables are both numeric and categorical for the person associated with the insurance claim cost. 

The overall goal is to develop an explanatory regression model with the highest R^2 value. This is an exploratory project so the emphasis will be in iteration and discovery. This project will use R and RStudio to develop the models. Note that there are many other ways compare regression models but using R^2 is a straight forward method for purposes of this demo.

Data description:
+ source: 'Machine Learning with R' dataset
+ data title: Insurance cost modeling data set
+ dimensions of the data (RxC): 1,338 x 7

I modified some of the numeric values by rounding them to two decimal places in Excel. Also, there were no missing values in this data set. If there were, then I would have considered using the typical methods to address the situation. For example, removing the records if the count is small and not serious bias would be introduced by this action. Or filling in the missing values using means/medians/modes if no bias would be introduced. In addition, more sophisticated methods could be used to impute missing values by using other models to predict what the missing values would be based on the data that was available for the record. No other data prep was needed for this demo project. 

A screen shot of the first few records is shown below.

<img width="257" alt="image" src="https://github.com/garth-c/regression/assets/138831938/9c0e1db4-e212-4dd0-92e5-392ac8558c8f">

------------------------------------------------------------------------------------------------

# road map for this demo
+ data ingest and data prep
+ explore the data
+ feature selection
+ Multivariate regression model
+ Gamma regression model
+ Poisson regression model
+ compare the models

-------------------------------------------------------------------------------------------------

### Go back to my profile page
[garth-c profile page] (https://github.com/garth-c)

------------------------------------------------------------------------

# data ingest and data prep

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

The output from 'skimr', which was used for the data quality asessment, is below. There were no conerns with this data.

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

Looking for outliers in the numeric predictors, there are some in the bmi, but not enough to be concerned with, which is also a good sign for the value of these variables as predictors. If there were a significant amount of outliers, then a transformation of some variety (log, Box-Cox analysis, etc.) would be considered for this project. 

![image](https://github.com/garth-c/regression/assets/138831938/1c9fa156-a022-4cdf-8f11-2bd61c8b9c4a)

Now I will analyze the categorical variables. The function that I will use for this Cramer's V, which incorporates additional aspects into a Chi-Square test such as sample size and then scales the effect size into a 0 -> 1 scale which is very convenient. The lower the Cramer's V score, the less correlation there is between the categorical factors which is what I want for categorical predictors.  

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

After reviewing all combinations of the numeric to categorical values, the only combination with a significant correlation was bmi over region which is shown below. The first part is a visual inspection for any obvious signs of a difference. In this specific example, there are definitely visual differences in bmi between the regions. 

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

The output from this test for bmi over region is shown below. The test result is significant which indicates that there is significant correlation for this combination. This significant result now needs more context to properly evaluate it. Note that categorical data used in a Kruskal test is an 'off-label' usage, but I have deemed this an acceptable deviation for an exploratory model.

<img width="410" alt="image" src="https://github.com/garth-c/regression/assets/138831938/70542598-13a4-4a32-af14-d2c92e7588e6">

The first part of this context is a power test to determine of there is enough source data to detect our desired effect size (small in this case) and if the test results are based on enough data that they would be considered reliable and repeatable. There isn't a specific power test for a Kruskal function that I am aware of so I will substitute a Chi-Square power test instead as it is probably close enough for an exploratory project. For the degrees of freedom, I used the predictor variable with the most factor levels as the input (4 different regions) as I felt this was the most conservative calcuation to use (more degrees of freedom would mean a higher power metric threshold to meet). 

```
#chi-square power testing - probably the closest one to a kruskal test
#set the desired effect size to detect
effect_size_chsq <- pwr::cohen.ES(test = 'chisq',
                                  size = 'small')
#conduct the power test
pwr::pwr.chisq.test(w = effect_size_chsq$effect.size,
                    N = nrow(insurance), #sample size is the row count
                    df = (4-1), #four regions is the most conservative input
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

The results are shown below. Since the effect size is small in magnitude, any adjustements or removals for this specific correlation combination is not warranted and I will leave this data alone. The confidence interval for the effect size is between 4% and 9% with 95% confidence. So there definitely is an effect with this combination but it is really small. 

+ overall, if there were significantly large relationships between the predictors, this could be accounted for within the regression model definition. This would be in the form of combined predictors as a single term and then evaluate the model output as if the combined predictor was another unique predictor variable. In this case, I don't need to do any of this. 

<img width="382" alt="image" src="https://github.com/garth-c/regression/assets/138831938/df4fb51b-dac7-4820-8de8-6194ed7759b9">


Now to evaluate the numeric predictors to the numeric response variable. This will informative as to what kind of regression is needed based for these relationships. For example if there is a polynomial relationship between one of the predictors and the response, then this could be accounted for in the regression model definition by raising the numeric predictor to an appropriate power. So for this section, I want to evaluate the bend in the relationship line between all combinations of these variable. The relationship line in this case is a 'loess' line. 

Relationship between charges and children is below. There isn't enough bend in the loess line to be compelling of a model definition adjustment.

![image](https://github.com/garth-c/regression/assets/138831938/d825be5c-5c79-4f43-b9aa-77463ad71226)

Relationship between charges and age is below. There isn't enough bend in the loess line to be compelling of a model definition adjustment.

![image](https://github.com/garth-c/regression/assets/138831938/9255008f-86ca-4884-acb8-c5deaab1f1a6)

Relationship between charges and bmi is below. There isn't enough bend in the loess line to be compelling of a model definition adjustment.

![image](https://github.com/garth-c/regression/assets/138831938/f8495245-b964-4b00-9285-264b46713ead)


The last part of the exploration phase is to compare the numeric response means/mean ranks over the predictor categoricals for all combinations. A strong correlation is expected since these are predictor variables. After performing this analysis using the same code as above for the cateorical to numeric predictors, only 'smoker' was a strong correlation. The modeling process will evaluate all of these relationships. 

-----------------------------------------------------------------------------------------

# feature selection

For the preliminary feature selection proces, I will use the Boruta library. 

```
#load the needed libs
library(Boruta) #feature selection

#set the rando seed generator
set.seed(12345)
```

From there, the Boruta library will identify the most important predictors for the response variable.

```
###~~~
#boruta section
###~~~

#all predictors need to be factors
glimpse(insurance)

#feature Selection identification
key_predictors <- Boruta::Boruta(charges ~ .,
                                 data = insurance,
                                 doTrace = 3,
                                 pValue = 0.05,
                                 mcAdj = TRUE,
                                 holdHistory = TRUE,
                                 getImp = getImpRfGini,
                                 maxRuns = 20)

#look at the output
print(key_predictors)
attributes(key_predictors) #see what is available

#create a df to hold the results
key_predictors_df <- as.data.frame(key_predictors$finalDecision)
View(key_predictors_df)

#plot of the key predictor variables
windows()
plot(key_predictors,
     las = 2,
     cex.axis = 0.7,
     main = 'Key Predictors')

#plot the history - in case this is needed
windows()
plotImpHistory(key_predictors,
               main = 'Importance History')
```

The output from Boruta was put into a data frame and the Boruta decision on each of the predictors is shown below. This will be informative when the regression model is built but for now this is still just an advisement as this is an exploratory model. 

<img width="125" alt="image" src="https://github.com/garth-c/regression/assets/138831938/e41558fc-9b79-4e79-bd55-8c7d7b5fc990">

The Boruta plot is shown below. The model shows that age, bmi, and smoker are the best predictors for insurance claims. 

![image](https://github.com/garth-c/regression/assets/138831938/02f3fa83-7f02-42d1-b4f0-7aa2c4840654)

The Boruta model iterations and results for each of the predictors is shown below. The green lines represent the best Boruta predictors referenced above. 

![image](https://github.com/garth-c/regression/assets/138831938/d557c0f9-2b19-4978-99a6-269939c97866)

---------------------------------------------------------------------------------------------------------

# Multivariate Regression Model

This section will focus on building regression models to meet the project goal which is a model with the highest R^2 explanatory value. There are many other metrics that need to be considered for comparing regression models, but since this is exploratory, I will focus on one simple metric for the comparisons. 

Before the model development begins, it is important to note that the R functions for 'lm' and 'glm' will automatically create dummy variables for the categorical predictors. This process replaces the actual categorical data as shown below.


```
###~~~
#set up the computing environment
###~~~

#load the needed libs
library(readxl) #read in excel files
library(tidyverse) #data wrangling
library(fastDummies) #dummy data creation for categoricals
library(stats) #glm models

#set the random seed number
set.seed(12345)
```

The code to produce the dummy variable version of the source data is shown below.

```
###~~~
#create dummy variables for visual inspection
###~~~


#create dummy variables using dummyVars()
dummied_data <- fastDummies::dummy_cols(insurance)

glimpse(dummied_data)

#remove the redundant columns
insurance_adj <- dummied_data[, -c(3,6,7)]
```

The output for this is shown below. As can be seen, the categorical predictors are represented with 1/0 depending on if the factor level was either true or not true for that record.

<img width="559" alt="image" src="https://github.com/garth-c/regression/assets/138831938/991537d0-773c-4702-8350-f3801c325f81">


The first regression model is a multivariate OLS (ordinary least squares) model. The code is shown below.

```
#initial model
multiple_ols_model_all <- lm(charges ~ ., #charges are predicted by all variables
                             data = insurance_adj)

#evaluate the model
multiple_ols_model_all
summary(multiple_ols_model_all)

#look at the output
anova(multiple_ols_model_all)
```

The OLS output is shown below. From the model output, the predictors age, bmi, children, and smoker = no are the significant ones for predicting insurance costs. Overall, the model has an adjusted R^2 value of 74.9% which isn't bad. Also the F statistic has a significant result (p-value is almost zero) so this model does have merit. 

<img width="406" alt="image" src="https://github.com/garth-c/regression/assets/138831938/d6f6102a-d8e6-4d5c-8635-3cfb1b1e9658">

A screen shot of the OLS model ANOVA is shown below. This is a nice summary for the predictor variables importance to the response variable.

<img width="442" alt="image" src="https://github.com/garth-c/regression/assets/138831938/dde89436-9f25-4d13-914b-d3a207a80d81">

OLS models have a lot of underlying assumptions and I will test one of them here which is that the residuals are normally distributed. 

```
#plot the residuals
windows()
plot(multiple_ols_model_all$residuals,
     main = 'Model Residuals')
```

The residuals plot is shown below

![image](https://github.com/garth-c/regression/assets/138831938/a6b730a6-2f02-42ab-80ac-6e18595e3be0)

```
#Ho: residuals are normally distributed
#Ha: residuals are not normally distributed
shapiro.test(multiple_ols_model_all$residuals)
```

The plot doesn't show signs of non constant variance, grouping, or any real skew. But the computational normality test (Shapiro Wilks test) provide strong evidence that the residuals are not normally distributed which is a key OLS model assumption. So the results of the OLS model are deemed to be suspect at this point. 

<img width="259" alt="image" src="https://github.com/garth-c/regression/assets/138831938/3f63215a-ddf7-4627-acfc-453e2755b43d">

----------------------------------------------------------------------------------------------------------

# Gamma Regression Model

Since the numeric response variable is right skewed, I wanted to try a Gamma regression which is specifically used for this type of situation. The code for a Gamma regression model is below.

```
#gamma regression model
gamma_model <- glm(charges ~ .,
                   data = insurance_adj,
                   family = Gamma(link = 'log'))

#evaluate the model
gamma_model
summary(gamma_model)
```



The model output is shown below. From the model output, the predictors age, bmi, children, and smoker = no are the significant ones for predicting insurance costs. These are the same predictors as OLS model 


<img width="504" alt="image" src="https://github.com/garth-c/regression/assets/138831938/a1510e2d-92e8-4c82-bb94-9bbb40cb52a6">


Gamma regression models don't produce an R^2 value directly. Thus a version of this metric called a pseudo R^2 will be used as a substitute. The code to do this is below.

```
#calculate the McFadden's pseudo-R-squared
pseudorsq <- 1 - (gamma_model$deviance / gamma_model$null.deviance)

#print the pseudo-R-squared value
print(pseudorsq)
```

The output is below. The value is ~68% which is about the same as the OLS model.

<img width="136" alt="image" src="https://github.com/garth-c/regression/assets/138831938/43cb0e0a-b874-4844-a65c-dda5e3a30aa5">


The Gamma model produces an Analysis of Deviance table in lieu of an ANOVA table. The largest deviance values are the strongest predictor variables. The table is shown below.

```
#analysis of deviance table
anova(gamma_model)
```

<img width="334" alt="image" src="https://github.com/garth-c/regression/assets/138831938/5e0fffda-489f-427a-bbe5-2b2193870a46">

The underlying model assumptions are mostly the same for a Gamma regression. There is not obvious evidence of non constant variance with this plot

![image](https://github.com/garth-c/regression/assets/138831938/a1969cd4-311f-4641-bca8-79436cc559fc)

The computational normality test (Shapiro Wilks test) provide strong evidence that the residuals are not normally distributed which is a key Gamma model assumption. So the results of the Gamma regression model is also deemed to be suspect at this point.

<img width="234" alt="image" src="https://github.com/garth-c/regression/assets/138831938/e4ad96f0-6a68-4fde-9c80-2ee21384e2ec">

-----------------------------------------------------------------

# Poisson Regression Model

The last model to try is a Poisson regression model which works a little differently than OLS and Gamma. The code to produce this model is below.

```
#poisson model
poisson_model <- glm(charges ~ .,
                     data = insurance_adj,
                     family = 'poisson')

poisson_model
summary(poisson_model)
```
The output from this model is shown below. 

<img width="446" alt="image" src="https://github.com/garth-c/regression/assets/138831938/aa6fbb97-4c0d-478e-a829-464418636ec7">

The same tactics will be used to evaluate the assumptions for this model as well. The pseudo R^2 value code is below.

```
#calculate the McFadden's pseudo-R-squared
pseudorsq_poisson <- 1 - (poisson_model$deviance / poisson_model$null.deviance)

#print the pseudo-R-squared value
print(pseudorsq)
```

The result is about the same as the other two models at ~68%. 

<img width="134" alt="image" src="https://github.com/garth-c/regression/assets/138831938/f70e3cdd-2c07-44a1-8305-a6cd251f3206">

The Poisson model produces an Analysis of Deviance table in lieu of an ANOVA table. The largest deviance values are the strongest predictor variables. The table is shown below.

```
#analysis of deviance table
anova(poisson_model)
```

<img width="319" alt="image" src="https://github.com/garth-c/regression/assets/138831938/78e313fb-8350-44a5-b8c9-16b25d0dd34a">

The model residuals plot also shows no obvious signs of non constant variance. 

![image](https://github.com/garth-c/regression/assets/138831938/495686bb-51fb-4b68-82e7-73cb0c5b826e)

The computational normality test (Shapiro Wilks test) provide strong evidence that the residuals are not normally distributed which is a key Poisson model assumption. So the results of the Poisson regression model is also deemed to be suspect at this point.

<img width="236" alt="image" src="https://github.com/garth-c/regression/assets/138831938/051563f9-b5b7-49c2-a14b-7dc0ba17d953">

------------------------------------------------------------------------------------

# compare the regression models
One way to compare the models is to determine the common predictors between the model choices and then put the heaviest emphasis on the predictors from the model that best explains the data. All of this is compiled and put into the table below. In the significant predictors row, the common significant predictors are color coded between the models. As can be seen all models produced almost the same list of significant predictors. These predictors are almost the same as the Boruta output. The model with the best R^2 score is the multivarite OLS model. Since all three models produced mostly the same results, and this is also consistent with the Boruta output, it is reasonable to use these predictors to inform the non exploratory model building process. The summary table is shown below.

<img width="200" alt="image" src="https://github.com/garth-c/regression/assets/138831938/d0f3d2eb-03cc-4831-b08e-da286557e434">

Since this is an exploratory demo, I didn't perform an analysis on other regression concerns such as influential points in the data of Cooke's Distance analysis, etc. These methods would generally be applied in the non exploratory modeling phase of a project. Centering and scaling the numeric values would reduce the impact of outlier data points. There are many other tactics that could be explored to improve the model such as finding a grouping (descritization) for 'age', 'bmi', or 'children' that would improve the signal coming from this data. Also looking for other data to incorporate is another possible approach to improving these models. 

Lastly, the interpretation of these regression models is that for every additional unit of response variable, the predictor variables would increase or decrease at the rate represented by the coefficient in the model output. These coefficients are in the 'Estimate' column shown in each model output. The p-value associated with each predictor variable has a series of asterics next to them if they are significant. The really significant predictors have 3 asterics next to them. 

The output of these exploratory models is a good start to understanding the relationship between the predictor variables and the response variables. Once a reasonable model is developed, management is then able to respond to these relationships in order to acheive a goal such as increasing sales, decresing costs or risks, etc. Thus, regression modeling is a very valuable approach for management to use. 

Thanks for reading this!

-----------------------------------------------------------------------------

### Go back to my profile page
[garth-c profile page] (https://github.com/garth-c)

---------------------------------------------------------------


