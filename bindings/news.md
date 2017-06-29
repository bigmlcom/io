##API news - June 2017

Previous news: [February 2017: Boosted ensembles](archive/news_201702.md)

Features
========
Time-series and Forecasts
-------------------------

*Affects:* REST API calls

*Description:* New kinds of resources (`timeseries` and `forecast`) have been
added to BigML.

The `timeseries` is a kind of supervised machine learning method
which uses `exponential smoothing` to analize a temporal numeric series.
By default, a handful of models is built when analyzing a time-series.
The parameters
that define these models are:

- the error type (additive -A- or multiplicative -M-)
- the trend type (none -N- , additive -A- or multiplicative -M-)
- the trend damping, only applicable to additive or multiplicative trends (true, false)
- the seasonal type (none -N-, additive -A- or multiplicative -M-)

according to this types, the models are labeled using the letters that
describe the error, the trend (a "d" is appended if using damping) and the
seasonal type. For instance, the model with additive error, multiplicative
damped trend and no seasonal type is named "A,Md,N".

The `forecast` resource is the predicted output for a series of points
as computed using the `timeseries` model. It expects as input the
`timeseries` object or resource ID and some `input_data`. The `input_data`
format should be a map whose keys are the objective field IDs whose predictions
will be computed. The value of this map will contain:

- `horizon`: an integer setting the total of points that will be computed
- `submodels`: (optional) map that specify the submodels to be used when
               computing the  forecast.

The submodels map structure can have the following keys:
- `indices`: that will contains a list of the indices of the
 `times_series['submodels']` array.
- `names`: that will contain a list of names in the above-mentioned format
- `criterion`: that can be one of the ones available `aic`, `aicc`, and `bic`
  properties in each submodel.
- `limit`: a positive integer which will set the maximum number of results
  returned.

The submodels returned by selecting by `indices` and `names` will be merged,
and if none is present, then all models in the `timeseries` will be used. After
that, the `criterion` will be used to sort them and the `limit` to truncate
the results.

An example of `input_data` would be:

{"000005": {"horizon": 10,
            "submodels":{
                "indices": [2,5],
                "names": ["A,N,N", "M,N,N"],
                "criterion": "aic",
                "limit": 2}}}

Which would select the submodels in position 2 and 5 in the submodels list,
add the ones named "A,N,N" and "M,N,N", sort them by `aic` and return the
point forecast (of 10 new points) for the first two.


*Test samples:*

We use the `data/grades.csv` sample file to build a dataset and use this
dataset to create, update and delete a `timeseries` resource and their related
`forecast` resource.

<a name="localTimeSeries"></a>
Local Time-Series
-----------------

*Affects:* It's a new object that encapsulates the JSON information downloaded
from a remote `timeseries` resource and adds a `forecast` method
to create forecasts locally. Similar to the `Model` object in Python
bindings.

*Description:* The local time-series object will encapsulate the
information found in the `time_series` attribute of the resource
JSON needed to compute the forecasts.

The `timeseries` JSON will contain under the `time_series` attribute
a list of submodels and their characteristic coefficients.

A difference with the rest of supervised learning methods is that a `timeseries`
model can have more than one objective field. They can be specified
using `objective_fields`. If the `all_numeric_objective`
parameter is set to `true`, then all the numeric fields will be considered
as objective fields.

The optional parameters for a `timeseries` include also `error`, `damped_trend`,
`seasonality`, `trend`, `time_range`, `period` (if is set to values > 1,
then seasonality is applied) and `field_parameters`.

The created submodels will be defined by some coefficients. They all have a
level component `l` and a smoothing coefficient `alpha`.
Additive and multiplicative trend models introduce
a trend component `b` and its smoothing coefficient `beta`.
Damped trend models introduce an additional damping coefficient `phi`
Seasonal models will add `m` seasonal components `s_m`,
where `m` is the period length (e.g. 12 for monthly data)
and a single smoothing coefficient gamma.

Counting all parameter combinations, there are 30 different exponential
smoothing types which can be used to model the data, each defining
a unique recursive relationship. Some of them are not explored, though,
for numerical stability reasons. These are: "A,M,\*"  "A,\*,M", "M,M,A" and
"M,Md,A".

The user can either specify a desired
submodel type, or wintermute can simply return the results
from fitting every type.

An example of local time series model is the one in the
[Python bindings](https://github.com/mmerce/python/blob/timeseries/bigml/timeseries.py).

The local `forecasts` will be different from the remote ones, because they
will only contain the `point_forecasts` (the predictions for the new points),
but not the error intervals. Each type of submodel has its own formula to
compute forecasts, however the number of function can be reduced because:

- the error type does not change the computation formula.
- seasonality coefficients are only added or multiplied to the non-seasonal
  formula when required.

This reduces the number of functions to be computed to 5, one per
`trend + damped_trend` combination. An example of the required functions
can be found in the
[tssubmodels.py](https://github.com/mmerce/python/blob/timeseries/bigml/tssubmodels.py)
file in the Python bindings.


*Test samples:*

We can test the local time-series and forecasts using the `data/grades.csv`
sample file. See also the examples of
[tests](https://github.com/mmerce/python/blob/timeseries/bigml/tests/test_35_compare_predictions.py)
in the Python bindings.
