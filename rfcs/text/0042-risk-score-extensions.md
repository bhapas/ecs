# 0042: Risk field extensions
<!-- Leave this ID at 0000. The ECS team will assign a unique, contiguous RFC number upon merging the initial stage of this RFC. -->

- Stage: **0 (strawperson)** <!-- Update to reflect target stage. See https://elastic.github.io/ecs/stages.html -->
- Date: **2023-07-13** <!-- The ECS team sets this date at merge time. This is the date of the latest stage advancement. -->

<!--
As you work on your RFC, use the "Stage N" comments to guide you in what you should focus on, for the stage you're targeting.
Feel free to remove these comments as you go along.
-->

<!--
Stage 0: Provide a high level summary of the premise of these changes. Briefly describe the nature, purpose, and impact of the changes. ~2-5 sentences.
-->
This RFC seeks to extend the [existing risk fields](https://www.elastic.co/guide/en/ecs/current/ecs-risk.html) [(RFC 0031)](https://github.com/elastic/ecs/pull/2048) to support new/extended Risk Score investigation workflows. The workflows that this RFC intends to enable include all those described in 0031, along with the following:

1. Risk Score Explainability
  * We want to provide more insight into the anatomy of a risk score. The first (and simplest) way we intend to do this is by showing the documents (referred to commonly as Risk Inputs) that contributed to a particular risk score. Given that there may be a large number of these documents, we expect to have to choose a representative subset of these documents to persist along with the score (most obviously: top N riskiest inputs).
  * Since we cannot realistically persist the _entire_ contributing document along with the risk score (let alone several), we intend to persist just enough information to allow one to uniquely identify those documents at a later point in time (i.e. during investigation/analysis of a risk score), along with any information that would not be present on the original document (e.g. the document's calculated risk score).
2. Categorical Risk Scores
  * While the initial iteration of risk scoring ingested Detection Engine Alerts, we intend to expand risk scoring to include more data sources from multiple new categories of data. While we will still present a single risk score for most investigative purposes (composed of all these evaluated data sources), we believe that it will be useful to present individual risk scores _per category_ of data.
  * These categories (and their definitions) are still being discussed [in this internal ticket](https://github.com/elastic/security-team/issues/5485), we currently know that categories will have the following traits:
    * There will be a finite (<10) number of categories
    * These categories' definitions may be _extended_ in the future to include new data sources
  * Due to the above category traits, we need to come up with a naming convention for these categorical score fields that allows them to be extended without invalidating the existing field names.

<!--
Stage 1: If the changes include field additions or modifications, please create a folder titled as the RFC number under rfcs/text/. This will be where proposed schema changes as standalone YAML files or extended example mappings and larger source documents will go as the RFC is iterated upon.
-->

<!--
Stage X: Provide a brief explanation of why the proposal is being marked as abandoned. This is useful context for anyone revisiting this proposal or considering similar changes later on.
-->

## Fields

<!--
Stage 1: Describe at a high level how this change affects fields. Include new or updated yml field definitions for all of the essential fields in this draft. While not exhaustive, the fields documented here should be comprehensive enough to deeply evaluate the technical considerations of this change. The goal here is to validate the technical details for all essential fields and to provide a basis for adding experimental field definitions to the schema. Use GitHub code blocks with yml syntax formatting, and add them to the corresponding RFC folder.
-->

<!--
Stage 2: Add or update all remaining field definitions. The list should now be exhaustive. The goal here is to validate the technical details of all remaining fields and to provide a basis for releasing these field definitions as beta in the schema. Use GitHub code blocks with yml syntax formatting, and add them to the corresponding RFC folder.
-->

## Usage

<!--
Stage 1: Describe at a high-level how these field changes will be used in practice. Real world examples are encouraged. The goal here is to understand how people would leverage these fields to gain insights or solve problems. ~1-3 paragraphs.
-->

## Source data

<!--
Stage 1: Provide a high-level description of example sources of data. This does not yet need to be a concrete example of a source document, but instead can simply describe a potential source (e.g. nginx access log). This will ultimately be fleshed out to include literal source examples in a future stage. The goal here is to identify practical sources for these fields in the real world. ~1-3 sentences or unordered list.
-->

<!--
Stage 2: Included a real world example source document. Ideally this example comes from the source(s) identified in stage 1. If not, it should replace them. The goal here is to validate the utility of these field changes in the context of a real world example. Format with the source name as a ### header and the example document in a GitHub code block with json formatting, or if on the larger side, add them to the corresponding RFC folder.
-->

<!--
Stage 3: Add more real world example source documents so we have at least 2 total, but ideally 3. Format as described in stage 2.
-->

## Scope of impact

<!--
Stage 2: Identifies scope of impact of changes. Are breaking changes required? Should deprecation strategies be adopted? Will significant refactoring be involved? Break the impact down into:
 * Ingestion mechanisms (e.g. beats/logstash)
 * Usage mechanisms (e.g. Kibana applications, detections)
 * ECS project (e.g. docs, tooling)
The goal here is to research and understand the impact of these changes on users in the community and development teams across Elastic. 2-5 sentences each.
-->

## Concerns

<!--
Stage 1: Identify potential concerns, implementation challenges, or complexity. Spend some time on this. Play devil's advocate. Try to identify the sort of non-obvious challenges that tend to surface later. The goal here is to surface risks early, allow everyone the time to work through them, and ultimately document resolution for posterity's sake.
-->

<!--
Stage 2: Document new concerns or resolutions to previously listed concerns. It's not critical that all concerns have resolutions at this point, but it would be helpful if resolutions were taking shape for the most significant concerns.
-->

<!--
Stage 3: Document resolutions for all existing concerns. Any new concerns should be documented along with their resolution. The goal here is to eliminate risk of churn and instability by ensuring all concerns have been addressed.
-->

## People

The following are the people that consulted on the contents of this RFC.

* @rylnd | author
* @SourinPaul | SME / EA product manager

<!--
Who will be or has been consulted on the contents of this RFC? Identify authorship and sponsorship, and optionally identify the nature of involvement of others. Link to GitHub aliases where possible. This list will likely change or grow stage after stage.

e.g.:

* @Yasmina | author
* @Monique | sponsor
* @EunJung | subject matter expert
* @JaneDoe | grammar, spelling, prose
* @Mariana
-->


## References

* [existing risk fields](https://www.elastic.co/guide/en/ecs/current/ecs-risk.html)
* [previous risk fields RFC (stage 3)](https://github.com/elastic/ecs/pull/2048)
* [internal risk categories epic](https://github.com/elastic/security-team/issues/5485)

### RFC Pull Requests

<!-- An RFC should link to the PRs for each of it stage advancements. -->

* Stage 0: https://github.com/elastic/ecs/pull/2232

<!--
* Stage 1: https://github.com/elastic/ecs/pull/NNN
...
-->
