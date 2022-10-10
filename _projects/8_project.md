---
layout: page
title: hierarchical agglomerative clustering
description: to understand pointnet ++ 
img: assets/img/hierarchical_clustering/dendrogram.png
importance: 5
category: research
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/hierarchical_clustering/dendrogram.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1. Dendrogram
</div>

```python
import numpy as np
import math
import pandas as pd
from scipy.cluster import hierarchy
import scipy
import matplotlib.pyplot as plt

data = np.random.randn(6,3)

mtx = np.zeros((len(data), len(data)))# the number of row

for i in range(len(data)):
    for j in range(len(data)):
        print("%s between %s" % (i, j))
        distance_x = data[i, :1] - data[j, :1]  # using ',' taking col+1
        distance_y = data[i, :2] - data[j, :2]
        distance_z = data[i, :3] - data[j, :3]
        uclidean_distance = np.sqrt(np.square(distance_x[0]) + np.square(distance_y[0]) + np.square(distance_z[0]))

        print(uclidean_distance)
        uclidean_distance = np.array(uclidean_distance)

        mtx[i, j] = uclidean_distance
print(mtx)
arr1 = []
arr2 = []
pd_idx = ['A','B','C','D','E','F']
pd_col = ['A','B','C','D','E','F']

for i in range(len(mtx)):
    new_mtx = pd.DataFrame(data=mtx)
    print(new_mtx)
    if (np.any(mtx!=0)):
        compare = mtx[np.where(mtx != 0)]
        min_value = np.min(compare)
        #print(min_value)  # min value

        find_idx = np.where(mtx == min_value)
        #print(find_idx[0])  # location of min

        linkage_a = find_idx[0]
        linkage_b = find_idx[1]

        arr1.append([find_idx[0]])

        #print("location value is converted from list to value")

        #print(linkage_a[0],linkage_b[0])  # location value is converted from list to value

        float_len = float(len(mtx))
        new_linkage = np.arange(float_len)

        for j in range(len(mtx)):  # take single linkage
            var_a = mtx[linkage_a[0], j]
            var_b = mtx[linkage_b[0], j]
            if (var_a < var_b):
                new_linkage[j] = np.array([var_a])
            else:
                new_linkage[j]  # must be +1= np.array([var_b])


        new_linkage = np.delete(new_linkage, linkage_b[0])

        #print(new_linkage,"\n")  # single linkage

        mtx = np.delete(mtx, linkage_a[0], axis=0)
        mtx = np.delete(mtx, linkage_b[0], axis=1)
        mtx = np.insert(mtx, linkage_a[0], new_linkage, axis=0)

        mtx = np.delete(mtx, linkage_b[0], axis=0)
        mtx = np.insert(mtx, linkage_a[0], new_linkage, axis=1)
        mtx = np.delete(mtx, linkage_a[0] + 1, axis=1)   # must be +1

    else:
        print("the end of the clustering")
```

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/hierarchical_clustering/result.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2. Result
</div>