# Layer 5, Model gateway and routing: what an application-layer AI engineer needs to know

This file covers the Model gateway and routing layer from `ai_stack.md`. Every model call passes through this layer. It chooses which model or provider handles each request, and switches to another when one is unavailable. It also caches repeated responses, enforces rate limits and spending caps, and manages API keys. This file is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this layer and how deeply to learn each one.

## The rule for depth

Every model call passes through this layer. That makes it the one place where you control, for all calls at once:
- which model answers
- what happens when a provider fails
- which answers get reused
- who may spend how much

Unlike the layers below it, this layer is yours: application teams build or configure it themselves.

So most concepts in this layer are level 3, and three are level 4:
- routing every call through one place
- designing fallbacks
- attributing cost

Every feature depends on these decisions, and no one outside your team will make them for you. Learn how gateway products work inside only as far as you need to choose and run one.

For every call, the gateway answers four questions:

1. Where should this request go? (Routing)
2. What happens if that fails? (Fallbacks and retries)
3. Can it be answered without calling a model? (Caching)
4. Is this caller allowed, and can you afford it? (Rate limits, spending caps and API keys)

A gateway is worth having because it answers these questions once, in one place.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| The gateway | 3 (some topics 4) | Every call passes through it, so its design is yours |
| Routing | 3 | Each request should go to the cheapest model that handles it well |
| Fallbacks and retries | 3 (some topics 4) | Providers fail, and users shouldn't notice |
| Caching | 3 | The cheapest model call is the one you don't make |
| Rate limits and quotas | 3 | Provider limits cap you, and your own limits protect you |
| Spending caps and cost control | 3 (some topics 4) | You can't control costs you can't attribute |
| API keys and access | 3 | Anyone with a leaked key can spend your money |
| Evaluation | 3 | Every route and fallback is a model choice to test |
| Observability | 3 | The gateway sees every call |
| Security | 2–3 | Every request passes through this one checkpoint |
| Governance and safety | 2–3 | It enforces which models and regions data may go to |

## The gateway [3, some topics 4]

A gateway is the single layer that every model call goes through. It can be a library in your code, a service you run, or a hosted product.

- [4] One place for every call. Route every model call through the gateway. It's the only place where you can switch models, enforce limits, control spending and log calls the same way for every feature. Code that calls providers directly is how teams lose track of cost and keys. Make routing through the gateway a team rule.
- [3] Build, buy or use a hosted service. There are four options:
  - a thin wrapper in your own code
  - an open-source gateway you run yourself, such as LiteLLM
  - a hosted gateway, such as Cloudflare AI Gateway or Portkey
  - an aggregator, such as OpenRouter, that also resells access to many models

  Every gateway adds some latency and is one more thing that can fail. A hosted gateway also sees all your prompts and outputs, so review its data handling as you would a model provider's.
- [3] A unified interface. Gateways usually expose one API format for every provider, often the OpenAI format, and translate requests and responses. Translation can drop provider-specific features, such as caching controls or reasoning settings. Check what passes through before you depend on it.
- [3] Configuration as code. Keep routes, fallbacks, limits and model versions in configuration files under version control. Then changes get reviewed, tested and rolled back like code.

Check: Put every model call in one product behind a single gateway or client library. Show that you can switch the model for one feature by changing only configuration, and that no code calls a provider directly.

## Routing [3]

Routing decides which model handles each request. Which models are good enough for each job is decided by evaluation (see Choosing models in `layer_08_foundation_models.md`). This section covers how each request reaches the model you chose.

- [3] Static routing. Assign each feature or task a model in configuration. This is the starting point for every product.
- [3] Rule-based routing. Choose the model for each request from simple signals. Examples are the user's plan, the input's length, its language, whether it includes images, or whether it needs tools. A small classifier can add topic as another signal.
- [3] Routing by difficulty. A fast classifier or small model predicts how hard each request is. It sends easy requests to a cheap model and hard ones to a strong model. Routers like this can cut costs a lot at similar quality, but only if they work on your traffic, so measure them there.
- [3] Cascades. Send each request to a cheap model first. If the answer fails a check, such as validation, a confidence threshold or a grader, send it to a stronger model. This costs little when most requests pass, but escalated requests take longer. It's also how you send hard cases to the teacher model after distillation (see `layer_07_model_adaptation.md`).
- [3] Reasoning effort for each request. Set a reasoning effort or budget for each request: low for simple requests, higher for hard ones. This is routing within one model. It trades cost and latency against quality the same way routing between models does.
- [2] Provider routing. You can often get the same model from several providers: an open-weight model from several hosts, or a lab's model from its own API and from cloud platforms. Route between them by price, speed, region or availability. The same open model can differ in quality between providers (see `layer_09_compute.md`).
- [3] Traffic splitting. Send a set share of traffic to a new model or prompt version, and keep each user on the same version. Use this for gradual rollouts and A/B tests, and roll back by changing configuration. When to run online tests is covered in `layer_10_data.md`.
- [2] Shadow traffic. Send a copy of real requests to a candidate model without showing its answers to users, then compare quality, latency and cost. It costs extra, and the copied data must be allowed to go to that provider.

Check: Add a router to one feature that sends easy requests to a cheap model and hard ones to a strong model. Compare its cost, and its quality on your evaluation set, with sending everything to the strong model. Then roll it out to 10% of traffic, and back again, by changing only configuration.

## Fallbacks and retries [3, some topics 4]

Providers slow down, run out of capacity and go down. This section covers how the gateway keeps requests working when that happens. The ways a single call can fail are covered in `layer_06_inference_and_serving.md`.

- [3] Retries with backoff. Retry temporary errors, such as timeouts, overloads, rate limits and server errors. Wait longer before each retry (exponential backoff), and add a random delay (jitter) so that many clients don't retry at the same moment. Follow the provider's retry-after guidance when it gives one. Don't retry errors that will just fail again, such as invalid requests, prompts that are too long or refusals. Keep the total time spent on retries within your latency budget.
- [3] Safe retries. Retrying a model call is safe on its own. It isn't safe if the response has already started streaming to the user or has triggered an action. Retry only before anything has happened, and design actions to be safe to repeat (Layer 3).
- [4] Fallback design. For every feature, decide what happens when the main model is unavailable:
  - which alternatives to try, and in what order, such as the same model from another provider or region, then a different model
  - which prompt each alternative uses, since prompts don't carry over perfectly between models
  - what users see if everything fails, such as a cached answer, a simpler path without AI, or a clear message

  Test fallbacks regularly, because a fallback that never runs often turns out not to work.
- [3] Circuit breakers. After repeated failures, stop sending traffic to a failing provider for a while and go straight to the fallback. Without this, every request waits for its own timeout. Send occasional test requests to detect when the provider recovers.
- [2] Hedged requests. For calls where speed is critical, send a backup request to a second provider if the first hasn't started answering within a set time, and use whichever answers first. This cuts the slowest response times at extra cost.

Check: Simulate an outage of your main provider in a test environment. Show three things: requests fail over within your latency budget, the circuit breaker stops traffic to the failed provider, and quality on your evaluation set stays acceptable on the fallback.

## Caching [3]

Response caching returns a stored answer without calling a model at all. Prompt caching is different: it only reuses the provider's processing of a shared prompt start, and the model still generates a new answer. Prompt caching is covered in `layer_06_inference_and_serving.md`.

- [3] Exact-match caching. Store each response under a key made from the whole request, including the model, prompt and settings. Return the stored response when an identical request arrives. This is cheap and safe, but it only helps when identical requests repeat. That's common for fixed prompts, batch jobs, evaluations and popular questions, and rare in open-ended chat.
- [3] Cache keys and freshness. Include the model version and prompt version in every cache key, so a change to either stops old answers from being served. Set expiry times that fit how quickly the right answer changes, and clear the cache when the source data changes.
- [3] What not to cache. Don't cache answers that are personal, time-sensitive or meant to vary. A cache shared between users can leak one user's data to another, so scope cache keys to the user or customer where needed.
- [2] Semantic caching. Semantic caching returns a stored answer for a request that means the same thing as an earlier one, found by comparing embeddings. It gets more cache hits than exact matching, but it also risks returning the wrong answer to a request that only looks similar. For example, "cancel my order" and "don't cancel my order" can look nearly the same to an embedding. Use it only where a wrong cached answer does little harm. Set a strict similarity threshold, and measure how often it returns wrong answers.

Check: Add exact-match response caching to one feature, with the model and prompt versions in the cache key and an expiry time. Report the hit rate and savings over a week. Show that a prompt change stops old entries from being served.

## Rate limits and quotas [3]

Rate limits run in two directions: providers limit you, and you limit your own users. Forecasting capacity and asking providers for higher limits are covered in `layer_09_compute.md`.

- [3] Provider limits. Providers limit requests and tokens per minute, and sometimes per day. These limits apply per model and per organization or project, and some providers count input and output tokens separately. Know the limits for every model you use, and track your usage against them.
- [3] Client-side throttling. Don't wait for rate-limit errors. Pace requests with a queue or a token bucket, a counter that refills at the rate your limit allows. Base it on the remaining limit that providers report in each response. This smooths out bursts before they hit the limit.
- [3] Prioritization. When capacity is short, serve requests from waiting users before background jobs. Send bulk work to batch APIs or run it at quieter times.
- [3] Your own limits. Limit how many requests, tokens or dollars each user, customer or API key can use. Then one heavy or abusive user can't use up your provider limits or your budget.
- [2] Pooling capacity. Spread traffic across several deployments, regions or accounts of the same model to get more total capacity. Stay within each provider's terms.

Check: Load-test one feature past your provider's token limit. Show three things: throttling keeps rate-limit errors near zero, requests from waiting users are served before background ones, and one heavy test user hits your per-user limit before affecting anyone else.

## Spending caps and cost control [3, some topics 4]

Model costs grow with usage and can spike without warning. The gateway sees every call, so it's where you measure and limit spending.

- [4] Cost attribution. Tag every call with its feature, customer, user and environment, so you can see what each one costs. You can't control costs you can't attribute. The gateway is the only place to do this consistently.
- [3] Budgets and caps. Set spending limits for each key, team, feature, customer and environment. Send alerts before a limit is reached, and decide what happens at the limit: block requests, switch to a cheaper model or queue them. Providers offer account-wide limits too, but those are too coarse on their own.
- [3] Runaway protection. Agents and loops can make thousands of calls. Cap the number of calls, tokens and cost for each request, session and agent run, and stop any run that exceeds its cap. Set maximum output length and reasoning budgets for each route, so a single request can't become unexpectedly expensive.
- [2] Metering for billing. If you charge customers by usage, the gateway's records of tokens and cost for each customer are the basis for their bills.

Check: Tag every model call with its feature and customer, and build a daily cost report from the tags. Set a budget with an alert for one feature. Cap a test agent run so that it stops at a set cost.

## API keys and access [3]

A provider API key lets anyone who holds it spend money on your account.

- [3] Keys stay on the server. Never put provider keys in apps or browser code. Clients call your backend or gateway, which holds the keys.
- [3] Secrets management. Store keys in a secrets manager, not in code or configuration files. Rotate them regularly, and immediately after a leak. Give each environment and service its own key, so that a leak stays contained and usage can be traced.
- [3] Virtual keys. The gateway issues its own keys to teams, services or customers and maps them to provider keys behind the scenes. Each virtual key has its own limits, budget, allowed models and usage record. Revoking one doesn't affect the others.
- [2] Short-lived credentials. Where possible, use short-lived tokens or your cloud platform's own identity system instead of long-lived keys.
- [2] Customers' own keys. Some products let customers supply their own provider keys, so usage is billed to them. Store those keys encrypted, and keep each customer's keys separate from everyone else's.
- [2] Leak detection. Scan code and logs for keys, and watch for sudden spikes in usage that suggest a stolen key. Some providers revoke keys that appear in public code repositories.

Check: Move one product's provider keys into a secrets manager. Give each environment and service its own virtual key with a budget. Rotate a provider key without downtime.

## Cross-cutting concerns in this layer

### Evaluation [3]

- [3] Testing routes and fallbacks. Every route and fallback is a model choice. Test each one on the traffic it will actually receive, and test again when models, prompts or traffic change. A router that sends hard requests to a weak model shows up as a quality drop on one slice of traffic, so check results slice by slice.

### Observability [3]

- [3] Logging at the gateway. The gateway sees every call, so it's the natural place to record the route chosen, any retries and fallbacks, cache hits, cost, and the tags that attribute each call. Pass a trace ID through the gateway, so that each model call links to the user request it served. The rest of what to log for each call is covered in `layer_08_foundation_models.md`.

### Security [2–3]

- [3] A policy checkpoint. Every call passes through the gateway, so it's a natural place to remove personal data before it leaves, block unapproved models, and run input and output filters. Which checks to run is covered in `concern_03_security.md` and `concern_04_governance_and_safety.md`.
- [2] Denial of wallet. Attackers can drain your budget by sending expensive requests through your product. The defenses are your own limits and runaway protection, covered above.

### Governance and safety [2–3]

- [3] Approved models and regions. Keep a list of the models and providers your company has approved, with their data terms and regions. Enforce the list in the gateway so no feature can quietly use an unapproved model. Route each customer's data only to the regions and providers allowed for it.
- [2] Audit trail. Record which model and provider processed each request, for compliance reviews and investigating incidents.

Reliability is covered under Fallbacks and retries, and the gateway itself must be at least as reliable as the providers behind it. Cost and latency are covered under Routing, Caching, and Spending caps and cost control. The latency the gateway itself adds is covered under The gateway. Data quality has nothing specific to this layer.

## Not in this layer

- Deciding which models are good enough for each job is covered in `layer_08_foundation_models.md`. This file covers sending each request to the model you chose.
- Provider capacity, reserved throughput and batch pricing are covered in `layer_09_compute.md`.
- Timeouts, streaming and cancellation for single calls, and how prompt caching works, are covered in `layer_06_inference_and_serving.md`.
- Routing between agents and workflow steps belongs to Layer 2 (Agents and orchestration).
- Tool calls and their side effects belong to Layer 3 (Tools and protocols).
- Writing and versioning prompts belongs to Layer 4 (Context, memory and retrieval).
- Which security and safety checks to run is covered in `concern_03_security.md` and `concern_04_governance_and_safety.md`. This file covers the gateway as the place to run them.

## Learning order

1. The gateway. Put every call through one place first.
2. API keys and access.
3. Fallbacks and retries.
4. Rate limits and quotas.
5. Spending caps and cost control.
6. Routing, which saves the most once the basics are in place.
7. Caching.

## Key reading

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 10 covers routers, gateways and caches in an AI application's architecture.
- Chen, Zaharia and Zou, "FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance" (2023), on cascades.
- Ong et al., "RouteLLM: Learning to Route LLMs with Preference Data" (2024), on routing by difficulty.
- Marc Brooker, "Timeouts, retries, and backoff with jitter" (Amazon Builders' Library).
- Martin Fowler, "CircuitBreaker" (2014).
- Dean and Barroso, "The Tail at Scale" (2013), on hedged requests.
- Google, *Site Reliability Engineering* (2016), the chapters on handling overload and cascading failures.
- OWASP Top 10 for LLM Applications (2025), especially "Unbounded Consumption".
- The documentation for an open-source gateway such as LiteLLM, and your providers' documentation on rate limits and errors.
