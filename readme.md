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
![data](https://github.com/ziad0x0f/Analysis-of-the-Egyptian-Market-for-Data-Specialist-Jobs/blob/main/imgs/before_clean.png?raw=true)
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
- grouping giza divsions to giza governorate
- cleaning companies names
- changing experience column from this (· 2 - 4 Yrs of Exp
) to this (2-4)
- categorizing every role to their experience level according to their respected years of experience in a new **levels** column

```python
giza_gov = ["Dokki",
"El-Haram",
"Agouza",
"El Ayyat",
"El Badrashein",
"And El Hawamdeya",
"Giza",
"El Omraniya",
"El Wahat El Bahariya",
"El Warraq",
"Sheikh Zayed",
"Smart Village",
"Zamalek",
"El Saff",
"Atfeh",
"Ossim",
"Bulaq",
"Imbaba",
"Kerdasa",
"6th of October"]

# extract cities only
df2["location"] = df2["location"].map(lambda x: x.split(',')[0])

df2["location"] = df2["location"].apply(lambda loc: "Giza" if loc in giza_gov else loc)

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
![categorizing](https://github.com/ziad0x0f/Analysis-of-the-Egyptian-Market-for-Data-Specialist-Jobs/blob/main/imgs/after_categorizing.png?raw=true)
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
![salary](https://github.com/ziad0x0f/Analysis-of-the-Egyptian-Market-for-Data-Specialist-Jobs/blob/main/imgs/Screenshot%202023-02-23%20192942.png?raw=true)
```python
    df_noskill = df2.drop(columns=["skill"]).drop_duplicates()
```
```python
df_skills = df2.copy()
# continue the list of conditions
conditions = [
    (df_skills["skill"].str.contains("(?i)excel")
     | df_skills["skill"].str.contains("(?i)office")
     | df_skills["skill"].str.contains("(?i)Accounting")),

    (df_skills["skill"].str.contains("(?i)power bi")
     | df_skills["skill"].str.fullmatch("(?i)bi")),

    (df_skills["skill"].str.contains("(?i)sql server")
     | df_skills["skill"].str.fullmatch("(?i)ssis")
     | df_skills["skill"].str.fullmatch("(?i)ssas")
     | df_skills["skill"].str.fullmatch("(?i)ssrs")),

    df_skills["skill"].str.fullmatch("(?i)sql"),

    (df_skills["skill"].str.contains("(?i)computer science")
     | df_skills["skill"].str.fullmatch("(?i)coding")
     | df_skills["skill"].str.fullmatch("(?i)software")),

    df_skills["skill"].str.fullmatch("(?i)Tableau"),

    df_skills["skill"].str.fullmatch("(?i)Statistics"),

    df_skills["skill"].str.fullmatch("(?i)python"),

    df_skills["skill"].str.contains("(?i)spark")
]

values = ['excel', 'power bi', 'sql server', 'sql',
          'programming', 'tableau', 'Statistics', 'python', 'spark']

df_skills['skill'] = np.select(conditions, values)
df_skills = df_skills[df_skills.skill.isin(values) == True]
```
## 4. Analyze

### 1. What is the difference between job oppurtunties for every data role?

out of every 10 job postings there are:

- about 6 open positions for data analyst or bussiness analyst

- 2  open positions for a data engineer

- 1  open position for a Bi analyst

only **4%** of job postings is for data scientist role which is very low compared to other roles

### 2. What role that have the most chances for entry levels & juniors?

- **31.7%** of job openings is for juniors and entry levels -

- **84.6%** of these jobs is for data analysts

- juniors & entry levels have much less oppurtunities as a data engineer or as a data scientist roles 

### 3. top 4 cities with job openings 

- Most of job opportunities is in cairo & giza governorates 

### 4. What is the role with the best salaries?

- the salaries data is not comprehensive, since we only have the team lead level that is present in all the roles. 

- its pretty obvious that the **data engineer & scientist** roles get paid double or even triple the amount compared to the other roles.

### 5. what is the most wanted range of experience in the market?

- most of the vacancies are for the 3+ years of experience employees, representing **65%** of the available opportunities for data professionals.

### 6. What is the role with the best salaries with 3+ exp level?
- data engineer gain almost **30%** higher salary than data analyst & **twice** as much as bi developer

### 7. What are the most common skill tools in jobs description?
- Due to data inconsistency, skills for data engineers and data scientists are very few. as a result, i will only consider skills for data analysts and bi developers

- Suprisingly, that programming is the most wanted skill even for data analysts & powerbi is more in demand than excel

## 5. Share

the detailed code and report is shared on github

## 6. Act

- Most of the available vacancies are for data and business analysts

- If you are shifting from another career, you should consider data analysis role as it has the most open vacancies for entry levels and juniors

- If you are looking for a fulltime from office job, you should relocate to cairo or giza divisions

- After collecting 2 year of experience as a data analyst, i recommend to acquire data engineering and science skills, due to high salary earnings at this experience level

- Develope your programming skills(Sql, DBMS, python or R), since The demand for it is constantly increasing


*sorry for any vocab or grammar mistakes, i believe it is a lot*
 