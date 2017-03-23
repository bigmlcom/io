##API news - February 2017

Previous news: [December 2016: Logistic Regression Changes](archive/news_201612.md)

Features
========

Local boosted ensembles
-----------------------

*Affects:* The local `Ensemble` class, the local `Model` class, the
`MultiModel` and `MultiVote` classes and adds a new `BoostedTree` class.

*Description:* A new kind of ensemble is available to predict classifications
and regressions. The ensemble uses a boosting algorithm which computes the
predictions using several interations. In each iteration a new set of models
is build whose objective is computing a gradient that leads to the final model.
Thus, in this case there are several differences when compared to the
`random decision forests` or `bagging` ensembles:

- All the models have a numeric objective field (the gradient)
  regardless of whether the original objective field is a numeric or
  categorical one.
- When used for regression, the prediction of the ensemble
  will be computed by adding up (with some
  weights and normalizations) the predictions of all the models.
  The result of the ensemble
  prediction will be a number and will have no associated error.
- When used for classification, each model will be associated to one of the
  categories. The list of models asociated to a particular category will
  be used to predict the probability of the category given the input data.
  Again, the prediction per category will be computed by adding up (with some
  weights and normalizations) the contributions of its associated models. The
  result of the ensemble prediction will be the most probable category and its
  confidence will be the associated probability.

The ensemble JSON constains a `type` attribute which must be used to
decide whether the ensemble is a `Decision Forest` (bagging or Random
Decision Forest) or a `Boosted Trees` ensemble (`type=1`). If your binding
supports local ensembles from lists of models, you will also need to decide
whether the models in the list are boosted trees. In order to do so, look
for the `boosted_ensemble` attribute in the models, which is a boolean
telling whether the models belong to a boosted trees ensemble.

The computation of predictions for boosted ensembles uses some information
stored in the structure of their associated trees, besides the output
predicted in each node:

- `weight` is an attribute of each model (at the model level)
  that contains the weight that will
  be used when adding the model's contribution.
- `lambda` is an attribute of each model (one of the `boosting` attributes)
- `g_sum` and `h_sum` are attributes stored in each node of the models.

Note that these models won't store `distribution` informations in their nodes.
Thus, a new class is used to substitute the `Tree` class used by the existing
`Model` class. The `BoostedTree` python class, that can be found in:

[BoostedTree](https://github.com/bigmlcom/python/blob/boosted/bigml/boostedtree.py)

and is used in the `Model` instantiation:

[Model](https://github.com/bigmlcom/python/commit/1c55e346156491c1383db92655224f8eedab1ac1#diff-95c7cbead76744330bc93b197b3d14e9)

The `BoostedTree`  class will provide a `predict` method. The usual
missing strategies
(LAST_PREDICTION and PROPORTIONAL) are to be allowed.

- LAST_PREDICTION: When using this strategy we'll need to return the
  information of the last node whose condition is met by the input data.

- PORPORTIONAL: When using this strategy we'll need to return
  the acumulated `g_sum` and `h_sum` for all the nodes whose conditions are
  met by the input data.

The final ensemble prediction will be computed by following these rules:

- Regression with last prediction missing strategy: The prediction is computed by
  adding the product of `prediction * weight` for each model in the ensemble.

- Classification with last prediction missing strategy: A probability is
  computed for each possible category by selecting the models associated to
  this category
  and computing the sum of products (`prediction * weight`) for them. Once you
  have this quantities computed per each category, you normalize them using the
  `softmax` function.

- Regression with proportional missing strategy: the prediction is computed by
  adding the product of `(- sum(g_sum) / (sum(h_sum) + lambda)) * weight` for
  each model in the ensemble. `lambda` is one of the attributes in the model.

- Classification with proportional missing strategy: the probability of each
  category is computed
  by adding the product of `(- sum(g_sum) / (sum(h_sum) + lambda)) * weight`
  for each model associated to this category in the ensemble. The final
  prediction
  is the most probable category. The `softmax` function is
  used to normalize these probabilities. To decide in case of tie break the
  order of categories
  in the ensemble objective field summary is used.
  This has caused the `Ensemble`
  to include also the fields structure.

[Ensemble](https://github.com/bigmlcom/python/blob/boosted/bigml/ensemble.py)

The information of `weight` and `class` is added to each prediction in the
`MultiModel` class:
[MultiModel](https://github.com/bigmlcom/python/commit/1c55e346156491c1383db92655224f8eedab1ac1#diff-21ed79c5ab53d55f5e4f79a211d6875f)

The implementation of these sums and normalizations is done
in the `MultiVote` class.
[MultiVote](https://github.com/bigmlcom/python/commit/1c55e346156491c1383db92655224f8eedab1ac1#diff-90180690cbb5b54d110f92b01c9e3878)

We choose to add a `BOOSTING = -1` code in the `MultiVote` class to mean
that boosting combiner is used. The code is set to a negative to allow that
other user-given combiner codes can be added in the future using the following
correlative integers.

*Test samples:*

We use the `data/iris.csv` and `data/grades.csv` sample files to
build a boosted ensemble and use it in a local `Ensemble`.
