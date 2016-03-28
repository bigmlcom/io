##API news - March 2016

Previous news: [November 2015: Logistic Regression](archive/news_201511.md)

Features
========

Local Logistic Regression Changes
---------------------------------

*Affects:* The object built to handle local predictions based on the
remote `logisticregression` resource. The `LogisticRegression` object in
Python bindings.

*Description:* The local logistic regression object encapsulates the
information found in the `logistic_regression` attribute of the resource
JSON, namely the `bias`, `c`, `eps`, `lr_normalize`, `regularization` and the
`coefficients` structure (see the
[developers documentation](https://bigml.com/developers/logisticregressions#lr_retrieving_a_logistic_regression)
for extensive details). A new option is added: `missing_numerics`. If set
to True, an additional coefficient will be appended after the coefficient
for each numeric field, which should be used to compute predictions
when the field is missing. Previously built logistic regressions should be
updated to include this parameter set to False. Thus, the number of
coefficients associated to each numeric field can be 1 (as it was before,
if `missing_numerics` is False) or 2 (if `missing_numerics` is True).

The Python bindings implementation can be found in
[logistic.py](https://github.com/bigmlcom/python/blob/master/bigml/logistic.py).

*Test samples:*

We can test numeric fields using the `data/iris.csv` sample file.

New Item field type
-------------------

*Affects:* The classes defined to encapsulate models, anomalies, clusters and
every field type check.

*Description:* A new field type is added: items. The type is inteded for
fields whose contents are lists of strings joined by any separator. The
separator is inferred but can also be specified by the user. Separators can
be set either as a string or as a regular expression. The typical structure
for an items field would be:

```python
    {u'column_number': 7,
     u'datatype': u'string',
     u'item_analysis': {u'separator': u'$', u'separator_regexp': None},
     u'name': u'genres',
     u'optype': u'items',
     u'order': 7,
     u'preferred': True,
     u'summary': {u'average_length': 15.41724,
                  u'items': [[u'Drama', 108],
                             [u'Action', 97],
                             [u'Comedy', 86],
                             [u'Adventure', 63],
                             [u'Sci-Fi', 54],
                             [u'Romance', 45],
                             [u'Thriller', 43],
                             [u'Horror', 23],
                             [u"Children's", 20],
                             [u'Crime', 20],
                             [u'War', 16],
                             [u'Animation', 14],
                             [u'Fantasy', 14],
                             [u'Musical', 13],
                             [u'Mystery', 10],
                             [u'Film-Noir', 4],
                             [u'Western', 3],
                             [u'Documentary', 1]],
                  u'missing_count': 0}}
```

which shows an `item_analysis` attribute that contains the `separator` or
`separator_regexp` used.

In the Python bindings implementation the affected classes are:

- [Predicate](https://github.com/bigmlcom/python/commit/b9b208598483044ab7f5118d299dd44b21f82b52) (changes are used from Model and Anomaly class)
- [Cluster](https://github.com/bigmlcom/python/commit/016b5a1f444a8af2e5b1caafa3dd43196d57fa72)
- [LogisticRegression](https://github.com/bigmlcom/python/commit/3fe4a1d2bfe125a9aa1246cd15411b50358bdcc6)

In `prediction` and `centroid` methods, the `items` fields are handled
very similarly to `text` fields, the main difference being that no stemming
is applied and no case-sensitivity is considered.
Thus, no alternative forms are added and each item is counted
as present or absent in input data (as opposed to `text` fields where
occurrences are computed and significant to the prediction algorithm).

*Test samples:*

We can test items fields using the `data/movies.csv` sample file.


Associations
------------

*Affects:* REST API calls

*Description:* A new kind of resource (`association`) has been
added to BigML. It is meant to bring association discovery to the set
of tools to find out relations among values
in high-dimensional datasets.

In BigML, the Association resource object can be built from any dataset, and
its results are a list of association rules between the items in the dataset.
There are some metrics to ponder the quality of these association rules:
support, coverage, confidence, leverage and lift. Check the
[developers section](https://bigml.com/developers/associations) for more
detailed information.

The basics will be creating wrappers for the
REST api calls to create, get, update and delete associations. Their
structure will be similar to the `cluster` CRUD calls. Associations
have a private resource id and can also be shared.

The information they return, encloses an `associations` block
that contains the fields structure and also contains an `items`
attribute, which lists all the items considered. Items are based on categories
for categorical fields, terms for text fields and items for items fields.
Numeric fields are discretized in ranges. See the `items object` properties
in [association properties](https://bigml.com/developers/associations#ad_association_properties).
It also includes a `rules` attribute that lists the association rules that
relate these items. Each association has a set of items as antecendents
in the left-hand-side (lhs) of the rule and an item or consequent in the
right-hand-side (rhs) of the rule. See the `rules object` section in
[association properties](https://bigml.com/developers/associations#ad_association_properties).

The python example for the REST calls for `associations` can be found in
[commit 62050f0](https://github.com/bigmlcom/python/commit/62050f0a3b377fefec1d51b1f035ae76df1d3ae1).

*Test samples:*

We use the `data/iris.csv` sample file or the `data/tiny_mushrooms.csv`
to build a dataset and use this
dataset to create, update and delete an `association` resource.

<a name="local-associations"></a>

Local Associations
------------------

*Affects:* Its a new object that encapsulates the JSON information downloaded
from a remote `association` resource and adds methods to retrieve and filter
- the list of items in the association relations
- the list of association rules
It also provides methods to export these rules to a CSV and generate a
printable summary of them.

*Description:* The local association object will encapsulate the
information found in the `associations` attribute of the resource
JSON, namely the `search_strategy`, `complement`, `discretization`,
`fields_discretization`, `k`, `prune`, `significance_level` and the
`items` and `rules` lists (see the
[developers documentation](https://bigml.com/developers/associations#ad_association_properties)
for extensive details).

Currently, there's no `predict` method associated to this object but it's
expected to be added in a short-medium term. The predictions of `associations`
are `association sets`.

The Python bindings implementation can be found in
[association.py](https://github.com/bigmlcom/python/blob/master/bigml/association.py).

*Test samples:*

We can test the categorical and numeric fields using the `data/iris.csv` sample
and other fields with the `data/movies.csv` sample file.


Bug fixes
=========

Local Model Predictions for missing text fields
-----------------------------------------------

*Affects:* The object built to handle local predictions based on the
remote `model` resource. The `Model`, `Tree` and `Predicate` objects in
Python bindings. It also affects the `actionable model` outputs, like the
one in the `Model` `.python()` method.

*Description:* Missing fields (fields that are not present in the prediction
request) are already handled in the local model predict method. The default
behaviour when the algorithm reaches a split where the associated field is
missing in the input data is using `last prediction` strategy. In this case,
the last prediction found in the parent node is returned and the process stops.
There's a second available strategy, namely `proportional` strategy, where
the process goes on by following both branches that emerge from the node and
averaging or using plurality on the resulting predictions as final predicition.
However, if the missing field is a `text` or `items` field, this should be
equivalent to having an empty field. In this case, when you have no information
for a `text` or `items` field and find a split on this field, the algorithm
should follow the `doesn't contain` branch.

The Python bindings implementation can be found in three commits.

For the
[last prediction strategy](https://github.com/bigmlcom/python/commit/4f5b1d25be9dd4670c944adccda159b61cef1869)
part of the code.
For the [proportional strategy](https://github.com/bigmlcom/python/commit/ca291847d50927cbe3ff015cc28a809e15f558ef)
part of the code.
For the
[actionable model](https://github.com/bigmlcom/python/commit/9b74739863622b6dfe7f3a4104c9abd60aaafeaf)
part of the code.


*Test samples:*

We can test missing `text` fields using the `data/text_missing.csv`
sample file following the
[tests in the Python bindings](https://github.com/bigmlcom/python/commit/9cddf1941d6e35aca0c8f75a6d5af79a34750d96).
