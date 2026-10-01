# Cost and latency: what an application-layer AI engineer needs to know

This file covers the Cost and latency concern from `ai_stack.md`. This concern keeps the system affordable and fast. It covers cost per request or task, token and GPU spending, and response time, all weighed against quality. It is one of the seven cross-cutting concerns: not a layer, but something every layer needs. This file is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this concern and how deeply to learn each one.

## The rule for depth

Most software costs about the same whether a feature is used once a day or a thousand times. AI features cost money every time they run, and the cost varies widely from one request to the next. A long conversation, a reasoning model or an agent that loops can cost a hundred times more than a short question. AI features are also slow by software standards, because models generate one token at a time and agents run many steps in sequence. And cost and latency trade against quality: a stronger model, more context or more reasoning usually helps, and always costs more.

So the job in this concern is to know what each feature costs and how long it takes, as users experience it, and to find the cheapest and fastest design that still meets the quality bar.

For every AI feature, this concern answers four questions:

1. What does it cost, for the value it delivers? (Foundations, Cost per unit of value)
2. How can it cost less? (Reducing cost)
3. How can it be faster? (Reducing latency)
4. What's the right balance, and how do you keep it? (Trading off against quality, Keeping spending under control)

Most concepts in this concern are level 3, and three things are level 4:
- knowing each feature's full cost per unit of value, such as per resolved support ticket
- knowing which changes save the most money and time for a feature, and making them without losing quality
- choosing each feature's balance of quality, cost and latency

Hardware costs and self-hosting are covered in `layer_09_compute.md`, and how serving engines trade speed for throughput in `layer_06_inference_and_serving.md`. Learn those to the levels those files set.

The layer files cover cost and latency as they apply to each layer, such as context size in Layer 4 and routing and caching in Layer 5. This file covers what they share, and Cost and latency at each layer maps where the rest is covered.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Foundations | 3 | Every use costs money, and costs vary widely between requests |
| Cost per unit of value | 3 (some topics 4) | Price per token says little about what a feature costs |
| Reducing cost | 3 (some topics 4) | A few changes usually save most of the money |
| Reducing latency | 3 (some topics 4) | A few changes usually save most of the time |
| Trading off against quality | 3 (some topics 4) | The cheapest design that meets the quality bar wins |
| Keeping spending under control | 3 | Spending that belongs to nobody grows |
| Reliability | 3 | Cutting cost can cut reliability |
| Governance and safety | 3 | The cheapest route may not be an allowed one |

## Foundations [3]

These ideas come up in every other section, so learn them first.

- [3] Where the money goes. Model tokens are usually the largest cost, but others add up: embedding and indexing, reranking, tool and API calls, sandboxes and browsers, GPU hosting if you run models yourself, stored traces, evaluation runs, labeling, and the time people spend reviewing outputs. Know each one's share for a feature before you try to cut any of them.
- [3] Where the time goes. A request's time splits into network and queueing, the steps before the model starts, such as retrieval and guardrails, the time to the first token, generation, tool calls, and every extra step an agent takes. Output length often dominates (see `layer_06_inference_and_serving.md`), and steps in sequence add up.
- [3] Cost and latency move separately. Some changes cut both, such as caching and shorter outputs. Others cut one and raise the other: running calls in parallel saves time but not money, and batch APIs save money but take hours. More reasoning raises both. Know which one matters more for each feature.
- [3] Cost varies by request. The same feature can cost a cent for one request and dollars for another, depending on the length of the context and the conversation, the reasoning and the number of agent steps. Averages hide this, so look at the distribution, and at the most expensive requests and users (see `concern_02_observability.md`).
- [2] Tokens for different inputs. The same content can need different numbers of tokens in different languages, and images, audio and long documents can cost far more tokens than what the user typed (see `layer_08_foundation_models.md`). Check costs with the inputs your users actually send.

Check: For one feature, break the cost of a typical request into its parts, and its time into its steps. Then find last week's ten most expensive requests, and explain what made each one expensive.

## Cost per unit of value [3, some topics 4]

Price per token is the starting point, not the answer. Cost per task is covered in `layer_08_foundation_models.md` and `layer_02_agents_and_orchestration.md`, and attributing cost to features and customers in `layer_05_model_gateway_and_routing.md`. This section covers judging whether a feature's cost is worth it.

- [4] The unit that matters. For each feature, choose the unit of value, such as a resolved support ticket, an accepted code change or a finished report. Measure the full cost of each unit: every model call, retry, tool call, guardrail check, evaluation sample and minute of human review that went into it. A cheaper model that resolves fewer tickets can cost more per resolved ticket.
- [3] Failures cost too. Wrong answers lead to retries, handoffs to people, support tickets and users who leave. Count those when you compare designs, because the cheaper design may simply move cost from your model bill to your support team.
- [3] Margins. Compare the cost of each user and each customer with what they pay. Heavy users can cost many times the average (see `layer_01_product_and_ux.md`), so check the margin on your heaviest users and on each plan, including free ones, not just on the average user.
- [3] Forecasting. Estimate future cost as volume times tokens per request times price. Check what happens when traffic grows tenfold, conversations get longer or a feature becomes an agent. Long conversations and agents make each request use more tokens, not just add more requests (see `layer_04_context_memory_and_retrieval.md`).
- [3] Estimates against invoices. Costs worked out from token counts drift from what providers actually bill, because of cached and reasoning tokens, price tiers, batch discounts and calls that bypassed your tracking. Compare your figures with each invoice, and look into any gap.
- [2] Prices fall, usage rises. The price of a given level of capability keeps falling (see `layer_08_foundation_models.md`), but new kinds of features, such as reasoning and agents, often raise the tokens per task faster than prices fall. Don't count on cheaper models to fix a feature's margin.

Check: Choose the unit of value for one feature, and compute its full cost per unit for last month, including retries, guardrails and human review. Compare it with the revenue or value each unit brings, and forecast both at ten times today's traffic.

## Reducing cost [3, some topics 4]

Most ways to cut cost are covered in the layer files: smaller context in `layer_04_context_memory_and_retrieval.md`, routing and caching in `layer_05_model_gateway_and_routing.md`, prompt caching in `layer_06_inference_and_serving.md`, fewer tool round trips in `layer_03_tools_and_protocols.md`, distillation in `layer_07_model_adaptation.md`, and batch pricing in `layer_09_compute.md`. This section covers choosing among them.

- [4] The usual order of savings. For most features, the biggest savings come in roughly this order. First, avoid calls you don't need, through caching, simple rules, or not running AI that nobody asked for (see `layer_01_product_and_ux.md`). Then shrink the context, make repeated content cacheable, use a cheaper model for each step that allows it, shorten outputs and limit reasoning, and send work that isn't urgent to batch APIs. Measure quality after each change (see `concern_01_evaluation.md`), and stop when it starts to fall.
- [3] Output costs more than input. Output tokens cost several times as much as input tokens (see `layer_09_compute.md`), and reasoning tokens are billed as output. Ask for concise answers, set output limits, and choose the reasoning effort for each request (see `layer_05_model_gateway_and_routing.md`).
- [3] More than one cache. Besides response caching and prompt caching, cache embeddings so that unchanged documents aren't embedded again, cache tool results that a task requests more than once, and reuse summaries of content that hasn't changed. Each cache needs a rule for when its contents go stale.
- [3] Finding waste. Look for spending that buys nothing: retries of calls that will fail again, tool definitions that are never used, long instructions that a shorter version handles just as well, calls made twice by a double click, and evaluation runs that nobody reads. Start with the most expensive features, customers and requests.
- [2] Bigger changes. Fine-tuning a smaller model or hosting a model yourself saves money only at steady, high volume, once you count their full cost (see `layer_07_model_adaptation.md` and `layer_09_compute.md`). Try the cheaper fixes first.

Check: For one feature, work through the usual order of savings one change at a time, recording cost per unit and quality after each. Stop when quality falls, and report the total saving and which change saved the most.

## Reducing latency [3, some topics 4]

Latency metrics and latency budgets are covered in `layer_06_inference_and_serving.md`, retrieval latency in `layer_04_context_memory_and_retrieval.md`, latency in agents in `layer_02_agents_and_orchestration.md`, and how waiting feels to users in `layer_01_product_and_ux.md`. This section covers choosing among the ways to make a feature faster.

- [4] The usual order of speedups. Start with what runs in sequence, and run independent steps in parallel, such as retrieval and guardrail checks. Then shorten outputs, use a smaller, faster model for steps where users wait, lower the reasoning effort where it doesn't help, and make prompts cacheable to cut the time to the first token. Drop steps that don't earn their time, and move work that needn't happen while the user waits into the background (see `layer_02_agents_and_orchestration.md`).
- [3] Speed users can feel. Streaming the answer and showing progress make a feature feel faster without changing its total time (see `layer_01_product_and_ux.md`). Do this first when the total time is hard to cut, but don't let it hide a feature that's simply too slow.
- [3] The slowest requests. Users remember the slow requests, and agents hit them more often because they make many calls (see `layer_06_inference_and_serving.md`). Look at the 95th and 99th percentiles, find what the slowest requests have in common, and fix that.
- [3] Starting early. Start steps that will probably be needed before you know for sure, such as retrieving documents while a classifier decides whether to search, and cancel them if they turn out not to be needed. This costs more, but saves time.
- [3] Network and payloads. Call providers in regions close to your servers, reuse connections, and keep large inputs small: resize images, and send only the pages of a document the task needs. Smaller inputs also cost fewer tokens.
- [2] Faster generation. For edits and other outputs that are easy to predict, predicted outputs and speculative decoding can make generation much faster (see `layer_06_inference_and_serving.md`).

Check: For one feature, measure each step's share of the total time at the 50th and 95th percentiles. Work through the usual order of speedups, measuring after each change, and report how far the 95th percentile fell and what each change cost.

## Trading off against quality [3, some topics 4]

Comparing versions on quality, cost and latency together is covered in `concern_01_evaluation.md`, and setting latency targets in `concern_05_reliability.md`. This section covers choosing the balance.

- [4] A target for each of the three. Give each feature a quality bar, a latency target and a cost budget per unit of value, and choose the cheapest design that meets the quality bar within the latency target. The balance differs by feature: autocomplete is mostly about speed, overnight document tagging about cost, and contract review about quality.
- [3] Spending more where it pays. Sometimes a stronger model, more reasoning or several attempts are worth what they cost, such as on the hard cases a cascade passes up (see `layer_05_model_gateway_and_routing.md`). Measure what each extra dollar or second buys in quality, and spend where the gain is largest.
- [3] Different balances for different uses. Free and paid users, background jobs and live requests, and small and large customers can each get a different balance, such as a cheaper model on the free plan (see `layer_01_product_and_ux.md`). Make sure each one still meets the quality bar for its use.
- [3] Checking again as models change. New models and price cuts change the best balance every few months (see `layer_08_foundation_models.md`). Re-run the comparison regularly, and switch when a new design meets the bar for less.

Check: For one feature, set its quality bar, latency target and cost budget. Compare three designs on all three, choose one, and write down what change in prices or models would make you choose differently.

## Keeping spending under control [3]

Budgets, spending caps and protection against runaway use are enforced in the gateway (see `layer_05_model_gateway_and_routing.md`), and alerts on cost are covered in `concern_02_observability.md`. This section covers the habits around them.

- [3] Cost in every change. Have every change to prompts, models, tools or retrieval report its effect on cost and latency per request on the evaluation set, next to its effect on quality (see `concern_01_evaluation.md`). Require approval for any change that raises cost past a set limit.
- [3] An owner for each budget. Give each feature's spending an owner who looks at its cost every week, explains changes and decides what to cut. Spending that belongs to nobody grows.
- [3] Limits for users. Plan limits, usage-based pricing and caps for each user keep heavy users from costing more than they pay (see `layer_01_product_and_ux.md`). Set them with product and finance, and revisit them as costs change.
- [2] Buying better. Once spending is large, ask providers about volume discounts, committed-spend agreements and reserved capacity (see `layer_09_compute.md`). Weigh what you save against the flexibility you give up.

Check: Add cost and latency per request to your evaluation reports, and hold back one change that raised cost past your limit until someone approves it. Name an owner for each feature's spending, and hold one weekly review.

## Cost and latency at each layer

Each layer file covers cost and latency as they apply to that layer. This table shows where.

| Layer | What to make cheaper or faster there | Level | Where |
|---|---|---|---|
| 1. Product and UX | Speed for each kind of interaction, cost per user, limits users understand, AI nobody asked for, precomputing | 3 | `layer_01_product_and_ux.md` |
| 2. Agents and orchestration | Cost per task, budgets the agent knows about, a model for each step, steps in sequence | 3 | `layer_02_agents_and_orchestration.md` |
| 3. Tools and protocols | Round trips, tool definitions and results in the context, slow tools, sandbox and API costs | 3 | `layer_03_tools_and_protocols.md` |
| 4. Context, memory and retrieval | Context size, conversation history that grows, retrieval latency, indexing and storage costs | 3 (some topics 4) | `layer_04_context_memory_and_retrieval.md` |
| 5. Model gateway and routing | Routing, cascades, caching, reasoning effort for each request, cost attribution, budgets and caps | 3 (some topics 4) | `layer_05_model_gateway_and_routing.md` |
| 6. Inference and serving | Time to first token, output speed, percentiles, latency budgets, prompt caching, faster generation | 3 | `layer_06_inference_and_serving.md` |
| 7. Model adaptation | When adapting a model pays off, how training is billed, distillation | 3 | `layer_07_model_adaptation.md` |
| 8. Foundation models | How models are priced, cost per task, prompt caching, latency by model, choosing models | 3 (choosing models 4) | `layer_08_foundation_models.md` |
| 9. Compute | Cost per token, self-hosting versus APIs, utilization, batch and reserved capacity | 3 | `layer_09_compute.md` |
| 10. Data | Costs of labeling, synthetic data and fine-tuning runs, the smallest dataset that works | 2–3 | `layer_10_data.md` |

Check: For one feature, list every layer it touches and the cost and latency levers that layer's file names. Mark which you've used, and try the most promising one you haven't.

## Other cross-cutting concerns in cost and latency

### Reliability [3]

- [3] Cutting cost can cut reliability. Removing a fallback, shrinking spare capacity or dropping retries saves money until the next outage. Check each cut against the feature's reliability target (see `concern_05_reliability.md`).

### Governance and safety [3]

- [3] Cheaper routes must still be allowed. The cheapest provider or region may not be approved for your data (see `layer_05_model_gateway_and_routing.md` and `concern_04_governance_and_safety.md`). Routing for cost has to stay within the approved list.

The other concerns add cost and latency of their own, and each file covers its share: evaluation runs in `concern_01_evaluation.md`, storing traces in `concern_02_observability.md`, security checks and attackers who run up your bill in `concern_03_security.md`, guardrails in `concern_04_governance_and_safety.md`, and retries, fallbacks and spare capacity in `concern_05_reliability.md`.

Data quality meets cost in what you store and search: duplicate and outdated documents cost embedding, storage and context tokens while making answers worse (see `layer_04_context_memory_and_retrieval.md`).

## Not in this concern

- Each layer's own levers, such as context size or routing, are covered in each layer's file (see Cost and latency at each layer).
- Hardware costs, utilization and self-hosting versus APIs are covered in `layer_09_compute.md`, and how serving engines trade speed for throughput in `layer_06_inference_and_serving.md`.
- Enforcing budgets, spending caps and protection against runaway use is covered in `layer_05_model_gateway_and_routing.md`.
- Measuring cost and latency and alerting on them are covered in `concern_02_observability.md`, and comparing versions on quality, cost and latency in `concern_01_evaluation.md`.
- Latency targets and what reliability costs are covered in `concern_05_reliability.md`.
- Pricing the product itself is a product and finance decision. This file covers the costs that the pricing has to cover.

## Learning order

1. Foundations.
2. Cost per unit of value.
3. Reducing cost.
4. Reducing latency.
5. Trading off against quality.
6. Keeping spending under control, before costs grow large.

Learn each layer's levers when you work on that layer. Most of them are in Layers 4, 5, 6 and 8.

## Key reading

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 9 covers inference optimization, and chapter 10 covers routers, gateways and caches.
- Chen, Zaharia and Zou, "FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance" (2023).
- Kapoor et al., "AI Agents That Matter" (2024), on comparing agents by cost as well as accuracy.
- Databricks, "LLM Inference Performance Engineering: Best Practices" (2023).
- Guido Appenzeller, "Welcome to LLMflation" (Andreessen Horowitz, 2024), on how quickly the price of a given capability falls.
- Artificial Analysis (artificialanalysis.ai), for independent comparisons of models' quality, price and speed.
- The FinOps Foundation, "FinOps for AI Overview" (finops.org), on managing AI spending.
- Your model providers' pricing pages, and their guides to prompt caching, batch processing and reducing latency.
