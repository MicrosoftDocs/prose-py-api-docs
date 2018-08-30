---
title: Read a Fixed-Width File
ms.date: 08/30/2018
ms.topic: conceptual
ms.service: prose-codeaccelerator
---

# Read a Fixed-Width File

`ReadFwfBuilder` will analyze a fixed-width file and produce code to split the fields yielding a dataframe.  When it is
given only the fixed-width input file, Code Accelerator makes every effort to determine the boundaries between fields.
Sometimes, however, this simply isn't possible.  If a file has two separate number fields placed directly next to one
another, for example, there just may not be enough information to determine the boundary between them.  So,
`ReadFwfBuilder` optionally also takes a text string containing a description of the file schema.  This schema
description does not have to be in an exact format--PROSE will do its best to locate lists of fields and their column
ranges and use that information to generate the code.

Example Usage:

``` python
import pathlib
import prose.codeaccelerator as cx

schema = pathlib.Path(path_to_schema_file).read_text()
builder = cx.ReadFwfBuilder(path_to_file, schema)
# optional: builder.Target = cx.Target.pyspark
result = builder.learn()
result.data(5)
# examine top 5 rows to see if they look correct
result.code()
```
