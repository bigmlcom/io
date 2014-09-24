##API news - September 2014

Previous news: [August 2014: missing splits](archive/news_201408.md)

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

Docs describing the new resources and some usage are to be found in [commit 95cde22](https://github.com/mmerce/python/commit/95cde228bbd7e0f016e6a7a8b7ddcbeff3dbda72).

The last step should be creating a local anomaly detector using the JSON for
the anomaly detector resource, as we have created a local model from the
model's JSON. This object should have a `score` method to compute anomaly
scores for each input data. The code for this computation is not available yet.
We will provide it asap, but you can have a first glimpse of the underlying
iforest structure in our [iforest description](iforest.md).

*Test samples:*

You can use as example a very small subsample of the
[kdd dataset](data/tiny_kdd.csv).

The anomaly detectors and anomaly scores are not yet available in our current
production servers to be tested. They can be found in our
[anomaly](https://anomaly.dev.bigml.com/) test server that publishes its API
through [anomaly.dev.bigml.io](https://anomaly.dev.bigml.io/). 

(Note that the domain used
in the Python and Node.js bindings can be changed by setting an environment
variable called `BIGML_DOMAIN`. This variable is not only used for test
purposes, but it can also be used to redirect to VPCs, or other specific
domains)

If you need help setting up an account in this server, please contact us via
[support@bigml.com](support@bigml.com).


Local anomaly detector and anomaly score
----------------------------------------

*Affects:* Previously existing `Predicate` object and new objects to be
created (`Predicates`, `AnomalyTree` --resembling `Tree`-- and `Anomaly`
--resembling `Model`--)

*Description:* The goal is to download the JSON `anomaly` resource and build
an `Anomaly` object that provides a method to produce the scoring of new
input data locally. As described in our [iforest description](iforest.md),
the JSON contains an `object` key where you can find the following information
used in the scoring method:

    anomaly['object']['sample_size']
    anomaly['object']['model']['mean_depth']
    anomaly['object']['model']['trees']

The last property is a list of tree dicts, each of which has a `['root']` key
where the entire tree structure is stored. In our python implementation, our
`Anomaly` object has an `iforest` attribute where the trees are stored in a
list as `AnomalyTree` objects. The `AnomalyTree` object is similar to the
`Tree` object used in the local model construction. The main difference is
that each `Tree` object has a predicate attribute where a `Predicate` object
is stored. `Predicate` contains the last rule that
the node represented by the `Tree` instance fulfills. In the `AnomalyTree`
object, the instances in the node must fulfill the list of predicates that
you will find in the `predicates` key. For instance, a node
defined by

    {'population': 105,
     'predicates': [   {   'field': '000017',
                           'op': '<=',
                           'value': 13.21754},
                       {   'field': '000009',
                           'op': 'in',
                           'value': [   '0']}]}],
      'children': [ ... ]}

where the instances in the node fulfill that field `000017`'s value is equal
or less than `13.21754` and the value of field `000009` is in the list of
values `['0']` would be represented by an `AnomalyTree` with a `predicates`
attribute containing a `Predicates` object that stores the list of two
`Predicate` objects built on the dicts of the `predicates` key.

The existing `Predicate` object has also been modified to include the new
`in` operator that you can see in the second rule of our previous example.

Based on such a structure, the local computation of the anomaly score for
a given input data involves computing:

- The maximum depth that input data reaches when run through each `AnomalyTree`
- The values of `anomaly['object']['sample_size']` and
  `anomaly['object']['model']['mean_depth']`

Combining the list of depths, the number of `AnomalyTrees` and the
`sample_size` and `mean_depth` as explained in the
[iforest description](iforest.md) or in the python
[commit f5d56f5](https://github.com/bigmlcom/python/commit/f5d56f58e7fe1524e83f61f7be6dfe517b9069df)
you obtain the anomaly score for the given input data.


*Test samples:*

No specific test samples are provided, because the best way of testing local
anomaly objects is comparing the obtained local scores obtained with them in
any anomaly detector you build, regardless of the dataset, to
the ones obtained calling the API using the corresponding
`create_anomaly_score` function. You can also check the anomaly detector
scoring method by
using the `top_anomalies` information, that contains both the fields' value
of the anomalies and the scores computed remotely for these instances.


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
