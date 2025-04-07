# ASSIGNMENT: Sampling and Reproducibility in Python

Read the blog post [Contact tracing can give a biased sample of COVID-19 cases](https://andrewwhitby.com/2020/11/24/contact-tracing-biased/) by Andrew Whitby to understand the context and motivation behind the simulation model we will be examining.

Examine the code in `whitby_covid_tracing.py`. Identify all stages at which sampling is occurring in the model. Describe in words the sampling procedure, referencing the functions used, sample size, sampling frame, any underlying distributions involved, and how these relate to the procedure outlined in the blog post.

Run the Python script file called whitby_covid_tracing.py as is and compare the results to the graphs in the original blog post. Does this code appear to reproduce the graphs from the original blog post?

Modify the number of repetitions in the simulation to 100 (from the original 1000). Run the script multiple times and observe the outputted graphs. Comment on the reproducibility of the results.

Alter the code so that it is reproducible. Describe the changes you made to the code and how they affected the reproducibility of the script file. The output does not need to match Whitby’s original blogpost/graphs, it just needs to produce the same output when run multiple times

# Author: ELLEN NIKELSKI


**Examine the code in `whitby_covid_tracing.py`. Identify all stages at which sampling is occurring in the model.**

Sampling occurs in two steps in this model. Sampling first occurs during primary contact tracing where individuals infected with covid-19 are contact traced to determine where they contracted the virus. Here, our sample consists of the 20% of infected individuals for which contact tracing was successful out of all infected individuals. The second case of sampling occurs during secondary contact tracing where events that have at least two covid-19 cases traced back to them receive heavy contact tracing resulting in all infected individuals being identified. Here, we designate a sample of “high infection events” out of all events where an infected individual was contact traced to and focus contract tracing efforts at those events. Based on these two steps, our sample for calculating the proportion of individuals who contracted covid-19 from weddings consists of infected individuals that were successfully contact traced during primary contact tracing and infected individuals identified from “high infection events” that received secondary contact tracing and that weren’t identified during primary contact tracing (i.e. no double counting of individuals). 

**Describe in words the sampling procedure, referencing the functions used, sample size, sampling frame, any underlying distributions involved, and how these relate to the procedure outlined in the blog post.**

In this model, the sampling frame is all people infected with covid-19 in the community while the sample that we use to estimate the proportion of infections attributable to weddings is infected individuals that have been successfully contact traced. The sampling procedure includes two steps related to primary and secondary contact tracing. The primary contract tracing step involves contact tracing all infected individuals in the community which is about 100 individuals based on a 10% chance of infection (ATTACK _RATE) for a community of 1000 individuals. Contact tracing is only successful for 20% of infected individuals or about 20 out of the 100 infected individuals which would be the sample size at this point. This step is completed with the following line of code in the model:

`ppl.loc[ppl['infected'], 'traced'] = np.random.rand(sum(ppl['infected'])) < TRACE_SUCCESS`

where each infected individual is given a random value between 0 and 1 representing the probability that contact tracing is successful and then individuals with probability values less than the “TRACE_SUCCESS” threshold of 0.2 are recorded as being successfully traced (i.e. they receive “TRUE” in the traced column). Secondary contact tracing requires the identification of events that have at least two individuals contact traced to them which is done with the following two lines of code:

`event_trace_counts = ppl[ppl['traced'] == True]['event'].value_counts()`

`events_traced = event_trace_counts[event_trace_counts >= SECONDARY_TRACE_THRESHOLD].index`

Here, the first line of code identifies how many individuals were traced to each event and the second line of code pulls out the events that require secondary tracing based on a “SECONDARY_TRACE_THRESHOLD” of 2. Following this, all infected individuals that attended the secondarily traced events are attributed to those events and designated as being successfully traced if they weren’t previously traced during primary contact tracing. This is done with the following line of code:

`ppl.loc[ppl['event'].isin(events_traced) & ppl['infected'], 'traced'] = True`
  
where individuals recorded as attending one of the secondarily traced events are given a “TRUE” value in the “traced column”. Performing secondary contact tracing increases our sample size so that it now includes both infected individuals successfully traced during primary contact tracing and infected individuals associated with secondarily traced events who were not identified during primary contact tracing. This sample size would likely still be less than the total number of infected individuals in the population and the sample itself would be biased towards larger events where more people get infected simply because more people were in attendance even though the probability of infection does not change between large and small events. Overall, this sampling procedure could be described as “non-probability survey sampling” since all infected individuals do not have the same chance of successful contact tracing and being included in the sample. More specifically, this sampling procedure could be described as “judgement or purposive sampling” as the surveyor is judging individuals from events where higher numbers of infected individuals were detected (larger events) as being more representative of the population. It could also potentially be described as “convenience sampling” as the surveyor is trying to maximize the number of infected individuals that could be included in the sample for the least cost and therefore targets larger events. This script is consistent with the procedures outlined by Whitby as it maintains the population size (1000), probability of infection (0.1) and probability of successful contact tracing (0.2) in his model and also also captures the characteristics of primary and secondary contact tracing that he described. However, it also greatly differs from Whitby’s original procedures in the fact that it collapses “wedding” and “brunch” into single events rather than into two and eighty events respectively. This greatly impacts the results from the simulations of the model and removes the skew in the sampling distributions of the real and estimated proportion of covid-19 cases attributed to weddings that Whitby describes in his blog post. I describe this in more detail below.

**Run the Python script file called whitby_covid_tracing.py as is and compare the results to the graphs in the original blog post. Does this code appear to reproduce the graphs from the original blog post?**

No, the script does not reproduce the graphs in the original blog post. In fact, the graph I produced did not show anything close to the difference in the sampling distributions of the real and estimated proportion of covid-19 cases attributed to weddings that Whitby reported. Upon further inspection, I realized that the code provided to the DSI cohort has been simplified from what Whitby originally wrote such that “wedding” and “brunch” are represented as single events rather than multiple. As a result of brunch containing 800 participants, it usually passes the secondary tracing threshold meaning that, on most occasions, ALL infected individuals in the community are successfully contact traced and included in the sample. In this situation the proportion calculated from the sample will be basically the same as the true proportion resulting in the real and estimated sampling distributions produced by simulations appearing extremely similar with basically the same means.

**Modify the number of repetitions in the simulation to 100 (from the original 1000). Run the script multiple times and observe the outputted graphs. Comment on the reproducibility of the results.**

As written, the script does not generate reproducible results. In other words, every time I run the script, it produces a slightly different set of sampling distributions due to the stochasticity associated with some of functions included in the code (e.g. `np.random.rand()`).

**Alter the code so that it is reproducible. Describe the changes you made to the code and how they affected the reproducibility of the script file.**

To make this script and its associated results reproducible, I included a function at the beginning of the file that set a random seed (`np.random.seed(1)`).  Including this line of code ensures that any functions requiring random number generation return the same set of numbers across runs. In this way, I can ensure that running this script always produces the same sampling distributions (provided that the same seed number is used) and that the results are reproducible.


## Criteria

|Criteria|Complete|Incomplete|
|--------|----|----|
|Altercation of the code|The code changes made, made it reproducible.|The code is still not reproducible.|
|Description of changes|The author explained the reasonings for the changes made well.|The author did not explain the reasonings for the changes made well.|

## Submission Information

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

### Submission Parameters:
* Submission Due Date: `23:59 - 09/04/2025`
* The branch name for your repo should be: `assignment-1`
* What to submit for this assignment:
    * This markdown file (a1_sampling_and_reproducibility.md) should be populated.
    * The `whitby_covid_tracing.py` should be changed.
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sampling/pull/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `assignment-1`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via the help channel in Slack. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.
