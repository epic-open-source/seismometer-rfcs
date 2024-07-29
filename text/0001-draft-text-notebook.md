- Feature Name: draft_text_notebook
- Start Date: 2024-06-05
- RFC PR: [seismometer/rfcs#0001](https://github.com/epic-open-source/seismometer-rfcs/pull/1)
- Seismometer Issue: [seismometer/#26](https://github.com/epic-open-source/seismometer/issues/26)

# Summary
[summary]: #summary

A growing generative AI use case is related to drafting text for a clinician to review and edit prior to saving to the patient's record. [Patient Medical Advice Requests](https://www.epicshare.org/share-and-learn/mayo-ai-message-responses) is a well publicized workflow but there is also past research about other workflows including discharge summaries and letter generation. This RFC will cover developing an initial set of metrics including operational ones like usage and edit distance, qualitative ones like thumbs up/down, and fairness related ones like reading level and integrating this model type with existing Seismometer analyses.

# Motivation
[motivation]: #motivation

* Why are we doing this? What use cases does it support? What use cases will it NOT support? What is the expected outcome? 

We will create a template in Seismometer for draft text generation models. This includes an example dataset, a notebook template, support for calculating the relevant metrics, and any net new visualizations required. We expect that this will support use cases like patient questions and note drafting. We will not calculate complex metrics like Bertscore or integrate with NLP engines here as we expect upstream data sources to provide this data.

# Detailed design
[design]: #design

Explain the design in enough detail that somebody familiar with the area would understand it and somebody familiar with the code could implement it.
If this is content to support a new type of evaluation, describe what dataset you will use to support it's development.

## Data set
We will start with a dataset of 1000 human generated messages and augment it with additional information. We will upload the initial 1000 message dataset and a script to generate the rest of the fields to the seismometer data repo.
The full dataset is listed below along with how the dataset will be composed.
* Message infomation
    * sent text - in dataset already
    * category - in dataset already
    * language - label with API; [ISO 693-1 codes](https://en.wikipedia.org/wiki/List_of_ISO_639_language_codes)
* Sender attributes
  * race - randomly selected
  * sex - randomly selected
  * preferred language - align with message language most of the time; otherwise select from [ISO 693-1 codes](https://en.wikipedia.org/wiki/List_of_ISO_639_language_codes)
* Replier attributes
    * role - randomly selected from doctor, nurse, administrative
    * work location - randomly selected between City Clinic, City Hospital, Rural Clinic, Rural Hospital
    * languages understood - align with message language most of the time; otherwise select from [ISO 693-1 codes](https://en.wikipedia.org/wiki/List_of_ISO_639_language_codes)
* Reply information
    * reply text - human and LLM generated
    * suggested text - human and LLM generated
    * suggested text used - randomized and will inform both the reply and suggested text
    * reply feedback - thumbs up/down randomly selected based on suggestion used
    * workflow time - define an algorithm that approximates number of minutes based on message and reply text lengths
    * reply had follow-up - Boolean to represent if the reply had follow-up messages from original sender

## Metrics
We will calculate the following metrics in Seismometer when loading the dataset.
* Suggested reply and actual reply edit distance using several methods bucketed into thresholds
    * [Levenshtein](https://en.wikipedia.org/wiki/Levenshtein_distance) word level
    * [Jaccard](https://en.wikipedia.org/wiki/Jaccard_index) unique words
* Similarity based on edit distance between draft and actual
* Message reading level using [Flesch-Kincaid](https://en.wikipedia.org/wiki/Flesch%E2%80%93Kincaid_readability_tests)
* Message language differs from sender preferred language
* Message language not in replier languages understood

We will NOT calculate any clinical similarity metrics like [CBertscore](https://arxiv.org/abs/2303.05737) or [BioLORD](https://academic.oup.com/jamia/advance-article/doi/10.1093/jamia/ocae029/7614965) as they are challenging to calculate and better suited to being calculated in the dataset upstream from Seismometer. 

## Analyses
We intend to use the pattern defined by the binary classifier as inspiration for this content. The primary analyses will use edit distance as their key metric as low edit distances may indicate automation bias risks while high edit distances may indicate poorly performing draft text models. Additionally, analyzing edit distances between sender and replier attributes may indicate areas that models do not perform well or automation bias when replying to certain sender groups.

The secondary analyses will use reading level and language similarities with similar goals.

## Changes to Seismometer Dependencies
We will defer this to implementation and do follow-up RFCs for any net new direct dependencies. We have already evaluated packages like [nltk](https://www.nltk.org/), [spaCY](https://spacy.io/), and PyPI for useful metrics and determined that the cost of their inclusion does not outweigh the additional challenges they will incur on packaging as the metrics we intend to use are trivial to implement in Python and do not require data dictionaries.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?

This design is the best possible design as it incorporates common analyses used by a number of health systems to evaluate their draft text model performance.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this change?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

We expect the RFC process will resolve the specifics about the dataset we will use for development and the metrics that will be calculated based on it.

The specific design of the analyses will be resolved in the implementation of this RFC and may require also follow the RFC process.
