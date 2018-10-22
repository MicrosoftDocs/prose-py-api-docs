---
title: Read a fixed-width file with the Microsoft PROSE Code Accelerator SDK - Python
ms.date: 09/24/2018
ms.topic: conceptual
ms.service: non-product-specific
author: simmdan
ms.author: dsimmons
description: Learn how to analyze a fixed-width file with PROSE Code Accelerator for Python.
---

# Read a fixed-width file with the Microsoft PROSE Code Accelerator SDK

`ReadFwfBuilder` will analyze a fixed-width file and produce code to split the fields yielding a data frame.  When it is
given only the fixed-width input file, Code Accelerator makes every effort to determine the boundaries between fields.
Sometimes, however, this isn't possible. For example, if a file has two separate number fields placed directly next to one
another, there might not be enough information to determine the boundary between them.  So,
`ReadFwfBuilder` optionally also takes a file containing a description of the file schema.  This schema
description does not have to be in an exact format; Code Accelerator will do its best to locate lists of fields and their column
ranges and use that information to generate the code.

> [!NOTE]
> The `ReadFwfBuilder` explicitly reads columns as strings to prevent loss of precision during reading the data.
> It is recommended to use `DetectTypesBuilder` to detect and fix the data types after reading the file. 

## Usage

```python
import prose.codeaccelerator as cx

builder = cx.ReadFwfBuilder(path_to_file, path_to_schema)
# note: path_to_schema is optional (see examples below)
# optional: builder.target = 'pyspark' to switch to `pyspark` target (default is 'pandas')
result = builder.learn()
result.preview_data # examine top 5 rows to see if they look correct
result.code() # generate the code in the target
```

### Example of a fixed-width schema

```text
Name   Start   End    Description
First  1       2      The first thing
Second 3       3      The second, a singleton
Third  4       6      The rest
```

> [!NOTE]
> The following examples assume `import prose.codeaccelerator as cx`.

## Read a fixed-width file with schema

> [!NOTE]
> Assume that `'schema.txt'` contains the schema from the previous example.

```python
>>> b = cx.ReadFwfBuilder('some_file.txt', 'schema.txt')
>>> r = b.learn()
>>> r.code()
import pandas as pd
import csv

def read_file(file):
    columns = [
        ("The_first_thing", (0, 2)),
        ("The_second__a_singleton", (2, 3)),
        ("The_rest", (3, 6)),
    ]

    names, colspecs = zip(*columns)

    df = pd.read_fwf(
        file,
        skiprows=0,
        header=None,
        names=names,
        quoting=csv.QUOTE_NONE,
        doublequote=False,
        colspecs=colspecs,
        skip_blank_lines=False,
        index_col=False,
        dtype=str,
        na_values=[],
        keep_default_na=False,
        skipinitialspace=True,
    )

    return df

```

## Read a fixed-width file without schema

```python
>>> b = cx.ReadFwfBuilder('some_file.txt')
>>> r = b.learn()
>>> r.code()
import pandas as pd
import csv

def read_file(file):
    columns = [
        ("DATE", (0, 12)),
        ("TIME", (12, 24)),
        ("TYPE", (24, 39)),
        ("NAME", (39, None)),
    ]

    names, colspecs = zip(*columns)

    df = pd.read_fwf(
        file,
        skiprows=1,
        header=None,
        names=names,
        quoting=csv.QUOTE_NONE,
        doublequote=False,
        colspecs=colspecs,
        index_col=False,
        dtype=str,
        na_values=[],
        keep_default_na=False,
        skipinitialspace=True,
    )

    return df

```

## Read a fixed-width file - PySpark

```python
>>> b = cx.ReadFwfBuilder('some_file.txt')
>>> b.target = 'pyspark'
>>> r = b.learn()
>>> r.code()
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, monotonically_increasing_id, trim, udf
from pyspark.sql.types import StringType

def read_file(file):
    spark = SparkSession.builder.getOrCreate()

    df = spark.read.text(file)

    df = df.filter(trim(df.value) != "")

    df = df.select(
        trim(df.value.substr(1, 12)).alias("DATE"),
        trim(df.value.substr(13, 12)).alias("TIME"),
        trim(df.value.substr(25, 15)).alias("TYPE"),
        trim(df.value.substr(40, 2147483647)).alias("NAME"),
    )

    df = df.withColumn("_skip_index", monotonically_increasing_id())
    df = df.filter("_skip_index >= 1").drop("_skip_index")

    rstrip_udf = udf(lambda s: None if s is None else s.rstrip("\r"), StringType())
    df = df.withColumn("NAME", rstrip_udf(col("NAME")))

    return df

```

## Specify number of lines to analyze

```python
>>> b = cx.ReadFwfBuilder('some_file.txt')
>>> b.lines_to_analyze = 500
>>> r = b.learn()
>>> r.code()
...

```