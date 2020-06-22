**These days when a new movie is being released, it isn't a question of IF it will be released in another country, but rather HOW MANY other countries? The question we aim to analyze for Microsoft Entertainment Studios is:**

**Are movies that are released in more countries more profitable?**

**The findings in this notebook will show that the answer to that question is in fact: yes. This notebook will walk step-by-step through the importing, cleaning, exploration, and visualization processes undertaken to try and answer Microsoft's question and help inform strategic business decisions going forward.**

**The data being analyzed are from two sources: IMDB country data per movie and Box Office Mojo revenue and budget information. These datasets have been saved into an SQL database previously and will be imported into this notebook further down.**

Data Cleaning & Converting 

**Drop 'None' types from the region feature & check the unique values.**

df_title_region = df_title_region.dropna()
df_title_region['region'].unique()

**We noticed that there were different types of country codes in this list. This led to more research to see how one could check that these were all accurate. We found the |pycountry| library, installed it, then imported it below.**

import pycountry

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

**Great, now we have a new column that identifies the type of alpha code in that row.**

**Let's now convert that to a country name using |pycountry|.**

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

**Looks good, though we can clearly see that there are lots of duplicates, as well as a 'None' in the country feature.**

df_title_region[(df_title_region['alpha_code'] == 'alpha_3')
                | 
                (df_title_region['alpha_code'] == 'alpha_4')
               &
               (df_title_region['country'] == 'None')].head()

**Based on the information provided above, we can see that a lot of country_codes containing alpha_3 OR alpha_4 also have a 'None' value for their country feature, making these rows essentially useless for this analysis.  To proceed, we will drop all rows with the value of 'None' for the 'country' feature.**

df_title_region = df_title_region[df_title_region['country'] != 'None']
df_title_region.head()

df_title_region['alpha_code'].value_counts()

**Looks like all of the alpha_3 codes went along with the 'none' values. Let's see what we have left for alpha_4 codes.**

df_title_region[df_title_region['alpha_code'] == 'alpha_4']

**As we can see above, the countries that were assigned alpha_4 codes are no longer officially recognized today.  For this analysis it is okay to remove these from our dataset.**

df_title_region = df_title_region[df_title_region['alpha_code'] != 'alpha_4']
df_title_region['alpha_code'].value_counts()

**Great, now let's drop the alpha_code and region columns.**

df_title_region = df_title_region.drop(['alpha_code', 'region'], axis=1)
print(df_title_region.shape)
df_title_region.head()

**OK, so now we are only left with the duplicates. Let's drop them now so that later on our lists of countries (for our country_count feature) will be accurate. Let's also update the name of our DF as well.**

df_title_country = df_title_region.drop_duplicates()
print(df_title_country.shape)
df_title_country.head()

**Dropped about 20K duplicates, very nice.  Now let's build out our feature containing a list of each country per movie.**

df_title_countrylist = df_title_country.groupby('primary_title')['country'].apply(list).reset_index(name='country_list')
print(df_title_countrylist.shape)
df_title_countrylist.head()

**We are now down to 114,284 rows, and with the cleaning that we have done up to this point, it seems fair to assume that all of these movies are unique, but just to be safe, lets check with the .value_counts() method.**

df_title_countrylist['primary_title'].value_counts().head()

**Perfect!  Next we are going to make a simple feature that counts the number of countries each movie was released in.  This is easily achieved by calculating the length of the country_list feature for each movie using some nice list comprehension. Then update the DF name to be a bit more descriptive.**

pwd

df_title_countrylist['country_count'] = [len(x) for x in df_title_countrylist['country_list']]
df_title_countrylist_count = df_title_countrylist
df_title_countrylist_count.to_csv(r'C:\Users\tcast\Data Science Program\Module 1\Mod 1 Project - Movie Analysis\Movie_Analysis\Are movies more profitable if they are released in more countries\CLEAN-title_countrylist_count.csv')
# df_title_countrylist_count.head()

**Here we are going to subset the dataframe.  We are going to do so based on whether a movie was either a domestic or international release.  This significant because we will be able to compare these two different release types in the future should we have any questions involving the two.  We will create the domestic dataframe by filtering for movies whose country_count feature has a value of 1.**

df_domestic_title_country = df_title_countrylist_count[
    df_title_countrylist_count[
        'country_count'
    ] == 1
]

df_domestic_title_country = df_domestic_title_country.drop('country_count', axis=1)
# Reassigned variable to avoid 'A value is trying to be set on a copy of a slice from a DataFrame.' error.
df_domestic_title_country['country_list'] = df_domestic_title_country['country_list'].map(lambda x: ''.join(x))
df_domestic_title_country.head()

**Great, now we can see all of the domestic movie releases. If we want to, we can also group this dataframe by its country feature and create a list of domestic movies released in each one respectively.**

df_domestic_country_movie_list = df_domestic_title_country.groupby('country_list')['primary_title'].apply(list).reset_index(name='movie_list')
df_domestic_country_movie_list.head()

**Or we can just see how many domestic movies per country without a list of titles.  We can rearrange this as needed to prepare for a future merge or join with another dataset.**

df_dom_movies_per_country = df_domestic_title_country
df_dom_movies_per_country['movies_per_country'] = 1
df_dom_movies_per_country = df_domestic_title_country.groupby('country_list').sum()
df_dom_movies_per_country.head()

**Now we are going to create out dataframe of international movies, those with more than 1 in their country_count feature.
We then reset the index just to make it look nicer, and to lessen any potential merge or join issues later.**

df_int_title_clist_ccount = df_title_countrylist_count[
    df_title_countrylist_count[
        'country_count'
    ] > 1
]

df_int_title_clist_ccount = df_int_title_clist_ccount.reset_index().drop('index', axis=1)
df_int_title_clist_ccount.head()

## Cleaning & Converting - Domestic & International Budgets/Revenues

**Let's have a loser look at this dataset and see if anything stands out.**

print(df_movie_moneys.info())

df_movie_moneys.head()

**Let's drop the 'id' column here.**

df_movie_moneys = df_movie_moneys.drop('id', axis=1)
df_movie_moneys.head()

**Clean and convert the 'production_budget', 'domestic_gross', and 'worldwide_gross' columns from strings to floats using the function created below**

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

**Convert the 'release_date' column to pandas datetime objects.**

df_movie_moneys['release_date'] = pd.to_datetime(df_movie_moneys['release_date'])
df_movie_moneys.head()

**Excellent, now let's have a closer look at our numeric columns.**

print(df_movie_moneys.shape)
df_movie_moneys.describe()

**Lots to see here.  The first thing to notice is the 0 values in the 'domestic_gross' and 'worldwide_gross' columns.  Let's have a look at what kinds of movies fit this criteria before we decide how to proceed.**

print(df_movie_moneys[(df_movie_moneys['worldwide_gross'] == 0)
                                  |
                                  (df_movie_moneys['domestic_gross'] == 0)].shape)
df_movie_moneys[(df_movie_moneys['worldwide_gross'] == 0)
                                  |
                                  (df_movie_moneys['domestic_gross'] == 0)]

**If we sort by 'release_date' then we can see that some of these movies have not even been released yet.  Furthermore, there are release dates that have since passed and yet these movies still have no financial information.  Finally we have some examples of movies where they have a 'worldwide_gross' figure, yet no 'domestic_gross' figure.  I can think of no good, or logical reason as to why this may be the case other than error in the data, therefore we will drop these rows.  This constitutes a drop of 10% of our data, though it is likely that most of these movies we did not have country information for either.**

df_movie_moneys = df_movie_moneys[(df_movie_moneys['worldwide_gross'] != 0)
                                  &
                                  (df_movie_moneys['domestic_gross'] != 0)]
print(df_movie_moneys.describe())
print(df_movie_moneys.shape)
df_movie_moneys.head()

**Finally, lets's change the feature name from 'movie' to 'primary_title' so as to make future merges less stressful!**

df_movie_moneys = df_movie_moneys.rename(columns={'movie':'primary_title'})
df_movie_moneys.to_csv(r'C:\Users\tcast\Data Science Program\Module 1\Mod 1 Project - Movie Analysis\Movie_Analysis\Are movies more profitable if they are released in more countries\CLEAN-BOM_budget_revenues.csv')
df_movie_moneys.head()

**We have left the rest of the outlier data in for now so that we can have a better look at the data through EDA before determining the best method of dealing with them.**

# Exploration, Feature Engineering, and Visualizations

## Bringing the data together

**The first step in our Data Exploration & Feature Engineering Phase will be to bring our data together and start to have a look at what stands out.**

df_int_movies_analysis_inner = df_movie_moneys.merge(df_int_title_clist_ccount, on='primary_title')
print(df_int_movies_analysis_inner.shape)
df_int_movies_analysis_inner.head()

## Feature Engineering - Adding more metrics

### Foreign Gross, Net Revenue, and Return on Investment

**Great! We are now down to our ~1800 rows.  This is less than half of the original 'bom_movies_gross' data set but we now have a lot more accurate information about each movie.  Before we look at any relationships, let's add a few more important features to this dataset.  Our question is surrounding profitability so it makes sense to add in features related to that measurement.  Let's add a foreign_gross', 'net_revenue', and 'return_on_investment' each to this dataset.**

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

### Add profit/loss feature

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

### Category Feature - Country Bins

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
df_int_movies_analysis_inner

  **That was easy! Now that we have the features that we would liketo see, let's start out with Seaborn's .pairplot and see what kinds of relationships exist.** 

## Pairplot - Initial Visualization

sns.set(context='notebook', style='darkgrid', color_codes=True, palette='muted')

sns.pairplot(df_int_movies_analysis_inner, kind='reg');

**This is great! On our first look we can already see that a positive relationship exists between 'country_count' and our gross figures. Though remember, our question is interested in profitability, not just revenue. The last column on the above matrix shows a very wide range of potential profitability, however the direction that is it going is great! We also observe that there are A LOT of positively-skewed features for budget and revenue distributions.**

**Next, let's take a closer look at our individual features using a boxplot to help us better contextualize the outliers.** 

## Boxplot - Outlier Identification

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

**Whoa! Looks like the data here are littered with outliers on the positive end of the spectrum.  Let's try a few different techniques and see which ones result in a more normalized distribution!**

## Outlier Exploration & Visualizations

**Let's create a new dataframe to use for the following EDA purposes since we have so many outliers in our original dataset.**

df_int_movies_analysis_inner_cleaned = df_int_movies_analysis_inner.copy()
print(df_int_movies_analysis_inner_cleaned.shape)
df_int_movies_analysis_inner_cleaned.head()

**The feature that looks like it contains the most egregious amount of outliers is 'worldwide_gross'. Let's see how the data are affected when we start to preform some outlier cleaning in the cell below.  Let's write a function in the following cell that will take in information about how we want to adjust our dataframe using quantiles. This function will allow us to perform quick EDA, and if we see something that we like, we can just re-define a new dataframe as needed.**

##  Function - Quantile Column Cleaning

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

between_quantile_col_cleaner(df_int_movies_analysis_inner_cleaned, 'worldwide_gross', .10, .90)

## Funtion - Quantile Column Cleaning (Removed Rows)

**Here is a function that shows us the movies that were dropped.  This is a useful function as it allows us to see which movies don't make the cleaning-cut!**

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

between_quantile_col_removed(df_int_movies_analysis_inner_cleaned, 'worldwide_gross', .10, .90)

## Boxplot - Box Office Numbers w/ Outlier Manipulation

 **Ok, let's have a look at our data now using our new function and see how the below plots change. The data between the 10th and 90th percentile looks the most reasonable for this visualization.**

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

## Countplot - Countries Per Movie w/ Outlier Manipulation

**Now lets do the same testing but with a countplot.  This visual looks quite similar regardless of the manner with which it was sliced.  Therefore the full dataset was chosen for the best representation of this visual.**

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

# Microsoft Question Visualization & Conclusion

**Okay, now for the last step.  We have seen above how the data can change depending on how we treat our outliers.  Let's now aim to answer the big question:**

**Are movies that are released in more countries more profitable?**

**Let's create two more plots and put them side by side.  The firs plot will look at the costs of producing these movies, as well as how much money the tend to bring back in.  The second plot will look at the number of movies that come out of the international scene with either a profit or a loss.  The following plots will use the MEDIAN as its estimator fucntion as it is the best tool we have to combat against the many outliers that remain.**

**These last two visualizations will be shown using the data from the 1st to 99th percentile. There were some variations in the plotting depending on how the dataset was sliced, but the reasons were mostly due to not having a large sample size of movies in over 40 coutnries.  Therefore we chose to stick with the subset below in the end.**

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

**These last two visualizations paint the picture quite clearly that in fact yes, movies that are released in more countries are generally more profitable.  We can also make some conclusions about the risks involved in making movies that are meant to be for a global audience.**

# * * Bonus * * - "Domestic" Movie Country Count Estimator

**As we can see below, the IMDB database was not as accurate as we would have liked.  Avatar is still the largest movie ever made in almost all aspects, yet it has a country list consisting only of Japan.  We thought it would be interesting to see, based on the cleaning of the previous data, if we could predict the number of movies that a country was released in based on its financial metrics.  Below we have made a function to check against the 'worldwide_gross' value, though this function could easily be altered to measure against any of the numerical features.**

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

df_int_movies_analysis_inner_cleaned.groupby('country_count_category').median()

**The function below is based on the above table, replace the values as needed to switch between features to compare.**

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

**Feel free to check out your favourite  movie source online to compare the results!** 
