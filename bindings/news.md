##API news - October 2015

Previous news: [January 2015: Projects and Samples](archive/news_201501.md)

Features
========

Correlations
------------

*Affects:* REST API calls

*Description:* A new kind of resource (`correlation`) has been
added to BigML.
The `correlation` is treated like the rest of existing resources.
There should be REST api calls to create, get, update and delete them.
The
information it returns encloses a `correlations` block
that contains the fields structure and a second `correlations` block, which
contains a list of different correlation objects. The correlation objects
are labeled by a `name` (coefficients, contingency_tables, and one_way_anova)
and the corresponding result.

The python example for the REST calls for `correlations` can be found in
[commit e26e957](https://github.com/bigmlcom/python/commit/e26e9577f1c818473087163da7e4fb814f3c61a2).

*Test samples:*

We use the same `data/iris.csv` sample file to build a dataset and use this
dataset to create, update and delete a `correlation` resource.

Statistical Tests
-----------------

*Affects:* REST API calls

*Description:* A new kind of resource (`statisticaltest`) has been
added to BigML.
The `statisticaltest` is treated like the rest of existing resources.
There should be REST api calls to create, get, update and delete them.
The
information it returns encloses a `statistical_tests` block
that contains the fields structure, a `fraud` block, which
contains a list of different tests, and `normality` and `outliers` lists.
The tests objects
are labeled by a `name` (benford is the only current test)
and the corresponding result.

The python example for the REST calls for `statisticaltests` can be found in
[commit 70217dd](https://github.com/bigmlcom/python/commit/70217dd7e4ddfa85269cccf1c86c4e191257278f)
and [commit 5a74ef6](https://github.com/bigmlcom/python/commit/5a74ef66ca21efa5a25dd0645f58b581eee8c5d3)

*Test samples:*

We use the same `data/iris.csv` sample file to build a dataset and use this
dataset to create, update and delete an `statisticaltest` resource.
