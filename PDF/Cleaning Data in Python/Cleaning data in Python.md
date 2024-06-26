# Data Cleaning in Python :

Why clean data :
It's crucial to clean the data we use as it **_directly influence the insights_** we extract from our data.
Garbage in Garbage out.

### Data type constraints :
[chapter1.pdf](chapter1.pdf)
Text data : str
Numeric data : int, float
Dates and times : datetime
Categories : category
Binary : bool
To check the data types used in a Pandas dataframe we can use :
```python
df.dtypes
#or
df.info()
```
Examples of the importance of data types :
```python
sales['Revenue'].sum()
#return "214$41$24312$1313$12313$"
#meaning that the data type was a string to fix that :
sales['Revenue'] = sales['Revenue'].str.strip('$')
sales['Revenue'] = sales['Revenue'].astype('int')
assert sales['Revenue'].dtype == 'int'
```

#### Categorical data :
Numeric or categorical ? 
marriage status : single:1, married:2, divorced:3
This is a categorical data, but it's stored as a numeric data.
To fix that we can use the .astype() method to convert the data to a category type.
```python
sales['marriage_status'] = sales['marriage_status'].astype('category')
sales.describe()
```
#### Data in range constraint :
if for example a movie rating is supposed to be with 0 and 5, but we have a 10 in the data, we are sure that this is a mistake because it well beyond the range.
or for example having dates in the future.
for this last example  we could do for example :
```python
import datetime as dt
today = dt.date.today()
#convert to date : 
user_signups['signup_date'] = pd.to_datetime(user_signups['signup_date'])
user_signups[user_signups['signup_date'] > today]
```
We can drop the out of range values, but this is only preferable if the number of outlays is small.
We can set maximums and minimums.
We can treat the data as missing or impute.
or assign a custom value
```python
movies[movies['avg_rating'] > 5] #select the movies with a rating higher than 5
#we can drop them :
movies = movies[movies['avg_rating'] <= 5]
#or by thr .drop() method
movies.drop(movies[movies['avg_rating'] > 5].index, inplace=True)

#we can set a maximum
movies.loc[movies['avg_rating'] > 5, 'avg_rating'] = 5



```
#### Duplicate data / uniqueness :
finding duplicates values : 
```python
duplicates = df.duplicated()
print(duplicates) #True or False values
df[duplicates] #to see the duplicated values
df.duplicated(subset=['column1', 'column2']) #to check for duplicates in a subset of columns
df.drop_duplicates(keep = ) # "first" or "last" or False(keep all duplicates) 
#drop complete duplicates (same values for all columns):
df.drop_duplicates(inplace=True)
#for incomplete duplicates with different values we can combine them with avg : .groupby() and .agg()
df.groupby(by=["name", "family_name","adres"]).agg({'height':'max',"weight":"mean"}).reset_index()

```

#### Membership constraints :
[chapter2.pdf](chapter2.pdf)
Texts and categorical variables .
##### categorical data.
they are often incoded as intgeres. they can't have values that go beyond the predefined values.
Dropping data :
it's good practice to have a log of all possible categorical values. "categories".
types of joins : 
A Anti joins B => data in A that aren't present in B of a common column
A Inner join B => data in A that are present in B of a common column
```python
incosistent_categories = set(df['column']).difference(categories['column'])
incosistent_rows = df['column'].isin(incosistent_categories)
#drop incosistent rows 
consisent_data = df[~incosistent_rows]
```
Some other problems : 
- Value inconsistency : "married", "Married", "married " :
```python
column = df["column"]
column.value_counts()  #for series
#or :
df.groupby("column").count() #for datagrames

#for a capitalization problem we can 
df["column"] = df[column].str.lower()
#or
df["column"] = df[column].str.upper()

#leading or trailing spaces  :
df["column"] = df[column].str.strip()

#collapsing data into categories :
group_names = ['0-20k', '20k-40k', '40k-60k', '60k-80k', '100k+']
ranges = [0,20000, 40000, 60000, 80000, 100000,np.inf ]
df["new_column"] = pd.cut(df["column"], bins = ranges, labels = group_names)
df[['column', 'new_column']].head()

#mapping categories to fewer ones
mapping ={"old_val1": "new_val1", "old_val2": "new_val2","old_val3": "new_val1", "old_value4":"new_val 1"}
df["column"] = df["column"].replace(mapping)
```
- Collapsing too many categories to few : 0-20k$ , 20k-40k$,.. => rich , 0-1k : poor
- making sure data is of type category.

### Cleaning text data :[chapter3.pdf](chapter3.pdf)
handling inconsistency
fixed lenghts
typos
replacing with regular expressions :
```python
phones['phone number'] = phones['phone number'].str.replace(r'\D+', '',regex=True)
```

#### Uniformity :
Unit uniformity :
temprature in C° and F° weight in KF or Lbs, money in $ or €. having different units in the same column can cause misinterpretations.
```python
#we can visualize out of bound data points :
plt.scatter(df['weight'], df['height'])
# or for temperature :
plt.scatter(df['date'], df['temperature'])
#to convert all temperatures :
temp_fah = temperatures.loc[temperatures['temp']>50, 'temp']
temp_cel = (temp_fah - 32) * 5/9
temperatures.loc[temperatures['temp']>50, 'temp'] = temp_cel
```
using dates 
```python
birthdays['birthday'] = pd.to_datetime(birthdays['birthday'])
# but will raise an error
# we can use the errors parameter to handle the error and infer_datetime_format to speed up the process of identifying the formats :
birthdays['birthday'] = pd.to_datetime(birthdays['birthday'], errors='coerce', infer_datetime_format=True)
#NaT for dates that didnt work
#convert the date format :
birthdays['birthday'] =birthdays['birthday'].dt.strftime('%Y-%m-%d')
```

### Cross field validation : for diagnosing data quality issues
use multiple fieds in a dataset to sanity check data integrity : 
economy_class + business_class + first_class = total_seats


```python
sum_classes = flights[['economy_class', 'business_class', 'first_class']].sum(axis=1)
passenger_equ = sum_classes == flights['total_passengers']
incosistent = flights[~passenger_equ]
consisent = flights[passenger_equ]

users['birthdays'] = pd.to_datetime(users['birthdays'])
today = dt.date.today()
age_manual = today.year- users['birthdays'].dt.year
age_equ = age_manual == users['age']
inconsistents = users[~age_equ]
consistents = users[age_equ]
```
or like :
today().year - users['birthdays'].dt.year ==0

What to do with inconsistent data :
drop it, set to missing, apply rules from domain knowledge.

### Completeness : (missing data)
can be represented using NA, nan, 0 , . etc..
we can find which rows have missing data using :
```python  
missing = df.isna()
missing.sum()
import missingno as msno 
import matplotlib.pyplot as plt
msno.matrix(df)
plt.show()
missing_data = df[df['column'].isna()]
missing_date.describe()
#sometimes we can findout that the missing data occurs on an interval say low temperatures for a temperature measurement device :
sorted_data = df.sort_values('temperature')
msno.matrix(sorted_data)
plt.show()
```

So we conclude that there are multiple types of missingness, :
- missing completely at random : no systematic replationship between missing data and other values
- missing at random systematic  relationship between missing data and other observed values
- missing not at random : systematic relationship between missing data and other unobserved values.

To deal with missing data :
- drop missing data
```python
df = df.dropna(subset=['column'])
```
- impute with statistical measures(mean, median, mode..) 
```python
column_mean = df['column'].mean()
df = df.fill_na(value = {'column': column_mean})
```
- impute using an algorithm
- impute using machine learning models

### Compare strings 
[chapter4.pdf](chapter4.pdf)
minimum edit distance
![img.png](Static/med.png)

We're gonna compare strings using the Levenshtein Algorithm : thefuzz package
```python
from thefuzz import fuzz
#compare two strings
fuzz.WRation("Adam", "Madam") #0-100 : 100 exact match

#with arrays :
from thefuzz import process
process.extract("Adam", ["Madam", "Adams", "Adams"], limit=2)
```

#### collapsing categories :
"Eu", "eur", "Europ" => "europe"
```python
for state in categories["state"]: #correct
    matches = process.extract(state, survey["state"], limit = survey.shape[0])
    for potential_match in matches:
        if potential_match[1] >= 85:
            survey.loc[survey["state"] == potential_match[0], "state"] = state
```
## Record linkage
linking data from different sources regarding the same entity (can have diffrent names for the same row)
![img.png](Static/record_linkage.png)
 we use the recordlinkage package
Matching all possible pairs is not feasible especially if we're dealing with millions of records.
Therefore, we use Blocking meaning we create pairs base on a matching column
````python
import recordlinkage
indexer = recordlinkage.Index() #object we can use tog eerate pairs

indexer.block('state')
pairs = indexer.index(df1,df2) #array of possible pairs of indices 
#finding potential matches
compare_cl = recordlinkage.Compare()
#find exact matches for pairs :
compare_cl.exact('first_name', 'Name', label='first_name')
compare_cl.string('address', 'addresse', label='address') #label column name in the resulting dataframe

#for columns with similar values :
compare_cl.string('Last_name', 'Family Name', threshold=0.8, label='last_name')
compare_cl.string("city", "City", threshold=0.8, label="city")

#find matches
potential_matches = compare_cl.compute(pairs, df1, df2)

#finding the pairs : 
potential_matches[potential_matches.sum(axiss = 1)=> 2]


````

## Linking dataframes :

after comparing the pairs and scoring them and finding potential matches it's now time to link the dfs :
![img.png](img.png)

````python
#we isolate matches :
matches = potential_matches[potential_matches.sum(axiss = 1)=> 3] #too many constraints 

duplicate_rows = matches.index.get_level_values(1) ## of df2 or get_level_values("id_column_name")
df2_new = df2[~df2.index.isin(duplicate_rows)]

full_df = df1.append(df2_new)
````
## Possible operations on a dataframe column : 
```python
df["lol"].sum()
df["lol"].str.strip("something")
df["lol"].count()
df["lol"].mean()
df["lol"].median()
df["lol"].max()
df["lol"].min()
df["lol"].sort_values(by = "column_name")
df["lol"].unique() #return the unique values
df["column"] = df["column"].replace(mapping) #replace values with new ones uisng the mapping dictionary
df["column"].str.replace("+", "lol") #replace the + with lol
df.loc[df["cloimn"]<10, "column"] = np.nan #replace the values that are less than 10 with NaN
assert df["column"].str.contains("+|-").any() == False #check if the column contains + or -
```

