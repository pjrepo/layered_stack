# Layer 10, Data: what an application-layer AI engineer needs to know

This file covers the Data layer from `ai_stack.md`. It is for an AI engineer who builds products on existing models (layers 1 to 5) and doesn't pre-train them. It lists the concepts in this layer and how deeply to learn each one.

## The rule for depth

Learn the data you create and own in depth. For data that other people create for you, a lighter understanding is enough. Model labs build pre-training corpora, so you only need to know what goes into them. Your users' feedback comes only from your product, so you should be the expert on it.

Most of your data work involves small datasets that you build carefully: evaluation sets, fine-tuning sets and feedback from your product. You build all of them with the same five skills:

1. Reading real outputs.
2. Writing labeling rules.
3. Labeling.
4. Generating synthetic examples.
5. Writing graders.

Once you learn these skills, you use them in every section below.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Foundations | 3 | Every other section uses them |
| Pre-training corpora | 1 (some topics 2) | Labs build them; you only need them to explain model behavior |
| Fine-tuning and preference datasets | 3 | When you fine-tune, the dataset is what you control |
| Human labeling | 3 (error analysis 4) | You and your domain experts produce the correct answers |
| Synthetic data | 3 | Fills gaps before you have users, and covers rare or sensitive cases |
| RL environments | 2 (graders 3) | Mostly a lab tool, but the graders double as your eval graders |
| Product feedback | 4 | Only your product has this data |
| Evaluation datasets | 4 | The dataset you'll build and rely on most |
| Data quality | 3 | Bad data quietly breaks evaluations and fine-tunes |
| Governance and safety | 2–3 | Decides which data you may use at all |
| Security | 2 | Poisoned or leaked data is rare but costly |
| Cost | 2–3 | Labels and generated data cost real money |

## Foundations [3]

These basics come up in every other section, so learn them first.

- [3] File formats. JSONL, with one JSON example per line, is the standard format for evaluation and fine-tuning data. Learn the chat message format, which has system, user, assistant and tool roles and includes tool calls. Also learn Parquet and CSV for larger tables.
- [3] Working with data. SQL, DuckDB, pandas or Polars, and the Hugging Face `datasets` library and Hub.
- [3] Splits and leakage. Keep training, validation and test sets separate. Split by user or by conversation, not by row, so near-copies of an example don't end up on both sides.
- [3] Deduplication. Find exact matches by hash, and near-duplicates with MinHash or embedding similarity.
- [3] Sampling. Random and stratified sampling (for example, by topic, user segment or difficulty), and how to sample production traffic for review.
- [3] Basic statistics. Distributions, class imbalance, confidence intervals, and how many examples you need before you can trust a difference between two versions.
- [3] Versioning. Save each dataset as a fixed snapshot with a version ID, and record which snapshot you used with which prompt and model. Tools such as DVC or Hugging Face Hub revisions help, but a well-kept folder of snapshots is enough to start.
- [3] Embeddings for analysis. Clustering, near-duplicate detection, topic discovery and measuring diversity. Using embeddings to build search belongs to Layer 4.
- [2] Tokens. How to count them, and how tokenization differs across languages and code. These differences change cost and how much fits in the context window.
- [2] Pipelines. Enough data engineering to run a scheduled job that turns logs into datasets, typically a warehouse, SQL and a job scheduler.
- [2] Multimodal data. Images, audio, screenshots and scanned documents, and the extra labeling and privacy work they need because of faces, voices and handwriting.

Check: Take 10,000 logged conversations and deduplicate them. Draw a stratified sample of 300 and split it without leakage. Save it as a versioned JSONL snapshot with a short note on where the data came from.

## Pre-training corpora [1, some topics 2]

Pre-training corpora are the huge text collections that labs train foundation models on. You need to know enough about them to explain model behavior and to judge claims about models.

- [1] What goes in. Web crawls such as Common Crawl, filtered web datasets such as FineWeb, code, books, papers, math, text in many languages, licensed data, and a growing share of synthetic data.
- [1] How they're built. Labs crawl the web and extract the text. They detect the language and filter for quality. They remove duplicates, personal data, toxic text and benchmark questions. Finally they choose the mix of sources and tokenize.
- [2] Quality filtering and deduplication. Rules plus trained classifiers that score quality. These are worth learning properly, because you'll use the same techniques on your own data.
- [1] Data and scale. Models train on trillions of tokens. The Chinchilla result shows how to balance model size against the amount of data. Data quality now matters more than quantity.
- [2] Knowledge cutoff. A model knows nothing about events after its training data ends. It also knows less about recent events, niche topics and less-common languages. This is one of the main reasons to use retrieval (Layer 4).
- [2] Contamination. Questions from public benchmarks leak into training data, which inflates benchmark scores. This is why you need private evaluation sets.
- [2] Memorization. Models can repeat training text word for word, which creates copyright and privacy risk in their outputs.
- [2] Model cards. Reading the data section of a model card or technical report, and noticing what it leaves out.
- [1] Licensing debates. Opt-outs, robots.txt, copyright lawsuits and licensing deals. Know enough to answer a legal or procurement question about a model.
- [1] Continued pre-training. Training a model further on a large domain corpus, such as legal or medical text, and when a team would pay for that rather than use fine-tuning or retrieval.

Check: Explain why a model is weak on a topic, based on what its training data probably contained. Explain to a stakeholder why a public benchmark score doesn't predict results on your task.

## Fine-tuning and preference datasets [3]

When prompting and retrieval aren't enough, app teams fine-tune models. They either use a provider's API or apply a lightweight method such as LoRA to an open-weight model. The training method is mostly handled for you, and how it works belongs to Layer 7. What you control is the dataset. If your team never fine-tunes, level 2 is enough, but these dataset skills still carry over to evaluation.

- [3] When fine-tuning helps. Fine-tuning reliably changes format, style, tone and behavior on a narrow task. It also lets a smaller, cheaper model do a job. It is unreliable for teaching a model new facts, so use retrieval for those.
- [3] Supervised fine-tuning (SFT) data. Example conversations that show the ideal response. They are in chat format and can include system prompts, multi-turn conversations and tool calls. Usually the model learns only from the assistant's turns.
- [3] Dataset design. Cover the real range of tasks. Include edge cases and examples of requests the model should decline. Keep formatting consistent. Hundreds of excellent examples often beat thousands of noisy ones.
- [3] Distillation data. Train a smaller, cheaper model on a strong model's outputs, and filter those outputs first. One filter, called rejection sampling, generates several answers and keeps the best one. Check the provider's terms, which often forbid using outputs to train competing models.
- [2] Agent trajectories and reasoning. Agent trajectories are multi-step records of tool calls and their results, used as training examples. Usually only successful runs are kept. Also decide whether to include step-by-step reasoning in the examples.
- [3] Preference data. Labels that say which response is better. They can be pairs of a better and a worse response to the same input, rankings, or single thumbs-up or thumbs-down labels.
- [2] Which methods use which data. DPO needs pairs. KTO works with unpaired thumbs-up and thumbs-down labels, which is what most products collect. Reward models learn from comparisons. Preference labels can come from people (RLHF) or from AI models (RLAIF).
- [2] Chat templates. Open-weight models expect a specific format with special tokens. If the format used in training doesn't match the one used in serving, quality drops and you get no error.
- [2] Retrieval training data. Used to fine-tune embedding models and rerankers. Each example has a query, a passage that answers it, and a passage that looks relevant but doesn't answer it (a "hard negative"). Learn this to level 3 if you own retrieval.
- [3] Classifier data. Training data for small models you build yourself, such as routers, intent detectors and guardrails. An LLM often labels it and people check the labels.
- [3] Held-out tests. A test set that never enters training, checked for near-duplicates of training examples.
- [2] Failure signs. Overfitting, and losing general ability after fine-tuning. Mixing general data back into training helps prevent the second.

Check: Build a dataset for a narrow task and use it to fine-tune a small model. The fine-tuned model should beat the prompted baseline on a held-out set, and you should be able to say which dataset changes improved the result.

## Human labeling [3]

Evaluation and fine-tuning both need correct answers to compare against. On most app teams, the people who label are the engineers and a few domain experts.

- [4] Error analysis. Read real outputs and write a short note on each failure. Then group the notes into a small set of failure types. Of all the data work on this list, this does the most to improve a product. It is shared with Evaluation (see `concern_01_evaluation.md`).
- [3] Labeling guidelines. Precise definitions, worked examples, and rulings on edge cases. Give the guidelines a version number, so you can tell which rules each label was made under.
- [3] Label types. Pass/fail, which is usually more consistent than a 1-to-5 scale. Also categories, highlighted spans, side-by-side preferences, rankings, and a written reason for each label.
- [3] Agreement. Have two people label the same items and measure how often they agree, using Cohen's kappa or Krippendorff's alpha. Then meet to align on the rules and settle disagreements. On subjective tasks, disagreement can be real information rather than noise.
- [3] Quality control. Hidden test items with known answers, spot audits, and watching for labelers who drift from the rules or get tired.
- [3] Labeling tools. Hands-on experience with one tool, such as Label Studio or Argilla. A simple review app built around your own traces is often faster.
- [3] LLM-assisted labeling. An LLM labels first and people check its labels. Before you trust it, measure how often it agrees with people. This is also how you validate an LLM judge.
- [2] Labeler bias. Favoring the first option shown, favoring longer answers, and anchoring on the model's answer.
- [2] What to label next. Label uncertain outputs, disagreements, new kinds of requests and failures first, rather than picking items at random. This approach is called active learning.
- [2] Who labels. Domain experts, trained crowd workers, or vendors. Specialized work is increasingly done by paid experts. Weigh cost, speed and quality, and make sure labelers are paid and treated fairly.

Check: Write a labeling guide and have two people label the same 100 items. Reach acceptable agreement after one alignment round, then use the labels to measure an LLM judge.

## Synthetic data [3]

Synthetic data is data that a model generates. It fills gaps before you have users, and it covers rare or risky cases and cases where real data is too sensitive to use. It is also what distillation runs on.

- [3] Generation along dimensions. List what varies in your task, such as user type, intent, difficulty and scenario. Pick combinations, then have a model write a realistic input for each one. This way you control what the data covers.
- [3] Expanding from seeds. Start from real examples. Have a model write variations (Self-Instruct), harder versions (Evol-Instruct), or versions in the voice of different personas.
- [3] Generating from your documents. Pairs of questions and answers drawn from your documents, including questions that need more than one document to answer. Use them to test retrieval.
- [3] Simulated users. A model plays the user in multi-turn conversations or agent tasks.
- [3] Structured output. JSON schemas or typed models, so that every generated example can be parsed.
- [3] Quality checks. Filter examples with graders, remove duplicates and measure diversity. Verify answers where you can, for example by running generated code. Have a person review a sample.
- [3] Failure modes. Examples that all sound alike. Inputs that are cleaner and more polite than what real users write. Made-up facts. Bias that the generating model amplifies. A model that grades its own outputs too generously.
- [2] Model collapse. When models are trained repeatedly on model-generated data, they lose the variety found in real data.
- [2] Adversarial examples. Generating jailbreaks, prompt injections and out-of-scope requests for red-teaming. This is shared with Security (see `concern_03_security.md`).
- [2] Privacy-preserving synthetic data. Realistic stand-ins for sensitive records, and why they can still leak real ones.
- [2] Legal limits. Provider terms on using model outputs for training, and the license status of generated data.

Check: Build a synthetic evaluation set and compare its topics and style with real traffic. Fix at least one gap you find.

## RL environments [2, graders 3]

RL environments are simulated tasks that models learn from, with automatic scoring. Labs use them to train reasoning and agent models. Some providers also offer reinforcement fine-tuning, where you supply the tasks and a grader. Agent evaluation harnesses are built the same way, so the skills overlap.

- [2] Parts of an environment. The tasks, the tools or actions available, what the model sees after each step, when an attempt ends, and how it is scored.
- [2] Reward types. The main types are checkable results (tests pass, the answer matches, the code runs), rubrics scored by an LLM, and learned reward models. Know the difference between scoring only the final result and also scoring each step.
- [3] Writing graders. Use exact checks where you can, code-based checks next, and LLM rubrics last. The same graders run your evaluations.
- [3] Reward hacking. Models find ways to score well without doing the task. For example, a model may delete failing tests instead of fixing the code. Knowing these patterns makes your graders harder to fool.
- [2] Task difficulty. A model learns nothing from tasks it always passes or always fails. Measure pass rates across several attempts (pass@k) to find tasks at the right difficulty.
- [2] Agent environments. Sandboxed tools, mock APIs, simulated users, state that resets, and runs that can be repeated exactly, often in containers. Public examples include SWE-bench, tau-bench, WebArena and OSWorld.
- [2] Reinforcement fine-tuning through a provider. What you supply, how it differs from SFT, and when it's worth the cost.
- [2] Dual use. Design your agent evaluation harness so that it could also serve as an RL environment. It should be repeatable, resettable and graded automatically.

Check: Build a small environment with a working grader for one agent task. Find a way to score well without doing the task, then close that gap.

## Product feedback [4]

Your product is the only source of this data, and it is the main advantage an application team has over the model labs. As `ai_stack.md` describes, feedback flows from Product and UX back into Data.

- [4] Explicit feedback. Thumbs up or down, ratings, reports and comments. There is little of it and it is skewed, because people mostly rate interactions that went memorably well or badly.
- [4] Implicit feedback. Users editing the output, accepting or rejecting a suggestion, copying, regenerating, rephrasing the same request, abandoning a session, or asking for a human. There is a lot of it, but it is noisy.
- [4] Outcome signals. Business results tied to an interaction, such as a resolved ticket, merged code or a completed purchase.
- [4] Traces into datasets. Sample traces for review and cluster them by topic and failure. Turn failures into regression tests. Add good interactions to fine-tuning or preference data. One corrected output can serve three uses: as an evaluation case, as a fine-tuning example, and together with the original output as a preference pair. Put each example in only one split.
- [4] Closing the loop. Decide what will fix a failure: a prompt change, a retrieval change, new evaluation cases, fine-tuning, or routing. Then confirm the fix with the evaluation set.
- [3] Feedback design. Work with Product and UX on this. Keep feedback prompts low-friction, ask users for a reason, and save user edits as corrected examples.
- [3] Instrumentation. Log the full trace: input, retrieved context, prompt version, model, tool calls, output, latency and cost. Give it IDs that tie each piece of feedback to the exact trace and version. This is shared with Observability (see `concern_02_observability.md`).
- [3] Bias in feedback. Only some users choose to give feedback, and the interface shapes what they say. Feedback may have been collected on an older version of the product. The model's own outputs can also shape its future data, which creates a loop.
- [3] Drift. Notice when what users ask about changes over time, using topic clustering and embedding distances. Refresh datasets to match.
- [3] Permission to use the data. Consent, terms of service, enterprise contracts that forbid training on customer data, zero-data-retention agreements and data residency. This is shared with Governance and safety (see `concern_04_governance_and_safety.md`).
- [2] Online tests. A/B tests that confirm improvements measured offline also hold for real users. This is shared with Evaluation (see `concern_01_evaluation.md`).

Check: Link the feedback in your product to its traces. Set up a scheduled job that turns new failures into evaluation cases. Point to one change that came out of this loop and improved a metric.

## Cross-cutting concerns in this layer

### Evaluation: building evaluation datasets [4]

How you measure is covered in `concern_01_evaluation.md`. This part covers the data you measure with. For an application engineer, the evaluation set is the dataset you will build and rely on most.

- [4] Types of sets. Regression sets hold past failures that must stay fixed. Capability sets cover what the product should do. Adversarial sets cover attacks and abuse. Slices break results down by user segment, language or difficulty.
- [4] Sources. Production traces, synthetic generation, domain experts, and public benchmarks. Use public benchmarks carefully, because of contamination.
- [4] Coverage. Map out the dimensions of your task, and check that each important combination has examples.
- [4] Reference answers. Decide which cases need a known correct answer, and which can be graded against criteria alone.
- [4] Judge calibration sets. Examples labeled by people and used to measure how often an LLM judge agrees with them.
- [4] Maintenance. Give each set a version, and retire cases that every version passes. Add new failure types as they appear. Don't tune prompts against the test set so much that it stops predicting real results.
- [3] Size. Enough examples that you can trust a difference between versions. Learn confidence intervals and bootstrapping.
- [3] Retrieval test sets. Queries paired with the passages that retrieval should find.
- [2] Public benchmarks. What the common benchmarks measure, and why they rarely predict results on your task.

### Data quality [3]

- [3] Dimensions. Accuracy, completeness, consistency, freshness, uniqueness, validity, and representativeness (whether the data looks like real use).
- [3] Validation. Check every example against a schema, using JSON Schema or Pydantic. Run automated checks whenever a dataset changes, for example with pandera or Great Expectations.
- [3] Lineage. Record where each example came from: whether it is real or synthetic, which model and prompt generated it, and which labeler labeled it under which guideline version.
- [3] Dataset documentation. A short data card for each dataset that covers its purpose, sources, collection method, known gaps and license. "Datasheets for Datasets" is the model to follow.
- [3] Decontamination. Keep evaluation examples out of training data and out of few-shot prompts.
- [3] Staleness. Datasets go out of date as the product and its users change, so set a refresh schedule.
- [2] Label noise. Estimate how many labels are wrong and find them, for example with confident learning or by relabeling items where labelers disagreed.

### Governance and safety [2–3]

- [3] Personal data. Detect personal data and remove or mask it before it enters any dataset, for example with Presidio, pattern matching or an LLM. Masked data can still identify people.
- [3] Consent, purpose and retention. Use data only for what users agreed to, and keep only what you need. Delete data on schedule and when users ask.
- [3] Provider data policies. Whether a provider stores API inputs or trains on them, how long it keeps them, and where it processes them.
- [2] Bias and representation. Which groups, languages and cases are missing or overrepresented. Check results one slice at a time.
- [2] Licensing and IP. Dataset licenses, such as CC BY versus non-commercial or research-only licenses. Also provider terms on outputs, and copyrighted material in fine-tuning data.
- [2] Regulations. GDPR, CCPA, HIPAA, and the EU AI Act's rules on data governance and training-data documentation. Know enough to recognize when to bring in legal.
- [1] Machine unlearning. Removing specific data from a trained model is hard and unreliable. This is one reason to filter data before training.

### Security [2]

- [3] Leaks. Secrets and personal data can slip into logs, then into datasets, then into fine-tuned models, where attackers can extract them.
- [2] Data poisoning. Malicious examples planted in fine-tuning data or scraped sources to create hidden behaviors.
- [2] Feedback manipulation. Users gaming ratings to steer what the system learns.
- [2] Access control. Permissions and audit logs for datasets that contain customer data.

### Cost [2–3]

- [3] Sample size. Build the smallest dataset that answers your question.
- [2] Data costs. Expert labeling hours, tokens spent generating synthetic data, and fine-tuning runs. Weigh them against savings, such as cheaper inference after distillation.

Observability and Reliability have little of their own in this layer. Trace capture is covered under Product feedback, and the tracing infrastructure is covered in `concern_02_observability.md`. Reliability comes in only as reproducible datasets and environments, which Foundations and RL environments cover.

## Not in this layer

- Parsing documents, chunking, embedding, vector databases and knowledge graphs belong to Layer 4 (Context, memory and retrieval), even though they process data.
- Long-term memory (what to store, update and forget) belongs to Layer 4.
- How SFT, DPO and RL training work belongs to Layer 7 (Model adaptation).
- Trace storage, dashboards and alerts are covered in `concern_02_observability.md`.
- Graders, LLM judges and experiment design are covered in `concern_01_evaluation.md`.

## Learning order

1. Foundations.
2. Error analysis and labeling, then evaluation datasets. You'll judge everything that follows with these.
3. Synthetic data.
4. Product feedback.
5. Data quality, governance and security.
6. Fine-tuning and preference datasets. You need evaluation sets before you can tell whether fine-tuning helped.
7. RL environments, which build on graders and evaluation.

You can read about pre-training corpora at any point. The material is light, and it explains model behavior you'll see along the way.

## Key reading

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 8 covers dataset engineering and chapter 10 covers user feedback.
- Hamel Husain's and Shreya Shankar's writing on evals and error analysis.
- Shankar et al., "Who Validates the Validators?" (2024), on aligning LLM judges with human labels.
- Gebru et al., "Datasheets for Datasets."
- Zhou et al., "LIMA: Less Is More for Alignment" (2023), on small, high-quality fine-tuning sets.
- Penedo et al., "The FineWeb Datasets" (2024), on how a pre-training corpus is built.
- Nathan Lambert, *Reinforcement Learning from Human Feedback* (rlhfbook.com), on preference data, reward models and RL training.
- Shumailov et al., "AI models collapse when trained on recursively generated data" (*Nature*, 2024).
