- Feature Name: replace_aequitas_with_fairness_table
- Start Date: 2024-06-05
- RFC PR: [seismometer/rfcs#0002](https://github.com/epic-open-source/seismometer-rfcs/pull/2)
- Seismometer Issue: [seismometer/#26](https://github.com/epic-open-source/seismometer/issues/26)

# Summary
[summary]: #summary

[Aequitas](https://github.com/dssg/aequitas) uses altair plots to display information, and when hosted in a notebook this causes a number of awkward issues to code around (loading in a frame, loading in vscode, etc). In addition, when there are many different cohort splits that need to be evaluated, the horizontal nature of the table leads to a difficult to read table. 

# Motivation
[motivation]: #motivation

* Why are we doing this? What use cases does it support? What use cases will it NOT support? What is the expected outcome? 

Working with Aequitas's current fairness audit bring in additional complexity and UX concerns. We would like to reduce code complexity, while allowing an extension to non-binary classifier situations. 

# Detailed design
[design]: #design

* Explain the design in enough detail that somebody familiar with the area would understand it and somebody familiar with the code could implement it.

Rather than leveraging [aequitas](https://github.com/dssg/aequitas), we will use [great_tables](https://github.com/posit-dev/great-tables) to display a list of computed metrics, as well as the percentage disparity between the a default cohort (the largest subgroup for each chort category) and every other cohort value (excepting any cohort that falls below the censoring threshold).

An example table for custom metrics would look like

![Fairness table for GenAI use case](fairness_genai.png)

We will have out of the box support for Binary Classifiers, including common existing metrics (false positive rate, accuracy, ppv, etc), that are computed based on a model threshold. As a result we can drop support for aequitas while maintaining the same functionality.

![Fairness Options for Binary Classifier](fairness_header_binary_classifier.png)

The output table will be consistent across any model/model metrics, giving a common framework for looking at results at a glance.  There will be a legend to inform people of the iconography used.

![Fairness Table for Binary Classier](fairness_table_binary_classifier.png)


## Custom Metric API

We will also support the creation of custom metrics that are not threshold dependant, these can be fed into a new Exploration* control to allow for exploring those metrics from a fairness perspective. 

```python
import pandas as pd
from seismometer.data.performance import MetricGenerator
from random import random

def create_random_realistic_metrics(data: pd.DataFrame, **kwargs):
    """
    Fake metrics, so ignoring the input data
    """
    return {"Readability": random(),
            "Message Length": 100 + 50*random(),
            "Reading Level": 7 + 2*random(),
           }

fake_metrics = MetricGenerator(["Readability", "Message Length", "Reading Level",] create_random_realistic_metrics)
```

The `MetricGenerator` class is just a wrapper that defines a list of metrics, and the callable that will generate those metrics from an input dataframe (and any additional required kwargs). The kwargs allows passing in additional data like model thresholds or score/target descriptors which may be appropriate for some models (binary classifiers), but not others. 

When included as part of an Explore control, the controls header will looke like this:

![Explore control header for non-threshold metrics](fairness_header_non_binary_classifier.png)

## Binary Classifier Metrics

For binary classifiers, we already calculate many of the base metrics that we want to compare:

```["TP", "FP", "TN", "FN", "Accuracy", "Sensitivity","Specificity", "PPV", "NPV", "Flagged", "LR+", "NetBenefitScore","NNT@0.33"
]```

Our current implmentation supports the following from `aequitas` - [fairness metrics](https://github.com/dssg/aequitas?tab=readme-ov-file#fairness-metrics): 

```python
["tpr", "tnr", "for", "fdr", "fpr", "fnr", "npv", "ppr", "precision", "pprev"]
```

Many of these defintiions are calculated at the group level, we we will extend `seismomemter.data.performance.calculate_bin_stats` to indclude these combinations. 


## Changes to Seismometer Dependencies
We will remove our current dependency on [aequitas](https://github.com/dssg/aequitas) and add a dependency to [great_tables](https://github.com/posit-dev/great-tables). The new dependency is for ease of styling the new fairness table. This also lets us take advantage of updates to `great_tables` for some common styling options in the future (collapsible row groups for example).

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?

This design is the best possible design as it allows us to continue to support fairness for binary classifiers, as well as allowing us to talk about fairness in terms of model specific metrics (like GenAI use cases)

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this change?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

We expect the RFC process will resolve the specifics about the user interface design and how we can handle custom metrics by use case. 