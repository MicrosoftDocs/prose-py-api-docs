---
title: Release Notes for the Microsoft PROSE Code Accelerator SDK - Python
ms.date: 10/24/2018
ms.topic: conceptual
ms.service: non-product-specific
author: simmdan
ms.author: dsimmons
description: Summary of changes for each release of PROSE Code Accelerator for Python.
---

# Release Notes

## 1.1.0 - 2018-10-24

- New configuration mechanism for specifying Code Accelerator options.  See [Configuration](config.md) for more details.
- Renamed `.data` to `.preview_data` on all builders.  This property now returns a fixed number of rows (5).
- The `prose.codeaccelerator.jupyter` namespace has been removed and its functionality is now included in the main
  `prose.codeaccelerator` namespace.  When Code Accelerator is used in an environment that supports improved output
  rendering, that rendering is displayed automatically.
- Now supports Jupyterlab as well as Jupyter notebooks.
- Many bug fixes.

### ReadCsvBuilder

- Now supports single column files.
- Improved support for detection of file encoding.

### ReadJsonBuilder

- New `ReadJsonBuilder.lines_to_analyze` property allows specifying how many lines from the input file to read when
  learning programs.
- Now supports reading large newline-delimited JSON files (NDJSON).
- Improved flattening of embedded arrays in new-line delimited JSON files.
- Delimiter used in generated column names has been changed from `.` to `_` when targeting pyspark.

### DetectTypesBuilder

- Generated code for pandas now uses `df.apply` on every column instead of `df.transform`.
- Fixed some cases where a stack-overflow could result from attempting to coerce the types in a data frame.

### FindPatternsBuilder

- New `FindPatternsBuilder.include_outlier_patterns` property which optionally directs the builder to include additional
  (possibly low quality) patterns in the returned result.
- Generated code now includes examples and stats in comments.
- Regex names produced have been simplified to make them easier to read.
- Improved performance when finding patterns in long strings.

## 1.0.0 - 2018-09-20

First release.  Includes ReadCsvBuilder, ReadFwfBuilder, ReadJsonBuilder, DetectTypesBuilder and FindPatternsBuilder.