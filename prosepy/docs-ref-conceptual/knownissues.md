---
title: Known issues with the Microsoft PROSE Code Accelerator SDK - Python
ms.date: 2/15/2019
ms.topic: conceptual
ms.service: non-product-specific
author: simmdan
ms.author: dsimmons
description: These are known issues with the current release of PROSE Code Accelerator for Python.
---

# Known issues with the Microsoft PROSE Code Accelerator SDK
The following are issues that the PROSE team is aware of with the 1.5.0 release of the Microsoft PROSE Code Accelerator
SDK.  We are working to fix them in a future release.  If you encounter issues not on this list, please report them on
the [prose-codeaccelerator GitHub repository](https://github.com/Microsoft/prose-codeaccelerator/issues).
  
## ReadJsonBuilder

### Pandas:
- If the JSON has a single top level array of values (e.g., `{"top": [1, 2]}`), pandas' `json_normalize` throws
  `TypeError: must be str, not int`. GitHub issue: https://github.com/pandas-dev/pandas/issues/21536. This has been
  fixed since pandas 0.23.2, but Azure Data Studio uses pandas 0.22.0

- If the JSON has a single object of object of array, for instance:

  ```json
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
  pandas' `json_normalize` throws `TypeError: string indices must be integers`. GitHub issue:
  https://github.com/pandas-dev/pandas/issues/22706

### PySpark:
- If the JSON is an array of values (e.g., `[1, 2, 3]`), `spark.read.json` throws `AnalysisException: 'Since Spark 2.3, the
  queries from raw JSON/CSV files are disallowed when the referenced columns only include the internal corrupt record
  column`. Note that array of objects is not affected.

- If the JSON has a backtick (`` ` ``) in a property name (e.g., ``{ "property has backtick `": 1 }``),
`spark.read.json` throws `org.apache.spark.sql.AnalysisException: syntax error in attribute name`. 
Spark issue: [https://issues.apache.org/jira/browse/SPARK-18502](https://issues.apache.org/jira/browse/SPARK-18502).

## DetectTypesBuilder
- A numeric column formatted like `##.#`, where `#` represents a digit, is detected as a time if all the values are
  between `0.0` and `24.0` with no fractional part exceeding `.59`. So, if numeric values all look like
  valid time values in the `HH.M` (2 digit hour, 1 or 2 digit minute values) format, the builder might incorrectly
  identified their type as `time`, rather than as numeric.

### Pandas:
- Pandas DataFrame objects with non string-valued column identifiers are not supported. Such a DataFrame object commonly
  arises when a `pandas.DataFrame` is created from a list of values. Suggested workaround: rename the columns of the
  DataFrame by calling `str()` on the non string-valued column identifiers.

- When the input is a pandas DataFrame, and the `target` property on the `DetectTypesBuilder` instance is set to
  `pandas` or `auto`, if a column containing only integer values contains one more more `NA` values, then although the
  generated code promises to return an `int`, the pandas DataFrame object returned will have the values in that column
  coerced/promoted to `numpy.float64`. This is a "feature" of pandas and is documented
  [here](https://pandas.pydata.org/pandas-docs/stable/gotchas.html#nan-integer-na-values-and-na-type-promotions).

### PySpark:
- A column that contains only times (rather than datetimes) will result in the date values being set to `2000-01-01` in
  PySpark mode. This is a workaround to the fact that PySpark DataFrames do not support dates earlier than `1970-01-01`.
  Further, a bug in Python 3.6 causes dates earlier than `1970-01-02` to be handled incorrectly. To ensure reliable
  operation across platforms, time values without dates are represented with dates of `2000-01-01` in PySpark mode. The
  default date value for other targets remains `1900-01-01`, which is the default for Python.
