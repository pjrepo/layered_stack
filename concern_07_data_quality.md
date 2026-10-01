# Data quality: what an application-layer AI engineer needs to know

This file covers the Data quality concern from `ai_stack.md`. Data quality keeps the information moving through the system trustworthy. It covers accuracy, freshness, lineage (where data came from) and how data changes over time. It applies equally to training data, retrieved documents, tool outputs and evaluation sets. It is one of the seven cross-cutting concerns: not a layer, but something every layer needs. This file is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this concern and how deeply to learn each one.

## The rule for depth

A model doesn't notice when its inputs are wrong. Give it an outdated policy, a duplicated record or a tool result with a missing field, and it writes a fluent, confident answer from them. Many failures that look like model failures are data failures (see `layer_04_context_memory_and_retrieval.md`). AI systems also write data: extracted fields, summaries, tags, memories, labels and synthetic examples flow back into databases, indexes and training sets, where a mistake spreads and starts to look like fact.

So data quality runs in both directions, for every kind of data in the system: the training data and evaluation sets in `layer_10_data.md`, the documents and memories in `layer_04_context_memory_and_retrieval.md`, tool results and other structured data, and everything the model writes.

For every AI feature, data quality answers four questions:

1. Can you trust what the model reads? (Foundations, Data the model reads)
2. Can you trust what the model writes? (Data the model writes)
3. Where did each piece of data come from, and what did it affect? (Lineage and impact)
4. Is it still true, and how would you know? (Change over time, Checking and fixing data)

Most concepts in this concern are level 3, and two things are level 4:
- marking and checking everything the model writes into your systems, so its mistakes can be found and fixed later
- tracing any answer back to its sources, and any source forward to everything built from it

Data engineering, such as building pipelines, warehouses and data catalogs, is a discipline of its own, and data teams often own the sources you depend on. Learn their tools to level 2, or to level 3 where your features run their own pipelines, and work with the owners of your sources rather than around them.

The layer files cover data quality as it applies to each layer, mainly retrieved documents and memories in Layer 4 and training data and evaluation sets in Layer 10. This file covers what they share, and Data quality at each layer maps where the rest is covered.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Foundations | 3 | You rarely own the data your features depend on |
| Data the model reads | 3 | Models use bad input without noticing |
| Data the model writes | 3 (some topics 4) | Model mistakes written into your systems start to look like facts |
| Lineage and impact | 3 (some topics 4) | You must be able to trace any answer to its sources, and back |
| Change over time | 3 | Data stops being true without raising an error |
| Checking and fixing data | 3 | Checks help only if problems get fixed where the data lives |
| Security | 3 | Some bad data is planted on purpose |
| Governance and safety | 3 | Wrong data about people harms them |

## Foundations [3]

These ideas come up in every other section, so learn them first.

- [3] The parts of quality. Data quality has several parts: accuracy, completeness, consistency, freshness, uniqueness, validity, and whether the data looks like real use (see `layer_10_data.md`). Apply them to every kind of data in the system, not only to datasets: documents, database records, tool results, memories and what the model writes.
- [3] Every source, listed. List every source of data that reaches the model or feeds your datasets: document collections, databases, tool APIs, memories, uploads, feedback, synthetic data and other agents' outputs. For each, note its owner, how often it changes and how far you trust it.
- [3] Agreements with source owners. You rarely own the data your features depend on. Agree with each source's owner on what the data will look like, how fresh it will be and how they'll warn you before changing it. Agreements like these are often called data contracts.
- [2] Quality has a cost. Perfect data is impossible, and chasing it is expensive. Decide how good each source must be for the features that use it, and spend where errors do the most harm.

Check: List every source of data for one feature, with its owner, how often it changes and how far you trust it. Take a sample of 50 records from the least trusted source, check them against the parts of quality, and report what you find.

## Data the model reads [3]

Retrieved documents, memories and the context as a whole are covered in `layer_04_context_memory_and_retrieval.md`, and checking tool arguments and results in `layer_03_tools_and_protocols.md`. This section covers what all inputs share, especially structured data such as tool results and database records.

- [3] Checks at every boundary. Before data enters the context, check tool results, API responses, database records and other agents' outputs against a schema and sensible ranges (see `layer_02_agents_and_orchestration.md`). Watch for failures that look like answers, such as a tool that returns an empty list when it fails, which the model reads as "nothing found".
- [3] Dates on everything. Give every fact in the context its date, or the time it was true as of, so the model and the user can tell how current it is. Decide how fresh each source must be for each feature, and check it before use.
- [3] Units, codes and blanks. Models misread structured data that a person would question: amounts without currencies, times without time zones, internal codes without meanings, and blanks that could mean zero, unknown or not applicable. Label units, explain codes and make missing values explicit before the data reaches the model.
- [3] One source of truth for each fact. When systems disagree, such as a billing system and a CRM that list different plans for the same customer, decide which system is authoritative for each kind of fact, and use that one. Documents that conflict are covered in `layer_04_context_memory_and_retrieval.md`.
- [3] Inputs from users. Uploaded files can be corrupted, badly scanned, password-protected or in an unexpected language or format. Detect this and tell the user, rather than answering from whatever text came out (see parsing in `layer_04_context_memory_and_retrieval.md`).
- [2] Definitions for data the model queries. When a model writes queries against your data, it needs to know what each column and metric means, such as what counts as an active customer. A data dictionary, or a semantic layer that defines metrics in one place, cuts down on queries that run but answer the wrong question (see `layer_04_context_memory_and_retrieval.md`).

Check: For one feature, list every kind of structured data that reaches the model, and add schema checks, dates, units and explicit missing values to each. Then show one case where a check caught bad data the model would otherwise have used.

## Data the model writes [3, some topics 4]

AI features increasingly write data, not just text for a person to read: fields extracted into a CRM, tags on tickets, summaries stored with records, memories, labels and synthetic examples. Memories the model writes are covered in `layer_04_context_memory_and_retrieval.md`, and synthetic data and labels made by models in `layer_10_data.md`. This section covers what they share.

- [4] Marking what the model wrote. Store every value the model wrote with a mark that says a model produced it, which model and prompt version, from which inputs and with what confidence, kept apart from values people entered or confirmed. Without the mark, the model's mistakes become indistinguishable from facts, and nobody can find them later to check or fix them.
- [3] Checks before writing. Before generated values are written, check them against schemas, allowed values and rules that span fields, such as an end date that must follow the start date. Send values that fail, or that the model was unsure of, to a person for review (see `layer_02_agents_and_orchestration.md`).
- [3] Mistakes that spread. A wrong extracted value feeds reports, other features, search indexes and future training data. Know where each kind of generated data flows, so that when a prompt or model turns out to be wrong, you can find every copy (see Lineage and impact).
- [3] Model output isn't ground truth. Don't put model outputs into evaluation sets or training data as correct answers without review (see `layer_10_data.md`). Mark documents and summaries that AI wrote in your own sources, so that retrieval and training can tell them apart from material people wrote.
- [2] Fixing past outputs. When you fix a prompt or upgrade a model, decide whether to regenerate past outputs. Regenerating a large backlog costs money and can overwrite corrections people made, so regenerate only values nobody has confirmed, in batches you can roll back.

Check: For one feature that writes data, mark every generated value with the model, prompt version and inputs, and add checks before writing. Then simulate a bad prompt release: find every value it wrote and correct them, and show that values people had confirmed were left alone.

## Lineage and impact [3, some topics 4]

Lineage records where data came from and what was built from it. Lineage for training and evaluation data is covered in `layer_10_data.md`, lineage for the chunks in an index in `layer_04_context_memory_and_retrieval.md`, versions and IDs on every trace in `concern_02_observability.md`, and records of where models, datasets and tools came from in `concern_04_governance_and_safety.md`. This section covers connecting them.

- [4] From answer to source, and back. Link each answer to the context it used, each piece of context to the document or record it came from and that source's version, and each dataset example to the trace and source behind it. Then you can answer two questions quickly: where did this wrong answer's information come from, and which answers, datasets and generated values used this source after it went wrong?
- [3] When a source turns out to be bad. A document may prove outdated, a dataset unlicensed, a customer's data due for deletion, or a tool's results wrong for a week. Use lineage to find everything built from it, such as index entries, memories, summaries, evaluation cases, training examples and fine-tuned models, and fix or remove each one (see `concern_04_governance_and_safety.md`).
- [3] Lineage you can afford. Full lineage for every value is expensive. Record IDs and versions at each step, as traces already do (see `concern_02_observability.md`), rather than copies of the data, and keep more detail for data that feeds training, decisions about people or regulated features.
- [2] Lineage tools. Data teams track lineage between tables and pipelines with data catalogs and with standards such as OpenLineage. Find out whether your company has one, and connect your AI pipelines to it where you can.

Check: Take one wrong answer from production and trace it back to the source record and version it came from. Then go the other way: from that source, find every answer, index entry, memory and dataset example that used it after it went wrong.

## Change over time [3]

Data changes after you check it. Freshness and keeping indexes in sync are covered in `layer_04_context_memory_and_retrieval.md`, and changes in what users ask and datasets going stale in `layer_10_data.md`. This section covers noticing change in every source.

- [3] Kinds of change. Content changes, such as an updated price list. Structure changes, such as a renamed field or a new code value. Meaning changes, such as a metric whose definition was revised. And the world changes, such as a new law or product. Each needs its own way of being noticed.
- [3] Watching sources. Monitor each important source for how recently it updated, how much data arrived, whether its structure changed and whether its values shifted. Alert when a source stops updating or its volume drops suddenly, because a broken feed often looks like a quiet day.
- [3] Changes upstream. A renamed field or a new category in a source system can break extraction or retrieval without raising any error. Checks at every boundary catch some of these (see Data the model reads), and agreements with source owners catch more before they ship (see Foundations).
- [3] When facts were true. For facts that change, such as prices, policies and people's roles, record when each was true, not just when it was stored. Then questions about the past get the facts that held at the time, and current answers get the latest.

Check: Add monitoring for freshness, volume and structure to the three sources one feature depends on most. Break one in a test environment, such as by stopping its updates or renaming a field, and show that the monitoring catches it before users would notice.

## Checking and fixing data [3]

Validating datasets, finding wrong labels and documenting datasets are covered in `layer_10_data.md`, and inspecting parsed documents in `layer_04_context_memory_and_retrieval.md`. This section covers keeping checks running and getting problems fixed.

- [3] Checks in every pipeline. Run automated checks wherever data is loaded or transformed: schemas, ranges, missing values, duplicates, links between records and freshness. Tools such as Great Expectations, Soda, dbt tests and pandera do this. When a batch fails, hold it back for review rather than silently dropping it or loading it anyway.
- [3] Reading samples. Automated checks catch broken structure, not wrong meaning. Read samples of each source regularly, such as parsed documents, extracted fields and labels, and look for what the checks can't see.
- [3] Models as checkers. A model can flag records that look contradictory, outdated, duplicated or off-topic, at a scale people can't read. Check its flags against human judgment before relying on them, as you would any judge (see `concern_01_evaluation.md`).
- [3] Fixing where the data lives. When data is wrong, get it fixed at the source, by its owner, rather than patching around it in prompts. Workarounds in prompts pile up, hide the problem from everyone else who uses the data, and break when the data changes. Track data problems like bugs, with owners and dates.
- [2] Quality signals the system can use. Store signals such as how authoritative a source is, how fresh a record is and whether a person confirmed a value, as metadata. Retrieval, ranking and the model itself can then favor better data (see `layer_04_context_memory_and_retrieval.md`).

Check: Add automated checks to one pipeline that feeds an AI feature, and hold back batches that fail. Run a model-based check over one source, compare its flags with 50 labels you made yourself, and get one real problem fixed by the source's owner.

## Data quality at each layer

Four layer files cover data quality as it applies to that layer. This table shows where.

| Layer | What to keep trustworthy there | Level | Where |
|---|---|---|---|
| 2. Agents and orchestration | Checks between steps, so one step's bad output doesn't spread to the next | 3 | `layer_02_agents_and_orchestration.md` |
| 3. Tools and protocols | Checking tool arguments and results, and dating results where freshness matters | 3 | `layer_03_tools_and_protocols.md` |
| 4. Context, memory and retrieval | Freshness, duplicates and versions, source quality, lineage for chunks, gaps, parsing, memories that go wrong | 3 | `layer_04_context_memory_and_retrieval.md` |
| 10. Data | The parts of quality, validation, lineage for examples, data cards, decontamination, staleness, wrong labels, drift | 3 | `layer_10_data.md` |

Layers 1, 7 and 8 point to `layer_10_data.md` for their data quality, and Layers 5, 6 and 9 have nothing specific to it.

Check: For one feature, list every layer it touches and the data each layer passes on. Mark which data is checked where it enters today, and add a check at the most important gap.

## Other cross-cutting concerns in data quality

### Security [3]

- [3] Bad data on purpose. Some bad data is planted: poisoned documents, memories, feedback and training examples. Quality checks catch some of it, but treat any source that outsiders can write to as untrusted (see `concern_03_security.md`).

### Governance and safety [3]

- [3] Wrong data about people. A wrong address, a mistaken flag or a confused identity harms the person it's about, and privacy laws give people the right to have it corrected (see `concern_04_governance_and_safety.md`). Make each correction reach every copy, including values the model wrote and memories.

The other concerns each meet data quality in one place: wrong labels in evaluation sets in `concern_01_evaluation.md`, complete traces in `concern_02_observability.md`, sources that stop updating without an error in `concern_05_reliability.md`, and what duplicate and outdated documents cost in `concern_06_cost_and_latency.md`.

## Not in this concern

- Training data, evaluation sets, labeling and their quality are covered in `layer_10_data.md`.
- Documents, indexes and memories, including their freshness, duplicates and lineage, are covered in `layer_04_context_memory_and_retrieval.md`.
- Checking tool arguments and results is covered in `layer_03_tools_and_protocols.md`.
- Recording versions and IDs on every trace is covered in `concern_02_observability.md`, and records of where models and datasets came from in `concern_04_governance_and_safety.md`.
- Poisoned data and memories are covered in `concern_03_security.md`, `layer_04_context_memory_and_retrieval.md` and `layer_10_data.md`.
- Building data pipelines, warehouses and catalogs is data engineering, a discipline of its own. This file covers what changes when AI reads and writes the data.

## Learning order

1. Foundations.
2. Data the model reads.
3. Data the model writes, before any feature writes into a system other people rely on.
4. Change over time.
5. Checking and fixing data.
6. Lineage and impact.

Learn the data quality parts of `layer_04_context_memory_and_retrieval.md` and `layer_10_data.md` alongside this file, since most of the detail is there.

## Key reading

- Sambasivan et al., "Everyone wants to do the model work, not the data work: Data Cascades in High-Stakes AI" (2021), on how data problems compound.
- Barr Moses, Lior Gavish and Molly Vorwerck, *Data Quality Fundamentals* (O'Reilly, 2022).
- Breck et al., "Data Validation for Machine Learning" (Google, 2019).
- Gebru et al., "Datasheets for Datasets" (2021).
- Northcutt, Athalye and Mueller, "Pervasive Label Errors in Test Sets Destabilize Machine Learning Benchmarks" (2021).
- Chip Huyen, *Designing Machine Learning Systems* (O'Reilly, 2022). Chapter 8 covers changes in data over time and how to monitor them.
- Shumailov et al., "AI models collapse when trained on recursively generated data" (*Nature*, 2024), on why model output shouldn't be treated as ground truth.
- The OpenLineage documentation (openlineage.io), on recording lineage across pipelines.
- The documentation for one data validation tool, such as Great Expectations, Soda or pandera.
