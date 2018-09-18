---
title: Known Issues
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: prose-codeaccelerator
---

# Known Issues
The following are issues that the PROSE team is aware of with the 1.0.0 release of prose-codeaccelerator.  We are
working to fix them in a future release.  If you encounter issues not on this list, please report them here: **_TODO_**

## ReadCsvBuilder
- **_TODO:_** File encoding: UTF16/32 aren't supported.  Some other formats have issues depending on back end.
  
## ReadFwfBuilder
- **_TODO:_** Fixed-width files with \r line endings?
- **_TODO:_** File encoding: UTF16/32 aren't supported.  Some other formats have issues depending on back end.
  
## ReadJsonBuilder
- **_TODO:_** Read JSON issues related to older version of pandas.

## DetectTypesBuilder
- **_TODO:_** Stackoverflow can occur if any code passed to pd.transform() throws a `ValueError`.
- **_TODO:_** When a column contains only a time results in a datetime object where the date is 2000-01-01.
- **_TODO:_** Cannot process a dataframe where the column names are not strings.  This happens if a dataframep is
  created from a list.
- **_TODO:_** Column with only the value `24.0` is treated as a time rather than a number.