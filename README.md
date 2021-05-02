# Treatment Recomendations from Open Source Clinical Trials

I propose an architecture and a basic implementation for treatment recomendations based on clinical trials data from [ClinicalTrials.gov](https://clinicaltrials.gov). The goal is present treatments for a condition, what symptoms they treat, and how effective they are at treating them.

## The Data

All data comes from the ClinicalTrials.gov website. It is the largest repository of clinical trails in the U.S. and all clinical trials must be posted here by law.

### Structuring a Clincal Trial
The data is a series of clinical trials where each trial is represented as a JSON blob. Working with the data was difficult due to the complexity of a clinical trial. Often times fields were 7 layers into the json strucutre. The best way to work with the data was to create a series of relational tables where each table housed part of clinical trail.

![Clinical Trial Structure](figures/ClinicalTrialStructure.png)

Not all clinical trials were collected. The clinical trials used were intervention trials with target conditions and posted results. This totaled to about 28 thousand studies out of the 300 thousand posted on the website.

**Studies**

- `study_id` A string with the unique NCT medical id of the study
- `name` The the title of the study
- `verified_date` Date study was verified by third party
- `responsible_party` Json string of describing name and type of party responsible for the study
- `conditions` A list of conditions which are the focus of the study
- `type` The type of the study, either interventional, observational or none
- `purpose` Required purpose of study, such as treatment
- `intervention_type` Parallel, crossover, single group, etc.
- `mesh_terms` MEdidical Subject Headers for each one of the studies, represented as a list

**Measures**

- `study_id` A string with the unique NCT medical id of the study
- `measure` The title of the measure
- `type` Primary or Secondary
- `description` Description of the measure as provided by the research group
- `dispersion_param` Parameter detailing the range of the measure. Either NA, 95% Confidence Interval, IQR, SD, or Range
- `measure_param` Type of values the measure is creating, can be mean, median, Least Squares Mean, etc. 
- `units` Units of the measure

**Administrations**

- `study_id` A string with the unique NCT medical id of the study
- `group_id` Id of the group within the study 
- `measure` Title of the measure
- `title` Title of the administration group
- `description` Description of the administration group, typically what treatments are given, how much, and how

**Outcomes**

- `study_id` A string with the unique NCT medical id of the study
- `group_id` Id of the group within the study 
- `measure` Title of the measure
- `title` Title of the outcome or NA if not present
- `value` Value of the outcome in units described by Administrations
- `dispersion` Standard deviation if present
- `upper` upper part of the range if present
- `lower` lower part of the range if present
- `participants` number of participants in the outcome


## Architecture

Assuming data has been parsed as above, the architecture for treatment effectiveness is devided into two steps: preprocessing and runtime.

![TreatmentArchitecture](figures/TreatmentArchitecture.png)

Preprocessing consists of two main steps. The first is creating the statistics for each outcome. The statistics are created by merging the outcomes, administrations, and measures table then creating a p-value from the outcome values, participants, and ranges in the study. Note that some studies have their own analysis section. I opted not to use it since the p-values are often suspect and can't be recreated with the posted data.

The next step is treatment detection. The interventions of each group are mentioned in the administration description and title. We use a version of BERT to preform NER on the interventions which are then passed through a rules based filter. 

### Treatment Detection

One of the core NLP problems with Clinical Trials analysis is treatment detection. Groups in an intervention trial are given one or more treatments to differentiate them. Some treatments are a placebo meant to act as a baseline. The treatments for each group are detailed in the administration description. There is surprisingly no consistent line items for treatments used in a group, so a model is necessary.

I chose an NER + rules based approach. The NER came from [ClinicalBERT](https://github.com/EmilyAlsentzer/clinicalBERT) which is BERT pre-trained on the PubMed corpus. I then trained BERT on the [BC4CHEMD](https://github.com/cambridgeltl/MTL-Bioinformatics-2016/tree/master/data) chemical NER dataset. 

TODO POST RESULTS from TRAINING

Results of the NER were then passed through some rules-based logic. The logic was comparing the treatments found to the names of the groups. If a treatment was present in a group name, then that treatment was removed from the groups that did not have the treatment in the name. This worked well since many of the gorup names follow similar naming conventions. 

TODO GET RESULTS

### Clinical Measures Clustering

A medical condition has many aspects












