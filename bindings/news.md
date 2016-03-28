##API news - March 2016

Previous news: [January 2016: Local Logistic Regression,
Associations](archive/news_201601.md)

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
