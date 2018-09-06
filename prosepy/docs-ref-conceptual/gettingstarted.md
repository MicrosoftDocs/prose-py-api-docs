---
title: Getting Started
ms.date: 08/30/2018
ms.topic: get-started-article
ms.service: prose-codeaccelerator
---

# Getting Started
```
pip install prose-codeaccelerator --extra-index-url https://prose-python-packages.azurewebsites.net
```

Then from within a python interactive prompt (`python`, `ipython`, `jupyter notebook`, etc.):

``` python
import prose.codeaccelerator as cx 
# or, if running in a jupyter notebook: 
# import.prose.codeaccelearator.jupyter as cx 

builder = cx.ReadCsvBuilder("/path/to/file.csv")
result = builder.learn()
result.code()
```

To generate code for pyspark, set `builder.target = "pyspark"` before calling `builder.learn()`.
