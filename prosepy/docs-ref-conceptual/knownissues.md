---
title: Known Issues
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: prose-codeaccelerator
---

# Known Issues
The following are issues that the PROSE team is aware of with the 1.0.0 release of prose-codeaccelerator.  We are
working to fix them in a future release.  If you encounter issues not on this list, please report them on the
[prose-codeaccelerator Github repository](https://github.com/Microsoft/prose-codeaccelerator/issues).

## ReadCsvBuilder
- **_TODO:_** File encoding: UTF16/32 aren't supported.  Some other formats have issues depending on back end.
  
## ReadFwfBuilder
- **_TODO:_** Fixed-width files with \r line endings?
- **_TODO:_** File encoding: UTF16/32 aren't supported.  Some other formats have issues depending on back end.
  
## ReadJsonBuilder

### Known Issues for Pandas target
- If the json has a single top level array of values (e.g., `{"top": [1, 2]}`). pandas' `json_normalize` throws
  `TypeError: must be str, not int`. Github issue: https://github.com/pandas-dev/pandas/issues/21536. This has been
  fixed since pandas 0.23.2, but Azure Data Studio uses pandas 0.22.0
- If the json has a single object of object of array. For instance,
  ```
  {
      "name": "alan smith",
      "info": {
          "phones": [{
              "area": 111,
              "number": 2222
          }, {
              "area": 333,
              "number": 4444
          }]
      }
  }
  ```
  pandas' `json_normalize` throws `TypeError: string indices must be integers`. Github issue:
  https://github.com/pandas-dev/pandas/issues/22706

### Known Issues for Pyspark target
- If the json is an array of values (e.g., `[1, 2, 3]`). pyspark throws `AnalysisException: 'Since Spark 2.3, the
  queries from raw JSON/CSV files are disallowed when the referenced columns only include the internal corrupt record
  column`. Note that array of objects is not affected.

## DetectTypesBuilder
- **_TODO:_** Stackoverflow can occur if any code passed to pd.transform() throws a `ValueError`.
- **_TODO:_** When a column contains only a time results in a datetime object where the date is 2000-01-01.
- **_TODO:_** Cannot process a dataframe where the column names are not strings.  This happens if a dataframep is
  created from a list.
- **_TODO:_** Column with only the value `24.0` is treated as a time rather than a number.