##API news - November 2015

Previous news: [October 2015: Correlations and Statistical Tests](archive/news_201510.md)

Features
========

Logistic Regression
-------------------

*Affects:* REST API calls

*Description:* A new kind of resource (`logisticregression`) has been
added to BigML.

The `logisticregression` is a kind of supervised machine learning method
for classification problems, so your dataset must have
a categorical field as objective field to use this model.
The method uses the input fields or
predictors of the dataset to compute the probability associated to
each possible category in the objective field. The probability is the
value of a logistic function of the form,

p = 1 / (1 + exp[-(b<sub>1</sub> * x<sub>1</sub> + b<sub>2</sub> * x<sub>2</sub>
... + b<sub>n</sub> * x<sub>n</sub> + b<sub>missing</sub> + b<sub>0</sub>)])

which linearly combines the values in the
input_data fields using some coefficients (please, see more details in the
[Local Logistic Regression](#locallogisticregression) section).
Each category has its own
coefficient set and the `logisticregression` computes these coefficients
from the training data.

The basics will be creating wrappers for the
REST api calls to create, get, update and delete logistic regressions. The
resource is similar to the `model` or `cluster` resources, it has a
private resource id and can also be shared.

The information it returns, encloses a `logistic_regression` block
that contains the fields structure and also contains a `coefficients`
attribute, that can
be used to compute predictions locally.

The python example for the REST calls for `logisticregression` can be found in
[commit 1042538](https://github.com/bigmlcom/python/commit/1042538762096b89f71cb9e3f8ac96660b9bca03).

*Test samples:*

We use the `data/iris.csv` sample file to build a dataset and use this
dataset to create, update and delete a `logisticregression` resource. The
`data/spam.csv` sample is also used to test the local Logistic Regression
object (see next section)

<a name="localogisticregression"></a>
Local Logistic Regression
-------------------------

*Affects:* Its a new object that encapsulates the JSON information downloaded
from a remote `logisticregression` resource and adds a `predict` method
to create predictions locally. Similar to the `Model` object in Python
bindings.

*Description:* The local logistic regression object will encapsulate the
information found in the `logistic_regression` attribute of the resource
JSON, namely the `bias`, `c`, `eps`, `lr_normalize`, `regularization` and the
`coefficients` structure (see the [developers documentation](https://bigml.com/developers/logisticregressions#lr_retrieving_a_logistic_regression)
for extensive details). As to the `coefficients` structure, its an array of
elements that pair each category in the list of possible categories for the
objective field and the coefficients that must be used to compute its
associated probability. The final coefficient in the list of coefficients
will always be the `bias` term `b<sub>0</sub>`, but the rest of the list
will have a different structure depending on the types of the
fields in the dataset.

The predict method should compute the probabilites for all the possible
categories in the objective field and return the list of category, probability
pairs and, as prediction, the most probable one.

For a use case where the rest of fields in the dataset
are numeric fields (like `iris.csv`), the structure of the coefficients
array would be:

[['category1', [b<sub>1</sub>, b<sub>2</sub>,... , b<sub>n</sub>, b<sub>0</sub>],
 ['category2', [b<sub>1</sub>, b<sub>2</sub>,... , b<sub>n</sub>, b<sub>0</sub>],
...]

where the first element in the pair is the category and the second is the
list of coefficients `b<sub>i</sub>`. `i` corresponds to the field in the
`ith` position (from `1 to n`) of `input_fields` and `b<sub>0</sub>`
is the `bias` term. Thus, when computing the probability for each category
to build the predict method you need:

- for each category, compute the linear combination:
    lr_exponent = b<sub>0</sub> + b<sub>1</sub> * input_data[input_field[1]] +
    ... + b<sub>n</sub> * input_data[input_field[n]]
- compute the logistic function:
    p = 1 / (1 + exp[- lr_exponent ])
- normalize, so that the total sum of probabilities for all the possible
  categories is 1.

For categorical fields, each categorical field will be related to `m+1`
coefficients in each list of coefficients,
where `m` is the number of categories. Ranging from `1 to m`,
the `ith` coefficient will be the one related to the `ith` category, as
ordered in the corresponding field's `['summary']['categories']` list. There
will be an additional `m+1` coefficient that should be used if the field is
missing in the input data. Thus, to compute the probability when input data
fieds are categorical you will need:

- The list of categories of the field as found in the
  `['summary']['categories']` attribute of the field structure
- Check the categorical value of the field in your input data.
  If missing, then add to `lr_exponent` the
  `mth + 1` coefficient. Otherwise, add the `ith` coefficient where `i` is the
  index in the list of categories of the one in your input data.
- Compute the logistic function and normalize across objective field
  categories, as in the second and third steps of the numeric case.

For text fields, each field will be related to `m+1` coefficients of each
list of coefficients, where `m` is the number of terms in the bag of words
of the field, as described in the `[tag_cloud]` attribute of the field's
structure. The contribution of
each term in the input data contents of the field to the `lr_exponent` will
be the product of the occurrences of the `ith` term (according to the
`tag_cloud` list order) in the input data field and
the `ith` coefficient. As for categorical fields, there's also an `mth + 1`
coefficient if the field is missing. In this case, though, a field is
considered to be missing either if it doesn't appear in the input data or
if it appears but the terms that contains are not in the `tag_cloud`. Thus,
to compute the probability when input data fields are text fields you will
need:

- The terms in the `tag_cloud` as described in the corresponding fields
  structure.
- When stemming is used, the related terms in `term_forms` (also available in
  the field's structure). Any occurrence of a term in `term_forms` must be
  added to the occurrences of the corresponding main term in the `tag_cloud`.
- Check the text field value of the field in your input data. If missing, then
  add to the `lr_exponent` the `mth + 1` coefficient. Otherwise, parse the
  terms, compare them to the `tag_cloud` and `term_forms` lists, count the
  occurrences of each and add the corresponding `b<sub>i</sub> * occurences`
  contribution to `lr_exponent`.
- Compute the logistic function and normalize across objective field
  categories, as in the numeric case.

The Python bindings implementation can be found in
[logistic.py](https://github.com/bigmlcom/python/blob/master/bigml/logistic.py).
The `LogisticRegression` object stores the categories list for each categorical
field in `self.categories[field_id]`, the tag cloud for each text field in
`self.tag_clouds[field_id]` and the equivalent terms
in `self.term_forms[field_id]`. It also updates the fields structure, adding
a `coefficient_shift` index to each field in the fields structure.
This `coefficient_shift`
stores the position in the `coefficients` list where the coefficients related
to the concrete field start. For a dataset containing a numeric field, a
categorical field with two categories and another numeric field, the
`coefficient_shift` for the first field would be `0`, for the second it
would be `1` and for the third it would be `4` (2 categories + missing).
Finally, the total coefficients array would have `6` elements, the last one
corresponding to the bias term.

*Test samples:*

We can test the categorical and numeric fields using the `data/iris.csv` sample
and text fields with the `data/spam.csv` sample file.
