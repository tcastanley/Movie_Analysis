# README
## Introduction
**These days when a new movie is being released, it isn't a question of IF it will be released in another country, but rather HOW MANY other countries? The question we aim to analyze for Microsoft Entertainment Studios is: Are movies that are released in more countries more profitable?**

**The data being analyzed are from two sources: IMDB country data per movie and Box Office Mojo revenue and budget information.**

## Notebook Outline
### 1. Import packages and set SQL cursor
#### --Adjust View Space
### 2. Bring in Data From SQL Database
#### --Creating Countries per Movie DF
#### --Creating Movie Budgets/Revenues DF
### 3. Data Cleaning & Converting
#### --Country Releases per Movie DF
#### --Domestic & International Budgets/Revenues DF
### 4. Exploration, Feature Engineering, and Visualisations
#### --Bringing The Data Together (.merge())
#### --Adding More Metrics
##### -----Foreign Gross
##### -----Net Revenue
##### -----Return on Investment
##### -----Profit/Loss Feature
##### -----Country Bins Feature
#### --Pairplot - Initial Visualization
#### --Boxplot - Outlier Identification
#### --Function - Quantile Column Cleaning
#### --Quantile Column Cleaning (Removed Rows)
#### --Boxplot - Box Office Numbers w/ Outlier Manipulation
#### --Countplot - Countries Per Movie w/ Outlier Manipulation
### 5. Microsoft's Question - Visualization
### 6. Conclusion
### 7. Future Work Recommendations
### 8. --BONUS-- Country Count Estimator

## Findings
**This analysis demonstrated a positive corellation between increased profitability in movies relative to the number 
of countries they are released in.**

**We can also make some conclusions about the risks involved in making movies that are meant to be for a global audience,
as our findings show that it's still possible to spend a lot of money in production and not make it back.**


## Future Work
**To take this analysis further, we would like to have access to more accurate information about how movies performed in 
each country. For example, if we had a how these international releases were timed, a lot of work could be done in 
determining the best time to release a movie and where.**

**Additionally, more detailed information per country regarding the distribution of the revenue per movie would have 
been very insightful.  This could have led to observations surrounding each country and their movie consumption patterns 
and preferences.**

# Sample Visualizations
# Pairplot
![png](README_files/README_81_1.png)
# BoxPlot
![png](README_files/README_97_1.png)
# Distribution Plot
![png](README_files/README_100_0.png)