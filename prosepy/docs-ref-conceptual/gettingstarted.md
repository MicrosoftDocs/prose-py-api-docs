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

If using Jupyter notebook, you must also enable the corresponding extension:

```
jupyter nbextension enable --py prose.codeaccelerator.jupyter
```

To verify, `jupyter nbextension list` should now list `prose/extension enabled` as one of the known notebook extensions.

Then from within a python interactive prompt (`python`, `ipython`, `jupyter notebook`, etc.):

``` python
import prose.codeaccelerator as cx 
# or, if running in a jupyter notebook: 
# import.prose.codeaccelerator.jupyter as cx 

builder = cx.ReadCsvBuilder("/path/to/file.csv")
result = builder.learn()
result.code()
```

To generate code for pyspark, set `builder.target = "pyspark"` before calling `builder.learn()`.
