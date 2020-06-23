
## Readme
[link to my data analysis](./student.ipynb)

**The format for the readme should be as follows**:

    Goals and Objectives
    The goal of the project is to provide Mircosoft with a recomnaded list of genres and what the typical budget for each genres. 
    
    The data is genreated from: 
        Box Office Mojo
        IMDB
        Rotten Tomatoes
        TheMovieDB.org


    Question 1: what are the 3 top genres and who are the top 3 producers and writers?
    
     1. Describe the EDA done to get the data, include any table joins performed and any sort of cleaning.
    
    To get the data, the following data cleaning was done:
    
        1. Converted the data type from string to float
        
        2. Join imdb_title_basics with imdb_title_ratings by the tconst column. Converted genres to string and splited the genres into a list for each row. 
        
        3. Join this table with the budgets table by title.
        
        4. Select only column of interest which are genres, averge rating, numvotes and  find the means.
        
        5. Sort to get a list of genres by numvotes. The numvotes should correspond to their popularity.
        
        6. Repeat the same process to find out the budgets for each genres.
        
        7. Repeat step 1 to 6 with the writers and procuders to get the top writers and producers using title budget table with imbd_title_crew table. 
        
    Question 2: what the target production budget should be?
  
        To find the production budget for the genres of interest, I used the table created in step 3 and select only the genres to find out the average budgets for each using ".iloc" method. 
    

    2. Note down any findings/recommendations/future work you may have
        1. List all the data needed for the project.
        2. Create a flow chart starting with the end goal. Update the chart nescessary to reflect the most current findings. Make changes to the plan if nescessary.
        3. Keep track of the time spent on each part of the project and set out daily goal to acheive.
        4. Communicate with the team regard of any changes in the plan. 
        5. Keep track of a list of common coding mistakes to avoid or debug 
    


```python

```
