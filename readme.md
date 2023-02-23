# Analysis of the Egyptian Job Market for Data Professionals
#### Author: **Ziad Zakaria** 
#### Date: **23/1/2023**
#

## What is this?
this project focuses on analyzing the Egyptian Job Market for data professionals to get insights that might help the data roles seekers in their decision and what skills they should improve to be ready for the market. the project's data is fetched by:     
- My last web scrapping script [Wuzzuf Web Scrapper](https://github.com/ziad0x0f/Wuzzuf-Web-Scraper) 
- Tech scene survey
 [Egypt's Tech Scene 2022 (Part 1 - Salaries)](https://lookerstudio.google.com/reporting/fc89c7a2-5dd9-4954-afc3-3ea7f3c7241a/page/DUS6C)

*What is Tech scene survey?*

There is a team of engineers who conducted a survey in which 1,300 male/female engineers participated in the period between August to October 2022 about salaries for different job stages or different years of experience.

They already published a salary report for technical workers.
The team has published a [dashboard](https://egyptscene.tech) in which you can choose your field specifically, such as Backend - Data Analyst - Data Scientist and others, and choose either the career stage or years of experience, from which you can know people at the same level as you in the Egyptian technical market and their salaries approximately.

The team wrote a report on the Egyptian technical market, such as the ratio of females to men, remote work policies, and others.
You can read the report in [Arabic](https://lnkd.in/eNhtZMF2)
Or in [English](https://lnkd.in/ewYMsAQJ).

This report is for employees of technical companies only, and the data in the custody of the work team that conducted the questionnaire and reports.

All appreciation and respect goes to the Egypt's Tech scene Team for providing their data to the public without any charges.
#

#### The project follows the six step data analysis process: ####

* [Ask](#1-ask)
* [Prepare](#2-prepare)
* [Process](#3-process)
* [Analyze](#4-analyze)
* [Share](#5-share)
* [Act](#6-act)
#



## 1. Ask
:red_circle: **GOAL: Analyze the market's data to gain insight and help data enthusiastes in their decision and what skills they should improve.**


## 2. Prepare 
Data Source: 

- [Wuzzuf](https://wuzzuf.net/jobs/egypt)
- [Egypt's Tech Scene 2022](https://lookerstudio.google.com/reporting/fc89c7a2-5dd9-4954-afc3-3ea7f3c7241a/page/DUS6C)


The dataset does follow the **ROCCC** approach:
- Reliability: The can be considered as **Reliable** becouse it produces nearly 41 records of data from Wuzzuf & 90 from Egypt's Tech Scene which is enough to represent the population according to the central limit theorem.  

- Original: The data is **Original** since it is collected from wuzzuf's website directly through web scraping & survey from the egyptian workers.
- Comprehensive: the data is **Not Compehensive**  since the data that is collected from Egypt's Tech Scene is biased toward the network and the companies the data provider interested in.
- Current:  the scraped data from wuzzuf is *current* but the data from Egypt's Tech Scene is not *current* so, the overall data can be considered as **Not Current**
- Cited: the data is **Cited**, it is collected through [Wuzzuf](https://wuzzuf.net/jobs/egypt) & [Egypt's Tech Scene 2022](https://lookerstudio.google.com/reporting/fc89c7a2-5dd9-4954-afc3-3ea7f3c7241a/page/DUS6C).
#

## 3. Process 
importing the libraries 
```python
import re
import statistics as stat
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import seaborn as sns
# scrapping_with_selenium: past project which scrapes data from wuzzuf(check it in my past repo)
import scrapping_with_selenium as sc
```
after scraping the data using scraper function, i have saved the pandas dataframe to excel sheet to analyze it instead of scraping the webpages every time for analysis
```python
# the function which scrapes the data for the firt 150
jobs = ["analyst",
        "Data analysis",
        "Data Analyst",
        "Business Analyst",
        "Data Engineer",
        "Data Scientist",
        "Bi developer",
        "Business Intelligence"]

table = sc.scraper("Data", 150, jobs, 0.7)
```
*Data frame head image*

### **Data Cleaning**
I kept only the job titles i am intersted in, then inserted them to their respected categories

```python
df2 = df.copy()
df2 = df2[
    df2["job_title"].str.contains("Data") |
    df2["job_title"].str.contains("Analyst") |
    df2["job_title"].str.contains("BI") |
    df2["job_title"].str.contains("Business Intelligence")
]
```
```python
# create a list of our conditions
conditions = [
    df2["job_title"].str.contains(
        "Analyst") | df2["job_title"].str.contains("Analysis"),
    df2["job_title"].str.contains("BI"),
    df2["job_title"].str.contains("Engineer"),
    df2["job_title"].str.contains("Scientist")
]

# create a list of the values we want to assign for each condition
values = ['data analyst', 'bi developer', 'data engineer', 'data scientist']

# create a new column and use np.select to assign values to it using our lists as arguments
df2['roles'] = np.select(conditions, values)
df2 = df2[df2.roles.isin(values) == True]

```
the next lines of code acheive the following:

- extracting cities only from the location column
- cleaning companies names
- changing experience column from this (· 2 - 4 Yrs of Exp
) to this (2-4)
- categorizing every role to their experience level according to their respected years of experience in a new **levels** column

```python
# extract cities only
df2["location"] = df2["location"].map(lambda x: x.split(',')[0])

# clean companies names & extract experience range
df2["company"] = df2["company"].str.replace(" -", "")
df2["experience"] = df2["experience"].str.replace(" ", "")
df2["experience"] = df2["experience"].str.findall(".(.+?)Y")
df2["experience"] = [''.join(map(str, l)) for l in df2["experience"]]
```
```python
conditions = [
    df2["experience"].str.contains("^0.*"),
    df2["experience"].str.contains("^1.*"),
    df2["experience"].str.contains("^[23].*"),
    df2["experience"].str.contains("^[45].*"),
    df2["experience"].str.contains("^[67].*")
]

values = ['entry level', 'junior', 'mid level', 'senior', 'team lead']

df2['levels'] = np.select(conditions, values)
```
*after cleaning image*

there are three problems with the salaries column in the dataset:

1. many companies didn't mention their jobs salaries and just wrote that it is *confidential*. even their is some companies wrote *male* or *female* in the salary section.

2. some companies mention a range for their salary insted of single number.

3. too few companies provided their salaries as a single number

I have done the following to fix this issues:

1. i have replaced the confidemtial values with a single number based on the role & the experience level with the aid of the Egypt's Tech Scene data.

2. to make the data consistent, using regular expression methodology in python, i have extracted the two numbers in the salary range provided by the companies then replaced with their mean value.

```python
def clean_salaries(df, rol, lst):

    conds = [
        (df['roles'] == rol) &
        ((df["salaries"].str.contains("Confidential")) | (df["salaries"].str.contains("Male")) | (df["salaries"].str.contains("Females"))) &
        (df['levels'] == 'entry level'),

        (df['roles'] == rol) &
        ((df["salaries"].str.contains("Confidential")) | (df["salaries"].str.contains("Male")) | (df["salaries"].str.contains("Females"))) &
        (df['levels'] == 'junior'),

        (df['roles'] == rol) &
        ((df["salaries"].str.contains("Confidential")) | (df["salaries"].str.contains("Male")) | (df["salaries"].str.contains("Females"))) &
        (df['levels'] == 'mid level'),

        (df['roles'] == rol) &
        ((df["salaries"].str.contains("Confidential")) | (df["salaries"].str.contains("Male")) | (df["salaries"].str.contains("Females"))) &
        (df['levels'] == 'senior'),

        (df['roles'] == rol) &
        ((df["salaries"].str.contains("Confidential")) | (df["salaries"].str.contains("Male")) | (df["salaries"].str.contains("Females"))) &
        (df['levels'] == 'team lead')
    ]

    for _ in range(len(conds)):
        df['salaries'] = np.where(conds[_], lst[_], df["salaries"])

```
```python
# replace confidential with avg market salaries lists
sal = {
    "data analyst": [6000, 8000, 10950, 28500, 28500],
    "bi developer": [5625, 8550, 10000, 15000, 20000],
    "data engineer": [9000, 10375, 23000, 30000, 47000],
    "data scientist": [9400, 11000, 19200, 39584, 67815]
}

for i in sal:
    clean_salaries(df2, i, sal[i])
```
```python
# replace ("number" To "number") expression, to only mean value
df2["salaries"] = df2["salaries"].apply(lambda salary: re.findall(
    (r"\d+"), salary) if type(salary) == str else salary)

df2["salaries"] = df2["salaries"].apply(lambda salary: list(
    map(int, salary)) if type(salary) == list else salary)

df2["salaries"] = df2["salaries"].apply(
    lambda salary: stat.mean(salary) if type(salary) == list else salary)

df2["salaries"] = df2["salaries"].round()
```
the skills is not consistent as it has many repeated skills that resembels the same meaning but different spelling. like Excel & Microsoft Excel, sql server and Microsoft SQL Server, ...etc. so i have combined those repeated values using regular expressions and only kept the skill set that we are interested in within a new dataframe.

