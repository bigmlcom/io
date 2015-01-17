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
be assigned. The assignation is currently done only when creating `sources`,
where a project id can be given as additional argument. Then, all the
resources stemming from a source assigned to a project will inherit the same
project id property. This provides a mechanism for grouping together under
the same project a set of resources.

The main differences with the rest of existing resources are:

- Projects are only created for organizational purposes. They are nor related
or stem from any other existing resource.
- Each project can have an associated name, but they don't need to be unique.
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
