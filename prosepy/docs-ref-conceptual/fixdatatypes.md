---
title: Fix data types
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: prose-codeaccelerator
---

# Fix data types

One common pain point when working with data in python is that often values
in columns in a dataframe are stored as strings whereas they should be numbers or dates. This prevents doing logical operations
on those columns.  `DetectTypesBuilder` will examine data and, if appropriate,
produce code to transform the data to correct types.  While the underlying pandas and
pyspark libraries in some cases have the ability to infer data types from
strings, often the results are less than ideal: the set of supported
formats is often small. Further, these inference techniques usually fail
completely if a column of data consists of values formatted in more than
one way.

The data type detection features in the PROSE Code Accelerator come in handy in
such scenarios. Not only does the data type detection convert data into the
appropriate data type, the process is completely transparent. After generating code for the data type transformation/conversion, 
user can then inspect or modify the code as desired: the system is no
longer a magical black-box.

## Usage

``` python
import pandas
import prose.codeaccelerator as cx

# get a dataframe using pandas.read_csv
df = pandas.read_csv('data.csv')
print('Preview:')
print(df.head(10))
print('\nData types:')
print('Date:   %s' % type(transformed_df['Date'][0]))
print('Value:  %s' % type(transformed_df['Value'][0]))
```
which produces the following output:

```
Preview:
         Date              Value
0  2102-06-23       4.997562e+07
1  1972_09_19     5034052.989972
2  2132_04_15  -3,374,271.588363
3  2127-01-11      -1.658914e+07
4  1985_04_15  69,212,245.496658
5  1857-06-25       7.675148e+07
6  1847-11-13    76281853.858563
7  1926-01-22   -63152823.016701
8  1942_06_09   -47132086.386679
9  1906_09_05   -46287953.408743

Data types:
Date:   <class 'str'>
Value:  <class 'str'>
```

Observe that the dates are in differing formats. Further, also notice that
the `Value` column contains numbers formatted in three different ways: one
with commas as the thousands separator, one without, and the last format
uses the scientific notation. With so much heterogeneity in the data,
pandas is unable to automatically convert to the right data types.

To convert the columns into the right data types, we can simply use the data type detection APIs in the PROSE Python SDK:

```python
builder = cx.DetectTypesBuilder(df)
result = builder.learn()
transformed_df = result.data()
# examine new types
print('Preview:')
print(transformed_df.head())
print('Data types:')
print('Date:   %s' % type(transformed_df['Date'][0]))
print('Value:  %s' % type(transformed_df['Value'][0]))
```
which produces the following output:
```
Preview:
         Date         Value
0  2102-06-23  4.997562e+07
1  1972-09-19  5.034053e+06
2  2132-04-15 -3.374272e+06
3  2127-01-11 -1.658914e+07
4  1985-04-15  6.921225e+07
5  1857-06-25  7.675148e+07
6  1847-11-13  7.628185e+07
7  1926-01-22 -6.315282e+07
8  1942-06-09 -4.713209e+07
9  1906-09-05 -4.628795e+07
Data types:
Date:   <class 'datetime.date'>
Value:  <class 'numpy.float64'>
```

Further, we can generate the code for the learned data type transformation:

```
# display the code to transform the data types
print(result.code())
```
which produces the following output:

```python
def _parse_value_from_Date(value):
    import datetime
    # We try to parse using the following formats. If parsing using any format fails,
    # we simply ignore the failure and try the next format.
    # Parse values formatted like: "2102-06-23", "2127-01-11", "1857-06-25" ...
    try:
        return datetime.datetime.strptime(value, '%Y-%m-%d').date()
    except ValueError:
        pass
    # We try to parse using the following formats. If parsing using any format fails,
    # we simply ignore the failure and try the next format.
    # Parse values formatted like: "1972_09_19", "2132_04_15", "1985_04_15" ...
    try:
        return datetime.datetime.strptime(value, '%Y_%m_%d').date()
    except ValueError:
        pass
    # We didn't encounter a value formatted like this when the data type detection was performed.
    raise Exception('Unhandled case in type conversion for column %s: \'%s\'' % ('Date', value))

def _parse_value_from_Value(value):
    import regex
    if regex.match(r'^[+-]?\d*(?:\.\d+)?[eE][+-]?\d+$', value):
        # Parse values formatted like: "4.997562e+07", "-1.658914e+07", "7.675148e+07"...
        return float(value)

    if regex.match(r'^\d+(?:,\d+)*\.\d+$', value):
        # Parse values formatted like: "69,212,245.496658", "76281853.858563"...
        value = value.replace(',', '')
        value = value.replace('.', '.')
        return float(value)

    if regex.match(r'^-\d+(?:,\d+)*\.\d+$', value):
        # Parse values formatted like: "-3,374,271.588363", "-63152823.016701", "-47132086.386679"...
        value = value.replace('-', '')
        value = value.replace(',', '')
        return -float(value)

    # We didn't encounter a value formatted like this when the data type detection was performed.
    raise Exception('Unhandled case in type conversion for column %s: \'%s\'' % ('Value', value))

def coerce_types(df):
    return df.transform({
        'Date': _parse_value_from_Date, # Date
        'Value': _parse_value_from_Value, # Float
    })
```
Observe that the generated code handles the various cases in the formatting
of the data. Further, the generated code contains descriptive comments
which make it easy for a user to modify the generated code to handle
new and/or unseen formats in the data.

## Inputs and Targets
The data type detection APIs accept the following three forms of input: (1) A
simple list, (2) a dictionary with string-valued keys and lists of strings
as values, or (3) a pandas DataFrame with string-valued column identifiers.

A user may also specify the target for code generation. The data type
detection API can generate code that can transform data contained in
(1) a list, (2) a dictionary, (3) a pandas DataFrame, or (4) a pyspark
DataFrame. Note that using a pyspark DataFrame as input is not supported. A
user may manually sample data from the pyspark dataframe into a
dictionary, or a pandas DataFrame. This sampled data may then be used as
input to learn the data type transformation that can then be applied on the
pyspark DataFrame.

## Supported types and limitations
The data type detection APIs support detection of the following types:
- Dates
- Numbers (real and integer valued)
- Boolean values

Limitations:
- When a numeric column includes NA values, the default behavior of the
code generated by the data type detection APIs is to return `None` in
response to a value detected as an `NA` value. When used with a pandas
DataFrame, any integer values intermixed with `None` in a column results in
the entire column being represented using `numpy.float64` values. A user
may workaround this by editing the code and choosing an appropriate
non-`None` value to return when an `NA` value is converted.

- The data type detection APIs currently only process columns where the
  data is string-valued. Non string-valued columns are simply returned
  as-is, with no transformation applied.
