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

The next step is to review the response data at a high level. I will use a histogram for this that uses 30 groups to display the data.

```
#quick plot of the response variable
windows()
hist(insurance$charges,
     breaks = 30,
     main = 'Histogram of insurance costs')
```




















  
