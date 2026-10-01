# Evaluation: what an application-layer AI engineer needs to know

This file covers the Evaluation concern from `ai_stack.md`. Evaluation measures whether the system does its job, using test sets, automated graders (including models that judge other models), human review, and A/B tests in production. It is one of the seven cross-cutting concerns: not a layer, but something every layer needs, from choosing a model to checking retrieval quality and whether an agent finished its task. This file is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this concern and how deeply to learn each one.

## The rule for depth

Many of the checks in the layer files end the same way: show that it works on your evaluation set. This concern is how you do that well. You can buy models, gateways and vector databases, but no one can sell you an evaluation of your product, because only you know what doing its job means for it. Evaluation is also how you find out whether anything else in this curriculum worked. That makes it the deepest cross-cutting concern for an application engineer.

For every change, evaluation answers four questions:

1. What does doing its job mean for this feature? (The evaluation loop, Defining what to measure)
2. How is each output scored? (Code-based checks, LLM judges, Human review)
3. Is the new version really better than the old one? (Comparing versions, Evaluation in development)
4. Does it hold up with real users, and against people trying to break it? (Evaluation in production, Experiments with real users, Safety and adversarial testing)

Most concepts in this concern are level 3, and four things are level 4:
- the evaluation loop: building evaluations from the failures you actually see, not from generic metrics
- defining what good means, as criteria that people and code can check
- building LLM judges and showing that they agree with people
- running evaluation on every change, and deciding in advance what a change must show to ship

The layer files cover evaluation as it applies to each layer, such as retrieval metrics in Layer 4 and agent task success in Layer 2, and `layer_10_data.md` covers building evaluation datasets. This file covers the methods they all share, and Evaluation at each layer maps where the rest is covered. Learn public benchmarks to level 2, because they only narrow a list of models before your own evaluation decides. Experiments with real users are level 3: data scientists often run the experimentation platform, but you design the tests for AI changes.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| The evaluation loop | 4 | Evaluations built from real failures catch what generic metrics miss |
| Defining what to measure | 3 (some topics 4) | You can't check "good" until you've said what it means |
| Code-based checks | 3 | Fast, free and exact wherever code can decide |
| LLM judges | 3 (some topics 4) | They scale human judgment, but only once they agree with people |
| Human review | 3 (some topics 4) | People decide what good means and check what nothing else can |
| Comparing versions | 3 (some topics 4) | Small differences between versions are often chance |
| Evaluation in development | 3 (some topics 4) | A change that skips evaluation ships untested |
| Evaluation in production | 3 | Real users find what your test set missed |
| Experiments with real users | 3 | Only real users show whether a change helps them |
| Safety and adversarial testing | 3 | Harms and attacks need test cases of their own |
| Public benchmarks | 2 | They narrow a list of models but don't predict your results |
| Observability | 3 | Traces are what you evaluate |
| Cost and latency | 3 | Every comparison needs them, and evaluation costs money too |
| Reliability | 3 | An answer that changes between runs is a quality problem |
| Security | 2–3 | Judges and graders can be fooled |
| Governance and safety | 2–3 | Customers and regulators increasingly ask for test results |

## The evaluation loop [4]

Evaluation is a loop, not a one-off test. You read what the system produces, decide what counts as a failure, measure it, change something and measure again.

- [4] Start from real outputs. Read a sample of real or realistic outputs, note what went wrong in each, and group the notes into failure types. This is error analysis, covered in `layer_10_data.md`. Then build evaluations for the failure types that matter. Ready-made scores for qualities such as "helpfulness" or "coherence" rarely match the failures your product actually has.
- [4] Evaluation first. Before building a feature, write down what success looks like and collect a small set of test cases, the way tests come before code in test-driven development. Judge every change by those cases. Over time, the evaluation set becomes the working definition of what the feature must do.
- [3] Beyond trying a few examples. Trying a handful of inputs by hand, often called a vibe check, is how every feature starts, and it's fine early on. It stops working once the feature handles many kinds of input, because fixing one case quietly breaks another. Write the cases down the first time a change breaks something you had already fixed.
- [3] Three tiers. Cheap code checks run on every change. Larger suites with LLM judges and human review run before releases. Experiments with real users run in production. Each tier catches what the cheaper ones miss, and costs more.
- [4] One change at a time. Run the evaluation, read the failures, change one thing and run it again. Keep every run's results with the versions it used, so you can see which change helped. The loop never ends, because new users, models and features bring new kinds of failure.
- [4] Who decides what good means. Someone who knows the domain has to decide what counts as a good output. On a small team, making one domain expert the final judge keeps the standard consistent. Engineers grading outputs in a field they don't know miss the failures that matter most.

Check: For one feature, run error analysis on 100 real outputs (see `layer_10_data.md`) and turn the three most common failure types into evaluations. Use them to judge three changes, and show which change fixed a failure type without breaking another.

## Defining what to measure [3, some topics 4]

An evaluation is only as good as its definition of success. This section covers turning a goal into things you can check.

- [4] From goals to criteria. Turn "the summaries should be good" into criteria that two reviewers would apply the same way, such as "includes every action item", "makes no claim that isn't in the source" and "is under 150 words". Write a definition for each, with one passing and one failing example.
- [4] One result for each failure type. Measure each criterion separately, as pass or fail, rather than rolling everything into one score out of 10. Separate results show what to fix, and one overall score hides it. Pass/fail labels are also more consistent than scales (see `layer_10_data.md`).
- [3] What one test case is. Decide whether a case is a single output, one turn of a conversation, a whole conversation or a whole agent task. For chat, also grade whole conversations: did the user get what they came for, and did the model keep to its instructions to the end? Simulated users make multi-turn tests repeatable (see `layer_10_data.md`), and agent tasks are graded by their end state (see `layer_02_agents_and_orchestration.md`).
- [3] With or without a reference answer. Some criteria compare the output with a known correct answer. Others check the output on its own, against a rule or a rubric. Only checks that need no reference answer work on production traffic, where nobody knows the correct answer. Which cases need reference answers is covered in `layer_10_data.md`.
- [3] Quality, cost and latency together. Report cost and latency next to quality for every version you compare. Plot quality against cost, and choose among the versions that no other version beats on both. A version that is slightly better but twice as slow may not be better.
- [3] Baselines. A score means little on its own. Compare it with the current version, with a simple approach such as a rule or a keyword search, and, where you can, with how well people do the task today.
- [3] Guardrail metrics. Besides the metric you're trying to improve, choose metrics that mustn't get worse, such as safety failures, refusals of reasonable requests, format errors and cost. Product guardrail metrics are covered in `layer_01_product_and_ux.md`.
- [3] Standard metrics for standard tasks. For classification, measure precision and recall for each class. Precision is how many of the items given a label really have it, and recall is how many of the items that have it were given it. Decide which mistake costs more, a false alarm or a miss, and weigh the two to match, because overall accuracy hides failures on rare classes. For extraction, measure accuracy field by field. Your judges, routers and guardrails are classifiers too, so the same metrics apply to them.
- [3] Metrics become targets. A metric you push hard stops measuring what you meant, which is known as Goodhart's law. Prompts tuned against one score find ways to raise it without improving the product. Keep reading outputs, not just scores.
- [2] Scores and thresholds. If your product acts on a score, such as a guardrail that blocks above a threshold or a cascade that escalates uncertain answers (see `layer_05_model_gateway_and_routing.md`), check that the scores mean what they say. About 90% of the answers scored at 90% confidence should be right. Set thresholds from labeled data, by seeing how precision and recall change as the threshold moves.
- [2] Older automatic metrics. BLEU and ROUGE count the words an output shares with a reference answer, and embedding similarity compares meanings. They're cheap, but on open-ended text they agree poorly with people, so use them only for narrow tasks such as translation. Perplexity, which measures how predictable a text is to a model, is mainly a training metric.

Check: For one feature, write five to eight pass/fail criteria from your error analysis, each with a definition and examples. Have a colleague apply them to the same 30 outputs as you, and rewrite every criterion you disagreed on.

## Code-based checks [3]

Checks written in code are fast, free and exact, and they give the same verdict every time. Use them for everything code can decide, and save LLM judges for the rest. The same checks can also score RL environments (see `layer_10_data.md`).

- [3] Format checks. Valid JSON that matches its schema, required fields present, values within allowed ranges, length limits, the right language, and no leftover template text or placeholders.
- [3] Content checks. Required facts or phrases present, forbidden ones absent, citations that point to real sources, numbers that match the source document, and links that work. String matching and regular expressions cover many of these.
- [3] Running the output. When the output can be run, run it. Execute generated code against tests, run generated SQL and compare its results with the expected ones, or send generated tool arguments to a sandbox. A correct answer can be written many ways, so checking what the output does beats comparing its text. Checking an agent's end state is covered in `layer_02_agents_and_orchestration.md`.
- [3] Comparing with references. Normalize before comparing: case, spacing, and number and date formats. Compare lists as sets where order doesn't matter, allow a tolerance for numbers, and compare structured output field by field.
- [3] Behavioral tests. Change inputs in controlled ways and check how the output responds. A typo or a different customer name shouldn't change the answer (an invariance test). Adding a constraint should change it in a predictable way (a directional test). Simple cases that must always work should always pass (a minimum functionality test). The CheckList method named these three kinds of test.
- [3] Checking the checks. Run each check on outputs you've labeled by hand, including correct answers in unusual formats. A check that fails correct outputs does as much harm as one that passes wrong ones. Anything tuned against a check can also learn to pass it without doing the task (see reward hacking in `layer_10_data.md`).

Check: Write code checks for every criterion that code can decide. Run them on 200 outputs, and hand-check 30 of their verdicts. Fix every check that failed a correct output or passed a wrong one.

## LLM judges [3, some topics 4]

An LLM judge is a model call whose job is to grade another output. It can apply criteria that code can't, such as tone, or whether a summary left out something important. But it's a model call like any other: it can be wrong, biased, inconsistent and expensive, so it needs evaluating too. Choosing the judge's model is covered in `layer_08_foundation_models.md`.

- [4] Writing a judge. Give each judge one criterion, and ask for pass or fail rather than a score from 1 to 10, which models use as inconsistently as people do. Define pass and fail precisely, with labeled examples of each, especially borderline ones. Ask for a short explanation before the verdict. It tends to make verdicts more accurate, and it shows you why the judge decided as it did. Give the judge what it needs: the input, the output, and the source documents or reference answer where they matter.
- [4] Checking a judge against people. Before trusting a judge, compare its verdicts with human labels (see judge calibration sets in `layer_10_data.md`). Measure two things separately: how often it catches real failures, and how often it passes real successes. Overall agreement misleads: when one output in ten fails, a judge that passes everything agrees with people 90% of the time. Improve the judge's prompt on one part of the labeled data and measure it on another, so the result isn't inflated.
- [3] Correcting the judge's numbers. A judge's pass rate is an estimate with known errors. If you know how often the judge misses failures and how often it fails good outputs, you can correct the pass rate it reports and put a range around the corrected figure.
- [3] Kinds of judges. A judge can grade one output against criteria, compare two outputs and pick the better one, or grade an output against a reference answer. Comparisons suit subjective quality and choosing between two versions. Grading single outputs gives pass rates you can track over time.
- [3] A checklist for each case. For open-ended tasks, write a short checklist for each test case of what a good answer must include or avoid, and have the judge check each item. Specific items are easier to grade consistently than overall quality.
- [3] Judge biases. Judges tend to favor the answer shown first, longer answers and answers from their own model family (see `layer_08_foundation_models.md`). Run each comparison in both orders, and count it as a tie when the two verdicts disagree. Check length separately if it matters.
- [3] Stable judges. Pin the judge's model snapshot, and version its prompt like any other prompt (see `layer_04_context_memory_and_retrieval.md`). A new judge model changes every score, so check it against people again before comparing old results with new ones.
- [3] Criteria that shift. People refine what they mean by good as they read more outputs. This is called criteria drift. Expect the judge's definitions to change during the first rounds of labeling, and check the judge again after each change.
- [2] Judges that can't do the task. A judge can't reliably check what it couldn't do itself, such as a hard proof or a fact it doesn't know. Give it the reference answer or the source, or use a code check.
- [2] Panels and trained judges. A panel of smaller judges from different model families can do as well as one large judge, at lower cost. Models trained to evaluate, including reward models (see `layer_07_model_adaptation.md`), can score outputs too. Small models trained for one check, such as whether a claim follows from a source or whether text is toxic, are cheaper and faster still. A strong model with a good prompt is the usual starting point.

Check: Build a judge for one criterion. On 100 human labels it wasn't tuned on, report how often it catches failures and how often it passes successes. Improve it until both are acceptable, then report its corrected pass rate on a sample of production outputs.

## Human review [3, some topics 4]

People define what good means, and they check what code and judges can't. Writing labeling rules, measuring agreement between labelers and running a labeling process are covered in `layer_10_data.md`. This section covers using people's judgment to evaluate.

- [4] What people are for. People decide what counts as good, label the data that judges are checked against, review what automated checks can't decide, and check high-stakes outputs directly. As automated evaluation earns trust, people review fewer outputs. But someone should still read real outputs every week.
- [3] Side-by-side comparisons. To choose between two versions, show reviewers both outputs for the same input, in random order and without saying which is which. Ask which is better, or whether they're tied. People compare more consistently than they score. Report the win rate with a confidence interval.
- [3] Random and flagged samples. Send reviewers a random sample of outputs to estimate how often things go wrong. Separately, send them flagged outputs, such as judge failures, user complaints and uncertain answers, to find problems. Report the two separately, because flagged outputs overstate the failure rate. Choosing what to label first is covered in `layer_10_data.md`.
- [3] Expert review. In fields such as medicine, law and finance, only experts can tell a correct answer from a plausible one. Their time is scarce, so send them the cases that need expertise, and use their labels to check your judges.
- [3] Reviewing whole traces. Show reviewers the input, the context, the tool calls and the output together. Seeing every step shows where things went wrong, not just that they did. A simple review screen built around your traces, with one-click labels, gets far more reviewing done than a spreadsheet (see `layer_10_data.md`).

Check: Run a blind side-by-side comparison of two versions of a feature on 100 inputs, with two reviewers. Report the win rate with a confidence interval, and whether your automated evaluation picked the same winner.

## Comparing versions [3, some topics 4]

Most evaluation work compares two versions: an old and a new prompt, model or retrieval setting. Small differences between them are often just chance. The basic statistics are covered in `layer_10_data.md`. This section covers using them to compare versions.

- [3] Scores vary between runs. The same version can score differently on two runs, because model outputs vary, and so do judges. Run the evaluation more than once, or run each case several times, to see how much scores move by chance before calling a difference real. Success across repeated runs of agent tasks is covered in `layer_02_agents_and_orchestration.md`.
- [3] Error bars. Report every score with a confidence interval. With 100 cases and a 70% pass rate, the 95% interval runs from about 61% to 79%, so on its own a 3-point gain could easily be chance. Bootstrapping, which resamples the cases many times, gives an interval for any metric.
- [3] Paired comparisons. Run both versions on the same cases and compare them case by case. This is far more sensitive than comparing two overall scores, because a case that is hard for one version is usually hard for the other. Count the cases each version fixed and the cases it broke, not just the net change. McNemar's test, or a bootstrap of the differences between pairs, tells you whether the difference is likely real.
- [3] Slices. Break results down by task type, user segment, language and difficulty. An overall gain can hide a drop on a slice that matters, and slices can move in opposite directions.
- [3] Held-out cases. If you tune prompts on the same cases you report on, the score rises faster than real quality. Keep some cases aside and use them only to confirm a final choice (see `layer_10_data.md`).
- [2] Cases that aren't independent. Cases drawn from the same document or conversation tend to pass or fail together, so they carry less information than their number suggests. Group them when you compute intervals, or the intervals come out too narrow.
- [2] Many comparisons. Try twenty prompt variants, and one will look better by chance. Confirm the winner on fresh cases before you ship it.
- [4] Deciding in advance. Before the evaluation runs, agree what a change must show to ship: which criteria must improve, by how much, which must not get worse, and which failures block a release whatever the average says. Otherwise people read the results as support for what they already wanted.

Check: Run two prompt versions on the same 200 cases, three times each. Report each version's pass rate with a confidence interval, the paired difference with its interval, and the cases each version fixed and broke. Decide whether to ship, using a rule you wrote down before the runs.

## Evaluation in development [3, some topics 4]

Evaluation helps only if it runs before changes ship. This section covers building it into everyday development.

- [3] An evaluation harness. A harness runs a dataset through your system, applies the checks and judges, and records the results. Record the version of everything with each run: the dataset, prompts, models, checks, judges and code. Then any run can be repeated and compared with earlier ones.
- [4] Evaluation on every change. Run the evaluation set before any change ships, whether it's a prompt, a model, a tool, a retrieval setting or a route. Each layer file says this for its own kind of change. Make it part of code review, with the results posted on the change.
- [3] Gates in CI. Run fast checks on every commit in continuous integration (CI), and larger suites before each release. Block a merge when a regression case fails or a guardrail metric falls past a set limit. Set limits wider than normal run-to-run variation, or the gate fails at random and people learn to ignore it.
- [3] Fast, cheap runs. Cache model responses, so rerunning unchanged cases costs nothing. Run cases in parallel, and send large runs to batch APIs (see `layer_09_compute.md`). A suite that takes an hour runs rarely. One that takes five minutes runs on every change.
- [3] Reading what changed. Compare two runs side by side: scores for each criterion and slice, and the cases whose verdicts changed, with both outputs. Most of what you learn comes from reading the cases that changed.
- [3] Replaying real inputs. Before a release, run a recent sample of real inputs through the old and the new version, and compare the outputs with a comparison judge or by reading them. This catches changes your curated cases don't cover. Sending copies of live traffic to a new version without showing users (shadow traffic) is covered in `layer_05_model_gateway_and_routing.md`.
- [3] Tools. Open-source frameworks such as promptfoo, Inspect and DeepEval run evaluations from code or configuration. Platforms such as Braintrust, LangSmith, Langfuse and Arize Phoenix combine tracing, datasets, evaluation and human review. Learn one well. A script and a spreadsheet are enough to start, and many teams build their own review screens.

Check: Set up a harness that runs on every pull request that changes a prompt, model or tool. Have it post scores and changed cases on the pull request, and block the merge when a regression case fails. Keep a full run under ten minutes.

## Evaluation in production [3]

Real users send inputs your evaluation set never imagined, and what they ask changes over time. Evaluation in production catches what offline evaluation misses, and its failures become new offline cases (see `layer_10_data.md`).

- [3] Scoring live traffic. Run code checks, and judges you've checked against people, on a sample of production traces. Use criteria that need no reference answer, since nobody knows the right one. Attach each score to its trace (see Observability below), and track pass rates by feature, version and slice.
- [3] Alerts on quality. Alert when a pass rate falls, when format errors or refusals rise, or when the mix of requests shifts (see drift in `layer_10_data.md`). Quality can fall without a single error in the logs: a model changes behind an alias (see `layer_08_foundation_models.md`), a document source changes, or users start asking about something new.
- [3] Blocking or measuring. The same check can run in the request path, stopping or repairing a bad output before the user sees it, or run afterwards on samples to measure quality. Checks in the request path must be fast and cheap, because they add latency to every request. Which guardrails to run is covered in `concern_04_governance_and_safety.md`.
- [3] Gradual rollouts. Release a change to a small share of traffic first, called a canary, and watch quality, errors and cost. Roll back if they get worse. The traffic splitting itself is covered in `layer_05_model_gateway_and_routing.md`.
- [3] What users do as a signal. Accepts, edits, regenerations, thumbs-down and requests for a person are noisy but plentiful signs of quality (see `layer_01_product_and_ux.md`). Use them to find problems, and to check that offline gains reach users.

Check: Run your code checks, and two judges you've checked against people, on a daily sample of production traces for one feature. Chart pass rates by version, set an alert on a drop, and turn one alerted failure into a regression case.

## Experiments with real users [3]

Offline evaluation predicts how a change will do. An experiment shows how it does with real users. Splitting traffic is covered in `layer_05_model_gateway_and_routing.md`, and product metrics in `layer_01_product_and_ux.md`. This section covers designing experiments.

- [3] When to run one. Run an experiment when a change could change how users behave, such as a new model, a new interface or different answer lengths, and when you have enough traffic to detect the effect. Small, safe fixes can ship on offline evaluation alone.
- [3] Planning. Before you start, write down the hypothesis, the main metric, the guardrail metrics, the sample size and the duration. Assign users, not requests, to each version, so each user has a consistent experience and no user's behavior is split across versions.
- [3] Sample size and duration. Work out how many users you need to detect the smallest effect worth acting on. This is called a power calculation. Run for at least a full week to cover weekly patterns, and longer for new features, whose use often spikes and then falls.
- [3] Peeking. Checking results every day and stopping as soon as they look good produces false wins. Fix the duration in advance, or use sequential testing methods, which are designed for early looks.
- [3] Checking the experiment itself. Confirm that each version got the share of users it should. A mismatch, called a sample ratio mismatch, means something in the assignment is broken, and the results can't be trusted. Also confirm that users actually saw the version they were assigned.
- [3] Reading results. Look at the main metric, the guardrail metrics and the key slices together. A change that raises engagement but also raises complaints or cost is not a win. Compare the result with what offline evaluation predicted, and fix the evaluation set when they disagree (see `layer_01_product_and_ux.md`).
- [2] Other designs. Interleaving mixes results from two rankers into one list and sees which ones users pick. For search and recommendations, it needs far less traffic than an A/B test. Multi-armed bandits shift traffic toward the better version while the test runs. Holdouts keep a small group of users without a feature for months, to measure its long-term effect.
- [2] Experimentation platforms. Many companies already have an experimentation platform, built in-house or bought, such as Statsig, Optimizely or GrowthBook, and data scientists who run it. Use it and involve them rather than building your own.

Check: Design and run an A/B test of one model or prompt change. Write the hypothesis, metrics and sample size in advance, run for the planned duration, and check for sample ratio mismatch. Report the main and guardrail results with confidence intervals, next to what offline evaluation predicted.

## Safety and adversarial testing [3]

Ordinary test sets check that the system does its job. Safety and adversarial tests check that it doesn't cause harm, even when someone is trying to make it. Building adversarial test sets is covered in `layer_10_data.md`, and defenses in `concern_03_security.md` and `concern_04_governance_and_safety.md`. This section covers testing.

- [3] The harms your product could cause. List the harms that matter for your product, such as harmful or dangerous content, leaked personal data or system prompts, bad medical, legal or financial advice, unfair treatment of some users, and actions an agent shouldn't take. Build cases for each and run them before every release, like any regression set. Safety cases for tools are covered in `layer_03_tools_and_protocols.md`.
- [3] Refusing too much. Test that the product still helps with reasonable requests that look risky, such as a nurse asking about medication doses. Refusing reasonable requests is a failure too, and tighter safety rules often cause it.
- [3] Red-teaming. People, and models prompted to attack, deliberately try to make the product fail, with jailbreaks, prompt injection through documents and tool results, and misuse of features. Run a round before launch and after major changes, and turn every successful attack into a regression case. Tools such as garak, PyRIT and promptfoo automate common attacks.
- [3] Fairness. Compare results across user groups, languages and dialects. Test whether changing a name, gender or dialect in an input changes the outcome, using the behavioral tests described under Code-based checks. Bias in models is covered in `layer_08_foundation_models.md`.
- [3] After a model changes. A new model or a fine-tune can change safety behavior even when nothing else changes. Run the safety set after every model change (see `layer_07_model_adaptation.md`).
- [1] Dangerous-capability evaluations. Before release, labs test frontier models for dangerous capabilities, such as helping with weapons or cyberattacks, and publish the results in system cards. Know what they cover. Application teams don't run them.

Check: Build a safety set for one feature that covers its main harms, plus 20 reasonable requests that look risky. Run a red-team session with two colleagues, add every successful attack to the set, and show the set running in your release checks.

## Public benchmarks [2]

Public benchmarks are shared test sets for comparing models. Benchmarks for particular kinds of work are covered in the layer files: agents in `layer_02_agents_and_orchestration.md`, tool use in `layer_03_tools_and_protocols.md`, retrieval and memory in `layer_04_context_memory_and_retrieval.md`, and leaderboards in `layer_08_foundation_models.md`. This section covers reading benchmark results.

- [2] What the common ones measure. Widely reported benchmarks cover knowledge and reasoning (such as MMLU-Pro, GPQA and Humanity's Last Exam), math (such as AIME), coding (such as SWE-bench Verified and LiveCodeBench), instruction following (such as IFEval) and factual accuracy (such as SimpleQA). Know what a typical question in each looks like, so you can tell whether a score says anything about your task.
- [2] Reading a claim. Check how a score was produced: the prompt, the number of examples shown, how many attempts were allowed (pass@1 means one), the reasoning budget, and for agent benchmarks the harness around the model. Scores from different setups can't be compared, and a model's maker often reports its best setup.
- [2] Why benchmarks wear out. Benchmark questions leak into training data (see contamination in `layer_10_data.md`). And once top models score near the maximum, the differences between them are noise. That's why new benchmarks keep replacing old ones.
- [2] Ratings from comparisons. Leaderboards such as LMArena turn many side-by-side votes into ratings with methods such as Elo and Bradley-Terry. The ratings reflect what voters prefer, which rewards style and length, and labs can test many private versions before release. You can use the same rating methods to rank many versions of your own system from pairwise judgments.
- [2] Running benchmarks yourself. Tools such as EleutherAI's lm-evaluation-harness, Stanford's HELM and Inspect run public benchmarks on any model. This helps when you compare open-weight or quantized models, since published scores may not match the version you run (see `layer_09_compute.md`).

Check: Take one recent model announcement. For each headline benchmark, explain what it measures, whether its setup matches the numbers reported for competitors, and what it predicts about your task. Then run your own evaluation set to find out.

## Evaluation at each layer

Each layer file covers evaluation as it applies to that layer. This table shows where.

| Layer | What to evaluate there | Level | Where |
|---|---|---|---|
| 1. Product and UX | The whole experience as users get it, product metrics, experiments, and whether offline gains reach users | 3 | `layer_01_product_and_ux.md` |
| 2. Agents and orchestration | Task success checked in the end state, success on every one of several runs, trajectories, each step on its own, cost per success | 3 (some topics 4) | `layer_02_agents_and_orchestration.md` |
| 3. Tools and protocols | The right tools with the right arguments, mock tools and replayed responses, cases with injected instructions | 3 | `layer_03_tools_and_protocols.md` |
| 4. Context, memory and retrieval | Which step failed, retrieval metrics, faithfulness and citations, memory across sessions | 3 (some topics 4) | `layer_04_context_memory_and_retrieval.md` |
| 5. Model gateway and routing | Each route and fallback, on the traffic it gets, slice by slice | 3 | `layer_05_model_gateway_and_routing.md` |
| 6. Inference and serving | Quality lost to quantization or to schema enforcement | 2 | `layer_06_inference_and_serving.md` |
| 7. Model adaptation | Beating the best prompt and a stronger model, regressions outside the task, safety after fine-tuning | 3 | `layer_07_model_adaptation.md` |
| 8. Foundation models | Choosing and switching models on your own task, choosing a judge model | 3 (choosing models 4) | `layer_08_foundation_models.md` |
| 9. Compute | The same model on different hardware or providers | 2 | `layer_09_compute.md` |
| 10. Data | Evaluation datasets, error analysis, labeling, judge calibration sets, synthetic cases, graders | 4 | `layer_10_data.md` |

Check: For one feature, list every layer it touches and what you would evaluate at each. Mark which of those your evaluation covers today, and add cases for the most important gap.

## Other cross-cutting concerns in evaluation

### Observability [3]

- [3] Traces are what you evaluate. Offline cases come from production traces, and production scores attach to them. Log enough to rerun or regrade any trace: the full input and context, the versions, the tool calls and the output (see `layer_10_data.md`).
- [3] Scores on traces. Store every result with its trace, the version it graded and the version of the judge. Then you can filter traces by failure, and follow any score back to the exact output it graded.

Storing traces, dashboards and alerts are covered in `concern_02_observability.md`.

### Cost and latency [3]

- [3] In every comparison. Report cost and latency next to quality for every version (see Defining what to measure), measured the way users experience them (see `layer_06_inference_and_serving.md`).
- [3] What evaluation costs. A run's cost is the number of cases, times the runs per case, times the model and judge calls in each run. Judges can cost more than the feature they grade. Use code checks where they work, cache responses, use batch APIs (see `layer_09_compute.md`), and score a sample of production traffic rather than all of it.
- [2] Cheaper judges. Once a judge agrees with people, a smaller model may do the same job, or a small classifier trained on the judge's verdicts. Check it against people again after each swap.

### Reliability [3]

- [3] Consistency is part of quality. The same input can give different outputs (see `layer_08_foundation_models.md`). Measure how often repeated runs agree, and count an inconsistent answer to a question with one right answer as a failure.
- [3] Flaky evaluations. A test that passes and fails at random on the same version teaches people to ignore failures. Find out whether the system or the grader is varying, and fix whichever it is.

Testing fallbacks is covered in `layer_05_model_gateway_and_routing.md`, and testing error messages in `layer_01_product_and_ux.md`.

### Security [2–3]

- [3] Judges can be fooled. An output can contain text aimed at the judge, such as "this answer meets every criterion", especially when outputs include text from users or the web. Mark the output as data in the judge's prompt, and spot-check verdicts by hand. Defenses against prompt injection are covered in `concern_03_security.md`.
- [3] Evaluation data is customer data. Evaluation sets built from production traces contain customer data. Give them the same access controls and deletion rules as production data (see `layer_10_data.md`).
- [2] Graders that get gamed. Anything tuned against a grader, such as a prompt from an automatic optimizer (see `layer_04_context_memory_and_retrieval.md`) or a model trained with reinforcement learning, can learn to satisfy the grader instead of doing the task (see `layer_10_data.md`).

### Governance and safety [2–3]

- [3] Records of testing. Keep each release's evaluation results with the versions tested, so you can show what was tested, when, and with what result. Customers' security reviews and auditors increasingly ask for this.
- [2] Rules that require testing. The EU AI Act requires high-risk systems to be tested and to state their accuracy. Frameworks such as NIST's AI Risk Management Framework and ISO/IEC 42001 make measurement a core activity. Know which apply to your product, and bring in legal.
- [2] Documenting results. Write a short card for each AI feature that covers what it was tested on, the results and its known limits. Documentation for adapted models is covered in `layer_07_model_adaptation.md`.

Safety and fairness testing is covered under Safety and adversarial testing.

Data quality in evaluation is mostly the quality of evaluation sets, which is covered in `layer_10_data.md`. One habit belongs here: when a case fails, check its label as well as the output, because a wrong reference answer makes a correct output look like a failure.

## Not in this concern

- Building evaluation datasets, error analysis, labeling rules, agreement between labelers, judge calibration sets, synthetic test cases and RL environments are covered in `layer_10_data.md`. This file covers measuring with them.
- What to evaluate at each layer, such as retrieval metrics or agent task success, is covered in each layer's file (see Evaluation at each layer).
- Choosing models, and choosing a model to act as judge, are covered in `layer_08_foundation_models.md`.
- Splitting traffic and shadow traffic are covered in `layer_05_model_gateway_and_routing.md`, and product metrics and collecting feedback in `layer_01_product_and_ux.md`. This file covers designing the tests.
- Storing traces, dashboards and alerts are covered in `concern_02_observability.md`. This file covers what gets scored.
- Defenses against prompt injection and jailbreaks are covered in `concern_03_security.md`, and guardrails that block harmful output in `concern_04_governance_and_safety.md`. This file covers testing them.
- Training metrics, such as loss and reward curves, are covered in `layer_07_model_adaptation.md`.
- Building public benchmarks and testing frontier models for dangerous capabilities are lab work. Awareness is enough.

## Learning order

1. The evaluation loop, starting with error analysis of real outputs (see `layer_10_data.md`).
2. Defining what to measure.
3. Code-based checks.
4. Evaluation in development. Start with a simple harness that runs on every change, and add the rest as your evaluation grows.
5. LLM judges, once you have human labels to check them against, together with human review.
6. Comparing versions, before you trust any small difference.
7. Safety and adversarial testing, before launch.
8. Evaluation in production, from the first release.
9. Experiments with real users, once you have enough traffic.

You can read about public benchmarks at any point. They matter most when you choose models (see `layer_08_foundation_models.md`). Learn each layer's part of evaluation when you work on that layer.

## Key reading

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 3 covers evaluation methods, including AI judges, and chapter 4 covers evaluating AI systems.
- Hamel Husain, "Your AI Product Needs Evals" (2024) and "Creating a LLM-as-a-Judge That Drives Business Results" (2024).
- Shankar et al., "Who Validates the Validators? Aligning LLM-Assisted Evaluation of LLM Outputs with Human Preferences" (2024), on criteria drift and checking judges against people.
- Zheng et al., "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena" (2023), on the biases of LLM judges.
- Eugene Yan, "Evaluating the Effectiveness of LLM-Evaluators (aka LLM-as-Judge)" (2024).
- Ribeiro et al., "Beyond Accuracy: Behavioral Testing of NLP Models with CheckList" (2020).
- Evan Miller, "Adding Error Bars to Evals: A Statistical Approach to Language Model Evaluations" (Anthropic, 2024).
- Kohavi, Tang and Xu, *Trustworthy Online Controlled Experiments* (Cambridge, 2020).
- Liang et al., "Holistic Evaluation of Language Models" (Stanford, 2022), on measuring many criteria across many scenarios.
- Bullwinkel et al., "Lessons From Red Teaming 100 Generative AI Products" (Microsoft, 2025).
- Singh et al., "The Leaderboard Illusion" (2025), on how leaderboard rankings can mislead.
- Your model providers' guides to defining success criteria and building evaluations, and the documentation for one evaluation framework, such as Inspect or promptfoo.
