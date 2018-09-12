---
title: Read a CSV file
ms.date: 08/30/2018
ms.topic: conceptual
ms.service: prose-codeaccelerator
---

# Read a CSV File

The `ReadCsvBuilder` will analyze a given delimited text file (it doesn't actually need to be comma-separated file, it
could be delimited other ways) and determine all the details about that file necessary to successfully parse it and
produce a dataframe (either `pandas` or `pyspark`).  This includes the encoding, the delimiter, how many lines to skip at
the beginning of the file, etc.

Example Usage:

``` python
import prose.codeaccelerator as cx

builder = cx.ReadCsvBuilder(path_to_file)
# optional: builder.Target = cx.Target.pyspark
result = builder.learn()
result.data(5)
# examine top 5 rows to see if they look correct
result.code()
```
