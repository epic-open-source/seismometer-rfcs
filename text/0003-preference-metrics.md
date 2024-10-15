- Feature Name: preference-metrics
- Start Date: 2024-10-04
- RFC PR: [seismometer/rfcs#0003](https://github.com/epic-open-source/seismometer-rfcs/pull/0003)
- Seismometer Issue: [seismometer/#0098](https://github.com/epic-open-source/seismometer/issues/98)

# Summary
[summary]: #summary

We will enhance seismometer to better support the commonly used generative AI metrics of binary user feedback and either human or LLM generated Likert Scale preference metrics. To support this, we propose additional configuration for usage config and end user facing parameters for the explore controls. We will also create an ordinal categorical visualization and associated explore controls to display this type of data.

# Motivation
[motivation]: #motivation

We want seismometer to support gen AI metrics and the proposed preference metrics are typically used when there is not ground truth to compare against. This will also address some of the missing features required by [rfc#0001](https://github.com/epic-open-source/seismometer-rfcs/blob/main/text/0001-draft-text-notebook.md).


# Detailed design
[design]: #design

We will add some new data elements to our current usage configuration and data dictionary to add additional alignment with our mental model of decoupling what the data is from how it is used. 
 
For usage config, we will add a metrics section similar to cohorts. It will be used to denote which source features are metrics that need to be ingested into seismometer and what the treatment of the metrics will be. The treatment of the metrics will be called metric_type and we will initially support binary_classification and add support for ordinal_categorical. We will also support a metric_details subkey that will be used to provide metric level details. This could include how to handle N/As or binning strategies. For backwards compatibility reasons, we will use the current logic if the new section is not present. We will also add a group key that will be used to further partition the metrics into distinct sections. This is intended to separate user feedback from evaluated metrics.
 
Usage Config Update Example:
```
metrics:
    - source: LikertA
      display_name: Likert A
      metric_type: ordinal_categorical
      metric_details:
       - combine_na_and_middle_values: true
      group_key: LLM-as-a-judge
    - source: LikertB
      display_name: Likert B
      metric_type: ordinal_categorical
      metric_details:
       - combine_na_and_middle_values: true
      group_key: LLM-as-a-judge
    - source: LikertC
      display_name: Likert C
      metric_type: ordinal_categorical
      metric_details:
       - combine_na_and_middle_values: true
      group_key: LLM-as-a-judge
    - source: LikertD
      display_name: Likert D
      metric_type: ordinal_categorical
      metric_details:
       - combine_na_and_middle_values: true
      group_key: LLM-as-a-judge
    - source: UserFeedback
      display_name: User Feedback
      metric_type: ordinal_categorical
      group_key: User Feedback
``` 

We will also update the data dictionary to contain categorical value display names and assume the order of values is in the proper sequence.
 
The explore controls will be updated to accept an optional group_key that will filter which metrics they will attempt to display.

Notebook Explore Specialization Example:
```
OrdinalCategoricalWidget(group_key="Rubric metrics")
```

We will also create a new set of ordinal categorical visualizations. The [plot-likert](https://github.com/nmalkin/plot-likert) package was used as a proof of concept and while it was fit for this purpose, it was not flexible enough about input data format to fit our needs. Our resultant visualizations will likely look similar to it but better support our themes and integration with the cohorting features. See a mockup in 0003/ordinal_categorical.png

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

We want seismometer to support generative AI type content and given the output of those model can be very disparate, we think adding more structure around metrics is the best approach and will allow composing metrics for models with multiple outputs such as a summary and classification. We also considered alternative approaches to this such as additional configuration files or pushing more into the notebooks, but with the simplicity and flexibility of the proposed approach, they were put aside.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this change?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

