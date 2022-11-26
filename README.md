# Survey Data Analysis

***Access Kaggle Notebook [Here](https://www.kaggle.com/code/tyroneogarrojr/survey-data-analysis/notebook)***

## Abstract

Students in a school network completed a survey about students' learning experience to identify any impediments that may effect their ability to learn to help the school network better serve its students.

## Objective
The objective of the analysis is to compare scores and the overall sentiment of students who recieve an services oraccomodations with students who do not recieve services or accommodations at each campus. 

## Variables

There are several variables that will have a significant effect on the outcome of the analysis:

- Grade: Student grade level
- Campus: Campus the student attends in the school network
    - PS: Grades 0-2
    - IS: Grades 3-5 
    - MS: Grades 6-8
    - HS: Grades 9-12
- Score: Survey result on a 100 point scale
- Sentiment: Interpretation of survey result
    - High Satisfaction: Score is greater 31
    - Moderate Satisfaction: Score is between 21 and 30
    - Low Moderate Satisfaction: Score is between 6 and 20
    - Low Satisfaction: Score is less than 6
        - A 'Low Satisfaction' Sentiment measure prompts intervention on behalf of school administration.
- Student Accomodations:
    - IEP: Intellectual disability.
    - 504: Behavioral disability.
    - ESL/ELL: English as a Second Language or English Language Learner.
    - SPED: Students who recieve an educational accomodation or service,e.g IEP, 504, ESL/ELL.
    
    
## Questions 

This analysis will be guided by the following questions:

1. How many students completed the survey?
2. How many students have accommodations?
3. What is the average score and sentiment of the surevy results of students with accommadations compared to students without accommodations?

## Importing Python Packages and CSV

``` python
#Import packages
import pandas as pd 
import numpy as np 
import matplotlib.pyplot as plt
import pandasql as sql

#Import datasets
df = pd.read_csv('../input/pass-results/PASS_Survey_Data_Repository - Campus_Data_Aggregate_Anonymous (2).csv')

df1 = pd.read_csv('../input/pass-results/PASS_Survey_Data_Repository - Demographic Information_Anonymous (1).csv')

#Creating new column - Campus 
conditions = [(df['Grade']<=2), (df['Grade']>=3) & (df['Grade']<6), (df['Grade']>=6) & (df['Grade']< 9), (df['Grade']>=9)]
values = ['PS', 'IS','MS','HS']

#Adding new column - Campus 
df['Campus'] = np.select(conditions,values)

#Creating new column - Sentiment
conditions1 = [(df['Score']>= 31), (df['Score']>=21) & (df['Score']< 31), (df['Score']>=6) & (df['Score']<21), (df['Score']<6)]
values1 = ['High Satisfaction', 'Moderate Satisfaction', 'Low Moderate Satisfaction', 'Low Satisfaction']

#Adding new column - Sentiment
df['Sentiment'] = np.select(conditions1,values1)

```

# Analysis

1. **How many students completed the survey?**

In total 898 students completed the school survey. The HS campus has 247 students,the largest student population of all campuses



 Total number of students that completed the survey.
``` python 
df_total = sql.sqldf('SELECT COUNT(upn) as Total FROM df')
df_total.head()

```


Number of students that completed survey at each campus.
``` python 

df_campus = sql.sqldf('SELECT Campus,COUNT(upn) as Campus_Count FROM df GROUP BY Campus ORDER BY Grade')

df_campus.head()
```

Horizontal bar graph of campus population
``` python 
df_campus.plot(kind = 'barh', x = 'Campus', y = 'Campus_Count',title = 'Campus Population', color = ['red','blue','green','gray'], figsize = (18,6), ylabel = 'Campus Count')

```

2. **How many students recieve accomodations?**

169 students recieve services or accommodations and 733 students recieve do not recieve an accommodation. The HS campus has the 58 students that recieve accommodations, the most of all campuses.


Total number of students with accommodations.
``` python 
df_SPED_Cnt_Campus = sql.sqldf('SELECT df1."SPED",COUNT(df."UPN") as SPED_Count FROM df JOIN df1 ON df1."Student ID" = df."UPN" GROUP BY df1."SPED"')

df_SPED_Cnt_Campus.head(13)
```

Horizontal bar graph comparing students who recieve accommodations or services with students who do not recieve accommodations or services. 

``` python 
df_SPED_Cnt_Campus.plot(kind = 'barh', x = 'SPED', y= 'SPED_Count', title = 'Accommodation Count Comparison', color = ['red', 'blue'],figsize = (18,6))
```

Number of students with accommodations at each campus.

``` python 
df_SPED = sql.sqldf('SELECT df."Campus",df1."SPED", COUNT(df."UPN") as SPEDCount FROM df JOIN df1 ON df1."Student ID" = df."UPN" GROUP BY df."Campus", df1."SPED"')

df_SPED.head(9)

```

Grouped bar graph of campus student population with and without accommodations

```python 
df_SPED.plot(kind = 'bar', x = 'Campus', y = 'SPEDCount',ylabel = 'SPED Count',color = ['red', 'blue'], edgecolor = ['black'], title = 'SPED v No SPED Student Comparison', figsize = (18,6))
```

3. **What is the average score and sentiment of the surevy results of students that recieve accommadations compared to students without accommodations?**


Overall there is very little difference in the average scores of students that recieve services or accommodations, 40, compared to students who do not recieve services or accommodations, 39. However, there is a slight difference in the average score of students that don't recieve services or accommodations at the PS campus,58, and students that recieve services or accommodations at the PS campus, 53. Additionally,there is a noticable difference in the average score of students that don't recieve services or accommodations at the HS campus, 40, and students that recieve services or accommodations at the HS campus, 32. Lastly,there is a large difference in the average score students that don't recieve services or accommodations at the IS campus, 26, compared to students that recieve services or accommodations at the IS campus, 43.

``` python df_SPED_comp = sql.sqldf('SELECT df1."SPED",ROUND(AVG(df."Score"),0) as Avg_Score FROM df JOIN df1 ON df1."Student ID" = df."UPN" GROUP BY df1."SPED" ORDER BY Avg_Score DESC')

conditions = [(df_SPED_comp['Avg_Score']>= 31), (df_SPED_comp['Avg_Score']>=21) & (df_SPED_comp['Avg_Score']< 31), (df_SPED_comp['Avg_Score']>=6) & (df_SPED_comp['Avg_Score']<21), (df_SPED_comp['Avg_Score']<6)]
values = ['High Satisfaction', 'Moderate Satisfaction', 'Low Moderate Satisfaction', 'Low Satisfaction']

df_SPED_comp['Sentiment'] = np.select(conditions,values)

df_SPED_comp.head()

```

Horizontal bar graph comparing the average scores of students who do and do not recieve services or accommodations.

``` python 
df_SPED_comp.plot(kind = 'barh', x = 'SPED', y = 'Avg_Score', title = 'SPED v Gen Ed Survey Score Comparison',figsize= (18,6), color = ['red','blue'])
```

Average campus score and sentiment of students without accommodations 

``` python 
df_campus_average = sql.sqldf('SELECT df."Campus", ROUND(AVG(df."Score"),0) as Avg_Score FROM df JOIN df1 ON df1."Student ID" = df."UPN" WHERE df1."SPED" = "No" GROUP BY df."Campus" ORDER BY df."Grade"')

conditions = [(df_campus_average['Avg_Score']>= 31), (df_campus_average['Avg_Score']>=21) & (df_campus_average['Avg_Score']< 31), (df_campus_average['Avg_Score']>=6) & (df_campus_average['Avg_Score']<21), (df_campus_average['Avg_Score']<6)]
values = ['High Satisfaction', 'Moderate Satisfaction', 'Low Moderate Satisfaction', 'Low Satisfaction']

df_campus_average['Sentiment'] = np.select(conditions, values)

df_campus_average.head()

```

Horizontal bar graph displaying the average scores of students that don't recieve services or accommodations groups by campus.

``` python 
df_campus_average.plot(kind = 'barh', x = 'Campus', y = 'Avg_Score', title = 'Average Score of Students Without Accommodations', figsize = (18,6),  color = ['red','blue','green','gray'])
```

Average campus score and sentiment of students that recieve accommodations.

``` python 
df_SPED_campus = sql.sqldf('SELECT df."Campus", ROUND(AVG(df."Score"),0) as Avg_Score FROM df JOIN df1 ON df1."Student ID" = df."UPN" WHERE df1."SPED" = "Yes" GROUP BY df."Campus" ORDER BY df."Grade" ')

conditions = [(df_SPED_campus['Avg_Score']>= 31), (df_SPED_campus['Avg_Score']>=21) & (df_SPED_campus['Avg_Score']< 31), (df_SPED_campus['Avg_Score']>=6) & (df_SPED_campus['Avg_Score']<21), (df_SPED_campus['Avg_Score']<6)]
values = ['High Satisfaction', 'Moderate Satisfaction', 'Low Moderate Satisfaction', 'Low Satisfaction']


df_SPED_campus['Sentiment'] = np.select(conditions, values)



df_SPED_campus.head()
```

Horizontal bar graph of students that recieve services or accommodations grouped by campus

``` python 
df_SPED_campus.plot(kind = 'barh', x = 'Campus', y = 'Avg_Score', title = 'Average Score of Students With Accommodations',figsize = (18,6),  color = ['red','blue','green','gray'] )
```

# Conclusions

- Overall, there is very little difference in the average scores and sentiments of students who recieve services or accommodations compared to students who do not. However, there are noticable difference in the average scores of students who recieve services and accommodations and students who do not recieve services or accommodations across the school network especially at the IS and HS campuses.

# Suggestions

- School administrators should monitor the student experinece at the IS campus as students without a service or accommodation had the lowest average score, 26, and sentiment, 'Moderate Satisfaction', of all campuses regardless of service or accommodation status. 
- School administrators should monitor the student experience at the HS campus as it is the campus with the largest student population that recieves services or accommodations, and the second largest difference in average score between students that recieve services or accommodations and students that do not recieve services or accommodations.
