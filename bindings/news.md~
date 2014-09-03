##API news - August 2014

Features
========

Missing splits
--------------

*Affects:* Local Models and Ensembles' predictions

*Description:* Trees have been improved by letting splits include also missing
values in one of their branches. Previously, each split in a tree had two
predicates, that compared to a numeric or string **non-null** value
(field < 1 or field >= 1). Now the operators in the tree predicates include
also expressions
such as <* or >=* where the asterisk means that the predicate must be true also
in case the field is missing. An example of what could be found in the
tree split would be:

predicate: {
    field: "000002",
    operator: "<=*",
    value: 47
}

in this case, the expression means that the predicate is true either when the
field `000002` is missing or when its value is less or equal 47.

Another type of missing value could be found when using the regular operators
`=`, `\=` or `!=`:

predicate: {
    field: "000002",
    operator: "=",
    value: null
}

that fulfills only if the field `000002` is missing. If the operator is `\=` or
`!=` it will fulfill if the field exists, regardless of its value.

You can see the main changes in Python bindings (commits
[6cdee48](https://github.com/bigmlcom/python/commit/6cdee48879272018faa0b1701dc608666e1c3e66)
and [67213ec](https://github.com/bigmlcom/python/commit/67213ec2fb044a22e3dd6a213088d960b32de71c)
and Node.js bindings (commits
[57a4e84](https://github.com/bigmlcom/bigml-node/commit/57a4e8425f41a1b5f8e4953518ea7e2f1c278ce5)
and [1a1090b](https://github.com/bigmlcom/bigml-node/commit/1a1090b9309bc2422b43b0183da4f392e95a0a71).

At the same time, the API has a new argument for the `create` call of models
and ensembles that can be used to set on or off the use of missing split
operators. The setting the `missing_splits` argument to `false` will cause
the created model to avoid using the new operators. This can be used, for
instance, to ensure success in test suites made before the new operators
appearance.

*Test samples:*

The simplest example of tree using missing splits is obtained by creating
a model from the following
[source file](bindings/data/iris_missing.csv)
using the
create source, create dataset and create model calls and ensuring that
`missing_splits` is set to true in the last one.

Other examples are the
[titanic dataset](bindings/data/titanic.csv.gz),
excluding fields "000000",
"000004", and "000008" (which are "Name", "Fare", and "Job") to predict
survival.  The first split ends up checking whether the categorical
field "000009" (LifeBoat) is missing or not.
Deeper in the tree there are some missing splits
on numeric fields (">*" and "<=*"splits).

Also a very small
[handmade dataset](bindings/data/missing-test.csv).
The dataset has 5 fields. Training the model with any of the field
combinations below should end up with a tree that uses missing splits
and manages to fit the training data exactly.  So predictions using the
training set should be perfect.  Without missing splits, the tree won't fit
the training data exactly.

Input fields: 0, 1 -- Objective field: 3
Input fields: 0, 1 -- Objective field: 4
Input fields: 0, 2 -- Objective field: 3
Input fields: 0, 2 -- Objective field: 4

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
