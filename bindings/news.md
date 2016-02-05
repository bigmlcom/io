##API news - January 2016

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
