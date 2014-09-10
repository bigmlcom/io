# Isolation Forests

This document outlines how to generate anomaly scores using BigML's
isolation forests.  It does not currently explain how to generate the
`importance` metric that comes with each anomaly score, but that will
be added in the future.

## Node Predicates

Isolation trees have a structure similar to that of BigML
classification and regression trees.  A major difference is that
instead of a single `predicate` each node will have a list of
`predicates`.

```json
{"root":
  {"predicates": [true],
   "population": 100,
   "children": [
     {"predicates": [
        {"field": "000003",
         "op": ">",
         "value": 2.37},
        {"field": "000004",
         "op": "in",
         "value": ["Iris-virginica", "Iris-setosa"]}],
      "population": 80},
     {"predicates": [
       {"field": "000003",
         "op": "<=",
         "value": 2.37}],
      "population": 20}]},
 "training_mean_depth": 8.46875}
```

Individual predicates can include the value `true` (which always
evaluates as true) or maps containing a `field`, `op`, and `value`.
The operators can include the or-missing `*` form.  iForests also add
the `in` predicate for categorical fields.  When the predicate's
operator is `in`, the value will be a set of categories.  At test
time, if the field's value is a member of the set then the predicate
is true.  When a missing value is tested as part of the `in` or `=`
predicates, we use `null`.

A sample:

```json
"predicates" : [ {
  "field" : "000003",
  "op" : ">*",
  "value" : 32.35
}, {
  "field" : "000001",
  "op" : ">=",
  "value" : 25.25
}, {
  "field" : "000001",
  "op" : "<=",
  "value" : 41.75
}, {
  "field" : "000009",
  "op" : "in",
  "value" : [ null, "foo", "bar" ]
} ]
```

For a more exhaustive example of an isolation forest containing
numeric and categorical fields with missing values:
https://gist.github.com/ashenfad/4c8097ea16d66528b339

## Evaluating an Isolation Tree

To evaluate a single isolation tree, we want to run an instance
through the tree and output the depth of the node the instance
terminates at.  The root of the tree is depth 0, its children are
depth 1, their children depth 2, etc.

If a node has any children whose predicates are all true given the
instance, then the instance will flow through that child.  If the node
has no children or no children with all valid predicates, then the
node will output its depth.

Here is some example psuedo-code returning the final depth given an
instance/row:

```
function evalPredicates (node, row) {
  for (predicate in node.predicates) {
    if (!predicate.isTrue(row)) {
      return false;
    }
  }
  return true;
}

function evalNode (node, depth, row) {
  if (node.hasChildren) {
    for (child in node.children) {
      if (evalPredicates(child, row)) {
        return evalNode(child, depth + 1, row);
      }
    }
  }

  // return current depth if there are no children or no children with
  // valid predicates
  return depth;
}

// evalNode(root, 0, someRow);
```

Server-side implementation for those with access to BigML source:
https://github.com/bigmlcom/models/blob/master/src/java/models/iforest/ANode.java#L65

## Combining Tree Evaluations

To produce an anomaly score, we first need to evaluate each tree for
its depth result.  We find the average of those depths to produce
an `observedMeanDepth`.  We calculate an `expectedMeanDepth` using the
`sample_size` and `mean_depth` parameters which come as part of the
forest message.  We combine those values as seen below, which should
result in a value between 0 and 1.

Pseudo-code for generating an anomaly score:

```
function defaultDepth(sample_size) {
  return 2 * ((0.5772156649 + Math.log(sample_size - 1)) -
              (sample_size - 1) / sample_size);
}

function score(forest, row) {
  var depthSum = 0;
  for (tree in forest.trees) {
    depthSum += evalNode(tree, 0, row);
  }
  var observedMeanDepth = depthSum / forest.trees.length();
  var expectedMeanDepth = Math.min(defaultDepth(forest.sample_size),
                                   forest.mean_depth);
  return Math.pow(2, -observedMeanDepth / expectedMeanDepth);
}
```

Server-side implementation for those with access to BigML source:
https://github.com/bigmlcom/models/blob/master/src/clj/models/iforest/score.clj
