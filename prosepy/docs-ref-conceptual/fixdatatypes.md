---
title: Fix Data Types
ms.date: 08/30/2018
ms.topic: conceptual
ms.service: prose-codeaccelerator
---

# Fix Data Types

One common pain point when working with data in python is that often values in columns in a data frame are often
incorrectly given string as their datatype.  An entire column may consist of numbers or dates but with values that are
actually typed as strings which prevents doing logical operations on them.  `DetectTypesBuilder` will examine data and,
if appropriate, produce code to transform the types.  While the underlying pandas and pyspark libraries in some cases
have the ability to infer data types from strings, often the results are less than ideal.  If a column contains a series
of zip codes, for instance, it's much better to keep that column as a string than to transform it to a number and
potentially lose leading zeros or the like.

Example Usage:

``` python
import prose.codeaccelerator as cx

# get a dataframe, this could be other code you already have, but here's a snippet using other parts of Code Accelerator
csvBuilder = cx.ReadCsvBuilder(csv_file)
# optional: csvBuilder.Target = cx.Target.pyspark
csvResult = csvBuilder.learn()
read_file = csvResult.code()
df = read_file(csv_file)
df.dtypes
# notice that all of the column types are strings

# now fix the datatypes
builder = cx.DetectTypesBuilder(df)
# make sure we target the same thing as above
builder.Target = csvBuilder.Target 
result = builder.learn()
result.data.dtypes
# examine new types
result.data(5)
# make sure the data still looks right
result.code()
```