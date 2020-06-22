
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


![png](Group_Question_2_Final_Copy1_files/Group_Question_2_Final_Copy1_78_0.png)


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


![png](Group_Question_2_Final_Copy1_files/Group_Question_2_Final_Copy1_82_0.png)


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


![png](Group_Question_2_Final_Copy1_files/Group_Question_2_Final_Copy1_86_0.png)


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


![png](Group_Question_2_Final_Copy1_files/Group_Question_2_Final_Copy1_90_0.png)


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


![png](Group_Question_2_Final_Copy1_files/Group_Question_2_Final_Copy1_100_0.png)


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


![png](Group_Question_2_Final_Copy1_files/Group_Question_2_Final_Copy1_104_0.png)


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


![png](Group_Question_2_Final_Copy1_files/Group_Question_2_Final_Copy1_108_0.png)


This bar plot of average and median profit based on nominations helps us verify that nominated films return much higher profits.

Our bar plots of budget tells us that, on average, award-nominated films made way more money up front in the production phase. This would suggest that things like higher-quality production, renowned film writers, directors and actors is worth spending more money on. However, considering the positively skewed distribution, the median value tells us there is usually no big difference in production budget. It may be worth it to conduct a follow up analysis based only on the most recent films.

What we know for sure is, if a film is nominated for an award, the subsequent profits are then boosted substantially.

Future exploration: Are the budget and gross values adjusted for inflation? Could using only more recent films (within the last 20 years) give us more accurate values or perhaps a more normal distribution?


_______________________________________________________________________________________________________
# README Group Question 1

**These days when a new movie is being released, it isn't a question of IF it will be released in another country, but rather HOW MANY other countries? The question we aim to analyze for Microsoft Entertainment Studios is:**

**Are movies that are released in more countries more profitable?**

**The findings in this notebook will show that the answer to that question is in fact: yes. This notebook will walk step-by-step through the importing, cleaning, exploration, and visualization processes undertaken to try and answer Microsoft's question and help inform strategic business decisions going forward.**

**The data being analyzed are from two sources: IMDB country data per movie and Box Office Mojo revenue and budget information. These datasets have been saved into an SQL database previously and will be imported into this notebook further down.**

# Import packages and set SQL cursor


```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import sqlite3
import os
%matplotlib inline

conn = sqlite3.connect('movies_db.sqlite')
cur = conn.cursor()
```

## Adjust view space


```python
pd.set_option('display.max_rows', 500)
pd.set_option('display.max_columns', 500)
pd.set_option('display.width', 1000)
```

# Bring in data from our SQL database

## Creating our Countries per Movie DataFrame


```python
conn.execute("select name from sqlite_master where type='table';").fetchall()
```




    [('bom_movie_gross',),
     ('imdb_name_basics',),
     ('imdb_title_akas',),
     ('imdb_title_basics',),
     ('imdb_title_crew',),
     ('imdb_title_principals',),
     ('imdb_title_ratings',),
     ('tmdb_movies',),
     ('tn_movie_budgets',)]




```python
cur.execute('''SELECT primary_title, region
                FROM imdb_title_akas a
                JOIN imdb_title_basics b
                ON a.title_id = b.tconst
                ;''')
df_title_region = pd.DataFrame(cur.fetchall())
df_title_region.columns = [x[0] for x in cur.description]
df_title_region.head()
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
      <th>primary_title</th>
      <th>region</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sunghursh</td>
      <td>IN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Sunghursh</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sunghursh</td>
      <td>IN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sunghursh</td>
      <td>IN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sunghursh</td>
      <td>IN</td>
    </tr>
  </tbody>
</table>
</div>



## Creating our Movie Budgets/Revenues DataFrame 


```python
cur.execute('''SELECT * 
                FROM tn_movie_budgets
                ;''')
df_movie_moneys = pd.DataFrame(cur.fetchall())
df_movie_moneys.columns = [x[0] for x in cur.description]
df_movie_moneys.head()
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
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>$425,000,000</td>
      <td>$760,507,625</td>
      <td>$2,776,345,279</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>$410,600,000</td>
      <td>$241,063,875</td>
      <td>$1,045,663,875</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>$350,000,000</td>
      <td>$42,762,350</td>
      <td>$149,762,350</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>May 1, 2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>$330,600,000</td>
      <td>$459,005,868</td>
      <td>$1,403,013,963</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Dec 15, 2017</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>$317,000,000</td>
      <td>$620,181,382</td>
      <td>$1,316,721,747</td>
    </tr>
  </tbody>
</table>
</div>



# Data Cleaning & Converting 

## Cleaning & Converting - Country Releases per Movie 

**Drop 'None' types from the region feature & check the unique values.**


```python
df_title_region = df_title_region.dropna()
df_title_region['region'].unique()
```




    array(['IN', 'XWW', 'VE', 'PL', 'DE', 'PT', 'BR', 'US', 'GB', 'IT', 'RU',
           'AR', 'ES', 'FR', 'CL', 'AU', 'CA', 'NL', 'BG', 'HR', 'HU', 'SE',
           'RO', 'HK', 'FI', 'EE', 'DK', 'LT', 'PK', 'GE', 'TR', 'GR', 'IL',
           'UY', 'RS', 'SI', 'CZ', 'UA', 'MX', 'JP', 'VN', 'PE', 'AZ', 'LV',
           'NO', 'SK', 'AL', 'KR', 'CO', 'EG', 'XEU', 'IR', 'SG', 'BE', 'IS',
           'CH', 'BA', 'ZA', 'CN', 'BD', 'LU', 'CU', 'AM', 'AT', 'MK', 'PH',
           'XSA', 'BO', 'TW', 'LB', 'PR', 'PA', 'IE', 'MY', 'CM', 'KZ', 'NZ',
           'TH', 'ID', 'BY', 'MA', 'CG', 'CR', 'XAS', 'MD', 'PY', 'EC', 'GT',
           'DO', 'TZ', 'DZ', 'BS', 'HT', 'JM', 'CY', 'MZ', 'NG', 'SL', 'PG',
           'MO', 'MN', 'XYU', 'ET', 'AE', 'PS', 'ZW', 'MW', 'FJ', 'MC', 'IQ',
           'SV', 'GL', 'KE', 'CSHH', 'QA', 'AO', 'GP', 'NP', 'ZM', 'AF', 'GH',
           'SZ', 'UG', 'CD', 'ME', 'JO', 'KG', 'RW', 'SN', 'LR', 'NI', 'KH',
           'AG', 'NE', 'VI', 'LI', 'UZ', 'HN', 'TT', 'BF', 'XKV', 'SUHH',
           'TN', 'CSXX', 'LK', 'AN', 'TG', 'BH', 'LS', 'SA', 'SY', 'KW', 'CV',
           'MV', 'BB', 'TJ', 'MH', 'BJ', 'XKO', 'ML', 'GA', 'GU', 'BT', 'SD',
           'MG', 'CI', 'LA', 'BZ', 'BM', 'MM', 'KP', 'IM', 'AW', 'MT', 'XNA',
           'MU', 'AS', 'SR', 'YE', 'SM', 'GW', 'TM', 'NC', 'BN', 'TD', 'KY',
           'TO', 'AD', 'TL', 'MR', 'VU', 'OM', 'PF', 'FO', 'XWG', 'RE', 'BI',
           'SO', 'MQ', 'AQ', 'GM', 'CF', 'DM', 'KN', 'ER', 'VC', 'WF', 'BUMM',
           'LY', 'EH', 'LC', 'SB', 'AI'], dtype=object)



**We noticed that there were different types of country codes in this list. This led to more research to see how one could check that these were all accurate. We found the |pycountry| library, installed it, then imported it below.**


```python
import pycountry
```


```python
def alpha_code_check(value):
    """This function takes in a an alpha code country value, determines its
    classification, and returns that result.  This function is meant to be 
    mapped along a DataFrame series.
    
    Returns:
    Assigned categorical value
    
    Example:
    df['region'].map(lambda x: alpha_code_check(x))"""

    if len(value) == 2:
        value = 'aplha_2'
        return value
    elif len(value) == 3:
        value = 'alpha_3'
        return value
    else:
        value = 'alpha_4'
        return value


df_title_region['alpha_code'] = df_title_region['region'].map(lambda x: alpha_code_check(x))
print(df_title_region['alpha_code'].value_counts())
df_title_region.head()
```

    aplha_2    259181
    alpha_3     19206
    alpha_4        23
    Name: alpha_code, dtype: int64
    




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
      <th>primary_title</th>
      <th>region</th>
      <th>alpha_code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>One Day Before the Rainy Season</td>
      <td>XWW</td>
      <td>alpha_3</td>
    </tr>
  </tbody>
</table>
</div>



**Great, now we have a new column that identifies the type of alpha code in that row.**

**Let's now convert that to a country name using |pycountry|.**


```python
def country_alpha_converter(value):
    '''This is a function to map columns and convert values of country alpha 
    codes to country names.'''
    
    if len(value) == 2:
        x = pycountry.countries.get(alpha_2=value)
        if x == None:
            return 'None'
        return x.name
    elif len(value) == 3:
        x = pycountry.countries.get(alpha_3=value)
        if x == None:
            return 'None'
        return x.name
    else:
        x = pycountry.historic_countries.get(alpha_4=value) #old country codes
        if x == None:
            return 'None'
        return x.name

df_title_region['country'] = df_title_region['region'].map(lambda x: country_alpha_converter(x))
df_title_region.head()
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
      <th>primary_title</th>
      <th>region</th>
      <th>alpha_code</th>
      <th>country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
      <td>India</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
      <td>India</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
      <td>India</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
      <td>India</td>
    </tr>
    <tr>
      <th>5</th>
      <td>One Day Before the Rainy Season</td>
      <td>XWW</td>
      <td>alpha_3</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



**Looks good, though we can clearly see that there are lots of duplicates, as well as a 'None' in the country feature.**


```python
df_title_region[(df_title_region['alpha_code'] == 'alpha_3')
                | 
                (df_title_region['alpha_code'] == 'alpha_4')
               &
               (df_title_region['country'] == 'None')].head()
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
      <th>primary_title</th>
      <th>region</th>
      <th>alpha_code</th>
      <th>country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>One Day Before the Rainy Season</td>
      <td>XWW</td>
      <td>alpha_3</td>
      <td>None</td>
    </tr>
    <tr>
      <th>26</th>
      <td>The Wandering Soap Opera</td>
      <td>XWW</td>
      <td>alpha_3</td>
      <td>None</td>
    </tr>
    <tr>
      <th>47</th>
      <td>So Much for Justice!</td>
      <td>XWW</td>
      <td>alpha_3</td>
      <td>None</td>
    </tr>
    <tr>
      <th>56</th>
      <td>Children of the Green Dragon</td>
      <td>XWW</td>
      <td>alpha_3</td>
      <td>None</td>
    </tr>
    <tr>
      <th>60</th>
      <td>The Tragedy of Man</td>
      <td>XWW</td>
      <td>alpha_3</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



**Based on the information provided above, we can see that a lot of country_codes containing alpha_3 OR alpha_4 also have a 'None' value for their country feature, making these rows essentially useless for this analysis.  To proceed, we will drop all rows with the value of 'None' for the 'country' feature.**


```python
df_title_region = df_title_region[df_title_region['country'] != 'None']
df_title_region.head()
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
      <th>primary_title</th>
      <th>region</th>
      <th>alpha_code</th>
      <th>country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
      <td>India</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
      <td>India</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
      <td>India</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sunghursh</td>
      <td>IN</td>
      <td>aplha_2</td>
      <td>India</td>
    </tr>
    <tr>
      <th>6</th>
      <td>One Day Before the Rainy Season</td>
      <td>IN</td>
      <td>aplha_2</td>
      <td>India</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_title_region['alpha_code'].value_counts()
```




    aplha_2    259176
    alpha_4        23
    Name: alpha_code, dtype: int64



**Looks like all of the alpha_3 codes went along with the 'none' values. Let's see what we have left for alpha_4 codes.**


```python
df_title_region[df_title_region['alpha_code'] == 'alpha_4']
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
      <th>primary_title</th>
      <th>region</th>
      <th>alpha_code</th>
      <th>country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>38031</th>
      <td>Mumu</td>
      <td>CSHH</td>
      <td>alpha_4</td>
      <td>Czechoslovakia, Czechoslovak Socialist Republic</td>
    </tr>
    <tr>
      <th>60649</th>
      <td>Like Crazy</td>
      <td>SUHH</td>
      <td>alpha_4</td>
      <td>USSR, Union of Soviet Socialist Republics</td>
    </tr>
    <tr>
      <th>63250</th>
      <td>Battery Man</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>89541</th>
      <td>Waldheim-KGB Agent Kurt</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>138179</th>
      <td>Albertov Put</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>146591</th>
      <td>Kosma</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>153295</th>
      <td>Danube Floating Free</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>154420</th>
      <td>Drugo ime za slobodu</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>160770</th>
      <td>Zivan Makes a Punk Festival</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>165231</th>
      <td>Skile</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>166504</th>
      <td>Stay Where You Are</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>196688</th>
      <td>It's a journey not a destination</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>196689</th>
      <td>Summertime Summerfun</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>226314</th>
      <td>Jovica and his teeth</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>230436</th>
      <td>Celebrating Serbia</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>242486</th>
      <td>The Absurd Scam</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>245734</th>
      <td>State of soul</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>256314</th>
      <td>Controindicazione</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>272535</th>
      <td>Prison's Prayer</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>274838</th>
      <td>The Taste of the Dhamma</td>
      <td>BUMM</td>
      <td>alpha_4</td>
      <td>Burma, Socialist Republic of the Union of</td>
    </tr>
    <tr>
      <th>279953</th>
      <td>The Roots of Maya: Adept</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>312340</th>
      <td>Resumption</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
    <tr>
      <th>325964</th>
      <td>Lijenstina</td>
      <td>CSXX</td>
      <td>alpha_4</td>
      <td>Serbia and Montenegro</td>
    </tr>
  </tbody>
</table>
</div>



**As we can see above, the countries that were assigned alpha_4 codes are no longer officially recognized today.  For this analysis it is okay to remove these from our dataset.**


```python
df_title_region = df_title_region[df_title_region['alpha_code'] != 'alpha_4']
df_title_region['alpha_code'].value_counts()
```




    aplha_2    259176
    Name: alpha_code, dtype: int64



**Great, now let's drop the alpha_code and region columns.**


```python
df_title_region = df_title_region.drop(['alpha_code', 'region'], axis=1)
print(df_title_region.shape)
df_title_region.head()
```

    (259176, 2)
    




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
      <th>primary_title</th>
      <th>country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sunghursh</td>
      <td>India</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sunghursh</td>
      <td>India</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sunghursh</td>
      <td>India</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sunghursh</td>
      <td>India</td>
    </tr>
    <tr>
      <th>6</th>
      <td>One Day Before the Rainy Season</td>
      <td>India</td>
    </tr>
  </tbody>
</table>
</div>



**OK, so now we are only left with the duplicates. Let's drop them now so that later on our lists of countries (for our country_count feature) will be accurate. Let's also update the name of our DF as well.**


```python
df_title_country = df_title_region.drop_duplicates()
print(df_title_country.shape)
df_title_country.head()
```

    (237752, 2)
    




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
      <th>primary_title</th>
      <th>country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sunghursh</td>
      <td>India</td>
    </tr>
    <tr>
      <th>6</th>
      <td>One Day Before the Rainy Season</td>
      <td>India</td>
    </tr>
    <tr>
      <th>9</th>
      <td>The Other Side of the Wind</td>
      <td>Venezuela, Bolivarian Republic of</td>
    </tr>
    <tr>
      <th>10</th>
      <td>The Other Side of the Wind</td>
      <td>Poland</td>
    </tr>
    <tr>
      <th>11</th>
      <td>The Other Side of the Wind</td>
      <td>Germany</td>
    </tr>
  </tbody>
</table>
</div>



**Dropped about 20K duplicates, very nice.  Now let's build out our feature containing a list of each country per movie.**


```python
df_title_countrylist = df_title_country.groupby('primary_title')['country'].apply(list).reset_index(name='country_list')
print(df_title_countrylist.shape)
df_title_countrylist.head()
```

    (114284, 2)
    




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
      <th>primary_title</th>
      <th>country_list</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>!Women Art Revolution</td>
      <td>[Russian Federation, United States]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>#1 Serial Killer</td>
      <td>[United States]</td>
    </tr>
    <tr>
      <th>2</th>
      <td>#5</td>
      <td>[United States]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>#50Fathers</td>
      <td>[United States]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>#66</td>
      <td>[Indonesia]</td>
    </tr>
  </tbody>
</table>
</div>



**We are now down to 114,284 rows, and with the cleaning that we have done up to this point, it seems fair to assume that all of these movies are unique, but just to be safe, lets check with the .value_counts() method.**


```python
df_title_countrylist['primary_title'].value_counts().head()
```




    Pyro Smugglers                                   1
    Living in Seduced Circumstances                  1
    Muhurlu Kosk                                     1
    24/7/365: The Evolution of Emergency Medicine    1
    For Old Time Sake                                1
    Name: primary_title, dtype: int64



**Perfect!  Next we are going to make a simple feature that counts the number of countries each movie was released in.  This is easily achieved by calculating the length of the country_list feature for each movie using some nice list comprehension. Then update the DF name to be a bit more descriptive.**


```python
pwd
```




    'C:\\Users\\tcast\\Data Science Program\\Module 1\\Mod 1 Project - Movie Analysis\\Movie_Analysis\\Group Question 1'




```python
df_title_countrylist['country_count'] = [len(x) for x in df_title_countrylist['country_list']]
df_title_countrylist_count = df_title_countrylist
df_title_countrylist_count.to_csv(r'C:\Users\tcast\Data Science Program\Module 1\Mod 1 Project - Movie Analysis\Movie_Analysis\Group Question 1\CLEAN-title_countrylist_count.csv')
# df_title_countrylist_count.head()
```

**Here we are going to subset the dataframe.  We are going to do so based on whether a movie was either a domestic or international release.  This significant because we will be able to compare these two different release types in the future should we have any questions involving the two.  We will create the domestic dataframe by filtering for movies whose country_count feature has a value of 1.**


```python
df_domestic_title_country = df_title_countrylist_count[
    df_title_countrylist_count[
        'country_count'
    ] == 1
]

df_domestic_title_country = df_domestic_title_country.drop('country_count', axis=1)
# Reassigned variable to avoid 'A value is trying to be set on a copy of a slice from a DataFrame.' error.
df_domestic_title_country['country_list'] = df_domestic_title_country['country_list'].map(lambda x: ''.join(x))
df_domestic_title_country.head()
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
      <th>primary_title</th>
      <th>country_list</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>#1 Serial Killer</td>
      <td>United States</td>
    </tr>
    <tr>
      <th>2</th>
      <td>#5</td>
      <td>United States</td>
    </tr>
    <tr>
      <th>3</th>
      <td>#50Fathers</td>
      <td>United States</td>
    </tr>
    <tr>
      <th>4</th>
      <td>#66</td>
      <td>Indonesia</td>
    </tr>
    <tr>
      <th>5</th>
      <td>#BKKY</td>
      <td>Thailand</td>
    </tr>
  </tbody>
</table>
</div>



**Great, now we can see all of the domestic movie releases. If we want to, we can also group this dataframe by its country feature and create a list of domestic movies released in each one respectively.**


```python
df_domestic_country_movie_list = df_domestic_title_country.groupby('country_list')['primary_title'].apply(list).reset_index(name='movie_list')
df_domestic_country_movie_list.head()
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
      <th>country_list</th>
      <th>movie_list</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>[A Man's Desire for Fifth Wife, Afghanistan, A...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Albania</td>
      <td>[6 Idiotet, A Shelter Among the Clouds, Albani...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Algeria</td>
      <td>[Abd El-Kader, Africa Is Back, Azib Zamoum, a ...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>American Samoa</td>
      <td>[Pacmakman, Seki A Oe: A Crazy Samoan Love Sto...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Andorra</td>
      <td>[73', Impacto]</td>
    </tr>
  </tbody>
</table>
</div>



**Or we can just see how many domestic movies per country without a list of titles.  We can rearrange this as needed to prepare for a future merge or join with another dataset.**


```python
df_dom_movies_per_country = df_domestic_title_country
df_dom_movies_per_country['movies_per_country'] = 1
df_dom_movies_per_country = df_domestic_title_country.groupby('country_list').sum()
df_dom_movies_per_country.head()
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
      <th>movies_per_country</th>
    </tr>
    <tr>
      <th>country_list</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Afghanistan</th>
      <td>29</td>
    </tr>
    <tr>
      <th>Albania</th>
      <td>35</td>
    </tr>
    <tr>
      <th>Algeria</th>
      <td>42</td>
    </tr>
    <tr>
      <th>American Samoa</th>
      <td>3</td>
    </tr>
    <tr>
      <th>Andorra</th>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



**Now we are going to create out dataframe of international movies, those with more than 1 in their country_count feature.
We then reset the index just to make it look nicer, and to lessen any potential merge or join issues later.**


```python
df_int_title_clist_ccount = df_title_countrylist_count[
    df_title_countrylist_count[
        'country_count'
    ] > 1
]

df_int_title_clist_ccount = df_int_title_clist_ccount.reset_index().drop('index', axis=1)
df_int_title_clist_ccount.head()
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
      <th>primary_title</th>
      <th>country_list</th>
      <th>country_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>!Women Art Revolution</td>
      <td>[Russian Federation, United States]</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>#Female Pleasure</td>
      <td>[Lithuania, Poland, Germany, Switzerland, Spain]</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>#FollowFriday</td>
      <td>[United States, France, Brazil]</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>#Horror</td>
      <td>[United States, Russian Federation]</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>#REALITYHIGH</td>
      <td>[United States, Russian Federation]</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



## Cleaning & Converting - Domestic & International Budgets/Revenues

**Let's have a loser look at this dataset and see if anything stands out.**


```python
print(df_movie_moneys.info())
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 5782 entries, 0 to 5781
    Data columns (total 6 columns):
    id                   5782 non-null int64
    release_date         5782 non-null object
    movie                5782 non-null object
    production_budget    5782 non-null object
    domestic_gross       5782 non-null object
    worldwide_gross      5782 non-null object
    dtypes: int64(1), object(5)
    memory usage: 271.1+ KB
    None
    


```python
df_movie_moneys.head()
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
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>$425,000,000</td>
      <td>$760,507,625</td>
      <td>$2,776,345,279</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>$410,600,000</td>
      <td>$241,063,875</td>
      <td>$1,045,663,875</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>$350,000,000</td>
      <td>$42,762,350</td>
      <td>$149,762,350</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>May 1, 2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>$330,600,000</td>
      <td>$459,005,868</td>
      <td>$1,403,013,963</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Dec 15, 2017</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>$317,000,000</td>
      <td>$620,181,382</td>
      <td>$1,316,721,747</td>
    </tr>
  </tbody>
</table>
</div>



**Let's drop the 'id' column here.**


```python
df_movie_moneys = df_movie_moneys.drop('id', axis=1)
df_movie_moneys.head()
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
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>$425,000,000</td>
      <td>$760,507,625</td>
      <td>$2,776,345,279</td>
    </tr>
    <tr>
      <th>1</th>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>$410,600,000</td>
      <td>$241,063,875</td>
      <td>$1,045,663,875</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>$350,000,000</td>
      <td>$42,762,350</td>
      <td>$149,762,350</td>
    </tr>
    <tr>
      <th>3</th>
      <td>May 1, 2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>$330,600,000</td>
      <td>$459,005,868</td>
      <td>$1,403,013,963</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dec 15, 2017</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>$317,000,000</td>
      <td>$620,181,382</td>
      <td>$1,316,721,747</td>
    </tr>
  </tbody>
</table>
</div>



**Clean and convert the 'production_budget', 'domestic_gross', and 'worldwide_gross' columns from strings to floats using the function created below**


```python
def string_to_float_converter(value, to_replace=None, new_value='', new_dtype=float):
    
    '''This function cleans and converts any column of strings with UP TO 3
    variables at a time. This function is designed to be mapped along columns
    in a Pandas DataFrame and takes in a (value) and the desired string characters
    to be replacesd (to_replace). By default, new_value is: '' and new_dtype 
    is: 'float', though they may be changed as required.
    
    Returns:
    A cleaned and converted value.
    
    Example: 
    df['col'].map(lambda x: string_to_float_converter(x, ['$',',','&']))''' 
    
    if type(to_replace) == list:
        n = len(to_replace)
        if n == 2:
            return new_dtype(value.replace(to_replace[0], new_value).replace(to_replace[1], new_value))
        if n == 3:
            return new_dtype(value.replace(to_replace[0], new_value).replace(to_replace[1], new_value).replace(to_replace[2]))
    else:
        return new_dtype(value.replace(to_replace, new_value))

df_movie_moneys['worldwide_gross'] = df_movie_moneys['worldwide_gross'].map(lambda x: string_to_float_converter(x, to_replace=['$',',']))
df_movie_moneys['production_budget'] = df_movie_moneys['production_budget'].map(lambda x: string_to_float_converter(x, to_replace=['$',',']))
df_movie_moneys['domestic_gross'] = df_movie_moneys['domestic_gross'].map(lambda x: string_to_float_converter(x, to_replace=['$',',']))

df_movie_moneys.head()
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
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
    <tr>
      <th>3</th>
      <td>May 1, 2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dec 15, 2017</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>317000000.0</td>
      <td>620181382.0</td>
      <td>1.316722e+09</td>
    </tr>
  </tbody>
</table>
</div>



**Convert the 'release_date' column to pandas datetime objects.**


```python
df_movie_moneys['release_date'] = pd.to_datetime(df_movie_moneys['release_date'])
df_movie_moneys.head()
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
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2009-12-18</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2011-05-20</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-06-07</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2015-05-01</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-12-15</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>317000000.0</td>
      <td>620181382.0</td>
      <td>1.316722e+09</td>
    </tr>
  </tbody>
</table>
</div>



**Excellent, now let's have a closer look at our numeric columns.**


```python
print(df_movie_moneys.shape)
df_movie_moneys.describe()
```

    (5782, 5)
    




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
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>5.782000e+03</td>
      <td>5.782000e+03</td>
      <td>5.782000e+03</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>3.158776e+07</td>
      <td>4.187333e+07</td>
      <td>9.148746e+07</td>
    </tr>
    <tr>
      <th>std</th>
      <td>4.181208e+07</td>
      <td>6.824060e+07</td>
      <td>1.747200e+08</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.100000e+03</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>5.000000e+06</td>
      <td>1.429534e+06</td>
      <td>4.125415e+06</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>1.700000e+07</td>
      <td>1.722594e+07</td>
      <td>2.798445e+07</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>4.000000e+07</td>
      <td>5.234866e+07</td>
      <td>9.764584e+07</td>
    </tr>
    <tr>
      <th>max</th>
      <td>4.250000e+08</td>
      <td>9.366622e+08</td>
      <td>2.776345e+09</td>
    </tr>
  </tbody>
</table>
</div>



**Lots to see here.  The first thing to notice is the 0 values in the 'domestic_gross' and 'worldwide_gross' columns.  Let's have a look at what kinds of movies fit this criteria before we decide how to proceed.**


```python
print(df_movie_moneys[(df_movie_moneys['worldwide_gross'] == 0)
                                  |
                                  (df_movie_moneys['domestic_gross'] == 0)].shape)
df_movie_moneys[(df_movie_moneys['worldwide_gross'] == 0)
                                  |
                                  (df_movie_moneys['domestic_gross'] == 0)].head()
```

    (548, 5)
    




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
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>194</th>
      <td>2020-12-31</td>
      <td>Moonfall</td>
      <td>150000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>479</th>
      <td>2017-12-13</td>
      <td>Bright</td>
      <td>90000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>480</th>
      <td>2019-12-31</td>
      <td>Army of the Dead</td>
      <td>90000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>535</th>
      <td>2020-02-21</td>
      <td>Call of the Wild</td>
      <td>82000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>617</th>
      <td>2012-12-31</td>
      <td>AstÃ©rix et ObÃ©lix: Au service de Sa MajestÃ©</td>
      <td>77600000.0</td>
      <td>0.0</td>
      <td>60680125.0</td>
    </tr>
  </tbody>
</table>
</div>



**If we sort by 'release_date' then we can see that some of these movies have not even been released yet.  Furthermore, there are release dates that have since passed and yet these movies still have no financial information.  Finally we have some examples of movies where they have a 'worldwide_gross' figure, yet no 'domestic_gross' figure.  I can think of no good, or logical reason as to why this may be the case other than error in the data, therefore we will drop these rows.  This constitutes a drop of 10% of our data, though it is likely that most of these movies we did not have country information for either.**


```python
df_movie_moneys = df_movie_moneys[(df_movie_moneys['worldwide_gross'] != 0)
                                  &
                                  (df_movie_moneys['domestic_gross'] != 0)]
print(df_movie_moneys.describe())
print(df_movie_moneys.shape)
df_movie_moneys.head()
```

           production_budget  domestic_gross  worldwide_gross
    count       5.234000e+03    5.234000e+03     5.234000e+03
    mean        3.403348e+07    4.625747e+07     1.007615e+08
    std         4.296048e+07    7.029651e+07     1.811226e+08
    min         1.100000e+03    3.880000e+02     4.010000e+02
    25%         6.500000e+06    4.289718e+06     8.142571e+06
    50%         2.000000e+07    2.198422e+07     3.543844e+07
    75%         4.500000e+07    5.756598e+07     1.093357e+08
    max         4.250000e+08    9.366622e+08     2.776345e+09
    (5234, 5)
    




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
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2009-12-18</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2011-05-20</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-06-07</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2015-05-01</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-12-15</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>317000000.0</td>
      <td>620181382.0</td>
      <td>1.316722e+09</td>
    </tr>
  </tbody>
</table>
</div>



**Finally, lets's change the feature name from 'movie' to 'primary_title' so as to make future merges less stressful!**


```python
df_movie_moneys = df_movie_moneys.rename(columns={'movie':'primary_title'})
df_movie_moneys.to_csv(r'C:\Users\tcast\Data Science Program\Module 1\Mod 1 Project - Movie Analysis\Movie_Analysis\Group Question 1\CLEAN-BOM_budget_revenues.csv')
df_movie_moneys.head()
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
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2009-12-18</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2011-05-20</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-06-07</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2015-05-01</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-12-15</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>317000000.0</td>
      <td>620181382.0</td>
      <td>1.316722e+09</td>
    </tr>
  </tbody>
</table>
</div>



**We have left the rest of the outlier data in for now so that we can have a better look at the data through EDA before determining the best method of dealing with them.**

# Exploration, Feature Engineering, and Visualizations

## Bringing the data together

**The first step in our Data Exploration & Feature Engineering Phase will be to bring our data together and start to have a look at what stands out.**


```python
df_int_movies_analysis_inner = df_movie_moneys.merge(df_int_title_clist_ccount, on='primary_title')
print(df_int_movies_analysis_inner.shape)
df_int_movies_analysis_inner.head()
```

    (1760, 7)
    




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
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>country_list</th>
      <th>country_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2011-05-20</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>[Japan, Sweden, Peru, Ukraine, United States, ...</td>
      <td>39</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-06-07</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>[France, Mexico, Italy, Poland, Hungary, Portu...</td>
      <td>32</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015-05-01</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>[Azerbaijan, Peru, United States, Israel, Mexi...</td>
      <td>34</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-04-27</td>
      <td>Avengers: Infinity War</td>
      <td>300000000.0</td>
      <td>678815482.0</td>
      <td>2.048134e+09</td>
      <td>[Argentina, Spain, Serbia, United States, Czec...</td>
      <td>31</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-11-17</td>
      <td>Justice League</td>
      <td>300000000.0</td>
      <td>229024295.0</td>
      <td>6.559452e+08</td>
      <td>[Serbia, Argentina, United States, Hungary, Vi...</td>
      <td>29</td>
    </tr>
  </tbody>
</table>
</div>



## Feature Engineering - Adding more metrics

### Foreign Gross, Net Revenue, and Return on Investment

**Great! We are now down to our ~1800 rows.  This is less than half of the original 'bom_movies_gross' data set but we now have a lot more accurate information about each movie.  Before we look at any relationships, let's add a few more important features to this dataset.  Our question is surrounding profitability so it makes sense to add in features related to that measurement.  Let's add a foreign_gross', 'net_revenue', and 'return_on_investment' each to this dataset.**


```python
df_int_movies_analysis_inner['foreign_gross'] = df_int_movies_analysis_inner[
    'worldwide_gross'] - df_int_movies_analysis_inner['domestic_gross']

df_int_movies_analysis_inner['net_revenue'] = df_int_movies_analysis_inner[
    'worldwide_gross'] - df_int_movies_analysis_inner['production_budget']

df_int_movies_analysis_inner['return_on_investment'] = (
    (df_int_movies_analysis_inner[
        'worldwide_gross'] - df_int_movies_analysis_inner[
        'production_budget'])/df_int_movies_analysis_inner[
        'production_budget'])

df_int_movies_analysis_inner.head()
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
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>country_list</th>
      <th>country_count</th>
      <th>foreign_gross</th>
      <th>net_revenue</th>
      <th>return_on_investment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2011-05-20</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>[Japan, Sweden, Peru, Ukraine, United States, ...</td>
      <td>39</td>
      <td>8.046000e+08</td>
      <td>6.350639e+08</td>
      <td>1.546673</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-06-07</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>[France, Mexico, Italy, Poland, Hungary, Portu...</td>
      <td>32</td>
      <td>1.070000e+08</td>
      <td>-2.002376e+08</td>
      <td>-0.572108</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015-05-01</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>[Azerbaijan, Peru, United States, Israel, Mexi...</td>
      <td>34</td>
      <td>9.440081e+08</td>
      <td>1.072414e+09</td>
      <td>3.243841</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-04-27</td>
      <td>Avengers: Infinity War</td>
      <td>300000000.0</td>
      <td>678815482.0</td>
      <td>2.048134e+09</td>
      <td>[Argentina, Spain, Serbia, United States, Czec...</td>
      <td>31</td>
      <td>1.369319e+09</td>
      <td>1.748134e+09</td>
      <td>5.827114</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-11-17</td>
      <td>Justice League</td>
      <td>300000000.0</td>
      <td>229024295.0</td>
      <td>6.559452e+08</td>
      <td>[Serbia, Argentina, United States, Hungary, Vi...</td>
      <td>29</td>
      <td>4.269209e+08</td>
      <td>3.559452e+08</td>
      <td>1.186484</td>
    </tr>
  </tbody>
</table>
</div>



### Add profit/loss feature


```python
def profit_loss_function(ROI):
    
    '''This function takes in an ROI and determins if it value constitues a 
    profit or a loss on a particular investment.
    
    Returns:
    str 'profit' or 'loss'
    
    Eg:
    INPUT:
    profit_loss_function(1.5)
    
    OUTPUT:
    'profit'
    '''
    
    if ROI == 0:
        x = 'break-even'
    if ROI > 0:
        x = 'profit'
    else:
        x = 'loss'
    return x
        
df_int_movies_analysis_inner['profit/loss'] = df_int_movies_analysis_inner['return_on_investment'].map(lambda x: profit_loss_function(x))
df_int_movies_analysis_inner.head()
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
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>country_list</th>
      <th>country_count</th>
      <th>foreign_gross</th>
      <th>net_revenue</th>
      <th>return_on_investment</th>
      <th>profit/loss</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2011-05-20</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>[Japan, Sweden, Peru, Ukraine, United States, ...</td>
      <td>39</td>
      <td>8.046000e+08</td>
      <td>6.350639e+08</td>
      <td>1.546673</td>
      <td>profit</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-06-07</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>[France, Mexico, Italy, Poland, Hungary, Portu...</td>
      <td>32</td>
      <td>1.070000e+08</td>
      <td>-2.002376e+08</td>
      <td>-0.572108</td>
      <td>loss</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015-05-01</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>[Azerbaijan, Peru, United States, Israel, Mexi...</td>
      <td>34</td>
      <td>9.440081e+08</td>
      <td>1.072414e+09</td>
      <td>3.243841</td>
      <td>profit</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-04-27</td>
      <td>Avengers: Infinity War</td>
      <td>300000000.0</td>
      <td>678815482.0</td>
      <td>2.048134e+09</td>
      <td>[Argentina, Spain, Serbia, United States, Czec...</td>
      <td>31</td>
      <td>1.369319e+09</td>
      <td>1.748134e+09</td>
      <td>5.827114</td>
      <td>profit</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-11-17</td>
      <td>Justice League</td>
      <td>300000000.0</td>
      <td>229024295.0</td>
      <td>6.559452e+08</td>
      <td>[Serbia, Argentina, United States, Hungary, Vi...</td>
      <td>29</td>
      <td>4.269209e+08</td>
      <td>3.559452e+08</td>
      <td>1.186484</td>
      <td>profit</td>
    </tr>
  </tbody>
</table>
</div>



### Category Feature - Country Bins


```python
def country_count_category(value):
    '''This function is meant to be mapped along a DataFrame series.  It is 
    specifically meant to take in a country_count value from a dataframe and 
    assigns it to the appropriate category.  These categories can be 
    manipulated as needed, see the commented out potential changes below.
    
    Returns:
    Assigned categorical value
    
    Example:
    df['country_count'].map(lambda x: country_count_category(x))'''
    
    if value <=10:
        value = '2 - 10'
    elif value > 10 and value <= 20:
        value = '11 - 20'
    elif value > 20 and value <= 30:
        value = '21 - 30'
    elif value > 30 and value <= 40:
        value = '31 - 40'
    else: 
#         value > 40 and value <= 50:
        value = '41 +'
#     elif value > 50 and value <= 60:
#         value = '51 - 60'
#     else: 
#         value = '61 +'
    return value
    
df_int_movies_analysis_inner['country_count_category'] = df_int_movies_analysis_inner['country_count'].map(lambda x: country_count_category(x))
df_int_movies_analysis_inner.head()
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
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>country_list</th>
      <th>country_count</th>
      <th>foreign_gross</th>
      <th>net_revenue</th>
      <th>return_on_investment</th>
      <th>profit/loss</th>
      <th>country_count_category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2011-05-20</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>[Japan, Sweden, Peru, Ukraine, United States, ...</td>
      <td>39</td>
      <td>8.046000e+08</td>
      <td>6.350639e+08</td>
      <td>1.546673</td>
      <td>profit</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-06-07</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>[France, Mexico, Italy, Poland, Hungary, Portu...</td>
      <td>32</td>
      <td>1.070000e+08</td>
      <td>-2.002376e+08</td>
      <td>-0.572108</td>
      <td>loss</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015-05-01</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>[Azerbaijan, Peru, United States, Israel, Mexi...</td>
      <td>34</td>
      <td>9.440081e+08</td>
      <td>1.072414e+09</td>
      <td>3.243841</td>
      <td>profit</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-04-27</td>
      <td>Avengers: Infinity War</td>
      <td>300000000.0</td>
      <td>678815482.0</td>
      <td>2.048134e+09</td>
      <td>[Argentina, Spain, Serbia, United States, Czec...</td>
      <td>31</td>
      <td>1.369319e+09</td>
      <td>1.748134e+09</td>
      <td>5.827114</td>
      <td>profit</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-11-17</td>
      <td>Justice League</td>
      <td>300000000.0</td>
      <td>229024295.0</td>
      <td>6.559452e+08</td>
      <td>[Serbia, Argentina, United States, Hungary, Vi...</td>
      <td>29</td>
      <td>4.269209e+08</td>
      <td>3.559452e+08</td>
      <td>1.186484</td>
      <td>profit</td>
      <td>21 - 30</td>
    </tr>
  </tbody>
</table>
</div>



  **That was easy! Now that we have the features that we would liketo see, let's start out with Seaborn's .pairplot and see what kinds of relationships exist.** 

## Pairplot - Initial Visualization


```python
sns.set(context='notebook', style='darkgrid', color_codes=True, palette='muted')

sns.pairplot(df_int_movies_analysis_inner, kind='reg');
```

    C:\Users\tcast\anaconda3\envs\learn-env\lib\site-packages\scipy\stats\stats.py:1713: FutureWarning: Using a non-tuple sequence for multidimensional indexing is deprecated; use `arr[tuple(seq)]` instead of `arr[seq]`. In the future this will be interpreted as an array index, `arr[np.array(seq)]`, which will result either in an error or a different result.
      return np.add.reduce(sorted[indexer] * weights, axis=axis) / sumval
    


![png](README_files/README_81_1.png)


**This is great! On our first look we can already see that a positive relationship exists between 'country_count' and our gross figures. Though remember, our question is interested in profitability, not just revenue. The last column on the above matrix shows a very wide range of potential profitability, however the direction that is it going is great! We also observe that there are A LOT of positively-skewed features for budget and revenue distributions.**

**Next, let's take a closer look at our individual features using a boxplot to help us better contextualize the outliers.** 

## Boxplot - Outlier Identification


```python
fig, (ax1, ax2, ax3, ax4, ax5) = plt.subplots(ncols=5, figsize=(15,10), sharey=True)

ax_list = [ax1, ax2, ax3, ax4, ax5]
x = [df_int_movies_analysis_inner['production_budget'],
     df_int_movies_analysis_inner['domestic_gross'],
     df_int_movies_analysis_inner['worldwide_gross'],
     df_int_movies_analysis_inner['foreign_gross'],
     df_int_movies_analysis_inner['net_revenue']]


for n in range(1,6):
    ax = ax_list[n-1]
    x_new = x[n-1]
    sns.set_style('darkgrid')
    sns.boxplot(x_new, ax=ax, orient='v')
```


![png](README_files/README_84_0.png)


**Whoa! Looks like the data here are littered with outliers on the positive end of the spectrum.  Let's try a few different techniques and see which ones result in a more normalized distribution!**

## Outlier Exploration & Visualizations

**Let's create a new dataframe to use for the following EDA purposes since we have so many outliers in our original dataset.**


```python
df_int_movies_analysis_inner_cleaned = df_int_movies_analysis_inner.copy()
print(df_int_movies_analysis_inner_cleaned.shape)
df_int_movies_analysis_inner_cleaned.head()
```

    (1760, 12)
    




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
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>country_list</th>
      <th>country_count</th>
      <th>foreign_gross</th>
      <th>net_revenue</th>
      <th>return_on_investment</th>
      <th>profit/loss</th>
      <th>country_count_category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2011-05-20</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>[Japan, Sweden, Peru, Ukraine, United States, ...</td>
      <td>39</td>
      <td>8.046000e+08</td>
      <td>6.350639e+08</td>
      <td>1.546673</td>
      <td>profit</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-06-07</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>[France, Mexico, Italy, Poland, Hungary, Portu...</td>
      <td>32</td>
      <td>1.070000e+08</td>
      <td>-2.002376e+08</td>
      <td>-0.572108</td>
      <td>loss</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015-05-01</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>[Azerbaijan, Peru, United States, Israel, Mexi...</td>
      <td>34</td>
      <td>9.440081e+08</td>
      <td>1.072414e+09</td>
      <td>3.243841</td>
      <td>profit</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-04-27</td>
      <td>Avengers: Infinity War</td>
      <td>300000000.0</td>
      <td>678815482.0</td>
      <td>2.048134e+09</td>
      <td>[Argentina, Spain, Serbia, United States, Czec...</td>
      <td>31</td>
      <td>1.369319e+09</td>
      <td>1.748134e+09</td>
      <td>5.827114</td>
      <td>profit</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-11-17</td>
      <td>Justice League</td>
      <td>300000000.0</td>
      <td>229024295.0</td>
      <td>6.559452e+08</td>
      <td>[Serbia, Argentina, United States, Hungary, Vi...</td>
      <td>29</td>
      <td>4.269209e+08</td>
      <td>3.559452e+08</td>
      <td>1.186484</td>
      <td>profit</td>
      <td>21 - 30</td>
    </tr>
  </tbody>
</table>
</div>



**The feature that looks like it contains the most egregious amount of outliers is 'worldwide_gross'. Let's see how the data are affected when we start to preform some outlier cleaning in the cell below.  Let's write a function in the following cell that will take in information about how we want to adjust our dataframe using quantiles. This function will allow us to perform quick EDA, and if we see something that we like, we can just re-define a new dataframe as needed.**

##  Function - Quantile Column Cleaning


```python
def between_quantile_col_cleaner(df, colname, lower_quantile, upper_quantile):
    
    '''This function\'s purpose is to address outliers in data.  This funciton
    takes in a dataframe, column name, and both lower/upper quantiles to keep
    data using the .between method.
    
    Returns:
    Dataframe
    
    Example:
    between_quantile_cleaner(df_cars, 'type', 0.05, 0.95)'''
    
    column_total = df[colname]
    
    column_remaining = column_total.between(
        column_total.quantile
        (
            lower_quantile
        ),
        column_total.quantile
        (
            upper_quantile
        ))
    df = df.iloc[column_remaining[column_remaining].index]
    print(df.shape)
    return df

between_quantile_col_cleaner(df_int_movies_analysis_inner_cleaned, 'worldwide_gross', .10, .90).head()
```

    (1408, 12)
    




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
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>country_list</th>
      <th>country_count</th>
      <th>foreign_gross</th>
      <th>net_revenue</th>
      <th>return_on_investment</th>
      <th>profit/loss</th>
      <th>country_count_category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>2019-06-07</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>149762350.0</td>
      <td>[France, Mexico, Italy, Poland, Hungary, Portu...</td>
      <td>32</td>
      <td>107000000.0</td>
      <td>-200237650.0</td>
      <td>-0.572108</td>
      <td>loss</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2013-07-02</td>
      <td>The Lone Ranger</td>
      <td>275000000.0</td>
      <td>89302115.0</td>
      <td>260002115.0</td>
      <td>[Italy, Ukraine, Slovakia, Venezuela, Bolivari...</td>
      <td>33</td>
      <td>170700000.0</td>
      <td>-14997885.0</td>
      <td>-0.054538</td>
      <td>loss</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2012-03-09</td>
      <td>John Carter</td>
      <td>275000000.0</td>
      <td>73058679.0</td>
      <td>282778100.0</td>
      <td>[Ukraine, United States, Argentina, India, Lit...</td>
      <td>28</td>
      <td>209719421.0</td>
      <td>7778100.0</td>
      <td>0.028284</td>
      <td>profit</td>
      <td>21 - 30</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1998-08-14</td>
      <td>The Avengers</td>
      <td>60000000.0</td>
      <td>23385416.0</td>
      <td>48585416.0</td>
      <td>[Peru, Turkey, Mexico, Slovakia, Colombia, Swe...</td>
      <td>39</td>
      <td>25200000.0</td>
      <td>-11414584.0</td>
      <td>-0.190243</td>
      <td>loss</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2012-05-18</td>
      <td>Battleship</td>
      <td>220000000.0</td>
      <td>65233400.0</td>
      <td>313477717.0</td>
      <td>[Uruguay, Greece, Viet Nam, Lithuania, Colombi...</td>
      <td>25</td>
      <td>248244317.0</td>
      <td>93477717.0</td>
      <td>0.424899</td>
      <td>profit</td>
      <td>21 - 30</td>
    </tr>
  </tbody>
</table>
</div>



## Funtion - Quantile Column Cleaning (Removed Rows)

**Here is a function that shows us the movies that were dropped.  This is a useful function as it allows us to see which movies don't make the cleaning-cut!**


```python
def between_quantile_col_removed(df, colname, lower_quantile, upper_quantile):
    
    '''This function\'s purpose is to address outliers in data.  This funciton
    takes in a dataframe, column name, and both lower/upper quantiles to keep
    data using the .between method.
    
    Returns:
    Dataframe
    
    Example:
    between_quantile_cleaner(df_cars, 'type', 0.05, 0.95)'''
    
    column_total = df[colname]
    
    column_remaining = column_total.between(
        column_total.quantile
        (
            lower_quantile
        ),
        column_total.quantile
        (
            upper_quantile
        ))
    df = df.drop(column_remaining[column_remaining].index)
    print(df.shape)
    return df

between_quantile_col_removed(df_int_movies_analysis_inner_cleaned, 'worldwide_gross', .10, .90).head()
```

    (352, 12)
    




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
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>country_list</th>
      <th>country_count</th>
      <th>foreign_gross</th>
      <th>net_revenue</th>
      <th>return_on_investment</th>
      <th>profit/loss</th>
      <th>country_count_category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2011-05-20</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>[Japan, Sweden, Peru, Ukraine, United States, ...</td>
      <td>39</td>
      <td>8.046000e+08</td>
      <td>6.350639e+08</td>
      <td>1.546673</td>
      <td>profit</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015-05-01</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>[Azerbaijan, Peru, United States, Israel, Mexi...</td>
      <td>34</td>
      <td>9.440081e+08</td>
      <td>1.072414e+09</td>
      <td>3.243841</td>
      <td>profit</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-04-27</td>
      <td>Avengers: Infinity War</td>
      <td>300000000.0</td>
      <td>678815482.0</td>
      <td>2.048134e+09</td>
      <td>[Argentina, Spain, Serbia, United States, Czec...</td>
      <td>31</td>
      <td>1.369319e+09</td>
      <td>1.748134e+09</td>
      <td>5.827114</td>
      <td>profit</td>
      <td>31 - 40</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-11-17</td>
      <td>Justice League</td>
      <td>300000000.0</td>
      <td>229024295.0</td>
      <td>6.559452e+08</td>
      <td>[Serbia, Argentina, United States, Hungary, Vi...</td>
      <td>29</td>
      <td>4.269209e+08</td>
      <td>3.559452e+08</td>
      <td>1.186484</td>
      <td>profit</td>
      <td>21 - 30</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2015-11-06</td>
      <td>Spectre</td>
      <td>300000000.0</td>
      <td>200074175.0</td>
      <td>8.796209e+08</td>
      <td>[Bulgaria, Portugal, Serbia, Russian Federatio...</td>
      <td>29</td>
      <td>6.795467e+08</td>
      <td>5.796209e+08</td>
      <td>1.932070</td>
      <td>profit</td>
      <td>21 - 30</td>
    </tr>
  </tbody>
</table>
</div>



## Boxplot - Box Office Numbers w/ Outlier Manipulation

 **Ok, let's have a look at our data now using our new function and see how the below plots change. The data between the 10th and 90th percentile looks the most reasonable for this visualization.**


```python
df_int_movies_analysis_inner_cleaned_boxtest = between_quantile_col_cleaner(
    df_int_movies_analysis_inner_cleaned,
    'worldwide_gross',
    .10,
    .90)

fig, (ax1, ax2, ax3, ax4, ax5) = plt.subplots(ncols=5, sharey=True, figsize=(15,10))

ax_list = [ax1, ax2, ax3, ax4, ax5]
x = [df_int_movies_analysis_inner_cleaned_boxtest['production_budget'],
     df_int_movies_analysis_inner_cleaned_boxtest['domestic_gross'],
     df_int_movies_analysis_inner_cleaned_boxtest['worldwide_gross'],
     df_int_movies_analysis_inner_cleaned_boxtest['foreign_gross'],
     df_int_movies_analysis_inner_cleaned_boxtest['net_revenue']]

for n in range(1,6):
    ax = ax_list[n-1]
    x_new = x[n-1]
    sns.set_style('darkgrid');
    sns.boxplot(x_new, ax=ax, orient='v', showmeans=True, color='lightpink');
```

    (1408, 12)
    


![png](README_files/README_97_1.png)


## Countplot - Countries Per Movie w/ Outlier Manipulation

**Now lets do the same testing but with a countplot.  This visual looks quite similar regardless of the manner with which it was sliced.  Therefore the full dataset was chosen for the best representation of this visual.**


```python
df_int_movies_analysis_inner_cleaned_histtest = df_int_movies_analysis_inner_cleaned

# between_quantile_col_cleaner(
#     df_int_movies_analysis_inner_cleaned,
#     'worldwide_gross',
#     .10,
#     .90)

plt.figure(figsize=(15,10))

column = df_int_movies_analysis_inner_cleaned_histtest['country_count']

sns.countplot(x=column,
              data=df_int_movies_analysis_inner_cleaned_histtest,
              saturation=2,
              palette="rocket");

plt.ylabel('Number of Movies', fontsize=23)
plt.xlabel('Number of Countries', fontsize=23)
plt.title('Countries Per Movie - Countplot Distribution', fontsize=30, pad=10, loc='right');
```


![png](README_files/README_100_0.png)


# Microsoft Question Visualization & Conclusion

**Okay, now for the last step.  We have seen above how the data can change depending on how we treat our outliers.  Let's now aim to answer the big question:**

**Are movies that are released in more countries more profitable?**

**Let's create two more plots and put them side by side.  The firs plot will look at the costs of producing these movies, as well as how much money the tend to bring back in.  The second plot will look at the number of movies that come out of the international scene with either a profit or a loss.  The following plots will use the MEDIAN as its estimator fucntion as it is the best tool we have to combat against the many outliers that remain.**

**These last two visualizations will be shown using the data from the 1st to 99th percentile. There were some variations in the plotting depending on how the dataset was sliced, but the reasons were mostly due to not having a large sample size of movies in over 40 coutnries.  Therefore we chose to stick with the subset below in the end.**


```python
df_int_movies_analysis_inner_cleaned_bartest = between_quantile_col_cleaner(
    df_int_movies_analysis_inner_cleaned,
    'worldwide_gross',
    .01,
    .99)

fig = plt.figure(figsize=(18,10));
sns.set(style='darkgrid');
order = ['2 - 10', '11 - 20', '21 - 30', '31 - 40', '41 +']
#  - 50', '51 - 60', '61 +'

ax1 = fig.add_subplot(121)

sns.set_color_codes('pastel')
sns.pointplot(x='country_count_category',
            y='worldwide_gross',
            data=df_int_movies_analysis_inner_cleaned_bartest,
            color='b',
            order=order,
            estimator=np.median,
            ax=ax1,
            ci=None);

sns.set_color_codes('pastel')
sns.pointplot(x='country_count_category',
            y='net_revenue',
            data=df_int_movies_analysis_inner_cleaned_bartest,
            color='g',
            order=order,
            estimator=np.median,
            ax=ax1,
            ci=None);

sns.set_color_codes('muted')
sns.pointplot(x='country_count_category',
            y='production_budget',
            data=df_int_movies_analysis_inner_cleaned_bartest,
            color='r',
            order=order,
            estimator=np.median,
            ax=ax1,
            ci=None);

plt.title('International Movie Costs vs. Revenues ', fontsize=25, loc='center', pad=15);
plt.legend(['Worldwide Gross', 'Net Revenue', 'Production Budget'], frameon=False, fontsize=17, loc='upper left');
plt.ylabel('Hundreds of Millions (Median)', fontsize=22);
plt.xlabel('Number of Country Releases', fontsize=20);
plt.xticks(fontsize=18);
plt.yticks(fontsize=18);

ax2 = fig.add_subplot(122)

# sns.set_color_codes('pastel')
# sns.barplot(x='country_count_category',
#             y='foreign_gross',
#             data=df_int_movies_analysis_inner_cleaned_bartest,
#             color='orange',
#             order=order,
#             estimator=np.median,
#             ax=ax2,
#             ci=None);

sns.barplot(x='country_count_category',
            y='return_on_investment',
            data=df_int_movies_analysis_inner_cleaned_bartest,
            order=order,
            estimator=np.median,
            ax=ax2,
            ci=None,
            saturation=20,
            hue='profit/loss',
            hue_order=['loss', 'profit']);

# sns.set_color_codes('pastel')
# sns.pointplot(x='country_count_category',
#             y='net_revenue',
#             data=df_int_movies_analysis_inner_cleaned_bartest,
#             color='darkgreen',
#             order=order,
#             estimator=np.median,
#             ax=ax2,
#             ci=None);

plt.title('International Profits vs. Losses', fontsize=25, loc='right', pad=15);
plt.legend(['Loss', 'Profit'], frameon=False, fontsize=20, loc='upper left');
plt.ylabel('Return on Investment (Median)', fontsize=22);
plt.xlabel('Number of Country Releases', fontsize=20);
plt.xticks(fontsize=18);
plt.yticks(fontsize=18);
```

    (1724, 12)
    


![png](README_files/README_103_1.png)


# Observation Conclusions
**These last two visualizations paint the picture quite clearly that in fact yes, movies that are released in more countries are generally more profitable.  We can also make some conclusions about the risks involved in making movies that are meant to be for a global audience.**

# Future Work Recommendations
**- To take this analysis further, it would have been great to see more accurate information for country counts per movie. If we had a better picture about not just which countries, but the timing of the release in each subsequent country, then a lot of work could be done in determining the best time to release and where.**

**- Additionally, more detailed information about the distribution of the revenue between each country would have been very insightful.  This could have led to observations surrounding each country and their consumption patterns and preferences.**

# * * Bonus * * - "Domestic" Movie Country Count Estimator

**As we can see below, the IMDB database was not as accurate as we would have liked.  Avatar is still the largest movie ever made in almost all aspects, yet it has a country list consisting only of Japan.  We thought it would be interesting to see, based on the cleaning of the previous data, if we could predict the number of movies that a country was released in based on its financial metrics.  Below we have made a function to check against the 'worldwide_gross' value, though this function could easily be altered to measure against any of the numerical features.**


```python
df_domestic_country_check = df_movie_moneys.merge(df_domestic_title_country, on='primary_title')

df_domestic_country_check = df_domestic_country_check.sort_values('worldwide_gross', ascending=False)

df_domestic_country_check['return_on_investment'] = (
    (df_domestic_country_check['worldwide_gross'] - 
     df_domestic_country_check['production_budget']) / 
    df_domestic_country_check['production_budget'])

df_domestic_country_check['profit/loss'] = df_domestic_country_check[
    'return_on_investment'].map(lambda x: profit_loss_function(x))

df_domestic_country_check['foreign_gross'] = df_domestic_country_check[
    'worldwide_gross'] - df_domestic_country_check['domestic_gross']

df_int_movies_analysis_inner['net_revenue'] = df_int_movies_analysis_inner[
    'worldwide_gross'] - df_int_movies_analysis_inner['production_budget']

df_domestic_country_check.head()
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
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>country_list</th>
      <th>movies_per_country</th>
      <th>return_on_investment</th>
      <th>profit/loss</th>
      <th>foreign_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2009-12-18</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
      <td>Japan</td>
      <td>1</td>
      <td>5.532577</td>
      <td>profit</td>
      <td>2.015838e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1997-12-19</td>
      <td>Titanic</td>
      <td>200000000.0</td>
      <td>659363944.0</td>
      <td>2.208208e+09</td>
      <td>United Kingdom</td>
      <td>1</td>
      <td>10.041042</td>
      <td>profit</td>
      <td>1.548844e+09</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2009-12-25</td>
      <td>Sherlock Holmes</td>
      <td>90000000.0</td>
      <td>209028679.0</td>
      <td>4.984382e+08</td>
      <td>United States</td>
      <td>1</td>
      <td>4.538202</td>
      <td>profit</td>
      <td>2.894095e+08</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2000-05-05</td>
      <td>Gladiator</td>
      <td>103000000.0</td>
      <td>187683805.0</td>
      <td>4.576838e+08</td>
      <td>Australia</td>
      <td>1</td>
      <td>3.443532</td>
      <td>profit</td>
      <td>2.700000e+08</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2000-12-22</td>
      <td>Cast Away</td>
      <td>85000000.0</td>
      <td>233632142.0</td>
      <td>4.272305e+08</td>
      <td>Puerto Rico</td>
      <td>1</td>
      <td>4.026241</td>
      <td>profit</td>
      <td>1.935984e+08</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_int_movies_analysis_inner_cleaned.groupby('country_count_category').median()
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
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>country_count</th>
      <th>foreign_gross</th>
      <th>net_revenue</th>
      <th>return_on_investment</th>
    </tr>
    <tr>
      <th>country_count_category</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>11 - 20</th>
      <td>10600000.0</td>
      <td>7710234.0</td>
      <td>13019253.0</td>
      <td>16.0</td>
      <td>2828779.0</td>
      <td>1927779.0</td>
      <td>0.209835</td>
    </tr>
    <tr>
      <th>2 - 10</th>
      <td>8000000.0</td>
      <td>4829804.0</td>
      <td>7263222.0</td>
      <td>5.0</td>
      <td>98053.0</td>
      <td>-204.5</td>
      <td>-0.001842</td>
    </tr>
    <tr>
      <th>21 - 30</th>
      <td>30000000.0</td>
      <td>35599744.5</td>
      <td>70671126.0</td>
      <td>26.0</td>
      <td>31200720.0</td>
      <td>38215892.0</td>
      <td>1.433405</td>
    </tr>
    <tr>
      <th>31 - 40</th>
      <td>75000000.0</td>
      <td>89302115.0</td>
      <td>260002115.0</td>
      <td>33.0</td>
      <td>154599082.0</td>
      <td>167023808.0</td>
      <td>2.594978</td>
    </tr>
    <tr>
      <th>41 +</th>
      <td>187500000.0</td>
      <td>246200985.0</td>
      <td>899906806.5</td>
      <td>43.5</td>
      <td>575300946.5</td>
      <td>687406806.5</td>
      <td>3.799302</td>
    </tr>
  </tbody>
</table>
</div>



**The function below is based on the above table, replace the values as needed to switch between features to compare.**


```python
def domestic_movie_guess(movie):
    
    '''This function takes in the name of a movie (string) from the 
    df_domestic_country_check dataset and returns an estimate of the number of
    countries that the movie was released in.'''
    
    dom_df = df_domestic_country_check
    int_df = df_int_movies_analysis_inner_cleaned
    
    country_prob_list = df_int_movies_analysis_inner_cleaned.groupby('country_count_category').median()['worldwide_gross']
#     print(country_prob_list)
    
    country_prob_list_keys = list(country_prob_list.keys())
#     print(country_prob_list_keys)
    
    country_prob_list_values = list(country_prob_list.unique())
#     print(country_prob_list_values)
    
    prob_dict = {}
    
#     for i in country_prob_list_values:
#         prob_dict[i] = country_prob_list_values[i]
    prob_dict['7263222.0'] = '10 or less'
    prob_dict['13019253.0'] = '11 - 20'
    prob_dict['70671126.0'] = '21 - 30'
    prob_dict['260002115.0'] = '31 - 40'
    prob_dict['899906806.5'] = '41 +'
#     print(prob_dict)
    
    worldwide_gross = int(dom_df[dom_df['primary_title'] == movie]['worldwide_gross'].unique())
#     print(worldwide_gross)

    for i in country_prob_list_values:
        n = 0
        if worldwide_gross > country_prob_list_values[-1]:
            return country_prob_list_keys[-1]
        elif i < worldwide_gross:
#             print('passed')
            continue
        else:
            return print('Hmmmm, I\'m thinking this movie was released in {} countries.'.format(prob_dict[str(i)]))
        
    
domestic_movie_guess('Cast Away')
```

    Hmmmm, I'm thinking this movie was released in 41 + countries.
    

**Feel free to check out your favourite  movie source online to compare the results!** 

_______________________________________________________________________________________________________


## My Individual Question: What type of films win the most awards?



So our group questions covered the topics concerning profits vs number of international releases, and also production budgets vs Academy Awards. I wanted to expand a bit on the Academy Award topic, given that an award nomination not only greatly increases profits, but heightens the reputation of the film studio. 

If we could get a bit more insight into what genres of movies are more likely to win an award recently, then we can infer that Microsoft should aim to create a movie of that genre. 

Let's get started by importing our data.


```python
import pandas as pd
import os
import sqlite3
```

### Import Data Tables 

Let's bring in and view our initial scraped award table that we used in our Group Question


```python
conn = sqlite3.connect("movies_db.sqlite") 
cur = conn.cursor()
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



As we have already created SQL tables in our group question notebook, we will skip to viewing the dataframes from those SQL tables.


```python
cur.execute('''SELECT *
               FROM films_by_awards2                
                ''') 

nominated_films_df = pd.DataFrame(cur.fetchall())
nominated_films_df.columns = [x[0] for x in cur.description]
nominated_films_df
```

```python
nominated_films_df.info()

```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1316 entries, 0 to 1315
    Data columns (total 6 columns):
    index          1316 non-null int64
    Unnamed: 0     1316 non-null int64
    film           1316 non-null object
    year           1316 non-null object
    awards         1316 non-null object
    nominations    1316 non-null object
    dtypes: int64(2), object(4)
    memory usage: 61.8+ KB


```python
nominated_films_df['year'].unique()
```

    array(['2019', '2018', '2017', '2016', '2015', '2014', '2013', '2012',
           '2011', '2010', '2009', '2008', '2007', '2006', '2005', '2004',
           '2003', '2002', '2001', '2000', '1999', '1998', '1997', '1996',
           '1995', '1994', '1993', '1992', '1991', '1990', '1989', '1988',
           '1987', '1986', '1985', '1984', '1983', '1982', '1981', '1980',
           '1979', '1978', '1977', '1976', '1975', '1974', '1973', '1972',
           '1971', '1970', '1969', '1968', '1967', '1966', '1965', '1927/28',
           '1954', '1949', '1938', '1951', '1943', '1960', '1953', '1957',
           '1950', '1929/30', '1941', '1958', '1963', '1956', '1945', '1946',
           '1936', '1940', '1937', '1947', '1952', '1931/32', '1942', '1964',
           '1959', '1948', '1962', '1961', '1928/29', '1955', '1935', '1939',
           '1932/33', '1930/31', '1934', '1944'], dtype=object)



## Cleaning Data

Let's clean up these values containing two years, so that we can figure out how many films were awarded most recently (since 2010).


```python
value_split = nominated_films_df['year'].str.split('/')
```


```python
nominated_films_df['year'] = value_split.str.get(0).astype('int')
```


```python
nominated_films_df['year'].unique()
```


    array([2019, 2018, 2017, 2016, 2015, 2014, 2013, 2012, 2011, 2010, 2009,
           2008, 2007, 2006, 2005, 2004, 2003, 2002, 2001, 2000, 1999, 1998,
           1997, 1996, 1995, 1994, 1993, 1992, 1991, 1990, 1989, 1988, 1987,
           1986, 1985, 1984, 1983, 1982, 1981, 1980, 1979, 1978, 1977, 1976,
           1975, 1974, 1973, 1972, 1971, 1970, 1969, 1968, 1967, 1966, 1965,
           1927, 1954, 1949, 1938, 1951, 1943, 1960, 1953, 1957, 1950, 1929,
           1941, 1958, 1963, 1956, 1945, 1946, 1936, 1940, 1937, 1947, 1952,
           1931, 1942, 1964, 1959, 1948, 1962, 1961, 1928, 1955, 1935, 1939,
           1932, 1930, 1934, 1944])


Let's see how many films were awarded since 2010.


```python
newest_films = nominated_films_df['year'] > 2010    
newest_films.value_counts()
```

    False    1180
    True      136
    Name: year, dtype: int64


We have 136 films that were awarded since 2010.

Now let's view a table that contains films and their respective genres since 2010.


```python
cur.execute('''SELECT *
               FROM imdb_title_basics
               ORDER BY start_year
                ''') 

movies_df = pd.DataFrame(cur.fetchall())
movies_df.columns = [x[0] for x in cur.description]
movies_df
```


## Joining Awards Data to Genre Data

Let's do an inner-join of this imdb_title_basics table to our Awards data and hopefully we will have genre info for most of the recently-nominated films.


```python
cur.execute('''SELECT a.film, a.year, a.awards, a.nominations, b.genres
               FROM films_by_awards2 a
               JOIN imdb_title_basics b 
               ON a.film = b.primary_title
               AND a.year = b.start_year
                ''') 

awards_to_genres_df = pd.DataFrame(cur.fetchall())
awards_to_genres_df.columns = [x[0] for x in cur.description]
awards_to_genres_df
```


We are left with 120 rows, and a few apparent duplicates. Let's drop those duplicates below:

### Cleaning Duplicated Data


```python
awards_to_genres_df.drop_duplicates(subset='film', keep='last', inplace=True)
```


We are now left with the most recent 114 award winning films, and their attributed genres. Not too bad! My general thought for this data is that using the most recent 10 years of films would give a more accurate representation of the distribution of genres that are more likely to get awarded in the near future.

Let's get to work on creating a column for each genre, starting by splitting each row's list of genres.

### Reformatting Data 


```python
awards_to_genres_df['genres'] = awards_to_genres_df['genres'].apply(lambda x: x.split(",") if x in x else x)

#the above iterates through each rows for genre, splitting from the comma if a comma is contained
```


```python
awards_to_genres_df['genres'].head()
```

    1                 [Drama, Thriller]
    2        [Action, Biography, Drama]
    3                  [Drama, Romance]
    4              [Comedy, Drama, War]
    5    [Adventure, Animation, Comedy]
    Name: genres, dtype: object

Let's make a new set, by iterating through each separated value in genres with a for loop


```python
all_genres = set()                      
for genres in awards_to_genres_df['genres']:
    if genres:
        all_genres.update(genres)
```


```python
all_genres
```
    {'Action',
     'Adventure',
     'Animation',
     'Biography',
     'Comedy',
     'Crime',
     'Documentary',
     'Drama',
     'Family',
     'Fantasy',
     'History',
     'Horror',
     'Music',
     'Musical',
     'Mystery',
     'Romance',
     'Sci-Fi',
     'Sport',
     'Thriller',
     'War',
     'Western'}


Now let's iterate through each genre, creating a new column for each iteration (each genre) and assigning it a 0 value


```python
for genre in all_genres:
    awards_to_genres_df[genre] = np.zeros(shape=awards_to_genres_df.shape[0])
                                

for index, row in awards_to_genres_df.iterrows():   #iterates through the dataframe rows as pairs of index and row
    if row['genres']:                               #if the row contains the genres:
        for genre in row['genres']:                 #for each iteration (genre) of the genres list, 
            awards_to_genres_df.loc[index, genre] = 1                  #the column for that genre get a 1

awards_to_genres_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 114 entries, 1 to 119
    Data columns (total 26 columns):
    film           114 non-null object
    year           114 non-null object
    awards         114 non-null object
    nominations    114 non-null object
    genres         114 non-null object
    Sport          114 non-null float64
    Musical        114 non-null float64
    Action         114 non-null float64
    Comedy         114 non-null float64
    Thriller       114 non-null float64
    Drama          114 non-null float64
    Sci-Fi         114 non-null float64
    War            114 non-null float64
    Animation      114 non-null float64
    Family         114 non-null float64
    Documentary    114 non-null float64
    Music          114 non-null float64
    Fantasy        114 non-null float64
    Crime          114 non-null float64
    Western        114 non-null float64
    Horror         114 non-null float64
    Mystery        114 non-null float64
    Adventure      114 non-null float64
    Biography      114 non-null float64
    Romance        114 non-null float64
    History        114 non-null float64
    dtypes: float64(21), object(5)
    memory usage: 29.0+ KB


Now all of our values in each column is filled.


We can now drop our old column of genre lists.


```python
awards_to_genres_df2 = awards_to_genres_df.drop(columns = 'genres')
```


```python

Looking good- let's just quickly check our nominations and awards columns for value cleanliness.


```python
awards_to_genres_df['nominations'].unique()
```



    array(['6', '4', '2', '11', '10', '1', '5', '7', '8', '3', '13', '14',
           '12', '9'], dtype=object)



```python
awards_to_genres_df['awards'].unique()
```




    array(['4', '2', '1', '3', '6', '7', '5'], dtype=object)



Now, let's make a list of each column and assign the genre columns to a variable.

### Extracting the Specific Data We Need


```python
cols = list(awards_to_genres_df.columns)
cols
```


    ['film',
     'year',
     'awards',
     'nominations',
     'genres',
     'Sport',
     'Musical',
     'Action',
     'Comedy',
     'Thriller',
     'Drama',
     'Sci-Fi',
     'War',
     'Animation',
     'Family',
     'Documentary',
     'Music',
     'Fantasy',
     'Crime',
     'Western',
     'Horror',
     'Mystery',
     'Adventure',
     'Biography',
     'Romance',
     'History']



```python
genre_cols = cols[5:]
```

let's create a dictionary of genres to store their respective value counts


```python
nominated_genre_count = {}       
for col in genre_cols:                       #iterate through the genre cols to only add genres where the count is 1 and add them to a dictionary
    count = np.sum(awards_to_genres_df[col] == 1).sum() 
    nominated_genre_count[col] = count
```


```python
nominated_genre_count
```


    {'Sport': 3,
     'Musical': 2,
     'Action': 12,
     'Comedy': 19,
     'Thriller': 16,
     'Drama': 78,
     'Sci-Fi': 9,
     'War': 3,
     'Animation': 10,
     'Family': 5,
     'Documentary': 9,
     'Music': 8,
     'Fantasy': 7,
     'Crime': 8,
     'Western': 1,
     'Horror': 4,
     'Mystery': 5,
     'Adventure': 24,
     'Biography': 32,
     'Romance': 14,
     'History': 12}


Now let's sort this dictionary by number of nominations!


```python
sorted_genre_count = {k: v for k, v in sorted(nominated_genre_count.items(), key=lambda item: item[1])}

#here, we iterated through each key-value pair and assigned the sorting key based on the second [1] item, the value

```
```python
sorted_genre_count
```


    {'Western': 1,
     'Musical': 2,
     'Sport': 3,
     'War': 3,
     'Horror': 4,
     'Family': 5,
     'Mystery': 5,
     'Fantasy': 7,
     'Music': 8,
     'Crime': 8,
     'Sci-Fi': 9,
     'Documentary': 9,
     'Animation': 10,
     'Action': 12,
     'History': 12,
     'Romance': 14,
     'Thriller': 16,
     'Comedy': 19,
     'Adventure': 24,
     'Biography': 32,
     'Drama': 78}


Now we are separating the keys and values of the dictionary for plotting.


```python
genres = list(sorted_genre_count.keys())
number_nominated = list(sorted_genre_count.values())
```

### Visualization


```python
import seaborn as sns
import matplotlib.pyplot as plt
sns.set(style="darkgrid", context="talk")

f, (ax1) = plt.subplots(figsize=(12, 8), sharex=True)

sns.barplot(x=number_nominated, y=genres, palette=("rocket_r"), ax=ax1)
ax1.set_ylabel("Genres", fontsize=20)
ax1.set_xlabel("Number of Films Awarded")
ax1.set_title("Number of Academy-Award Winners Since 2010 by Genre");
```


![png](Orin_Individual_Question_Final_files/Orin_Individual_Question_Final_60_0.png)


This chart tells us that Drama films win the most Academy Awards by far! With Biographies and Aventure films following in 2nd and 3rd, respectively.

As most films are a combination of 2 or 3 genres, I would suggest Microsoft ties dramatic themes to their films, and not stick by purely action or purely sci-fi for example. 

