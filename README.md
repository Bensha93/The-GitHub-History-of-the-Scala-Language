# The-GitHub-History-of-the-Scala-Language


## Scala Open Source Project Analysis

This project investigates the real-world data of Scala, a general-purpose programming language, focusing on its open-source repository. The data covers almost 30k commits over a span of 10 years and provides insights into the contributions to the Scala project, user activity, and community dynamics. The data is sourced from GitHub and is available in three CSV files.

### **Dataset Description**

- **pulls_2011-2013.csv**: Contains basic pull request information from 2011 to 2013.
- **pulls_2014-2018.csv**: Contains pull request information from 2014 to 2018.
- **pull_files.csv**: Contains the files modified by each pull request.

### **Tasks and Analysis**

## **1. Importing the Data**
We first load the CSV files using `pandas`:

```python
import pandas as pd

# Loading in the data
pulls_one = pd.read_csv('datasets/pulls_2011-2013.csv')
pulls_two = pd.read_csv('datasets/pulls_2014-2018.csv')
pull_files = pd.read_csv('datasets/pull_files.csv')
```

## **2. Preparing and Cleaning the Data**

We combine the data from two different pull request datasets and convert the date columns to DateTime objects.

```python
# Append pulls_one to pulls_two
pulls = pulls_one.append(pulls_two)

# Convert the date for the pulls object
pulls['date'] = pd.to_datetime(pulls['date'], utc=True)
```

## **3. Merging the DataFrames**

We merge the `pulls` and `pull_files` datasets to make the analysis easier.

```python
# Merge the two DataFrames
data = pulls.merge(pull_files, on='pid')
```

## **4. Is the Project Still Actively Maintained?**

To determine the activity level, we plot the number of pull requests submitted each month over the project's lifetime.

```python
%matplotlib inline

# Create a column that will store the month
data['month'] = data['date'].dt.month

# Create a column that will store the year
data['year'] = data['date'].dt.year

# Group by the month and year and count the pull requests
counts = data.groupby(['year', 'month'])['pid'].count()

# Plot the results
counts.plot(kind='bar', figsize = (12,4))
```

![download (7)](https://github.com/user-attachments/assets/75c2324f-772d-45f5-aa2c-ddc1129bbcdd)


## **5. Is There Camaraderie in the Project?**

To evaluate the project's community dynamics, we plot a histogram of the number of pull requests submitted by each user.

```python
%matplotlib inline

# Group by the submitter
by_user = data.groupby('user').agg({'pid': 'count'})

# Plot the histogram
by_user.hist()
```

![download (8)](https://github.com/user-attachments/assets/4d4a4e4b-ddcf-4182-9bb0-7d410c8aefd2)


## **6. What Files Were Changed in the Last Ten Pull Requests?**

We analyze the last 10 pull requests to identify which files were modified.

```python
# Identify the last 10 pull requests
last_10 = pulls.sort_values(by = 'date').tail(10)

# Join the two data sets
joined_pr = pull_files.merge(last_10, on='pid')

# Identify the unique files
files = set(joined_pr['file'])

# Print the results
files
```

## **7. Who Made the Most Pull Requests to a Given File?**

We find the top 3 developers who contributed most to the file `src/compiler/scala/reflect/reify/phases/Calculate.scala`.

```python
# This is the file we are interested in:
file = 'src/compiler/scala/reflect/reify/phases/Calculate.scala'

# Identify the commits that changed the file
file_pr = data[data['file'] == file]

# Count the number of changes made by each developer
author_counts = file_pr.groupby('user').count()

# Print the top 3 developers
author_counts.nlargest(3, 'file')
```

## **8. Who Made the Last Ten Pull Requests on a Given File?**

We find the last 10 contributors to the file `src/compiler/scala/reflect/reify/phases/Calculate.scala`.

```python
# Select the pull requests that changed the target file
file_pr = data[data['file'] == file]

# Merge the obtained results with the pulls DataFrame
joined_pr = data[data['file'] == file]

# Find the users of the last 10 most recent pull requests
users_last_10 = set(joined_pr.nlargest(10, 'date')['user'])

# Printing the results
users_last_10
```

## **9. The Pull Requests of Two Special Developers**

We focus on two specific developers (`xeno-by` and `soc`) and analyze their pull request activity by year.

```python
%matplotlib inline

# The developers we are interested in
authors = ['xeno-by', 'soc']

# Get all the developers' pull requests
by_author = pulls[pulls['user'].isin(authors)]

# Count the number of pull requests submitted each year
counts = by_author.groupby([by_author['user'], by_author['date'].dt.year]).agg({'pid': 'count'}).reset_index()

# Convert the table to a wide format
counts_wide = counts.pivot_table(index='date', columns='user', values='pid', fill_value=0)

# Plot the results
counts_wide.plot(kind='bar')
```
![download (9)](https://github.com/user-attachments/assets/bbae189c-759a-4afb-907b-007402b60c59)


## **10. Visualizing the Contributions of Each Developer**

We analyze the contributions of `xeno-by` and `soc` to the file `src/compiler/scala/reflect/reify/phases/Calculate.scala`.

```python
authors = ['xeno-by', 'soc']
file = 'src/compiler/scala/reflect/reify/phases/Calculate.scala'

# Select the pull requests submitted by the authors, from the `data` DataFrame
by_author = data[data['user'].isin(authors)]

# Select the pull requests that affect the file
by_file = by_author[by_author['file'] == file]

# Group and count the number of PRs done by each user each year
grouped = by_file.groupby(['user', by_file['date'].dt.year]).count()['pid'].reset_index()

# Transform the data into a wide format
by_file_wide = grouped.pivot_table(index='date', columns='user', values='pid', fill_value=0)

# Plot the results
by_file_wide.plot(kind='bar')
```

![download (10)](https://github.com/user-attachments/assets/dfa1dbb7-ac97-46be-a34b-bbadab191ad5)


### **Conclusion**

This analysis of Scala's open-source repository provides valuable insights into the project's activity, contribution trends, and developer dynamics. By understanding who is contributing, what files are being modified, and the project's overall state, we can better target areas for contribution and collaboration.
