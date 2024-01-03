# Wealthy Households Pre and Post-Pandemic

## Table of Contents

- [Case Study Overview](#case-study-overview)
- [The Question](#the-question)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning/Prepartion](#data-cleaningpreparation)
- [Limitations](#limitations)
- [Data Analysis](#data-analysis)
- [Results/Findings](#resultsfindings)
- [Next Steps](#next-steps)

### Case Study Overview

The Covid-19 Pandemic caused many shifts in the way Americans live and work including *where* they live and work. One such shift that we've heard much about anecdotally is the shift from large, expensive urban metros to smaller metros and more 
rural areas. Stories are abound of wealthy implants from far away states snapping up properties in smaller, more traditionally rural areas and driving up prices. My personal curiousity got the better of me and I wanted to know if the data supported this.

### The Question

The question I wanted to ask and answer is the following: **Since the pandemic, where are wealthy people moving to and from?**

### Data Sources

- Since the question I'm trying to answer is demographically related, I sought and sourced the data from the [US Census Bureau](https://data.census.gov/). 
  
- I'm comparing pre and post-pandemic data about wealthy people and where they are moving. So, I decided to define those parameteras as follows:
  - Pre-Pandemic = 2018 (2 years prior)
  - Post-Pandemic = 2022 (2 years on)
  - Wealthy people = Households with a median annual income exceeding 6-figures ($100,000). Upon further analysis, I later revised this to "ultra-wealthy" (houeholds where median annual income exceeds $200,000).
  - Location: Here I was torn between zip code and county. upon research, I felt that zip code is not the best representaion of demographics and I went with county as location.
  - In order to get more accurate demographic data, I used the ACS 5-year estimates for 2018 and 2022 instead of the 1 year estimates.
 
- I downloaded the data tables from the Census Bureau as .csv files.

### Tools

- [Excel](https://www.office.com/?auth=1): Data Cleaning, Data Analyis
- [MySQL](https://www.mysql.com/): Data Cleaning, Data Merging, Data Analysis
- [Tableau Public](https://public.tableau.com/app/discover): Creating a dashboard

### Data Cleaning/Preparation
In the initial data preparation phase, I performed the following tasks:
1. Data loading and inspection.
2. Data cleaning and formating
3. Data merging

In Excel I removed non-relevant columns, cleaned column names, checked for duplicates, formatted the percentages, and calculated the number of 6-figure households in each county for both the 2018 and 2022 datasets. I then uploaded 
the datasets to MySQL for further analysis. 

In MySQL, I ran the following query to calculate the number of "ultra wealthy" $200,000+ median income households for each county:
```sql
SELECT 
	geo_code, 
	county, 
        total_households, 
       	households_200k_or_more, 
        round((total_households * households_200k_or_more),0) as ultra_wealthy_households
FROM
	wealthy_households2018;
```

### Limitations

I faced several limitations with the data. First, with the data I had, it was impossible to isolate the change in the number of wealthy households for each county to reasons related to relocation. Perhaps, the existing residents saw an increase in their incomes. With this in mind, this analysis is meant only to be a starting point for further analysis. 

Secondly, I discovered that high post-pandemic inflation could skew the results. If you think aobut it, a 6-figure income household is just two $50K earners and in the post-pandemic inflationary environment, that doesn't seem to be worthy of the label "wealthy". Initial analysis did reveal that the percentage of 6-figure income households in the counties of the United States was significantly higher than I expected. This led me to further segment the wealthy households into ultra-wealthy, $200K+. Very few people would dispute the label of "wealthy" to this level of household income and the data shows that this is a much more exclusive demographic. 

### Data Analysis 

In order to compare between pre and post-pandemic, I had to combine the two datasets.  In SQL, I used the   `JOIN` fuction to do this as illustrated in the following query:
```sql
CREATE TABLE
	before_and_after AS
SELECT
	a.geo_code, a.county, 
    	a.total_households as total_households_2018,
    	b.total_households as total_households_2022,
    	b.total_households - a.total_households as total_diff,
	round(((b.total_households/a.total_households)-1),3) as percent_change,
    	a.percent_wealthy_households as wealth_percent_18,
    	b.percent_wealthy_households as wealth_percent_22,
    	a.total_wealthy_households as total_wealthy_2018,
    	b.total_wealthy_households as total_wealthy_2022,
    	b.total_wealthy_households - a.total_wealthy_households as wealthy_diff,
    	round(((b.total_wealthy_households/a.total_wealthy_households)-1),3) as wealthy_percent_change
FROM
	wealthy_households2018 as a
JOIN
	wealthy_households2022 as b
ON
	a.geo_code = b.geo_code;
```
I did the same for the more exclusive ultra-wealthy ($200K+):
```sql
CREATE TABLE
	ultra_wealthy_ba AS
SELECT 
	a.geo_code, a.county, 
	a.total_households as total_hh_2018, 
	b.total_households as total_hh_2022, 
	b.total_households - a.total_households as diff,
	a.households_200k_or_more as percent_18, b.households_200k_or_more as percent_22,
	a.ultra_wealthy_households as ultra_wealthy_18, b.ultra_wealthy_households as ultra_wealthy_22,
	b.ultra_wealthy_households - a.ultra_wealthy_households as difference,
	round(((b.ultra_wealthy_households/a.ultra_wealthy_households)-1),2) as percent
FROM
	ultra_wealthy2018 a
JOIN
	ultra_wealthy2022 b
ON
	a.geo_code = b.geo_code;
```
In order to make more sense of this data, I exported the tables back into excel, split the columns for county and state, then uploaded the tables to Tableau to visualize the geo-data. 


### Results/Findings

The analysis results are summarized as follows:

![6-Figures, 18 v 22](https://github.com/carlussery/wealthy_households/assets/153660397/72f334b0-9dfa-4cbd-bbe5-1a815d178834)

                      
1. **6-figure income households increased in prevalence and distribution nationwide post-pandemic:** The general trend was that 6-figure annual median income households increased in and across all significant areas of the country. There was little to no evidence of some regions/states increasing at the expense of other regions/states.
   
![Ultra 18 v 22](https://github.com/carlussery/wealthy_households/assets/153660397/5d9f3b58-092e-4cad-9f4b-ac12ec48cd27)


   
2. **Counties where ultra-high income households comprise 10% or more of total households saw increased clustering in certain areas post-pandemic:** Counties where 10% or more of the households had annual median incomes exceeding $200,000 saw a post-pandemic increase represented by clustering in certain areas. Areas of note include New England north of Boston, the Mountain West,
Central Colorado, South Texas and the Dallas-Fort Worth area, the Pacific Northwest corridor between Portland, OR and Seattle, WA, South Florida, and the Atlanta Metro area. My hunch is that this is due to migration but more analysis is needed to verify this.
![Ultra mountain west](https://github.com/carlussery/wealthy_households/assets/153660397/dbded51a-e8a4-4cd4-809d-f08afffb7648)<cite> *Idaho and Montana (Mountain West), pre and post-pandemic* </cite>

   
3. ***Before the pandemic, a significant number of states that had no counties where ultra-high income households comprised 10% or more of total households. That changed post-pandemic:** In 2018, many states had no counties with a high concentration (10% or more) of households where the annual median income exceeded $200,000. However, in 2022 a number of counties in those states saw a high concentration of ultra-high income households. I suspect that this is due to migration, but more analysis is needed to state such definitively. 
![Ultra 18 none to some](https://github.com/carlussery/wealthy_households/assets/153660397/8fc454d7-2b76-473b-b4e4-3cea020c988c) <cite> *The states that went from zero in 2018 to hero in 2022*</cite>



To view the entire dashboard on Tableau click [here](https://public.tableau.com/app/profile/carl.ussery/viz/WealthyHouseholdsPreandPostPandemic/Ultra18v22#1))

### Next Steps

Based on the results of the analysis I would recommend with following next steps:
- Drill down the root catalysts behind the increase in 6-figure and ultra-high income households in the hotspot areas. 
- Supplement this analysis with accurate real estate data tracking demand and purchases in these areas to isolate migrating households and families. 
