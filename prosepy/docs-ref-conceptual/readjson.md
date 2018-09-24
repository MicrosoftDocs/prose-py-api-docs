---
title: Read a JSON file with the Microsoft PROSE Code Accelerator SDK - Python
ms.date: 09/24/2018
ms.topic: conceptual
ms.service: non-product-specific
author: simmdan
ms.author: dsimmons
description: Learn how to read a JSON file with the Microsoft PROSE Code Accelerator SDK for Python.
---

# Read a JSON file with the Microsoft PROSE Code Accelerator SDK

`ReadJsonBuilder` will produce code to read a JSON file into a data frame.

## Usage

``` python
import prose.codeaccelerator as cx

builder = cx.ReadJsonBuilder('path_to_json_file')
# optional: builder.target = 'pyspark' to switch to `pyspark` target (default is 'pandas')
result = builder.learn()
result.data(5) # examine top 5 rows to see if they look correct
result.code() # generate the code in the target 
```

> [!NOTE]
> All examples assume `import prose.codeaccelerator as cx`.

## Read object

Content of object.json:

```json
{
  "name": {
    "first": "Carrie",
    "last": "Dodson"
  },
  "phone": "123-456-7890"
}
```

The following example uses `ReadJsonBuilder` to generate code to read object.json.

```python
>>> b = cx.ReadJsonBuilder('object.json')
>>> r = b.learn()
>>> r.data()
  name.first name.last         phone
0     Carrie    Dodson  123-456-7890

>>> r.code()
import json
from pandas.io.json import json_normalize


def read_json(file):
    with open(file, encoding="utf-8") as f:
        d = json.load(f)
        df = json_normalize(d)
        return df

>>> b.target = 'pyspark'
>>> r = b.learn()
>>> r.code()
from pyspark.sql import SparkSession
from pyspark.sql.functions import col


def read_json(file):
    spark = SparkSession.builder.getOrCreate()
    df = spark.read.json(file, multiLine=True)
    df = df.select(col("name.first").alias("name.first"),
                   col("name.last").alias("name.last"),
                   col("phone").alias("phone"))
    return df
```

## Read array

Content of array.json:

```json
[
    {
        "name": {
            "first": "Carrie",
            "last": "Dodson"
        },
        "phone": "123-456-7890"
    },
    {
        "name": {
            "first": "Leonard",
            "last": "Robledo"
        },
        "phone": "789-012-3456"
    }
]
```

The following examples uses `ReadJsonBuilder` to generate code to read array.json.

```python
>>> b = cx.ReadJsonBuilder('array.json')
>>> r = b.learn()
>>> r.data()
  name.first name.last         phone
0     Carrie    Dodson  123-456-7890
1    Leonard   Robledo  789-012-3456

>>> r.code()
import json
from pandas.io.json import json_normalize


def read_json(file):
    with open(file, encoding="utf-8") as f:
        d = json.load(f)
        df = json_normalize(d)
        return df

>>> b.target = 'pyspark'
>>> r = b.learn()
>>> r.code()
from pyspark.sql import SparkSession
from pyspark.sql.functions import col


def read_json(file):
    spark = SparkSession.builder.getOrCreate()
    df = spark.read.json(file, multiLine=True)
    df = df.select(col("name.first").alias("name.first"),
                   col("name.last").alias("name.last"),
                   col("phone").alias("phone"))
    return df
```

## Read line-delimited

Content of delimited.json:

```json
{"name":{"first":"Carrie","last":"Dodson"},"phone":"123-456-7890"}
{"name":{"first":"Leonard","last":"Robledo"},"phone":"789-012-3456"}
```

The following example uses `ReadJsonBuilder` to generate code to read delimited.json.

```python
>>> b = cx.ReadJsonBuilder('delimited.json')
>>> r = b.learn()
>>> r.data()
  name.first name.last         phone
0     Carrie    Dodson  123-456-7890
0    Leonard   Robledo  789-012-3456

>>> r.code()
import json
from pandas.io.json import json_normalize


def read_json(file):
    with open(file, encoding="utf-8") as f:
        d = [json.loads(line) for line in f]
        return json_normalize(d)

>>> b.target = 'pyspark'
>>> r = b.learn()
>>> r.code()
from pyspark.sql import SparkSession
from pyspark.sql.functions import col


def read_json(file):
    spark = SparkSession.builder.getOrCreate()
    df = spark.read.json(file)
    df = df.select(col("name.first").alias("name.first"),
                   col("name.last").alias("name.last"),
                   col("phone").alias("phone"))
    return df
```

## Read split- array

Content of split.json:

```json
[
    {
        "color": "black",
        "category": "hue",
        "type": "primary",
        "code": {
            "rgba": [255, 255, 255, 1],
            "hex": "#000"
        }
    },
    {
        "color": "white",
        "category": "value",
        "code": {
            "rgba": [0, 0, 0, 1],
            "hex": "#FFF"
        }
    }
]
```

The following example uses `ReadJsonBuilder` to generate code to read split.json.

```python
>>> b = cx.ReadJsonBuilder('split.json')
>>> r = b.learn()
>>> r.data()
  category code.hex  color     type  code.rgba.0  code.rgba.1  code.rgba.2  code.rgba.3
0      hue     #000  black  primary          255          255          255            1
1    value     #FFF  white      NaN            0            0            0            1

>>> r.code()
import json
from pandas.io.json import json_normalize
import pandas as pd


def read_json(file):
    with open(file, encoding="utf-8") as f:
        d = json.load(f)
        df = json_normalize(d)
        df = (
            df.drop("code.rgba", 1)
              .join(df["code.rgba"]
                    .apply(lambda t: pd.Series(t))
                    .add_prefix("code.rgba."))
        )
        return df

>>> b.target = 'pyspark'
>>> r = b.learn()
>>> r.code()
from pyspark.sql import SparkSession
from pyspark.sql.functions import col


def read_json(file):
    spark = SparkSession.builder.getOrCreate()
    df = spark.read.json(file, multiLine=True)
    df = df.select(col("color").alias("color"),
                   col("category").alias("category"),
                   col("type").alias("type"),
                   col("code.rgba")[0].alias("code.rgba.0"),
                   col("code.rgba")[1].alias("code.rgba.1"),
                   col("code.rgba")[2].alias("code.rgba.2"),
                   col("code.rgba")[3].alias("code.rgba.3"),
                   col("code.hex").alias("code.hex"))
    return df
```

## Read nested arrays

Content of nested.json:

```json
[
    {
        "name": {
            "first": "Carrie",
            "last": "Dodson"
        },
        "phone": [
            {
                "area": "123",
                "number": "456-7890"
            },
            {
                "area": "098",
                "number": "765-4321"
            }
        ]
    },
    {
        "name": {
            "first": "Leonard",
            "last": "Robledo"
        },
        "phone": [
            {
                "area": "789",
                "number": "012-3456"
            },
            {
                "area": "654",
                "number": "321-0987"
            }
        ]
    }
]
```

The following example uses `ReadJsonBuilder` to generate code to read nested.json.

```python
>>> b = cx.ReadJsonBuilder('nested.json')
>>> r = b.learn()
>>> r.data()
  phone.area phone.number name.first name.last
0        123     456-7890     Carrie    Dodson
1        098     765-4321     Carrie    Dodson
2        789     012-3456    Leonard   Robledo
3        654     321-0987    Leonard   Robledo

>>> r.code()
import json
from pandas.io.json import json_normalize


def read_json(file):
    with open(file, encoding="utf-8") as f:
        d = json.load(f)
        df = json_normalize(d,
            record_path="phone",
            meta=[
                ["name", "first"],
                ["name", "last"]],
            record_prefix="phone.")
        return df

>>> b.target = 'pyspark'
>>> r = b.learn()
>>> r.code()
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, explode


def read_json(file):
    spark = SparkSession.builder.getOrCreate()
    df = spark.read.json(file, multiLine=True)
    df = df.select(col("name.first").alias("name.first"),
                   col("name.last").alias("name.last"),
                   explode("phone").alias("phone_explode"),
                   col("phone_explode.area").alias("phone.area"),
                   col("phone_explode.number").alias("phone.number"))
    df = df.drop("phone_explode")
    return df

```

## Read nested arrays (2)

The array elements in `"phone"` are nested objects, which requires an additional call to `json_normalize` to flatten.

Content of nested2.json:

```json
[
    {
        "name": {
            "first": "Carrie",
            "last": "Dodson"
        },
        "phone": [
            {
                "area": "123",
                "number": {
                    "number1": "456",
                    "number2": "7890"
                }
            },
            {
                "area": "098",
                "number": {
                    "number1": "765",
                    "number2": "4321"
                }
            }
        ]
    },
    {
        "name": {
            "first": "Leonard",
            "last": "Robledo"
        },
        "phone": [
            {
                "area": "789",
                "number": {
                    "number1": "0123",
                    "number2": "456"
                }
            }
        ]
    }
]
```

The following example uses `ReadJsonBuilder` to generate code to read nested2.json.

```python
>>> b = cx.ReadJsonBuilder('nested2.json')
>>> r = b.learn()
>>> r.data()
  name.first name.last phone.area phone.number.number1 phone.number.number2
0     Carrie    Dodson        123                  456                 7890
1     Carrie    Dodson        098                  765                 4321
2    Leonard   Robledo        789                 0123                  456

>>> r.code()
import json
from pandas.io.json import json_normalize


def read_json(file):
    with open(file, encoding="utf-8") as f:
        d = json.load(f)
        df = json_normalize(d,
            record_path="phone",
            meta=[
                ["name", "first"],
                ["name", "last"]],
            record_prefix="phone.")
        # flatten objects in "phone"
        df = json_normalize(df.to_dict('r'))
        return df

>>> b.target = 'pyspark'
>>> r = b.learn()
>>> r.code()
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, explode


def read_json(file):
    spark = SparkSession.builder.getOrCreate()
    df = spark.read.json(file, multiLine=True)
    df = df.select(col("name.first").alias("name.first"),
                   col("name.last").alias("name.last"),
                   explode("phone").alias("phone_explode"),
                   col("phone_explode.area").alias("phone.area"),
                   col("phone_explode.number.number1").alias("phone.number.number1"),
                   col("phone_explode.number.number2").alias("phone.number.number2"))
    df = df.drop("phone_explode")
    return df
```

## Read multiple arrays

If there are multiple top-level arrays, preserve these arrays because joining them would cause exponential blow-up.

Content of multiple.json:

```json
{
    "name": "Carrie Dodson",
    "addresses": [
        "1640 Riverside Drive, Hill Valley, California",
        "1630 Revello Drive, Sunnydale, CA"
    ],
    "phone": [
        "202-555-0180",
        "202-555-0103"
    ]
}
```

The following example uses `ReadJsonBuilder` to generate code to read multiple.json.

```python
>>> b = cx.ReadJsonBuilder('multiple.json')
>>> r = b.learn()
>>> r.data()
               name             addresses               phone
0     Carrie Dodson ["1640...","1630..."] ["202...","202..."]

>>> r.code()
import json
from pandas.io.json import json_normalize


def read_json(file):
    with open(file, encoding="utf-8") as f:
        d = json.load(f)
        df = json_normalize(d)
        return df

>>> b.target = 'pyspark'
>>> r = b.learn()
>>> r.code()
from pyspark.sql import SparkSession
from pyspark.sql.functions import col


def read_json(file):
    spark = SparkSession.builder.getOrCreate()
    df = spark.read.json(file, multiLine=True)
    df = df.select(col("name").alias("name"),
                   col("addresses").alias("addresses"),
                   col("phone").alias("phone"))
    return df
```

## Read single top array

If the JSON has this pattern `{"key_1": ... {"key_n" : [ ... ]} ... }`, you only need to flatten the top array `d["key_1"]...["key_n"]`.

Content of top.json file:

```json
{
    "top": [
        {
            "name": {
                "first": "Carrie",
                "last": "Dodson"
            },
            "phone": "123-456-7890"
        },
        {
            "name": {
                "first": "Leonard",
                "last": "Robledo"
            },
            "phone": "789-012-3456"
        }
    ]
}
```

The following example uses `ReadJsonBuilder` to generate code to read top.json.

```python
>>> b = cx.ReadJsonBuilder('top.json')
>>> r = b.learn()
>>> r.data()
  name.first name.last         phone
0     Carrie    Dodson  123-456-7890
1    Leonard   Robledo  789-012-3456

>>> r.code()
import json
from pandas.io.json import json_normalize


def read_json(file):
    with open(file, encoding="utf-8") as f:
        d = json.load(f)
        df = json_normalize(d["top"])
        return df.add_prefix("top.")

>>> b.target = 'pyspark'
>>> r = b.learn()
>>> r.code()
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, explode


def read_json(file):
    spark = SparkSession.builder.getOrCreate()
    df = spark.read.json(file, multiLine=True)
    df = df.select(explode("top").alias("top_explode"),
                   col("top_explode.name.first").alias("top.name.first"),
                   col("top_explode.name.last").alias("top.name.last"),
                   col("top_explode.phone").alias("top.phone"))
    df = df.drop("top_explode")
    return df
```
