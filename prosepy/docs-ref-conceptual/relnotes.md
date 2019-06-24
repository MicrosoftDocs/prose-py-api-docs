---
title: Release Notes for the Microsoft PROSE Code Accelerator SDK - Python
ms.date: 2/15/2019
ms.topic: conceptual
ms.service: non-product-specific
author: simmdan
ms.author: dsimmons
description: Summary of changes for each release of PROSE Code Accelerator for Python.
---

# Release Notes

## 1.7.0 - 2019-06-24

- Fixed bug where the DetectTypesBuilder would reorder columns.
- More sophisticated detection of number of skip lines for ReadCsvBuilder.

## 1.6.0 - 2019-05-31

- Miscellaneous bug fixes.

## 1.5.0 - 2019-04-11

- Miscellaneous bug fixes and performance improvements.

## 1.4.0 - 2019-02-15

- Miscellaneous bug fixes and performance improvements.

## 1.3.0 - 2018-12-17

- Miscellaneous bug fixes and performance improvements.

### DetectTypesBuilder

- Added alternate, more efficient entry point for completing the task of performing datatype detection and conversion on
  a data set rather than generating code for doing so.

### ReadCsvBuilder

- Added a new experimental feature for filtering "junk" rows (ie. repeated headers and non-data, non-comment rows).  To
  enable set the property `enable_regex_row_filtering` to `True`.

## 1.2.1 - 2018-11-20

- Changed dependency on the dotnetcore2 package back to version 2.1.0 rather than 2.1.5 in order to better inter-operate
  with the azureml-dataprep python package.

## 1.2.0 - 2018-11-16

### ReadJsonBuilder

- Code generation improved to prevent unnecessary column aliases: `col("color"`) instead of
  `col("color").alias("color")`.

### DetectTypesBuilder

- Generated code is now easier to read and understand.
- Now supports non-separated dates (`20120513`), numbers in parenthesis (`(45,345)`), and other improvements to parsing
  ambiguous date formats.
- Better handling of integer columns that look like categorical data, and handling of `NaN` values in string columns.

## 1.1.0 - 2018-10-24

- New configuration mechanism for specifying Code Accelerator options.  See [Configuration](config.md) for more details.
- Renamed `.data` to `.preview_data` on all builders.  This property now returns a fixed number of rows (5).
- The `prose.codeaccelerator.jupyter` namespace has been removed and its functionality is now included in the main
  `prose.codeaccelerator` namespace.  When Code Accelerator is used in an environment that supports improved output
  rendering, that rendering is displayed automatically.
- Now supports JupyterLab as well as Jupyter notebooks.
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
- Produced regex names have been simplified to make them easier to read.
- Improved performance when finding patterns in long strings.

## 1.0.0 - 2018-09-20

First release.  Includes ReadCsvBuilder, ReadFwfBuilder, ReadJsonBuilder, DetectTypesBuilder and FindPatternsBuilder.