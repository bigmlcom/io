##API news - September 2014

Previous news: [missing splits](archive/news_201408.md)

Features
========

Anomaly detection
-----------------


*Affects:* REST API calls and new local model type

*Description:* A new kind of unsupervised model has been added to BigML.
Using iforests, this
model detects anomalous data in a training dataset.  The
information it returns encloses a `top_anomalies` block
that contains a list of the most anomalous
points. For each, we capture a `score` from 0 to 1.  The closer to 1,
the more anomalous. We also capture the `row` which gives values for
each field in the order defined by `input_fields`.  Similarly we give
a list of `importances` which match the `row` values.  These
importances tell us which values contributed most to the anomaly
score.

The first step to bring anomaly detection to the bindings is adding the
REST API calls interface. Their code resembles the model's REST API calls.
They can be created from a dataset or list of datasets. You can find an
example in 
[commit ebaea97](https://github.com/bigmlcom/python/commit/ebaea97a5d5eae32b0f4440716d3a97ee37a829e).

The second part is adding the REST API calls for the anomaly score. The anomaly
score interface must be similar to the model's predictions interface. They need
an anomaly detector and the input data to be created. You can find an example
in [commit 0c56bc2](https://github.com/bigmlcom/python/commit/0c56bc2000c40be7102f4ccaa92188397d209345).

The last step should be creating a local anomaly detector using the JSON for
the anomaly detector resource, as we have created a local model from the
model's JSON. This object should have a `score` method to compute anomaly
scores for each input data. The code for this computation is not available yet.
We will provide it asap, but you can have a first glimpse of the underlying
iforest structure in our [iforest description](iforest.md).

*Test samples:*

You can use as example a very small subsample of the
[kdd dataset](data/tiny_kdd.csv).

Bugs
====

An implementation bug has been detected both in the Python and Node.js old
version of the bindings. It applies only to the regression models when
proportional missing strategy is used in local predictions. There's a singular
use case where the node in the tree that gives the final prediction contains
only 1 instance. This is a singular case where the algorithm that is used
at present to compute the associated error cannot be used (the previous code
was returning NaN as result). Instead, the
returned prediction, error and distribution values must be the ones in the
1-instance node. You can check commit
[bdc625f](https://github.com/bigmlcom/python/commit/bdc625fe4ea0ba84c265e5cf7545c10595f86b95)
for a solution to the issue in the Python bindings.
