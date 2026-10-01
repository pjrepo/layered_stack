# Reliability: what an application-layer AI engineer needs to know

This file covers the Reliability concern from `ai_stack.md`. Reliability keeps the system working when parts fail. It covers retries, switching between models and providers, uptime targets, falling back to reduced functionality instead of failing outright, and keeping model outputs consistent. It is one of the seven cross-cutting concerns: not a layer, but something every layer needs. This file is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this concern and how deeply to learn each one.

## The rule for depth

An AI feature depends on more outside services than most software: model providers, a gateway, search, embedding models, tools, MCP servers and sandboxes. Each can fail, slow down or return something malformed. Model APIs are young and often run close to capacity, so slowdowns and overload errors are common. On top of that, the model's output varies: the same request can come back different, cut off or in the wrong format. Reliability means the product keeps working through all of this. It retries what's worth retrying, switches to another model or provider, does less when it must, and keeps outputs consistent where users depend on them.

For every AI feature, reliability answers four questions:

1. How reliable must it be? (Foundations, Service levels)
2. What happens when a part fails? (Retries and fallbacks across the system, Graceful degradation, Timeouts and overload)
3. Can users depend on what it produces? (Consistent outputs)
4. How do you keep it working as things change? (Changes without outages, Testing for failure, Incidents and recovery)

Most concepts in this concern are level 3, and two things are level 4:
- setting reliability targets for each AI feature, from the user's point of view
- planning the steps each feature takes as parts fail, from full function down to a clear message

The mechanics, such as retries with backoff, circuit breakers and timeouts for single calls, are covered in the layer files, mainly in Layer 5. Site reliability engineering is a discipline of its own, and your company may have teams that run on-call, incident processes and infrastructure. Learn their practices to level 3 as they apply to AI features.

The layer files cover reliability as it applies to each layer, such as fallbacks between models in Layer 5 and resuming agent tasks after crashes in Layer 2. This file covers what they share, and Reliability at each layer maps where the rest is covered.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Foundations | 3 | AI features depend on many services that fail |
| Service levels | 3 (some topics 4) | Each feature needs its own target, set from the user's side |
| Retries and fallbacks across the system | 3 | Retries and fallbacks in several layers can make failures worse |
| Graceful degradation | 3 (some topics 4) | Doing less beats doing nothing |
| Timeouts and overload | 3 | One slow step can use up the time and capacity others need |
| Consistent outputs | 3 | Users can't depend on answers that change or break format |
| Changes without outages | 3 | Many AI incidents start with a change |
| Testing for failure | 3 | Fallbacks that never run break unnoticed |
| Incidents and recovery | 3 | Some failures will reach users |
| Cost and latency | 3 | Reliability costs money and time |
| Security | 3 | Some checks must stop the request when they fail |
| Governance and safety | 3 | Fallbacks must follow the same data rules |

## Foundations [3]

These ideas come up in every other section, so learn them first.

- [3] Every dependency can fail. List what each feature depends on: model providers, the gateway, search and embedding services, tools and the APIs behind them, MCP servers, sandboxes, queues and your own services. Each can be down, slow, rate-limited or wrong. Failures multiply along a chain: five dependencies that each work 99.9% of the time all work together only about 99.5% of the time. Agent steps compound the same way (see `layer_02_agents_and_orchestration.md`).
- [3] Kinds of failure. A call can fail outright, with an error or a timeout, or succeed too slowly to be useful, which users experience as failure. It can fail partway, such as a stream that breaks or an answer cut off at the length limit (see `layer_06_inference_and_serving.md`). It can succeed with output that's malformed or wrong, which raises no error (see `concern_02_observability.md`). And behavior can drift when a provider changes the model behind a name you use (see `layer_08_foundation_models.md`).
- [3] The model is a flaky dependency. Treat every model response like a reply from an unreliable outside service. Before anything uses it, check that it arrived, that it ended for the expected reason (see stop reasons in `layer_08_foundation_models.md`), that it parses and that it follows your rules.
- [3] Containing failures. Keep one failing feature, provider or customer from taking down the rest. Give features and customers their own limits, queues and keys (see `layer_05_model_gateway_and_routing.md`), so a runaway agent or a traffic spike in one place can't use up capacity everywhere.
- [2] Reliability costs money. Each step up in reliability costs more: a second provider, reserved capacity, duplicate requests and more testing. Decide for each feature how much reliability is worth paying for.

Check: For one AI feature, list every dependency, how each can fail and what the user sees when it does. Estimate the feature's availability from its dependencies, and name the failure that would hurt users most.

## Service levels [3, some topics 4]

A service level objective (SLO) is a target for how reliable and fast a feature must be, such as "99.5% of requests succeed within 8 seconds, measured over 28 days". Measuring service levels and alerting on them are covered in `concern_02_observability.md`. This section covers setting the targets.

- [4] A target for each feature. Set targets for each AI feature from the user's point of view, based on what users and the rest of the product depend on. Inline autocomplete can fail quietly now and then, but an agent that processes refunds can't. Cover speed as well as success, and write the targets down.
- [3] What counts as success. Decide which responses count as good: complete, valid and within the time limit. Decide too how to count a degraded response, such as an answer from a fallback model or a cached result. Counting it as a success hides problems, and counting it as a failure raises alarms for a system working as designed, so many teams track degraded responses separately.
- [3] Error budgets. A target of 99.5% allows 0.5% of requests to fail, and that allowance is called the error budget. When failures use it up, slow down risky changes, such as switching models or rewriting prompts, and spend the time on reliability instead. Agree on this rule with product before you need it.
- [3] Your providers set a ceiling. Without a fallback, a feature can't be more reliable than the providers it depends on. Know their published availability, any service level agreement they offer and their incident history. Standard API plans often promise little, and paid tiers or reserved capacity may promise more (see `layer_09_compute.md`).
- [3] Independent fallbacks raise the ceiling. If two providers each fail 0.5% of the time and their failures are independent, both fail at once only 0.0025% of the time. Real failures are often linked, such as when both run in the same cloud region or depend on the same upstream service, so choose fallbacks that fail independently.
- [2] Promises to customers. A service level agreement (SLA) is a promise in a contract, often with credits when it's broken. Set it below your internal target, so you can miss the target now and then without breaking the contract.

Check: Set SLOs for two AI features with different needs, covering success and speed, with a rule for counting degraded responses. Compare them with what your providers promise, and show how much a fallback raises the target you can reach.

## Retries and fallbacks across the system [3]

Retries, circuit breakers and fallbacks for model calls are covered in `layer_05_model_gateway_and_routing.md`, retries inside tools in `layer_03_tools_and_protocols.md`, and fallbacks for agents in `layer_02_agents_and_orchestration.md`. This section covers how they work together.

- [3] Retrying in one place. Provider SDKs often retry by default, and so may your gateway, your agent framework and your own code. If three layers each make up to three attempts, one failure can become 27 calls, and an outage turns into a flood. Decide which layer retries each kind of call, and turn retries off in the others.
- [3] Limits on retrying. Cap retries for each request, and across the system, such as allowing retries to add no more than 10% to traffic, so that retries can't make an outage worse. Back off between attempts, with random jitter (see `layer_05_model_gateway_and_routing.md`).
- [3] Safe to retry. Retry only what's safe to repeat. A model call is safe until its output has been shown or acted on. A tool call that changes something needs an idempotency key (see `layer_03_tools_and_protocols.md`). An agent task should resume from its last checkpoint rather than start over (see `layer_02_agents_and_orchestration.md`).
- [3] Fallbacks need capacity. When you fail over to another provider, all your traffic arrives there at once. If your rate limits or quota there were sized for a trickle, the failover causes a second outage. Size the fallback's limits for your peak traffic, and keep it warm with a small share of real traffic.
- [3] Fallbacks need testing. A fallback model, with its own prompt, behaves differently from the main one. Run the evaluation set on every fallback path (see `concern_01_evaluation.md`), and test the switch itself regularly (see Testing for failure).

Check: Trace one request's retry behavior through every layer, from the SDK through the gateway to your own code, and remove the duplicate retries. Then fail over one feature's peak traffic to its fallback provider in a test, and show that the fallback's limits hold and its quality on your evaluation set is acceptable.

## Graceful degradation [3, some topics 4]

When parts fail, a feature should do less rather than nothing. What the rest of the product does when AI fails is covered in `layer_01_product_and_ux.md`, fallbacks between models in `layer_05_model_gateway_and_routing.md`, and what to do when search fails in `layer_04_context_memory_and_retrieval.md`. This section covers planning the steps.

- [4] Steps from full to off. For each feature, decide in advance the steps it takes as things fail. A typical order is the main model, then a fallback model, then a cached or precomputed answer, then a simpler path that doesn't generate text, such as showing search results instead of a written answer, and finally a clear message with a way forward. Know what triggers each step and what users see at each one.
- [3] Optional parts stay optional. Memory, personalization, reranking, suggestions and other extras should be skipped, not waited for, when they're slow or down. Give each one a short time limit, and let the request succeed without it.
- [3] Saved answers. Cached responses, summaries made in advance and the last good result can stand in when generation fails (see `layer_05_model_gateway_and_routing.md`). Decide how old a saved answer may be before it's worse than none.
- [3] Shedding features. Under heavy load or during an outage, switch off expensive or low-priority AI features first, such as background summaries, and keep the core ones running. Use feature flags that change in seconds, without a deploy.
- [3] Telling users. If users would trust a weaker result as much as a normal one, tell them the product is running with less, such as "Answers may be shorter while we fix a problem". For small changes that don't affect what users should trust, a quiet fallback is fine.

Check: Write the degradation steps for one feature, with the trigger and what users see at each step. Force each step in a test environment, and show that the feature still does something useful at each one.

## Timeouts and overload [3]

Timeouts for single model calls are covered in `layer_06_inference_and_serving.md`, timeouts for whole agent tasks in `layer_02_agents_and_orchestration.md`, and rate limits and priorities in `layer_05_model_gateway_and_routing.md`. This section covers time and load across a whole request.

- [3] One deadline for the whole request. Give each request a deadline, and pass the remaining time to every step, so each step knows how long it may take. Leave time for the fallback: a model call that uses the full deadline leaves nothing for the next option.
- [3] Overload. When demand passes capacity, whether that's your provider's rate limits, your own GPUs or a tool's API, requests pile up and every user waits. Keep queues bounded, turn away excess requests early with a clear message, and serve waiting users before background work (see `layer_05_model_gateway_and_routing.md`). An unbounded queue turns overload into timeouts for everyone.
- [3] Capacity ahead of demand. Forecast peak traffic for launches and busy periods, and ask providers for higher limits well in advance (see `layer_09_compute.md`). Leave headroom, because capacity for AI takes time to add.
- [2] Slowing intake. Pipelines that process batches or run agents in the background should slow down when the services they call are saturated, rather than keep pushing work at them.

Check: Give one feature a request deadline that's passed to every step, and show that a slow step still leaves time for the fallback. Then load-test the feature past its capacity, and show that excess requests are turned away quickly while accepted ones still meet their target.

## Consistent outputs [3]

The same request can give different outputs, and models change over time. Why outputs vary is covered in `layer_08_foundation_models.md`, deciding where variety is wanted in `layer_01_product_and_ux.md`, and measuring consistency in `concern_01_evaluation.md`. This section covers making outputs dependable.

- [3] Validating and repairing. Check every output against its schema and rules before anything uses it. When it fails, retry with the error included, so the model can fix it, up to a set limit, and then fall back. Libraries such as Instructor and Pydantic AI build this loop in.
- [3] Reducing variation. Use structured output with strict schemas, fixed instructions and examples, a stable order of context (see `layer_04_context_memory_and_retrieval.md`) and a pinned model snapshot. Lower the temperature where the provider allows it, though some reasoning models don't (see `layer_08_foundation_models.md`).
- [3] Same question, same answer. Where answers must not change from one ask to the next, such as prices, policies or account details, compute them in code, look them up from one source of truth, or cache them (see `layer_05_model_gateway_and_routing.md`), instead of generating them fresh each time.
- [2] Voting. For classification and extraction where accuracy matters more than cost, run the call several times and take the most common answer. It smooths out random variation, at a multiple of the cost.
- [2] Determinism. Some APIs accept a seed for more repeatable outputs, but few providers guarantee identical results. On your own servers, engines can be made fully deterministic with specific settings, at some cost in speed.

Check: Run one feature's evaluation set five times and measure how often the runs agree. Add validation with a repair loop, and one change that reduces variation, and show that agreement rises and invalid outputs fall to nearly zero.

## Changes without outages [3]

Many AI incidents start with a change: a prompt edit, a model upgrade, a new tool or a rebuilt index. Gradual rollouts are covered in `layer_05_model_gateway_and_routing.md`, rolling out prompts in `layer_04_context_memory_and_retrieval.md`, and changing tools in `layer_03_tools_and_protocols.md`. This section covers what they share.

- [3] Every change is a release. Prompts, model versions, routing rules, tool definitions, guardrail settings and indexes change behavior as much as code does. Version them, test them on your evaluation set (see `concern_01_evaluation.md`), and release them through the same process as code, with a way back.
- [3] Gradual, with a fast way back. Release changes to a small share of traffic first, watch errors, latency, cost and quality (see `concern_02_observability.md`), and widen the release step by step. Practice rolling back, so it takes minutes and nobody has to work out how during an incident.
- [3] Changes you don't control. Providers change models behind aliases, adjust rate limits and safety filters, and retire models on a schedule (see `layer_08_foundation_models.md`), and tools and MCP servers change too. Pin versions where you can, track retirement dates, and send fixed test requests on a schedule to spot changes early (see `concern_02_observability.md`).
- [2] Work in progress. Agent tasks that are running during a release started under the old version (see `layer_02_agents_and_orchestration.md`), and a new index should be built alongside the old one before you switch (see `layer_04_context_memory_and_retrieval.md`). Plan for both.

Check: Release a prompt change to 5% of traffic, compare it with the current version, and roll it back through configuration, timing the rollback. List every model you depend on, with its pinned version and retirement date.

## Testing for failure [3]

Fallbacks and error handling that never run tend to break without anyone noticing. This section covers making failures happen on purpose, before they happen on their own.

- [3] Injecting faults. In tests and staging, make each dependency fail in each way it can: timeouts, rate-limit and server errors, streams that break partway, malformed or cut-off output, slow tools and search that's down. Mock servers and proxies that inject faults let you do this without waiting for a real outage. Check that each failure leads to the right retry, fallback or message.
- [3] Testing at load. Test at expected peak traffic and beyond, measured from the client, including how rate limits, queues and cold starts behave (see `layer_09_compute.md`). Many failures appear only under load.
- [3] Rehearsing outages. Run a drill in which your main provider goes down, in staging or carefully in production. Measure how long it takes to notice, to fail over and to recover, and fix whatever was slow.
- [3] Exercising fallbacks all the time. Send a small share of real traffic through each fallback path at all times, or test each one on a schedule. A fallback that runs every day is one you know works.
- [2] Failures in production. Some teams inject failures into production on purpose, within tight limits, to prove the system copes. This is called chaos engineering. Know the practice, and start in staging.

Check: Build a fault-injection test that makes each dependency of one feature fail in each way, and show the right behavior for each. Then run a drill in which the main provider goes down, and record how long it takes to notice, fail over and recover.

## Incidents and recovery [3]

Investigating what went wrong is covered in `concern_02_observability.md`. This section covers keeping the product running during an incident and recovering afterwards.

- [3] Runbooks for known failures. Write short runbooks for the failures you expect, such as a provider outage, exhausted rate limits, a sudden change in model behavior, a broken index and runaway costs. Each says how you'll know, what to do first and whom to contact, including the provider's support.
- [3] Manual switches. Be able to force a fallback, switch provider, turn off a feature or raise a limit through configuration, without a deploy (see `layer_05_model_gateway_and_routing.md`). In an incident, a switch that takes an hour to change is no switch.
- [3] Recovering carefully. When a provider comes back, return traffic to it gradually, and drain queues at a pace it can handle. Resume paused agent tasks from their checkpoints (see `layer_02_agents_and_orchestration.md`), and check that retried actions didn't happen twice.
- [3] Learning from incidents. Afterwards, write a short review that blames no one: what happened, why the defenses didn't stop it and what will change. Track the follow-up work until it's done, and add the failing cases to your tests.
- [2] Keeping users informed. Tell users and customers about incidents and reduced service through a status page or messages in the product, especially while the product is running with less.

Check: Write runbooks for a provider outage and for exhausted rate limits, and run one as a drill using only the runbook and the manual switches. Afterwards, write a short review of the drill and fix one thing it revealed.

## Reliability at each layer

Each layer file covers reliability as it applies to that layer. This table shows where.

| Layer | What to keep working there | Level | Where |
|---|---|---|---|
| 1. Product and UX | The product working without AI, never losing the user's work, clear messages when AI can't help, variety or consistency | 3 | `layer_01_product_and_ux.md` |
| 2. Agents and orchestration | Failing well, timeouts for whole tasks, falling back, resuming after crashes, compounding errors | 3 | `layer_02_agents_and_orchestration.md` |
| 3. Tools and protocols | Retrying tools, protecting downstream systems, tools that are down, actions that are safe to repeat, changing tools safely | 3 | `layer_03_tools_and_protocols.md` |
| 4. Context, memory and retrieval | When search fails, rebuilding indexes without downtime | 2–3 | `layer_04_context_memory_and_retrieval.md` |
| 5. Model gateway and routing | Retries, fallbacks, circuit breakers, hedged requests, rate limits, gradual rollouts | 3 (some topics 4) | `layer_05_model_gateway_and_routing.md` |
| 6. Inference and serving | How calls fail, timeouts and cancellation, failures of self-hosted engines, health checks and turning away excess load | 2–3 | `layer_06_inference_and_serving.md` |
| 7. Model adaptation | A prompt-based fallback for every fine-tune, retraining and retirement | 3 | `layer_07_model_adaptation.md` |
| 8. Foundation models | Why outputs vary, stop reasons, pinned versions and retirement dates, a tested second-choice model | 3 | `layer_08_foundation_models.md` |
| 9. Compute | Provider capacity incidents, headroom, spot interruptions | 2 | `layer_09_compute.md` |
| 10. Data | Datasets and environments that can be reproduced | 2–3 | `layer_10_data.md` |

Check: For one feature, list every layer it touches and how that layer's file says to handle failure there. Mark which are in place and tested, and fix the most important gap.

## Other cross-cutting concerns in reliability

### Cost and latency [3]

- [3] What reliability costs. Retries, fallbacks, hedged requests, voting and reserved capacity all cost extra, and retries and fallbacks add latency. Measure what each adds, and spend where the feature's target needs it.

### Security [3]

- [3] Stopping or carrying on. Reliability favors carrying on when a check fails, while security and safety favor stopping. Decide for each check: when the checks on risky actions or high-risk output fail, stop the request, and for low-risk features, carry on and log the failure (see `concern_03_security.md` and `concern_04_governance_and_safety.md`).

### Governance and safety [3]

- [3] Fallbacks follow the same rules. A fallback provider must be approved for the same data and regions as the main one (see `layer_05_model_gateway_and_routing.md`), and degraded modes must still keep audit records and honor deletion. A failover that sends customer data to an unapproved region trades an outage for a compliance breach.

Measuring service levels, alerting on them and investigating incidents are covered in `concern_02_observability.md`, which also covers keeping telemetry from breaking the product. Measuring consistency and testing the quality of fallback paths are covered in `concern_01_evaluation.md`.

Data quality meets reliability in the sources that retrieval depends on: an index that falls out of sync or a source that stops updating breaks answers without raising an error (see `layer_04_context_memory_and_retrieval.md`).

## Not in this concern

- Retries with backoff, circuit breakers, hedged requests and fallbacks for model calls are covered in `layer_05_model_gateway_and_routing.md`. This file covers how retries and fallbacks work across the whole system.
- Handling failure at each layer, such as resuming agent tasks or rebuilding indexes, is covered in each layer's file (see Reliability at each layer).
- Measuring service levels, alerting and investigating incidents are covered in `concern_02_observability.md`. This file covers setting the targets and keeping the product running.
- Measuring consistency and testing the quality of fallbacks are covered in `concern_01_evaluation.md`.
- Floods of requests sent on purpose to slow or stop the product are covered in `concern_03_security.md`.
- Running infrastructure, such as clusters, networks and data centers, and company-wide incident processes belong to site reliability engineering, a discipline of its own. This file covers what changes when AI is involved.

## Learning order

1. Foundations.
2. Service levels. Set targets before you design for them.
3. Retries and fallbacks across the system, after the mechanics in `layer_05_model_gateway_and_routing.md`.
4. Graceful degradation.
5. Timeouts and overload.
6. Consistent outputs.
7. Changes without outages.
8. Testing for failure.
9. Incidents and recovery, before your first on-call rotation for an AI feature.

Learn each layer's part of reliability when you work on that layer.

## Key reading

- Google, *Site Reliability Engineering* (2016), especially the chapters on service level objectives, handling overload and addressing cascading failures, and *The Site Reliability Workbook* (2018), on implementing SLOs.
- Alex Hidalgo, *Implementing Service Level Objectives* (O'Reilly, 2020).
- Michael Nygard, *Release It!* (2nd edition, Pragmatic Bookshelf, 2018), on timeouts, bulkheads, circuit breakers and other patterns for stable systems.
- The Amazon Builders' Library, especially "Avoiding fallback in distributed systems" and "Using load shedding to avoid overload".
- Casey Rosenthal and Nora Jones, *Chaos Engineering: System Resiliency in Practice* (O'Reilly, 2020).
- Horace He and Thinking Machines Lab, "Defeating Nondeterminism in LLM Inference" (2025), on why the same request can give different outputs.
- Your model providers' status pages and their documentation on errors, rate limits and model retirement.
- The documentation for one library that validates and repairs model output, such as Instructor or Pydantic AI.
