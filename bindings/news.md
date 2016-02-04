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
