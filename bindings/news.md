##API news - August 2016

Previous news: [March 2015: Logistic Regression Changes](archive/news_201603.md)

Features
========

Local Logistic Regression Changes
---------------------------------

*Affects:* The object built to handle local predictions based on the
remote `logisticregression` resource. The `LogisticRegression` object in
Python bindings.

*Description:* The local logistic regression object encapsulates the
information found in the `logistic_regression` attribute of the resource
JSON, namely the `bias`, `c`, `eps`, `lr_normalize`, `regularization`and the
`coefficients` structure (see the
[developers documentation](https://bigml.com/developers/logisticregressions#lr_retrieving_a_logistic_regression)
for extensive details).

Several attributes have been added to the last existing version of
logistic regression. The attributes `balance_fields` and `field_codings` are
the most important ones. In addition to that, the coefficients array
has been changed too. All of these changes affect the `predict` method:

1. Coefficients are restructured. The new syntax is an array of arrays. Each
   nested array corresponds to the group of coefficients related to a single
   input field. The nested arrays are sorted according to the order of the
   fields in the `input_fields` attribute. Internally, the coefficients of
   each group are:

- for numeric fields, one coefficient if `missing_numerics` is `false` and
  two if otherwise (the second one is used when the field is missing).
- for categorical fields, if no `field_codings` is used (please, see the
  the next change for details about this) `n + 1` where `n` is the number
  of categories
  found in the summary of the categorical field. The coefficients will be
  sorted according to the list of categories in the summary
  plus a last coefficient used when the field is missing.
- for text or items fields, `n + 1` where `n` is the number of terms in the
  tag_cloud or items list respectively. The coefficients will be sorted
  according to the list of terms in the tag_cloud/items
  plus a last coefficient used when
  the field is missing.

A last array which contains a single coefficient is added at the end of the
rest of groups. This is the bias coefficient.

2. Several coding schemes can be used for categorical fields. They change
   the amount in which each category contributes to the logistic function.
   The one by
   default is `one-shot`, where each category is considered as an individual
   component and assigned a factor of 1
   as described
   in the previous paragraph. Thus, the logistic function is computed
   assigning one coefficient to one
   category, and then the number of coefficients for a
   categorical field is `n + 1` where `n` is the number of categories.
   Using the `field_codings` attribute, you can set other coding schemes:
    2.1. `dummy`: One of the categories is considered a `dummy_class`, so the
          coefficients will be like the ones in `one-shot` but
          their number is `n` because this category does not contribute
          to the logistic function computation.
    2.2. `contrast`: The categories are combined to form vectors of categories.
         The logistic function assigns a coefficient per vector. To define
         how much does each category contribute to each vector, the associated
        `field_codings` entry has a `coefficients` attribute
         which describes this contribution.
         Thus, for a categorical
         field with 4 categories and two vectors (e.g.
         `[[0.5, 0.5, -0.5, -0.5], [1, -1, 1, -1]]) the number of coefficients
         will be 3 (2 coefficients, one per vector and 1 used when
         the field is missing).
    2.3. `other`: this scheme has the same structure as `contrast`, the only
         difference being the restrictions imposed on the vectors'
         coefficients.

For more details on coding schemes, you can check the
`coding categorical fields` subsection of
[API developers docs](https://labs.dev.bigml.com/developers/logisticregressions#lr_logistic_regression_arguments).

3. A new attribute called `balance_fields` controls the numeric fields
   contribution to
   the logistic function. If set to `true`, the value of each numeric field
   in your input data must be balanced by substracting the mean and dividing
   by the standard deviation (both values can be found
   in the `summary` of the field in the `fields` attribute)
   before being multiplied by the logistic regression coefficients to compute
   probabilities.

4. When `lr_normalize` (`normalize` in the JSON sctructure)
   is set to `true`, the contributions of input data must
   be normalized. The norm is computed as the usual square root of the
   scalar product of a vector formed with the input data values.
   Note that this vector includes also a 1 component every time a field
   is missing (is the factor which multiplies the missing coefficient). It
   also includes a 1 component when bias is set.

Another important thing to note is that ties in probabilites
are broken according to the order in which the
classes appear in the objective field summary.

The Python bindings implementation can be found in
[logistic.py](https://github.com/bigmlcom/python/blob/master/bigml/logistic.py).

*Test samples:*

As this computation has different implications for numeric, categorical and
text / items fields, we recommend that you test it on a varied dataset,
such as the `data/movies.csv`.
