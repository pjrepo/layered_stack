# Observability: what an application-layer AI engineer needs to know

This file covers the Observability concern from `ai_stack.md`. Observability shows what the system is actually doing. It uses traces that follow a request through agents, tools and model calls, plus logs and metrics for finding and fixing failures. It is one of the seven cross-cutting concerns: not a layer, but something every layer needs. This file is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this concern and how deeply to learn each one.

## The rule for depth

Ordinary observability asks whether a service is up, fast and free of errors. That still matters for AI features, but they usually fail another way: the request succeeds, quickly, and the answer is wrong. The model misread a document, retrieval found an outdated version, a tool returned an error the model ignored, or an agent reported success it didn't achieve. None of these raises an error. You see them only if each request records what went in, what happened at each step and what came out, and if that record links to the signs that something went wrong, such as a complaint, a thumbs-down or a failed evaluation.

Observability answers three questions:

1. What exactly happened on this request? (Tracing, What to record)
2. Is the system healthy, and is anything getting worse? (Metrics and dashboards, Alerts)
3. Why did something go wrong, and how many other requests did the same? (Debugging from traces)

You build and instrument the application yourself, so most concepts in this concern are level 3, and two things are level 4:
- deciding what every trace records, including the IDs and versions that let you explain and reproduce any result
- tracing a bad result back to its cause

Your company probably already has observability tools and a team that runs them. What changes with AI is mostly what you record and how you read it. So learn the pipeline behind the tools, such as collectors and storage, only to level 2. Learn infrastructure metrics, such as GPU and serving-engine metrics, to level 2 unless you self-host (see `layer_06_inference_and_serving.md` and `layer_09_compute.md`).

The layer files cover observability as it applies to each layer, such as logging the assembled context in Layer 4 and tracing tool calls in Layer 3. This file covers what they share, and Observability at each layer maps where the rest is covered.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Foundations | 3 | AI features fail without raising errors |
| Tracing | 3 | A trace follows one request through agents, tools and model calls |
| What to record | 4 | You can't recover what you didn't record at the time |
| Metrics and dashboards | 3 | Metrics show that something changed, and where |
| Alerts | 3 | Someone should know before users complain |
| Debugging from traces | 3 (some topics 4) | Most causes are visible in the trace once you look |
| Observability tools | 2 (one tool 3) | Tools change quickly, and the practices behind them don't |
| Security | 3 | Traces hold secrets and private conversations |
| Governance and safety | 3 | Traces copy personal data out of your product, often to another company |
| Cost and latency | 3 | Full traces are expensive to keep |
| Reliability | 2–3 | Recording must never break the product |

## Foundations [3]

These ideas come up in every other section, so learn them first.

- [3] Wrong answers don't raise errors. Most AI failures come back as successful requests: a confident wrong answer, an ignored instruction, a tool called with the wrong arguments, or an agent that stopped early. Error rates and latency miss them all. To see them, record the content of each request, not just its status, and attach signs of quality such as user feedback and evaluation scores (see `concern_01_evaluation.md`).
- [3] Traces, logs and metrics. A trace follows one request through every step. Each step is a span, with a start and end time, a status, details called attributes, and a parent span. Logs record single events. Metrics are numbers aggregated over time, such as request counts, error rates and latency percentiles. Metrics show that something changed. Traces and logs show what happened.
- [3] One wide record per step. Record each step as one structured event with many fields, such as IDs, versions, token counts, cost, latency and outcome, rather than as scattered lines of text. Structured records can be filtered and grouped by any field, like rows in a database.
- [3] Asking new questions. Dashboards and alerts answer questions you thought of in advance. AI failures are often new, such as "which answers this week quoted the old refund policy?" Record rich, searchable data, so you can answer questions nobody planned for. This is the difference between monitoring and observability.
- [2] Standard service signals. The usual health signals for any service still apply: how many requests arrive, how many fail and how long they take, plus how close resources are to their limits. Google's four golden signals and the RED and USE methods are common checklists.

Check: For one AI feature, list the ways it could fail without raising an error. For each, name the recorded field or signal that would reveal it, and add whatever is missing.

## Tracing [3]

A trace follows one request through agents, tools and model calls. Tracing whole agent runs is covered in `layer_02_agents_and_orchestration.md`, and tracing tool calls in `layer_03_tools_and_protocols.md`. This section covers how tracing works for any AI feature.

- [3] Spans for AI steps. Give each kind of step its own span: the incoming request, each agent step, model calls, retrieval, embedding, reranking, tool calls and guardrail checks. Name spans for what they do, and nest them the way the work is nested, so a trace reads as a tree.
- [3] Passing the trace along. The trace ID has to travel with the request across services, queues and background workers, usually in a standard header defined by W3C Trace Context. Without it, the records kept by your API, your gateway (see `layer_05_model_gateway_and_routing.md`) and your workers can't be joined into one trace. Store the trace ID with each long-running task's state, so later steps join the same trace.
- [3] Conversations and users. Group traces into sessions, such as a conversation or an agent task, and tag each trace with user and customer IDs. Then you can open everything in one conversation, or everything one customer did last week.
- [3] OpenTelemetry. OpenTelemetry is the open standard for traces, metrics and logs. It has SDKs for most languages and a collector that sends data to any backend. Its conventions for generative AI give standard names for model calls, token counts, tool calls and agents, and are still changing. OpenInference and OpenLLMetry are related sets of conventions and libraries. Instrumenting to a standard lets you switch tools later.
- [3] Automatic and manual instrumentation. Libraries can trace model SDKs, agent frameworks and vector databases with little code. Add your own spans around your own logic. Check what automatic tracing leaves out: many libraries skip prompt and output text by default, for privacy, and some miss streamed output or reasoning tokens.
- [3] Streaming and long runs. For a streamed response, record when the first token arrived, and record the full output when the stream ends, including streams that were cut off or cancelled. Agent runs can last hours and produce very large traces. Send spans as each step finishes, so a crash doesn't lose the whole trace.
- [3] Logs in the trace. Put the trace ID on every log line, so logs from any service can be pulled up next to the trace they belong to.
- [2] Sampling. At high volume, you may keep only some traces. Deciding at the start of each request (head sampling) is cheap, but keeps a random mix. Deciding after it ends (tail sampling) lets you keep every error, slow or costly request, and thumbs-down. Many AI teams keep every trace for low-volume features, since each one is possible evaluation data.

Check: Trace one feature end to end, from the user's request through the gateway, retrieval, model calls and tool calls, including one step that runs in a background worker. Show the whole request as a single tree, open it from the conversation it belongs to, and find its log lines by trace ID.

## What to record [4]

The layer files list what to record at each step: each model call in `layer_08_foundation_models.md`, the assembled context in `layer_04_context_memory_and_retrieval.md`, tool calls in `layer_03_tools_and_protocols.md`, routes, retries and cost in `layer_05_model_gateway_and_routing.md`, and what users do with each result in `layer_01_product_and_ux.md`. This section covers the standard that ties them together.

- [4] One standard for every call. Decide once, for the whole team, what every model call and step records, and enforce it in the gateway or a shared client library (see `layer_05_model_gateway_and_routing.md`) rather than feature by feature. A field that one feature skipped is missing exactly when you need it.
- [4] Versions on everything. Record the version of everything that shaped a result: the prompt, the model snapshot, the retrieval index and embedding model, the tool definitions, the configuration and feature flags, and the code release. Without versions, you can't tell which change caused a problem, and you can't rebuild what the model saw.
- [4] Enough to replay. Record each step's full input and output, or enough to rebuild them exactly. The test is whether you can rerun a failed request with one thing changed (see `layer_02_agents_and_orchestration.md`). If you can't, the record is missing something.
- [3] IDs that connect everything. Every trace needs its trace ID, its session or task ID, and user and customer IDs. Everything that refers to a request, such as feedback, support tickets, evaluation scores and audit records, should carry the request's trace ID. A thumbs-down that can't be tied to its trace tells you almost nothing (see `layer_01_product_and_ux.md`).
- [3] What happened next. Record outcomes as well as inputs: what the user did with the result, guardrail verdicts, evaluation scores (see `concern_01_evaluation.md`), errors, retries and fallbacks, and whether the task finished.
- [3] Content and metadata apart. Full prompts and outputs are large and hold user data. Many teams keep metadata, such as IDs, versions, tokens, cost and latency, in the tracing tool, and store full content separately, with tighter access and shorter retention, linked by ID. Decide for each feature what content you may keep, and for how long (see Governance and safety below).

Check: Write the standard record for one product's model calls and agent steps, and enforce it in the gateway or client library. Then take a failed request recorded under the standard. Show exactly what the model saw, which versions produced the result and what the user did next, and rerun it with one change.

## Metrics and dashboards [3]

Metrics show the whole system over time, and dashboards put the ones that matter in front of people.

- [3] Tokens, cost and speed. Track token counts, including cached and reasoning tokens (see `layer_08_foundation_models.md`), cost per request, task, user and feature (see `layer_05_model_gateway_and_routing.md`), time to first token and output speed (see `layer_06_inference_and_serving.md`), and cache hit rates.
- [3] How calls end. Track stop reasons, such as outputs cut off at the length limit and refusals, along with tool errors, retries, fallbacks and guardrail blocks. Add signs of quality, such as evaluation pass rates and user feedback (see `concern_01_evaluation.md`).
- [3] Breaking down by what changes. Split every metric by feature, model, provider, prompt version and customer. AI problems usually start in one slice: one prompt version, one provider or one customer's documents. A total can look normal while one slice fails.
- [3] Distributions, not averages. Averages hide the slow and the expensive. Track latency and cost per request as percentiles, such as p50, p95 and p99. A few runaway agent runs can cost more than thousands of ordinary chats.
- [3] Dashboards people use. Keep a few dashboards with clear owners: one for each feature, showing traffic, errors, latency, cost and quality together, one for models and providers, and one for spending. Mark releases, prompt changes and model changes on the charts, so you can see what changed when a line moves.
- [2] Cardinality. Metric systems handle labels with a limited number of values, such as model or feature names. Labels with millions of values, such as user IDs or prompt text, make them slow and expensive. Keep that detail in traces, and aggregate it in metrics.

Check: Build one dashboard for a feature that shows traffic, errors, p95 latency, cost per request and a quality signal, broken down by prompt version and model, with releases marked. Use it to explain one change you see on it.

## Alerts [3]

Alerts tell someone to act. Setting targets for uptime and latency, called service level objectives (SLOs), is covered in `concern_05_reliability.md`, and alerts on quality in `concern_01_evaluation.md`. This section covers measuring service levels and alerting well.

- [3] Measuring service levels. Measure service level indicators, such as the share of requests that succeed within a time limit, the way users experience them. For AI features, a request that ended in a fallback, was cut off at the length limit or took 40 seconds may count as failed, even though it raised no error.
- [3] Alerting on symptoms. Alert on what users feel, such as errors, slowness, falling quality and cost spikes, rather than on every possible cause. Give each alert an owner and a short runbook: what to check first and what to do. Alerts that often fire without needing action teach people to ignore all alerts.
- [3] Alerts for AI features. Common ones are cost or tokens per request jumping (a looping agent, or a prompt change that doubled the context), more outputs cut off at the length limit, more refusals, tool errors or fallbacks, a falling prompt cache hit rate (often a prompt change that broke caching, see `layer_06_inference_and_serving.md`), and usage nearing rate limits (see `layer_05_model_gateway_and_routing.md`).
- [3] Watching providers. A provider's outage, slowdown or model change is your incident too. Measure each provider's and model's errors and latency from your side instead of relying on status pages, and send small test requests on a schedule, to catch problems before users do.
- [2] Alerting on burn rate. An SLO allows some failures, called the error budget. Instead of alerting on every brief spike, alert when failures are using up that budget too fast. This catches real problems quickly and ignores noise.
- [2] Unusual changes. For metrics with no fixed normal level, such as daily cost or the mix of topics users ask about, alert on unusual changes compared with history. Tune these alerts, or they'll fire constantly.

Check: For one feature, define two service level indicators measured from the user's side, and set up alerts on them and on cost per request, each with a runbook. Trigger each alert in a test environment and follow its runbook.

## Debugging from traces [3, some topics 4]

Traces are how you find and fix failures. Reading many traces to find the common kinds of failure (error analysis) is covered in `layer_10_data.md`, and measuring which step of a retrieval pipeline failed in `layer_04_context_memory_and_retrieval.md`. This section covers investigating failures.

- [4] From a bad result to its cause. Walk the trace step by step. Was the needed information in the context (see `layer_04_context_memory_and_retrieval.md`)? Did each tool return what it should have? Did the model follow its instructions, and did it stop for the reason you expected? Most causes are visible in the trace once you look, and often the model isn't the cause at all.
- [3] Every signal leads to a trace. User reports, support tickets, alerts and failed evaluations should each open the traces behind them in one click. If finding the trace takes an hour, it won't get done.
- [3] Searching traces. Find traces by any field: user, customer, feature, version, stop reason, cost, latency, tool error, feedback, evaluation result, or text in the input or output. Once you understand one bad trace, searching tells you how many others share its cause.
- [3] Finding patterns. Group many traces to see what they share: cluster requests by topic with embeddings, group failures by step or error, and compare traces from before and after a change. Some tools use a model to summarize or label traces at scale. Check its labels as you would any judge's (see `concern_01_evaluation.md`).
- [3] Reproducing a failure. Rerun the recorded request with the same inputs, then with one thing changed, to confirm a fix on the exact case that failed (see `layer_02_agents_and_orchestration.md`). Outputs vary between runs, so rerun it several times before you conclude anything.
- [3] Incidents. When something breaks in production, establish when it started, what changed at that time (a release, a prompt or model version, a provider incident or a document sync), and who's affected. Write a short review afterwards, and add the failing cases to your evaluation set (see `layer_10_data.md`).

Check: Take three real complaints or failed evaluations. For each, find the trace in under a minute, name the step that caused the failure and count how many other requests share the cause. Rerun one with a fix, and show that it passes several times.

## Observability tools [2, one tool 3]

Observability tools change quickly. Learn what they do first, then learn one of them well.

- [3] LLM observability platforms. Tools such as Langfuse, LangSmith, Arize Phoenix, Braintrust, Helicone and W&B Weave show model calls, traces, sessions, costs and feedback, and connect them to evaluation and datasets. Learn one well, including searching traces and exporting them.
- [2] General observability tools. Your company probably already runs a general tool, such as Datadog, Grafana, Honeycomb or New Relic, and many of these now have AI features. Where you can, send AI traces to the same place as the rest of your telemetry, so you can follow a request from the browser through the AI steps to the database.
- [2] How data gets in. You can trace with an SDK inside your code, or log every model call at a proxy or gateway. A proxy needs no code changes, but it sees only model calls, not retrieval, tools or your own logic. Many teams use both.
- [2] The pipeline behind the tools. Telemetry passes from your code through collectors to storage and query engines, and it can be dropped, delayed or sampled at each stage. Platform teams usually run this pipeline. Know where your data flows, so you can tell when some of it is missing.
- [2] Choosing. Weigh OpenTelemetry support, whether you can host the tool yourself (a hosted tool receives every prompt and output), cost at your volume, search over full content, evaluation features, retention and access controls.

Check: Instrument one feature with OpenTelemetry and send its traces to an LLM observability tool. Switch to a second tool by changing only the export configuration, and list what each shows that the other doesn't.

## Observability at each layer

Each layer file covers observability as it applies to that layer. This table shows where.

| Layer | What to observe there | Level | Where |
|---|---|---|---|
| 1. Product and UX | What users experienced in the client, and links from complaints and feedback to traces | 3 | `layer_01_product_and_ux.md` |
| 2. Agents and orchestration | Whole runs traced as trees, reading runs, task status, replaying runs | 3 | `layer_02_agents_and_orchestration.md` |
| 3. Tools and protocols | Every tool call in the trace, audit logs of actions, tool usage, redaction | 3 | `layer_03_tools_and_protocols.md` |
| 4. Context, memory and retrieval | The assembled context of every call, retrieval in production, index health | 3 | `layer_04_context_memory_and_retrieval.md` |
| 5. Model gateway and routing | Routes, retries, fallbacks, cache hits, cost tags and trace IDs, recorded at the gateway | 3 | `layer_05_model_gateway_and_routing.md` |
| 6. Inference and serving | Time to first token and output speed as percentiles, serving-engine metrics | 2–3 | `layer_06_inference_and_serving.md` |
| 7. Model adaptation | Training metrics next to real outputs, experiment tracking | 2 | `layer_07_model_adaptation.md` |
| 8. Foundation models | What to log for each model call: model, snapshot, settings, tokens, stop reason, latency | 3 | `layer_08_foundation_models.md` |
| 9. Compute | GPU metrics | 2 | `layer_09_compute.md` |
| 10. Data | Full traces tied to feedback, turning traces into datasets, drift in what users ask | 3 (some topics 4) | `layer_10_data.md` |

Check: For one feature, list every layer it touches and what its traces record at each. Mark what's missing, and add the most important gap.

## Other cross-cutting concerns in observability

### Security [3]

- [3] Secrets in telemetry. Prompts, tool arguments and tool results can contain API keys, passwords and tokens. Remove them before they're recorded, because traces get copied to vendors, exported and kept for months (see `layer_03_tools_and_protocols.md`).
- [3] Who can read traces. Traces hold user conversations and customer data. Limit who can read full content, log who viewed it, and keep each customer's data separate in shared tools.
- [3] Hostile content in trace viewers. Recorded inputs and outputs can contain attacks, such as scripts, malicious links or injected instructions. Trace viewers should show them as plain text, and models that summarize traces should treat them as data (see `layer_01_product_and_ux.md`).
- [2] Spotting attacks. Traces and metrics reveal abuse, such as one key sending far more requests than usual, repeated jailbreak attempts or sudden jumps in cost. Defenses are covered in `concern_03_security.md`.

### Governance and safety [3]

- [3] Personal data in traces. Traces copy user data out of your product, often into another company's tool. Mask personal data you don't need when you record it, limit how long you keep it, and make deletion requests reach traces and exported copies too (see `layer_10_data.md`).
- [3] Where traces go. Your observability vendor receives everything you send it. Check where it stores data, for how long and who can see it, as you would for a model provider. A zero-data-retention agreement with a model provider means little if the same prompts sit in your trace store for a year.
- [3] Traces are not audit logs. Traces are for debugging, so they may be sampled, trimmed and deleted early. Audit logs are for accountability, so they must be complete, unchangeable and kept longer (see `layer_03_tools_and_protocols.md`). Audit trails are covered in `concern_04_governance_and_safety.md`.
- [2] Looking without reading. Models can summarize and cluster conversations with personal details removed, so you can see how the product is used without people reading raw conversations.

### Cost and latency [3]

- [3] What observability costs. Storing full prompts and outputs for every call adds up fast, and some tools charge by the span or by the gigabyte. Control it with sampling, shorter retention for full content, and cheaper storage for older data.
- [3] Off the request path. Send telemetry in the background, in batches, so recording never slows down a user's request. Measure what tracing adds to latency.

Measuring cost and latency is covered under Metrics and dashboards.

### Reliability [2–3]

- [3] Recording must not break the product. If the tracing tool is down or slow, requests should carry on without it. Buffer telemetry, and drop it rather than block or fail a request.
- [2] Seeing during outages. Your observability has to keep working when a provider or region fails, because that's when you need it most. Check that you could watch your fallbacks work (see `layer_05_model_gateway_and_routing.md`) during a provider outage.

Evaluation uses traces as its raw material, and its scores become fields on each trace. Both are covered in `concern_01_evaluation.md`.

Data quality in observability means complete records: a trace with missing fields or dropped spans can mislead more than no trace at all. Turning traces into datasets is covered in `layer_10_data.md`.

## Not in this concern

- What to record at each layer, such as the assembled context or each tool call, is covered in each layer's file (see Observability at each layer).
- Scoring outputs, evaluation in production and alerts on quality are covered in `concern_01_evaluation.md`. This file covers recording, storing and searching what they score.
- Error analysis, turning traces into datasets and detecting drift in what users ask are covered in `layer_10_data.md`.
- Product metrics and recording what users do with each result are covered in `layer_01_product_and_ux.md`.
- Cost attribution, budgets and spending caps are covered in `layer_05_model_gateway_and_routing.md`.
- Setting uptime and latency targets, retries and fallbacks are covered in `concern_05_reliability.md`. This file covers measuring them.
- Audit trails are covered in `concern_04_governance_and_safety.md`, and defenses against attacks in `concern_03_security.md`.
- Training metrics and experiment tracking are covered in `layer_07_model_adaptation.md`, and GPU and serving-engine metrics in `layer_06_inference_and_serving.md` and `layer_09_compute.md`.
- Running the observability pipeline itself, such as collectors and storage clusters, is platform work. Knowing how data flows through it is enough.

## Learning order

1. Foundations, especially why AI failures don't raise errors.
2. Tracing, starting with one feature end to end.
3. What to record, as a standard for your whole team.
4. Debugging from traces. Start reading traces from the first day.
5. Metrics and dashboards.
6. Alerts, before you go on call for an AI feature.
7. Observability tools. Learn one well, then the pipeline behind it.

Learn each layer's part of observability when you work on that layer.

## Key reading

- Charity Majors, Liz Fong-Jones and George Miranda, *Observability Engineering* (O'Reilly, 2022).
- Cindy Sridharan, *Distributed Systems Observability* (O'Reilly, 2018).
- Sigelman et al., "Dapper, a Large-Scale Distributed Systems Tracing Infrastructure" (Google, 2010).
- Google, *Site Reliability Engineering* (2016), chapter 6 on monitoring distributed systems, and *The Site Reliability Workbook* (2018), chapter 5 on alerting on SLOs.
- Rob Ewaschuk, "My Philosophy on Alerting" (2013), written from his time as a site reliability engineer at Google.
- The OpenTelemetry documentation, including its semantic conventions for generative AI.
- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 10 covers monitoring and observability for AI applications.
- Hamel Husain, "A Field Guide to Rapidly Improving AI Products" (2025), on looking at your data and building simple tools to read traces.
- Tamkin et al., "Clio: Privacy-Preserving Insights into Real-World AI Use" (Anthropic, 2024), on finding patterns across many conversations without people reading them.
- The documentation for one LLM observability tool, such as Langfuse or Arize Phoenix.
