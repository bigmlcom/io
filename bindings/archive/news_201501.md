##API news - January 2015

Previous news: [September 2014: anomaly detection](archive/news_201409.md)

Features
========

Projects
--------

*Affects:* REST API calls

*Description:* A new kind of resource (`project`) has been added to BigML.
The `project` is treated like the rest of existing resources.
There should be REST api calls to create, get, update and delete them.
The `project` is to be used as an organizational unit where all resources can
be assigned. The assignment is currently done only when creating `sources`,
where a project id can be provided as an additional argument. Then, all the
resources stemming from a source assigned to a project will inherit the same
project id property. This provides a mechanism for grouping together under
the same project a set of resources.

The main differences with the rest of existing resources are:

- Projects are only created for organizational purposes. They are not related
or stem from any other existing resource.
- Each project can have an associated name, but it doesn't need to be unique.
The id for the project will be a string like `project/50c0de043b563519830001c2`
similar to the ones that identify the rest of resources.
- Projects are created in a synchronous way, so they have no explicit
`status` info in the returned JSON structure and must be considered
in a `FINISHED` state once they are created.
- The `delete` call for a project will cause the deletion of all resources
linked to that project. This can be a long task, so at present even if the
call returns immediately, the project and its associated resources
might take some time to disappear completely from the resources lists.
For instance, if you send two consecutive
delete calls on the same project, both can return the same `204` code.

The python example for the REST calls for `projects` can be found in
[commit 9bb710f](https://github.com/bigmlcom/python/commit/9bb710f80460fadd7a0b4e21a48af53753140892).

*Test samples:*

There are no test samples for projects, because they don't rely on any
existing data or resources to be created.

Samples
--------

*Affects:* REST API calls

*Description:* A new kind of resource (`sample`) has been added to BigML.
The `sample` is an in-memory object that is available for queries to obtain
subsets of the data in a dataset. The queries can filter the size, fields
or rows of the dataset.

There should be REST api calls to create, get, update and delete them.
The `create` method needs a dataset object or id as origin and returns a
sample object that will follow the usual status workflow until finished.
The sample id will be something like `sample/50c0de043b56351983000152`.

A singularity of samples is that they are automatically deleted once a certain
time to live expires. The time to live is updated every time a new GET call is
done on the sample.

The python example for the REST calls for `samples` can be found in
[commit f983766](https://github.com/mmerce/python/commit/f983766208c944fd0169e06e8352dbeb96f0d38a).

*Test samples:*

For samples, any dataset can be a good candidate to try them, so we include
no particular data file for that.


Bugs
====

An implementation bug has been detected both in the Python and Node.js old
version of the bindings. It applies only to the local anomaly detector objects
when `id_fields` are used. The fields in the `id_fields` array are only used
as reference, not in the iforest models, and their values do not appear in
the `row` attribute of the `top_anomalies`. The old code used
`input_fields`, included the `id_fields` if any. It also corrects
that double quotes should be escaped if they appear in values of categorical
fields. You can check commit
[b879a24](https://github.com/bigmlcom/python/commit/b879a243bf30f8ee8efed9d3150f1358e34bf405)
for a solution to the issue in the Python bindings.
