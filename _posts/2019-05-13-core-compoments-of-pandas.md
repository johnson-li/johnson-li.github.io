---
title: "Core Compoments of Pandas"
date: 2019-05-13
categories:
  - Pandas
---

This doc aims to explain how Pandas program is structured and how it persists data in memory. 
Briefly, Pandas relies on numpy to store data column by column. 
More details about the data structures and the operations are given below.

## DataFrame
DataFrame is a two-dimensional size-mutable, potentially heterogeneous tabular data structure with labeled axes (rows and columns).
It is the core class in Pandas to represent table-formatted data and is the most commonly used Pandas class. 
It can be created in various ways, E.g., from array, from dictionary, from file and so on.

Internally, DataFrame uses a number of other data structures to record the information of the table.
- data, an instance of [BlockManager](), it memories all of the data stored in a DataFrame.
- columns, an instance of [Index](), it memories all information about the columns.
- loc, an instance of [_LocIndexer](), it provides row-based access to the DataFrame.

Since DataFrame stores data by column, it only supports column-based readings by itself.
```
>>> df = pd.DataFrame([{'a': 1, 'b': 2}, {'a': 5, 'b': 10, 'c': 20}, {'a': 7, 'b': 8, 'c': 9}])
>>> df
   a   b     c
0  1   2   NaN
1  5  10  20.0
2  7   8   9.0
>>> df['a']
0    1
1    5
2    7
Name: a, dtype: int64
```

The return value of of `df['a']` is an instance of [Series](), which literally represents a column and will be described later.
If we want to access to DataFrame by row, we need to use `DataFrame.loc` or `DataFrame.iloc`. 
The difference between `loc` and `iloc` can be found [here]().
We use `loc` as example here.

```
>>> df.loc[:2]  # memory reused
   a   b     c
0  1   2   NaN
1  5  10  20.0
>>> df.loc[[0, 1]]  # memory copy
   a   b     c
0  1   2   NaN
1  5  10  20.0
>>> df.loc[[True, True, False]]  # memory copy
   a   b     c
0  1   2   NaN
1  5  10  20.0
```
All of the three operations listed above retrieve the first 2 rows from a DataFrame. But the internal implementation is a little different.
For the first one, no memory copy is required. It simply creates slices of each columns from the original DataFrame to form the new DataFrame.
For the seconds one, memory copy is required. All of the rows selected by the indexes are copied out of the original DataFrame. A new DataFrame is generated to wrap the copied rows.
For the third one, the boolean array is translated into the `indexes` just as the seconds example. So memory copy is required, too.

## Series
Series is a one-dimensional labeled array capable of holding any data type (integers, strings, floating point numbers, Python objects, etc.)
We can understand it as a one-dimensional version of DataFrame.


## BlockManager
BlockManager is the core internal data structure to store data for DataFrame and Series. 
It manages a bunch of labeled 2D arrays, which are implemented by numpy.
A BlockManager contains a serious of blocks, namely FloatBlock, IntBlock, BoolBlock, etc. 
The blocks are stored in a list parameter `BlockManager.blocks`. 
Each block is a 2D numpy array which stores the data column by column.
```
>>> df._data.blocks
(FloatBlock: slice(2, 3, 1), 1 x 3, dtype: float64,
 IntBlock: slice(0, 2, 1), 2 x 3, dtype: int64)
>>> df._data.blocks[1].values  # the column a and the column b
array([[1,   5,   7],
       [  2,  10,   8]])
```

One important API provided by BlockManager is `BlockManager.take`. 
It takes a list of indexes as input and generates the corresponding slice of the blocks as output.
It always makes a copy of the blocks because it calls `reindex_indexer` with the parameter `copy` as True.
It will be interesting to check what if the `copy` is set to False.

