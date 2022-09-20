---
description: >-
  In this page we describe the importance of having different domains and how
  they relate to each other.
---

# Domains

* Why domains exist - what kind of domains exist - how many - how are they represented (some kind of a tree, graph?)
* How are domains aggregated
* Special domains for managers
* Remember how it would differ with addition of domains - we could decide the order of managers to query based on their reputation in the previous epoch in some domain related to management. Also explain that if manager manages 10 peers there have to be 10 different domains, one for each peer. In each domain managers will have 256 neighbours, and there will be just 1 valid manager for each of these neighbours that we will interact with. We will give the reputation equally to each one of them. The invalid managers will not receive any reputation.
* Explain how do we organise signatures
* How do we query each domain - does with come within one signature or there is a separate signature for each domain, if so, how do we find them

### Why domains exist?

Encapsulating someones reputation from all field into one number would be incorrect. If you earn your reputation as a software developer, that reputation should not be used in a context of community building. Each of these should be seen as a separate domains, and they should be completely separated from the others, but at the same time be part of the same more broader domain.

### Domain aggregation

TBA

### Special domain for managers

Managers should earn reputation is specially reserved domain. This domain should be direct child of the root domain.
