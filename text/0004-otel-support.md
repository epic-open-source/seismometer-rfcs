- Feature Name: otel_support
- Start Date: 2025-06-02
- RFC PR: [seismometer/rfcs#0004](https://github.com/epic-open-source/seismometer-rfcs/pull/4)
- Seismometer Issue: [seismometer/#0000](https://github.com/epic-open-source/seismometer/issues/0000)

# Summary
[summary]: #summary

<!-- One paragraph explanation of the proposed change. -->
Seismometer currently exists as an API generating widgets and plots for use in analyzing model quality. Such an approach, while well-suited
to the user who would like to see at a glance how their model is operating, lacks the customizability to be adapted to custom backends or
processing steps. This feature will add support for emitting metrics in the OpenTelemetry standard, which is not only supported by standard
visualization backends like Prometheus or Jaeger but also allows greater flexibility of data processing and sharing.

# Motivation
[motivation]: #motivation

<!-- Why are we doing this? What use cases does it support? What use cases will it NOT support? What is the expected outcome? -->
Some customers have their own data-processing backends for analyzing the quality of their models. Currently, seismometer is set up in
such a way that the visualization and data generation are fused together. Adding support for emitting OpenTelemetry metrics would keep the
existing notebook and plotting capabilities, but also allow the end user to separate metric generation from visualization. Doing it via
OpenTelemetry in particular will give seismometer the ability to interface with whatever other tools -- of which there are many -- follow
this same open-source standard.

# Detailed design
[design]: #design

<!-- Explain the design in enough detail that somebody familiar with the area would understand it and somebody familiar with the code could implement it.
If this is content to support a new type of evaluation, describe what dataset you will use to support its development. -->

## The Core
At its core, the new proposed telemetry emission would lie in a new `OpenTelemetryRecorder` class, which would provide methods to record
data in the form of OpenTelemetry metrics. In the methods which currently run when new metrics are requested (such as many of the plot
functions in `api/plots.py`), calls to `OpenTelemetryRecorder` instances would be added.

More elaborately, `OpenTelemetryRecorder` would possess the following fields and methods:
- `instruments: dict[str, Gauge]`, where `Gauge` is an OpenTelemetry class for recording metrics. The dictionary would be accessed by name,
so for instance `self.instruments["PPV"]` would be the instrument recording all PPV-related metrics.
- `__init__(self, metric_names, name)`, which would initialize `self.instruments` to have Gauges for every `metric_name`, and also set the
name of the recorder to be `name`. (This `name` would then show up in OpenTelemetry output.)
- `populate_metrics(self, attributes: dict[str, Any], metrics: dict[str, Any])`, which uses `self.instruments` to record the metrics
stored in metrics, with `attributes` storing the parameters of the call, along with other metadata we want to store like the timestamp.
For example, if a score threshold of 0.2 resulted in an accuracy of 0.25 on 70+ patients, we might call
`populate_metrics({"score_threshold":0.2, "Age":"70+"}, {"accuracy":0.25})`, which would then use `self.instruments["accuracy"]` to record
the received data. The values in the `metrics` dictionary must be numerics, lists, or dictionaries. (If string data is desired, a more
appropriate place to put it might be in the attributes section.)

The aim here would be to have each `instrument` recording a particular type of value, so for example all measurements of `accuracy` would
be grouped together as opposed to all sorts of measurements on patients in the age range `[10, 20)`.

## The Output System
While it may be expedient to simply write metrics into files or even to the console, because the entire point of OpenTelemetry support is
to allow integrated use of seismometer with other applications we would have more sophisticated output methods.

Luckily, OpenTelemetry provides such capabilities, by means of "exporters" (which emit telemetry) and "collectors" (which accept it for
later use), which can run over web protocols like HTTP/protobuf. As such, each `MetricGenerator` instance will contain a field of type
`OpenTelemetryRecorder` which will abstract away the necessary OpenTelemetry interfacing. For some of the widgets which do not have state
in their plotting functionality, metrics will be exhausted during the plotting function -- otherwise, when possible, metrics will be
exhausted when calculated.

In order to more cleanly separate logging and plotting, metrics will be exhausted when they are in form ready to be plotted, instead of
immediately when they are generated. Given the structure of the code, this may not be possible to do in a perfectly consistent manner,
but we would aim to exhaust metrics approximately in places corersponding to the `plot_` functions in `api/plots.py`.

## Brief Example
For instance, to log metrics to standard out (as an initial/debug step), the necessary code might look something like this:
```py
# (... in a class constructor of some sort ...)
    self.recorder = otel.OpenTelemetryRecorder(metrics=['specificity', 'sensitivity'])

# (... in some sort of data generation method, whose results get passed to plotting code ...)
    plot_data = get_all_the_data(parameters) # a dictionary or dataframe with specificity/sensitivity as keys or column headers
    self.recorder.exhaust_metrics(attributes=parameters, metrics=plot_data)
```

## Output configuration
To configure which metrics are emitted and how, extra data will be added to each seismograph's `usage_config.yml` file.
(It may also, as configured in `config.yml` under `other_info: export:`, be in any other YAML file.)
There will be a section titled `otel_info`, configured as follows:
```yaml
otel_info:
  metric_type:
    output_metrics: true
    log_all: true
    granularity: 4
    measurement_type: Gauge
  other_metric_type:
    ...
```
For each type of metric (sensitivity, specificity, etc.):
- `output_metrics` will decide whether this metric is dumped.
- `log_all` will indicate whether all information needed to reconstruct the entire graphic or
plot is dumped. For example, in emitting metrics from a widget with an ROC curve, `log_all: true` will emit every datapoint
in the entire ROC curve, while `log_all: false` will only emit the points on the data curve specified by the thresholds inherent
in the widget.
- For metrics representing quantile data, `granularity` specifies how many quantiles for each to output.
- `measurement_type` dictates how individual metric outputs relate to one another. There will be at least three types supported
here: `Gauge` for points of data which have no relation to one another (like for a fairness audit, where it would make no sense
to add together two accuracy scores), `Counter` for points of data which are meant to be summed up, and `Histogram` for data
which is meant to be displayed in a histogram format, such as in several of the widgets.

The defaults for each will be: `output_metrics: true, log_all: false`, `granularity: 4`, `measurement_type: Gauge`.

## Automatic metric exporting

Additionally, for the plotting widgets which select by cohort, the default OpenTelemetry behavior will be to export metrics
as they appear on the plot -- that is, the user will select cohort information and this information will be plotted and exported.
However, the following might appear in a `metric-automation.yml` file:
```yml
plot_function:
  cohorts:
    Age: ["[10, 20)", "70+"]
    Race: ["AfricanAmerican", "Hispanic"]
  options:
    Targets: ["Readmitted within 30 Days"]
    Scores: ["Risk30DayReadmissionPercentile", "RiskLongStay"]
  intersecting: false
other_plot_function:
  # ...
```
This would indicate that `plot_function` should *automatically* export metrics from each of the selected cohorts. The `intersecting`
flag would signal whether to look at combinations of cohorts: in this case the plot would log metrics corresponding to four
groups (both age categories and both race categories), while with `intersecting: true` the widget would log metrics corresponding
to all four possible combinations of age and race from those provided. (Default behavior is that `intersecting` is `false`.)

The `options` section provides information on what other parameters to log on, as applicable by plot.

Underneath a section can be multiple specifications of which sets of metrics to export, if needed.
```yml
plot_function:
  - cohorts:
      Age: ["[10, 20)"]
    options:
      # ...
  - cohorts:
      Age: ["70+"]
    options:
      # possibly something entirely different
```

Without this file, no automatic exporting will occur. With it, a full export of all selected metrics may be performed with
a single core seismometer function call, after loading in the relevant automation file. A command-line interface will be
available to perform a full export with an arbitrary seismogram.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

<!--
- Why is this design the best of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?
-->

In order to disentangle the metric exhaustion from the
metrics being plotted, it makes sense to organize it as described above. In any case, every call which renders a plot should
output metrics exactly once (or never if the corresponding `output_metrics` fields are set to false), which is the only
consistency that should matter for an end user. The other organization is largely for the developers of seismometer.

As for why metrics are grouped by type of measurement instead of by cohort, I believe this fits better with the metaphor of an
instrument, in terms of a physical device whose job it is to measure one sort of thing. However, because individual measurements have
many defining attributes, in principle a user could group them by any one.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

<!--
- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this change?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?
-->

One thing we should consider in the future is how separate we want the visualization and data steps to be from one another. This RFC will
add metric-emitting functionality into the code while keeping current plotting capabilities intact, but it would probably be more elegant
to separate seismometer into two steps: an OpenTelemetry-emitting stage which calculates the metrics, and then an OpenTelemetry-accepting
stage which can create plots or interactive widgets in which to explore the resulting data.

