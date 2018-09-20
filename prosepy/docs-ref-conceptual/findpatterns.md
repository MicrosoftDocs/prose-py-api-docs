---
title: Find patterns in strings with Code Accelerator - Python
ms.date: 09/24/2018
ms.topic: conceptual
ms.service: prose-codeaccelerator
author: simmdan
ms.author: dsimmons
description: LEarn how to automatically identify different formats and patterns in input string data with PROSE Code Accelerator for Python.
---

# Find patterns in strings with Code Accelerator

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
# option 0 (default): builder.Target = cx.Target.pandas   # Produces code to operate on pandas dataframes
# option 1          : builder.Target = cx.Target.pyspark  # Produces code to operate on pyspark dataframes
# option 2          : builder.Target = cx.Target.auto     # Produces code to operate on individual strings
result = builder.learn()

result.data()
# produce a dictionary mapping pattern names to sub-lists of the inputs that match the pattern.

result.regexes
# produce a list of regexes corresponding to the patterns

result.code(task=result.Tasks.classify)
# produce a function that takes a data frame (pandas or pyspark, depending on
# the target) and a column name, and returns the data frame grouped by the
# the produced patterns in the column.

result.code(task=result.Tasks.check)
# produce a function that takes a data frame (pandas or pyspark, depending on
# the target) and a column name, and asserts that all values in the column
# the produced patterns in the column.

result.code() # Equivalent to result.code(task=result.Tasks.classify)
```

## Extraction and standardization

**Find Patterns Operation** is designed not only to identify patterns in input strings and
cluster or check the given data per these patterns, but to also produce code that
can be easily modified to perform further operations, such as extracting relevant parts of the strings
or normalizing the data. We illustrate these use cases.

Suppose we have with us a `pandas` data frame having columns `Name` and `BirthDate`. We want to process
this data frame to end up with a `LastName` and `BirthYear` column. Let us begin by eyeing the data frame.
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

Already, in the first 5 rows, the data looks to be in several different formats which all need to be
handled differently. So, we need go through the data frame carefully to ensure that our processing
functions don't assume the wrong format. Or, we can use the **Find Patterns Operation**.

### Extracting Last Names

We sample a small number of names from the data frame and use it to seed find patterns.

```python
# Let's take care of the names first.

sample_names = df['Name'].sample(25, random_state=0).tolist()
b = cx.FindPatternsBuilder(sample_names)
r = b.learn()

# Check that the set of patterns we have learned from the sample
# matches all names in the data frame. If it doesn't this next line
# will fail an assertion.
r.code(task=r.Tasks.check)(df, 'Name')
r.code()
```

This produces some sample code to cluster the data frame by the format of the name. For instance,
you could run `classify(df, 'Name').plot()` to see how frequently the name formats occur.

```python
import regex

titleword_whitespace_titleword_whitespace_titleword = regex.compile(r'^[A-Z][a-z]+[\s][A-Z][a-z]+[\s][A-Z][a-z]+$')
titleword_whitespace_lowers_whitespace_titleword = regex.compile(r'^[A-Z][a-z]+[\s][a-z]+[\s][A-Z][a-z]+$')
titleword_whitespace_titleword___titleword = regex.compile(r'^[A-Z][a-z]+[\s][A-Z][a-z]+-[A-Z][a-z]+$')
titleword_whitespace_titleword = regex.compile(r'^[A-Z][a-z]+[\s][A-Z][a-z]+$')

def identify_pattern(input_str):
    if input_str is not None and titleword_whitespace_titleword_whitespace_titleword.match(input_str):
        # 'Mukhtar Beatriz Savic', 'Martin Gezahegne Kongsangchai'
        return (True, 'TitleWord & [Space]{1} & TitleWord & [Space]{1} & TitleWord')
    elif input_str is not None and titleword_whitespace_lowers_whitespace_titleword.match(input_str):
        # 'Davit le Roux', 'Marta van Zyl'
        return (True, 'TitleWord & [Space]{1} & [Lower]+ & [Space]{1} & TitleWord')
    elif input_str is not None and titleword_whitespace_titleword___titleword.match(input_str):
        # 'Ariette Woo-Nicolaou', 'Bolormaa Ivanov-Kassios'
        return (True, 'TitleWord & [Space]{1} & TitleWord & Const[-] & TitleWord')
    elif input_str is not None and titleword_whitespace_titleword.match(input_str):
        # 'Eka Nastase', 'Jamuna Pavlovski'
        return (True, 'TitleWord & [Space]{1} & TitleWord')
    else:
        return (False, None)

def classify(df, column):
    return df.groupby(lambda row: identify_pattern(df[column][row]))
```

More importantly, this code can be manually edited easily to extract last names per the
format.

```diff
-def identify_pattern(input_str):
+def extract_last_name(input_str):
     if input_str is not None and titleword_whitespace_titleword_whitespace_titleword.match(input_str):
         # 'Mukhtar Beatriz Savic', 'Martin Gezahegne Kongsangchai'
-        return (True, 'TitleWord & [Space]{1} & TitleWord & [Space]{1} & TitleWord')
+        return input_str.split()[-1]  # Ignore the middle name
     elif input_str is not None and titleword_whitespace_lowers_whitespace_titleword.match(input_str):
         # 'Davit le Roux', 'Marta van Zyl'
-        return (True, 'TitleWord & [Space]{1} & [Lower]+ & [Space]{1} & TitleWord')
+        return input_str.split(maxsplit=1)[1]  # Take everything after the first name
     elif input_str is not None and titleword_whitespace_titleword___titleword.match(input_str):
         # 'Ariette Woo-Nicolaou', 'Bolormaa Ivanov-Kassios'
-        return (True, 'TitleWord & [Space]{1} & TitleWord & Const[-] & TitleWord')
+        return input_str.split()[-1]  # Take both halves of the double barrel
     elif input_str is not None and titleword_whitespace_titleword.match(input_str):
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

We now demonstrate a similar extraction and standardization operation for the birth years,
but now in the PySpark environment.

```python
sample_dates = df['BirthDate'].sample(25, random_state=0).tolist()
b = cx.FindPatternsBuilder(sample_dates)

# Now, set the target to pyspark
b.target = cx.Target.pyspark
r = b.learn()
r.code()
```

This now produces a clustering function that handles pyspark data frames.
```python
digit2___titleword___digit2 = '^[0-9]{2}-[A-Z][a-z]+-[0-9]{2}$'
digit2_whitespace_titleword_whitespace_digit4 = r'^[0-9]{2}[\s][A-Z][a-z]+[\s][0-9]{4}$'
digit4 = '^[0-9]{4}$'
Unknown = '^Unknown$'

from pyspark.sql import functions

def classify(df, column):
    identify_pattern = functions \
        .when(df[column].rlike(digit2___titleword___digit2), '[Digit]{2} & Const[-] & TitleWord & Const[-] & [Digit]{2}') \
        .when(df[column].rlike(digit2_whitespace_titleword_whitespace_digit4), '[Digit]{2} & [Space]{1} & TitleWord & [Space]{1} & [Digit]{4}') \
        .when(df[column].rlike(digit4), '[Digit]{4}') \
        .when(df[column].rlike(Unknown), 'Const[Unknown]') \

    return df.groupBy(identify_pattern.alias('identify_pattern'))
```

Again, we can modify this code to extract the dates from each date format.
```diff
+  from pyspark.sql.functions import concat, lit

-def classify(df, column):
+def standardize_year(df, column):
-    identify_pattern = functions \
+  extract_and_standardize_year = functions \
-        .when(df[column].rlike(digit2___titleword___digit2), '[Digit]{2} & Const[-] & TitleWord & Const[-] & [Digit]{2}') \
+        .when(df[column].rlike(digit2___titleword___digit2), concat(lit('19'), df[column].substr(-2, 2))) \
-        .when(df[column].rlike(digit2_whitespace_titleword_whitespace_digit4), '[Digit]{2} & [Space]{1} & TitleWord & [Space]{1} & [Digit]{4}') \
+        .when(df[column].rlike(digit2_whitespace_titleword_whitespace_digit4), df[column].substr(-4, 4)) \
-        .when(df[column].rlike(digit4), '[Digit]{4}') \
+        .when(df[column].rlike(digit4), df[column]) \
-        .when(df[column].rlike(Unknown), 'Const[Unknown]') \
+        .when(df[column].rlike(Unknown), lit(None)) \

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
