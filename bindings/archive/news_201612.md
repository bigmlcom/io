##API news - December 2016

Previous news: [August 2016: Logistic Regression Changes](archive/news_201608.md)

Features
========

Topic Model, Topic Distribution, Batch Topic Distribution
---------------------------------------------------------

*Affects:* REST API calls

*Description:* New kinds of resources (`topicmodel`, `topicdistribution`,
`batchtopicdistribution`) have been
added to BigML.

The `topicmodel` is a kind of unsupervised machine learning method
which uses `LDA (Latent Dirichlet Allocation)` to extract the
topics contained in a collection of documents. The BigML `topicmodel`
is built considering each row in your dataset as a document and its text
fields contents as the document's text. If multiple fields are given as
inputs, they will be automatically concatenated and the content will be
considered as a bag of words.

When input data is run through a `topicmodel`, the generated resource is
a `topicdistribution`. The `topicdistribution` resource
shows in a list the probability that the given input_data belongs to each
of its topics. To create a `topicdistribution` you need a topicmodel/id
and the input data.

A `batchtopicdistribution` will compute a topic distribution
for each instance in a dataset in only one request.
To create a new batch topic distribution you need a topicmodel/id and a
dataset/id.

The basics will be creating wrappers for the
REST api calls to create, get, update and delete each of these resources.


The python example for the REST calls for `topicmodel`, `topicdistribution`,
`batchtopicdistribution` can be found in

- [topicmodel](https://github.com/bigmlcom/python/blob/next/bigml/topicmodelhandler.py).
- [topicdistribution](https://github.com/bigmlcom/python/blob/next/bigml/topicdistributionhandler.py).
- [batchtopicdistribution](https://github.com/bigmlcom/python/blob/next/bigml/batchtopicdistributionhandler.py).

*Test samples:*

We use the `data/spam.csv` sample file to build a dataset and use this
dataset to create, update and delete a `topicmodel` resource and their related
`topicdistribution` and `batchtopicdistribution` resources.

<a name="localtopicmodel"></a>
Local Topic Model
-----------------

*Affects:* It's a new object that encapsulates the JSON information downloaded
from a remote `topicmodel` resource and adds a `distribution` method
to create topic distributions locally. Similar to the `Model` object in Python
bindings.

*Description:* The local topic model object will encapsulate the
information found in the `topic_model` attribute of the resource
JSON needed to compute the topic distributions. The python example for the
local topic model can be found in:

- [local topic model](https://github.com/bigmlcom/python/blob/next/bigml/topicmodel.py)


*Test samples:*

We can test the local topic model using the `data/sample.csv` sample file.
