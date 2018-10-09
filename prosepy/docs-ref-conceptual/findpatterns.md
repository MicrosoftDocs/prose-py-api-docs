---
title: Find patterns in strings with the Microsoft PROSE Code Accelerator SDK - Python
ms.date: 09/24/2018
ms.topic: conceptual
ms.service: non-product-specific
author: simmdan
ms.author: dsimmons
description: Learn how to automatically identify different formats and patterns in input string data with PROSE Code Accelerator for Python.
---

# Find patterns in strings with the Microsoft PROSE Code Accelerator SDK

Have you ever written a script to perform a string transformation and have it either crash or
produce wrong results silently due to unexpected formats in your input data? Or do you want
to figure out how many different cases you need to handle in your standardization procedure?
**Find Patterns Operation** automatically identifies different formats and patterns in
string data.  Given a set of input strings, **Find Patterns Operation** produces a small number
of regular expressions such that they together match all the input strings, except possibly a
small fraction of outliers.

## Usage

``` python
import prose.codeaccelerator as cx

builder = cx.FindPatternsBuilder(inputs)  # Inputs need to be list of strings
# option 0 (default): builder.target = 'pandas'   # Produces code to operate on pandas dataframes
# option 1          : builder.target = 'pyspark'  # Produces code to operate on pyspark dataframes
# option 2          : builder.target = 'auto'     # Produces code to operate on individual strings
result = builder.learn()

result.data()
# produce a dictionary mapping pattern names to sub-lists of the inputs that match the pattern.

result.regexes
# produce a list of regexes corresponding to the patterns

result.code(task='classify')
# produce a function that takes a dataframe (pandas or pyspark, depending on
# the target) and a column name, and returns the dataframe grouped by the
# the produced patterns in the column.

result.code(task='check')
# produce a function that takes a dataframe (pandas or pyspark, depending on
# the target) and a column name, and asserts that all values in the column
# the produced patterns in the column.

result.code() # Equivalent to result.code(task='classify')
```

## Extraction and standardization

**Find Patterns Operation** is designed not only to identify patterns in input strings and
cluster or check the given data per these patterns, but to also produce code that
you can easily modify to perform further operations, such as extracting relevant parts of the strings
or normalizing the data. This section illustrates these use cases.

Suppose you have with us a `pandas` DataFrame with columns `Name` and `BirthDate`. You want to process
this DataFrame to end up with a `LastName` and `BirthYear` column. First, review the DataFrame.
The data and notebooks used in this use case are freely available.

```python
import pandas as pd
import prose.codeaccelerator as cx


df = pd.read_csv('test_data.csv', names=['Name', 'BirthDate'], skipinitialspace=True)
df.head()
```
|   |Name                      |BirthDate      |
|---|:-------------------------|:--------------|
| 0 |Bertram du Plessis        |1995           |
| 1 |Naiara Moravcikova        |Unknown        |
| 2 |Jihoo Spel                |2014           |
| 3 |Viachaslau Gordan Hilario |22-Apr-67      |
| 4 |Maya de Villiers          |19-Mar-60      |

From the first 5 rows, you can see that the data looks is in several different formats that all must be
handled differently. You will need go through the DataFrame carefully to ensure that the processing
functions don't assume the wrong format. Or, you can use the **Find Patterns Operation**.

### Extracting Last Names

Sample a small number of names from the pandas DataFrame and use it to seed find patterns.

```python
# Let's take care of the names first.

sample_names = df['Name'].sample(25, random_state=0).tolist()
b = cx.FindPatternsBuilder(sample_names)
r = b.learn()

# Check that the set of patterns you have learned from the sample
# matches all names in the dataframe. If it doesn't this next line
# will fail an assertion.
r.code(task='check')(df, 'Name')
r.code()
```

This produces some sample code to cluster the DataFrame by the format of the name. For instance,
you could run `classify(df, 'Name').plot()` to see how frequently the name formats occur.

```python
import regex

pattern_0 = regex.compile(r'^[A-Z][a-z]+[\s][A-Z][a-z]+[\s][A-Z][a-z]+$')
pattern_1 = regex.compile(r'^[A-Z][a-z]+[\s][a-z]+[\s][A-Z][a-z]+$')
pattern_2 = regex.compile(r'^[A-Z][a-z]+[\s][A-Z][a-z]+-[A-Z][a-z]+$')
pattern_3 = regex.compile(r'^[A-Z][a-z]+[\s][A-Z][a-z]+$')

def identify_pattern(input_str):
    if input_str is not None and pattern_0.match(input_str):
        # 'Mukhtar Beatriz Savic', 'Martin Gezahegne Kongsangchai'
        return (True, 'TitleWord & [Space]{1} & TitleWord & [Space]{1} & TitleWord')
    elif input_str is not None and pattern_1.match(input_str):
        # 'Davit le Roux', 'Marta van Zyl'
        return (True, 'TitleWord & [Space]{1} & [Lower]+ & [Space]{1} & TitleWord')
    elif input_str is not None and pattern_2.match(input_str):
        # 'Ariette Woo-Nicolaou', 'Bolormaa Ivanov-Kassios'
        return (True, 'TitleWord & [Space]{1} & TitleWord & Const[-] & TitleWord')
    elif input_str is not None and pattern_3.match(input_str):
        # 'Eka Nastase', 'Jamuna Pavlovski'
        return (True, 'TitleWord & [Space]{1} & TitleWord')
    else:
        return (False, None)

def classify(df, column):
    return df.groupby(lambda row: identify_pattern(df[column][row]))
```

More importantly, you can easily edit this code manually to extract last names per the
format.

```diff
-def identify_pattern(input_str):
+def extract_last_name(input_str):
     if input_str is not None and pattern_0.match(input_str):
         # 'Mukhtar Beatriz Savic', 'Martin Gezahegne Kongsangchai'
-        return (True, 'TitleWord & [Space]{1} & TitleWord & [Space]{1} & TitleWord')
+        return input_str.split()[-1]  # Ignore the middle name
     elif input_str is not None and pattern_1.match(input_str):
         # 'Davit le Roux', 'Marta van Zyl'
-        return (True, 'TitleWord & [Space]{1} & [Lower]+ & [Space]{1} & TitleWord')
+        return input_str.split(maxsplit=1)[1]  # Take everything after the first name
     elif input_str is not None and pattern_2.match(input_str):
         # 'Ariette Woo-Nicolaou', 'Bolormaa Ivanov-Kassios'
-        return (True, 'TitleWord & [Space]{1} & TitleWord & Const[-] & TitleWord')
+        return input_str.split()[-1]  # Take both halves of the double barrel
     elif input_str is not None and pattern_3.match(input_str):
         # 'Eka Nastase', 'Jamuna Pavlovski'
-        return (True, 'TitleWord & [Space]{1} & TitleWord')
+        return input_str.split()[-1]  # Take the last name
     else:
-        return (False, None)
+        return None

+def add_last_name(df, column):
+    df['LastName'] = df[column].apply(extract_last_name)
-def classify(df, column):
-    return df.groupby(lambda row: identify_pattern(df[column][row]))

+add_last_name(df, 'Name')
+df.head()
```

This will produce:

|   |Name                      |BirthDate      |LastName       |
|---|:-------------------------|:--------------|:--------------|
| 0 |Bertram du Plessis        |1995           |du Plessis     |
| 1 |Naiara Moravcikova        |Unknown        |Moravcikova    |
| 2 |Jihoo Spel                |2014           |Spel           |
| 3 |Viachaslau Gordan Hilario |22-Apr-67      |Hilario        |
| 4 |Maya de Villiers          |19-Mar-60      |de Villiers    |

### Standardizing Birth Years

The following example demonstrates a similar extraction and standardization operation for the birth years,
but now in the PySpark environment.

```python
sample_dates = df['BirthDate'].sample(25, random_state=0).tolist()
b = cx.FindPatternsBuilder(sample_dates)

# Now, set the target to pyspark
b.target = 'pyspark'
r = b.learn()
r.code()
```

This now produces a clustering function that handles PySpark DataFrames.
```python
pattern_0 = '^[0-9]{2}-[A-Z][a-z]+-[0-9]{2}$'
pattern_1 = r'^[0-9]{2}[\s][A-Z][a-z]+[\s][0-9]{4}$'
pattern_2 = '^[0-9]{4}$'
pattern_3 = '^Unknown$'

from pyspark.sql import functions

def classify(df, column):
    identify_pattern = functions \
        .when(df[column].rlike(pattern_0), '[Digit]{2} & Const[-] & TitleWord & Const[-] & [Digit]{2}') \
        .when(df[column].rlike(pattern_1), '[Digit]{2} & [Space]{1} & TitleWord & [Space]{1} & [Digit]{4}') \
        .when(df[column].rlike(pattern_2), '[Digit]{4}') \
        .when(df[column].rlike(pattern_3), 'Const[Unknown]') \

    return df.groupBy(identify_pattern.alias('identify_pattern'))
```

Again, you can modify this code to extract the dates from each date format.

```diff
+  from pyspark.sql.functions import concat, lit

-def classify(df, column):
+def standardize_year(df, column):
-    identify_pattern = functions \
+  extract_and_standardize_year = functions \
-        .when(df[column].rlike(pattern_0), '[Digit]{2} & Const[-] & TitleWord & Const[-] & [Digit]{2}') \
+        .when(df[column].rlike(pattern_0), concat(lit('19'), df[column].substr(-2, 2))) \
-        .when(df[column].rlike(pattern_1), '[Digit]{2} & [Space]{1} & TitleWord & [Space]{1} & [Digit]{4}') \
+        .when(df[column].rlike(pattern_1), df[column].substr(-4, 4)) \
-        .when(df[column].rlike(pattern_2), '[Digit]{4}') \
+        .when(df[column].rlike(pattern_2), df[column]) \
-        .when(df[column].rlike(pattern_3), 'Const[Unknown]') \
+        .when(df[column].rlike(pattern_3), lit(None)) \

-    return df.groupBy(identify_pattern.alias('identify_pattern'))
+    return df.withColumn('BirthYear', extract_and_standardize_year)


+df = standardize_year(df, 'BirthDate')
+df.show()
```

|Name                      |BirthDate       |BirthYear |
|:-------------------------|:---------------|---------:|
|Bertram du Plessis        |1995            |      1995|
|Naiara Moravcikova        |Unknown         |      null|
|Jihoo Spel                |2014            |      2014|
|Viachaslau Gordan Hilario |22-Apr-67       |      1967|
|Maya de Villiers          |19-Mar-60       |      1960|
|Sandeep du Plessis        |08-Jul-62       |      1962|
|Tijana Albert Shrivastav  |05 November 1951|      1951|
