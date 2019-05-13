---
title: "Merge Algorithm in Pandas"
date: 2019-05-13
categories:
  - Pandas
  - Algorithm
---

The `merge` operation joins two columns just like what database does. 
Pandas supports `left joint`, `right join`, `inner join`, and `outer join`.
A simple example is shown below.
```
>>> f1 = pandas.DataFrame({'id1': [1, 2, 3, 4, 4], 'val1': [10, 20, 30, 40, 50]})
   id1  val1
0    1    10
1    2    20
2    3    30
3    4    40
4    4    50
>>> f2 = pandas.DataFrame({'id2': [2, 3, 7, 1, 2], 'val2': [200, 300, 400, 500, 600]})
   id2  val2
0    2  200
1    3  300
2    7  400
3    1  500
4    2  600
>>> merged = f1.merge(f2, left_on='id1', right_on='id2')
   id1  val1  id2  val2
0    1    10    1  500
1    2    20    2  200
2    2    20    2  600
3    3    30    3  300
```

The algorithm of `merge` is as below:
1. Get the value of left_on and right_on, namely id1 and id2, respectively.
2. Compress the value space of both id1 and id2. 
This process ensures a) the same value in the original ids are also the same in the new ids and b) the minimal value is 0 and the maximal value is minimized.
In our example, `id1 = [0, 1, 2, 3, 3]` and `id2 = [1, 2, 4, 0, 1]`. 
Notice that order is not considered at this step. An array of `[3, 2, 2]` will be compressed into `[0, 1, 1]`.
3. Use bucket sort to 'argsort' id1 and id2. The results are denoted as sorter1 and sorter2, respectively. 
Also, the size of each bucket is recorded as count1 and count2.
In out example, `sorter1 = [0, 1, 2, 3, 4]`, `sorter2 = [1, 3, 4, 0, 2]`, `count1 = [1, 1, 1, 2]`, and `count2 = [1, 2, 1, 1]`. 
4. Everything is ready, we just need to iterate over all of the buckets and append the joint values to the final result.
Note that if there are multiple entries in the same bucket, we need to calculate all of the combinations of them. 
```python
result = []
for i in range(len(count1)):
    lc, rc = (count1[i], count2[i])
    il, ir = (0, 0)
    for _ in range(lc):
        ir_old = ir
        for _ in range(rc):
            ir += 1
            result.append(id1[il], val1[il], id2[il], val2[il])  # the tuple is (id1, val1, id2, val2)
        il += 1
```

The above algorithm is only for explanation. Pandas actually returns a DataFrame instead of a list. 
And the joined data is also stored by column.
