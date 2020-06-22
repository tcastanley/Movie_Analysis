# README 
# Individual Question: Why do some international movies flop?

Name: **Taylor Stanley**

Program Pace: **Full-Time**

Scheduled Project Review: **Wed Jun 24, 2020 11am – 11:45am**

Instructor: **Abhineet Kulkarni**

BLOG POST URL: **https://medium.com/@tcastanley/movie-analysis-why-do-some-international-movies-flop-287db1792cca?sk=aef5c279f461a2868fb17bc2530d6036**

## Introduction - Goals and Objectives
**After exploring our initial group question about the profitability of movies related to the number of countries they
are released in, I was curious to find out more about why some of this movies fail.  Is there a relationship that exists 
between these movies, and if so can it be measured?**

**The data being analyzed are from two sources: IMDB country data per movie and Box Office Mojo revenue and budget information.**

## Notebook Outline
### 1. Import packages and set SQL cursor
#### --Adjust View Space
### 2. Bring in Data From Previously Cleaned CSV Files
### 3. Data Cleaning & Converting
#### --Minor Adjustments for this Analysis
### 4. Data Exploration Analysis & Visualization
#### --A Mean vs. Median Look at Our Data
#### --What is the Distribution of Movies that are Profitable?
#### --What are the Median Net Revenues of each?
#### --Movie Trends Over Time
##### -----Movie Releases Per Year
##### -----Movies Released Between 2000 - 2019
#### --Revenue vs. Production Costs (2000 - 2019)
### 5. Observations & Conclusions
### 6. Future Work Recommendations


## Findings
***We can observe above that the observations for both the data subsets across both y-axis measurements are quite similar.  
There seems to exist a relatively stable balance between costs and revenue for internationally released movies up until around 2008.  
We should also remember that we had a very large increase in information available for movies made post 2010.  
Regardless, we see a shift in the relationship afterwards as the gap between the two widens significantly.**

**So what does this mean?**

**Perhaps the rapid proliferation and improvement of web services around that time meant that production companies could 
reduce costs for their marketing efforts? What if there is a relationship here between the consumption of entertainment 
and fiction in the western world while it was going through the financial crisis of 2008? Or maybe that was the dawn of 
the superhero movie craze. It is worth noting that between 2008-2011 Marvel released their Iron Man 1 & 2, 
The Incredible Hulk, Thor, and Captain America movies. Seeing as the movies in this universe now account for a massive 
market share of the industry, it would not be unreasonable to think that they could have sparked new interest in films 
for an entire generation. Whatever the cause, it seems to have had a global impact as this gap (in Worldwide Gross) 
remained constant for nearly a decade.**

**What does this tell us about why some international movies fail?**

**What we can notice from these last two plots is that we seem to see a production costs value, above which movies tend 
to make a positive return on their investments.  This is the tipping point that we were looking for.  This is by no means 
a conclusive showing, further digging into what is comprised within these costs may provide valuable insights into which 
areas should be streamlined.** 


## Future Work
**Having a much closer look at what happened with data collection techniques, movie industry technological leaps, and 
social media trends from the years 2008 - 2012.  Really breaking down the production budget numbers.**

**More information about the nature of the streaming industry would be an excellent contrast from the perspective of 
production costs.  Does that new business model have a similar relationship as observed above?**

# Sample Visualizations
# Distribution Plot
![png](README_files/README_23_0.png)
# Countplot
![png](README_files/README_30_0.png)
# Lineplot
![png](README_files/README_42_2.png)


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






# README

## Goals and Objectives

We were tasked with analyzing movie data from the likes of IMDb, tmdb (the movie database), and Box Office Mojo, to create actionable insight that Microsoft can use to start a film studio.

Our initial view took note of the production budgets and profits from a list of +5,000 films. Other sets had data regarding the film crew, countries where the films were released, etc.

**Objectives** 
Our clear objectives consisted of answering 2 group questions and 1 individual question per team member. 

Here I will be describing the group questions and my individual question:



## Group Question 2: Does an Academy Award require a certain budget?

This question explores the relationship between film production budgets and one of the highest-level artistic recognitions for a film, the Academy Award. Since Academy Awards nominations are known to help drive subsequent box-office sales (see [this link](https://knowledge.wharton.upenn.edu/article/oscars-economics/#:~:text=%E2%80%9CAccording%20to%20the%20data%2C%20it,the%20movie%2C%E2%80%9D%20he%20said.&text=%E2%80%9CThere's%20no%20evidence%20that%20%5BOscar,%2C%20your%20salary%20goes%20up.%E2%80%9D)), is there any way we can possibly quantify differences between award-nominated films and non-award nominated films? Can we see how much Microsoft would generally need to spend to create an award-winning film?

Our hypothesis is that it costs more money to hire better acting talent, writers, directors, the rest of the professional film crew, special fx etc, which should result in a better chance at an Academy Award. 

Let's try to see if, based on our data, making an Academy Award-worthy film requires a certain budget range.


```python
import pandas as pd
import os
```


### Import data

Let's batch import our data sets to save some time.


```python
from glob import glob
```

```python
csv_files = glob("./zippedData/*.csv.gz")
csv_files
```
    ['./zippedData/imdb.title.crew.csv.gz',
     './zippedData/tmdb.movies.csv.gz',
     './zippedData/imdb.title.akas.csv.gz',
     './zippedData/imdb.title.ratings.csv.gz',
     './zippedData/imdb.name.basics.csv.gz',
     './zippedData/imdb.title.basics.csv.gz',
     './zippedData/tn.movie_budgets.csv.gz',
     './zippedData/bom.movie_gross.csv.gz',
     './zippedData/imdb.title.principals.csv.gz']

```python
type(csv_files)
d = {}
```

```python
for file in csv_files:              #creating a new dictionary for each .csv file
    d[file] = pd.read_csv(file)
```


### Clean file names

Let's clean the file titles, for easier accessibility throughout our EDA, and create their respective dataframes.


```python
csv_files_dict = {}
for filename in csv_files:
    filename_cleaned = os.path.basename(filename).replace(".csv", "").replace(".", "_").replace('_gz','') # cleaning the filenames
    filename_df = pd.read_csv(filename, index_col=0)
    csv_files_dict[filename_cleaned] = filename_df
```

```python
csv_files_dict.keys()
```

    dict_keys(['imdb_title_crew', 'tmdb_movies', 'imdb_title_akas', 'imdb_title_ratings', 'imdb_name_basics', 'imdb_title_basics', 'tn_movie_budgets', 'bom_movie_gross', 'imdb_title_principals'])


```python

```


### Creating SQL tables from DFs


```python
import sqlite3
```

```python
conn = sqlite3.connect("movies_db.sqlite") 
cur = conn.cursor()
```

```python
def create_sql_table_from_df(df, name, conn):     #batch-creating sql tables from multiple dataframes, with a for loop.
    try:
        df.to_sql(name, conn)
        print(f"Created table {name}")
    
    except Exception as e:
        print(f"could not make table {name}")
        print(e)
```


```python
for name, table in csv_files_dict.items():
    create_sql_table_from_df(table, name, conn)
```

 
```python
cur.execute("select name from sqlite_master where type='table';").fetchall()
```


    [('imdb_title_crew',),
     ('tmdb_movies',),
     ('imdb_title_akas',),
     ('imdb_title_ratings',),
     ('imdb_name_basics',),
     ('imdb_title_basics',),
     ('tn_movie_budgets',),
     ('bom_movie_gross',),
     ('imdb_title_principals',),
     ('films_by_awards.csv',),
     ('films_by_awards',),
     ('films_by_awards1',),
     ('films_by_awards2',),
     ('tn_movie_budgets2',),
     ('tn_movie_budgets_clean',)]



For this question, we will be starting with the tn_movie_budgets dataframe, as this dataframe has the higher number of films with monetary information. 


```python
tn_movie_budgets_df = csv_files_dict['tn_movie_budgets']
```

```python
tn_movie_budgets_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 5782 entries, 1 to 82
    Data columns (total 5 columns):
    release_date         5782 non-null object
    movie                5782 non-null object
    production_budget    5782 non-null object
    domestic_gross       5782 non-null object
    worldwide_gross      5782 non-null object
    dtypes: object(5)
    memory usage: 271.0+ KB



```python

#Checking for null values:

tn_movie_budgets_df.isna().sum()
```

    release_date         0
    movie                0
    production_budget    0
    domestic_gross       0
    worldwide_gross      0
    dtype: int64



### Data Cleaning

Let's create a separate column in tn_movie_budgets for release year only. This will help us merge our scraped wikipedia table.


```python
tn_movie_budgets_df['year'] =  pd.DatetimeIndex(tn_movie_budgets_df['release_date']).year
```


Now let's use a function to clean up our monetary columns, so that we can simply calculate the worldwide profit of each film.


```python
def convert_amt_to_int(df, col):
    df[col] = df[col].str.replace("$", "").str.replace(",", "").astype('int')       #replacing unwanted characters
    return df
```
```python
money_cols = ['production_budget', 'domestic_gross', 'worldwide_gross']

for col in money_cols:
    tn_movie_budgets_df = convert_amt_to_int(tn_movie_budgets_df, col)
```


```python
tn_movie_budgets_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 5782 entries, 1 to 82
    Data columns (total 6 columns):
    release_date         5782 non-null object
    movie                5782 non-null object
    production_budget    5782 non-null int64
    domestic_gross       5782 non-null int64
    worldwide_gross      5782 non-null int64
    year                 5782 non-null int64
    dtypes: int64(4), object(2)
    memory usage: 316.2+ KB


Checking values to make sure they are clean


```python
for col in tn_movie_budgets_df:
    print(f'Viewing values in col: {col}')
    print(f'Top 5 values:\n{tn_movie_budgets_df[col].value_counts(normalize = True)[:5]}')
    print("-------------------")
```

    Viewing values in col: release_date
    Top 5 values:
    Dec 31, 2014    0.004151
    Dec 31, 2015    0.003978
    Dec 31, 2010    0.002594
    Dec 31, 2008    0.002421
    Dec 31, 2009    0.002248
    Name: release_date, dtype: float64
    -------------------
    Viewing values in col: movie
    Top 5 values:
    King Kong       0.000519
    Home            0.000519
    Halloween       0.000519
    Poltergeist     0.000346
    Total Recall    0.000346
    Name: movie, dtype: float64
    -------------------
    Viewing values in col: production_budget
    Top 5 values:
    20000000    0.039952
    10000000    0.036666
    30000000    0.030612
    15000000    0.029920
    25000000    0.029575
    Name: production_budget, dtype: float64
    -------------------
    Viewing values in col: domestic_gross
    Top 5 values:
    0           0.094777
    8000000     0.001557
    2000000     0.001211
    7000000     0.001211
    10000000    0.001038
    Name: domestic_gross, dtype: float64
    -------------------
    Viewing values in col: worldwide_gross
    Top 5 values:
    0          0.063473
    8000000    0.001557
    7000000    0.001038
    2000000    0.001038
    4000000    0.000692
    Name: worldwide_gross, dtype: float64
    -------------------
    Viewing values in col: year
    Top 5 values:
    2015    0.058457
    2010    0.047388
    2008    0.045659
    2006    0.044967
    2014    0.044102
    Name: year, dtype: float64
    -------------------


Making a new columns for worldwide gross profit. Maybe we can test award winners against profit later!


```python
tn_movie_budgets_df['budget_gross_profit'] = tn_movie_budgets_df['worldwide_gross'] - tn_movie_budgets_df['production_budget']

```

```python
create_sql_table_from_df(tn_movie_budgets_df, 'tn_movie_budgets_clean', conn)
```



### Importing scraped Academy Award table

Let's import the dataframe of our scraped Academy-Award winning films, and clean up the column names.


```python
df2 = pd.read_csv('films_by_awards.csv')
df2.rename(columns = {"Film\n": "film", "Awards\n": "awards", "Nominations\n": "nominations", "Year\n": "year"}, inplace=True)
df2
create_sql_table_from_df(df2, 'films_by_awards2', conn)
```


```python
df2.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1316 entries, 0 to 1315
    Data columns (total 5 columns):
    Unnamed: 0     1316 non-null int64
    film           1316 non-null object
    year           1316 non-null object
    awards         1316 non-null object
    nominations    1316 non-null object
    dtypes: int64(1), object(4)
    memory usage: 51.5+ KB


### Joining separate data tables

Let's join our Academy Award winner data to our table of gross and profit data.


```python
cur.execute('''SELECT b.*, awards, nominations
                FROM tn_movie_budgets_clean b
                LEFT JOIN films_by_awards2 a
                ON a.film = b.movie
                AND a.year = b.year                                
                ORDER BY budget_gross_profit DESC                  
                ''')                            # we needed to join our scraped table on two conditions, because some 
                                                #films were created on multiple years with the same name.            
awards_to_budget_df = pd.DataFrame(cur.fetchall())
awards_to_budget_df.columns = [x[0] for x in cur.description]
awards_to_budget_df.head()
```





```python
import seaborn as sns
import matplotlib.pyplot as plt
%matplotlib inline
```

### Further cleaning of new data


```python
awards_to_budget_df['awards'].unique()
```


    array(['3', '11', None, '2', '1', '4', '6', '7', '5', '8 (2)', '8',
           '4 (1)', '0 (1)', '1 (1)', '9', '0 (2)', '10', '5 (1)', '7 (1)'],
          dtype=object)



```python
awards_to_budget_df['nominations'].unique()
```



    array(['9', '14', None, '7', '2', '11', '3', '5', '4', '1', '6', '8',
           '13', '10', '12', '10[4]\n', '0', '6[11]\n'], dtype=object)



Let's remove any extra characters included from the Wikipedia table, to leave only the number of competitive awards for each film. Let's also convert any null values to 0, for graphing purposes later on.


```python
def clean_vals(df, col):
    df[col] = df[col].str.rstrip('\n').str.replace("\(.*\)", "").fillna("0").astype('int')
    return df
```


```python
academy_award_cols = ['awards']

for col in academy_award_cols:
    awards_to_budget_df = clean_vals(awards_to_budget_df, col)
```


```python
awards_to_budget_df['awards'].unique()
```


    array([ 3, 11,  0,  2,  1,  4,  6,  7,  5,  8,  9, 10])


```python
awards_to_budget_df['nominations'].unique()
```

    array(['9', '14', None, '7', '2', '11', '3', '5', '4', '1', '6', '8',
           '13', '10', '12', '10[4]\n', '0', '6[11]\n'], dtype=object)


Cleaning the nominations column. This took quite a bit of tries as we tried to separate the "\n" first and then the bracketed footnote. Eventually we stepped back onto a simple but effective solution of dividing the string at the cutoff of our desired value, which was the inner bracket "["



```python
'10[4]\n'.split('[')
```

    ['10', '4]\n']


```python
value_split = awards_to_budget_df['nominations'].str.split('[').fillna("0")
```


```python
type(value_split)
```


    pandas.core.series.Series


```python
(value_split[0][0])
```

    '9'

Reassigning the cleaned value back to the 'nominations' column


```python
awards_to_budget_df['nominations'] = value_split.str.get(0).astype('int')
```


```python
awards_to_budget_df['nominations'].unique()
```

    array([ 9, 14,  0,  7,  2, 11,  3,  5,  4,  1,  6,  8, 13, 10, 12])



### Cleaning Out Movies that Grossed 0 dollars

Remove movies which have not been released yet, or made absolutely no money whatsoever.


```python
current_movies = awards_to_budget_df[awards_to_budget_df['worldwide_gross'] == 0]

```

```python
gross_index = awards_to_budget_df[awards_to_budget_df['worldwide_gross'] == 0].index
awards_to_budget_df.drop(gross_index, inplace=True)
```


```python
awards_to_budget_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 5415 entries, 0 to 5781
    Data columns (total 10 columns):
    id                     5415 non-null int64
    release_date           5415 non-null object
    movie                  5415 non-null object
    production_budget      5415 non-null int64
    domestic_gross         5415 non-null int64
    worldwide_gross        5415 non-null int64
    year                   5415 non-null int64
    budget_gross_profit    5415 non-null int64
    awards                 5415 non-null int64
    nominations            5415 non-null int64
    dtypes: int64(8), object(2)
    memory usage: 465.4+ KB


### Nominated or Not nominated?

Let's create a column for the boolean value of nomination (nominated or not).


```python
awards_to_budget_df['nominated'] = (awards_to_budget_df['nominations'] > 0)

```



### Stats Check

Checking summary statistics to get a feel for the distribution of the sample.


```python
awards_to_budget_df.describe()
```


## Exploring Visualizations

Let's try to make sense of our data by exploring some visualizations

### Awards vs Production Budget


```python
awards_to_budget_df.plot.scatter(x='awards', y='production_budget', figsize=(6,6))
plt.xlabel("Number of Awards Won")
plt.ylabel("Production Budget (Hundred Million)")
plt.title("Number of Awards vs Production Budget");

```


![png](README_files/README_78_0.png)


This quick scatter plot shows us that many films with relatively low production budgets were able to not only get nominated but win Academy Awards. 

#### Seaborn Visualizations

Let's make use of seaborn visuals and map a different color to each film based upon the range of release date they fall into.


```python
import seaborn as sns
```


```python
sns.set(style="darkgrid")


# Draw a scatter plot while assigning point colors and sizes to different
# variables in the dataset
f, ax = plt.subplots(figsize=(9, 9))
sns.despine(f, left=True, bottom=True)
sns.scatterplot(x="awards", y="production_budget",
                hue="year",
                hue_order='year',
                sizes=(1, 8), linewidth=0,
                data=awards_to_budget_df, ax=ax)

ax.set_ylabel("Production Budget (Hundred Million)", fontsize=15)
ax.set_xlabel('Number of Awards', fontsize=15)
ax.set_title("Average Production Budget for Academy Award-Winning Films", fontsize=15);
```


![png](README_files/README_82_0.png)


Here we can see that older Academy Award-winning films generally have lower production budgets. Perhaps earlier film companies had less access to capital investments as the film industry took time to grow in popularity.

### Nominations vs Production Budget

In order to have won an Academy Award, a film must first be nominated for that award. Let's explore those numbers.


```python
sns.set(style="darkgrid")


# Draw a scatter plot while assigning point colors and sizes to different
# variables in the dataset
f, ax = plt.subplots(figsize=(9, 9))
sns.despine(f, left=True, bottom=True)
sns.scatterplot(x="nominations", y="production_budget",
                hue="year",
                hue_order='year',
                sizes=(1, 8), linewidth=0,
                data=awards_to_budget_df, ax=ax);

ax.set_ylabel("Production Budget (Hundred Million)", fontsize=15)
ax.set_xlabel('Number of Nominations', fontsize=15)
ax.set_title("Average Production Budget for Academy Award-Nominated Films", fontsize=15);

```


![png](README_files/README_86_0.png)


There are more films for the nomination categories now. The majority of nominated films seem to spend around $200 Million or less.


### Nominated or Not vs Production Budget

Maybe it would be smarter to try a boxplot in order to better understand the distribution of production budgets against whether they were nominated or not. 


```python

f, ax = plt.subplots(figsize=(12, 12))

sns.boxplot(x="nominated", y="production_budget",
             palette=["m", "g"],
            data=awards_to_budget_df)

ax.set_ylabel("Production Budget (Hundred Million)", fontsize=20)
ax.set_xlabel('',)
ax.set(xticklabels=["Not Nominated", "Nominated"])
ax.set_title("Average Budget for Academy Award-Nominated Films", fontsize=20);

sns.despine(offset=10, trim=True)
```


![png](README_files/README_90_0.png)


### Removing furthest outliers from data

The our data is getting crunched by some far-reaching outliers. Let's try removing the the top and bottom 1% of outliers and see if we can get a better idea of the distribution of each category.


```python
budget = awards_to_budget_df['production_budget']
removed_outliers = budget.between(budget.quantile(.01), budget.quantile(.99))
index_names = awards_to_budget_df[~removed_outliers].index                     # "~" is inverting the dataframe
print(index_names)
```

    Int64Index([   0,    2,    3,    4,    6,   13,   17,   19,   28,   29,   37,
                  45,   54,   61,   63,   65,   73,   81,   82,   85,   89,  101,
                 108,  123,  167,  170,  210,  232,  236,  246,  264,  396,  530,
                 633,  729,  883,  916, 1072, 1708, 2277, 2783, 2942, 3161, 3221,
                3292, 3310, 3360, 3383, 3456, 3501, 3518, 3535, 3547, 3559, 3563,
                3565, 3574, 3576, 3589, 3590, 3596, 3601, 3602, 3606, 3609, 3613,
                3616, 3619, 3623, 3630, 3633, 3638, 3660, 3661, 3662, 3664, 3665,
                3666, 3668, 3671, 3675, 3681, 3683, 3689, 3692, 3697, 3700, 3711,
                3717, 3718, 3719, 5341, 5781],
               dtype='int64')



```python

```
awards_to_budget_df.drop(index_names, inplace=True)
```


```python
awards_to_budget_df.describe() 
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>year</th>
      <th>budget_gross_profit</th>
      <th>awards</th>
      <th>nominations</th>
    </tr>
  </thead>

The summary stats tell us that the mean has gone down slightly. The Max production budget is now only 200 Million, more than half of the highest budget of 425 Million  before.


## Final Visualizations

### New Distribution Boxplot


```python
f, ax = plt.subplots(figsize=(12, 12))


sns.boxplot(x="nominated", y="production_budget", palette=["m", "y"],
            data=awards_to_budget_df, showmeans=True, ax=ax)

ax.set(xticklabels=["Not Nominated", "Nominated"])



ax.set_ylabel("Production Budget (Hundred Million)", fontsize=15)
ax.set_xlabel('', fontsize=15)
ax.set_title("Average Production Budget for Academy Award-Nominated Films", fontsize=18);
```


![png](README_files/README_100_0.png)


Comparing the mean value (green triangle) with the median line, our boxplot shows us that the data set does not have a normal distribution, and is positively skewed. 

The average budget is still pulled higher by outliers, although the IQR of production_budgets reaches further for nominated films. Perhaps the concentration of older films on the low end of the budget scale is pulling back the median. 

Because of the skewed distribution, the median might give us the more accurate idea of award-winning production budget. Therefore, our hypothesis can not be confirmed here as *most* nominated films don't necessarily spend more than non-nominated ones.

### Average Budget vs Nomination Bar Plot

Let's try a different visualization of average production budget, and median production budget with a bar plot.


```python
sns.set(style="white", context="talk")


# Set up the matplotlib figure
f, (ax1) = plt.subplots(figsize=(10, 7), sharex=True)

# Generate some sequential data
x = awards_to_budget_df['nominated']
y1 = awards_to_budget_df['production_budget']
sns.barplot(x=x, y=y1, palette="rocket", ax=ax1)
ax1.axhline(0, color="k", clip_on=False)
ax1.set_ylabel("Average Production Budget (Hundred Million)")
ax1.set_xlabel('')
ax1.set(xticklabels=["Not Nominated", "Nominated"])
ax1.set_title("Average Budget for Academy Award-Nominated Films");

from numpy import median                                                 #
x = awards_to_budget_df['nominated']
y1 = awards_to_budget_df['production_budget']
sns.barplot(x=x, y=y1, palette="rocket", ax=ax1, estimator=median)
ax1.axhline(0, color="k", clip_on=False)
ax1.set_ylabel("Average Production Budget (Hundred Million)")
ax1.set_xlabel('')
ax1.set(xticklabels=["Not Nominated", "Nominated"])
ax1.set_title("Average Budget for Academy Award-Nominated Films");
```


![png](README_files/README_104_0.png)


It is easier to see now that the average for production budget is higher for award winning films, however the median budgets are just slightly higher for nominated films. 

### Average Profit vs Nomination

Now let's make a bar plot to see if award-nominated films do in fact make more money.


```python
sns.set(style="white", context="talk")


# Set up the matplotlib figure
f, (ax1) = plt.subplots(figsize=(10, 7), sharex=True)

# Generate some sequential data
x = awards_to_budget_df['nominated']
y1 = awards_to_budget_df['budget_gross_profit']
sns.barplot(x=x, y=y1, palette="rocket", ax=ax1)
ax1.axhline(0, color="k", clip_on=False)
ax1.set_ylabel("Average Profit (Hundred Million)")
ax1.set_xlabel('')
ax1.set(xticklabels=["Not Nominated", "Nominated"])
ax1.set_title("Average Profit for Academy Award-Nominated Films");

x = awards_to_budget_df['nominated']
y1 = awards_to_budget_df['budget_gross_profit']
sns.barplot(x=x, y=y1, palette="rocket", ax=ax1, estimator=median)
ax1.axhline(0, color="k", clip_on=False)
ax1.set_ylabel("Average Profit (Hundred Million)")
ax1.set_xlabel('')
ax1.set(xticklabels=["Not Nominated", "Nominated"])
ax1.set_title("Average Profit for Academy Award-Nominated Films");

```


![png](README_files/README_108_0.png)


This bar plot of average and median profit based on nominations helps us verify that nominated films return much higher profits.

Our bar plots of budget tells us that, on average, award-nominated films made way more money up front in the production phase. This would suggest that things like higher-quality production, renowned film writers, directors and actors is worth spending more money on. However, considering the positively skewed distribution, the median value tells us there is usually no big difference in production budget. It may be worth it to conduct a follow up analysis based only on the most recent films.

What we know for sure is, if a film is nominated for an award, the subsequent profits are then boosted substantially.

Future exploration: Are the budget and gross values adjusted for inflation? Could using only more recent films (within the last 20 years) give us more accurate values or perhaps a more normal distribution?