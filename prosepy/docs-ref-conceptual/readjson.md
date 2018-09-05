---
title: Read a JSON File
ms.date: 08/30/2018
ms.topic: conceptual
ms.service: prose-codeaccelerator
---

# Read a JSON File

The `ReadJsonBuilder` will produce code to turn a json file into a flat table for use in a data_frame.

Example Usage:

``` python
import prose.codeaccelerator as cx

builder = cx.ReadCsvBuilder(path_to_json_file)
# optional: builder.Target = cx.Target.pyspark
result = builder.learn()
result.data(5)
# examine top 5 rows to see if they look correct
result.code()
```

ToDo: Explain flattening strategy.