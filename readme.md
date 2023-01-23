# Analysis of the Egyptian Job Market for Data Professionals
#### Author: **Ziad Zakaria** 
#### Date: **1/23/2023**
#

## What is this?
this project focuses on analyzing the Egyptian Job Market for data professionals to get insights that might help the data roles seekers in their decision and what skills they should improve to be ready for the market. the project's data is fetched by my last web scrapping script [Wuzzuf Web Scrapper.
](https://github.com/ziad0x0f/Wuzzuf-Web-Scraper) 

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
Data Source: [Wuzzuf](https://wuzzuf.net/jobs/egypt)

Dataset made available through [Wuzzuf Web Scrapper
](https://github.com/ziad0x0f/Wuzzuf-Web-Scraper) 

The dataset does follow the **ROCCC** approach:
- Reliability: The can be considered as **Reliable** becouse it produces nearly 130 records of data which is enough to represent the population according to the central limit theorem.  

- Original: The data is **Original** since it is collected from wuzzuf's website directly through web scraping
- Comprehensive: the data is **Not Compehensive** as most of the data is collected during the last month and a half.
- Current:  the data is **Current**. 
- Cited: the data is **Cited**, it is collected through [Wuzzuf](https://wuzzuf.net/jobs/egypt).
#

## 3. Process 
importing the libraries 
```
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import seaborn as sns
import scrapping_with_selenium as sc  # scrapping_with_selenium: past project which scrapes data from wuzzuf(check it in my past repo)
```
after scraping the data using scraper function, i have saved the pandas dataframe to excel sheet to analyze it instead of, scraping the webpages every time for analysis
```
# the function which scrapes the data for the firt 150
df = sc.scraper("Data", 150, ["Data Engineer","Analyst", "Data Scientist", "Bi", "Business Intelligence","Data Entry"])
```
*Data frame head image*

### **Data Cleaning**
I kept only the job titles i am intersted in, then inserted them to their respected categories

```
table= table[
table["job title"].str.contains("Data") |
table["job title"].str.contains("Analyst") |
table["job title"].str.contains("BI")
] 
```
```
# create a list of our conditions
conditions = [
table["job title"].str.contains("entry"),
table["job title"].str.contains("Entry"),
table["job title"].str.contains("Analyst"),
table["job title"].str.contains("BI"),
table["job title"].str.contains("Engineer"),
table["job title"].str.contains("Scientist")
]

# create a list of the values we want to assign for each condition
values = ['data entry', 'data entry', 'data analyst', 'bi developer','data engineer', 'data scientist']

# create a new column and use np.select to assign values to it using our lists as arguments
table['roles'] = np.select(conditions, values)

# display updated DataFrame
table.head()
```