
---
title: Read a CSV file
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: prose-codeaccelerator
---

# Read a CSV File

The `ReadCsvBuilder` will analyze a given delimited text file (it doesn't actually need to be comma-separated file, it
could be delimited other ways) and determine all the details about that file necessary to successfully parse it and
produce a dataframe (either `pandas` or `pyspark`).  This includes the encoding, the delimiter, how many lines to skip at
the beginning of the file, etc.

> [!NOTE] The `ReadCsvBuilder` explicitly reads columns as strings to prevent loss of precision during reading the data.
> It is recommended to use `DetectTypesBuilder` to detect and fix the datatypes after reading the file. 

## Usage

``` python
import prose.codeaccelerator as cx

builder = cx.ReadCsvBuilder(path_to_file)
# optional: builder.target = 'pyspark' to switch to `pyspark` target (default is 'pandas')
result = builder.learn()
result.data(5) # examine top 5 rows to see if they look correct
result.code() # generate the code in the target
```

> [!NOTE]
> All examples assume `import prose.codeaccelerator as cx`.

## Example: Read a CSV using pyspark

```python
>>> b = cx.ReadCsvBuilder('some_file.txt')
>>> b.target = 'pyspark'
>>> r = b.learn()
>>> r.code()
from pyspark.sql import SparkSession
from pyspark.sql.types import StructType, StructField, StringType

def read_file(file):
    spark = SparkSession.builder.getOrCreate()

    schema = StructType([
        StructField("column1", StringType(), True),
        StructField("column2", StringType(), True),
        StructField("column3", StringType(), True)])

    df = spark.read.csv(file,
        sep = ",",
        header = True,
        schema = schema,
        quote = "\"",
        escape = "\"",
        ignoreLeadingWhiteSpace = True,
        multiLine = True)
    return df

```

## Example: Specify the number of lines to analyze

```python
>>> b = cx.ReadCsvBuilder('some_file.txt')
>>> b.lines_to_analyze = 500
>>> r = b.learn()
>>> r.code()
...

```
