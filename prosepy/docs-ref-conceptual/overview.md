---
title: What is the Microsoft PROSE Code Accelerator SDK for Python?
ms.date: 09/24/2018
ms.topic: conceptual
ms.service: non-product-specific
author: simmdan
ms.author: dsimmons
description: Learn about requirements and functionality of the Microsoft PROSE Code Accelerator SDK for Python.
---

# What is the Microsoft PROSE Code Accelerator SDK?

The Microsoft PROSE Code Accelerator SDK uses program synthesis to generate Python code for common data preparation tasks. Code Accelerator includes [PROSE](https://microsoft.github.io/prose) functionality for
data ingestion, data type correction, and pattern identification in string data. 


## Requirements
Code Accelerator runs on Python 3.5, 3.6, or 3.7 on Windows (32-bit or 64-bit), Linux (64-bit), and macOS.  It supports
generating code for use with [pandas](https://pandas.pydata.org/) or [PySpark](https://pypi.org/project/pyspark/).


## Using Code Accelerator
The first thing to know about Code Accelerator is that it is a tool masquerading as an SDK. You call Code Accelerator
from a Python interactive environment and it produces code for you. You can examine the results, adjust your call to
Code Accelerator, and repeat as needed. But when you are done, you own the resulting code which has no dependencies
on Code Accelerator. You can modify or extend the code, copy it into another system, or work with it however you need.

Code Accelerator contains a series of classes which all follow the same pattern:

Interactions with Code Accelerator start with a builder, which is the object that will build code for you.  There are
builders for reading delimited files (`ReadCsvBuilder`), reading fixed width files (`ReadFwfBuilder`), reading JSON
files (`ReadJsonBuilder`), detecting data types (`DetectTypeBuilder`) and finding patterns in strings
(`FindPatternsBuilder`).  In each case the interaction is the same:

```
        init           learn               code
[task] -----> Builder ------> LearnResult -----> printable lambda
               ^   |                 \   \
               |   |                  \   \----> output for task
               \---/                   \   data
               clues                    \
                                         \-----> metadata
```

1. Create the builder supplying any absolutely required parameters.
2. Optionally provide clues to help the builder.
3. Learn.
4. Review the resulting data and code.
5. Return to step 2 and repeat as necessary until the results are correct.
6. Take the code and use it independently from Code Accelerator--Code Accelerator itself is just a tool, it is not
   required at runtime.


## About the code method on learn result
When `learn()` is called on any Code Accelerator builder, PROSE synthesizes Python code for the specified task.  This
code may be retrieved from the `code()` method on the result object.  The code takes the form of one or more functions
which may later be called with the same or similar data to what was initially provided to the builder to accomplish the
task.  The `ReadCsvBuilder`, for instance, will produce a `read_file` function that takes a path to a file of the same
format as the file passed to the builder, while the `DetectTypesBuilder` will produce a `coerce_types` function that
takes a dataframe of the same form as what was originally passed to the builder.

> [!NOTE]
> Calling the generated function with data having different schema may result in errors. For example, calling
> the generated `coerce_types` function with a dataframe that has a different schema than what was used to generate the
> function will likely fail or produce unwanted results.
 
 
## About the data method on learn result
The `data()` method is intended to give a quick look at what the generated code will do.  For the input originally given to the
builder, it will show what the output of running the generated code would be.  This way you can see if the code does
what you want, and if not, provide different or additional information to the builder and try learning again.  

Since the provided data may be much larger than what you need for a quick check, the method takes an optional parameter
of the number of output values to return.  For some learn results this is a number of a rows, for others it is just a
number of scalar values.


## Working with PySpark
Each builder supports the `Target` property which specifies the runtime environment for the generated code.  By default
the generated code will use pandas, but if you set the `Target` property to `"pyspark"`, then it will produce code
for that runtime instead.

Some things to keep in mind about PySpark:

- In order to target PySpark, you must first `pip install pyspark`.  Code Accelerator will dynamically use it if it is
  installed and you request to target it, but Code Accelerator will fail if you target it, and it isn't already
  installed.
- The generated code for PySpark does not `collect`.  Code Accelerator generates code that can fit into the beginning or
  middle of a spark pipeline.  This way you can mix Code Accelerator generated code with other code.  Once you are done
  processing your data, you will need to manually call `collect` or whatever other operator to produce final results.
- The `data()` method, however, ensures that you get some data back to examine so you can determine if the generated
  code is correct or not.  This may run a spark job and collect the results, or it may produce the data another way.  In
  any case the data is guaranteed to be the same data that the generated code will produce.
