# The AI stack in learning order: what an application-layer AI engineer needs to know

This file combines the ten layer files and seven concern files in this folder into one learning path. It is for an AI engineer who builds products on existing models (layers 1 to 5). It contains every concept from those files, in the order you'd meet them while building a product: from your first model call, through evaluating, building and launching a feature, to learning from real use and the layers underneath. The concepts, checks and readings are copied word for word. What's new is the order, the stage introductions and the notes on when to learn each section. References now point to sections of this file instead of to other files.

## Contents

- Stage 1. Working with models (1.1 to 1.13)
- Stage 2. Prompts, context and traces (2.1 to 2.8)
- Stage 3. Evaluation (3.1 to 3.20)
- Stage 4. Product basics (4.1 to 4.12)
- Stage 5. Tools (5.1 to 5.18)
- Stage 6. Retrieval, memory and data quality (6.1 to 6.20)
- Stage 7. Agents (7.1 to 7.15)
- Stage 8. The gateway, cost and latency (8.1 to 8.20)
- Stage 9. Reliability and monitoring (9.1 to 9.19)
- Stage 10. Safe to launch (10.1 to 10.21)
- Stage 11. Learning from use (11.1 to 11.12)
- Stage 12. Adapting models (12.1 to 12.13)
- Stage 13. Under the hood (13.1 to 13.18)
- Stage 14. Each concern across the stack (14.1 to 14.7)
- Outside this curriculum

## How to use this file

- Work through the stages in order. Each stage opens with a table of its sections and why each matters, and each section ends with a check.
- The first time a layer or concern appears, an About part explains how deeply to learn it and why.
- Each section starts by naming the layer or concern it came from. Some add when or how to learn it, such as "Learn this only for systems with no API". Most sections you need only in some situations come at the end of their stage.
- Some checks need parts you build in later stages. Do those when you get there.
- A reference such as "(see 6.1 Embedding models)" points to another section by its number and title.
- Each stage ends with key reading for its sections.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## AI stack: ten layers and seven cross-cutting concerns

Read the stack from top to bottom. It starts with what users touch and ends with the raw inputs everything is built from. Each layer depends on the layers below it. The cross-cutting concerns are not layers. They apply to every layer and have to be handled at each one.

### Layers, top to bottom

1. Product and UX. This is where people use the system: apps, copilots, chat and voice interfaces, and AI built into existing software. This layer also collects user feedback, which flows back into Data.
2. Agents and orchestration. This layer turns a request into a sequence of steps. It covers planning, multi-step workflows, coordination between agents, handoffs to humans, and tracking where each task stands.
3. Tools and protocols. This is how agents take action: function calling, APIs, and standard protocols. MCP connects models to tools and data, and A2A lets agents communicate with each other. It also covers the sandboxes and browsers where agents run code and take actions.
4. Context, memory and retrieval. This layer decides what information the model sees on each call. It includes retrieval-augmented generation (RAG), vector and keyword search, knowledge graphs, and long-term memory that lasts across sessions. Retrieval only reads information. Memory also writes, deciding what to store, update and forget.
5. Model gateway and routing. Every model call passes through this layer. It chooses which model or provider handles each request and switches to another when one is unavailable. It also caches repeated responses, enforces rate limits and spending caps, and manages API keys.
6. Inference and serving. This layer runs models in production, whether through a provider's API or on your own infrastructure. It covers serving engines, request batching, quantization, KV-cache management, and the endpoints that expose models.
7. Model adaptation. This layer specializes a base model for a domain or task. The main methods are fine-tuning, reinforcement learning from human or automated feedback, and distillation into smaller, cheaper models.
8. Foundation models. These are the pre-trained models everything else builds on: frontier proprietary models, open-weight models, multimodal models, and the embedding models used for search.
9. Compute. This is the physical base: GPUs and other accelerators, the clusters and cloud capacity they run in, and the power and data centers behind them.
10. Data. Data sits beside Compute as the other raw input. It includes pre-training corpora, fine-tuning and preference datasets, human labeling, synthetic data, and RL environments (simulated tasks that models learn from). It also includes feedback collected from the product.

### Across every layer

- Evaluation. This measures whether the system does its job, using test sets, automated graders (including models that judge other models), human review, and A/B tests in production. It applies everywhere, from choosing a model to checking retrieval quality and whether an agent finished its task.
- Observability. This shows what the system is actually doing. It uses traces that follow a request through agents, tools and model calls, plus logs and metrics for finding and fixing failures.
- Security. This defends against attackers. It covers prompt injection, authentication and authorization (including what each agent is allowed to do), secrets management, and preventing data from being stolen.
- Governance and safety. This makes sure the system behaves acceptably even when no one is attacking it. It covers guardrails against harmful or biased output, privacy and handling of personal data, regulatory compliance, and audit trails. It also covers intellectual property, where data came from, and human oversight of what agents do.
- Reliability. This keeps the system working when parts fail. It covers retries, switching between models and providers, uptime targets, falling back to reduced functionality instead of failing outright, and keeping model outputs consistent.
- Cost and latency. This keeps the system affordable and fast. It covers cost per request or task, token and GPU spending, and response time, all weighed against quality.
- Data quality. This keeps the information moving through the system trustworthy. It covers accuracy, freshness, lineage (where data came from) and how data changes over time. It applies equally to training data, retrieved documents, tool outputs and evaluation sets.

### How the parts connect

A request enters through Product and UX. The agent layer plans the work, calling tools and pulling in context as needed. Each model call goes through the gateway to inference, which runs a foundation model or an adapted model on compute. Feedback travels the other way: what users do in the product becomes data, and that data improves the models through adaptation.

## Stage 1. Working with models

Everything you build runs on a model, so the path starts with the model itself. This stage covers calling a model, what it can and can't do, how it works inside, and how fast and expensive each call is. By the end, you can make a model call, measure it, record it and explain what it did.

| Section | Level | Why this level | From |
|---|---|---|---|
| 1.1 Working with a model | 3 | You use this interface in every call | Layer 8 |
| 1.2 Capabilities and limits | 3 (some topics 4) | Knowing where models break is the core of this layer | Layer 8 |
| 1.3 How models work | 2 | Explains model behavior; labs handle the internals | Layer 8 |
| 1.4 Frontier proprietary models | 3 | Most products run on them | Layer 8 |
| 1.5 Inference performance | 3 | Users feel latency on every call | Layer 6 |
| 1.6 Calling models through an API | 3 | You write this code for every feature | Layer 6 |
| 1.7 Cost and latency in foundation models | 3 | Price per token misleads; cost per task is what counts | Layer 8 |
| 1.8 Reliability in foundation models | 3 | The same input can give different outputs | Layer 8 |
| 1.9 Reliability in inference and serving | 2–3 | Every call can be slow, fail or stop early | Layer 6 |
| 1.10 Security in foundation models | 2–3 | Models can't reliably tell instructions from data | Layer 8 |
| 1.11 Observability in foundation models | 3 | Stop reasons and token counts explain many surprises | Layer 8 |
| 1.12 Observability in inference and serving | 2–3 | Time to first token and output speed must be tracked separately | Layer 6 |
| 1.13 Governance and safety in foundation models | 2 | Licenses, usage policies and laws limit what you can ship | Layer 8 |

### About Layer 8: Foundation models

You don't build foundation models, but everything you build runs on them. The model is your most important dependency. Know it the way a backend engineer knows their database: what it does well, where it breaks, what it costs and how to replace it.

Go deepest on behavior and choice: what models can and can't do, and how to pick one by testing it on your own task. Learn the model's interface to level 3, because you use it in every call. Learn how models work inside, such as attention and training stages, to level 2. That's enough to explain their behavior, not to build one.

Most decisions in this layer come down to one question: which model does this job well enough, cheaply enough and fast enough? Answer it by testing on your own task. Benchmarks and price lists only narrow the list. New models arrive every few months, so you'll answer this question again and again. Build your product so that switching models is routine.

### 1.1 Working with a model [3]

From Layer 8, Foundation models. You need the interface to try anything else.

This is the model's interface. You use it in every call.

- [3] Tokens. Models read and write tokens, which are pieces of words. In English a token is roughly four characters, or three-quarters of a word. Other languages and code often need more tokens for the same content. Tokens explain some odd failures, such as miscounting the letters in a word. Different models also split the same text into different numbers of tokens.
- [3] Context window. The context window is the maximum number of tokens a model can handle in one call, counting both input and output. The output also has its own, smaller limit. The system prompt, conversation history, tool definitions, documents and images all count against the window.
- [3] Messages and roles. Chat models take a list of messages, and each message has a role:
  - a system (or developer) message sets the model's behavior
  - user messages hold requests
  - assistant messages hold the model's replies
  - tool messages hold tool results

  Most model APIs are stateless: the model remembers nothing between calls, so your application resends the conversation each time. Some APIs can store the conversation for you.
- [3] Sampling. The model produces probabilities, and sampling picks each token from them. Temperature controls randomness: lower for consistent answers, higher for variety. Top-p limits the choice to the most likely tokens. Stop sequences and output limits end generation. Some reasoning models don't let you change these settings.
- [3] Structured output. Models can return JSON that matches a schema you supply. With strict schema enforcement, the provider guarantees that the output matches, unless the response is cut off or refused. Without it, models usually but not always produce valid JSON. Keep schemas simple. When quality matters, let the model reason before it fills in the structure. How providers enforce schemas is covered in 13.7 Structured generation.
- [3] Reasoning controls. Reasoning models think before they answer, and you pay for the thinking tokens as output. You can usually set how much they think, with an effort level or a token budget. Some providers show the reasoning, some summarize it and some hide it.
- [3] Stop reasons. Every response reports why it ended: it finished, hit the output limit, called a tool, or was refused or blocked. Check this on every call, because a response cut off by the output limit can look complete.
- [2] Token probabilities. Some APIs return the probability of each output token (logprobs). Where they're available, they're a useful confidence signal for classification.
- [2] Provider-specific features. Providers add built-in tools, such as web search, code execution and file handling. Some can also store conversations on their side. These features save work but tie you to one provider.

Check: Write a small script that calls a model with a system prompt and a JSON schema. Count the tokens before sending, and check the stop reason on every response. Run the same request 10 times at each of two temperatures and compare how much the output varies.

### 1.2 Capabilities and limits [3, some topics 4]

From Layer 8, Foundation models. Learn them by running real inputs through the models you use.

Knowing what models do well and where they break is the core of this layer. Learn the general patterns below. Then learn the details of the models you ship with from your own evaluations and traces.

- [4] Uneven abilities. Models can be excellent at hard tasks and still fail at easy ones. Doing well on one task doesn't mean a model will do well on a related one. For the models you ship, know their specific strengths and failures from your own tests, not from their reputation.
- [3] Instruction following. Models follow the system prompt and user instructions, but not perfectly. Recent models follow instructions more literally, so vague instructions get vague results, and prompts written for older models may need rewriting. When instructions conflict, models are trained to rank the system prompt above the user, and the user above text in documents and tool results. That ranking doesn't always hold.
- [3] Learning from examples. Models can pick up a task from a few examples in the prompt (in-context learning, or few-shot prompting). They copy the examples' format, length and style, including their mistakes. So choose examples carefully and vary them.
- [3] Reasoning. Working step by step improves results on math, code, planning and other multi-step problems, but it adds cost and latency. Reasoning models do this automatically, so telling them to "think step by step" adds little. For simple lookup, extraction and classification tasks, extra reasoning rarely helps.
- [2] Faithfulness of reasoning. The reasoning a model shows doesn't always match how it actually reached its answer. Treat it as a useful hint, not a reliable explanation.
- [3] Long context. Models accept very long inputs but use them unevenly. Quality drops as the input grows, and material in the middle of a long input is easier to miss. Finding one fact in a long document (a "needle in a haystack") is much easier than reasoning across many parts of it. Test your model at the context lengths you actually use, and keep context lean.
- [3] Knowledge and hallucination. Models store a lot of knowledge but recall it unreliably. They are weakest on rare facts, recent events, exact numbers, quotes and citations. When a model doesn't know something, it often gives a confident, plausible answer anyway. This is called hallucination. Give the model the facts it needs in the context, ask it to cite them, and allow it to say it doesn't know. Knowledge cutoffs are covered in 3.20 Pre-training corpora.
- [3] Common failure patterns:
  - Models agree with the user too readily (sycophancy).
  - Their results can change with small changes in wording (prompt sensitivity).
  - They favor options because of where those options sit in a list.
  - They write longer answers than needed.
  - In long tasks they stop early or skip steps.
  - In long conversations they drift from their instructions.
  - They sometimes refuse reasonable requests.
  - In coding tasks they sometimes game the tests.
- [3] Tool use and agentic tasks. Models differ in how reliably they pick the right tool, fill in its arguments and recover from errors. Recent models can carry out long, multi-step tasks with tools, such as fixing a bug across a codebase. The length of task they can complete reliably has been growing fast, but reliability still drops as tasks get longer. How function calling works is covered in 5.2 Function calling.
- [2] Confidence. When a model says how sure it is ("I'm 90% sure"), that number is poorly calibrated. Token probabilities are a better signal where available, but they aren't perfect either.
- [2] Refusals and safety filters. Models refuse requests that their training or their provider's policy treats as harmful, and they sometimes refuse harmless ones too. Providers may also run separate filters that block inputs or outputs. Learn how your models behave close to those limits for your use case.

Check: Write a one-page profile of a model you ship with. Cover its strengths on your task and its common failures, with a real example of each failure. Include the prompt or design changes that work around them.

### 1.3 How models work [2]

From Layer 8, Foundation models. This explains the behavior you've just seen.

You won't build a model, but a working picture of how one works explains most of what it does.

- [2] Next-token prediction. A language model reads a sequence of tokens and gives a probability for every possible next token. It picks one, adds it to the sequence and repeats. Everything a chat model does is produced this way, including answering, reasoning and calling tools.
- [2] The transformer. Tokens are turned into vectors, passed through a stack of layers, and come out as next-token probabilities. Each layer uses attention, which lets every token draw on every earlier token. Attention is why models use context so well, and why cost grows with context length.
- [2] Training stages. Pre-training on huge amounts of text produces a base model that continues text but doesn't follow instructions. Post-training then turns it into an assistant, using three methods:
  - supervised fine-tuning on example conversations
  - preference tuning on human or AI judgments of which answer is better
  - reinforcement learning on tasks with checkable answers

  Safety training teaches the model what to refuse. How these methods work is covered in 12.3 Supervised fine-tuning, 12.6 Learning from human preferences and 12.8 Learning from automated rewards. Here, you only need to know what each stage does to the model's behavior.
- [2] Where quirks come from. Many quirks trace back to a training stage:
  - Preference tuning rewards answers people like. That can make models flatter and agree with the user (sycophancy), and write at length.
  - Reinforcement learning on checkable tasks makes models strong at math and code. It sometimes also teaches them to game tests.
  - Safety training causes refusals, including some unnecessary ones.
- [2] Base and instruction-tuned models. Base models only continue text. Instruction-tuned (chat) models follow instructions in a conversation format. Almost all products use instruction-tuned models.
- [2] Dense and mixture-of-experts models. A dense model uses all its parameters for every token. A mixture-of-experts (MoE) model has many sub-networks, called experts, and sends each token through only a few of them. So it has many parameters in total but uses only some for each token, which gives more quality for the same compute. Many recent large models use this design. What it means for memory and speed is covered in 13.2 Compute arithmetic.
- [2] Model size. More parameters generally mean more knowledge and better reasoning, at higher cost and latency. Well-trained small models can match much larger older models on many tasks. They are often trained to imitate larger ones (distillation).
- [2] Encoder models. Models such as BERT read text and produce vectors or labels without generating text. They are small, fast and cheap. They are still common for classification and as the base of embedding models.
- [1] Other architectures. State-space and hybrid models handle long inputs more cheaply. Diffusion language models generate many tokens at once instead of one at a time.
- [1] Interpretability. Research into what happens inside models, such as finding the internal features that represent concepts. It explains some behavior but isn't yet a practical tool for application teams.

Check: Pick three quirks you've seen in a model you use, such as flattering the user, refusing a harmless request or gaming a test. For each one, explain which training stage most likely caused it.

### 1.4 Frontier proprietary models [3]

From Layer 8, Foundation models.

Frontier models are the most capable models. Labs such as OpenAI, Anthropic, Google and xAI offer them through APIs, and most products run on them.

- [3] Families and tiers. Each lab offers a family of models in tiers: a large, most capable model, a balanced mid-sized one, and a small, fast, cheap one. For example, Anthropic's Claude comes in tiers such as Opus, Sonnet and Haiku, and Google's Gemini in tiers such as Pro and Flash. For the families you use, know the current tiers, their strengths, prices and limits.
- [2] The wider landscape. Know the other major families well enough to tell when one is worth testing. That means what each is known for, roughly how they compare, and how fast the rankings change.
- [3] Versions and aliases. Providers release dated snapshots, plus aliases that point to the latest snapshot. When an alias moves to a new snapshot, behavior can change without warning. Pin a snapshot in production and upgrade on purpose.
- [3] Retirement. Providers retire old models on a published schedule, sometimes with only a couple of months' notice. Track the retirement dates of every model you use and plan migrations early.
- [2] Access channels. You can often get the same model from its lab directly or through a cloud platform such as AWS Bedrock, Google Vertex AI or Microsoft Azure. Channels can differ in features, rate limits, regions, data terms and how soon new models arrive.
- [2] System cards. Labs publish a system card for each model, describing its capabilities, test results, known limits and safety testing. Read the card for every model you adopt.

Check: For the model family you use most, list the current tiers with their prices, context windows and main strengths. Add the snapshot you've pinned and its retirement date, if one has been announced.

### About Layer 6: Inference and serving

Every model call is an inference request, so this layer is where your product meets the model. Most application teams get inference from a provider's API, which hides the serving machinery. But this layer decides what your users experience: how long they wait for the first word, how fast the answer streams, and what happens when a call is slow or fails.

Go deep (level 3) on what you touch on every call: measuring latency, calling endpoints well, and how prompt caching works. Learn how serving works inside to level 2. That covers engines, batching, KV-cache management, quantization and faster decoding. It's enough to understand how providers behave and price, to run a model locally, and to judge when self-hosting makes sense. If your team self-hosts, take the self-hosting topics to level 3. No concept in this layer needs level 4. Building or tuning serving engines is infrastructure work.

Three numbers describe how a model call performs:

1. Time to first token: how long until the answer starts. Users feel this as waiting.
2. Output speed: how fast tokens arrive once the answer starts, in tokens per second.
3. Throughput: how many tokens or requests a server handles per second across all users. Your costs depend on this.

Total response time is roughly the time to first token plus the number of output tokens divided by the output speed. Serving engines trade these numbers against each other: running more requests together raises throughput and lowers cost, but it slows each request.

### 1.5 Inference performance [3]

From Layer 6, Inference and serving. Learn the three numbers first.

These numbers describe what users experience, so learn them first.

- [3] Time to first token. The time from sending a request to receiving the first output token. It includes network time, waiting in a queue, and processing the prompt. Long prompts and reasoning make it longer.
- [3] Output speed. How fast tokens arrive once output starts, in tokens per second. It is also measured as the time per output token. When text streams to a reader, anything faster than reading speed feels fast. When the output feeds other code or an agent's next step, only the total time matters.
- [3] Total response time. Roughly the time to first token plus the output tokens divided by the output speed. Output length usually dominates: 1,000 tokens at 50 tokens per second take 20 seconds. Hidden reasoning tokens count too. Shorter outputs are often the biggest latency win.
- [3] Percentiles. Averages hide slow requests. Track the median and the 95th and 99th percentiles (p50, p95, p99). Users remember the slow requests, and multi-step agents multiply the chance of hitting one. If each step of a 10-step agent has a 5% chance of being slow, about 40% of runs hit at least one slow step.
- [3] Latency budgets. Decide how long each feature may take, and split that time across its steps: retrieval, each model call and each tool call. Then design so that users see progress while they wait.
- [2] Throughput. How many tokens or requests a system serves per second across all users. When you self-host, it sets your costs. When you use an API, it's what rate limits cap. Running more requests together raises throughput but slows each request. Why this happens is covered in 13.2 Compute arithmetic.
- [2] Prefill and decode. Every request has two phases. Processing the prompt (prefill) sets the time to first token. Generating the output (decode) sets output speed. The two phases stress hardware differently, and engines schedule them differently.
- [2] Load and latency. Latency stays flat as load grows, then rises sharply as a server nears capacity, because requests start to queue. Test at the concurrency you expect, measured from the client. Benchmarking on your own traffic is covered in 13.12 Cost and latency in compute.

Check: For one feature, measure time to first token, output speed and total time at p50 and p95, from the client side. Break the total time into its parts. Make one change that cuts p95 total time, such as shorter outputs, parallel calls or a faster model.

### 1.6 Calling models through an API [3]

From Layer 6, Inference and serving.

The endpoint is how your code reaches a model. You write this code for every feature.

- [3] API styles. Most providers offer a chat-style endpoint that takes a list of messages and returns a response. Many providers, and nearly all self-hosted engines, also accept the OpenAI API format, which has become a common standard. Code written against it can switch providers, or point at a local engine, by changing a URL. Features and parameters still differ between providers.
- [3] Streaming. The server sends tokens as it generates them, usually as server-sent events. Your code has to assemble partial text, tool calls and structured output as they arrive. It also has to handle errors that arrive partway through, and pass the stream through your own backend to the user.
- [3] Timeouts and cancellation. Set timeouts that fit each request: short for small classification calls, long for reasoning or agent steps that can take minutes. Cancel requests the user no longer needs, for example when they leave or send a new message, so those requests don't keep using capacity.
- [3] Concurrency. Run independent model calls in parallel instead of one after another, using async clients and reused connections. Parallel calls cut total time, but they count against rate limits together.
- [2] Background requests. Some providers let you start a long request, such as a deep research task, and collect the result later instead of holding a connection open. Batch endpoints for bulk work are covered in 8.12 Cloud capacity.
- [2] Realtime connections. Voice and other live interactions use persistent connections, such as WebSockets or WebRTC, that stream audio both ways. Turn detection decides when the user has finished speaking. Voice feels natural only when responses start within about a second. Learn this to level 3 if you build voice products.

Check: Build a small backend endpoint that streams a model's answer to a browser and cancels the model request when the user disconnects. Have it run two independent model calls in parallel. Then switch it to a local model by changing only the base URL.

### 1.7 Cost and latency in foundation models [3]

From Layer 8, Foundation models.

- [3] Pricing structure. Models charge separately for input tokens, output tokens and cached input, and some charge more for very long inputs. Reasoning tokens are billed as output. Images and audio are converted to tokens and billed too.
- [3] Cost per task. Compare models by what a completed task costs, not by price per token. A model with cheaper tokens can cost more overall if it needs longer prompts, more reasoning, more retries or more tool calls. Use each model's own token counts when you compare.
- [3] Prompt caching. Providers charge much less for input that repeats the start of a recent prompt. This happens either automatically or when you mark content for caching. Put stable content first, such as instructions, tool definitions and reference documents, and put changing content last. How caching works is covered in 2.4 KV-cache management.
- [3] Latency. Time to the first token and output speed differ widely between models and providers. Reasoning adds delay before the first visible token. Use small, fast models where users are waiting.

### 1.8 Reliability in foundation models [3]

From Layer 8, Foundation models.

- [3] Consistency. The same input can give different outputs, even at temperature 0, because of how providers batch and compute requests. Use structured output, validate every response, retry failures, and test with repeated runs.
- [2] Outages. Models go down or slow down. Falling back to another model or provider is covered in 8.3 Fallbacks and retries, but you need a tested second-choice model for each feature.

Pinned versions and retirement dates are covered in 1.4 Frontier proprietary models.

### 1.9 Reliability in inference and serving [2–3]

From Layer 6, Inference and serving.

- [3] How calls fail. Any inference call can fail in several ways. It can time out. It can return a rate-limit or overload error. It can fail partway through a stream. Or it can return an empty or cut-off answer. Handle each case and show users something sensible. Retries and fallbacks are covered in 8.3 Fallbacks and retries.
- [2] Self-hosted failures. Engines crash when they run out of GPU memory, and they can hang on unusual inputs. They also slow down when the KV cache fills. Monitor them, and restart unhealthy servers automatically.

Why the same request can give different outputs is covered in 1.8 Reliability in foundation models.

### 1.10 Security in foundation models [2–3]

From Layer 8, Foundation models.

- [3] Prompt injection. Models can't reliably tell instructions from data. So text in a web page, document, email or tool result can take control of the model. Training reduces this but doesn't solve it. Assume any model that reads untrusted content can be hijacked, and limit what it's able to do. Defenses are covered in 5.8 Prompt injection.
- [2] Jailbreaks. Users can talk models into breaking their rules. No model fully resists this, so don't rely on the model alone to enforce your policies.
- [2] System prompt leaks. Users can often get a model to reveal its system prompt, so never put secrets in it.
- [2] Trusting model files. Download open-weight models from official sources. Prefer the safetensors format over formats that can run code when loaded. Be aware that weights can also contain hidden, planted behaviors.

### 1.11 Observability in foundation models [3]

From Layer 8, Foundation models.

- [3] What to log for each call. Record the model and snapshot, the settings, token usage (including cached and reasoning tokens), the stop reason and the latency. These explain most surprises in cost and quality.

### 1.12 Observability in inference and serving [2–3]

From Layer 6, Inference and serving.

- [3] Latency metrics. Record time to first token and output speed, not just total time. Track them as percentiles by model and provider. What else to log for each call is covered in 1.11 Observability in foundation models.
- [2] Engine metrics. Self-hosted engines report queue length, running requests, KV cache use and cache hit rate, usually in Prometheus format. These metrics warn you before latency climbs. GPU metrics are covered in 13.14 Observability in compute.

### 1.13 Governance and safety in foundation models [2]

From Layer 8, Foundation models.

- [2] Usage policies. Every provider restricts what you may build with its models. Read the policy for each model you use.
- [2] Regulation. Laws such as the EU AI Act give different duties to companies that make models and to companies that build products with them. Know which role you're in, and bring in legal when it matters.
- [2] Bias. Models can reflect social and cultural biases, and can perform worse in some languages. Test your product across the user groups and languages you serve.
- [1] Output ownership. Provider terms say who owns the outputs. Some providers also promise to defend customers against copyright claims over outputs.

Data quality has little specific to this layer. Knowledge cutoffs and other effects of training data are covered in 3.20 Pre-training corpora.

### Key reading for Stage 1

- Andrej Karpathy, "Deep Dive into LLMs like ChatGPT" (video, 2025), on how models are trained and why they behave as they do.
- 3Blue1Brown's videos on transformers and attention (2024), for the mechanics.
- Jay Alammar and Maarten Grootendorst, *Hands-On Large Language Models* (O'Reilly, 2024), on tokens, embeddings, semantic search and multimodal models.
- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 2 covers how foundation models are built, sampling, structured outputs and hallucination, and chapter 9 covers inference metrics, batching, caching, quantization and speculative decoding.
- Liu et al., "Lost in the Middle" (2023), and Chroma's "Context Rot" report (2025), on how models use long inputs.
- Sharma et al., "Towards Understanding Sycophancy in Language Models" (2023).
- Kalai et al., "Why Language Models Hallucinate" (OpenAI, 2025).
- METR, "Measuring AI Ability to Complete Long Tasks" (2025), on how fast agentic ability is improving.
- The documentation, prompting guides and system cards for the models you use, plus Simon Willison's blog for keeping up with new releases.
- Databricks, "LLM Inference Performance Engineering: Best Practices" (2023), on latency metrics and how batching trades latency for throughput.
- The vLLM documentation, plus your providers' guides to latency, streaming and prompt caching.

## Stage 2. Prompts, context and traces

What the model does depends mostly on what it sees on each call. This stage covers prompts, which belong to Layer 4 because the prompt is part of what the model sees, along with the rest of the context, long conversations and prompt caching. It ends with tracing, so you can see exactly what each call received and returned.

| Section | Level | Why this level | From |
|---|---|---|---|
| 2.1 Prompts | 3 (some topics 4) | You write and change them more than anything else | Layer 4 |
| 2.2 Context engineering | 4 | What the model sees decides most of what it does | Layer 4 |
| 2.3 Conversation history | 3 | Long conversations and agent runs outgrow the window | Layer 4 |
| 2.4 KV-cache management | 2 (some topics 3) | Prompt caching is the part you use daily | Layer 6 |
| 2.5 Observability foundations | 3 | AI features fail without raising errors | Observability |
| 2.6 Tracing | 3 | A trace follows one request through agents, tools and model calls | Observability |
| 2.7 What to record | 4 | You can't recover what you didn't record at the time | Observability |
| 2.8 Debugging from traces | 3 (some topics 4) | Most causes are visible in the trace once you look | Observability |

### About Layer 4: Context, memory and retrieval

A model works from two things: what it learned in training, and what's in its context on the current call. You can change the first only by training (see 12.1 Deciding whether to adapt). You control the second completely, on every call. So when an answer is wrong, the first question is whether the context held what the model needed.

Everything the model sees comes from four sources:

1. Instructions you write: the system prompt, examples and tool definitions. (2.1 Prompts)
2. The work so far: the conversation, tool results and notes from earlier steps. (2.3 Conversation history)
3. Knowledge you look up: documents, records and search results. (6.2 Retrieval-augmented generation to 6.8 Agentic retrieval, and 6.19 Knowledge graphs)
4. What you saved from earlier sessions: facts about users and past work. (6.9 Long-term memory)

Context engineering decides how much of each goes in, and in what order, within one context window. Retrieval only reads. Memory also writes, deciding what to store, update and forget, so its mistakes outlast the call that made them.

Like the gateway in Layer 5, this layer is yours. No provider knows what your model should see, so no one else will build it for you. That makes it the deepest layer in this curriculum for an application engineer. Most concepts are level 3, and four things are level 4:
- context engineering: deciding what goes into each call
- writing clear instructions
- managing prompts as code
- finding which step caused a bad answer

Learn how search works inside, such as index algorithms, only to level 2, because search engines and vector databases handle it. Knowledge graphs are level 2 unless your questions depend on relationships across many records.

### 2.1 Prompts [3, some topics 4]

From Layer 4, Context, memory and retrieval. You'll write them before anything else.

This section covers writing and managing the instructions you give the model. How models respond to instructions and examples is covered in 1.2 Capabilities and limits.

- [4] Clear instructions. Write the system prompt as a brief for a capable new colleague who knows nothing about your product. Say what the task is, who the users are, what a good result looks like, and what to do when information is missing or a request is out of scope. Explain the reason behind each rule, because models apply rules better when they know why. Recent models follow instructions literally, so say exactly what you want.
- [3] Structure. Break long prompts into labeled sections, such as task, rules, output format and examples. Keep instructions apart from the data they act on. Say what to do, not only what to avoid. A role such as "You are a support agent for Acme" sets tone, but specific instructions matter more.
- [3] Examples. Choose a few varied, realistic examples that include hard cases, since models copy their format, length and mistakes. For tasks with many kinds of input, keep a bank of good examples and retrieve the most similar ones for each request (dynamic few-shot prompting).
- [3] Output instructions. Say what format and length you want. Use structured output when code will read the result (see 1.1 Working with a model). When quality matters, let the model reason before its final answer.
- [3] Reasoning models. Reasoning models need less guidance on how to think. Give them the goal, the constraints and what a good result looks like. Detailed step-by-step instructions can do worse than a high-level request, so test both.
- [3] Templates. Code assembles most prompts from a template, filling in user data, documents and memories. Test templates with real values, including empty, very long and oddly formatted ones, because a broken variable fails without an error. Mark inserted content clearly, so the model can tell it apart from your instructions.
- [4] Prompts as code. Keep prompts in version control or a prompt registry, not scattered through the code or edited live in a dashboard. Give each version an ID and log it with every call. Review prompt changes like code, and run the evaluation set before any change ships. Roll versions out and back by configuration (see 8.1 The gateway and 8.6 Routing).
- [3] Maintaining prompts. Improve prompts from error analysis (see 3.3 Human labeling): read failures, change one thing, re-run the evaluation set. A special-case rule for every failure makes prompts long and contradictory, so rewrite them from scratch now and then. Prompts tuned for one model often do worse on another, so re-test them when you switch.
- [2] Automatic prompt optimization. Tools such as DSPy search for better instructions and examples against a metric on your evaluation set. The results are only as good as the metric, and they can overfit the evaluation set.

Check: Put one production prompt under version control, with its version ID logged on every call. Rewrite it from error analysis, and show on your evaluation set that the new version beats the old one. Roll it out and back by changing only configuration.

### 2.2 Context engineering [4]

From Layer 4, Context, memory and retrieval. Start by logging and reading the full context of real calls.

Context engineering means choosing what goes into the model's context on each call, and in what order. How models use long inputs is covered in 1.2 Capabilities and limits. This section covers what to put in them.

- [4] Check the context first. When an answer is wrong, look at the context before blaming the model. Was the needed information there? Was it buried, cut off, contradicted or out of date? Many failures that look like model failures are context failures. You can only check if you log the full context of each call (see 6.16 Observability in context, memory and retrieval).
- [4] The smallest context that works. More context isn't better. Irrelevant material distracts the model, quality tends to fall as the context grows, and every extra token adds cost and latency. Aim for the smallest set of information that lets the model do the job well.
- [4] Long context or retrieval. If everything relevant fits comfortably in the window, such as one contract or a small handbook, put it all in. It's simpler, it avoids retrieval errors, and prompt caching makes repeated use cheaper. Loading a whole knowledge base into a cached prompt is sometimes called cache-augmented generation. Retrieve when the material is too large, changes often or depends on user permissions, or when cost and latency per call matter. Many products retrieve a few whole documents rather than many small fragments. Test the options on your evaluation set.
- [4] A budget for each part. The system prompt, tools, examples, documents, history, memories and the user's request all share one window, and the output needs room too. For each feature, decide how many tokens each part may use and what gets cut first when a call runs over, such as the oldest history or the lowest-ranked documents. Count tokens before sending.
- [3] How context goes wrong. Four common patterns:
  - A mistake or made-up fact enters the context, and the model keeps building on it (poisoning).
  - So much material piles up that the model loses track of what matters (distraction).
  - Irrelevant documents or tools lead the model astray (confusion).
  - Parts of the context contradict each other, such as an old and a new version of a policy (clash).
- [3] Order and structure. Put stable content first and changing content last, even small things like the current date, so prompt caching keeps working (see 2.4 KV-cache management). With long documents, put the question after them. Provider guides differ on the details, so test the order on your model. Separate instructions, documents and user input with clear markers, such as XML tags or Markdown headings, and label each document with its source and date.
- [3] Tools in the context. Every tool definition takes tokens, and dozens of similar tools make wrong choices more likely. Load only the tools a task needs, or let the model search for the tool it needs. Designing tools is covered in 5.3 Designing tools.
- [3] Fetching context when needed. Instead of loading everything up front, let the model fetch information as it goes: through a search tool, a file it can open, or detailed instructions it loads only for the task at hand. This keeps the context small, at the cost of extra steps.

Check: Log 50 real calls for one feature and break each context into its parts in tokens: instructions, tools, history, documents and memories. Remove or shrink the part that adds least, and show on your evaluation set that quality holds while cost falls. Then sort 20 failures into missing, buried or conflicting context, or model errors.

### 2.3 Conversation history [3]

From Layer 4, Context, memory and retrieval.

Most model APIs are stateless, so your application resends the conversation on every call (see 1.1 Working with a model). This section covers keeping that history useful as it grows, within one session or agent task. Memory across sessions is covered in 6.9 Long-term memory.

- [3] Storing conversations. Keep each conversation in your own database, with messages, tool calls and results in order. Some providers can store conversation state and continue from an ID. That saves work, but ties you to the provider and its data retention rules.
- [3] Why history gets expensive. Every call resends all earlier turns, so each turn costs more than the last, and old material crowds the window. Agent runs grow fastest, because tool results are often long.
- [3] Trimming history. The main ways are:
  - keep only the most recent turns
  - summarize older turns into a running summary
  - shorten or drop old tool results once the model has used them
  - compact: near the limit, replace the history with a summary and carry on

  Some providers and agent frameworks do this for you. Whichever you use, test what gets lost.
- [3] What summaries lose. Summaries drop exact numbers, names, decisions and constraints the user set earlier. Keep what must survive, such as the user's goal and constraints, in a separate block that's never summarized.
- [3] Notes outside the context. Long agent tasks keep notes outside the context, such as a to-do list or a progress file, and read them back when needed. This lets a task continue across many context windows, even after the history has been compacted.
- [3] History and caching. Adding messages to the end keeps the prompt cache working. Editing or removing earlier messages breaks it from that point on (see 2.4 KV-cache management). So trim in occasional large steps, not a little on every call.
- [2] Sub-agents. A sub-agent can do a search-heavy piece of work in its own context and return only a short result, which keeps the main context clean. Coordinating sub-agents is covered in 7.8 Multi-agent systems.

Check: Run a 50-turn conversation or a long agent task through your product, and chart tokens and cost per call. Add a trimming or compaction strategy. Show that cost levels off while questions about facts from early turns are still answered correctly.

### 2.4 KV-cache management [2, some topics 3]

From Layer 6, Inference and serving. Focus on how provider prompt caching works.

The KV cache stores the intermediate results for each conversation. How much memory it takes is covered in 13.2 Compute arithmetic. This section covers how engines manage it.

- [2] Paged memory. Early servers reserved one large block of memory per request for its maximum possible length, and most of that block went unused. PagedAttention, introduced by vLLM, stores the KV cache in small fixed-size blocks, like an operating system's virtual memory. This fits many more requests on a GPU at once.
- [2] Prefix caching. When requests start the same way, such as with a shared system prompt, the engine can reuse the KV cache it already computed for that shared part. This cuts both time to first token and compute.
- [3] How provider prompt caching works. Provider prompt caching is prefix caching sold as a feature. The match has to be exact from the first token, so anything that changes early in the prompt, even a timestamp, breaks the cache for everything after it. Cached entries expire, often after a few minutes without use. Providers report cache hits in each response, so track your hit rate. Where to put content in a prompt is covered in 1.7 Cost and latency in foundation models and 2.2 Context engineering.
- [2] Cache pressure. When the KV cache is full, the engine can't take on more requests, so latency rises sharply. Long contexts and many simultaneous conversations fill it fastest.
- [1] Cache-aware routing. With several servers, sending requests that share a prefix to the same server raises cache hits. Some serving systems do this automatically.
- [1] Shrinking and moving the cache. Newer attention designs, such as grouped-query attention and multi-head latent attention, shrink the KV cache. Engines can also store the cache at lower precision, or move it to CPU memory or disk and back.

Check: Measure time to first token for a long prompt twice: once with a cold cache and once with a warm one. You can do this on a provider or on your own engine. Then change one early token, such as a timestamp, and show that the cache hit disappears.

### About Observability

Ordinary observability asks whether a service is up, fast and free of errors. That still matters for AI features, but they usually fail another way: the request succeeds, quickly, and the answer is wrong. The model misread a document, retrieval found an outdated version, a tool returned an error the model ignored, or an agent reported success it didn't achieve. None of these raises an error. You see them only if each request records what went in, what happened at each step and what came out, and if that record links to the signs that something went wrong, such as a complaint, a thumbs-down or a failed evaluation.

Observability answers three questions:

1. What exactly happened on this request? (2.6 Tracing, 2.7 What to record)
2. Is the system healthy, and is anything getting worse? (9.3 Metrics and dashboards, 9.10 Alerts)
3. Why did something go wrong, and how many other requests did the same? (2.8 Debugging from traces)

You build and instrument the application yourself, so most concepts in this concern are level 3, and two things are level 4:
- deciding what every trace records, including the IDs and versions that let you explain and reproduce any result
- tracing a bad result back to its cause

Your company probably already has observability tools and a team that runs them. What changes with AI is mostly what you record and how you read it. So learn the pipeline behind the tools, such as collectors and storage, only to level 2. Learn infrastructure metrics, such as GPU and serving-engine metrics, to level 2 unless you self-host (see 1.12 Observability in inference and serving and 13.14 Observability in compute).

The layers' own sections cover observability as it applies to each layer, such as logging the assembled context in Layer 4 and tracing tool calls in Layer 3. The Observability sections cover what they share, and 14.2 Observability at each layer maps where the rest is covered.

### 2.5 Observability foundations [3]

From the Observability concern. Focus on why AI failures don't raise errors.

These ideas come up in every other section, so learn them first.

- [3] Wrong answers don't raise errors. Most AI failures come back as successful requests: a confident wrong answer, an ignored instruction, a tool called with the wrong arguments, or an agent that stopped early. Error rates and latency miss them all. To see them, record the content of each request, not just its status, and attach signs of quality such as user feedback and evaluation scores (see 3.12 Observability in evaluation and 11.1 Evaluation in production).
- [3] Traces, logs and metrics. A trace follows one request through every step. Each step is a span, with a start and end time, a status, details called attributes, and a parent span. Logs record single events. Metrics are numbers aggregated over time, such as request counts, error rates and latency percentiles. Metrics show that something changed. Traces and logs show what happened.
- [3] One wide record per step. Record each step as one structured event with many fields, such as IDs, versions, token counts, cost, latency and outcome, rather than as scattered lines of text. Structured records can be filtered and grouped by any field, like rows in a database.
- [3] Asking new questions. Dashboards and alerts answer questions you thought of in advance. AI failures are often new, such as "which answers this week quoted the old refund policy?" Record rich, searchable data, so you can answer questions nobody planned for. This is the difference between monitoring and observability.
- [2] Standard service signals. The usual health signals for any service still apply: how many requests arrive, how many fail and how long they take, plus how close resources are to their limits. Google's four golden signals and the RED and USE methods are common checklists.

Check: For one AI feature, list the ways it could fail without raising an error. For each, name the recorded field or signal that would reveal it, and add whatever is missing.

### 2.6 Tracing [3]

From the Observability concern. Start with one feature end to end.

A trace follows one request through agents, tools and model calls. Tracing whole agent runs is covered in 7.10 Observability in agents and orchestration, and tracing tool calls in 5.15 Observability in tools and protocols. This section covers how tracing works for any AI feature.

- [3] Spans for AI steps. Give each kind of step its own span: the incoming request, each agent step, model calls, retrieval, embedding, reranking, tool calls and guardrail checks. Name spans for what they do, and nest them the way the work is nested, so a trace reads as a tree.
- [3] Passing the trace along. The trace ID has to travel with the request across services, queues and background workers, usually in a standard header defined by W3C Trace Context. Without it, the records kept by your API, your gateway (see 8.9 Observability in model gateway and routing) and your workers can't be joined into one trace. Store the trace ID with each long-running task's state, so later steps join the same trace.
- [3] Conversations and users. Group traces into sessions, such as a conversation or an agent task, and tag each trace with user and customer IDs. Then you can open everything in one conversation, or everything one customer did last week.
- [3] OpenTelemetry. OpenTelemetry is the open standard for traces, metrics and logs. It has SDKs for most languages and a collector that sends data to any backend. Its conventions for generative AI give standard names for model calls, token counts, tool calls and agents, and are still changing. OpenInference and OpenLLMetry are related sets of conventions and libraries. Instrumenting to a standard lets you switch tools later.
- [3] Automatic and manual instrumentation. Libraries can trace model SDKs, agent frameworks and vector databases with little code. Add your own spans around your own logic. Check what automatic tracing leaves out: many libraries skip prompt and output text by default, for privacy, and some miss streamed output or reasoning tokens.
- [3] Streaming and long runs. For a streamed response, record when the first token arrived, and record the full output when the stream ends, including streams that were cut off or cancelled. Agent runs can last hours and produce very large traces. Send spans as each step finishes, so a crash doesn't lose the whole trace.
- [3] Logs in the trace. Put the trace ID on every log line, so logs from any service can be pulled up next to the trace they belong to.
- [2] Sampling. At high volume, you may keep only some traces. Deciding at the start of each request (head sampling) is cheap, but keeps a random mix. Deciding after it ends (tail sampling) lets you keep every error, slow or costly request, and thumbs-down. Many AI teams keep every trace for low-volume features, since each one is possible evaluation data.

Check: Trace one feature end to end, from the user's request through the gateway, retrieval, model calls and tool calls, including one step that runs in a background worker. Show the whole request as a single tree, open it from the conversation it belongs to, and find its log lines by trace ID.

### 2.7 What to record [4]

From the Observability concern. Treat it as a standard for your whole team.

The layers' own sections list what to record at each step: each model call in 1.11 Observability in foundation models, the assembled context in 6.16 Observability in context, memory and retrieval, tool calls in 5.15 Observability in tools and protocols, routes, retries and cost in 8.9 Observability in model gateway and routing, and what users do with each result in 4.4 Feedback and measurement. This section covers the standard that ties them together.

- [4] One standard for every call. Decide once, for the whole team, what every model call and step records, and enforce it in the gateway or a shared client library (see 8.1 The gateway) rather than feature by feature. A field that one feature skipped is missing exactly when you need it.
- [4] Versions on everything. Record the version of everything that shaped a result: the prompt, the model snapshot, the retrieval index and embedding model, the tool definitions, the configuration and feature flags, and the code release. Without versions, you can't tell which change caused a problem, and you can't rebuild what the model saw.
- [4] Enough to replay. Record each step's full input and output, or enough to rebuild them exactly. The test is whether you can rerun a failed request with one thing changed (see 7.10 Observability in agents and orchestration). If you can't, the record is missing something.
- [3] IDs that connect everything. Every trace needs its trace ID, its session or task ID, and user and customer IDs. Everything that refers to a request, such as feedback, support tickets, evaluation scores and audit records, should carry the request's trace ID. A thumbs-down that can't be tied to its trace tells you almost nothing (see 4.4 Feedback and measurement and 4.12 Observability in product and UX).
- [3] What happened next. Record outcomes as well as inputs: what the user did with the result, guardrail verdicts, evaluation scores (see 3.12 Observability in evaluation), errors, retries and fallbacks, and whether the task finished.
- [3] Content and metadata apart. Full prompts and outputs are large and hold user data. Many teams keep metadata, such as IDs, versions, tokens, cost and latency, in the tracing tool, and store full content separately, with tighter access and shorter retention, linked by ID. Decide for each feature what content you may keep, and for how long (see 9.17 Governance and safety in observability).

Check: Write the standard record for one product's model calls and agent steps, and enforce it in the gateway or client library. Then take a failed request recorded under the standard. Show exactly what the model saw, which versions produced the result and what the user did next, and rerun it with one change.

### 2.8 Debugging from traces [3, some topics 4]

From the Observability concern. Start reading traces from the first day.

Traces are how you find and fix failures. Reading many traces to find the common kinds of failure (error analysis) is covered in 3.3 Human labeling, and measuring which step of a retrieval pipeline failed in 6.3 Evaluation in context, memory and retrieval. This section covers investigating failures.

- [4] From a bad result to its cause. Walk the trace step by step. Was the needed information in the context (see 2.2 Context engineering)? Did each tool return what it should have? Did the model follow its instructions, and did it stop for the reason you expected? Most causes are visible in the trace once you look, and often the model isn't the cause at all.
- [3] Every signal leads to a trace. User reports, support tickets, alerts and failed evaluations should each open the traces behind them in one click. If finding the trace takes an hour, it won't get done.
- [3] Searching traces. Find traces by any field: user, customer, feature, version, stop reason, cost, latency, tool error, feedback, evaluation result, or text in the input or output. Once you understand one bad trace, searching tells you how many others share its cause.
- [3] Finding patterns. Group many traces to see what they share: cluster requests by topic with embeddings, group failures by step or error, and compare traces from before and after a change. Some tools use a model to summarize or label traces at scale. Check its labels as you would any judge's (see 3.8 LLM judges).
- [3] Reproducing a failure. Rerun the recorded request with the same inputs, then with one thing changed, to confirm a fix on the exact case that failed (see 7.10 Observability in agents and orchestration). Outputs vary between runs, so rerun it several times before you conclude anything.
- [3] Incidents. When something breaks in production, establish when it started, what changed at that time (a release, a prompt or model version, a provider incident or a document sync), and who's affected. Write a short review afterwards, and add the failing cases to your evaluation set (see 3.4 Evaluation: building evaluation datasets).

Check: Take three real complaints or failed evaluations. For each, find the trace in under a minute, name the step that caused the failure and count how many other requests share the cause. Rerun one with a fix, and show that it passes several times.

### Key reading for Stage 2

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 5 covers prompt engineering, chapter 6 covers RAG, agents and memory, and chapter 10 covers monitoring and observability for AI applications.
- Anthropic, "Effective context engineering for AI agents" (2025), and Yichao Ji, "Context Engineering for AI Agents: Lessons from Building Manus" (2025).
- Drew Breunig, "How Long Contexts Fail" and "How to Fix Your Context" (2025), on the ways context goes wrong.
- Your model providers' prompting guides, and the documentation for your vector database or search engine.
- Kwon et al., "Efficient Memory Management for Large Language Model Serving with PagedAttention" (2023), the vLLM paper.
- Charity Majors, Liz Fong-Jones and George Miranda, *Observability Engineering* (O'Reilly, 2022).
- Cindy Sridharan, *Distributed Systems Observability* (O'Reilly, 2018).
- Sigelman et al., "Dapper, a Large-Scale Distributed Systems Tracing Infrastructure" (Google, 2010).
- The OpenTelemetry documentation, including its semantic conventions for generative AI.
- Hamel Husain, "A Field Guide to Rapidly Improving AI Products" (2025), on looking at your data and building simple tools to read traces.

## Stage 3. Evaluation

Before you build features, you need a way to tell whether a change made things better. This stage covers reading real outputs, labeling them, building evaluation sets, scoring outputs with code, model judges and people, and comparing versions fairly. It ends with choosing models, the first decision you'll make with an evaluation set.

| Section | Level | Why this level | From |
|---|---|---|---|
| 3.1 The evaluation loop | 4 | Evaluations built from real failures catch what generic metrics miss | Evaluation |
| 3.2 Data foundations | 3 | Every other section uses them | Layer 10 |
| 3.3 Human labeling | 3 (error analysis 4) | You and your domain experts produce the correct answers | Layer 10 |
| 3.4 Evaluation: building evaluation datasets | 4 | The dataset you'll build and rely on most | Layer 10 |
| 3.5 Defining what to measure | 3 (some topics 4) | You can't check "good" until you've said what it means | Evaluation |
| 3.6 Code-based checks | 3 | Fast, free and exact wherever code can decide | Evaluation |
| 3.7 Evaluation in development | 3 (some topics 4) | A change that skips evaluation ships untested | Evaluation |
| 3.8 LLM judges | 3 (some topics 4) | They scale human judgment, but only once they agree with people | Evaluation |
| 3.9 Human review | 3 (some topics 4) | People decide what good means and check what nothing else can | Evaluation |
| 3.10 Comparing versions | 3 (some topics 4) | Small differences between versions are often chance | Evaluation |
| 3.11 Synthetic data | 3 | Fills gaps before you have users, and covers rare or sensitive cases | Layer 10 |
| 3.12 Observability in evaluation | 3 | Traces are what you evaluate | Evaluation |
| 3.13 Cost and latency in evaluation | 3 | Every comparison needs them, and evaluation costs money too | Evaluation |
| 3.14 Reliability in evaluation | 3 | An answer that changes between runs is a quality problem | Evaluation |
| 3.15 Security in evaluation | 2–3 | Judges and graders can be fooled | Evaluation |
| 3.16 Governance and safety in evaluation | 2–3 | Customers and regulators increasingly ask for test results | Evaluation |
| 3.17 Choosing models | 4 | Your most important recurring decision in this layer | Layer 8 |
| 3.18 Evaluation in foundation models | 3 | Model judges favor answers like their own | Layer 8 |
| 3.19 Public benchmarks | 2 | They narrow a list of models but don't predict your results | Evaluation |
| 3.20 Pre-training corpora | 1 (some topics 2) | Labs build them; you only need them to explain model behavior | Layer 10 |

### About Evaluation

Many of the checks in the layers' sections end the same way: show that it works on your evaluation set. This concern is how you do that well. You can buy models, gateways and vector databases, but no one can sell you an evaluation of your product, because only you know what doing its job means for it. Evaluation is also how you find out whether anything else in this curriculum worked. That makes it the deepest cross-cutting concern for an application engineer.

For every change, evaluation answers four questions:

1. What does doing its job mean for this feature? (3.1 The evaluation loop, 3.5 Defining what to measure)
2. How is each output scored? (3.6 Code-based checks, 3.8 LLM judges, 3.9 Human review)
3. Is the new version really better than the old one? (3.10 Comparing versions, 3.7 Evaluation in development)
4. Does it hold up with real users, and against people trying to break it? (11.1 Evaluation in production, 11.3 Experiments with real users, 10.11 Safety and adversarial testing)

Most concepts in this concern are level 3, and four things are level 4:
- the evaluation loop: building evaluations from the failures you actually see, not from generic metrics
- defining what good means, as criteria that people and code can check
- building LLM judges and showing that they agree with people
- running evaluation on every change, and deciding in advance what a change must show to ship

The layers' own sections cover evaluation as it applies to each layer, such as retrieval metrics in Layer 4 and agent task success in Layer 2, and Layer 10 covers building evaluation datasets (see 3.4 Evaluation: building evaluation datasets). The Evaluation sections cover the methods they all share, and 14.1 Evaluation at each layer maps where the rest is covered. Learn public benchmarks to level 2, because they only narrow a list of models before your own evaluation decides. Experiments with real users are level 3: data scientists often run the experimentation platform, but you design the tests for AI changes.

### 3.1 The evaluation loop [4]

From the Evaluation concern. Start with error analysis of real outputs (see 3.3 Human labeling).

Evaluation is a loop, not a one-off test. You read what the system produces, decide what counts as a failure, measure it, change something and measure again.

- [4] Start from real outputs. Read a sample of real or realistic outputs, note what went wrong in each, and group the notes into failure types. This is error analysis, covered in 3.3 Human labeling. Then build evaluations for the failure types that matter. Ready-made scores for qualities such as "helpfulness" or "coherence" rarely match the failures your product actually has.
- [4] Evaluation first. Before building a feature, write down what success looks like and collect a small set of test cases, the way tests come before code in test-driven development. Judge every change by those cases. Over time, the evaluation set becomes the working definition of what the feature must do.
- [3] Beyond trying a few examples. Trying a handful of inputs by hand, often called a vibe check, is how every feature starts, and it's fine early on. It stops working once the feature handles many kinds of input, because fixing one case quietly breaks another. Write the cases down the first time a change breaks something you had already fixed.
- [3] Three tiers. Cheap code checks run on every change. Larger suites with LLM judges and human review run before releases. Experiments with real users run in production. Each tier catches what the cheaper ones miss, and costs more.
- [4] One change at a time. Run the evaluation, read the failures, change one thing and run it again. Keep every run's results with the versions it used, so you can see which change helped. The loop never ends, because new users, models and features bring new kinds of failure.
- [4] Who decides what good means. Someone who knows the domain has to decide what counts as a good output. On a small team, making one domain expert the final judge keeps the standard consistent. Engineers grading outputs in a field they don't know miss the failures that matter most.

Check: For one feature, run error analysis on 100 real outputs (see 3.3 Human labeling) and turn the three most common failure types into evaluations. Use them to judge three changes, and show which change fixed a failure type without breaking another.

### About Layer 10: Data

Learn the data you create and own in depth. For data that other people create for you, a lighter understanding is enough. Model labs build pre-training corpora, so you only need to know what goes into them. Your users' feedback comes only from your product, so you should be the expert on it.

Most of your data work involves small datasets that you build carefully: evaluation sets, fine-tuning sets and feedback from your product. You build all of them with the same five skills:

1. Reading real outputs.
2. Writing labeling rules.
3. Labeling.
4. Generating synthetic examples.
5. Writing graders.

Once you learn these skills, you use them in every Layer 10 section, in Stages 3, 11 and 12.

### 3.2 Data foundations [3]

From Layer 10, Data.

These basics come up in every other section, so learn them first.

- [3] File formats. JSONL, with one JSON example per line, is the standard format for evaluation and fine-tuning data. Learn the chat message format, which has system, user, assistant and tool roles and includes tool calls. Also learn Parquet and CSV for larger tables.
- [3] Working with data. SQL, DuckDB, pandas or Polars, and the Hugging Face `datasets` library and Hub.
- [3] Splits and leakage. Keep training, validation and test sets separate. Split by user or by conversation, not by row, so near-copies of an example don't end up on both sides.
- [3] Deduplication. Find exact matches by hash, and near-duplicates with MinHash or embedding similarity.
- [3] Sampling. Random and stratified sampling (for example, by topic, user segment or difficulty), and how to sample production traffic for review.
- [3] Basic statistics. Distributions, class imbalance, confidence intervals, and how many examples you need before you can trust a difference between two versions.
- [3] Versioning. Save each dataset as a fixed snapshot with a version ID, and record which snapshot you used with which prompt and model. Tools such as DVC or Hugging Face Hub revisions help, but a well-kept folder of snapshots is enough to start.
- [3] Embeddings for analysis. Clustering, near-duplicate detection, topic discovery and measuring diversity. Using embeddings to build search is covered in 6.5 Search.
- [2] Tokens. How to count them, and how tokenization differs across languages and code. These differences change cost and how much fits in the context window.
- [2] Pipelines. Enough data engineering to run a scheduled job that turns logs into datasets, typically a warehouse, SQL and a job scheduler.
- [2] Multimodal data. Images, audio, screenshots and scanned documents, and the extra labeling and privacy work they need because of faces, voices and handwriting.

Check: Take 10,000 logged conversations and deduplicate them. Draw a stratified sample of 300 and split it without leakage. Save it as a versioned JSONL snapshot with a short note on where the data came from.

### 3.3 Human labeling [3]

From Layer 10, Data. Learn error analysis and labeling, then evaluation datasets. You'll judge everything that follows with these.

Evaluation and fine-tuning both need correct answers to compare against. On most app teams, the people who label are the engineers and a few domain experts.

- [4] Error analysis. Read real outputs and write a short note on each failure. Then group the notes into a small set of failure types. Of all the data work on this list, this does the most to improve a product. It is shared with Evaluation (see 3.1 The evaluation loop).
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

### 3.4 Evaluation: building evaluation datasets [4]

From Layer 10, Data.

How you measure is covered in 3.5 Defining what to measure to 3.10 Comparing versions. This part covers the data you measure with. For an application engineer, the evaluation set is the dataset you will build and rely on most.

- [4] Types of sets. Regression sets hold past failures that must stay fixed. Capability sets cover what the product should do. Adversarial sets cover attacks and abuse. Slices break results down by user segment, language or difficulty.
- [4] Sources. Production traces, synthetic generation, domain experts, and public benchmarks. Use public benchmarks carefully, because of contamination.
- [4] Coverage. Map out the dimensions of your task, and check that each important combination has examples.
- [4] Reference answers. Decide which cases need a known correct answer, and which can be graded against criteria alone.
- [4] Judge calibration sets. Examples labeled by people and used to measure how often an LLM judge agrees with them.
- [4] Maintenance. Give each set a version, and retire cases that every version passes. Add new failure types as they appear. Don't tune prompts against the test set so much that it stops predicting real results.
- [3] Size. Enough examples that you can trust a difference between versions. Learn confidence intervals and bootstrapping.
- [3] Retrieval test sets. Queries paired with the passages that retrieval should find.
- [2] Public benchmarks. What the common benchmarks measure, and why they rarely predict results on your task.

### 3.5 Defining what to measure [3, some topics 4]

From the Evaluation concern.

An evaluation is only as good as its definition of success. This section covers turning a goal into things you can check.

- [4] From goals to criteria. Turn "the summaries should be good" into criteria that two reviewers would apply the same way, such as "includes every action item", "makes no claim that isn't in the source" and "is under 150 words". Write a definition for each, with one passing and one failing example.
- [4] One result for each failure type. Measure each criterion separately, as pass or fail, rather than rolling everything into one score out of 10. Separate results show what to fix, and one overall score hides it. Pass/fail labels are also more consistent than scales (see 3.3 Human labeling).
- [3] What one test case is. Decide whether a case is a single output, one turn of a conversation, a whole conversation or a whole agent task. For chat, also grade whole conversations: did the user get what they came for, and did the model keep to its instructions to the end? Simulated users make multi-turn tests repeatable (see 3.11 Synthetic data), and agent tasks are graded by their end state (see 7.4 Evaluation in agents and orchestration).
- [3] With or without a reference answer. Some criteria compare the output with a known correct answer. Others check the output on its own, against a rule or a rubric. Only checks that need no reference answer work on production traffic, where nobody knows the correct answer. Which cases need reference answers is covered in 3.4 Evaluation: building evaluation datasets.
- [3] Quality, cost and latency together. Report cost and latency next to quality for every version you compare. Plot quality against cost, and choose among the versions that no other version beats on both. A version that is slightly better but twice as slow may not be better.
- [3] Baselines. A score means little on its own. Compare it with the current version, with a simple approach such as a rule or a keyword search, and, where you can, with how well people do the task today.
- [3] Guardrail metrics. Besides the metric you're trying to improve, choose metrics that mustn't get worse, such as safety failures, refusals of reasonable requests, format errors and cost. Product guardrail metrics are covered in 4.4 Feedback and measurement.
- [3] Standard metrics for standard tasks. For classification, measure precision and recall for each class. Precision is how many of the items given a label really have it, and recall is how many of the items that have it were given it. Decide which mistake costs more, a false alarm or a miss, and weigh the two to match, because overall accuracy hides failures on rare classes. For extraction, measure accuracy field by field. Your judges, routers and guardrails are classifiers too, so the same metrics apply to them.
- [3] Metrics become targets. A metric you push hard stops measuring what you meant, which is known as Goodhart's law. Prompts tuned against one score find ways to raise it without improving the product. Keep reading outputs, not just scores.
- [2] Scores and thresholds. If your product acts on a score, such as a guardrail that blocks above a threshold or a cascade that escalates uncertain answers (see 8.6 Routing), check that the scores mean what they say. About 90% of the answers scored at 90% confidence should be right. Set thresholds from labeled data, by seeing how precision and recall change as the threshold moves.
- [2] Older automatic metrics. BLEU and ROUGE count the words an output shares with a reference answer, and embedding similarity compares meanings. They're cheap, but on open-ended text they agree poorly with people, so use them only for narrow tasks such as translation. Perplexity, which measures how predictable a text is to a model, is mainly a training metric.

Check: For one feature, write five to eight pass/fail criteria from your error analysis, each with a definition and examples. Have a colleague apply them to the same 30 outputs as you, and rewrite every criterion you disagreed on.

### 3.6 Code-based checks [3]

From the Evaluation concern.

Checks written in code are fast, free and exact, and they give the same verdict every time. Use them for everything code can decide, and save LLM judges for the rest. The same checks can also score RL environments (see 12.7 RL environments).

- [3] Format checks. Valid JSON that matches its schema, required fields present, values within allowed ranges, length limits, the right language, and no leftover template text or placeholders.
- [3] Content checks. Required facts or phrases present, forbidden ones absent, citations that point to real sources, numbers that match the source document, and links that work. String matching and regular expressions cover many of these.
- [3] Running the output. When the output can be run, run it. Execute generated code against tests, run generated SQL and compare its results with the expected ones, or send generated tool arguments to a sandbox. A correct answer can be written many ways, so checking what the output does beats comparing its text. Checking an agent's end state is covered in 7.4 Evaluation in agents and orchestration.
- [3] Comparing with references. Normalize before comparing: case, spacing, and number and date formats. Compare lists as sets where order doesn't matter, allow a tolerance for numbers, and compare structured output field by field.
- [3] Behavioral tests. Change inputs in controlled ways and check how the output responds. A typo or a different customer name shouldn't change the answer (an invariance test). Adding a constraint should change it in a predictable way (a directional test). Simple cases that must always work should always pass (a minimum functionality test). The CheckList method named these three kinds of test.
- [3] Checking the checks. Run each check on outputs you've labeled by hand, including correct answers in unusual formats. A check that fails correct outputs does as much harm as one that passes wrong ones. Anything tuned against a check can also learn to pass it without doing the task (see reward hacking in 12.7 RL environments).

Check: Write code checks for every criterion that code can decide. Run them on 200 outputs, and hand-check 30 of their verdicts. Fix every check that failed a correct output or passed a wrong one.

### 3.7 Evaluation in development [3, some topics 4]

From the Evaluation concern. Start with a simple harness that runs on every change, and add the rest as your evaluation grows.

Evaluation helps only if it runs before changes ship. This section covers building it into everyday development.

- [3] An evaluation harness. A harness runs a dataset through your system, applies the checks and judges, and records the results. Record the version of everything with each run: the dataset, prompts, models, checks, judges and code. Then any run can be repeated and compared with earlier ones.
- [4] Evaluation on every change. Run the evaluation set before any change ships, whether it's a prompt, a model, a tool, a retrieval setting or a route. Each layer's sections say this for its own kind of change. Make it part of code review, with the results posted on the change.
- [3] Gates in CI. Run fast checks on every commit in continuous integration (CI), and larger suites before each release. Block a merge when a regression case fails or a guardrail metric falls past a set limit. Set limits wider than normal run-to-run variation, or the gate fails at random and people learn to ignore it.
- [3] Fast, cheap runs. Cache model responses, so rerunning unchanged cases costs nothing. Run cases in parallel, and send large runs to batch APIs (see 8.12 Cloud capacity). A suite that takes an hour runs rarely. One that takes five minutes runs on every change.
- [3] Reading what changed. Compare two runs side by side: scores for each criterion and slice, and the cases whose verdicts changed, with both outputs. Most of what you learn comes from reading the cases that changed.
- [3] Replaying real inputs. Before a release, run a recent sample of real inputs through the old and the new version, and compare the outputs with a comparison judge or by reading them. This catches changes your curated cases don't cover. Sending copies of live traffic to a new version without showing users (shadow traffic) is covered in 8.6 Routing.
- [3] Tools. Open-source frameworks such as promptfoo, Inspect and DeepEval run evaluations from code or configuration. Platforms such as Braintrust, LangSmith, Langfuse and Arize Phoenix combine tracing, datasets, evaluation and human review. Learn one well. A script and a spreadsheet are enough to start, and many teams build their own review screens.

Check: Set up a harness that runs on every pull request that changes a prompt, model or tool. Have it post scores and changed cases on the pull request, and block the merge when a regression case fails. Keep a full run under ten minutes.

### 3.8 LLM judges [3, some topics 4]

From the Evaluation concern. Learn this once you have human labels to check judges against, together with human review.

An LLM judge is a model call whose job is to grade another output. It can apply criteria that code can't, such as tone, or whether a summary left out something important. But it's a model call like any other: it can be wrong, biased, inconsistent and expensive, so it needs evaluating too. Choosing the judge's model is covered in 3.18 Evaluation in foundation models.

- [4] Writing a judge. Give each judge one criterion, and ask for pass or fail rather than a score from 1 to 10, which models use as inconsistently as people do. Define pass and fail precisely, with labeled examples of each, especially borderline ones. Ask for a short explanation before the verdict. It tends to make verdicts more accurate, and it shows you why the judge decided as it did. Give the judge what it needs: the input, the output, and the source documents or reference answer where they matter.
- [4] Checking a judge against people. Before trusting a judge, compare its verdicts with human labels (see judge calibration sets in 3.4 Evaluation: building evaluation datasets). Measure two things separately: how often it catches real failures, and how often it passes real successes. Overall agreement misleads: when one output in ten fails, a judge that passes everything agrees with people 90% of the time. Improve the judge's prompt on one part of the labeled data and measure it on another, so the result isn't inflated.
- [3] Correcting the judge's numbers. A judge's pass rate is an estimate with known errors. If you know how often the judge misses failures and how often it fails good outputs, you can correct the pass rate it reports and put a range around the corrected figure.
- [3] Kinds of judges. A judge can grade one output against criteria, compare two outputs and pick the better one, or grade an output against a reference answer. Comparisons suit subjective quality and choosing between two versions. Grading single outputs gives pass rates you can track over time.
- [3] A checklist for each case. For open-ended tasks, write a short checklist for each test case of what a good answer must include or avoid, and have the judge check each item. Specific items are easier to grade consistently than overall quality.
- [3] Judge biases. Judges tend to favor the answer shown first, longer answers and answers from their own model family (see 3.18 Evaluation in foundation models). Run each comparison in both orders, and count it as a tie when the two verdicts disagree. Check length separately if it matters.
- [3] Stable judges. Pin the judge's model snapshot, and version its prompt like any other prompt (see 2.1 Prompts). A new judge model changes every score, so check it against people again before comparing old results with new ones.
- [3] Criteria that shift. People refine what they mean by good as they read more outputs. This is called criteria drift. Expect the judge's definitions to change during the first rounds of labeling, and check the judge again after each change.
- [2] Judges that can't do the task. A judge can't reliably check what it couldn't do itself, such as a hard proof or a fact it doesn't know. Give it the reference answer or the source, or use a code check.
- [2] Panels and trained judges. A panel of smaller judges from different model families can do as well as one large judge, at lower cost. Models trained to evaluate, including reward models (see 12.6 Learning from human preferences), can score outputs too. Small models trained for one check, such as whether a claim follows from a source or whether text is toxic, are cheaper and faster still. A strong model with a good prompt is the usual starting point.

Check: Build a judge for one criterion. On 100 human labels it wasn't tuned on, report how often it catches failures and how often it passes successes. Improve it until both are acceptable, then report its corrected pass rate on a sample of production outputs.

### 3.9 Human review [3, some topics 4]

From the Evaluation concern.

People define what good means, and they check what code and judges can't. Writing labeling rules, measuring agreement between labelers and running a labeling process are covered in 3.3 Human labeling. This section covers using people's judgment to evaluate.

- [4] What people are for. People decide what counts as good, label the data that judges are checked against, review what automated checks can't decide, and check high-stakes outputs directly. As automated evaluation earns trust, people review fewer outputs. But someone should still read real outputs every week.
- [3] Side-by-side comparisons. To choose between two versions, show reviewers both outputs for the same input, in random order and without saying which is which. Ask which is better, or whether they're tied. People compare more consistently than they score. Report the win rate with a confidence interval.
- [3] Random and flagged samples. Send reviewers a random sample of outputs to estimate how often things go wrong. Separately, send them flagged outputs, such as judge failures, user complaints and uncertain answers, to find problems. Report the two separately, because flagged outputs overstate the failure rate. Choosing what to label first is covered in 3.3 Human labeling.
- [3] Expert review. In fields such as medicine, law and finance, only experts can tell a correct answer from a plausible one. Their time is scarce, so send them the cases that need expertise, and use their labels to check your judges.
- [3] Reviewing whole traces. Show reviewers the input, the context, the tool calls and the output together. Seeing every step shows where things went wrong, not just that they did. A simple review screen built around your traces, with one-click labels, gets far more reviewing done than a spreadsheet (see 3.3 Human labeling).

Check: Run a blind side-by-side comparison of two versions of a feature on 100 inputs, with two reviewers. Report the win rate with a confidence interval, and whether your automated evaluation picked the same winner.

### 3.10 Comparing versions [3, some topics 4]

From the Evaluation concern. Learn this before you trust any small difference.

Most evaluation work compares two versions: an old and a new prompt, model or retrieval setting. Small differences between them are often just chance. The basic statistics are covered in 3.2 Data foundations. This section covers using them to compare versions.

- [3] Scores vary between runs. The same version can score differently on two runs, because model outputs vary, and so do judges. Run the evaluation more than once, or run each case several times, to see how much scores move by chance before calling a difference real. Success across repeated runs of agent tasks is covered in 7.4 Evaluation in agents and orchestration.
- [3] Error bars. Report every score with a confidence interval. With 100 cases and a 70% pass rate, the 95% interval runs from about 61% to 79%, so on its own a 3-point gain could easily be chance. Bootstrapping, which resamples the cases many times, gives an interval for any metric.
- [3] Paired comparisons. Run both versions on the same cases and compare them case by case. This is far more sensitive than comparing two overall scores, because a case that is hard for one version is usually hard for the other. Count the cases each version fixed and the cases it broke, not just the net change. McNemar's test, or a bootstrap of the differences between pairs, tells you whether the difference is likely real.
- [3] Slices. Break results down by task type, user segment, language and difficulty. An overall gain can hide a drop on a slice that matters, and slices can move in opposite directions.
- [3] Held-out cases. If you tune prompts on the same cases you report on, the score rises faster than real quality. Keep some cases aside and use them only to confirm a final choice (see 3.2 Data foundations).
- [2] Cases that aren't independent. Cases drawn from the same document or conversation tend to pass or fail together, so they carry less information than their number suggests. Group them when you compute intervals, or the intervals come out too narrow.
- [2] Many comparisons. Try twenty prompt variants, and one will look better by chance. Confirm the winner on fresh cases before you ship it.
- [4] Deciding in advance. Before the evaluation runs, agree what a change must show to ship: which criteria must improve, by how much, which must not get worse, and which failures block a release whatever the average says. Otherwise people read the results as support for what they already wanted.

Check: Run two prompt versions on the same 200 cases, three times each. Report each version's pass rate with a confidence interval, the paired difference with its interval, and the cases each version fixed and broke. Decide whether to ship, using a rule you wrote down before the runs.

### 3.11 Synthetic data [3]

From Layer 10, Data.

Synthetic data is data that a model generates. It fills gaps before you have users, and it covers rare or risky cases and cases where real data is too sensitive to use. It is also what distillation runs on.

- [3] Generation along dimensions. List what varies in your task, such as user type, intent, difficulty and scenario. Pick combinations, then have a model write a realistic input for each one. This way you control what the data covers.
- [3] Expanding from seeds. Start from real examples. Have a model write variations (Self-Instruct), harder versions (Evol-Instruct), or versions in the voice of different personas.
- [3] Generating from your documents. Pairs of questions and answers drawn from your documents, including questions that need more than one document to answer. Use them to test retrieval.
- [3] Simulated users. A model plays the user in multi-turn conversations or agent tasks.
- [3] Structured output. JSON schemas or typed models, so that every generated example can be parsed.
- [3] Quality checks. Filter examples with graders, remove duplicates and measure diversity. Verify answers where you can, for example by running generated code. Have a person review a sample.
- [3] Failure modes. Examples that all sound alike. Inputs that are cleaner and more polite than what real users write. Made-up facts. Bias that the generating model amplifies. A model that grades its own outputs too generously.
- [2] Model collapse. When models are trained repeatedly on model-generated data, they lose the variety found in real data.
- [2] Adversarial examples. Generating jailbreaks, prompt injections and out-of-scope requests for red-teaming. This is shared with Security (see 10.4 Security testing).
- [2] Privacy-preserving synthetic data. Realistic stand-ins for sensitive records, and why they can still leak real ones.
- [2] Legal limits. Provider terms on using model outputs for training, and the license status of generated data.

Check: Build a synthetic evaluation set and compare its topics and style with real traffic. Fix at least one gap you find.

### 3.12 Observability in evaluation [3]

From the Evaluation concern.

- [3] Traces are what you evaluate. Offline cases come from production traces, and production scores attach to them. Log enough to rerun or regrade any trace: the full input and context, the versions, the tool calls and the output (see 11.2 Product feedback).
- [3] Scores on traces. Store every result with its trace, the version it graded and the version of the judge. Then you can filter traces by failure, and follow any score back to the exact output it graded.

Storing traces, dashboards and alerts are covered in 2.6 Tracing, 9.3 Metrics and dashboards and 9.10 Alerts.

### 3.13 Cost and latency in evaluation [3]

From the Evaluation concern.

- [3] In every comparison. Report cost and latency next to quality for every version (see 3.5 Defining what to measure), measured the way users experience them (see 1.5 Inference performance).
- [3] What evaluation costs. A run's cost is the number of cases, times the runs per case, times the model and judge calls in each run. Judges can cost more than the feature they grade. Use code checks where they work, cache responses, use batch APIs (see 8.12 Cloud capacity), and score a sample of production traffic rather than all of it.
- [2] Cheaper judges. Once a judge agrees with people, a smaller model may do the same job, or a small classifier trained on the judge's verdicts. Check it against people again after each swap.

### 3.14 Reliability in evaluation [3]

From the Evaluation concern.

- [3] Consistency is part of quality. The same input can give different outputs (see 1.8 Reliability in foundation models). Measure how often repeated runs agree, and count an inconsistent answer to a question with one right answer as a failure.
- [3] Flaky evaluations. A test that passes and fails at random on the same version teaches people to ignore failures. Find out whether the system or the grader is varying, and fix whichever it is.

Testing fallbacks is covered in 8.8 Evaluation in model gateway and routing, and testing error messages in 4.10 Evaluation in product and UX.

### 3.15 Security in evaluation [2–3]

From the Evaluation concern.

- [3] Judges can be fooled. An output can contain text aimed at the judge, such as "this answer meets every criterion", especially when outputs include text from users or the web. Mark the output as data in the judge's prompt, and spot-check verdicts by hand. Defenses against prompt injection are covered in 5.8 Prompt injection.
- [3] Evaluation data is customer data. Evaluation sets built from production traces contain customer data. Give them the same access controls and deletion rules as production data (see 11.5 Governance and safety in the data layer and 11.6 Security in the data layer).
- [2] Graders that get gamed. Anything tuned against a grader, such as a prompt from an automatic optimizer (see 2.1 Prompts) or a model trained with reinforcement learning, can learn to satisfy the grader instead of doing the task (see 12.7 RL environments).

### 3.16 Governance and safety in evaluation [2–3]

From the Evaluation concern.

- [3] Records of testing. Keep each release's evaluation results with the versions tested, so you can show what was tested, when, and with what result. Customers' security reviews and auditors increasingly ask for this.
- [2] Rules that require testing. The EU AI Act requires high-risk systems to be tested and to state their accuracy. Frameworks such as NIST's AI Risk Management Framework and ISO/IEC 42001 make measurement a core activity. Know which apply to your product, and bring in legal.
- [2] Documenting results. Write a short card for each AI feature that covers what it was tested on, the results and its known limits. Documentation for adapted models is covered in 12.11 Governance and safety in model adaptation.

Safety and fairness testing is covered in 10.11 Safety and adversarial testing.

Data quality in evaluation is mostly the quality of evaluation sets, which is covered in 3.4 Evaluation: building evaluation datasets and 11.4 Data quality in the data layer. One habit belongs here: when a case fails, check its label as well as the output, because a wrong reference answer makes a correct output look like a failure.

### 3.17 Choosing models [4]

From Layer 8, Foundation models. This needs the evaluation sets described in 3.4 Evaluation: building evaluation datasets.

Choosing models, and switching when better ones arrive, is the most important recurring decision in this layer.

- [4] Selection criteria:
  - quality on your task
  - cost
  - latency
  - context window
  - the inputs and outputs it supports
  - support for tool calling and structured output
  - rate limits
  - data handling and regions
  - license
  - how long the model will be supported
- [4] Testing on your own task. Run candidate models on your evaluation set, and compare quality, cost and latency together. Public rankings narrow the list, but your own tests decide.
- [4] A model for each job. Different steps in a product need different models. For example, you might use a small, fast model to classify or route requests, a strong model for hard reasoning, and an embedding model for search. Give each step the cheapest model that does it well enough.
- [4] Migrations. When you switch or upgrade a model, re-run your evaluations and revise your prompts, because prompts tuned for one model often do worse on another. Then roll out gradually and watch production metrics.
- [3] Staying swappable. Keep the choice of model in configuration, and avoid depending on one model's quirks. Keep your evaluation set ready so that switching is routine. Switching models at runtime is covered in 8.3 Fallbacks and retries and 8.6 Routing.
- [2] Leaderboards. Human-vote leaderboards such as LMArena rank models by which answers people prefer. Independent sites compare quality, price and speed, and task leaderboards cover areas such as coding and agents. Use them to build a shortlist. Why public benchmarks rarely predict your results is covered in 3.4 Evaluation: building evaluation datasets and 3.19 Public benchmarks.
- [2] Pace of progress. New frontier models arrive every few months, and the price of a given level of capability keeps falling. Expect today's limits to move, and re-test regularly. Avoid building elaborate workarounds for weaknesses the next model may fix.

Check: For one feature, compare three models on your evaluation set, including at least one cheaper tier. Pick one based on quality, cost per task and latency. Write down what would make you switch.

### 3.18 Evaluation in foundation models [3]

From Layer 8, Foundation models.

- [3] Choosing a judge model. When models grade outputs, they tend to favor answers from their own model family, longer answers and whichever option is shown first. Use a judge from a different family, or check it against human labels.

Using evaluations to choose models is covered in 3.17 Choosing models.

### 3.19 Public benchmarks [2]

From the Evaluation concern. You can read about public benchmarks at any point. They matter most when you choose models (see 3.17 Choosing models).

Public benchmarks are shared test sets for comparing models. Benchmarks for particular kinds of work are covered in each layer's sections: agents in 7.4 Evaluation in agents and orchestration, tool use in 5.13 Evaluation in tools and protocols, retrieval and memory in 6.3 Evaluation in context, memory and retrieval, and leaderboards in 3.17 Choosing models. This section covers reading benchmark results.

- [2] What the common ones measure. Widely reported benchmarks cover knowledge and reasoning (such as MMLU-Pro, GPQA and Humanity's Last Exam), math (such as AIME), coding (such as SWE-bench Verified and LiveCodeBench), instruction following (such as IFEval) and factual accuracy (such as SimpleQA). Know what a typical question in each looks like, so you can tell whether a score says anything about your task.
- [2] Reading a claim. Check how a score was produced: the prompt, the number of examples shown, how many attempts were allowed (pass@1 means one), the reasoning budget, and for agent benchmarks the harness around the model. Scores from different setups can't be compared, and a model's maker often reports its best setup.
- [2] Why benchmarks wear out. Benchmark questions leak into training data (see contamination in 3.20 Pre-training corpora). And once top models score near the maximum, the differences between them are noise. That's why new benchmarks keep replacing old ones.
- [2] Ratings from comparisons. Leaderboards such as LMArena turn many side-by-side votes into ratings with methods such as Elo and Bradley-Terry. The ratings reflect what voters prefer, which rewards style and length, and labs can test many private versions before release. You can use the same rating methods to rank many versions of your own system from pairwise judgments.
- [2] Running benchmarks yourself. Tools such as EleutherAI's lm-evaluation-harness, Stanford's HELM and Inspect run public benchmarks on any model. This helps when you compare open-weight or quantized models, since published scores may not match the version you run (see 13.16 Evaluation in compute).

Check: Take one recent model announcement. For each headline benchmark, explain what it measures, whether its setup matches the numbers reported for competitors, and what it predicts about your task. Then run your own evaluation set to find out.

### 3.20 Pre-training corpora [1, some topics 2]

From Layer 10, Data. You can read about pre-training corpora at any point. The material is light, and it explains model behavior you'll see along the way.

Pre-training corpora are the huge text collections that labs train foundation models on. You need to know enough about them to explain model behavior and to judge claims about models.

- [1] What goes in. Web crawls such as Common Crawl, filtered web datasets such as FineWeb, code, books, papers, math, text in many languages, licensed data, and a growing share of synthetic data.
- [1] How they're built. Labs crawl the web and extract the text. They detect the language and filter for quality. They remove duplicates, personal data, toxic text and benchmark questions. Finally they choose the mix of sources and tokenize.
- [2] Quality filtering and deduplication. Rules plus trained classifiers that score quality. These are worth learning properly, because you'll use the same techniques on your own data.
- [1] Data and scale. Models train on trillions of tokens. The Chinchilla result shows how to balance model size against the amount of data. Data quality now matters more than quantity.
- [2] Knowledge cutoff. A model knows nothing about events after its training data ends. It also knows less about recent events, niche topics and less-common languages. This is one of the main reasons to use retrieval (see 6.2 Retrieval-augmented generation).
- [2] Contamination. Questions from public benchmarks leak into training data, which inflates benchmark scores. This is why you need private evaluation sets.
- [2] Memorization. Models can repeat training text word for word, which creates copyright and privacy risk in their outputs.
- [2] Model cards. Reading the data section of a model card or technical report, and noticing what it leaves out.
- [1] Licensing debates. Opt-outs, robots.txt, copyright lawsuits and licensing deals. Know enough to answer a legal or procurement question about a model.
- [1] Continued pre-training. Training a model further on a large domain corpus, such as legal or medical text, and when a team would pay for that rather than use fine-tuning or retrieval.

Check: Explain why a model is weak on a topic, based on what its training data probably contained. Explain to a stakeholder why a public benchmark score doesn't predict results on your task.

### Key reading for Stage 3

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 3 covers evaluation methods, including AI judges, chapter 4 covers evaluating AI systems, chapter 8 covers dataset engineering and chapter 10 covers user feedback.
- Hamel Husain, "Your AI Product Needs Evals" (2024) and "Creating a LLM-as-a-Judge That Drives Business Results" (2024).
- Shankar et al., "Who Validates the Validators? Aligning LLM-Assisted Evaluation of LLM Outputs with Human Preferences" (2024), on criteria drift and checking judges against people.
- Zheng et al., "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena" (2023), on the biases of LLM judges.
- Eugene Yan, "Evaluating the Effectiveness of LLM-Evaluators (aka LLM-as-Judge)" (2024).
- Ribeiro et al., "Beyond Accuracy: Behavioral Testing of NLP Models with CheckList" (2020).
- Evan Miller, "Adding Error Bars to Evals: A Statistical Approach to Language Model Evaluations" (Anthropic, 2024).
- Liang et al., "Holistic Evaluation of Language Models" (Stanford, 2022), on measuring many criteria across many scenarios.
- Singh et al., "The Leaderboard Illusion" (2025), on how leaderboard rankings can mislead.
- Your model providers' guides to defining success criteria and building evaluations, and the documentation for one evaluation framework, such as Inspect or promptfoo.
- Hamel Husain's and Shreya Shankar's writing on evals and error analysis.
- Penedo et al., "The FineWeb Datasets" (2024), on how a pre-training corpus is built.
- Shumailov et al., "AI models collapse when trained on recursively generated data" (*Nature*, 2024).

## Stage 4. Product basics

With a model, prompts and an evaluation set, you can decide what to build and how people will use it. This stage covers whether AI fits a problem, designing for wrong answers, streaming, interfaces, setting expectations and recording what users do. The screens people use to steer agents come in Stage 7.

| Section | Level | Why this level | From |
|---|---|---|---|
| 4.1 Choosing what to build | 3 | AI fits some problems well and others badly | Layer 1 |
| 4.2 Designing for wrong answers | 4 | Every AI feature is sometimes wrong, and users can't always tell | Layer 1 |
| 4.3 Streaming and waiting | 3 | Users judge speed by what they see, not by total time | Layer 1 |
| 4.4 Feedback and measurement | 3 (some topics 4) | Only your product can record what users do with each result | Layer 1 |
| 4.5 Interaction patterns | 3 | A chat box is rarely the only or best interface | Layer 1 |
| 4.6 Setting expectations | 3 | Users who expect too much are let down, and those who expect too little never try | Layer 1 |
| 4.7 Governance and safety in product and UX | 2–3 | Laws increasingly require disclosure, user controls and care for vulnerable users | Layer 1 |
| 4.8 Security in product and UX | 3 | Model output shown in a browser can attack the user | Layer 1 |
| 4.9 Cost and latency in product and UX | 3 | Heavy users can cost many times the average | Layer 1 |
| 4.10 Evaluation in product and UX | 3 | Users judge the whole experience, not single outputs | Layer 1 |
| 4.11 Reliability in product and UX | 3 | The rest of the product must keep working when AI fails | Layer 1 |
| 4.12 Observability in product and UX | 3 | A user's complaint should lead straight to the trace | Layer 1 |

### About Layer 1: Product and UX

This layer is where people meet what you've built. Product managers and designers usually lead it, and you share it with them. AI features differ from other software in two ways: their output is sometimes wrong in ways users can't always see, and every use takes time and costs money. You know better than anyone else on the team how and when your model fails, so the parts of the design that depend on that are yours.

For every AI feature, this layer answers four questions:

1. Is AI the right tool here, and what counts as success? (4.1 Choosing what to build)
2. How do users work with it? (4.5 Interaction patterns, 7.7 Users in control of agents)
3. What do users see while it works, and when it's wrong? (4.3 Streaming and waiting, 4.2 Designing for wrong answers, 4.6 Setting expectations)
4. What do users do with the result, and what does that tell you? (4.4 Feedback and measurement)

Most concepts in this layer are level 3, and two things are level 4:
- designing every feature for the times the AI is wrong
- recording what users do with each result

Learn product strategy to level 2: you contribute to it, but product managers own it. Visual design and front-end engineering are disciplines of their own, so this curriculum covers only what changes when AI is involved.

### 4.1 Choosing what to build [3]

From Layer 1, Product and UX. It decides whether the rest is worth doing.

Product managers usually decide what to build. You bring what they can't: how well models handle the task, how they fail and what each use costs.

- [3] Where AI fits. AI fits tasks where a good-enough result is valuable, the result is quick to check or cheap to fix, and the work is slow or tedious for people. It fits poorly where mistakes are costly and hard to spot, or where a simple rule would do the job.
- [3] Doing the task or helping with it. Decide whether the AI does the task for the user (automation) or helps the user do it, with drafts and suggestions (assistance). Assistance tolerates more errors, because a person checks every result. Automation needs higher accuracy and a way to catch mistakes. Start with assistance where mistakes are costly.
- [3] Defining success. Before building, decide what success means for the user and the business, and how you'll measure it, such as tasks completed, time saved or suggestions accepted. Link it to your evaluation set (see 3.4 Evaluation: building evaluation datasets), so improvements you measure offline show up in the product.
- [3] Prototyping with real models. Judge AI features with real models and real data, not mockups, because quality can't be judged from a picture of the interface. A demo that works on a few hand-picked examples is far from a product that works for real users. Put a working prototype in front of real users early.
- [3] The cost of each use. Unlike most features, an AI feature costs money every time it's used. Check that the value of each use covers its cost at the volume you expect, including your heaviest users (see 1.7 Cost and latency in foundation models). This shapes pricing, such as charging by usage, credits or plan limits, which you'll decide with product and finance.
- [2] Designing for better models. Models improve every few months (see 3.17 Choosing models). Features that are barely possible today may soon be easy, and workarounds for today's weaknesses may become unnecessary. Design so that a better model makes the product better without a redesign.
- [2] Where lasting advantage comes from. Every competitor can use the same models. Lasting advantage comes from your data, from how deeply you fit into users' work, from distribution, and from the feedback loop that improves your product with use (see 11.2 Product feedback).

Check: For one proposed feature, write a one-page case covering the user's problem, whether AI does the task or assists, how you'll measure success, what happens when the AI is wrong, and the cost of each use. Build a working prototype with a real model and test it with five real users.

### 4.2 Designing for wrong answers [4]

From Layer 1, Product and UX. It shapes most other decisions in this layer.

Every AI feature is sometimes wrong, and users can't always tell. This is the biggest difference between designing AI features and other software, and it shapes most of the decisions in this layer.

- [4] Plan for mistakes. For each feature, decide how users will notice a mistake, how cheaply they can fix it, and what harm a mistake that nobody notices can do. Make mistakes cheap before trying to make them rare.
- [4] Easy to check. Put sources and citations next to the claims they support, link to the exact passage, highlight what changed, and show the inputs the AI used. Checking a result should take much less time than doing the task. Producing citations is covered in 6.7 Answering from retrieved context.
- [4] Easy to fix. Let users edit results in place, undo what the AI did, and tell it what's wrong. Drafts and suggestions that users accept are safer than changes applied directly.
- [4] Trust that matches reality. Users tend to trust AI too much, accepting wrong answers without checking, or too little, ignoring useful help after one mistake. Aim for trust that matches how reliable the feature actually is: show sources and limits, and make checking easy.
- [3] Showing uncertainty. A model's own statement of how sure it is means little (see 1.2 Capabilities and limits). Show uncertainty through behavior instead: say "I couldn't find this in your documents", ask a clarifying question, offer alternatives or flag items for review.
- [3] When the AI can't help. When it declines, times out, fails or has nothing useful to say, show a clear message and a way forward: try again, a path that doesn't use AI, or a person. Never show a blank screen or a raw error, and never trap users with a bot that can't help them. What the system does when providers fail is covered in 8.3 Fallbacks and retries.
- [3] Variety or consistency. The same request can give different answers. Decide what each feature needs: several options for creative work, stable answers for factual questions. Design for it, for example with a regenerate button, or with caching for answers that shouldn't change.

Check: List the five most likely mistakes of one AI feature. For each, show how a user would notice it and how many steps it takes to fix. Redesign the worst case, and test whether users catch a planted mistake before and after the change.

### 4.3 Streaming and waiting [3]

From Layer 1, Product and UX.

AI features are slow by software standards. Users judge speed by how soon something happens and whether they can see progress, not only by total time. How streaming works at the API level is covered in 1.6 Calling models through an API.

- [3] Streaming text. Show text as it arrives, so users start reading within a second or two. Render partial Markdown, code and tables without flicker or jumping layouts, and handle a stream that breaks partway.
- [3] Rendering results. Render Markdown, code with a copy button, tables, math and links cleanly, and render them safely (see 4.8 Security in product and UX).
- [3] Something happens at once. Acknowledge every request immediately, then show progress, then the result. A blank screen for five seconds feels broken, even if the answer is good.
- [3] Showing progress. For multi-step work, show what's happening, such as "searching your documents" or "running tests", and let users see the plan. Don't present the model's raw reasoning as a reliable explanation of how it reached its answer (see 1.2 Capabilities and limits).
- [3] Stop, edit and retry. Let users stop a response, edit their message and try again, or regenerate. Stopping should cancel the model call, not just hide the output.
- [3] Long tasks. For work that takes minutes, move it out of the conversation: show its status, let users leave, and notify them when it's done (see 7.5 State and long-running tasks).
- [2] Streaming into components. Structured output can stream into interface components, such as filling in a form or a table field by field as the data arrives.

Check: Measure time to first visible text and total time for one feature, as users experience them. Add streaming and progress updates, and have users rate how fast it feels before and after. Show that the stop button cancels the model call.

### 4.4 Feedback and measurement [3, some topics 4]

From Layer 1, Product and UX. Record what users do from the first release.

This layer collects the feedback that flows back into Data. Turning feedback into datasets and evaluation cases is covered in 11.2 Product feedback. This section covers collecting it and measuring what AI features do for users.

- [4] Recording what users do. Record what users do with every AI result: accept, edit, copy, regenerate, undo, abandon or ask for a person. Link each event to the result's trace, and to the prompt and model versions. Only the product can capture this, and if you don't record it at the time, it's gone.
- [3] Feedback controls. Offer thumbs up and down, ratings or "report a problem" where users judge a result, with an optional reason from a short list plus free text. Few users give feedback this way, and those who do aren't typical, so don't rely on it alone.
- [3] Edits as feedback. When users edit an AI draft, save both versions. The difference shows exactly what was wrong.
- [3] Product metrics. Measure what AI features do for users: how many use them, how often they come back, how often tasks succeed, how many suggestions are accepted, time saved and how often users escalate to a person. Also track guardrail metrics that shouldn't get worse, such as complaints, reports and support tickets.
- [3] Experiments. Test prompt, model and design changes on real users with controlled experiments (see 8.6 Routing). AI outputs vary, so experiments can need more users or more time to show a real difference. Watch for novelty: use of a new AI feature often spikes, then falls.
- [3] Watching real use. Watch people use the feature, interview them, and read real conversations where users have agreed to it. Numbers show what happens. Watching shows why.
- [3] Offline and online agreement. Check that improvements on your evaluation set show up in product metrics. If they don't, your evaluation set doesn't reflect real use (see 3.4 Evaluation: building evaluation datasets).

Check: Instrument one feature so that every result records accepts, edits, regenerations and abandons, linked to its trace. Build a weekly report of acceptance and task success. Run one experiment on a prompt or model change, and report whether your offline evaluation predicted the result.

### 4.5 Interaction patterns [3]

From Layer 1, Product and UX.

How users work with AI shapes what they can do with it. A chat box is the default, but often not the best choice.

- [3] Chat. Conversation is flexible and familiar, and suits open-ended requests. But it leaves users to work out what to ask, and it's slow for structured or repeated tasks.
- [3] AI inside existing software. Autocomplete, a button that drafts or summarizes, suggestions in context and sensible defaults meet users where they already work, without new habits. They're often more useful than a separate chat window.
- [3] Assistants beside the work. A side panel that sees what the user is working on, such as the open document or the selected code, and can act on it, while the user stays in charge. In some products, the user and the AI edit the same document or canvas side by side.
- [3] Handing tasks to agents. The user assigns a task, leaves, and comes back to a result, such as a report or a code change. The interface is about assigning, monitoring, reviewing and correcting, not chatting (see 7.7 Users in control of agents).
- [3] Beyond free text. Forms, buttons, pickers, templates and file uploads often beat typing: they show users what's possible and give the model clean input. Show results in the form the task needs, such as a table, a diff or a draft in the editor, not only as paragraphs of text.
- [2] Generated interfaces. The model can choose interface components to show, such as a chart, a form or a booking card, instead of replying in text (see 5.2 Function calling and 5.9 Model Context Protocol (MCP)). Limit it to components you've built and tested.
- [2] Voice. Voice needs fast replies, handling of interruptions and turn-taking, and short answers written to be spoken (see 1.6 Calling models through an API and 6.20 Multimodal models). Learn this to level 3 if you build voice products.
- [2] Conversation features. Users expect chat products to offer a history they can search, the option to edit an earlier message and try again, and ways to copy, share and delete conversations.

Check: Take one feature that is currently a chat box and build another version as inline suggestions, a button or a form. With real users, compare how often each is used and how often tasks succeed.

### 4.6 Setting expectations [3]

From Layer 1, Product and UX.

Users who expect too much are let down, and users who expect too little never try. Both need an accurate picture of what the AI can do.

- [3] What it can and can't do. At first use, say what the feature is for, what it does well and where it falls short, in concrete terms, with example requests.
- [3] Starting points. An empty text box is hard to start from. Offer example requests, templates and suggested next steps based on what the user is doing.
- [3] What the AI can see and remember. Show what the AI has access to and what it used for each answer: which documents, which page, which memories. Let users see, edit and delete what the product remembers about them, and turn memory off (see 6.9 Long-term memory). Otherwise users guess wrong in both directions.
- [3] Labeling AI. Make clear which content the AI produced, and when users are dealing with an AI rather than a person. Some laws require this (see 4.7 Governance and safety in product and UX).
- [2] Voice and personality. The product's tone, such as formal, friendly or brief, is a product decision. Set it in the system prompt and keep it consistent across features. Don't pretend to be human, and don't flatter users (see 1.2 Capabilities and limits).

Check: Watch five new users try a feature for the first time without help. Note what they expect it to do and where they get stuck. Change the introduction and starting points, and repeat with five more users.

### 4.7 Governance and safety in product and UX [2–3]

From Layer 1, Product and UX.

- [3] Data controls. Let users see and delete their conversations, choose whether their data is used to improve the product, and export it. Explain these choices in plain words where users make them. Permission to use data is covered in 11.2 Product feedback.
- [3] Sensitive situations. Decide how the product responds when conversations turn to self-harm or to serious medical, legal or financial risk, such as pointing to professional help or crisis lines, and test those responses. Products that children may use need extra care, and laws increasingly require it.
- [2] Transparency laws. The EU AI Act requires telling people when they're interacting with an AI system, and marking some AI-generated content. Other laws set similar rules. Know which apply to your product, and bring in legal.
- [2] Respecting users. Don't design AI features to maximize engagement at users' expense, for example through flattery, false urgency or encouraging emotional dependence.
- [2] Accessibility. Streaming text, voice and generated content must work with screen readers and keyboards, and accessibility laws apply to AI features like any other. Learn this to level 3 if you build the front end.

### 4.8 Security in product and UX [3]

From Layer 1, Product and UX.

- [3] Rendering output safely. Model output is untrusted. Rendered as HTML or Markdown, it can run scripts, or load images and links that send data to an attacker (see 6.10 Security in context, memory and retrieval). Sanitize it, restrict which links and images load, and never run generated code in the user's browser outside a sandbox.
- [3] Abuse through the product. Some users will try jailbreaks, or try to use your product as free general-purpose AI. Limit message length, file size and use per user (see 8.4 Rate limits and quotas), and watch for abuse.
- [3] Untrusted uploads. Files users upload can carry injected instructions or malware. Treat them like any untrusted input (see 5.10 Security in tools and protocols).
- [2] Sharing. Shared conversation links can expose private data, and public links have ended up in search results. Make sharing deliberate, show what will be shared, and let users revoke links.

### 4.9 Cost and latency in product and UX [3]

From Layer 1, Product and UX.

- [3] Speed by interaction. Users expect different speeds from different interactions: autocomplete within a fraction of a second, a chat reply that starts within a second or two, a background task within minutes. Choose models and designs to match (see 1.5 Inference performance).
- [3] Cost per user. Heavy users can cost many times the average user. Track cost per active user and per plan, and use plan limits, rate limits and cheaper models for free tiers (see 8.4 Rate limits and quotas and 8.6 Routing).
- [3] Limits users understand. When users hit a limit, tell them what happened, when they can continue and what their options are.
- [3] Calls nobody asked for. AI that runs automatically, such as on every keystroke or page view, multiplies cost. Wait for a pause in typing, run on request where the value is low, and reuse results that haven't changed.
- [2] Precomputing. Generating results ahead of time, such as summarizing a document when it's saved rather than when it's opened, hides latency, but you pay for results nobody reads.
- [2] Letting users choose. Some products let users pick a model or a reasoning effort, trading speed and cost for quality. Most users shouldn't have to choose, so route automatically (see 8.6 Routing) and offer the choice to those who want it.

### 4.10 Evaluation in product and UX [3]

From Layer 1, Product and UX.

- [3] The whole experience. Evaluation sets test model outputs. Also test the experience as users get it: the rendered answer, citations, error messages, slow networks and broken streams, on the devices your users have. Automate tests of the main flows.
- [3] Using it yourself. Use the product every day, and get colleagues to use it too. People inside the company find rough edges before customers do.

Product metrics and experiments are covered in 4.4 Feedback and measurement.

### 4.11 Reliability in product and UX [3]

From Layer 1, Product and UX.

- [3] The product works without AI. When the AI is down or slow, the rest of the product should keep working, and AI features should fail clearly (see 8.3 Fallbacks and retries).
- [3] Never lose the user's work. Keep the user's message and any partial answer when a stream breaks, reconnect where possible, and save drafts. A failure should never cost users what they typed.

### 4.12 Observability in product and UX [3]

From Layer 1, Product and UX.

- [3] From complaint to trace. A user's report, a support ticket or a thumbs-down should lead straight to the exact trace, with its prompt and model versions (see 11.2 Product feedback).
- [2] What users experienced. Measure speed and errors in the client, as users saw them: time to first visible text, broken streams and rendering failures. Server metrics miss these.

Data quality has little of its own in this layer. The quality of the feedback you collect is covered in 11.2 Product feedback.

### Key reading for Stage 4

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 10 covers AI application architecture and user feedback.
- Amershi et al., "Guidelines for Human-AI Interaction" (Microsoft, 2019).
- Google PAIR, *People + AI Guidebook*.
- Apple, *Human Interface Guidelines*, the section on machine learning.
- Emily Campbell, *The Shape of AI* (shapeof.ai), a library of interface patterns for AI features.
- Nielsen Norman Group's research on AI user experience.
- Parasuraman and Riley, "Humans and Automation: Use, Misuse, Disuse, Abuse" (1997), on trusting automation too much and too little.
- OWASP Top 10 for LLM Applications (2025), especially "Improper Output Handling".
- The documentation for a toolkit for AI interfaces, such as the Vercel AI SDK.

## Stage 5. Tools

Tools let a model act, so this is where a wrong answer can become a wrong action. The stage starts with why the model can't be trusted to protect anything. It then covers function calling, designing tools, connecting to APIs, permissions, prompt injection, MCP and sandboxes.

| Section | Level | Why this level | From |
|---|---|---|---|
| 5.1 Security foundations | 3 (some topics 4) | The model can't be trusted to protect anything | Security |
| 5.2 Function calling | 3 | The mechanism under every tool, protocol and agent | Layer 3 |
| 5.3 Designing tools | 4 | The model knows your tools only through their names, descriptions and results | Layer 3 |
| 5.4 Connecting to APIs | 3 | Most tools are thin wrappers around an API | Layer 3 |
| 5.5 Actions and side effects | 3 (some topics 4) | A wrong action often can't be taken back | Layer 3 |
| 5.6 Access and identity | 3 (some topics 4) | The model must never reach more than the user could | Security |
| 5.7 Permissions | 3 (some topics 4) | An agent can be tricked into misusing anything it can reach | Layer 3 |
| 5.8 Prompt injection | 3 (some topics 4) | Anyone who can get text in front of the model can try to steer it | Security |
| 5.9 Model Context Protocol (MCP) | 3 (internals 2) | The standard way to share tools across AI applications | Layer 3 |
| 5.10 Security in tools and protocols | 3 | Tools turn injected instructions into actions | Layer 3 |
| 5.11 Supply chain | 3 | Models, tools and packages from others are ways in | Security |
| 5.12 Code execution and sandboxes | 3 | Code the model writes is untrusted code | Layer 3 |
| 5.13 Evaluation in tools and protocols | 3 | You must check what agents did, not just what they said | Layer 3 |
| 5.14 Reliability in tools and protocols | 3 | Every tool call depends on another system that can fail | Layer 3 |
| 5.15 Observability in tools and protocols | 3 | You can't debug or audit actions you didn't record | Layer 3 |
| 5.16 Cost and latency in tools and protocols | 3 | Each tool call adds a model round trip | Layer 3 |
| 5.17 Governance and safety in tools and protocols | 2–3 | When an agent acts, your company acts | Layer 3 |
| 5.18 Browsers and computer use | 2 | Slow and fragile, so only for systems with no API | Layer 3 |

### About Security

A model can't reliably tell your instructions from text an attacker wrote, and no one yet knows how to make it. Anyone who can get text in front of the model, by typing it or by planting it in a web page, an email or a document the model will read, can try to steer it. The rest of this curriculum tries to get the most out of the model. Security assumes the model may be working for someone else, and limits what it can do, reach and send, with controls enforced in ordinary code.

For every AI feature, security answers four questions:

1. Who might attack, and where can their text get in? (5.1 Security foundations, 5.8 Prompt injection, 10.3 Jailbreaks and abuse)
2. What could a hijacked model do or reach, and on whose authority? (5.6 Access and identity, 10.2 Secrets)
3. What could leave the system, and through which channel? (10.1 Preventing data theft)
4. What are you trusting that you didn't build? (5.11 Supply chain)

10.4 Security testing and 10.5 Responding to attacks check the answers and deal with what gets through.

Security teams know attacks and defenses in general, but only you know what your models can read, reach and do. So the AI-specific design is yours. Most concepts in this concern are level 3, and three things are level 4:
- designing each feature so that a hijacked model can do little harm
- enforcing the user's own permissions, in code, on everything the model can reach
- mapping and closing every way data can leave

Know attack techniques well enough to recognize their families, and leave inventing new ones to researchers. Classic application and network security are disciplines of their own, so work with your security team on them. Attacks on models during training, such as poisoning pre-training data, need only awareness, because labs and providers defend against them.

The layers' own sections cover security as it applies to each layer, such as tool poisoning in Layer 3 and permission-aware retrieval in Layer 4. The Security sections cover what they share, and 14.3 Security at each layer maps where the rest is covered.

### 5.1 Security foundations [3, some topics 4]

From the Security concern. Focus on why the model can't be a security boundary.

These ideas come up in every other section, so learn them first.

- [4] The model isn't a security boundary. Models can't reliably tell your instructions from text an attacker wrote (see 1.10 Security in foundation models), and no known method fixes this completely. So never rely on the model, or on instructions in its prompt, to protect anything. Treat it like an untrusted user: enforce every limit in code, outside the model, and design each feature so that a model under an attacker's control can do little harm.
- [3] Who attacks. Attackers include users of your product, anyone who can put text where your system will read it, such as in a web page, an email, a shared document or a product review, the publishers of tools, models and packages you use, and insiders. Many of them never use your product directly.
- [3] Threat modeling. For each feature, write down what an attacker could want, such as your data, your users' data, actions in their name or your money, how they could get in, and what stops them. Catalogs such as the OWASP Top 10 for LLM Applications and MITRE ATLAS list common attacks to check against. Update the threat model whenever a feature gets a new tool, data source or kind of user.
- [3] Defense in depth. No single defense stops every attack, so combine several: limit what the model can do, check what goes in and what comes out, require approval for what can't be undone, and watch for attacks. Then an attack has to get past all of them to cause harm.
- [3] Ordinary security still applies. An AI feature is still a web application, and many real incidents come from ordinary mistakes, such as exposed keys, missing authorization checks, public storage buckets and unpatched dependencies. Work with your security team on the basics. This curriculum covers what AI adds.

Check: Write a one-page threat model for one AI feature. List every way outside text reaches the model, everything the model can do and reach, and the worst thing an attacker could achieve through each input. For the worst case, name the control in code that stops it.

### About Layer 3: Tools and protocols

A model on its own can only produce text. Tools let it act: look things up, change records, send messages, run code and operate a browser. On every tool call, the model only asks. Your code decides whether to run the call, runs it, and sends back the result. So this layer is where a wrong answer can become a wrong action, such as an email sent, a record deleted or money spent.

For every tool you give an agent, answer four questions:

1. How does the model know when and how to use it? (5.2 Function calling, 5.3 Designing tools)
2. How does it reach the system behind it? (5.4 Connecting to APIs, 5.9 Model Context Protocol (MCP), 7.15 Agent-to-agent protocols)
3. What can it change, and on whose behalf? (5.5 Actions and side effects, 5.7 Permissions)
4. Where does it run, and what can it reach from there? (5.12 Code execution and sandboxes, 5.18 Browsers and computer use)

Like Layers 4 and 5, this layer is mostly yours. Providers supply the function-calling mechanism and some built-in tools, but you define the tools, connect them to your systems and decide what agents may do. So most concepts in this layer are level 3, and three are level 4:
- designing tools
- deciding which actions need a person's approval
- deciding what each agent may do, and on whose behalf

Learn protocol internals, such as MCP's message format, to level 2, because SDKs handle them. Agent-to-agent protocols are level 2, because most products still run all their agents in one system. Browsers and computer use are level 2 unless you build agents that operate websites or desktops.

### 5.2 Function calling [3]

From Layer 3, Tools and protocols. Build a tool loop by hand before you use a framework.

Function calling, also called tool use, is how a model asks your code to do something. How reliably models choose tools and fill in arguments is covered in 1.2 Capabilities and limits. This section covers the mechanism.

- [3] The loop. You send tool definitions with the request. The model replies with a tool call, made of a tool name and arguments, instead of or alongside text. Your code runs the tool and sends the result back in a tool message, and the model continues. This repeats until the model answers without calling a tool. The model never runs anything itself. Planning across many steps is covered in 7.3 Agent loops and planning.
- [3] Tool definitions. Each tool has a name, a description and an input schema written in JSON Schema. Most SDKs can generate the schema from a typed function or data model. Definitions are sent on every call and count as input tokens.
- [3] Tool choice. You can let the model decide whether to call a tool, require it to call one, force a specific tool, or turn tools off. Forcing a specific tool is also a common way to get structured output.
- [3] Parallel calls. A model can request several independent calls in one turn. Run them at the same time and return every result, each matched to its call ID. Turn parallel calls off when order matters.
- [3] Checking arguments. Arguments can break the schema, or fit it and still be wrong, such as a valid date in the wrong year. Use strict schema enforcement where the provider offers it (see 13.7 Structured generation), and validate in your code before running anything. Return validation errors to the model so it can fix the call.
- [3] Returning results. Return results as text or structured data, and as images or documents where the API supports them. When a tool fails, return the error as the result, marked as an error, instead of ending the loop. A clear error lets the model try again or try another way.
- [3] Keeping the history valid. Every tool call must be followed by its result, matched by ID, and APIs reject conversations that break this rule. Reasoning models may think between tool calls, and some APIs need that reasoning passed back unchanged with the results. Watch for both when you trim history (see 2.3 Conversation history).
- [3] Built-in tools. Providers also offer tools that run on their side, such as web search, code execution and file search. They save work, but you control less of how they behave, your data passes through the provider, and they tie you to it (see 1.1 Working with a model).
- [2] Tool runners. Provider SDKs and agent frameworks can run the loop for you. Build it by hand once, so you know what they're doing when something goes wrong.

Check: Build a tool loop by hand, without a framework, with three tools, one of which fails at random. Show parallel calls, a validation error that the model recovers from, and a trace of every call and result.

### 5.3 Designing tools [4]

From Layer 3, Tools and protocols. Most tool problems are design problems.

A model knows a tool only through its name, description, parameters and results. Tool design is to agents what interface design is to people, and it can matter as much as the system prompt.

- [4] Descriptions are prompts. Write each description like instructions for a new colleague: what the tool does, when to use it and when not to, what each parameter means, with formats and examples, and what it returns. Small wording changes can shift behavior as much as prompt changes do.
- [4] Tools for tasks, not endpoints. Don't expose every API endpoint as a tool. Build tools around what the agent needs to get done. For example, one tool that finds a free slot and books a meeting works better than separate tools to list calendars, check availability and create events. Fewer, task-level tools mean fewer steps and fewer wrong choices.
- [4] Results the model can use. Return only what the next step needs, not the whole API response. Prefer readable names to opaque IDs, but include the IDs that later calls need. Cap long results, say when you've cut them and how to get more, and consider a parameter that asks for a short or a detailed result.
- [4] Improving tools from transcripts. Run realistic tasks, read the transcripts, and note where the model picks the wrong tool, fills in arguments badly or wastes calls. Revise the tools and measure again on your evaluation set. A model can help: give it the failing transcripts and ask it to suggest better descriptions.
- [3] Helpful errors. Say what went wrong and what to do instead. "No customer named Jon Smith. Did you mean John Smith (ID 4521)?" is far more useful to a model than "404 Not Found".
- [3] Names and parameters. Give tools distinct names, and group related ones with a shared prefix, such as the service name. Avoid tools that overlap. Use few parameters, with clear names, explicit types, fixed lists of allowed values and sensible defaults. Make mistakes hard: if relative file paths cause errors, require full paths.
- [3] Reading apart from changing. Keep tools that only read separate from tools that change things. Then you can give them different permissions and approval rules (see 5.5 Actions and side effects, and 5.7 Permissions).

Check: Take one agent's tool set and redesign it: merge low-level endpoints into task-level tools, and rewrite descriptions, errors and results. Show on an evaluation set of tasks that the success rate rises and the tokens per task fall.

### 5.4 Connecting to APIs [3]

From Layer 3, Tools and protocols.

Most tools are thin wrappers around an API: your own backend, a database or another company's service. This section covers that wiring.

- [3] Wrapping an API. The tool translates between what the model sees and what the API needs. It handles pagination, rate limits, retries and error codes, so the model doesn't have to.
- [3] Credentials. Tools need API keys, service accounts or OAuth tokens. Keep them in a secrets manager and add them inside the tool, never in the prompt or anywhere else in the context (see 5.7 Permissions). Secrets management in general is covered in 8.2 API keys and access.
- [3] OAuth. OAuth lets a user grant your app limited access to their account on another service, such as their calendar or code repository, without sharing their password. Learn the authorization flow, scopes, access and refresh tokens, how to store and refresh tokens, and what happens when a user revokes access. Agents that act for users need it.
- [3] Databases. When a tool queries a database, use read-only credentials where possible, limit rows and query time, and expose views rather than raw tables. Make changes through narrow tools, not free-form SQL. Querying databases for retrieval is covered in 6.4 Preparing documents.
- [2] Generating tools from API descriptions. OpenAPI specifications describe REST APIs, and tools can be generated from them automatically. That's a quick start, but the result is usually one tool per endpoint, which is too low-level for agents (see 5.3 Designing tools).
- [2] Integration platforms. Services such as Composio, Arcade, Zapier and Pipedream offer ready-made tools for many apps, with OAuth handled for you. They save a lot of work, but their tools may suit your agent poorly, and your users' data and tokens pass through another company.

Check: Connect an agent to a third-party service with OAuth, so each user's agent sees only that user's data. Show that tokens never appear in prompts or logs, and that revoking access stops the agent cleanly.

### 5.5 Actions and side effects [3, some topics 4]

From Layer 3, Tools and protocols. Learn this before any agent changes anything real.

A wrong answer can be corrected. A wrong action, such as a sent email, a deleted record or a payment, often can't. This section covers tools that change things.

- [4] Which actions need approval. Sort every tool by what it can change: nothing; something you can undo; or something you can't undo or that reaches outside your system, such as sending, paying, deleting or posting. Require a person's approval for the last group, show them exactly what will happen, and decide who may approve. Designing handoffs between agents and people is covered in 7.6 Handoffs to people, and approval screens in 7.7 Users in control of agents.
- [3] Safe to repeat. Networks fail and agents retry, so a tool can run twice. Make changes safe to repeat (idempotent): pass an idempotency key that the API uses to ignore duplicates, check whether the action already happened, or set values rather than add to them. This is what makes retries safe (see 8.3 Fallbacks and retries).
- [3] Previews and dry runs. Let risky tools show what they would do before they do it, such as a draft email, a diff or a list of rows to delete. Previews make approvals meaningful and testing safer.
- [3] Undo. Prefer actions that can be reversed: drafts, soft deletes and staging environments. When a multi-step action fails partway, know how to undo the steps already done, or report exactly what changed.
- [3] Limits. Cap what one action or one run can do, such as the number of emails sent, rows changed or money spent. Runaway limits on model calls are covered in 8.5 Spending caps and cost control. These limits apply to actions.
- [3] Long-running tools. Give every tool a timeout. For work that takes minutes, have the tool start a job and return its ID, so the agent can check back later instead of waiting.

Check: For one agent, list every tool with what it changes, whether that can be undone, and whether it needs approval. Make one tool safe to call twice, and add an approval step with a preview to one irreversible action. Show both working in a test.

### 5.6 Access and identity [3, some topics 4]

From the Security concern. Get the user's permissions right before anything else.

This section covers who may use the system, and what the model may reach on their behalf. Permissions for agents and their tools are covered in 5.7 Permissions, permission-aware retrieval in 6.10 Security in context, memory and retrieval, and provider API keys in 8.2 API keys and access.

- [4] The user's permissions, everywhere. The model should never reach anything the user who asked couldn't reach themselves. Check the requesting user's rights at every point where data or actions are reached: search, tools, memories, caches, files and datasets. Check in code, at the point of access, never by asking the model.
- [3] Every caller identified. Authenticate users, services and agents, and give each its own identity, so every model call, tool call and action traces back to whoever caused it. Never leave a model endpoint, MCP server or internal tool reachable without authentication (see 13.10 Security in inference and serving).
- [3] Customers kept apart. In a product that serves many customers, one missing filter can show one customer's data to another. Check isolation everywhere data is stored: indexes and memories (see 6.10 Security in context, memory and retrieval), response and prompt caches (see 8.7 Caching and 13.10 Security in inference and serving), sandboxes (see 5.12 Code execution and sandboxes), traces (see 9.16 Security in observability), and datasets and fine-tuned models (see 11.6 Security in the data layer). Test it by trying to reach another customer's data on purpose.
- [3] Permissions that change. People leave teams, documents get restricted and access gets revoked. Make sure each change reaches every copy the AI system keeps, such as search indexes, memories and caches, and any long-running task still acting for that user.
- [2] Who can change the AI's behavior. Prompts, tool definitions, model settings and routing rules decide what the product does. Limit who can change them in production, and review changes like code (see 2.1 Prompts).

Check: For one feature, list every place the model reads data or takes actions, and show where the user's permissions are checked in code for each. Then, as a test user, try to reach another user's and another customer's data through the feature, including with injected instructions, and show that every attempt fails.

### 5.7 Permissions [3, some topics 4]

From Layer 3, Tools and protocols. Learn this before any agent changes anything real.

An agent can misuse anything it can reach, because it can be wrong or be tricked by injected instructions. Permissions decide what it can reach. Authentication and authorization in general are covered in 5.6 Access and identity. This section covers them for agents and their tools.

- [4] Least privilege. Give each agent only the tools and access its task needs: read-only where reading is enough, one project rather than the whole account, a narrow tool rather than a general one. Assume the agent will eventually misuse whatever it has.
- [4] Whose authority. Decide for each tool whether the agent acts with the user's own access, delegated through OAuth, or with its own identity and limited permissions. The agent should never be able to do more than the user who asked. Record which identity each action used.
- [3] Enforced in code. Check permissions in the tool or in the API behind it, never in the prompt. An instruction such as "don't delete records" is a request to the model, not a control.
- [3] Credentials out of the context. The model should never see tokens, passwords or keys. Tools add credentials when they call an API. Anything in the context can leak through output or logs, or be stolen by injected instructions.
- [3] Tool sets per agent. Different agents, and different users of the same agent, get different tools. Keep an explicit list of the tools each may use, and review changes to it like code.
- [3] The confused deputy. An agent with broad access of its own can be talked into using it for someone who lacks that access. Checking the requesting user's own permissions on every action prevents this.
- [2] Agent identities. Agents are increasingly given their own identities and credentials, separate from both users and ordinary services, so their actions can be limited, audited and revoked on their own. Standards for this are still settling.

Check: For one agent, write down each tool with the identity it acts as and the narrowest access that works, and remove anything broader. Then show that an injected instruction to act outside the user's permissions is blocked by the tool layer, not merely refused by the model.

### 5.8 Prompt injection [3, some topics 4]

From the Security concern.

Prompt injection is text that makes a model follow an attacker's instructions instead of yours. It is direct when the attacker types it, and indirect when it arrives inside content the model reads, such as a web page, a document or a tool result. How models fall for it is covered in 1.10 Security in foundation models, injection through retrieved content in 6.10 Security in context, memory and retrieval, and injection through tools in 5.10 Security in tools and protocols. This section covers the defenses they share.

- [4] Limiting the damage first. Filters and instructions reduce injection but don't stop it, so the defenses that matter most limit what a hijacked model can do. Give it only the tools and data the task needs, keep agents that read untrusted content away from powerful tools (see 7.13 Security in agents and orchestration), require a person's approval for actions that can't be undone, and close the channels data could leave through (see 10.1 Preventing data theft). Where the stakes are high, use agent designs that resist injection (see 5.10 Security in tools and protocols).
- [3] Every entry point. List every way outside text reaches each model call: user messages, uploaded files, retrieved documents, web pages, emails, tool results, memories and other agents' output. Instructions can hide where people won't see them, such as in white text, HTML comments, image text, file metadata or invisible Unicode characters.
- [3] Marking untrusted content. Put untrusted content inside clear markers, label where it came from, and tell the model to treat it as data. Some teams also transform it, such as by inserting a special character between its words, so the model can tell it apart from instructions (spotlighting). This makes injection harder, not impossible.
- [3] Detecting injection. Classifiers can flag text that looks like an injection before it reaches the model. Cloud providers offer some, and open models such as Llama Prompt Guard run on your own servers. They catch common attacks, miss new ones and flag some harmless text, so treat a flag as a signal, not a guarantee.
- [3] Checking actions against the request. Before an agent acts, check the action against what the user actually asked for and against fixed rules, such as which domains email may go to or which records a tool may change. An action the user's request doesn't explain is a warning sign (see goal drift in 7.13 Security in agents and orchestration).
- [2] Model-level defenses. Providers train models to resist injection and to give text in documents and tool results less authority than the system prompt (see 1.2 Capabilities and limits), and some publish how often attacks succeed. This raises the bar, but it doesn't replace the other defenses.

Check: Plant injected instructions in a document, a web page and a tool result that your agent reads. Have one ask for an action the user didn't request, and another ask for a different user's data. Show that each attempt is stopped by a control in code, such as a permission check or an approval, even when the model follows the instruction.

### 5.9 Model Context Protocol (MCP) [3, internals 2]

From Layer 3, Tools and protocols. Use existing servers first, then build one.

MCP is an open standard for connecting AI applications to tools and data. A service describes its tools once, in an MCP server, and any application that supports MCP can use them. MCP started at Anthropic in late 2024, and most major model providers and agent tools now support it.

- [3] Hosts, clients and servers. The host is the AI application, such as a chat app, a code editor or your own agent. It opens one client connection to each server. Servers expose capabilities to the host.
- [3] What servers offer. Servers offer three kinds of capability:
  - tools: actions the model can call, by far the most used
  - resources: data the application can read and add to the context
  - prompts: ready-made templates a user can pick

  Clients can offer features back to servers, such as asking the user for input (elicitation) or asking the host's model to generate text (sampling). Support for these varies by client.
- [3] Local and remote servers. A local server runs on the user's machine as a separate process and talks to the host over standard input and output (stdio). A remote server runs as a web service over HTTP. Remote servers are how companies offer MCP access to their products.
- [3] Authorization. Remote servers use OAuth, so users can grant a client access to their own accounts (see 5.4 Connecting to APIs). Build and debug the flow once, end to end.
- [3] Using existing servers. Many companies publish official servers for their products, and registries list thousands more. Before connecting one, check who maintains it, what access it asks for and what it does with your data (see 5.10 Security in tools and protocols).
- [3] Building a server. Use an official SDK, available for Python, TypeScript and other languages, and test the server with the MCP Inspector. The protocol is the easy part. Most of the work is tool design (see 5.3 Designing tools).
- [3] When to use MCP. MCP helps when tools must work across many applications, or when you want to use tools that others have built. For tools that live inside your own application, plain function calling is simpler. Some provider APIs can call remote MCP servers directly.
- [2] Protocol details. Messages use JSON-RPC. A session starts with a handshake in which client and server agree on a protocol version and on the features each supports. Servers can tell clients when their list of tools changes. The specification is versioned by date and still changing, so check which versions your clients and servers support.
- [2] Many servers, many tools. Every connected server adds tool definitions to the context, and connecting many servers can crowd the window and confuse the model. Load tools only when needed (see 2.2 Context engineering), or let the agent write code that calls them (see 5.12 Code execution and sandboxes).
- [1] Interactive results. Some hosts let tools return small interactive interfaces, such as a form or a chart, instead of only text. How they're shown is covered in 4.5 Interaction patterns.

Check: Build an MCP server with two task-level tools for one of your systems, test it with the MCP Inspector, and use it from two different MCP hosts. Deploy it as a remote server with OAuth, and show that each user sees only their own data.

### 5.10 Security in tools and protocols [3]

From Layer 3, Tools and protocols.

Tools turn prompt injection from a risk of wrong answers into a risk of wrong actions. How models fall for injected instructions is covered in 1.10 Security in foundation models, and injection through retrieved content in 6.10 Security in context, memory and retrieval.

- [3] Every tool result is untrusted. Emails, web pages, files, API responses and other agents' replies can all contain instructions aimed at the model. Limit what an agent can do after reading them.
- [3] The dangerous combination. An agent that can reach private data, reads untrusted content and can send data out can be made to leak that data. For any agent that handles sensitive data, remove at least one of the three. Data can leave through any tool argument, such as a URL, an email address or a search query.
- [3] Tool poisoning. A malicious MCP server can hide instructions in its tool descriptions, or change them after you approved it. Connect only servers you trust, pin their versions, and review tool descriptions when they change.
- [3] Supply chain. MCP servers, tool packages and integration platforms are dependencies. Check who publishes them, pin versions and limit what they can reach. A local MCP server runs with the user's own permissions on their machine.
- [2] Designs that resist injection. Some agent designs limit what injected text can do:
  - fix the plan of actions before the agent reads any untrusted content
  - have a separate model with no tools read the untrusted content, and pass on only limited, checked results
  - track which data came from untrusted sources, and block it from sensitive actions

  They reduce the risk, at some cost in flexibility.

### 5.11 Supply chain [3]

From the Security concern. Learn this before you connect outside tools, servers or models.

Your product depends on models, tools, packages and data that other people publish, and each is a possible way in. MCP servers and tool packages are covered in 5.10 Security in tools and protocols, model files in 1.10 Security in foundation models, and poisoned training data in 11.6 Security in the data layer.

- [3] Everything is a dependency. Model providers, open-weight models, MCP servers, agent frameworks, packages, prompt templates from shared hubs and public datasets can all be compromised or change without notice. Use official sources, pin versions, review changes before upgrading, and keep a list of what you depend on.
- [3] Invented packages. Models sometimes suggest packages that don't exist, and attackers register those names with malicious code (slopsquatting). Check that every package a model or coding agent adds is the real one you meant before installing it.
- [3] Coding agents on your machine. Coding agents run commands, read files and install packages with your permissions, and they can be injected through what they read, such as a README, an issue or a web page. Limit what they may run without asking (see 5.12 Code execution and sandboxes), keep production credentials off the machines they run on, and review what they change.
- [2] Models from others. Open-weight models and fine-tunes can hide planted behaviors that appear only on certain inputs (see 1.10 Security in foundation models), and inspection can't reliably find them. Prefer well-known publishers, and run your security tests on any model you download.
- [2] Knowing what you run. Keep a list of the models, datasets, tools and versions in each product, sometimes called an AI bill of materials. When a model or package turns out to be compromised, it tells you what's affected.

Check: List every model, MCP server, framework and AI-related package one product depends on, with its version and source. Pin anything that isn't pinned. Then show how a package a coding agent adds gets checked before it's installed.

### 5.12 Code execution and sandboxes [3]

From Layer 3, Tools and protocols. Learn this when your agents need to run code.

Agents that can run code can calculate exactly, analyze data, convert files and test their own work. But code the model writes is untrusted: it can be wrong, or written on an attacker's instructions.

- [3] Always in a sandbox. Run model-written code in an isolated sandbox, never in your application's process or on a machine with secrets or production access.
- [3] Sandbox options. You can use a provider's built-in code execution tool, a hosted sandbox service such as E2B, Modal or Daytona, or your own containers. Plain containers share the host's operating system kernel. MicroVMs, such as Firecracker, and gVisor isolate code more strongly. WebAssembly runtimes can sandbox code inside a single process.
- [3] Locking it down. Limit CPU, memory, disk and running time. Block network access by default, and allow only the domains a task needs. Keep secrets out of the sandbox: if code must call an API, send the call through a proxy outside the sandbox that adds the credentials.
- [3] State. Decide whether a sandbox keeps files and variables between calls, like a notebook, or starts fresh each time. Kept state is convenient, but never share a sandbox between users, and destroy it when the session ends.
- [3] Files in and out. Move inputs in and outputs out through controlled paths, and check generated files before you show or use them.
- [2] Code instead of tool calls. Instead of calling tools one at a time, an agent can write code that calls them, so intermediate results stay in the sandbox rather than passing through the context. For multi-step work, this can cut tokens and round trips sharply. It needs a sandbox, and the code can fail in new ways.
- [2] Tools for coding agents. Coding agents rely on a standard set of tools: read and edit files, run shell commands and search code. They add rules about which commands can run without asking. Learn this to level 3 if you build one.
- [2] Skills. Some agents support skills: folders of instructions, scripts and files that the agent loads only when a task needs them. They add abilities without adding tool definitions.

Check: Give an agent a code execution tool in a sandbox with no network access, a memory limit and a time limit. Show that it can analyze an uploaded file, that it can't reach the internet or your secrets, and that the sandbox is destroyed when the session ends.

### 5.13 Evaluation in tools and protocols [3]

From Layer 3, Tools and protocols.

- [3] Checking actions, not just answers. Check whether the agent called the right tools, with correct arguments and without wasted calls, and whether the task succeeded. Where possible, check the end state, such as the record in the database, rather than the agent's final message.
- [3] Mock tools and staging. Test against mock tools or staging systems that return realistic results, including errors, slow responses and empty results. Record real tool responses and replay them for repeatable tests. Building full environments is covered in 12.7 RL environments.
- [3] Tool changes are prompt changes. Every change to a tool's name, description, parameters or results can change agent behavior. Run the evaluation set before shipping it.
- [3] Safety cases. Include tasks where the right move is to ask for approval, refuse or stop, and tasks with injected instructions in tool results. Check that the agent doesn't take the bait.
- [2] Benchmarks. Public benchmarks such as the Berkeley Function Calling Leaderboard (BFCL) and τ-bench compare how well models use tools. Use them to shortlist models (see 3.17 Choosing models), not to predict results with your tools.

### 5.14 Reliability in tools and protocols [3]

From Layer 3, Tools and protocols.

- [3] Tools fail. APIs behind tools time out, hit rate limits and return errors. Retry temporary errors with backoff inside the tool. When retries run out, return a clear error, so the model can try another way or tell the user. Retries of model calls are covered in 8.3 Fallbacks and retries.
- [3] Protecting downstream systems. An agent can call an API far faster than a person. Limit tool calls per user and per run, so a looping agent can't overload a system or use up its limits for everyone else.
- [2] When a tool is down. Decide whether the agent should continue without it, use an alternative, or stop and tell the user.
- [2] Changing tools safely. Renaming a tool or changing its parameters or results can break agents, like any API change. Version tools and roll out changes gradually. Pin the versions of MCP servers you depend on, so they don't change under you.

### 5.15 Observability in tools and protocols [3]

From Layer 3, Tools and protocols.

- [3] Tracing tool calls. Record every tool call in the trace: the tool, its arguments, its result or a summary, errors and latency. Link each one to the model call that requested it and to the request's trace ID (see 8.9 Observability in model gateway and routing). OpenTelemetry's conventions for generative AI include standard names for tool calls.
- [3] Audit logs. For every action that changes something, keep a lasting record: which agent, for which user, with which inputs, what changed and who approved it. Traces are for debugging. Audit logs are for accountability, and are kept longer.
- [2] Tool usage. Track which tools are used most, which fail most and which are never used. Unused tools cost tokens and add confusion, and failing ones point to design or reliability problems.
- [2] Redaction. Tool arguments and results often contain personal data and secrets. Remove them before logging.

### 5.16 Cost and latency in tools and protocols [3]

From Layer 3, Tools and protocols.

- [3] Round trips. Each tool call adds a model call to read its result, and each of those resends the growing context. A task with ten tool calls in sequence pays for about ten model calls. Task-level tools, parallel calls and code that calls tools all cut the count.
- [3] Tokens. Tool definitions are sent on every call, and long results fill the context (see 2.2 Context engineering). Keep definitions stable, so they stay in the prompt cache (see 2.4 KV-cache management).
- [3] Slow tools. One slow tool stalls the whole agent. Measure each tool's latency, include it in the latency budget (see 1.5 Inference performance), and show users progress while tools run.
- [2] Execution costs. Sandboxes and browsers bill for running time, and many third-party APIs charge per call. Include these in cost per task (see 1.7 Cost and latency in foundation models), and shut down idle sandboxes and browsers.

### 5.17 Governance and safety in tools and protocols [2–3]

From Layer 3, Tools and protocols.

- [3] Human oversight. Write down which actions agents may take alone, which need approval and which they may never take. Enforce it in code (see 5.5 Actions and side effects, and 5.7 Permissions).
- [3] Approved tools and servers. Keep a list of the third-party tools and MCP servers your company has approved, with what data each receives and under what terms. Block the rest, as you would unapproved models (see 8.11 Governance and safety in model gateway and routing).
- [3] Accountability. When an agent acts, your company acts. Every action should trace back to the user who asked, the agent that acted and the rules it followed (see Audit logs in 5.15 Observability in tools and protocols).
- [2] Terms and law. Automating a website or API can break its terms of service. Agents that send messages, buy, sign or publish can create legal obligations. Bring in legal before agents act in these ways.
- [2] Regulation. Laws such as the EU AI Act require human oversight for some high-risk uses. Know enough to recognize when your agents fall under them.

Data quality has little of its own in this layer. Tool results feed straight into the agent's next decision, so validate them against a schema, and include timestamps where freshness matters.

### 5.18 Browsers and computer use [2]

From Layer 3, Tools and protocols. Learn this only for systems with no API.

Agents can operate websites and desktop apps the way people do. This is the last resort, for systems with no API. Models that operate computers are covered in 6.20 Multimodal models. This section covers the environments they run in. Learn it to level 3 if you build agents that operate websites or desktops.

- [2] Browser automation. Libraries such as Playwright let an agent open pages, click, type and read. The agent can read a page as structure, through its HTML or its accessibility tree (a list of the page's elements), or as screenshots. Structure is faster and cheaper, and screenshots catch what the structure misses.
- [2] Desktop environments. Agents that use a whole computer need a virtual machine with a virtual screen. Run it isolated, with only the apps, files and access the task needs.
- [2] Hosted browsers. Services such as Browserbase run many isolated browsers for agents, with a recording of each session for debugging.
- [2] Logins. Never give the model passwords. Use saved sessions, or a credential store that fills them in outside the model's view. Let a person take over for logins, CAPTCHAs and payments.
- [2] Reliability and cost. Browser agents are slow, expensive and fragile: pages change, load slowly and show pop-ups. Use an API whenever one exists.
- [2] Hostile pages. Any page can contain injected instructions, and a browser agent can act on them. Limit the sites it may visit, and require confirmation before it submits forms, buys or sends anything (see 5.10 Security in tools and protocols).

Check: Build a browser agent that completes one task on a test site. Over 20 runs, measure its success rate, time and cost, and compare them with an API if one exists. Show that it stops for confirmation before submitting a form.

### Key reading for Stage 5

- OWASP Top 10 for LLM Applications (2025), especially "Excessive Agency", and OWASP Top 10 for Agentic Applications (2026).
- Simon Willison's writing on prompt injection, including "The lethal trifecta for AI agents" (2025).
- Greshake et al., "Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection" (2023).
- Beurer-Kellner et al., "Design Patterns for Securing LLM Agents against Prompt Injections" (2025), and Debenedetti et al., "Defeating Prompt Injections by Design" (Google DeepMind, 2025).
- Hines et al., "Defending Against Indirect Prompt Injection Attacks With Spotlighting" (Microsoft, 2024).
- Wallace et al., "The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions" (OpenAI, 2024).
- Spracklen et al., "We Have a Package for You! A Comprehensive Analysis of Package Hallucinations by Code Generating LLMs" (2024).
- NIST, "Adversarial Machine Learning: A Taxonomy and Terminology of Attacks and Mitigations" (NIST AI 100-2), and MITRE ATLAS.
- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 6 covers agents, tools and how agents fail.
- Anthropic, "Building effective agents" (2024), especially its appendix on designing tools, and "Writing effective tools for agents" (2025).
- Yang et al., "SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering" (2024), on why tool design matters.
- The Model Context Protocol specification and documentation (modelcontextprotocol.io).
- Anthropic, "Code execution with MCP" (2025), and Cloudflare, "Code Mode: the better way to use MCP" (2025), on agents that write code to call tools.
- Stripe, "Designing robust and predictable APIs with idempotency" (2017).
- Invariant Labs, "MCP Security Notification: Tool Poisoning Attacks" (2025).
- Yao et al., "τ-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains" (2024), and the Berkeley Function Calling Leaderboard.
- Your model provider's tool use documentation, and the documentation for the sandbox and browser tools you use.

## Stage 6. Retrieval, memory and data quality

Retrieval gives the model knowledge it lacks, and memory lets a product remember users across sessions. This stage covers embedding models, the retrieval pipeline from documents to cited answers, agents that search for themselves, and long-term memory. It also covers the quality of the data the model reads and writes, since most of that data enters the system here.

| Section | Level | Why this level | From |
|---|---|---|---|
| 6.1 Embedding models | 3 | Search quality starts with the embedding model | Layer 8 |
| 6.2 Retrieval-augmented generation | 3 | The main way to give a model knowledge it lacks | Layer 4 |
| 6.3 Evaluation in context, memory and retrieval | 3 (some topics 4) | You can't fix a bad answer until you know which step failed | Layer 4 |
| 6.4 Preparing documents | 3 | Search can't find what parsing and chunking lost | Layer 4 |
| 6.5 Search | 3 (internals 2) | Keyword and vector search each miss what the other finds | Layer 4 |
| 6.6 Query handling and reranking | 3 | Often the biggest gains once a basic pipeline works | Layer 4 |
| 6.7 Answering from retrieved context | 3 | Retrieval helps only if answers stick to the sources | Layer 4 |
| 6.8 Agentic retrieval | 3 | Agents increasingly search for themselves, in several steps | Layer 4 |
| 6.9 Long-term memory | 3 | Memory writes, so its mistakes last | Layer 4 |
| 6.10 Security in context, memory and retrieval | 3 | Every retrieved document and memory is untrusted input | Layer 4 |
| 6.11 Data quality foundations | 3 | You rarely own the data your features depend on | Data quality |
| 6.12 Data the model reads | 3 | Models use bad input without noticing | Data quality |
| 6.13 Data the model writes | 3 (some topics 4) | Model mistakes written into your systems start to look like facts | Data quality |
| 6.14 Data quality in context, memory and retrieval | 3 | Stale or wrong documents produce confident wrong answers | Layer 4 |
| 6.15 Governance and safety in context, memory and retrieval | 3 | Indexes and memories copy personal data that must stay deletable | Layer 4 |
| 6.16 Observability in context, memory and retrieval | 3 | You can't debug an answer without the context it saw | Layer 4 |
| 6.17 Cost and latency in context, memory and retrieval | 3 | Context size drives the cost of most calls | Layer 4 |
| 6.18 Reliability in context, memory and retrieval | 2–3 | Search is one more service that can fail | Layer 4 |
| 6.19 Knowledge graphs | 2 | Worth the cost only when questions follow relationships | Layer 4 |
| 6.20 Multimodal models | 2–3 | Level 3 for the input types your product handles | Layer 8 |

### 6.1 Embedding models [3]

From Layer 8, Foundation models. Learn this before you build search.

Embedding models turn text, and sometimes images, into lists of numbers called vectors. Similar content gets similar vectors. Search systems are built on embedding models, and how those systems work is covered in 6.5 Search.

- [3] Similarity. Two vectors are compared with cosine similarity or a dot product, and a higher score means the meanings are more similar. A score only means something relative to other scores from the same model.
- [3] Choosing an embedding model. Weigh these factors:
  - quality on data like yours
  - the languages it supports
  - its maximum input length
  - its vector size, which drives storage and search cost
  - its price
  - whether you need to run it yourself

  The MTEB leaderboard is a starting point, but test on your own queries.
- [3] Queries and documents. Many embedding models expect you to say whether a text is a search query or a document, using a prefix or a parameter. Getting this wrong quietly lowers search quality.
- [3] Switching models. Vectors from different models can't be compared with each other. Changing the embedding model means re-embedding everything, so record which model and version produced each vector.
- [3] What embeddings miss. Embeddings capture topic and meaning, but they handle exact terms (such as product codes, names and IDs), numbers and negation poorly. Similar isn't the same as relevant. This is why search systems combine embeddings with keyword search and reranking (see 6.5 Search and 6.6 Query handling and reranking).
- [2] Smaller vectors. Some models produce vectors you can shorten with little loss of quality (Matryoshka embeddings). Vectors can also be stored at lower precision. Both cut storage and search costs.
- [2] Rerankers. A reranker reads a query and a document together and scores how relevant the document is. It is more accurate than comparing embeddings but too slow to run on everything, so it's used to reorder the top results.
- [2] Other kinds of embeddings:
  - Sparse embeddings give weights to important words.
  - Multi-vector models, such as ColBERT, keep a vector for each token to match more finely.
  - Multimodal embeddings put images and text in the same space, so a text query can find images.

Check: Compare two embedding models on 50 real queries from your product whose correct documents you know. Report which model finds more of them and what each costs to store.

### 6.2 Retrieval-augmented generation [3]

From Layer 4, Context, memory and retrieval. Build a simple pipeline end to end, with an evaluation set that measures retrieval and answers separately.

Retrieval-augmented generation (RAG) finds relevant information and adds it to the context before the model answers. It's the main way to give a model knowledge it lacks: private data, recent events and anything after its knowledge cutoff (see 3.20 Pre-training corpora). This section covers the whole pipeline, and 6.4 Preparing documents to 6.7 Answering from retrieved context cover the parts.

- [3] When to retrieve. Retrieve when the model needs facts that are private, recent, too large for the context, or must be cited. Fine-tuning is unreliable for teaching facts (see 12.2 Fine-tuning and preference datasets), and retrieval stays current without retraining. Whether to retrieve or load everything is covered in 2.2 Context engineering.
- [3] The two halves. Offline, you prepare knowledge: collect documents, parse them, split them into chunks, then embed and index the chunks. Online, for each request, you turn it into search queries, search, rerank and filter, and put the best results in the context. A failure at any step shows up as a bad answer at the end.
- [3] Start simple. Build the simplest complete pipeline first, with an evaluation set. Then improve whichever step the evaluation shows is weakest. Most gains come from better parsing, chunking, hybrid search and reranking, not from elaborate techniques.
- [3] Frameworks and managed services. Libraries such as LlamaIndex and LangChain, and managed services such as the file search built into provider APIs or Amazon Bedrock Knowledge Bases, handle much of the pipeline. They're a quick start, but their default parsing and chunking often suit your documents poorly. Know what each step does, so you can inspect and change it.
- [2] Named techniques. Papers and vendors use many names, such as HyDE, RAG-Fusion, Self-RAG, corrective RAG and GraphRAG. Most combine the steps covered in 6.4 Preparing documents to 6.7 Answering from retrieved context. Know them well enough to tell what a claim actually changes.

Check: Build a complete RAG pipeline over at least 1,000 of your own documents, with an evaluation set of 50 real questions. Measure retrieval and answer quality separately, and name the step that most needs improving.

### 6.3 Evaluation in context, memory and retrieval [3, some topics 4]

From Layer 4, Context, memory and retrieval.

Building retrieval test sets is covered in 3.4 Evaluation: building evaluation datasets. This part covers what to measure.

- [4] Finding which step failed. A bad answer can come from retrieval (the right information wasn't found), from context assembly (it was found but cut, buried or contradicted), or from generation (it was in the context, but the model ignored or misused it). Measure each step separately, and label each failure by step during error analysis. Otherwise you'll tune the wrong part.
- [3] Retrieval metrics. For queries whose relevant chunks you know, measure:
  - recall at k: the share of relevant chunks that appear in the top k results
  - precision at k: the share of the top k results that are relevant
  - ranking scores, such as mean reciprocal rank (MRR) and normalized discounted cumulative gain (nDCG), which reward putting the best results first

  Recall matters most for the candidate list before reranking. Precision and ranking matter most for what reaches the model.
- [3] Answer metrics. Faithfulness: is every claim supported by the retrieved context? Relevance: does the answer address the question? Correctness: does it match the reference answer, where there is one? Also check that citations are accurate. Model judges can score these, but calibrate them against human labels first (see 3.4 Evaluation: building evaluation datasets). Libraries such as Ragas implement common versions.
- [3] Testing memory. Memory needs tests that span sessions. Check that the product recalls what it should, updates what changed, forgets what was deleted, and doesn't use memories where they don't apply.
- [3] Testing every change. Changes to prompts, chunking, embedding models, retrieval settings and memory rules all change answer quality. Run the evaluation set before each one ships, as you would for a model change.
- [2] Public benchmarks. Retrieval benchmarks such as BEIR, memory benchmarks such as LongMemEval, and long-context tests such as needle in a haystack rarely predict results on your data. Use them to shortlist, not to decide.

### 6.4 Preparing documents [3]

From Layer 4, Context, memory and retrieval. Improve whichever pipeline step your evaluation shows is weakest.

Search can only find what this step kept. It runs offline, before anyone asks a question, and many retrieval failures start here.

- [3] Sources and connectors. Knowledge lives in files, wikis, ticket systems, databases, email and chat. Pull it in with connectors, and keep each document's ID, source, owner, dates and access permissions.
- [3] Parsing. Turn each document into clean text that keeps its structure: headings, lists, tables and reading order. PDFs are hardest, especially scans, multi-column layouts and tables. Tools range from plain text extraction to layout-aware parsers such as Docling or Unstructured, optical character recognition (OCR) and vision models that read page images. Bad parsing fails silently, so inspect the output from real documents by eye. Sending whole documents to a multimodal model is covered in 6.20 Multimodal models.
- [3] Chunking. Split documents into pieces small enough to retrieve precisely but large enough to make sense alone. The main approaches are:
  - fixed size in tokens, with some overlap between neighboring chunks
  - by structure, such as sections, paragraphs, or functions in code
  - by meaning, starting a new chunk where the topic changes

  Splitting by structure is usually the best start. Test chunk sizes on your evaluation set rather than copying a default.
- [3] Context for each chunk. A chunk cut from its document can lose what it refers to, such as which product "it" means. Add the document title and section heading to each chunk. Or have a model write a sentence placing the chunk in its document, and index that with the chunk (contextual retrieval).
- [3] Metadata. Store fields such as source, date, author, product, language and permissions with each chunk. You'll use them to filter searches, favor newer documents, cite sources and enforce permissions.
- [3] Search small, read big. Search over small chunks for precision, then give the model the surrounding section or the whole document (parent-document retrieval).
- [3] Keeping the index in sync. Documents are added, changed and deleted. Update the index incrementally, remove deleted documents promptly, and record when each document was last indexed. A deleted document that search still finds is both a quality bug and a compliance problem.
- [2] Structured data. Tables and databases are usually better queried than embedded. Let the model write SQL or call an API for them (text-to-SQL). A generated query can run without error and still answer the wrong question, so test it like any other retrieval. Learn this to level 3 if your knowledge lives mainly in databases.
- [2] Images and diagrams. When figures carry the meaning, describe them with a vision model and index the descriptions, or embed page images directly with a multimodal embedding model (see 6.1 Embedding models).

Check: Parse and chunk 100 of your hardest real documents in two different ways. Inspect a sample of chunks by eye, then compare retrieval on your evaluation set. Show that deleting a source document removes it from search results.

### 6.5 Search [3, internals 2]

From Layer 4, Context, memory and retrieval.

Search finds the chunks most likely to help with a request. What embedding models are and how to choose one is covered in 6.1 Embedding models. This section covers building search with them.

- [3] Keyword search. Keyword search matches the words in the query, usually scored with BM25. That formula gives rare words more weight, and gives each extra repeat of a word less credit. It's fast and cheap, and best at exact terms such as names, product codes, error messages and IDs. It misses synonyms and rephrasings.
- [3] Vector search. Vector search embeds the query and finds the chunks with the most similar vectors. It matches meaning, so it finds rephrasings, but it handles exact terms and negation poorly (see 6.1 Embedding models).
- [3] Hybrid search. Run both searches and merge the results, often with reciprocal rank fusion, which combines the two rankings without needing comparable scores. Hybrid search is the usual default, because each method finds what the other misses.
- [3] Filters. Narrow results by metadata, such as date, product or customer. If a database filters only after the vector search, a strict filter can leave too few results, so check how yours does it.
- [3] Where to store vectors. The main options are:
  - a vector extension to a database you already run, such as pgvector for PostgreSQL
  - a search engine with vector support, such as Elasticsearch or OpenSearch
  - a dedicated vector database, such as Pinecone, Qdrant, Weaviate or Milvus
  - an embedded library, such as Chroma or LanceDB, for prototypes and small collections

  Choose on scale, filtering, hybrid search, separating customers' data, operations and cost. For many products, a database you already run is enough.
- [2] Approximate search. Comparing a query with every vector is exact but slow at scale. Vector databases use approximate nearest-neighbor (ANN) indexes, such as HNSW or IVF, that find nearly all the closest vectors far faster, and some compress vectors too. Their settings trade recall for speed and memory, so measure recall at the settings you use.
- [2] Search you already have. Many companies already run a search engine over their content, and web search APIs cover public information. Calling an existing service can be quicker than building an index, but you inherit its ranking and its gaps.

Check: For 50 real queries with known correct documents, compare keyword, vector and hybrid search. Report recall at 10 for each (see 6.3 Evaluation in context, memory and retrieval), and find one kind of query that each method misses.

### 6.6 Query handling and reranking [3]

From Layer 4, Context, memory and retrieval.

User messages are rarely good search queries, and first-pass results are rarely in the best order. These steps fix both.

- [3] Rewriting queries. Have a model turn the user's message into one or more search queries. In a conversation, rewrite follow-ups so they stand alone: "what about the second one?" means nothing to a search engine. Pull out filters too, such as dates or product names.
- [3] Several queries. Break a complex question into sub-questions and search for each, or search with several phrasings and merge the results. This finds more, at the cost of extra calls and latency.
- [3] Whether and where to search. Not every message needs a search: a greeting or a question about the last answer doesn't. With several sources, such as documentation, tickets and a database, decide which to search. Rules, a small classifier or the model itself can decide.
- [3] Reranking. Retrieve a generous candidate list, such as the top 50 to 100 chunks, then rerank it and keep the best few. Rerankers are described in 6.1 Embedding models. You can use a hosted reranker, an open-weight one, or a model prompted to score relevance, which is slower and costs more. Reranking is often one of the largest improvements to a basic pipeline.
- [3] How many results. Choose how many chunks to pass to the model, or a reranker score to cut off at, by testing on your evaluation set. Too few misses information. Too many adds noise and cost.
- [2] Searching with a draft answer. Have a model write a plausible answer and search with it, since answers look more like documents than questions do (HyDE). It helps some searches and hurts others, so test it.
- [2] Diversity and other signals. Top results are often near-copies, so remove duplicates or use maximal marginal relevance (MMR), which favors results that add something new. You can also weigh freshness or source authority, for example ranking the current version of a policy above older ones.

Check: Add query rewriting and then a reranker to a basic pipeline. After each change, measure recall, answer quality and added latency on your evaluation set. Keep only the changes that pay for themselves.

### 6.7 Answering from retrieved context [3]

From Layer 4, Context, memory and retrieval.

Retrieval helps only if the model uses what was retrieved and sticks to it. This section covers the answering step of RAG.

- [3] Grounding instructions. Tell the model to answer from the documents provided and to say when they don't contain the answer. Decide for each feature whether the model may add general knowledge: some features should refuse without a source, others can fill small gaps.
- [3] Citations. Give each document an ID, and ask the model to cite the source of each claim. Check that each citation exists and supports its claim. Some providers offer built-in citations that return the exact passages used. Show citations so users can check answers themselves.
- [3] Presenting documents. Give each chunk its title, source, date and ID in a consistent format. Put the most relevant material where the model is least likely to miss it, since the middle of a long context is easiest to miss (see 1.2 Capabilities and limits).
- [3] Conflicting and outdated sources. Retrieved documents can disagree, such as an old and a new version of a policy. Filter out superseded documents where you can, include dates, and tell the model how to resolve conflicts, such as trusting the newest source or reporting the disagreement.
- [3] When nothing relevant is found. The model should say so, ask a clarifying question or search again, rather than answer from weak results. Weak results can be worse than none, because models tend to use whatever they're given.
- [2] Checking answers. A second model call can check that each claim is supported by the retrieved text before the answer is shown. It catches some unsupported claims, at extra cost and latency.

Check: Take 50 answers from a RAG feature and check each claim against its cited source. Report how many claims are unsupported and how many citations are wrong. Make one change that lowers both, and confirm it on your evaluation set.

### 6.8 Agentic retrieval [3]

From Layer 4, Context, memory and retrieval. Learn this once you've built agents that use tools (see 5.2 Function calling).

In a fixed pipeline, your code searches once and the model answers. In agentic retrieval, the model decides when to search, what to search for and when it has enough, calling search tools as often as it needs. As models have got better at using tools, this has become common, especially in coding agents and research tools. How tool calling works is covered in 5.2 Function calling. This section covers search as an agent uses it.

- [3] Search as a tool. Give the agent search tools with clear descriptions and useful parameters, such as filters, that return short results labeled with their sources. The agent can then search, read, refine its query and search again.
- [3] Pipeline or agent. A fixed pipeline is faster, cheaper and more predictable. An agent handles vague or multi-part questions better, and can recover from a bad first search. Use a pipeline for simple lookups where speed matters, and an agent where questions need several steps. Many products route each request to one or the other.
- [3] Searching without an index. Agents can search the way people do: list files, search for exact strings with a tool such as grep, open files and follow references. Coding agents often work this way, with no embedding index at all. It works well on content with clear structure and names, such as code, and there's no index to keep in sync.
- [3] Web search. Agents can search the web through a provider's built-in tool or a search API, then fetch and read pages. Web results can be wrong, outdated or hostile, like any retrieved text (see 6.10 Security in context, memory and retrieval).
- [3] Knowing when to stop. Agents can stop early on thin evidence, or keep searching long after they stop learning anything. Tell the agent what "enough" looks like, and cap its searches and tokens.
- [2] Deep research. Research agents run many searches over several minutes, read dozens of sources and write a cited report. Know how they work and what they cost, as a pattern for long questions that need many sources.
- [2] Connecting data sources. Protocols such as MCP expose many data sources to an agent as tools. Which sources to connect, and how much of their output enters the context, belong to this layer. The protocol itself is covered in 5.9 Model Context Protocol (MCP).

Check: For 30 multi-step questions from your product, compare a fixed RAG pipeline with an agent that has the same search as a tool. Report answer quality, number of searches, latency and cost for each, and decide which kinds of question should go to which.

### 6.9 Long-term memory [3]

From Layer 4, Context, memory and retrieval.

Long-term memory lets a product remember users and past work across sessions. Unlike retrieval, memory writes: it decides what to store, update and forget. So a wrong memory keeps causing wrong answers until something corrects it.

- [3] Kinds of memory. Each kind is stored and used differently:
  - facts, such as the user's role, preferences and current projects (semantic memory)
  - past events, such as earlier conversations and what was decided in them (episodic memory)
  - ways of working, such as how this user likes a task done, or notes an agent writes for itself (procedural memory)
- [3] Writing memories. A model can write memories in the background after a session, the agent can write them during the conversation with a memory tool, or the user can ask ("remember this"). Save facts that will matter later, not everything that was said. Store each memory with its source and date.
- [3] Updating and forgetting. New information can contradict old memories, for example when a user changes jobs. Update or replace the old memory instead of keeping both. Merge duplicates, and let memories expire when they go stale. Forgetting is part of the design, not a failure.
- [3] Reading memories. Retrieve the memories relevant to the current request, or load a short profile into every call. Too many memories distract the model, and bringing up unrelated personal details makes users uneasy.
- [3] Storage. Memories can live in a vector store, a database table, a knowledge graph that records when each fact was true, or plain files that an agent reads and edits. Records and files are easier to inspect, correct and delete than embeddings alone. Services and frameworks such as Mem0, Zep and Letta manage memory for you. Some providers offer a memory tool, where the model decides what to save and your application stores it.
- [3] Scope. Decide whether each memory belongs to a user, a team, a project or the whole organization. Keep scopes separate, so one user's memories never reach another user's context.
- [3] Memory for agents. Agents that return to the same project can keep notes: what they learned about a codebase, which approaches failed and the user's conventions. Many coding agents read a project instructions file, such as AGENTS.md or CLAUDE.md, at the start of every session, and both people and agents edit it.
- [3] User control. Let users see what the product remembers about them, correct it, delete it and turn memory off. Show when a memory is used. This is also a privacy requirement (see 6.15 Governance and safety in context, memory and retrieval).
- [3] How memory goes wrong. A memory can be wrong from the start because the model misunderstood. It can go stale, or be applied in the wrong place, such as a preference from one project applied to another. An attacker can also plant one (see 6.10 Security in context, memory and retrieval). Test memory with scripted scenarios that span several sessions.

Check: Build memory for one feature, covering writing, updating, retrieval and deletion. Run a scripted scenario across five sessions in which the user's preferences change. Show that the product uses the new preference, not the old one, and that a deleted memory never comes back.

### 6.10 Security in context, memory and retrieval [3]

From Layer 4, Context, memory and retrieval.

- [3] Indirect prompt injection. Every retrieved document, web page, email, tool result and memory is untrusted text that can contain instructions aimed at the model (see 1.10 Security in foundation models). Retrieval widens the risk: anyone who can get text into your index can try to steer your model. Mark retrieved content as data, limit what an agent that reads untrusted content can do, and never let retrieved text grant permissions. General defenses are covered in 5.8 Prompt injection.
- [3] Permission-aware retrieval. Users should retrieve only what they may see in the source system. Store permissions with each chunk, filter every search by the user's identity inside the search itself, and keep permissions in sync as they change. Asking the model to hold back something it has already been shown doesn't work.
- [3] Keeping customers apart. When you serve many customers, keep each one's data in a separate index, namespace or partition, or filter every query by customer. One missing filter can leak one customer's documents to another.
- [3] Memory poisoning. Anyone who can get text into a conversation or document can try to plant false or malicious memories that affect later sessions. Review what gets written to memory, and keep memories scoped to the user they came from.
- [2] Leaking data through output. Injected instructions can make the model leak retrieved data, for example inside a link or image address that your product then loads. Block or check links and images in model output. The risk is highest when one agent has private data, reads untrusted content and can send data out.
- [2] Embeddings aren't anonymous. Text can be partly reconstructed from its embedding, so protect vector stores as carefully as the documents they came from.

### About Data quality

A model doesn't notice when its inputs are wrong. Give it an outdated policy, a duplicated record or a tool result with a missing field, and it writes a fluent, confident answer from them. Many failures that look like model failures are data failures (see 2.2 Context engineering). AI systems also write data: extracted fields, summaries, tags, memories, labels and synthetic examples flow back into databases, indexes and training sets, where a mistake spreads and starts to look like fact.

So data quality runs in both directions, for every kind of data in the system: the training data and evaluation sets in 11.4 Data quality in the data layer, the documents and memories in 6.14 Data quality in context, memory and retrieval, tool results and other structured data, and everything the model writes.

For every AI feature, data quality answers four questions:

1. Can you trust what the model reads? (6.11 Data quality foundations, 6.12 Data the model reads)
2. Can you trust what the model writes? (6.13 Data the model writes)
3. Where did each piece of data come from, and what did it affect? (11.10 Lineage and impact)
4. Is it still true, and how would you know? (11.8 Change over time, 11.9 Checking and fixing data)

Most concepts in this concern are level 3, and two things are level 4:
- marking and checking everything the model writes into your systems, so its mistakes can be found and fixed later
- tracing any answer back to its sources, and any source forward to everything built from it

Data engineering, such as building pipelines, warehouses and data catalogs, is a discipline of its own, and data teams often own the sources you depend on. Learn their tools to level 2, or to level 3 where your features run their own pipelines, and work with the owners of your sources rather than around them.

The layers' own sections cover data quality as it applies to each layer, mainly retrieved documents and memories in Layer 4 and training data and evaluation sets in Layer 10. The Data quality sections cover what they share, and 14.7 Data quality at each layer maps where the rest is covered.

### 6.11 Data quality foundations [3]

From the Data quality concern.

These ideas come up in every other section, so learn them first.

- [3] The parts of quality. Data quality has several parts: accuracy, completeness, consistency, freshness, uniqueness, validity, and whether the data looks like real use (see 11.4 Data quality in the data layer). Apply them to every kind of data in the system, not only to datasets: documents, database records, tool results, memories and what the model writes.
- [3] Every source, listed. List every source of data that reaches the model or feeds your datasets: document collections, databases, tool APIs, memories, uploads, feedback, synthetic data and other agents' outputs. For each, note its owner, how often it changes and how far you trust it.
- [3] Agreements with source owners. You rarely own the data your features depend on. Agree with each source's owner on what the data will look like, how fresh it will be and how they'll warn you before changing it. Agreements like these are often called data contracts.
- [2] Quality has a cost. Perfect data is impossible, and chasing it is expensive. Decide how good each source must be for the features that use it, and spend where errors do the most harm.

Check: List every source of data for one feature, with its owner, how often it changes and how far you trust it. Take a sample of 50 records from the least trusted source, check them against the parts of quality, and report what you find.

### 6.12 Data the model reads [3]

From the Data quality concern.

Retrieved documents, memories and the context as a whole are covered in 2.2 Context engineering and 6.14 Data quality in context, memory and retrieval, and checking tool arguments and results in 5.2 Function calling and 5.17 Governance and safety in tools and protocols. This section covers what all inputs share, especially structured data such as tool results and database records.

- [3] Checks at every boundary. Before data enters the context, check tool results, API responses, database records and other agents' outputs against a schema and sensible ranges (see 7.2 Workflow patterns). Watch for failures that look like answers, such as a tool that returns an empty list when it fails, which the model reads as "nothing found".
- [3] Dates on everything. Give every fact in the context its date, or the time it was true as of, so the model and the user can tell how current it is. Decide how fresh each source must be for each feature, and check it before use.
- [3] Units, codes and blanks. Models misread structured data that a person would question: amounts without currencies, times without time zones, internal codes without meanings, and blanks that could mean zero, unknown or not applicable. Label units, explain codes and make missing values explicit before the data reaches the model.
- [3] One source of truth for each fact. When systems disagree, such as a billing system and a CRM that list different plans for the same customer, decide which system is authoritative for each kind of fact, and use that one. Documents that conflict are covered in 6.7 Answering from retrieved context.
- [3] Inputs from users. Uploaded files can be corrupted, badly scanned, password-protected or in an unexpected language or format. Detect this and tell the user, rather than answering from whatever text came out (see parsing in 6.4 Preparing documents).
- [2] Definitions for data the model queries. When a model writes queries against your data, it needs to know what each column and metric means, such as what counts as an active customer. A data dictionary, or a semantic layer that defines metrics in one place, cuts down on queries that run but answer the wrong question (see 6.4 Preparing documents).

Check: For one feature, list every kind of structured data that reaches the model, and add schema checks, dates, units and explicit missing values to each. Then show one case where a check caught bad data the model would otherwise have used.

### 6.13 Data the model writes [3, some topics 4]

From the Data quality concern. Learn this before any feature writes into a system other people rely on.

AI features increasingly write data, not just text for a person to read: fields extracted into a CRM, tags on tickets, summaries stored with records, memories, labels and synthetic examples. Memories the model writes are covered in 6.9 Long-term memory, and synthetic data and labels made by models in 3.11 Synthetic data and 3.3 Human labeling. This section covers what they share.

- [4] Marking what the model wrote. Store every value the model wrote with a mark that says a model produced it, which model and prompt version, from which inputs and with what confidence, kept apart from values people entered or confirmed. Without the mark, the model's mistakes become indistinguishable from facts, and nobody can find them later to check or fix them.
- [3] Checks before writing. Before generated values are written, check them against schemas, allowed values and rules that span fields, such as an end date that must follow the start date. Send values that fail, or that the model was unsure of, to a person for review (see 7.6 Handoffs to people).
- [3] Mistakes that spread. A wrong extracted value feeds reports, other features, search indexes and future training data. Know where each kind of generated data flows, so that when a prompt or model turns out to be wrong, you can find every copy (see 11.10 Lineage and impact).
- [3] Model output isn't ground truth. Don't put model outputs into evaluation sets or training data as correct answers without review (see 3.3 Human labeling and 3.11 Synthetic data). Mark documents and summaries that AI wrote in your own sources, so that retrieval and training can tell them apart from material people wrote.
- [2] Fixing past outputs. When you fix a prompt or upgrade a model, decide whether to regenerate past outputs. Regenerating a large backlog costs money and can overwrite corrections people made, so regenerate only values nobody has confirmed, in batches you can roll back.

Check: For one feature that writes data, mark every generated value with the model, prompt version and inputs, and add checks before writing. Then simulate a bad prompt release: find every value it wrote and correct them, and show that values people had confirmed were left alone.

### 6.14 Data quality in context, memory and retrieval [3]

From Layer 4, Context, memory and retrieval.

In this layer, data quality means the quality of what you retrieve and remember.

- [3] Freshness. Stale documents produce confident, wrong answers. Track when each source was last updated and last indexed, and refresh each source as often as it changes.
- [3] Duplicates and versions. Near-duplicates crowd out other results, and old versions contradict new ones. Remove duplicates during ingestion, and mark or remove superseded versions.
- [3] Source quality. Some sources are authoritative. Others are drafts, opinions or simply wrong. Rank or filter by source, and exclude sources that answers shouldn't come from.
- [3] Lineage. For each chunk, record the source document and version, the parser, the chunking settings and the embedding model. Then you can trace a bad answer to its origin, and rebuild the index when any step changes.
- [3] Gaps. Log the questions that retrieval can't answer. They show what content is missing, and give content owners a list of what to write.

### 6.15 Governance and safety in context, memory and retrieval [3]

From Layer 4, Context, memory and retrieval.

- [3] Personal data in indexes and memories. Indexes and memories copy data out of its original systems. Know what personal data they hold, remove or mask what you don't need, and apply the source's retention rules.
- [3] Deletion. When a user asks for deletion, or a source document is deleted, remove it from indexes, caches and memories too, including anything derived from it, such as summaries and extracted facts. Design for this from the start, because finding every copy later is hard.
- [3] Consent and transparency for memory. Tell users what the product remembers and why, and get consent where the law requires it. Don't store sensitive personal data, such as health, religion or political views, unless the product needs it and the user agrees.
- [2] Rights to the content. Check that you may index and show each source, especially web content and licensed material. Quote only as much as a citation needs.
- [2] Where the data lives. Vector databases and memory stores hold customer data too, so they must follow the same regional and retention rules as your other data stores.

### 6.16 Observability in context, memory and retrieval [3]

From Layer 4, Context, memory and retrieval.

- [3] Logging the context. For every call, log the assembled context, or enough to rebuild it exactly: the prompt version, the search queries, the retrieved chunk IDs with their scores and ranks, the memories used, and the token count of each part. You can't debug an answer without the context it saw. Linking traces to user feedback is covered in 11.2 Product feedback.
- [3] Retrieval in production. Track how often searches return nothing useful, and which documents are retrieved most and which never. These show gaps and drift before users complain.
- [2] Index health. Track index size, the last successful sync for each source, ingestion failures and search latency.

### 6.17 Cost and latency in context, memory and retrieval [3]

From Layer 4, Context, memory and retrieval.

- [3] Context size drives cost. Input tokens are most of the cost of many RAG and agent calls. A smaller, better-chosen context is usually the biggest saving, and prompt caching the next (see 2.4 KV-cache management).
- [3] Retrieval latency. Rewriting queries, embedding them, searching and reranking all happen before the model starts. Measure each step, run independent steps in parallel, and drop steps that don't pay for themselves. Latency budgets are covered in 1.5 Inference performance.
- [3] Indexing costs. Embedding a large collection, re-embedding it when you change models, adding context to every chunk with a model, and building a knowledge graph all cost money before any user benefits. Estimate these costs before you start, and use batch APIs for bulk work (see 8.12 Cloud capacity).
- [2] Storage costs. Vector storage grows with the number of chunks and the size of each vector. Smaller vectors and compression cut it (see 6.1 Embedding models).

### 6.18 Reliability in context, memory and retrieval [2–3]

From Layer 4, Context, memory and retrieval.

- [3] When search fails. Decide what the product does when search is slow or down: answer without retrieval and say so, use cached results, or tell the user. Give retrieval its own timeout, so a slow search doesn't stall the whole request.
- [2] Rebuilding indexes. Changing the embedding model or chunking means rebuilding the index. Build the new index alongside the old one, test it, then switch over, so search never goes down. Record the index version in each trace, so you can reproduce what the model saw.

### 6.19 Knowledge graphs [2]

From Layer 4, Context, memory and retrieval. Learn this if your evaluation shows relationship questions failing.

A knowledge graph stores entities, such as people, products and companies, and the relationships between them. It helps with questions that follow relationships across many records, which chunk search handles poorly.

- [2] What they're good for. A question such as "which customers use a product that depends on this part?" follows a chain of connections across many records. Chunk search finds passages that mention the words, not the chain. Graphs also help with questions about a whole collection, such as the main themes across a year of reports.
- [2] Building one. You can build a graph from structured data you already have, or have a model extract entities and relationships from documents. Model extraction is costly and imperfect: it misses relationships, and the same entity turns up under different names. Merging those duplicates (entity resolution) is much of the work.
- [2] Querying one. Graph databases such as Neo4j are queried with languages such as Cypher. A model can write the query, or your code can fetch an entity and its neighbors and add them to the context as text.
- [2] GraphRAG. Microsoft's GraphRAG and similar methods build a graph from documents, group related entities into communities and summarize each one. They answer broad questions about a whole collection from those summaries, which chunk search can't do, but indexing costs far more.
- [2] When to use one. Start with hybrid search and metadata filters. Add a graph only when your evaluation shows that relationship questions fail and matter. Good metadata and a relational database you already have often give most of the benefit.

Check: Find a kind of question in your product that depends on relationships between records. Show whether hybrid search answers it, and estimate what building and maintaining a graph for it would cost.

### 6.20 Multimodal models [2–3]

From Layer 8, Foundation models. Learn this as your product needs it.

Multimodal models take in or produce more than text, such as images, documents, audio and video.

- [3] Image input. Models can read screenshots, photos, charts, diagrams and handwriting. They are weaker at counting objects, reading very small text, locating things precisely and reasoning about space. Images cost tokens, usually more at higher resolution.
- [3] Document input. Many models accept PDFs directly. They read both the text and an image of each page, including tables and charts. This often works better than extracting the text first, though it costs more tokens. Building document-parsing pipelines is covered in 6.4 Preparing documents.
- [2] Speech and audio. Speech-to-text models transcribe audio, and text-to-speech models produce natural voices. Native audio models take in and produce speech directly, which cuts delay in voice conversations and keeps tone of voice. Learn this to level 3 if you build voice products.
- [2] Image generation and editing. These models create or edit images from text instructions. Know what they do well, where their results are inconsistent, and the policies that restrict what they'll produce.
- [2] Computer use. Some models can operate a computer from screenshots, clicking and typing like a person. This is slower and less reliable than calling an API, so use it only when no API exists. The sandboxes these agents run in are covered in 5.18 Browsers and computer use.
- [1] Video. Models can understand video input and generate short clips. Video generation is expensive and still maturing.

Check: Send the same 20 real documents or screenshots from your product to two models. Compare their accuracy and token cost, and list the cases that both get wrong.

### Key reading for Stage 6

- Muennighoff et al., "MTEB: Massive Text Embedding Benchmark" (2022), and the MTEB leaderboard.
- Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (2020), the paper that named RAG.
- Anthropic, "Introducing Contextual Retrieval" (2024), on adding context to chunks, hybrid search and reranking.
- Manning, Raghavan and Schütze, *Introduction to Information Retrieval* (2008, free online), on keyword search, ranking and retrieval metrics.
- Jason Liu's writing on systematically improving RAG applications.
- Es et al., "RAGAS: Automated Evaluation of Retrieval Augmented Generation" (2023).
- Edge et al., "From Local to Global: A Graph RAG Approach to Query-Focused Summarization" (Microsoft, 2024).
- Packer et al., "MemGPT: Towards LLMs as Operating Systems" (2023), and Park et al., "Generative Agents: Interactive Simulacra of Human Behavior" (2023), on memory for agents.
- Greshake et al., "Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection" (2023), and Simon Willison, "The lethal trifecta for AI agents" (2025).
- Sambasivan et al., "Everyone wants to do the model work, not the data work: Data Cascades in High-Stakes AI" (2021), on how data problems compound.
- Barr Moses, Lior Gavish and Molly Vorwerck, *Data Quality Fundamentals* (O'Reilly, 2022).
- Shumailov et al., "AI models collapse when trained on recursively generated data" (*Nature*, 2024), on why model output shouldn't be treated as ground truth.

## Stage 7. Agents

With tools and retrieval in place, you can decide how much a feature should do on its own. This stage covers workflows and agents, planning, long-running tasks, handoffs to people, the screens people use to steer agents, and systems of several agents.

| Section | Level | Why this level | From |
|---|---|---|---|
| 7.1 Workflows and agents | 4 | How much autonomy a feature gets shapes everything else | Layer 2 |
| 7.2 Workflow patterns | 3 | Most features need a workflow, not an agent | Layer 2 |
| 7.3 Agent loops and planning | 3 | An agent is only as good as its goal, checks and stopping rules | Layer 2 |
| 7.4 Evaluation in agents and orchestration | 3 (some topics 4) | An agent that works once may fail the next time | Layer 2 |
| 7.5 State and long-running tasks | 3 | Long tasks meet crashes, deploys and waits for people | Layer 2 |
| 7.6 Handoffs to people | 3 (some topics 4) | People catch what agents get wrong, if they're asked at the right points | Layer 2 |
| 7.7 Users in control of agents | 3 | People must be able to see, steer and stop what agents do | Layer 1 |
| 7.8 Multi-agent systems | 3 | Several agents help some tasks and hurt others | Layer 2 |
| 7.9 Frameworks and platforms | 2 (one framework 3) | Frameworks change fast, and the patterns underneath them don't | Layer 2 |
| 7.10 Observability in agents and orchestration | 3 | Agent failures are found by reading runs | Layer 2 |
| 7.11 Cost and latency in agents and orchestration | 3 | Agents multiply calls, tokens and waiting | Layer 2 |
| 7.12 Reliability in agents and orchestration | 3 | Every extra step is another chance to fail | Layer 2 |
| 7.13 Security in agents and orchestration | 3 | Injected instructions can pass from one agent to another | Layer 2 |
| 7.14 Governance and safety in agents and orchestration | 2–3 | Someone must be able to see and stop what agents do | Layer 2 |
| 7.15 Agent-to-agent protocols | 2 | Needed only when agents cross team or company lines | Layer 3 |

### About Layer 2: Agents and orchestration

This layer decides who chooses the steps. In a workflow, your code fixes the sequence of steps, and the model does the work inside each one. In an agent, the model chooses its own next step, in a loop, until it decides the task is done. Every step adds cost, latency and one more chance to fail, and handing control to the model makes the system harder to predict and test. So the central skill in this layer is giving each feature the least autonomy that does the job, and making whatever autonomy you give visible, recoverable and interruptible.

For every feature, this layer answers four questions:

1. Who chooses the steps: your code or the model? (7.1 Workflows and agents, 7.2 Workflow patterns, 7.3 Agent loops and planning)
2. Is the work done by one agent or several? (7.8 Multi-agent systems)
3. Where does the task's state live, and how does it survive failures and waits? (7.5 State and long-running tasks)
4. When does a person step in? (7.6 Handoffs to people)

Like Layers 3, 4 and 5, this layer is yours. Frameworks supply building blocks, but only you know which tasks need an agent, how they can go wrong, and where a person must decide. So most concepts in this layer are level 3, and three things are level 4:
- choosing between a workflow and an agent, and how much autonomy each feature gets
- deciding where people step in, and when to step back as an agent proves reliable
- measuring whether agents actually finish their tasks

Learn the range of agent frameworks to level 2 and one framework to level 3, because frameworks change quickly and the patterns underneath them don't. Designs where many agents debate or work as peers are level 2, because they rarely beat simpler designs in production.

### 7.1 Workflows and agents [4]

From Layer 2, Agents and orchestration. Most of the value is in knowing when not to build an agent.

This section covers the first decision for any feature: who chooses the steps.

- [4] Workflow or agent. In a workflow, your code decides the steps, and each step is a model call or ordinary code. In an agent, the model decides what to do next, calls tools, reads the results and repeats until it judges the task done. Workflows are cheaper, faster, more predictable and easier to test. Agents handle tasks whose steps can't be known in advance.
- [4] The least autonomy that works. Try a single well-prompted call first, then a fixed workflow, and use an agent only where the steps really can't be predicted. Add autonomy when your evaluation shows the simpler design falls short, not because agents are in fashion.
- [4] When agents fit. Agents do best when the task is open-ended, the agent can check its own progress (for example by running tests), mistakes are cheap to catch and undo, and the task is worth the extra cost. Coding and research are the clearest successes so far.
- [3] Compounding errors. Errors multiply across steps. If each step succeeds 95% of the time and nothing catches mistakes, a 10-step task succeeds only about 60% of the time. Shorten paths, check results between steps, and let agents notice and fix their own errors.
- [3] Mixing both. Most real systems combine the two: a workflow with one step handled by an agent, or an agent that calls fixed workflows as tools. Anything that doesn't need judgment belongs in plain code.
- [3] The harness. An agent is a model plus the code around it: the loop, tools, instructions, context management, permissions and stopping rules. This code is called the harness, and the same model can do much better or worse in different harnesses.

Check: Build one feature three ways: as a single call, a fixed workflow and an agent. Compare quality, cost, latency and consistency on your evaluation set, and write down which you'd ship and why.

### 7.2 Workflow patterns [3]

From Layer 2, Agents and orchestration. Most features need a workflow, not an agent.

Most features need a workflow, not an agent. These patterns cover most workflows, and they combine freely.

- [3] Chaining. Split the task into fixed steps, each working on the previous step's output, such as outline, then draft, then edit. Pass structured data between steps, and add checks between them that stop or repair bad output before it spreads.
- [3] Routing. Classify each request and send it down the path built for that kind of request, with its own prompt, tools and model. Choosing which model answers a call is covered in 8.6 Routing. This is choosing which path.
- [3] Parallel steps. Run independent pieces at the same time and combine the results, such as checking a contract against several policies at once, or summarizing each of 500 documents and then combining the summaries. Or run the same step several times and take a vote, for a more reliable judgment.
- [3] Orchestrator and workers. A model breaks the task into pieces that code can't predict in advance, hands each piece to a worker, and combines the results. It's like parallel steps, except the model decides the pieces.
- [3] Generate and critique. One call produces a result, and another checks it against clear criteria and gives feedback. The loop repeats until the result passes or hits a limit. It works when you can say what "good" means, and when feedback actually improves the result.
- [3] Checks as steps. Safety and policy checks can run as steps in a workflow, often in parallel with the main call, and stop the run when they fail. Which checks to run is covered in 5.8 Prompt injection, 10.3 Jailbreaks and abuse and 10.10 Content policy and guardrails.
- [2] Batch workflows. Some workflows run offline over large volumes, such as tagging a backlog of support tickets. Run them through queues and batch APIs (see 8.12 Cloud capacity), and make them safe to restart partway.

Check: Rebuild one multi-step feature as an explicit workflow with checks between steps. Show on your evaluation set that the checks catch bad intermediate outputs, and report quality, cost and latency for each step.

### 7.3 Agent loops and planning [3]

From Layer 2, Agents and orchestration. Build one agent by hand.

The loop itself, in which the model calls tools and reads results, is covered in 5.2 Function calling. This section covers making that loop reach its goal.

- [3] A clear goal and finish line. Tell the agent what the task is, what finished looks like, and what it must not do. Without a clear finish line, agents stop early or keep going long after they should.
- [3] Checking its own work. Give the agent ways to test its progress against reality: running tests, comparing results with the data, validating output. Agents that get real feedback at each step do far better than agents that only reason about what they did.
- [3] Planning. For long tasks, have the agent write a plan, such as a to-do list, then update it as it works and re-plan when something unexpected happens. A written plan keeps long tasks on track, and lets a person check the approach before the work starts.
- [3] Stopping rules. Stop the agent when it finishes, when it hits a limit on steps, time or cost, when it stops making progress (such as repeating the same call), or when it needs help. Decide what the user gets in each case. Spending caps are enforced in the gateway (see 8.5 Spending caps and cost control).
- [3] Recovering from errors. When a step fails, the agent should read the error, try a different approach, and ask for help after repeated failures instead of looping.
- [3] Tasks longer than one context window. Some tasks outlast any single context window. Design them to be resumed: work in small, verified increments, keep a progress file and a list of what's left, and leave things in a clean state after each increment, so a fresh session can pick up. Context techniques such as compaction are covered in 2.3 Conversation history.
- [2] Reflection and multiple attempts. An agent can review a failed attempt, write down what went wrong and try again with that note. Running several attempts and keeping the best, judged by tests or a grader, can raise success on hard tasks, at a multiple of the cost.

Check: Build an agent for one task with a written plan, a way to check its own work and stopping rules. Run it 20 times, and report its success rate, how often each stopping rule fired, and what it did when a tool failed.

### 7.4 Evaluation in agents and orchestration [3, some topics 4]

From Layer 2, Agents and orchestration. Learn this before you rely on any agent.

- [4] Measuring task success. Judge an agent by whether it finished the task correctly, checked in the end state, such as the ticket resolved, the tests passing or the record updated, not by its final message. Define success for each task before you build the agent.
- [3] Several runs per task. Agents take different paths on different runs, so run each task several times. Report how often the agent succeeds on every run, not just whether it can succeed once. A user who runs a task ten times needs it to work all ten. τ-bench introduced a measure of this, called pass^k.
- [3] Judging the path. Read trajectories, not just outcomes: wasted steps, wrong tools, risky actions that happened to work out, and right answers reached for the wrong reasons. A model judge can score trajectories against a rubric, once it's checked against human reviewers (see 3.4 Evaluation: building evaluation datasets).
- [3] Testing each step. Evaluate workflow steps and agent roles on their own too, such as the router's accuracy, the planner's plans and each worker's output. End-to-end results show that something failed. Step tests show where.
- [3] Test environments. Agents need somewhere safe to run: mock or sandboxed tools, seeded data and simulated users, reset between runs (see 3.11 Synthetic data and 12.7 RL environments). Run a set of full tasks, several times each, before every release.
- [3] Cost per success. Report cost, time and steps per successful task alongside the success rate. An agent that succeeds a little more often at five times the cost may not be better.
- [2] Benchmarks. Agent benchmarks such as SWE-bench, τ-bench, GAIA, WebArena, OSWorld and Terminal-Bench each measure a different kind of agentic work. Know what each measures, and don't expect their scores to predict results on your tasks.

### 7.5 State and long-running tasks [3]

From Layer 2, Agents and orchestration.

A single model call finishes in seconds. A task can run for hours, wait days for a person, and meet crashes and deploys along the way. This section covers tracking where each task stands.

- [3] Task state. Give every task an ID and a status, such as queued, running, waiting for input, waiting for approval, completed, failed or cancelled. Store it in a database with the steps taken and their results, so users, operators and the agent itself can see where each task stands.
- [3] Background tasks. Work that takes more than a few seconds shouldn't hang on one web request. Put it in a queue, run it on workers, show progress, and notify the user when it finishes.
- [3] Durable execution. Save the task's state after each step (a checkpoint), so a task that fails resumes where it stopped instead of starting over. Durable execution engines such as Temporal, and the checkpointing built into frameworks such as LangGraph, do this for you. Resuming can repeat the step that was running, so steps that change things must be safe to repeat (see 5.5 Actions and side effects).
- [3] Pausing and resuming. A task that waits for a person may wait for hours. Save its state, release its resources, and resume from exactly that point when the answer arrives.
- [3] Cancelling. Users and operators must be able to stop a task at any point. Stop cleanly: finish or undo the current step, record what was done and tell the user.
- [2] Triggers. Not every task starts with a user's message. Agents can start on a schedule, a new email, a webhook or a change in another system; these are sometimes called ambient agents. No one is watching when they run, so their limits, notifications and handoffs matter more.
- [2] Deploying while tasks run. When you release a new prompt, tool or workflow, tasks already running started under the old one. Decide whether they finish on the old version or switch, and record the version in each task's state.
- [2] Conflicts. Two tasks or agents may change the same record or file at once. Use locks, separate working copies such as separate code branches, or a single owner for each resource.

Check: Run a multi-step task as a background job with a status the user can see. Kill the worker partway through and show that the task resumes from its last checkpoint without repeating completed actions. Then pause it for approval, restart all workers while it waits, approve it and show that it finishes.

### 7.6 Handoffs to people [3, some topics 4]

From Layer 2, Agents and orchestration.

People catch what agents get wrong, but only if they're asked at the right points. Which actions need approval is covered in 5.5 Actions and side effects, and approval screens are covered in 7.7 Users in control of agents. This section covers where people fit into the flow of work.

- [4] Where people step in. For each workflow, decide the points where a person acts: approving risky actions, answering the agent's questions, reviewing a plan before work starts, checking the result before it's used, and taking over when the agent can't finish. Each point costs someone's time, so put them where they prevent the most harm.
- [4] Autonomy that grows with evidence. Start a new agent with more review. Measure how often reviewers change or reject what it proposes, and remove review steps for kinds of task where it has proven reliable. Keep review where mistakes are costly.
- [3] Asking instead of guessing. An agent should ask a clarifying question when a request is ambiguous and the answer matters, and otherwise proceed on a sensible assumption and say what it assumed. Too many questions annoy users, so test where the line falls.
- [3] Escalation. Set clear triggers for handing a task to a person: the user asks for one, the agent fails repeatedly or is unsure, or the topic is sensitive. Route each handoff to the right person or team, with a target response time.
- [3] Handing over context. Give the person who takes over a summary of the goal, what's been done and why they're needed, with the full history available. Nobody should have to ask the user to repeat themselves.
- [3] Review fatigue. People asked to approve too often start approving without reading. Keep approvals rare, specific and easy to judge: show exactly what will change and why. Group low-risk items, and track how often reviewers change anything.
- [3] Acting on the decision. The person's decision goes back into the task, and the agent continues from there. Treat an edit as the new plan, and ask for a reason with each rejection, so the agent can use it.
- [2] Reviews as data. Every approval, edit and rejection is a label. Collect them for evaluation and improvement (see 11.2 Product feedback).

Check: For one agent, map every point where a person can step in and what they see there. Add an escalation path that hands over a summary. Over a week, measure how often people approve, edit or reject at each point.

### 7.7 Users in control of agents [3]

From Layer 1, Product and UX. Learn this once you ship agents.

Where people step into agent workflows is covered in 7.6 Handoffs to people, and which actions need approval in 5.5 Actions and side effects. This section covers the screens people use to see, steer and stop agents.

- [3] Approval screens. Show exactly what the agent is about to do, with a preview such as the email text, the diff or the order. Say why approval is needed, and make approving, editing and rejecting equally easy.
- [3] Plans users can steer. Show the agent's plan before and during the work, and let users change it, skip steps or add constraints.
- [3] Stopping and taking over. Let users pause, stop or redirect an agent at any point, and take over the task themselves.
- [3] Reviewing results. When an agent finishes, show what it did, what changed, what it was unsure of and what needs the user's attention, not just "done". Link each claim to the detail behind it.
- [3] How much the agent does alone. Where it makes sense, let users choose how much the agent does without asking: ask before every action, ask only before risky ones, or act freely within set limits.
- [2] Many tasks at once. Users who run several agents need one place to see every task, its status and what's waiting for them.

Check: Build the approval and review screens for one agent. With five users, measure how long an approval takes and whether they catch a mistake planted in a preview. Improve the preview until most of them do.

### 7.8 Multi-agent systems [3]

From Layer 2, Agents and orchestration. Learn this once a single agent falls short.

A multi-agent system splits a task across several agents, each with its own instructions, tools and context. Keeping each agent's context small is covered in 2.3 Conversation history, and connecting agents across companies in 7.15 Agent-to-agent protocols.

- [3] When several agents help. Split the work when subtasks are independent and can run in parallel, such as researching several companies at once, or when each subtask needs its own tools or a clean context.
- [3] When they hurt. When subtasks depend on each other, agents working separately make conflicting choices, because each sees only part of the picture. Building a single feature in code usually goes better with one agent. Multi-agent systems also use far more tokens and are harder to debug. Try one agent first.
- [3] Lead agent and sub-agents. The most common design has a lead agent that plans the work, starts sub-agents with specific tasks and combines what they return. Sub-agents usually report only to the lead. Deeper hierarchies are harder to control and debug.
- [3] Delegating well. A sub-agent knows only what it's told. Give each one a clear objective, the context it needs, the output format, the tools to use and where its task ends. Vague instructions lead to duplicated work, gaps and sub-agents that wander.
- [3] Scaling effort. Match the number of agents and steps to the task: one agent and a few calls for a simple question, many sub-agents for broad research. Tell the lead agent how to judge this, or it will overspend on easy tasks and underspend on hard ones.
- [3] Handing off a conversation. In some designs, one agent passes a conversation to another, such as a triage agent passing a customer to a billing specialist. Pass the full history or a careful summary, so the next agent doesn't start blind or ask the user to repeat themselves.
- [3] Sharing results. Agents can pass results as messages, or write them to shared storage, such as files or a database, and pass a reference. Large results belong in storage, not in every agent's context.
- [2] How multi-agent systems fail. One large study groups the failures into three kinds: unclear specifications of the task and each agent's role, agents that misunderstand or ignore each other, and nobody properly checking the final result. Design and test for all three.
- [2] Other designs. Agents can also debate, vote, or work as a team of peers around a shared message board. These appear often in papers and frameworks, but rarely beat a simpler design in production. Know them well enough to judge the claims.

Check: Build one broad task, such as comparing five companies, with a single agent and with a lead agent and sub-agents. Compare quality, tokens, time and cost, and explain which you'd ship.

### 7.9 Frameworks and platforms [2, one framework 3]

From Layer 2, Agents and orchestration. Learn this after you've built the basics by hand.

Agent frameworks package the patterns in this layer. They change quickly, so learn the patterns first and the frameworks second.

- [3] Build by hand first. Build a simple workflow and a simple agent with direct API calls before you adopt a framework. Then you'll know what the framework does for you, and you can debug it. Frameworks add layers that can hide the actual prompts and responses.
- [2] What frameworks offer. Agent frameworks provide the loop, tool handling, state and checkpoints, pausing for people, multi-agent patterns, tracing and ready-made integrations. Widely used ones include LangGraph, the OpenAI Agents SDK, the Claude Agent SDK, Google's Agent Development Kit, CrewAI, Microsoft Agent Framework, Pydantic AI and Mastra.
- [3] One framework in depth. Learn one framework well enough to run it in production: how it stores state, how it pauses and resumes, how it traces, and how to see the raw model calls.
- [2] Graphs. Several frameworks describe a workflow as a graph: steps are nodes, edges decide what runs next, and shared state passes between them. Graphs make the flow of control explicit, including loops.
- [2] Managed platforms. Cloud and model providers offer hosted platforms that run agents for you, with memory, tools, identity and monitoring built in. They save infrastructure work, but tie you to one vendor.
- [2] Visual builders. No-code tools such as n8n let people assemble workflows and agents by connecting boxes. They're quick for simple flows, but harder to test, version and review than code.
- [2] Choosing. Weigh language support, control over prompts and the loop, durability, tracing, lock-in and maturity. Keep your prompts, tools and evaluation sets independent of the framework, so you can switch.

Check: Build the same small agent by hand and with one framework. Show that you can see every raw model call and tool call in both, and pause the framework version for approval and resume it. List what the framework saved you and what it hid.

### 7.10 Observability in agents and orchestration [3]

From Layer 2, Agents and orchestration.

- [3] Tracing whole runs. Trace every run as a tree: the task, each step, and each model call, tool call and sub-agent, linked parent to child. Tools such as LangSmith, Langfuse, Arize Phoenix and Braintrust display agent traces, and OpenTelemetry's conventions for generative AI include agents.
- [3] Reading runs. Agent traces are long. Use views that summarize a run, showing its plan, its steps, the tools it used and where time and tokens went, and that jump straight to failures. Read full runs regularly, not just metrics. Most agent problems are found this way.
- [3] Monitoring tasks. Track tasks by status: how many are running, waiting for people, stuck or failed, and how long each has waited. A task that has waited three days for approval is a product problem.
- [2] Replaying runs. Save enough to replay a run from any checkpoint with a changed prompt or tool, so you can test a fix on the exact case that failed.

### 7.11 Cost and latency in agents and orchestration [3]

From Layer 2, Agents and orchestration.

- [3] Cost per task. Agents make many calls, each resending a growing context, so budget cost per completed task, not per call (see 1.7 Cost and latency in foundation models). Multi-agent systems use far more: one published research system used about 15 times the tokens of a chat.
- [3] Budgets the agent knows about. Caps on calls, tokens and cost are enforced in the gateway (see 8.5 Spending caps and cost control). Also tell the agent its budget, so it can plan within it and wrap up cleanly instead of being cut off partway through a step.
- [3] A model for each step. Use strong models where judgment matters, such as planning and final review, and cheaper, faster models for routing, extraction and routine steps (see 3.17 Choosing models).
- [3] Latency. Each step in sequence adds its full latency, and slow steps compound (see 1.5 Inference performance). Run independent steps and sub-agents in parallel, stream progress to users, and move long tasks to the background.

### 7.12 Reliability in agents and orchestration [3]

From Layer 2, Agents and orchestration.

- [3] Failing well. When an agent can't finish, it should stop with a clear status, keep partial results and say what's left. Agents sometimes report success they didn't achieve, so confirm completion in code where you can.
- [3] Timeouts for whole tasks. Steps have timeouts (see 5.5 Actions and side effects), and so should whole tasks. A stuck task should fail or escalate, not run forever.
- [3] Falling back. When an agent fails, fall back to a simpler workflow, a smaller version of the task, or a person. Fallbacks between models are covered in 8.3 Fallbacks and retries.

Resuming after crashes is covered in 7.5 State and long-running tasks.

### 7.13 Security in agents and orchestration [3]

From Layer 2, Agents and orchestration.

- [3] Trust between agents. Treat another agent's output as data, not instructions, especially if that agent read untrusted content. An injected instruction can travel from a sub-agent that read a web page to a lead agent that can act.
- [3] Separating reading from acting. Keep agents that read untrusted content away from powerful tools, and give agents that act only trusted inputs. Orchestration is where you can build this separation. Designs that resist injection are covered in 5.10 Security in tools and protocols.
- [3] Permissions for each agent. Give each agent in a system the fewest tools and permissions its role needs (see 5.7 Permissions). A sub-agent shouldn't inherit everything the lead agent can do.
- [2] Goal drift. Over long runs, injected text or a build-up of context can pull an agent away from its task. Keep the original goal in view, and check actions against it.

### 7.14 Governance and safety in agents and orchestration [2–3]

From Layer 2, Agents and orchestration.

- [3] Stopping agents. Be able to pause or stop one task, all tasks of one agent, or every agent at once, and test that it works.
- [3] Records of decisions. Keep each task's plan, steps, approvals and results, so you can explain afterwards what the agent did, why, and who approved it. Audit logs of individual actions are covered in 5.15 Observability in tools and protocols.
- [2] Telling users. Make it clear when users are dealing with an agent rather than a person, especially when it acts or writes on their behalf. Laws such as the EU AI Act require this in some cases.
- [2] Unattended agents. Agents that start on triggers, with no user watching, need tighter limits and regular review of what they did.

Data quality has little of its own in this layer. Each step's output is the next step's input, so validate it at every boundary, as covered in 7.2 Workflow patterns. Agent trajectories as training data are covered in 12.2 Fine-tuning and preference datasets.

### 7.15 Agent-to-agent protocols [2]

From Layer 3, Tools and protocols. Learn this when your agents need to work with agents outside your system.

Sometimes an agent needs to hand work to another agent, built by another team or run by another company. Agent-to-agent protocols standardize this. Coordinating agents inside your own system is covered in 7.8 Multi-agent systems.

- [2] A2A. The Agent2Agent protocol (A2A) lets agents from different vendors work together. It started at Google in 2025 and is now run by the Linux Foundation. MCP connects an agent to tools. A2A connects an agent to another agent, which does its own reasoning and keeps its internals private.
- [2] Agent cards. Each agent publishes a card that describes what it can do, where to reach it and how to authenticate, so other agents can find and call it.
- [2] Tasks and results. Work is organized as tasks that move through states, such as working, waiting for input, completed and failed. Results come back as artifacts, such as a document or a set of data. Long tasks can stream updates, or send a notification when they finish.
- [2] When you need it. Most products run all their agents in one system, where ordinary function calls or an orchestration framework are simpler (see 7.8 Multi-agent systems and 7.9 Frameworks and platforms). A2A matters when agents cross team or company boundaries.
- [2] An agent as a tool. The simplest way to connect two agents is to expose one as a tool of the other, through function calling or MCP. A2A adds discovery, task tracking and streaming on top.
- [1] Newer protocols. Other protocols cover payments and checkout by agents, such as AP2 and the Agentic Commerce Protocol, and streaming an agent's progress to a user interface, such as AG-UI. Know they exist, and check how widely each is used before building on it.

Check: Explain the difference between MCP and A2A with an example from your product. For one idea that involves several agents, decide whether A2A, MCP or plain function calls fit best, and explain why.

### Key reading for Stage 7

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 6 covers agents, planning and how agents fail.
- Anthropic, "Building effective agents" (2024), on workflows, agents and when to use each.
- OpenAI, "A practical guide to building agents" (2025).
- Dex Horthy, "12-Factor Agents" (HumanLayer, 2025), on building agents as ordinary software you control.
- Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2022), the pattern behind most agent loops.
- Anthropic, "How we built our multi-agent research system" (2025), and Cognition, "Don't Build Multi-Agents" (2025), for the cases for and against.
- Anthropic, "Effective harnesses for long-running agents" (2025).
- Cemri et al., "Why Do Multi-Agent LLM Systems Fail?" (2025).
- Kapoor et al., "AI Agents That Matter" (2024), on evaluating agents with cost in view.
- OWASP Top 10 for Agentic Applications (2026).
- The documentation for one agent framework, such as LangGraph, and one durable execution engine, such as Temporal.
- Lisanne Bainbridge, "Ironies of Automation" (1983), on why automation makes human oversight harder.
- The A2A protocol specification (a2a-protocol.org).

## Stage 8. The gateway, cost and latency

Before real traffic arrives, every model call should pass through one place. This stage covers the gateway, which routes calls, retries them, caches answers and controls spending, and the capacity you buy from providers. It then covers what each feature costs and how long it takes, and how to make it cheaper and faster without losing quality.

| Section | Level | Why this level | From |
|---|---|---|---|
| 8.1 The gateway | 3 (some topics 4) | Every call passes through it, so its design is yours | Layer 5 |
| 8.2 API keys and access | 3 | Anyone with a leaked key can spend your money | Layer 5 |
| 8.3 Fallbacks and retries | 3 (some topics 4) | Providers fail, and users shouldn't notice | Layer 5 |
| 8.4 Rate limits and quotas | 3 | Provider limits cap you, and your own limits protect you | Layer 5 |
| 8.5 Spending caps and cost control | 3 (some topics 4) | You can't control costs you can't attribute | Layer 5 |
| 8.6 Routing | 3 | Each request should go to the cheapest model that handles it well | Layer 5 |
| 8.7 Caching | 3 | The cheapest model call is the one you don't make | Layer 5 |
| 8.8 Evaluation in model gateway and routing | 3 | Every route and fallback is a model choice to test | Layer 5 |
| 8.9 Observability in model gateway and routing | 3 | The gateway sees every call | Layer 5 |
| 8.10 Security in model gateway and routing | 2–3 | Every request passes through this one checkpoint | Layer 5 |
| 8.11 Governance and safety in model gateway and routing | 2–3 | It enforces which models and regions data may go to | Layer 5 |
| 8.12 Cloud capacity | 2–3 | You buy compute, either directly or through a model API | Layer 9 |
| 8.13 Cost and latency foundations | 3 | Every use costs money, and costs vary widely between requests | Cost and latency |
| 8.14 Cost per unit of value | 3 (some topics 4) | Price per token says little about what a feature costs | Cost and latency |
| 8.15 Reducing cost | 3 (some topics 4) | A few changes usually save most of the money | Cost and latency |
| 8.16 Reducing latency | 3 (some topics 4) | A few changes usually save most of the time | Cost and latency |
| 8.17 Trading off against quality | 3 (some topics 4) | The cheapest design that meets the quality bar wins | Cost and latency |
| 8.18 Keeping spending under control | 3 | Spending that belongs to nobody grows | Cost and latency |
| 8.19 Reliability in cost and latency | 3 | Cutting cost can cut reliability | Cost and latency |
| 8.20 Governance and safety in cost and latency | 3 | The cheapest route may not be an allowed one | Cost and latency |

### About Layer 5: Model gateway and routing

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

1. Where should this request go? (8.6 Routing)
2. What happens if that fails? (8.3 Fallbacks and retries)
3. Can it be answered without calling a model? (8.7 Caching)
4. Is this caller allowed, and can you afford it? (8.4 Rate limits and quotas, 8.5 Spending caps and cost control, and 8.2 API keys and access)

A gateway is worth having because it answers these questions once, in one place.

### 8.1 The gateway [3, some topics 4]

From Layer 5, Model gateway and routing. Put every call through one place first.

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

### 8.2 API keys and access [3]

From Layer 5, Model gateway and routing.

A provider API key lets anyone who holds it spend money on your account.

- [3] Keys stay on the server. Never put provider keys in apps or browser code. Clients call your backend or gateway, which holds the keys.
- [3] Secrets management. Store keys in a secrets manager, not in code or configuration files. Rotate them regularly, and immediately after a leak. Give each environment and service its own key, so that a leak stays contained and usage can be traced.
- [3] Virtual keys. The gateway issues its own keys to teams, services or customers and maps them to provider keys behind the scenes. Each virtual key has its own limits, budget, allowed models and usage record. Revoking one doesn't affect the others.
- [2] Short-lived credentials. Where possible, use short-lived tokens or your cloud platform's own identity system instead of long-lived keys.
- [2] Customers' own keys. Some products let customers supply their own provider keys, so usage is billed to them. Store those keys encrypted, and keep each customer's keys separate from everyone else's.
- [2] Leak detection. Scan code and logs for keys, and watch for sudden spikes in usage that suggest a stolen key. Some providers revoke keys that appear in public code repositories.

Check: Move one product's provider keys into a secrets manager. Give each environment and service its own virtual key with a budget. Rotate a provider key without downtime.

### 8.3 Fallbacks and retries [3, some topics 4]

From Layer 5, Model gateway and routing.

Providers slow down, run out of capacity and go down. This section covers how the gateway keeps requests working when that happens. The ways a single call can fail are covered in 1.9 Reliability in inference and serving.

- [3] Retries with backoff. Retry temporary errors, such as timeouts, overloads, rate limits and server errors. Wait longer before each retry (exponential backoff), and add a random delay (jitter) so that many clients don't retry at the same moment. Follow the provider's retry-after guidance when it gives one. Don't retry errors that will just fail again, such as invalid requests, prompts that are too long or refusals. Keep the total time spent on retries within your latency budget.
- [3] Safe retries. Retrying a model call is safe on its own. It isn't safe if the response has already started streaming to the user or has triggered an action. Retry only before anything has happened, and design actions to be safe to repeat (see 5.5 Actions and side effects).
- [4] Fallback design. For every feature, decide what happens when the main model is unavailable:
  - which alternatives to try, and in what order, such as the same model from another provider or region, then a different model
  - which prompt each alternative uses, since prompts don't carry over perfectly between models
  - what users see if everything fails, such as a cached answer, a simpler path without AI, or a clear message

  Test fallbacks regularly, because a fallback that never runs often turns out not to work.
- [3] Circuit breakers. After repeated failures, stop sending traffic to a failing provider for a while and go straight to the fallback. Without this, every request waits for its own timeout. Send occasional test requests to detect when the provider recovers.
- [2] Hedged requests. For calls where speed is critical, send a backup request to a second provider if the first hasn't started answering within a set time, and use whichever answers first. This cuts the slowest response times at extra cost.

Check: Simulate an outage of your main provider in a test environment. Show three things: requests fail over within your latency budget, the circuit breaker stops traffic to the failed provider, and quality on your evaluation set stays acceptable on the fallback.

### 8.4 Rate limits and quotas [3]

From Layer 5, Model gateway and routing.

Rate limits run in two directions: providers limit you, and you limit your own users. Forecasting capacity and asking providers for higher limits are covered in 8.12 Cloud capacity.

- [3] Provider limits. Providers limit requests and tokens per minute, and sometimes per day. These limits apply per model and per organization or project, and some providers count input and output tokens separately. Know the limits for every model you use, and track your usage against them.
- [3] Client-side throttling. Don't wait for rate-limit errors. Pace requests with a queue or a token bucket, a counter that refills at the rate your limit allows. Base it on the remaining limit that providers report in each response. This smooths out bursts before they hit the limit.
- [3] Prioritization. When capacity is short, serve requests from waiting users before background jobs. Send bulk work to batch APIs or run it at quieter times.
- [3] Your own limits. Limit how many requests, tokens or dollars each user, customer or API key can use. Then one heavy or abusive user can't use up your provider limits or your budget.
- [2] Pooling capacity. Spread traffic across several deployments, regions or accounts of the same model to get more total capacity. Stay within each provider's terms.

Check: Load-test one feature past your provider's token limit. Show three things: throttling keeps rate-limit errors near zero, requests from waiting users are served before background ones, and one heavy test user hits your per-user limit before affecting anyone else.

### 8.5 Spending caps and cost control [3, some topics 4]

From Layer 5, Model gateway and routing.

Model costs grow with usage and can spike without warning. The gateway sees every call, so it's where you measure and limit spending.

- [4] Cost attribution. Tag every call with its feature, customer, user and environment, so you can see what each one costs. You can't control costs you can't attribute. The gateway is the only place to do this consistently.
- [3] Budgets and caps. Set spending limits for each key, team, feature, customer and environment. Send alerts before a limit is reached, and decide what happens at the limit: block requests, switch to a cheaper model or queue them. Providers offer account-wide limits too, but those are too coarse on their own.
- [3] Runaway protection. Agents and loops can make thousands of calls. Cap the number of calls, tokens and cost for each request, session and agent run, and stop any run that exceeds its cap. Set maximum output length and reasoning budgets for each route, so a single request can't become unexpectedly expensive.
- [2] Metering for billing. If you charge customers by usage, the gateway's records of tokens and cost for each customer are the basis for their bills.

Check: Tag every model call with its feature and customer, and build a daily cost report from the tags. Set a budget with an alert for one feature. Cap a test agent run so that it stops at a set cost.

### 8.6 Routing [3]

From Layer 5, Model gateway and routing. It saves the most once the basics are in place.

Routing decides which model handles each request. Which models are good enough for each job is decided by evaluation (see 3.17 Choosing models). This section covers how each request reaches the model you chose.

- [3] Static routing. Assign each feature or task a model in configuration. This is the starting point for every product.
- [3] Rule-based routing. Choose the model for each request from simple signals. Examples are the user's plan, the input's length, its language, whether it includes images, or whether it needs tools. A small classifier can add topic as another signal.
- [3] Routing by difficulty. A fast classifier or small model predicts how hard each request is. It sends easy requests to a cheap model and hard ones to a strong model. Routers like this can cut costs a lot at similar quality, but only if they work on your traffic, so measure them there.
- [3] Cascades. Send each request to a cheap model first. If the answer fails a check, such as validation, a confidence threshold or a grader, send it to a stronger model. This costs little when most requests pass, but escalated requests take longer. It's also how you send hard cases to the teacher model after distillation (see 12.4 Distillation).
- [3] Reasoning effort for each request. Set a reasoning effort or budget for each request: low for simple requests, higher for hard ones. This is routing within one model. It trades cost and latency against quality the same way routing between models does.
- [2] Provider routing. You can often get the same model from several providers: an open-weight model from several hosts, or a lab's model from its own API and from cloud platforms. Route between them by price, speed, region or availability. The same open model can differ in quality between providers (see 13.16 Evaluation in compute).
- [3] Traffic splitting. Send a set share of traffic to a new model or prompt version, and keep each user on the same version. Use this for gradual rollouts and A/B tests, and roll back by changing configuration. When to run online tests is covered in 11.2 Product feedback and 11.3 Experiments with real users.
- [2] Shadow traffic. Send a copy of real requests to a candidate model without showing its answers to users, then compare quality, latency and cost. It costs extra, and the copied data must be allowed to go to that provider.

Check: Add a router to one feature that sends easy requests to a cheap model and hard ones to a strong model. Compare its cost, and its quality on your evaluation set, with sending everything to the strong model. Then roll it out to 10% of traffic, and back again, by changing only configuration.

### 8.7 Caching [3]

From Layer 5, Model gateway and routing.

Response caching returns a stored answer without calling a model at all. Prompt caching is different: it only reuses the provider's processing of a shared prompt start, and the model still generates a new answer. Prompt caching is covered in 2.4 KV-cache management.

- [3] Exact-match caching. Store each response under a key made from the whole request, including the model, prompt and settings. Return the stored response when an identical request arrives. This is cheap and safe, but it only helps when identical requests repeat. That's common for fixed prompts, batch jobs, evaluations and popular questions, and rare in open-ended chat.
- [3] Cache keys and freshness. Include the model version and prompt version in every cache key, so a change to either stops old answers from being served. Set expiry times that fit how quickly the right answer changes, and clear the cache when the source data changes.
- [3] What not to cache. Don't cache answers that are personal, time-sensitive or meant to vary. A cache shared between users can leak one user's data to another, so scope cache keys to the user or customer where needed.
- [2] Semantic caching. Semantic caching returns a stored answer for a request that means the same thing as an earlier one, found by comparing embeddings. It gets more cache hits than exact matching, but it also risks returning the wrong answer to a request that only looks similar. For example, "cancel my order" and "don't cancel my order" can look nearly the same to an embedding. Use it only where a wrong cached answer does little harm. Set a strict similarity threshold, and measure how often it returns wrong answers.

Check: Add exact-match response caching to one feature, with the model and prompt versions in the cache key and an expiry time. Report the hit rate and savings over a week. Show that a prompt change stops old entries from being served.

### 8.8 Evaluation in model gateway and routing [3]

From Layer 5, Model gateway and routing.

- [3] Testing routes and fallbacks. Every route and fallback is a model choice. Test each one on the traffic it will actually receive, and test again when models, prompts or traffic change. A router that sends hard requests to a weak model shows up as a quality drop on one slice of traffic, so check results slice by slice.

### 8.9 Observability in model gateway and routing [3]

From Layer 5, Model gateway and routing.

- [3] Logging at the gateway. The gateway sees every call, so it's the natural place to record the route chosen, any retries and fallbacks, cache hits, cost, and the tags that attribute each call. Pass a trace ID through the gateway, so that each model call links to the user request it served. The rest of what to log for each call is covered in 1.11 Observability in foundation models.

### 8.10 Security in model gateway and routing [2–3]

From Layer 5, Model gateway and routing.

- [3] A policy checkpoint. Every call passes through the gateway, so it's a natural place to remove personal data before it leaves, block unapproved models, and run input and output filters. Which checks to run is covered in 5.8 Prompt injection, 10.3 Jailbreaks and abuse, 10.10 Content policy and guardrails and 10.12 Privacy and personal data.
- [2] Denial of wallet. Attackers can drain your budget by sending expensive requests through your product. The defenses are your own limits and runaway protection, covered in 8.4 Rate limits and quotas and 8.5 Spending caps and cost control.

### 8.11 Governance and safety in model gateway and routing [2–3]

From Layer 5, Model gateway and routing.

- [3] Approved models and regions. Keep a list of the models and providers your company has approved, with their data terms and regions. Enforce the list in the gateway so no feature can quietly use an unapproved model. Route each customer's data only to the regions and providers allowed for it.
- [2] Audit trail. Record which model and provider processed each request, for compliance reviews and investigating incidents.

Reliability is covered in 8.3 Fallbacks and retries, and the gateway itself must be at least as reliable as the providers behind it. Cost and latency are covered in 8.6 Routing, 8.7 Caching, and 8.5 Spending caps and cost control. The latency the gateway itself adds is covered in 8.1 The gateway. Data quality has nothing specific to this layer.

### About Layer 9: Compute

Compute is mostly hidden from you. Model providers own the hardware, and you pay per token. You still run into compute in three places:

1. The limits and prices of model APIs.
2. Running a model yourself: self-hosting an open-weight model, running a fine-tuning job, or trying a model on your laptop.
3. The market forces that set prices and decide where capacity exists.

Learn the arithmetic behind these decisions in depth. Learn how the hardware works well enough to follow that arithmetic. For everything underneath, such as chip design, cluster networking, data centers and power, awareness is enough. No concept in this layer needs level 4. If you need that depth, you're doing an infrastructure job, not an application one.

Most compute questions come down to three calculations:

1. Memory: will the model fit? Multiply the number of parameters by the bytes each one takes. Then leave room for the KV cache (stored intermediate results for each conversation) and for overhead.
2. Speed: how fast can it generate? Each new token requires reading every weight from memory once. So memory bandwidth divided by the model's size in bytes gives the top speed for a single conversation.
3. Cost: what does a token cost? Divide the hourly cost of the hardware by the tokens it produces per hour, at the utilization you'll actually get.

Once you learn these calculations, you use them in 8.12 Cloud capacity and in the Layer 9 sections of Stage 13.

### 8.12 Cloud capacity [2–3]

From Layer 9, Compute. Start with how model APIs expose capacity.

Every model call runs on someone's hardware. You buy that capacity either indirectly, through a model API, or directly, by renting GPUs.

- [3] Ways to buy compute. You can pay per token through a model API, where the provider owns the hardware, or reserve throughput from a model provider. You can rent serverless GPUs billed by the second, or GPU virtual machines billed by the hour. You can also buy your own hardware. Know which option fits a given workload.
- [3] Provider capacity as you see it. Providers set rate limits and usage tiers. At busy times you may see overload errors and slower responses, and some models are available only in some regions. Before a launch, forecast your peak tokens per minute and ask for higher limits early. Handling these errors in code is covered in 8.3 Fallbacks and retries and 8.4 Rate limits and quotas.
- [3] Batch and priority options. Batch APIs run work on spare capacity at a discount, often 50%, and return results within about a day. Use them for evaluations, backfills and synthetic data. Some providers also sell priority or flexible tiers that trade price against speed or reliability.
- [2] Reserved throughput. Paying for a fixed share of a provider's capacity gives you predictable throughput and latency. It's usually worth it only at steady, high volume.
- [2] GPU pricing models. On-demand capacity is the most expensive. Reserved or committed capacity is cheaper, in return for a commitment of one to three years. Spot or preemptible capacity is much cheaper, but the provider can take it back at short notice. Some clouds also sell short reservations for a fixed window.
- [2] Quotas and scarcity. On a new cloud account, GPU quotas often start at zero. The newest chips are scarce and may require a waitlist or a long commitment. Availability differs by region.
- [2] Serverless GPUs and cold starts. You pay by the second and can scale down to zero. But a cold start has to find a machine and load the weights. That takes seconds for small models and can take minutes for large ones. Keep warm instances for anything latency-sensitive.
- [2] Data gravity. Moving data between clouds costs money (egress fees) and time. Put compute near your data, or budget for the transfer.
- [1] The provider landscape. Hyperscalers (AWS, Google Cloud, Microsoft Azure, Oracle), GPU-focused clouds (for example CoreWeave, Lambda, Nebius and Crusoe), and serverless and inference platforms (for example Modal, RunPod, Baseten and Together AI).

Check: For a feature launching next month, forecast peak tokens per minute and compare the forecast with your rate limits. Move one offline job to a batch API. Write down how you'd buy compute if traffic grew tenfold.

### About Cost and latency

Most software costs about the same whether a feature is used once a day or a thousand times. AI features cost money every time they run, and the cost varies widely from one request to the next. A long conversation, a reasoning model or an agent that loops can cost a hundred times more than a short question. AI features are also slow by software standards, because models generate one token at a time and agents run many steps in sequence. And cost and latency trade against quality: a stronger model, more context or more reasoning usually helps, and always costs more.

So the job in this concern is to know what each feature costs and how long it takes, as users experience it, and to find the cheapest and fastest design that still meets the quality bar.

For every AI feature, this concern answers four questions:

1. What does it cost, for the value it delivers? (8.13 Cost and latency foundations, 8.14 Cost per unit of value)
2. How can it cost less? (8.15 Reducing cost)
3. How can it be faster? (8.16 Reducing latency)
4. What's the right balance, and how do you keep it? (8.17 Trading off against quality, 8.18 Keeping spending under control)

Most concepts in this concern are level 3, and three things are level 4:
- knowing each feature's full cost per unit of value, such as per resolved support ticket
- knowing which changes save the most money and time for a feature, and making them without losing quality
- choosing each feature's balance of quality, cost and latency

Hardware costs and self-hosting are covered in 13.2 Compute arithmetic and 13.12 Cost and latency in compute, and how serving engines trade speed for throughput in 1.5 Inference performance and 13.5 Serving engines and batching. Learn those to the levels those sections set.

The layers' own sections cover cost and latency as they apply to each layer, such as context size in Layer 4 and routing and caching in Layer 5. The Cost and latency sections cover what they share, and 14.6 Cost and latency at each layer maps where the rest is covered.

### 8.13 Cost and latency foundations [3]

From the Cost and latency concern.

These ideas come up in every other section, so learn them first.

- [3] Where the money goes. Model tokens are usually the largest cost, but others add up: embedding and indexing, reranking, tool and API calls, sandboxes and browsers, GPU hosting if you run models yourself, stored traces, evaluation runs, labeling, and the time people spend reviewing outputs. Know each one's share for a feature before you try to cut any of them.
- [3] Where the time goes. A request's time splits into network and queueing, the steps before the model starts, such as retrieval and guardrails, the time to the first token, generation, tool calls, and every extra step an agent takes. Output length often dominates (see 1.5 Inference performance), and steps in sequence add up.
- [3] Cost and latency move separately. Some changes cut both, such as caching and shorter outputs. Others cut one and raise the other: running calls in parallel saves time but not money, and batch APIs save money but take hours. More reasoning raises both. Know which one matters more for each feature.
- [3] Cost varies by request. The same feature can cost a cent for one request and dollars for another, depending on the length of the context and the conversation, the reasoning and the number of agent steps. Averages hide this, so look at the distribution, and at the most expensive requests and users (see 9.3 Metrics and dashboards).
- [2] Tokens for different inputs. The same content can need different numbers of tokens in different languages, and images, audio and long documents can cost far more tokens than what the user typed (see 1.1 Working with a model and 1.7 Cost and latency in foundation models). Check costs with the inputs your users actually send.

Check: For one feature, break the cost of a typical request into its parts, and its time into its steps. Then find last week's ten most expensive requests, and explain what made each one expensive.

### 8.14 Cost per unit of value [3, some topics 4]

From the Cost and latency concern.

Price per token is the starting point, not the answer. Cost per task is covered in 1.7 Cost and latency in foundation models and 7.11 Cost and latency in agents and orchestration, and attributing cost to features and customers in 8.5 Spending caps and cost control. This section covers judging whether a feature's cost is worth it.

- [4] The unit that matters. For each feature, choose the unit of value, such as a resolved support ticket, an accepted code change or a finished report. Measure the full cost of each unit: every model call, retry, tool call, guardrail check, evaluation sample and minute of human review that went into it. A cheaper model that resolves fewer tickets can cost more per resolved ticket.
- [3] Failures cost too. Wrong answers lead to retries, handoffs to people, support tickets and users who leave. Count those when you compare designs, because the cheaper design may simply move cost from your model bill to your support team.
- [3] Margins. Compare the cost of each user and each customer with what they pay. Heavy users can cost many times the average (see 4.9 Cost and latency in product and UX), so check the margin on your heaviest users and on each plan, including free ones, not just on the average user.
- [3] Forecasting. Estimate future cost as volume times tokens per request times price. Check what happens when traffic grows tenfold, conversations get longer or a feature becomes an agent. Long conversations and agents make each request use more tokens, not just add more requests (see 2.3 Conversation history).
- [3] Estimates against invoices. Costs worked out from token counts drift from what providers actually bill, because of cached and reasoning tokens, price tiers, batch discounts and calls that bypassed your tracking. Compare your figures with each invoice, and look into any gap.
- [2] Prices fall, usage rises. The price of a given level of capability keeps falling (see 3.17 Choosing models), but new kinds of features, such as reasoning and agents, often raise the tokens per task faster than prices fall. Don't count on cheaper models to fix a feature's margin.

Check: Choose the unit of value for one feature, and compute its full cost per unit for last month, including retries, guardrails and human review. Compare it with the revenue or value each unit brings, and forecast both at ten times today's traffic.

### 8.15 Reducing cost [3, some topics 4]

From the Cost and latency concern.

Most ways to cut cost are covered in the layers' own sections: smaller context in 2.2 Context engineering, routing and caching in 8.6 Routing and 8.7 Caching, prompt caching in 2.4 KV-cache management, fewer tool round trips in 5.16 Cost and latency in tools and protocols, distillation in 12.4 Distillation, and batch pricing in 8.12 Cloud capacity. This section covers choosing among them.

- [4] The usual order of savings. For most features, the biggest savings come in roughly this order. First, avoid calls you don't need, through caching, simple rules, or not running AI that nobody asked for (see 4.9 Cost and latency in product and UX). Then shrink the context, make repeated content cacheable, use a cheaper model for each step that allows it, shorten outputs and limit reasoning, and send work that isn't urgent to batch APIs. Measure quality after each change (see 3.7 Evaluation in development), and stop when it starts to fall.
- [3] Output costs more than input. Output tokens cost several times as much as input tokens (see 13.2 Compute arithmetic), and reasoning tokens are billed as output. Ask for concise answers, set output limits, and choose the reasoning effort for each request (see 8.6 Routing).
- [3] More than one cache. Besides response caching and prompt caching, cache embeddings so that unchanged documents aren't embedded again, cache tool results that a task requests more than once, and reuse summaries of content that hasn't changed. Each cache needs a rule for when its contents go stale.
- [3] Finding waste. Look for spending that buys nothing: retries of calls that will fail again, tool definitions that are never used, long instructions that a shorter version handles just as well, calls made twice by a double click, and evaluation runs that nobody reads. Start with the most expensive features, customers and requests.
- [2] Bigger changes. Fine-tuning a smaller model or hosting a model yourself saves money only at steady, high volume, once you count their full cost (see 12.10 Cost and latency in model adaptation and 13.12 Cost and latency in compute). Try the cheaper fixes first.

Check: For one feature, work through the usual order of savings one change at a time, recording cost per unit and quality after each. Stop when quality falls, and report the total saving and which change saved the most.

### 8.16 Reducing latency [3, some topics 4]

From the Cost and latency concern.

Latency metrics and latency budgets are covered in 1.5 Inference performance, retrieval latency in 6.17 Cost and latency in context, memory and retrieval, latency in agents in 7.11 Cost and latency in agents and orchestration, and how waiting feels to users in 4.3 Streaming and waiting. This section covers choosing among the ways to make a feature faster.

- [4] The usual order of speedups. Start with what runs in sequence, and run independent steps in parallel, such as retrieval and guardrail checks. Then shorten outputs, use a smaller, faster model for steps where users wait, lower the reasoning effort where it doesn't help, and make prompts cacheable to cut the time to the first token. Drop steps that don't earn their time, and move work that needn't happen while the user waits into the background (see 7.5 State and long-running tasks).
- [3] Speed users can feel. Streaming the answer and showing progress make a feature feel faster without changing its total time (see 4.3 Streaming and waiting). Do this first when the total time is hard to cut, but don't let it hide a feature that's simply too slow.
- [3] The slowest requests. Users remember the slow requests, and agents hit them more often because they make many calls (see 1.5 Inference performance). Look at the 95th and 99th percentiles, find what the slowest requests have in common, and fix that.
- [3] Starting early. Start steps that will probably be needed before you know for sure, such as retrieving documents while a classifier decides whether to search, and cancel them if they turn out not to be needed. This costs more, but saves time.
- [3] Network and payloads. Call providers in regions close to your servers, reuse connections, and keep large inputs small: resize images, and send only the pages of a document the task needs. Smaller inputs also cost fewer tokens.
- [2] Faster generation. For edits and other outputs that are easy to predict, predicted outputs and speculative decoding can make generation much faster (see 13.8 Faster generation).

Check: For one feature, measure each step's share of the total time at the 50th and 95th percentiles. Work through the usual order of speedups, measuring after each change, and report how far the 95th percentile fell and what each change cost.

### 8.17 Trading off against quality [3, some topics 4]

From the Cost and latency concern.

Comparing versions on quality, cost and latency together is covered in 3.5 Defining what to measure, and setting latency targets in 9.2 Service levels. This section covers choosing the balance.

- [4] A target for each of the three. Give each feature a quality bar, a latency target and a cost budget per unit of value, and choose the cheapest design that meets the quality bar within the latency target. The balance differs by feature: autocomplete is mostly about speed, overnight document tagging about cost, and contract review about quality.
- [3] Spending more where it pays. Sometimes a stronger model, more reasoning or several attempts are worth what they cost, such as on the hard cases a cascade passes up (see 8.6 Routing). Measure what each extra dollar or second buys in quality, and spend where the gain is largest.
- [3] Different balances for different uses. Free and paid users, background jobs and live requests, and small and large customers can each get a different balance, such as a cheaper model on the free plan (see 4.9 Cost and latency in product and UX). Make sure each one still meets the quality bar for its use.
- [3] Checking again as models change. New models and price cuts change the best balance every few months (see 3.17 Choosing models). Re-run the comparison regularly, and switch when a new design meets the bar for less.

Check: For one feature, set its quality bar, latency target and cost budget. Compare three designs on all three, choose one, and write down what change in prices or models would make you choose differently.

### 8.18 Keeping spending under control [3]

From the Cost and latency concern. Learn this before costs grow large.

Budgets, spending caps and protection against runaway use are enforced in the gateway (see 8.5 Spending caps and cost control), and alerts on cost are covered in 9.10 Alerts. This section covers the habits around them.

- [3] Cost in every change. Have every change to prompts, models, tools or retrieval report its effect on cost and latency per request on the evaluation set, next to its effect on quality (see 3.13 Cost and latency in evaluation). Require approval for any change that raises cost past a set limit.
- [3] An owner for each budget. Give each feature's spending an owner who looks at its cost every week, explains changes and decides what to cut. Spending that belongs to nobody grows.
- [3] Limits for users. Plan limits, usage-based pricing and caps for each user keep heavy users from costing more than they pay (see 4.9 Cost and latency in product and UX). Set them with product and finance, and revisit them as costs change.
- [2] Buying better. Once spending is large, ask providers about volume discounts, committed-spend agreements and reserved capacity (see 8.12 Cloud capacity). Weigh what you save against the flexibility you give up.

Check: Add cost and latency per request to your evaluation reports, and hold back one change that raised cost past your limit until someone approves it. Name an owner for each feature's spending, and hold one weekly review.

### 8.19 Reliability in cost and latency [3]

From the Cost and latency concern.

- [3] Cutting cost can cut reliability. Removing a fallback, shrinking spare capacity or dropping retries saves money until the next outage. Check each cut against the feature's reliability target (see 9.2 Service levels).

### 8.20 Governance and safety in cost and latency [3]

From the Cost and latency concern.

- [3] Cheaper routes must still be allowed. The cheapest provider or region may not be approved for your data (see 8.11 Governance and safety in model gateway and routing and 10.13 Oversight and accountability). Routing for cost has to stay within the approved list.

The other concerns add cost and latency of their own, and each concern covers its share: evaluation runs in 3.13 Cost and latency in evaluation, storing traces in 9.18 Cost and latency in observability, security checks and attackers who run up your bill in 10.8 Cost and latency in security and 10.3 Jailbreaks and abuse, guardrails in 10.21 Cost and latency in governance and safety, and retries, fallbacks and spare capacity in 9.13 Cost and latency in reliability.

Data quality meets cost in what you store and search: duplicate and outdated documents cost embedding, storage and context tokens while making answers worse (see 6.14 Data quality in context, memory and retrieval).

### Key reading for Stage 8

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 9 covers inference optimization, and chapter 10 covers routers, gateways and caches in an AI application's architecture.
- Chen, Zaharia and Zou, "FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance" (2023), on cascades.
- Ong et al., "RouteLLM: Learning to Route LLMs with Preference Data" (2024), on routing by difficulty.
- Marc Brooker, "Timeouts, retries, and backoff with jitter" (Amazon Builders' Library).
- Martin Fowler, "CircuitBreaker" (2014).
- Dean and Barroso, "The Tail at Scale" (2013), on hedged requests.
- Google, *Site Reliability Engineering* (2016), the chapters on handling overload and cascading failures.
- OWASP Top 10 for LLM Applications (2025), especially "Unbounded Consumption".
- The documentation for an open-source gateway such as LiteLLM, and your providers' documentation on rate limits and errors.
- Kapoor et al., "AI Agents That Matter" (2024), on comparing agents by cost as well as accuracy.
- Databricks, "LLM Inference Performance Engineering: Best Practices" (2023).
- Guido Appenzeller, "Welcome to LLMflation" (Andreessen Horowitz, 2024), on how quickly the price of a given capability falls.
- Artificial Analysis (artificialanalysis.ai), for independent comparisons of models' quality, price and speed.
- The FinOps Foundation, "FinOps for AI Overview" (finops.org), on managing AI spending.
- Your model providers' pricing pages, and their guides to prompt caching, batch processing and reducing latency.

## Stage 9. Reliability and monitoring

An AI feature depends on many services that can fail, and on a model whose output varies. This stage covers reliability targets, failure handling across the system, consistent outputs, safe changes, testing for failure and incidents. It also covers the metrics, dashboards and alerts that show when something is wrong.

| Section | Level | Why this level | From |
|---|---|---|---|
| 9.1 Reliability foundations | 3 | AI features depend on many services that fail | Reliability |
| 9.2 Service levels | 3 (some topics 4) | Each feature needs its own target, set from the user's side | Reliability |
| 9.3 Metrics and dashboards | 3 | Metrics show that something changed, and where | Observability |
| 9.4 Retries and fallbacks across the system | 3 | Retries and fallbacks in several layers can make failures worse | Reliability |
| 9.5 Graceful degradation | 3 (some topics 4) | Doing less beats doing nothing | Reliability |
| 9.6 Timeouts and overload | 3 | One slow step can use up the time and capacity others need | Reliability |
| 9.7 Consistent outputs | 3 | Users can't depend on answers that change or break format | Reliability |
| 9.8 Changes without outages | 3 | Many AI incidents start with a change | Reliability |
| 9.9 Testing for failure | 3 | Fallbacks that never run break unnoticed | Reliability |
| 9.10 Alerts | 3 | Someone should know before users complain | Observability |
| 9.11 Incidents and recovery | 3 | Some failures will reach users | Reliability |
| 9.12 Observability tools | 2 (one tool 3) | Tools change quickly, and the practices behind them don't | Observability |
| 9.13 Cost and latency in reliability | 3 | Reliability costs money and time | Reliability |
| 9.14 Security in reliability | 3 | Some checks must stop the request when they fail | Reliability |
| 9.15 Governance and safety in reliability | 3 | Fallbacks must follow the same data rules | Reliability |
| 9.16 Security in observability | 3 | Traces hold secrets and private conversations | Observability |
| 9.17 Governance and safety in observability | 3 | Traces copy personal data out of your product, often to another company | Observability |
| 9.18 Cost and latency in observability | 3 | Full traces are expensive to keep | Observability |
| 9.19 Reliability in observability | 2–3 | Recording must never break the product | Observability |

### About Reliability

An AI feature depends on more outside services than most software: model providers, a gateway, search, embedding models, tools, MCP servers and sandboxes. Each can fail, slow down or return something malformed. Model APIs are young and often run close to capacity, so slowdowns and overload errors are common. On top of that, the model's output varies: the same request can come back different, cut off or in the wrong format. Reliability means the product keeps working through all of this. It retries what's worth retrying, switches to another model or provider, does less when it must, and keeps outputs consistent where users depend on them.

For every AI feature, reliability answers four questions:

1. How reliable must it be? (9.1 Reliability foundations, 9.2 Service levels)
2. What happens when a part fails? (9.4 Retries and fallbacks across the system, 9.5 Graceful degradation, 9.6 Timeouts and overload)
3. Can users depend on what it produces? (9.7 Consistent outputs)
4. How do you keep it working as things change? (9.8 Changes without outages, 9.9 Testing for failure, 9.11 Incidents and recovery)

Most concepts in this concern are level 3, and two things are level 4:
- setting reliability targets for each AI feature, from the user's point of view
- planning the steps each feature takes as parts fail, from full function down to a clear message

The mechanics, such as retries with backoff, circuit breakers and timeouts for single calls, are covered in the layers' own sections, mainly in 8.3 Fallbacks and retries. Site reliability engineering is a discipline of its own, and your company may have teams that run on-call, incident processes and infrastructure. Learn their practices to level 3 as they apply to AI features.

The layers' own sections cover reliability as it applies to each layer, such as fallbacks between models in Layer 5 and resuming agent tasks after crashes in Layer 2. The Reliability sections cover what they share, and 14.5 Reliability at each layer maps where the rest is covered.

### 9.1 Reliability foundations [3]

From the Reliability concern.

These ideas come up in every other section, so learn them first.

- [3] Every dependency can fail. List what each feature depends on: model providers, the gateway, search and embedding services, tools and the APIs behind them, MCP servers, sandboxes, queues and your own services. Each can be down, slow, rate-limited or wrong. Failures multiply along a chain: five dependencies that each work 99.9% of the time all work together only about 99.5% of the time. Agent steps compound the same way (see 7.1 Workflows and agents).
- [3] Kinds of failure. A call can fail outright, with an error or a timeout, or succeed too slowly to be useful, which users experience as failure. It can fail partway, such as a stream that breaks or an answer cut off at the length limit (see 1.9 Reliability in inference and serving). It can succeed with output that's malformed or wrong, which raises no error (see 2.5 Observability foundations). And behavior can drift when a provider changes the model behind a name you use (see 1.4 Frontier proprietary models).
- [3] The model is a flaky dependency. Treat every model response like a reply from an unreliable outside service. Before anything uses it, check that it arrived, that it ended for the expected reason (see stop reasons in 1.1 Working with a model), that it parses and that it follows your rules.
- [3] Containing failures. Keep one failing feature, provider or customer from taking down the rest. Give features and customers their own limits, queues and keys (see 8.2 API keys and access and 8.4 Rate limits and quotas), so a runaway agent or a traffic spike in one place can't use up capacity everywhere.
- [2] Reliability costs money. Each step up in reliability costs more: a second provider, reserved capacity, duplicate requests and more testing. Decide for each feature how much reliability is worth paying for.

Check: For one AI feature, list every dependency, how each can fail and what the user sees when it does. Estimate the feature's availability from its dependencies, and name the failure that would hurt users most.

### 9.2 Service levels [3, some topics 4]

From the Reliability concern. Set targets before you design for them.

A service level objective (SLO) is a target for how reliable and fast a feature must be, such as "99.5% of requests succeed within 8 seconds, measured over 28 days". Measuring service levels and alerting on them are covered in 9.10 Alerts. This section covers setting the targets.

- [4] A target for each feature. Set targets for each AI feature from the user's point of view, based on what users and the rest of the product depend on. Inline autocomplete can fail quietly now and then, but an agent that processes refunds can't. Cover speed as well as success, and write the targets down.
- [3] What counts as success. Decide which responses count as good: complete, valid and within the time limit. Decide too how to count a degraded response, such as an answer from a fallback model or a cached result. Counting it as a success hides problems, and counting it as a failure raises alarms for a system working as designed, so many teams track degraded responses separately.
- [3] Error budgets. A target of 99.5% allows 0.5% of requests to fail, and that allowance is called the error budget. When failures use it up, slow down risky changes, such as switching models or rewriting prompts, and spend the time on reliability instead. Agree on this rule with product before you need it.
- [3] Your providers set a ceiling. Without a fallback, a feature can't be more reliable than the providers it depends on. Know their published availability, any service level agreement they offer and their incident history. Standard API plans often promise little, and paid tiers or reserved capacity may promise more (see 8.12 Cloud capacity).
- [3] Independent fallbacks raise the ceiling. If two providers each fail 0.5% of the time and their failures are independent, both fail at once only 0.0025% of the time. Real failures are often linked, such as when both run in the same cloud region or depend on the same upstream service, so choose fallbacks that fail independently.
- [2] Promises to customers. A service level agreement (SLA) is a promise in a contract, often with credits when it's broken. Set it below your internal target, so you can miss the target now and then without breaking the contract.

Check: Set SLOs for two AI features with different needs, covering success and speed, with a rule for counting degraded responses. Compare them with what your providers promise, and show how much a fallback raises the target you can reach.

### 9.3 Metrics and dashboards [3]

From the Observability concern.

Metrics show the whole system over time, and dashboards put the ones that matter in front of people.

- [3] Tokens, cost and speed. Track token counts, including cached and reasoning tokens (see 1.11 Observability in foundation models), cost per request, task, user and feature (see 8.5 Spending caps and cost control), time to first token and output speed (see 1.5 Inference performance), and cache hit rates.
- [3] How calls end. Track stop reasons, such as outputs cut off at the length limit and refusals, along with tool errors, retries, fallbacks and guardrail blocks. Add signs of quality, such as evaluation pass rates and user feedback (see 11.1 Evaluation in production).
- [3] Breaking down by what changes. Split every metric by feature, model, provider, prompt version and customer. AI problems usually start in one slice: one prompt version, one provider or one customer's documents. A total can look normal while one slice fails.
- [3] Distributions, not averages. Averages hide the slow and the expensive. Track latency and cost per request as percentiles, such as p50, p95 and p99. A few runaway agent runs can cost more than thousands of ordinary chats.
- [3] Dashboards people use. Keep a few dashboards with clear owners: one for each feature, showing traffic, errors, latency, cost and quality together, one for models and providers, and one for spending. Mark releases, prompt changes and model changes on the charts, so you can see what changed when a line moves.
- [2] Cardinality. Metric systems handle labels with a limited number of values, such as model or feature names. Labels with millions of values, such as user IDs or prompt text, make them slow and expensive. Keep that detail in traces, and aggregate it in metrics.

Check: Build one dashboard for a feature that shows traffic, errors, p95 latency, cost per request and a quality signal, broken down by prompt version and model, with releases marked. Use it to explain one change you see on it.

### 9.4 Retries and fallbacks across the system [3]

From the Reliability concern. Learn this after the mechanics in 8.3 Fallbacks and retries.

Retries, circuit breakers and fallbacks for model calls are covered in 8.3 Fallbacks and retries, retries inside tools in 5.14 Reliability in tools and protocols, and fallbacks for agents in 7.12 Reliability in agents and orchestration. This section covers how they work together.

- [3] Retrying in one place. Provider SDKs often retry by default, and so may your gateway, your agent framework and your own code. If three layers each make up to three attempts, one failure can become 27 calls, and an outage turns into a flood. Decide which layer retries each kind of call, and turn retries off in the others.
- [3] Limits on retrying. Cap retries for each request, and across the system, such as allowing retries to add no more than 10% to traffic, so that retries can't make an outage worse. Back off between attempts, with random jitter (see 8.3 Fallbacks and retries).
- [3] Safe to retry. Retry only what's safe to repeat. A model call is safe until its output has been shown or acted on. A tool call that changes something needs an idempotency key (see 5.5 Actions and side effects). An agent task should resume from its last checkpoint rather than start over (see 7.5 State and long-running tasks).
- [3] Fallbacks need capacity. When you fail over to another provider, all your traffic arrives there at once. If your rate limits or quota there were sized for a trickle, the failover causes a second outage. Size the fallback's limits for your peak traffic, and keep it warm with a small share of real traffic.
- [3] Fallbacks need testing. A fallback model, with its own prompt, behaves differently from the main one. Run the evaluation set on every fallback path (see 3.7 Evaluation in development and 8.8 Evaluation in model gateway and routing), and test the switch itself regularly (see 9.9 Testing for failure).

Check: Trace one request's retry behavior through every layer, from the SDK through the gateway to your own code, and remove the duplicate retries. Then fail over one feature's peak traffic to its fallback provider in a test, and show that the fallback's limits hold and its quality on your evaluation set is acceptable.

### 9.5 Graceful degradation [3, some topics 4]

From the Reliability concern.

When parts fail, a feature should do less rather than nothing. What the rest of the product does when AI fails is covered in 4.11 Reliability in product and UX, fallbacks between models in 8.3 Fallbacks and retries, and what to do when search fails in 6.18 Reliability in context, memory and retrieval. This section covers planning the steps.

- [4] Steps from full to off. For each feature, decide in advance the steps it takes as things fail. A typical order is the main model, then a fallback model, then a cached or precomputed answer, then a simpler path that doesn't generate text, such as showing search results instead of a written answer, and finally a clear message with a way forward. Know what triggers each step and what users see at each one.
- [3] Optional parts stay optional. Memory, personalization, reranking, suggestions and other extras should be skipped, not waited for, when they're slow or down. Give each one a short time limit, and let the request succeed without it.
- [3] Saved answers. Cached responses, summaries made in advance and the last good result can stand in when generation fails (see 8.7 Caching). Decide how old a saved answer may be before it's worse than none.
- [3] Shedding features. Under heavy load or during an outage, switch off expensive or low-priority AI features first, such as background summaries, and keep the core ones running. Use feature flags that change in seconds, without a deploy.
- [3] Telling users. If users would trust a weaker result as much as a normal one, tell them the product is running with less, such as "Answers may be shorter while we fix a problem". For small changes that don't affect what users should trust, a quiet fallback is fine.

Check: Write the degradation steps for one feature, with the trigger and what users see at each step. Force each step in a test environment, and show that the feature still does something useful at each one.

### 9.6 Timeouts and overload [3]

From the Reliability concern.

Timeouts for single model calls are covered in 1.6 Calling models through an API, timeouts for whole agent tasks in 7.12 Reliability in agents and orchestration, and rate limits and priorities in 8.4 Rate limits and quotas. This section covers time and load across a whole request.

- [3] One deadline for the whole request. Give each request a deadline, and pass the remaining time to every step, so each step knows how long it may take. Leave time for the fallback: a model call that uses the full deadline leaves nothing for the next option.
- [3] Overload. When demand passes capacity, whether that's your provider's rate limits, your own GPUs or a tool's API, requests pile up and every user waits. Keep queues bounded, turn away excess requests early with a clear message, and serve waiting users before background work (see 8.4 Rate limits and quotas). An unbounded queue turns overload into timeouts for everyone.
- [3] Capacity ahead of demand. Forecast peak traffic for launches and busy periods, and ask providers for higher limits well in advance (see 8.12 Cloud capacity). Leave headroom, because capacity for AI takes time to add.
- [2] Slowing intake. Pipelines that process batches or run agents in the background should slow down when the services they call are saturated, rather than keep pushing work at them.

Check: Give one feature a request deadline that's passed to every step, and show that a slow step still leaves time for the fallback. Then load-test the feature past its capacity, and show that excess requests are turned away quickly while accepted ones still meet their target.

### 9.7 Consistent outputs [3]

From the Reliability concern.

The same request can give different outputs, and models change over time. Why outputs vary is covered in 1.8 Reliability in foundation models, deciding where variety is wanted in 4.2 Designing for wrong answers, and measuring consistency in 3.14 Reliability in evaluation. This section covers making outputs dependable.

- [3] Validating and repairing. Check every output against its schema and rules before anything uses it. When it fails, retry with the error included, so the model can fix it, up to a set limit, and then fall back. Libraries such as Instructor and Pydantic AI build this loop in.
- [3] Reducing variation. Use structured output with strict schemas, fixed instructions and examples, a stable order of context (see 2.2 Context engineering) and a pinned model snapshot. Lower the temperature where the provider allows it, though some reasoning models don't (see 1.1 Working with a model).
- [3] Same question, same answer. Where answers must not change from one ask to the next, such as prices, policies or account details, compute them in code, look them up from one source of truth, or cache them (see 8.7 Caching), instead of generating them fresh each time.
- [2] Voting. For classification and extraction where accuracy matters more than cost, run the call several times and take the most common answer. It smooths out random variation, at a multiple of the cost.
- [2] Determinism. Some APIs accept a seed for more repeatable outputs, but few providers guarantee identical results. On your own servers, engines can be made fully deterministic with specific settings, at some cost in speed.

Check: Run one feature's evaluation set five times and measure how often the runs agree. Add validation with a repair loop, and one change that reduces variation, and show that agreement rises and invalid outputs fall to nearly zero.

### 9.8 Changes without outages [3]

From the Reliability concern.

Many AI incidents start with a change: a prompt edit, a model upgrade, a new tool or a rebuilt index. Gradual rollouts are covered in 8.6 Routing, rolling out prompts in 2.1 Prompts, and changing tools in 5.14 Reliability in tools and protocols. This section covers what they share.

- [3] Every change is a release. Prompts, model versions, routing rules, tool definitions, guardrail settings and indexes change behavior as much as code does. Version them, test them on your evaluation set (see 3.7 Evaluation in development), and release them through the same process as code, with a way back.
- [3] Gradual, with a fast way back. Release changes to a small share of traffic first, watch errors, latency, cost and quality (see 9.3 Metrics and dashboards), and widen the release step by step. Practice rolling back, so it takes minutes and nobody has to work out how during an incident.
- [3] Changes you don't control. Providers change models behind aliases, adjust rate limits and safety filters, and retire models on a schedule (see 1.4 Frontier proprietary models), and tools and MCP servers change too. Pin versions where you can, track retirement dates, and send fixed test requests on a schedule to spot changes early (see 9.10 Alerts).
- [2] Work in progress. Agent tasks that are running during a release started under the old version (see 7.5 State and long-running tasks), and a new index should be built alongside the old one before you switch (see 6.18 Reliability in context, memory and retrieval). Plan for both.

Check: Release a prompt change to 5% of traffic, compare it with the current version, and roll it back through configuration, timing the rollback. List every model you depend on, with its pinned version and retirement date.

### 9.9 Testing for failure [3]

From the Reliability concern.

Fallbacks and error handling that never run tend to break without anyone noticing. This section covers making failures happen on purpose, before they happen on their own.

- [3] Injecting faults. In tests and staging, make each dependency fail in each way it can: timeouts, rate-limit and server errors, streams that break partway, malformed or cut-off output, slow tools and search that's down. Mock servers and proxies that inject faults let you do this without waiting for a real outage. Check that each failure leads to the right retry, fallback or message.
- [3] Testing at load. Test at expected peak traffic and beyond, measured from the client, including how rate limits, queues and cold starts behave (see 8.12 Cloud capacity). Many failures appear only under load.
- [3] Rehearsing outages. Run a drill in which your main provider goes down, in staging or carefully in production. Measure how long it takes to notice, to fail over and to recover, and fix whatever was slow.
- [3] Exercising fallbacks all the time. Send a small share of real traffic through each fallback path at all times, or test each one on a schedule. A fallback that runs every day is one you know works.
- [2] Failures in production. Some teams inject failures into production on purpose, within tight limits, to prove the system copes. This is called chaos engineering. Know the practice, and start in staging.

Check: Build a fault-injection test that makes each dependency of one feature fail in each way, and show the right behavior for each. Then run a drill in which the main provider goes down, and record how long it takes to notice, fail over and recover.

### 9.10 Alerts [3]

From the Observability concern. Learn this before you go on call for an AI feature.

Alerts tell someone to act. Setting targets for uptime and latency, called service level objectives (SLOs), is covered in 9.2 Service levels, and alerts on quality in 11.1 Evaluation in production. This section covers measuring service levels and alerting well.

- [3] Measuring service levels. Measure service level indicators, such as the share of requests that succeed within a time limit, the way users experience them. For AI features, a request that ended in a fallback, was cut off at the length limit or took 40 seconds may count as failed, even though it raised no error.
- [3] Alerting on symptoms. Alert on what users feel, such as errors, slowness, falling quality and cost spikes, rather than on every possible cause. Give each alert an owner and a short runbook: what to check first and what to do. Alerts that often fire without needing action teach people to ignore all alerts.
- [3] Alerts for AI features. Common ones are cost or tokens per request jumping (a looping agent, or a prompt change that doubled the context), more outputs cut off at the length limit, more refusals, tool errors or fallbacks, a falling prompt cache hit rate (often a prompt change that broke caching, see 2.4 KV-cache management), and usage nearing rate limits (see 8.4 Rate limits and quotas).
- [3] Watching providers. A provider's outage, slowdown or model change is your incident too. Measure each provider's and model's errors and latency from your side instead of relying on status pages, and send small test requests on a schedule, to catch problems before users do.
- [2] Alerting on burn rate. An SLO allows some failures, called the error budget. Instead of alerting on every brief spike, alert when failures are using up that budget too fast. This catches real problems quickly and ignores noise.
- [2] Unusual changes. For metrics with no fixed normal level, such as daily cost or the mix of topics users ask about, alert on unusual changes compared with history. Tune these alerts, or they'll fire constantly.

Check: For one feature, define two service level indicators measured from the user's side, and set up alerts on them and on cost per request, each with a runbook. Trigger each alert in a test environment and follow its runbook.

### 9.11 Incidents and recovery [3]

From the Reliability concern. Learn this before your first on-call rotation for an AI feature.

Investigating what went wrong is covered in 2.8 Debugging from traces. This section covers keeping the product running during an incident and recovering afterwards.

- [3] Runbooks for known failures. Write short runbooks for the failures you expect, such as a provider outage, exhausted rate limits, a sudden change in model behavior, a broken index and runaway costs. Each says how you'll know, what to do first and whom to contact, including the provider's support.
- [3] Manual switches. Be able to force a fallback, switch provider, turn off a feature or raise a limit through configuration, without a deploy (see 8.1 The gateway). In an incident, a switch that takes an hour to change is no switch.
- [3] Recovering carefully. When a provider comes back, return traffic to it gradually, and drain queues at a pace it can handle. Resume paused agent tasks from their checkpoints (see 7.5 State and long-running tasks), and check that retried actions didn't happen twice.
- [3] Learning from incidents. Afterwards, write a short review that blames no one: what happened, why the defenses didn't stop it and what will change. Track the follow-up work until it's done, and add the failing cases to your tests.
- [2] Keeping users informed. Tell users and customers about incidents and reduced service through a status page or messages in the product, especially while the product is running with less.

Check: Write runbooks for a provider outage and for exhausted rate limits, and run one as a drill using only the runbook and the manual switches. Afterwards, write a short review of the drill and fix one thing it revealed.

### 9.12 Observability tools [2, one tool 3]

From the Observability concern. Learn one well, then the pipeline behind it.

Observability tools change quickly. Learn what they do first, then learn one of them well.

- [3] LLM observability platforms. Tools such as Langfuse, LangSmith, Arize Phoenix, Braintrust, Helicone and W&B Weave show model calls, traces, sessions, costs and feedback, and connect them to evaluation and datasets. Learn one well, including searching traces and exporting them.
- [2] General observability tools. Your company probably already runs a general tool, such as Datadog, Grafana, Honeycomb or New Relic, and many of these now have AI features. Where you can, send AI traces to the same place as the rest of your telemetry, so you can follow a request from the browser through the AI steps to the database.
- [2] How data gets in. You can trace with an SDK inside your code, or log every model call at a proxy or gateway. A proxy needs no code changes, but it sees only model calls, not retrieval, tools or your own logic. Many teams use both.
- [2] The pipeline behind the tools. Telemetry passes from your code through collectors to storage and query engines, and it can be dropped, delayed or sampled at each stage. Platform teams usually run this pipeline. Know where your data flows, so you can tell when some of it is missing.
- [2] Choosing. Weigh OpenTelemetry support, whether you can host the tool yourself (a hosted tool receives every prompt and output), cost at your volume, search over full content, evaluation features, retention and access controls.

Check: Instrument one feature with OpenTelemetry and send its traces to an LLM observability tool. Switch to a second tool by changing only the export configuration, and list what each shows that the other doesn't.

### 9.13 Cost and latency in reliability [3]

From the Reliability concern.

- [3] What reliability costs. Retries, fallbacks, hedged requests, voting and reserved capacity all cost extra, and retries and fallbacks add latency. Measure what each adds, and spend where the feature's target needs it.

### 9.14 Security in reliability [3]

From the Reliability concern.

- [3] Stopping or carrying on. Reliability favors carrying on when a check fails, while security and safety favor stopping. Decide for each check: when the checks on risky actions or high-risk output fail, stop the request, and for low-risk features, carry on and log the failure (see 10.7 Reliability in security and 10.20 Reliability in governance and safety).

### 9.15 Governance and safety in reliability [3]

From the Reliability concern.

- [3] Fallbacks follow the same rules. A fallback provider must be approved for the same data and regions as the main one (see 8.11 Governance and safety in model gateway and routing), and degraded modes must still keep audit records and honor deletion. A failover that sends customer data to an unapproved region trades an outage for a compliance breach.

Measuring service levels, alerting on them and investigating incidents are covered in 9.10 Alerts and 2.8 Debugging from traces, and 9.19 Reliability in observability covers keeping telemetry from breaking the product. Measuring consistency and testing the quality of fallback paths are covered in 3.14 Reliability in evaluation and 8.8 Evaluation in model gateway and routing.

Data quality meets reliability in the sources that retrieval depends on: an index that falls out of sync or a source that stops updating breaks answers without raising an error (see 6.4 Preparing documents and 6.14 Data quality in context, memory and retrieval).

### 9.16 Security in observability [3]

From the Observability concern.

- [3] Secrets in telemetry. Prompts, tool arguments and tool results can contain API keys, passwords and tokens. Remove them before they're recorded, because traces get copied to vendors, exported and kept for months (see 5.15 Observability in tools and protocols).
- [3] Who can read traces. Traces hold user conversations and customer data. Limit who can read full content, log who viewed it, and keep each customer's data separate in shared tools.
- [3] Hostile content in trace viewers. Recorded inputs and outputs can contain attacks, such as scripts, malicious links or injected instructions. Trace viewers should show them as plain text, and models that summarize traces should treat them as data (see 4.8 Security in product and UX).
- [2] Spotting attacks. Traces and metrics reveal abuse, such as one key sending far more requests than usual, repeated jailbreak attempts or sudden jumps in cost. Defenses are covered in 10.3 Jailbreaks and abuse.

### 9.17 Governance and safety in observability [3]

From the Observability concern.

- [3] Personal data in traces. Traces copy user data out of your product, often into another company's tool. Mask personal data you don't need when you record it, limit how long you keep it, and make deletion requests reach traces and exported copies too (see 11.5 Governance and safety in the data layer).
- [3] Where traces go. Your observability vendor receives everything you send it. Check where it stores data, for how long and who can see it, as you would for a model provider. A zero-data-retention agreement with a model provider means little if the same prompts sit in your trace store for a year.
- [3] Traces are not audit logs. Traces are for debugging, so they may be sampled, trimmed and deleted early. Audit logs are for accountability, so they must be complete, unchangeable and kept longer (see 5.15 Observability in tools and protocols). Audit trails are covered in 10.14 Audit trails and documentation.
- [2] Looking without reading. Models can summarize and cluster conversations with personal details removed, so you can see how the product is used without people reading raw conversations.

### 9.18 Cost and latency in observability [3]

From the Observability concern.

- [3] What observability costs. Storing full prompts and outputs for every call adds up fast, and some tools charge by the span or by the gigabyte. Control it with sampling, shorter retention for full content, and cheaper storage for older data.
- [3] Off the request path. Send telemetry in the background, in batches, so recording never slows down a user's request. Measure what tracing adds to latency.

Measuring cost and latency is covered in 9.3 Metrics and dashboards.

### 9.19 Reliability in observability [2–3]

From the Observability concern.

- [3] Recording must not break the product. If the tracing tool is down or slow, requests should carry on without it. Buffer telemetry, and drop it rather than block or fail a request.
- [2] Seeing during outages. Your observability has to keep working when a provider or region fails, because that's when you need it most. Check that you could watch your fallbacks work (see 8.3 Fallbacks and retries) during a provider outage.

Evaluation uses traces as its raw material, and its scores become fields on each trace. Both are covered in 3.12 Observability in evaluation.

Data quality in observability means complete records: a trace with missing fields or dropped spans can mislead more than no trace at all. Turning traces into datasets is covered in 11.2 Product feedback.

### Key reading for Stage 9

- Google, *Site Reliability Engineering* (2016), especially chapter 6 on monitoring distributed systems and the chapters on service level objectives, handling overload and addressing cascading failures, and *The Site Reliability Workbook* (2018), on implementing SLOs, with chapter 5 on alerting on SLOs.
- Alex Hidalgo, *Implementing Service Level Objectives* (O'Reilly, 2020).
- Michael Nygard, *Release It!* (2nd edition, Pragmatic Bookshelf, 2018), on timeouts, bulkheads, circuit breakers and other patterns for stable systems.
- The Amazon Builders' Library, especially "Avoiding fallback in distributed systems" and "Using load shedding to avoid overload".
- Casey Rosenthal and Nora Jones, *Chaos Engineering: System Resiliency in Practice* (O'Reilly, 2020).
- Horace He and Thinking Machines Lab, "Defeating Nondeterminism in LLM Inference" (2025), on why the same request can give different outputs.
- Your model providers' status pages and their documentation on errors, rate limits and model retirement.
- The documentation for one library that validates and repairs model output, such as Instructor or Pydantic AI.
- Rob Ewaschuk, "My Philosophy on Alerting" (2013), written from his time as a site reliability engineer at Google.
- Tamkin et al., "Clio: Privacy-Preserving Insights into Real-World AI Use" (Anthropic, 2024), on finding patterns across many conversations without people reading them.
- The documentation for one LLM observability tool, such as Langfuse or Arize Phoenix.

## Stage 10. Safe to launch

Before a public launch, the product must hold up against attackers and behave acceptably when nobody is attacking it. This stage finishes the security sections that Stage 5 started. It then covers content policy and guardrails, safety testing, privacy, oversight, audit trails, fairness, regulation and intellectual property.

| Section | Level | Why this level | From |
|---|---|---|---|
| 10.1 Preventing data theft | 3 (some topics 4) | Attackers can get the model to hand data over | Security |
| 10.2 Secrets | 3 | AI systems copy text, and secrets travel with it | Security |
| 10.3 Jailbreaks and abuse | 3 | Every model can be talked out of its rules | Security |
| 10.4 Security testing | 3 | A defense nobody has attacked is untested | Security |
| 10.5 Responding to attacks | 3 | Some attacks will get through | Security |
| 10.6 Governance and safety in security | 2–3 | Laws treat a leak through AI like any other breach | Security |
| 10.7 Reliability in security | 2–3 | Security checks must fail safe | Security |
| 10.8 Cost and latency in security | 3 | Every check adds cost and delay | Security |
| 10.9 Governance and safety foundations | 3 | Acceptable is set by laws, contracts, policies and the people affected | Governance and safety |
| 10.10 Content policy and guardrails | 3 (some topics 4) | Every feature needs written rules and checks that enforce them | Governance and safety |
| 10.11 Safety and adversarial testing | 3 | Harms and attacks need test cases of their own | Evaluation |
| 10.12 Privacy and personal data | 3 (some topics 4) | AI systems copy personal data into many places | Governance and safety |
| 10.13 Oversight and accountability | 3 | Someone must own what each feature does | Governance and safety |
| 10.14 Audit trails and documentation | 3 | You must be able to show what happened and why | Governance and safety |
| 10.15 Fairness | 3 | AI can work worse for some groups without anyone noticing | Governance and safety |
| 10.16 Regulation | 2 (some topics 3) | Laws change fast, and legal decides how they apply | Governance and safety |
| 10.17 Intellectual property | 2 | Rights to inputs and outputs are partly unsettled | Governance and safety |
| 10.18 Security in governance and safety | 3 | Most privacy breaches start as security failures | Governance and safety |
| 10.19 Observability in governance and safety | 3 | Harm often shows up in production before anyone reports it | Governance and safety |
| 10.20 Reliability in governance and safety | 2–3 | Safety checks must fail safe | Governance and safety |
| 10.21 Cost and latency in governance and safety | 3 | Every guardrail adds cost and delay | Governance and safety |

### 10.1 Preventing data theft [3, some topics 4]

From the Security concern.

Attackers want the data your product can reach: users' conversations and files, customers' documents and your own data. With AI, they often don't need to break in, because they can get the model to hand the data over. Leaks through rendered output are covered in 4.8 Security in product and UX and 6.10 Security in context, memory and retrieval, leaks through tool arguments in 5.10 Security in tools and protocols, and leaks from fine-tuned models in 11.6 Security in the data layer. This section covers closing every way out.

- [4] Mapping every way out. List every channel data can leave through: rendered output that loads images or links, tools that send email, post messages or fetch URLs, code sandboxes and browsers with network access, webhooks, shared links, and exported logs and traces. For each, decide in code what it may reach, such as a list of allowed domains, no network access by default, or a person's approval before anything goes outside. A channel you didn't list is one nobody is guarding.
- [3] Scanning what goes out. Check outputs and outgoing tool arguments for secrets, personal data and confidential material before they leave, and block or mask what shouldn't go. Pattern matching catches keys and card numbers, and classifiers catch more. Neither catches data the model has summarized or reworded, so treat scanning as one defense among several.
- [3] Slow leaks. An attacker doesn't need one big transfer. An agent can be made to leak a little at a time, such as a few characters in each search query or web address it requests. Limits on volume help, but the reliable fix is keeping sensitive data away from agents that can reach outside (see the dangerous combination in 5.10 Security in tools and protocols).
- [2] Canary data. Plant fake records, such as a fake customer or a fake API key, in sensitive sources, and alert if they ever appear in outputs, tool calls or outside systems. They show that a leak happened, and where it came from.

Check: Map every way data can leave one agent. Use injected instructions to try to send private data out through each channel, including an image link in the output, and show that each attempt is blocked. Then plant a canary record and show that an attempt to leak it raises an alert.

### 10.2 Secrets [3]

From the Security concern.

Secrets are API keys, passwords, tokens and signing keys. Storing and rotating provider keys is covered in 8.2 API keys and access, keeping tool credentials out of the context in 5.7 Permissions, and keeping secrets out of system prompts in 1.10 Security in foundation models. This section covers the other ways secrets leak from AI systems.

- [3] How secrets travel. AI systems copy text into many places, and a secret travels with the text: from a document into a search index, from a tool result into a trace, from a log into a training set (see 11.6 Security in the data layer), or from a sandbox's environment variables into code the model wrote. Anything in the context can also come out in the output, or be taken by an injected instruction.
- [3] Secrets users share. Users paste keys, passwords and tokens into chats and uploads. Detect and mask them before they're stored, indexed, remembered or logged.
- [3] Scanning beyond code. Scan for secrets wherever text collects, not only in code: documents before indexing, tool results, memories, logs and traces (see 9.16 Security in observability). Mask what you find, and rotate any real secret that was exposed.
- [3] Developer machines. MCP server configurations, agent tools and local scripts often keep tokens in plain files, where any local program or coding agent can read them. Keep them in the operating system's credential store or a secrets manager, and give each token the narrowest scope that works.

Check: Plant a fake key in a document, a chat message and a tool result. Show that each is caught and masked before it reaches an index, a memory, a log, a trace or the model's output.

### 10.3 Jailbreaks and abuse [3]

From the Security concern. Learn this before a public launch.

A jailbreak is a user talking a model into breaking its rules. Abuse is using your product for something it wasn't meant for. How models respond to jailbreaks is covered in 1.10 Security in foundation models, and deciding which content to block in 10.10 Content policy and guardrails. This section covers defending the product against both.

- [3] Common techniques. Role play and made-up scenarios, long prompts full of fake examples of the model complying (many-shot jailbreaking), requests hidden in encodings or less common languages, and conversations that escalate a little at a time. New techniques appear constantly, so learn the families rather than a list.
- [3] Refusal isn't a control. Every model can be jailbroken with enough effort. For anything that matters, such as another user's data, money or dangerous content, enforce the limit in code or with a separate check that the conversation can't talk its way past.
- [3] Input and output classifiers. Separate classifiers can screen requests and responses for jailbreak attempts and prohibited content. Options include provider moderation endpoints, open models such as Llama Guard, and small classifiers you train yourself (see 12.3 Supervised fine-tuning). Checking the output catches jailbreaks that the input check missed.
- [3] Abuse of the product. Some users treat your product as free general-purpose AI, generate spam with it, script many accounts to get around limits, or send expensive requests on purpose to run up your bill. Limits per user and per account (see 8.4 Rate limits and quotas), checks at sign-up, and watching for unusual use (see 9.16 Security in observability) keep the cost and damage down.
- [2] Copying your product. Competitors can query your product at scale to extract its prompts, or to collect outputs to train their own models. Rate limits and terms of service are the practical defenses.

Check: Run a jailbreak session against one feature with a colleague, using at least three families of technique. Show which attempts the model resisted on its own, which your classifiers caught, and that none reached data or actions protected in code.

### 10.4 Security testing [3]

From the Security concern. Do this before launch and after every major change.

Red-teaming as part of evaluation, and testing that the product refuses harmful requests, are covered in 10.11 Safety and adversarial testing. This section covers testing security controls.

- [3] Testing the controls, not the model. A security test should pass because a control in code stopped the attack, such as a permission check, a blocked domain or an approval step, not because the model refused. To check, rerun your attack cases with a prompt that tells the model to follow any instruction it reads. If an attack then succeeds, the model was the only thing stopping it.
- [3] Attacks for every entry point. Build test cases with injected instructions for every way outside text reaches the model, including attempts to reach other users' data and to send data out. Run them before every release, like any regression set (see 3.7 Evaluation in development).
- [3] Attackers adapt. Published defenses often fail once an attacker designs attacks against them. Don't rely only on lists of known attacks: have people, and models prompted to attack, craft attacks aimed at your own product.
- [3] Benchmarks and tools. Benchmarks such as AgentDojo measure how often agents fall for injection, which helps when you compare models and defenses. Tools such as garak, PyRIT and promptfoo run common attacks automatically.
- [2] Specialists. Before launching features that take real actions or reach sensitive data, have security specialists review the design and test it. Some companies also pay outside researchers who report problems (a bug bounty).

Check: Build a security test suite for one agent, with injection cases for every entry point, including cases that try to reach other users' data and to send data out. Run it with a prompt that tells the model to obey every instruction, and show that each attack is still stopped by a control in code.

### 10.5 Responding to attacks [3]

From the Security concern.

Some attacks will get through. This section covers noticing them and limiting the damage. Spotting attacks in traces and metrics is covered in 9.16 Security in observability.

- [3] Logging security events. Record every blocked action, permission denial, classifier flag, approval and rejection, with its trace ID, user and agent. These records are what you investigate after an attack, and patterns in them reveal attacks in progress.
- [3] Kill switches. Be able to turn off a tool, an MCP server, an agent or a whole AI feature quickly, without a deploy, and test that it works (see 7.14 Governance and safety in agents and orchestration).
- [3] Cleaning up. After an attack, revoke the keys and tokens involved (see 8.2 API keys and access), remove planted content from indexes, memories and caches (see 6.15 Governance and safety in context, memory and retrieval), and use traces and audit logs to find what the attacker saw and did.
- [3] Learning from it. Turn each attack into a test case (see 10.4 Security testing), and fix the control that failed, not just the prompt. A data leak through an AI feature is a breach like any other, with the same duties to report it (see 10.6 Governance and safety in security).

Check: Run a drill in which an injected instruction in a shared document has made an agent leak data. Use your security events and traces to find every affected request, turn off the feature, revoke the credentials involved, remove the document, and add the attack to your test suite.

### 10.6 Governance and safety in security [2–3]

From the Security concern.

- [3] Where security ends and safety begins. Security defends against attackers. Governance and safety makes the system behave acceptably when nobody is attacking. Many controls serve both, such as classifiers, approvals and audit logs, so plan them together.
- [2] Rules that require security. The EU AI Act requires high-risk systems to resist attempts to manipulate them, including data poisoning and inputs designed to cause mistakes. Data protection laws such as GDPR treat a leak through an AI feature like any other breach, with deadlines for reporting it. Bring in legal and your security team.
- [2] Customers' security reviews. Enterprise customers increasingly ask how your AI features handle prompt injection, which providers see their data, whether it's used for training, and how they're kept apart from other customers. Keep written answers current.

### 10.7 Reliability in security [2–3]

From the Security concern.

- [3] Failing safe. When a security check fails or times out, block actions that are risky or can't be undone, rather than let them through. For low-risk features, you may choose to carry on and log the failure.
- [2] Attacks on availability. Floods of long or expensive requests can slow the product down for everyone. Limits per user and per key (see 8.4 Rate limits and quotas) keep one attacker from affecting others.

### 10.8 Cost and latency in security [3]

From the Security concern.

- [3] Paying for checks. Classifiers, second model calls and approval steps add cost and delay. Match checks to risk: run cheap checks everywhere and expensive ones where the stakes are high, and run checks alongside the main call where you can (see 7.2 Workflow patterns).

Protecting traces, and spotting attacks in traces and metrics, are covered in 9.16 Security in observability. Red-teaming, and judges that injected text can fool, are covered in 10.11 Safety and adversarial testing and 3.15 Security in evaluation.

Data quality meets security where data is poisoned on purpose: in training data (see 11.6 Security in the data layer) and in indexes and memories (see 6.10 Security in context, memory and retrieval).

### About Governance and safety

Security asks what happens when someone attacks the system. Governance and safety asks whether it behaves acceptably when nobody does, and whether you can show that it does. AI features can harm people with no attacker involved: a confident wrong answer about a medication, a hiring screen that favors one group, a memory that keeps what a user asked to forget, or an agent that acts without approval.

What counts as acceptable comes from laws, contracts, your company's policies and what the people affected can reasonably expect. Legal, privacy and policy teams interpret those. You build the controls that make the product follow them, and you're often the first to see where it doesn't.

For every AI feature, governance and safety answers four questions:

1. What must it never do, and what must it handle with care? (10.9 Governance and safety foundations, 10.10 Content policy and guardrails, 10.15 Fairness)
2. What happens to people's data? (10.12 Privacy and personal data)
3. Which laws apply, and whose rights are involved? (10.16 Regulation, 10.17 Intellectual property)
4. Who is responsible, and can you show what happened? (10.13 Oversight and accountability, 10.14 Audit trails and documentation)

Most concepts in this concern are level 3, and two things are level 4:
- writing each feature's content policy precisely enough to enforce and test
- mapping everywhere personal data goes, since only engineers know every copy

Learn regulation and intellectual property to level 2: know when a law or a right applies and what it asks of the product, and let legal decide how it applies to you.

The layers' own sections cover governance and safety as it applies to each layer, such as human approval for agent actions in Layer 3 and deleting data from indexes and memories in Layer 4. The Governance and safety sections cover what they share, and 14.4 Governance and safety at each layer maps where the rest is covered.

### 10.9 Governance and safety foundations [3]

From the Governance and safety concern. Focus on matching effort to risk.

These ideas come up in every other section, so learn them first.

- [3] Where "acceptable" comes from. It comes from laws and regulations, from contracts (your terms of service, your customers' agreements and your model providers' usage policies), from your company's own policies, and from what the people your product affects can reasonably expect. When these conflict or are unclear, bring in legal, privacy and policy colleagues. They decide what's acceptable, and you make the product follow it.
- [3] Naming the harms. For each feature, list what could go wrong when no one is attacking it: harmful or dangerous content, wrong information that people act on in health, law or money, unfair treatment of some groups, privacy violations, infringing others' rights, manipulation or unhealthy dependence, and agents taking actions nobody approved. Name who could be harmed: users, people mentioned in the data, your customers and the public.
- [3] Effort matched to risk. A tool that drafts internal meeting notes needs lighter controls than one that screens job applicants or answers medical questions. Rate each feature by how badly and how many people a failure could hurt, whether the harm can be undone, and whether its output feeds a decision about a person. The EU AI Act sorts systems into risk tiers in a similar way (see 10.16 Regulation).
- [3] Showing, not just doing. Governance means being able to show that the system behaves acceptably, through written policies, test results, records of who approved what and logs of what the system did. Regulators, auditors and customers ask for evidence, not assurances.
- [2] Who does what. Legal interprets laws and contracts, privacy teams own data protection, security teams defend against attacks (see 5.1 Security foundations), and trust and safety or policy teams write content rules. You build the controls, explain how the system actually works, and raise problems early.

Check: For one AI feature, list who could be harmed and how, when no one is attacking it, and rate the feature's risk. For each harm, name the policy that covers it, the control that enforces it and the evidence that shows the control works.

### 10.10 Content policy and guardrails [3, some topics 4]

From the Governance and safety concern.

A content policy says what a feature may and may not produce. Guardrails are the checks that enforce it. Testing guardrails is covered in 10.11 Safety and adversarial testing, and defending them against jailbreaks in 10.3 Jailbreaks and abuse.

- [4] A written policy for each feature. Write down what the feature must never produce, what it must handle with care, and what it should do even though it looks risky. For example, a pharmacy assistant should discuss drug doses, must point anyone in crisis to help, and must never give instructions for misuse. Make the policy specific enough that two reviewers would label an output the same way, because it's what your guardrails enforce and your tests check (see 10.11 Safety and adversarial testing).
- [3] Layers of guardrails. The model's safety training and the provider's filters come first (see 1.2 Capabilities and limits). Your system prompt sets the feature's rules. Separate checks on inputs and outputs catch what gets past both, and checks on actions stop what agents shouldn't do (see 5.5 Actions and side effects). No single layer is enough.
- [3] Guardrail tools. Options include provider moderation endpoints and safety settings, open classifiers such as Llama Guard, frameworks such as NeMo Guardrails and Guardrails AI, cloud services such as Amazon Bedrock Guardrails and Azure AI Content Safety, and small classifiers you train yourself (see 12.3 Supervised fine-tuning). Most let you choose which categories to block and how strictly.
- [3] Tuning for both kinds of mistake. A guardrail that blocks too little lets harm through. One that blocks too much refuses legitimate requests and drives users away. Measure both on labeled examples (see 3.5 Defining what to measure), and set each category's threshold by the harm it guards against.
- [3] When a guardrail fires. Tell the user plainly what the product can't help with, without accusing them, and offer an alternative or a person where it makes sense (see 4.2 Designing for wrong answers). Record the event, so you can review patterns and requests that were blocked by mistake.
- [3] Streaming and output checks. When output streams to the user, a harmful passage can appear before the output check finishes. For high-risk features, check the output in chunks as it streams, or hold it back until it passes.
- [3] Users at risk. Some users are in crisis, some are children, and some form attachments to the product. Decide how the product responds to them (see sensitive situations in 4.7 Governance and safety in product and UX), and design against flattery and dependence (see respecting users in the same section). Laws increasingly set rules here, especially for products that children may use or that act as companions.

Check: Write the content policy for one feature, with examples of what it must never produce, what it must handle with care and what it should help with anyway. Put an input and an output guardrail in place, and measure on 100 labeled examples how many harmful outputs they miss and how many legitimate requests they block. Change one threshold, and show that both numbers move as you expected.

### 10.11 Safety and adversarial testing [3]

From the Evaluation concern. Do this before launch.

Ordinary test sets check that the system does its job. Safety and adversarial tests check that it doesn't cause harm, even when someone is trying to make it. Building adversarial test sets is covered in 3.4 Evaluation: building evaluation datasets and 3.11 Synthetic data, and defenses in 5.8 Prompt injection, 10.3 Jailbreaks and abuse and 10.10 Content policy and guardrails. This section covers testing.

- [3] The harms your product could cause. List the harms that matter for your product, such as harmful or dangerous content, leaked personal data or system prompts, bad medical, legal or financial advice, unfair treatment of some users, and actions an agent shouldn't take. Build cases for each and run them before every release, like any regression set. Safety cases for tools are covered in 5.13 Evaluation in tools and protocols.
- [3] Refusing too much. Test that the product still helps with reasonable requests that look risky, such as a nurse asking about medication doses. Refusing reasonable requests is a failure too, and tighter safety rules often cause it.
- [3] Red-teaming. People, and models prompted to attack, deliberately try to make the product fail, with jailbreaks, prompt injection through documents and tool results, and misuse of features. Run a round before launch and after major changes, and turn every successful attack into a regression case. Tools such as garak, PyRIT and promptfoo automate common attacks.
- [3] Fairness. Compare results across user groups, languages and dialects. Test whether changing a name, gender or dialect in an input changes the outcome, using the behavioral tests described in 3.6 Code-based checks. Bias in models is covered in 1.13 Governance and safety in foundation models.
- [3] After a model changes. A new model or a fine-tune can change safety behavior even when nothing else changes. Run the safety set after every model change (see 12.11 Governance and safety in model adaptation).
- [1] Dangerous-capability evaluations. Before release, labs test frontier models for dangerous capabilities, such as helping with weapons or cyberattacks, and publish the results in system cards. Know what they cover. Application teams don't run them.

Check: Build a safety set for one feature that covers its main harms, plus 20 reasonable requests that look risky. Run a red-team session with two colleagues, add every successful attack to the set, and show the set running in your release checks.

### 10.12 Privacy and personal data [3, some topics 4]

From the Governance and safety concern. Draw the data map before your first launch.

Personal data is any information about a person who can be identified. AI systems copy it into many places: prompts, provider logs, traces, search indexes, memories, caches, datasets and fine-tuned models. Personal data in datasets is covered in 11.5 Governance and safety in the data layer, in indexes and memories in 6.15 Governance and safety in context, memory and retrieval, in traces in 9.17 Governance and safety in observability, and users' controls over their data in 4.7 Governance and safety in product and UX. This section covers the whole picture.

- [4] A map of where personal data goes. For each feature, list every place personal data is sent or stored: the model provider, the gateway's logs (see 8.9 Observability in model gateway and routing), traces, indexes, memories, caches, evaluation sets, training data and every vendor in between. Note how long each keeps it, who can see it and which country it's in. Deletion, retention, consent and privacy reviews all depend on this map, and only engineers know every copy.
- [3] Only what the task needs. Send the model only the personal data the task needs, mask or remove the rest before it leaves (see 8.10 Security in model gateway and routing), and keep it only as long as needed. Data you never collect can't leak or be misused.
- [3] Purpose and consent. Use personal data only for the purposes users were told about and, where the law requires it, agreed to. Using conversations to improve the product or train models is a separate purpose that needs its own basis (see 11.2 Product feedback and 11.5 Governance and safety in the data layer).
- [3] People's rights. Privacy laws such as GDPR give people rights to see, correct and delete their data, and to object to some uses. A request must reach every copy on your map, including derived ones such as summaries and extracted facts. Data inside a fine-tuned model can't be reliably removed without retraining, so keep personal data out of training sets.
- [3] Special categories. Health, biometric, financial and children's data, and data about beliefs or sexuality, carry stricter rules, such as HIPAA for health data in the US. Check that every provider that handles such data has agreed in its contract to the rules that apply.
- [2] Privacy reviews. A feature that processes personal data in a new way often needs a formal assessment, such as a data protection impact assessment under GDPR. The privacy team runs it, and your data map is its starting point.
- [2] What models can infer. Models can guess sensitive facts, such as a health condition or political views, from ordinary data, and memory features can bring those guesses up later. Treat inferred sensitive facts like collected ones (see 6.15 Governance and safety in context, memory and retrieval).

Check: Draw the personal-data map for one feature, with retention, access and location for each place. Then delete one test user's data, and show that it's gone from every place on the map, including traces, indexes, memories and evaluation sets.

### 10.13 Oversight and accountability [3]

From the Governance and safety concern.

Someone must be responsible for what each AI feature does, and people must be able to see and correct what agents do. Where people step into agent workflows is covered in 7.6 Handoffs to people, which actions need approval in 5.5 Actions and side effects, and the screens people use in 7.7 Users in control of agents. This section covers the governance around them.

- [3] An owner for every feature. Name one person responsible for each AI feature's behavior: its policy, its guardrails, its evaluation results and its incidents. Without an owner, problems that cross teams, such as a provider change that weakens a guardrail, belong to nobody.
- [3] Oversight that works. A person reviewing AI output helps only if they have the time, the information, the authority and the skill to disagree. Reviewers who approve nearly everything give false comfort (see review fatigue in 7.6 Handoffs to people). Measure how often they change or reject what the AI proposes.
- [3] Contesting decisions. When AI contributes to a decision about a person, tell them, explain the main reasons in plain words, and give them a way to have a person review it. Some laws require this.
- [3] Reviews scaled to risk. Before a feature launches, review its purpose, harms, data, evaluation results, guardrails, oversight and disclosures, with more depth and more sign-offs as the risk rises. For a low-risk feature, a one-page review is enough.
- [3] Approving what you build on. Review new model providers, tools, MCP servers and vendors for their data terms, security, licenses and regions before anyone uses them, and enforce the approved lists in code (see 5.17 Governance and safety in tools and protocols and 8.11 Governance and safety in model gateway and routing).
- [3] Changes that need review. Decide which changes need a fresh review, such as a new model, a new tool that can act, a new kind of data or user, or a change to a guardrail. A model upgrade can change behavior without any change to your code (see 1.4 Frontier proprietary models).
- [3] Safety incidents. Define what counts as an incident, such as harmful output reaching users, a privacy leak, an unfair decision or an agent acting without approval. Give users a way to report problems, decide who responds and who must be told, including regulators and customers where laws or contracts require it, and turn each incident into test cases (see 10.11 Safety and adversarial testing).
- [2] Staff use of AI tools. Employees use AI tools with company data whether or not the tools are approved. Publish which tools staff may use with which data, and make the approved ones easy to get.

Check: Name the owner of one feature and run a launch review scaled to its risk. Define what counts as an incident for it, then run a drill: report a harmful output, follow it through response and notification, and add it to the evaluation set.

### 10.14 Audit trails and documentation [3]

From the Governance and safety concern.

Audit trails record what the system did. Documentation records what the system is and how it was tested. Audit logs of agent actions are covered in 5.15 Observability in tools and protocols, records of agent decisions in 7.14 Governance and safety in agents and orchestration, and why traces aren't audit logs in 9.17 Governance and safety in observability. This section covers what they have in common.

- [3] What an audit trail shows. For every consequential output or action, it shows who asked, what the system did and why (the inputs, sources, model and prompt versions, and tool calls), what changed and who approved it. Unlike traces, audit trails must be complete, protected from changes and kept for a set period.
- [3] Records about the system. Keep records of decisions about the system itself: which models, tools and prompts were approved and by whom, evaluation results at each release (see 3.16 Governance and safety in evaluation), launch reviews, policy versions and incidents. Months later, someone will ask why a version shipped.
- [3] A card for each feature. Keep a short document for each AI feature that covers its purpose, users, models and providers, data, evaluation results, known limits, guardrails, human oversight and owner. Update it with each release. Cards for adapted models and datasets are covered in 12.11 Governance and safety in model adaptation and 11.4 Data quality in the data layer.
- [3] Where things came from. Record the source and terms of every model, dataset, document collection and tool you use (see lineage in 11.4 Data quality in the data layer and the AI bill of materials in 5.11 Supply chain). When a source turns out to be unlicensed, biased or compromised, the record shows what it affected.
- [2] Logging the law requires. The EU AI Act requires high-risk systems to log events automatically, and companies using them to keep those logs for at least six months. Other laws and contracts set their own retention rules.
- [2] Keeping records versus privacy. Audit trails favor keeping records for a long time, and privacy favors deleting them early. Resolve this by type of data: keep the record of what was done and by whom for longer, and keep personal content for as short a time as you can, or replace identities with pseudonyms.

Check: For one consequential action or output, show its audit record: who asked, what the system saw and did, which versions were used and who approved it. Then show the feature's card, and the evaluation results from the release that produced the output.

### 10.15 Fairness [3]

From the Governance and safety concern. Learn this before any feature that affects decisions about people.

AI features can work worse for some groups, or treat them unfairly. How models reflect bias is covered in 1.13 Governance and safety in foundation models, gaps in datasets in 11.5 Governance and safety in the data layer, and testing for fairness in 10.11 Safety and adversarial testing. This section covers deciding what fairness means for a feature and acting on it.

- [3] Kinds of unfairness. A feature can make worse decisions about some groups, such as when ranking job applicants or flagging fraud. It can work less well for some users, such as speakers of other languages or dialects, or people with accents in voice products. And it can describe groups through stereotypes in what it writes or draws.
- [3] Choosing what to check. Decide with people who know the feature's users which groups and kinds of unfairness matter. Cover characteristics that the law protects, such as gender, ethnicity, age and disability, and the languages and regions you serve.
- [3] Decisions about people. When AI helps decide something about a person, such as hiring, lending, insurance, housing, education or access to services, the stakes and the legal duties rise sharply. Keep a person responsible for each decision, check outcomes by group, and expect the rules in 10.16 Regulation to apply.
- [3] Fixing what you find. Change the prompt or examples, add data for groups the feature serves poorly, send some cases to a stronger model or to people, or narrow what the feature does. Then measure again. Removing a protected attribute from the input rarely removes the bias, because other details stand in for it.
- [2] Laws on bias. Anti-discrimination law applies to decisions made with AI just as it does to any other. Some places add specific rules, such as New York City's requirement for yearly bias audits of automated hiring tools.

Check: For one feature, list the groups and kinds of unfairness that matter. Measure quality or outcomes for each group on your evaluation set, find the largest gap, and make one change that narrows it.

### 10.16 Regulation [2, some topics 3]

From the Governance and safety concern. Learn this with legal, for the places your product operates.

AI law changes quickly and differs by country. This section aims at recognizing when a law applies and what it asks of the product. Legal decides how it applies to you.

- [2] The EU AI Act. It sorts AI systems by risk. A few uses are banned, such as social scoring and manipulation that exploits people's vulnerabilities. High-risk uses, such as hiring, credit, education and access to essential services, carry duties for risk management, data quality, documentation, logging, human oversight, accuracy and security. Systems that talk to people or generate content have transparency duties, and most others have few or none. Duties also differ between companies that make AI systems and companies that use them (see 1.13 Governance and safety in foundation models and 12.11 Governance and safety in model adaptation). Its obligations phase in over several years, and a 2026 amendment pushed back the dates for high-risk systems, so check the current schedule.
- [3] Transparency you build. Tell people when they're talking to an AI, label content the AI made where the law requires it (see 4.6 Setting expectations and 4.7 Governance and safety in product and UX), and mark generated content in a machine-readable way where required. Standards such as C2PA content credentials attach a record of origin to media, and watermarks such as Google's SynthID hide a mark in the content itself. Metadata is easily stripped and watermarks can be weakened, so treat both as signals, not proof.
- [3] Duties that land in the product. Most legal duties for application teams turn into product features: disclosures, logs, documentation, human review steps, ways for people to contest decisions, data deletion and storing data in the right region. Build them so that legal can point to them.
- [2] Other laws. Data protection laws such as GDPR apply to AI like any other processing, including limits on decisions made solely by automated means. Sector rules apply in health, finance and employment. Many other places regulate AI too, such as China's rules for generative AI services and US state laws on automated decisions, chatbots and deepfakes. Know which places your product operates in.
- [2] Staff knowledge. Some laws, such as the EU AI Act, expect companies to take steps so that staff who build or use AI systems understand them well enough for their roles. Teams often meet this with training on how their AI systems work and fail.
- [2] Frameworks and standards. NIST's AI Risk Management Framework and its profile for generative AI, and ISO/IEC 42001 for AI management systems, give companies a structure for governing AI. Enterprise customers increasingly ask whether you follow one.

Check: For one feature, list the places it's offered and the laws that may apply there, and confirm the list with legal. For each duty that applies, show the product feature that meets it, such as a disclosure, a log or a review step.

### 10.17 Intellectual property [2]

From the Governance and safety concern. Learn this before features that generate content others will publish or sell.

Intellectual property covers who owns what goes into the system and what comes out. Licenses for models are covered in 12.11 Governance and safety in model adaptation and 13.1 Open-weight models, rights to indexed content in 6.15 Governance and safety in context, memory and retrieval, and dataset licenses and provider terms on outputs in 11.5 Governance and safety in the data layer. This section covers the questions they share.

- [2] Copying in outputs. Models can reproduce text, code or images they were trained on, sometimes word for word (see 3.20 Pre-training corpora). A copied code snippet can bring its license with it. Some providers offer filters that block outputs matching public code, and some promise to defend customers against copyright claims over outputs (see 1.13 Governance and safety in foundation models).
- [2] Who owns the output. In many places, including the US, material made by AI without meaningful human creative input can't be copyrighted. If your product sells generated content, tell customers what rights they get, and check that your provider's terms allow it.
- [2] What you put in. Check that you may use every input: documents you index, data you fine-tune on, sites your agents read or operate (see 5.17 Governance and safety in tools and protocols), and other providers' model outputs used for training (see 3.11 Synthetic data and 12.2 Fine-tuning and preference datasets).
- [2] Real people and brands. Generated images, voices and text can imitate real people, artists' styles or trademarks. Get consent before cloning anyone's voice or likeness, and block requests to impersonate real people where the harm or legal risk is high.
- [1] Unsettled law. Courts are still deciding lawsuits over training data and outputs, and the rules may shift. Know that the questions are open, and bring in legal before launching features that depend on the answers.

Check: For one feature, list every input it uses and every kind of output it produces, with the license or terms that cover each. Find one input whose rights you can't confirm, and resolve it with legal.

### 10.18 Security in governance and safety [3]

From the Governance and safety concern.

- [3] Privacy depends on security. Most privacy breaches start as security failures, such as an exposed storage bucket or a missing permission check. Make sure the security controls (see 5.6 Access and identity and 10.1 Preventing data theft) cover every place on your personal-data map.

### 10.19 Observability in governance and safety [3]

From the Governance and safety concern.

- [3] Watching for harm. Track guardrail blocks, user reports of harmful output and conversations on sensitive topics, by feature and model version. A rise after a model upgrade is a safety regression, even if nothing else changed (see 9.3 Metrics and dashboards).

### 10.20 Reliability in governance and safety [2–3]

From the Governance and safety concern.

- [3] Failing safe. If a guardrail fails or times out, block high-risk output rather than show it unchecked. For low-risk features, you may choose to carry on and log the failure.
- [2] Consistent decisions. When AI helps decide something about people, similar cases should get similar outcomes. Answers that change from run to run (see 1.8 Reliability in foundation models) are a fairness problem as well as a quality one.

### 10.21 Cost and latency in governance and safety [3]

From the Governance and safety concern.

- [3] What safety costs. Every guardrail adds cost and delay. Run cheap checks on every request and expensive ones only where the risk calls for them, and run checks alongside the main call where you can (see 7.2 Workflow patterns).

Testing for harms, refusals of reasonable requests and fairness, and keeping records of testing, are covered in 10.11 Safety and adversarial testing and 3.16 Governance and safety in evaluation.

Data quality meets governance in documentation and lineage: data cards, records of where each example came from, and whether datasets represent the people the product serves (see 11.4 Data quality in the data layer and 11.5 Governance and safety in the data layer).

### Key reading for Stage 10

- Nasr et al., "The Attacker Moves Second: Stronger Adaptive Attacks Bypass Defenses against LLM Jailbreaks and Prompt Injections" (2025), on testing defenses against attackers who adapt.
- Debenedetti et al., "AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents" (2024).
- Anil et al., "Many-shot Jailbreaking" (Anthropic, 2024), and Sharma et al., "Constitutional Classifiers: Defending against Universal Jailbreaks across Thousands of Hours of Red Teaming" (Anthropic, 2025).
- The EU AI Act (Regulation (EU) 2024/1689, amended in 2026 by Regulation (EU) 2026/1744), especially the articles on banned practices, high-risk systems and transparency, and the European Commission's guidance on applying it. EUR-Lex has a consolidated version that includes the amendment.
- NIST, *Artificial Intelligence Risk Management Framework (AI RMF 1.0)* (2023), and its *Generative Artificial Intelligence Profile* (NIST AI 600-1, 2024).
- ISO/IEC 42001:2023, the standard for AI management systems.
- The UK Information Commissioner's Office, *Guidance on AI and data protection*.
- Weidinger et al., "Ethical and social risks of harm from Language Models" (DeepMind, 2021), a map of the kinds of harm.
- Inan et al., "Llama Guard: LLM-based Input-Output Safeguard for Human-AI Conversations" (Meta, 2023).
- Barocas, Hardt and Narayanan, *Fairness and Machine Learning: Limitations and Opportunities* (MIT Press, 2023; free at fairmlbook.org).
- Mitchell et al., "Model Cards for Model Reporting" (2019).
- Raji et al., "Closing the AI Accountability Gap: Defining an End-to-End Framework for Internal Algorithmic Auditing" (2020).
- US Copyright Office, *Copyright and Artificial Intelligence, Part 2: Copyrightability* (2025).
- The C2PA specification (c2pa.org), on content credentials.
- Your model providers' usage policies, and the documentation for one guardrail tool, such as NeMo Guardrails or Llama Guard.
- Bullwinkel et al., "Lessons From Red Teaming 100 Generative AI Products" (Microsoft, 2025).

## Stage 11. Learning from use

Real users send inputs your evaluation set never imagined, and what they ask changes over time. This stage covers evaluating production traffic, turning feedback into datasets, experimenting with real users, and keeping data trustworthy as it changes.

| Section | Level | Why this level | From |
|---|---|---|---|
| 11.1 Evaluation in production | 3 | Real users find what your test set missed | Evaluation |
| 11.2 Product feedback | 4 | Only your product has this data | Layer 10 |
| 11.3 Experiments with real users | 3 | Only real users show whether a change helps them | Evaluation |
| 11.4 Data quality in the data layer | 3 | Bad data quietly breaks evaluations and fine-tunes | Layer 10 |
| 11.5 Governance and safety in the data layer | 2–3 | Decides which data you may use at all | Layer 10 |
| 11.6 Security in the data layer | 2 | Poisoned or leaked data is rare but costly | Layer 10 |
| 11.7 Cost in the data layer | 2–3 | Labels and generated data cost real money | Layer 10 |
| 11.8 Change over time | 3 | Data stops being true without raising an error | Data quality |
| 11.9 Checking and fixing data | 3 | Checks help only if problems get fixed where the data lives | Data quality |
| 11.10 Lineage and impact | 3 (some topics 4) | You must be able to trace any answer to its sources, and back | Data quality |
| 11.11 Security in data quality | 3 | Some bad data is planted on purpose | Data quality |
| 11.12 Governance and safety in data quality | 3 | Wrong data about people harms them | Data quality |

### 11.1 Evaluation in production [3]

From the Evaluation concern. Use this from the first release.

Real users send inputs your evaluation set never imagined, and what they ask changes over time. Evaluation in production catches what offline evaluation misses, and its failures become new offline cases (see 11.2 Product feedback).

- [3] Scoring live traffic. Run code checks, and judges you've checked against people, on a sample of production traces. Use criteria that need no reference answer, since nobody knows the right one. Attach each score to its trace (see 3.12 Observability in evaluation), and track pass rates by feature, version and slice.
- [3] Alerts on quality. Alert when a pass rate falls, when format errors or refusals rise, or when the mix of requests shifts (see drift in 11.2 Product feedback). Quality can fall without a single error in the logs: a model changes behind an alias (see 1.4 Frontier proprietary models), a document source changes, or users start asking about something new.
- [3] Blocking or measuring. The same check can run in the request path, stopping or repairing a bad output before the user sees it, or run afterwards on samples to measure quality. Checks in the request path must be fast and cheap, because they add latency to every request. Which guardrails to run is covered in 10.10 Content policy and guardrails.
- [3] Gradual rollouts. Release a change to a small share of traffic first, called a canary, and watch quality, errors and cost. Roll back if they get worse. The traffic splitting itself is covered in 8.6 Routing.
- [3] What users do as a signal. Accepts, edits, regenerations, thumbs-down and requests for a person are noisy but plentiful signs of quality (see 4.4 Feedback and measurement). Use them to find problems, and to check that offline gains reach users.

Check: Run your code checks, and two judges you've checked against people, on a daily sample of production traces for one feature. Chart pass rates by version, set an alert on a drop, and turn one alerted failure into a regression case.

### 11.2 Product feedback [4]

From Layer 10, Data.

Your product is the only source of this data, and it is the main advantage an application team has over the model labs. As the stack overview at the start of this file describes, feedback flows from Product and UX back into Data.

- [4] Explicit feedback. Thumbs up or down, ratings, reports and comments. There is little of it and it is skewed, because people mostly rate interactions that went memorably well or badly.
- [4] Implicit feedback. Users editing the output, accepting or rejecting a suggestion, copying, regenerating, rephrasing the same request, abandoning a session, or asking for a human. There is a lot of it, but it is noisy.
- [4] Outcome signals. Business results tied to an interaction, such as a resolved ticket, merged code or a completed purchase.
- [4] Traces into datasets. Sample traces for review and cluster them by topic and failure. Turn failures into regression tests. Add good interactions to fine-tuning or preference data. One corrected output can serve three uses: as an evaluation case, as a fine-tuning example, and together with the original output as a preference pair. Put each example in only one split.
- [4] Closing the loop. Decide what will fix a failure: a prompt change, a retrieval change, new evaluation cases, fine-tuning, or routing. Then confirm the fix with the evaluation set.
- [3] Feedback design. Work with Product and UX on this. Keep feedback prompts low-friction, ask users for a reason, and save user edits as corrected examples.
- [3] Instrumentation. Log the full trace: input, retrieved context, prompt version, model, tool calls, output, latency and cost. Give it IDs that tie each piece of feedback to the exact trace and version. This is shared with Observability (see 2.7 What to record).
- [3] Bias in feedback. Only some users choose to give feedback, and the interface shapes what they say. Feedback may have been collected on an older version of the product. The model's own outputs can also shape its future data, which creates a loop.
- [3] Drift. Notice when what users ask about changes over time, using topic clustering and embedding distances. Refresh datasets to match.
- [3] Permission to use the data. Consent, terms of service, enterprise contracts that forbid training on customer data, zero-data-retention agreements and data residency. This is shared with Governance and safety (see 10.12 Privacy and personal data).
- [2] Online tests. A/B tests that confirm improvements measured offline also hold for real users. This is shared with Evaluation (see 11.3 Experiments with real users).

Check: Link the feedback in your product to its traces. Set up a scheduled job that turns new failures into evaluation cases. Point to one change that came out of this loop and improved a metric.

### 11.3 Experiments with real users [3]

From the Evaluation concern. Learn this once you have enough traffic.

Offline evaluation predicts how a change will do. An experiment shows how it does with real users. Splitting traffic is covered in 8.6 Routing, and product metrics in 4.4 Feedback and measurement. This section covers designing experiments.

- [3] When to run one. Run an experiment when a change could change how users behave, such as a new model, a new interface or different answer lengths, and when you have enough traffic to detect the effect. Small, safe fixes can ship on offline evaluation alone.
- [3] Planning. Before you start, write down the hypothesis, the main metric, the guardrail metrics, the sample size and the duration. Assign users, not requests, to each version, so each user has a consistent experience and no user's behavior is split across versions.
- [3] Sample size and duration. Work out how many users you need to detect the smallest effect worth acting on. This is called a power calculation. Run for at least a full week to cover weekly patterns, and longer for new features, whose use often spikes and then falls.
- [3] Peeking. Checking results every day and stopping as soon as they look good produces false wins. Fix the duration in advance, or use sequential testing methods, which are designed for early looks.
- [3] Checking the experiment itself. Confirm that each version got the share of users it should. A mismatch, called a sample ratio mismatch, means something in the assignment is broken, and the results can't be trusted. Also confirm that users actually saw the version they were assigned.
- [3] Reading results. Look at the main metric, the guardrail metrics and the key slices together. A change that raises engagement but also raises complaints or cost is not a win. Compare the result with what offline evaluation predicted, and fix the evaluation set when they disagree (see 4.4 Feedback and measurement).
- [2] Other designs. Interleaving mixes results from two rankers into one list and sees which ones users pick. For search and recommendations, it needs far less traffic than an A/B test. Multi-armed bandits shift traffic toward the better version while the test runs. Holdouts keep a small group of users without a feature for months, to measure its long-term effect.
- [2] Experimentation platforms. Many companies already have an experimentation platform, built in-house or bought, such as Statsig, Optimizely or GrowthBook, and data scientists who run it. Use it and involve them rather than building your own.

Check: Design and run an A/B test of one model or prompt change. Write the hypothesis, metrics and sample size in advance, run for the planned duration, and check for sample ratio mismatch. Report the main and guardrail results with confidence intervals, next to what offline evaluation predicted.

### 11.4 Data quality in the data layer [3]

From Layer 10, Data.

- [3] Dimensions. Accuracy, completeness, consistency, freshness, uniqueness, validity, and representativeness (whether the data looks like real use).
- [3] Validation. Check every example against a schema, using JSON Schema or Pydantic. Run automated checks whenever a dataset changes, for example with pandera or Great Expectations.
- [3] Lineage. Record where each example came from: whether it is real or synthetic, which model and prompt generated it, and which labeler labeled it under which guideline version.
- [3] Dataset documentation. A short data card for each dataset that covers its purpose, sources, collection method, known gaps and license. "Datasheets for Datasets" is the model to follow.
- [3] Decontamination. Keep evaluation examples out of training data and out of few-shot prompts.
- [3] Staleness. Datasets go out of date as the product and its users change, so set a refresh schedule.
- [2] Label noise. Estimate how many labels are wrong and find them, for example with confident learning or by relabeling items where labelers disagreed.

### 11.5 Governance and safety in the data layer [2–3]

From Layer 10, Data.

- [3] Personal data. Detect personal data and remove or mask it before it enters any dataset, for example with Presidio, pattern matching or an LLM. Masked data can still identify people.
- [3] Consent, purpose and retention. Use data only for what users agreed to, and keep only what you need. Delete data on schedule and when users ask.
- [3] Provider data policies. Whether a provider stores API inputs or trains on them, how long it keeps them, and where it processes them.
- [2] Bias and representation. Which groups, languages and cases are missing or overrepresented. Check results one slice at a time.
- [2] Licensing and IP. Dataset licenses, such as CC BY versus non-commercial or research-only licenses. Also provider terms on outputs, and copyrighted material in fine-tuning data.
- [2] Regulations. GDPR, CCPA, HIPAA, and the EU AI Act's rules on data governance and training-data documentation. Know enough to recognize when to bring in legal.
- [1] Machine unlearning. Removing specific data from a trained model is hard and unreliable. This is one reason to filter data before training.

### 11.6 Security in the data layer [2]

From Layer 10, Data.

- [3] Leaks. Secrets and personal data can slip into logs, then into datasets, then into fine-tuned models, where attackers can extract them.
- [2] Data poisoning. Malicious examples planted in fine-tuning data or scraped sources to create hidden behaviors.
- [2] Feedback manipulation. Users gaming ratings to steer what the system learns.
- [2] Access control. Permissions and audit logs for datasets that contain customer data.

### 11.7 Cost in the data layer [2–3]

From Layer 10, Data.

- [3] Sample size. Build the smallest dataset that answers your question.
- [2] Data costs. Expert labeling hours, tokens spent generating synthetic data, and fine-tuning runs. Weigh them against savings, such as cheaper inference after distillation.

Observability and Reliability have little of their own in this layer. Trace capture is covered in 11.2 Product feedback, and the tracing infrastructure is covered in 2.6 Tracing. Reliability comes in only as reproducible datasets and environments, which 3.2 Data foundations and 12.7 RL environments cover.

### 11.8 Change over time [3]

From the Data quality concern.

Data changes after you check it. Freshness and keeping indexes in sync are covered in 6.14 Data quality in context, memory and retrieval and 6.4 Preparing documents, and changes in what users ask and datasets going stale in 11.2 Product feedback and 11.4 Data quality in the data layer. This section covers noticing change in every source.

- [3] Kinds of change. Content changes, such as an updated price list. Structure changes, such as a renamed field or a new code value. Meaning changes, such as a metric whose definition was revised. And the world changes, such as a new law or product. Each needs its own way of being noticed.
- [3] Watching sources. Monitor each important source for how recently it updated, how much data arrived, whether its structure changed and whether its values shifted. Alert when a source stops updating or its volume drops suddenly, because a broken feed often looks like a quiet day.
- [3] Changes upstream. A renamed field or a new category in a source system can break extraction or retrieval without raising any error. Checks at every boundary catch some of these (see 6.12 Data the model reads), and agreements with source owners catch more before they ship (see 6.11 Data quality foundations).
- [3] When facts were true. For facts that change, such as prices, policies and people's roles, record when each was true, not just when it was stored. Then questions about the past get the facts that held at the time, and current answers get the latest.

Check: Add monitoring for freshness, volume and structure to the three sources one feature depends on most. Break one in a test environment, such as by stopping its updates or renaming a field, and show that the monitoring catches it before users would notice.

### 11.9 Checking and fixing data [3]

From the Data quality concern.

Validating datasets, finding wrong labels and documenting datasets are covered in 11.4 Data quality in the data layer, and inspecting parsed documents in 6.4 Preparing documents. This section covers keeping checks running and getting problems fixed.

- [3] Checks in every pipeline. Run automated checks wherever data is loaded or transformed: schemas, ranges, missing values, duplicates, links between records and freshness. Tools such as Great Expectations, Soda, dbt tests and pandera do this. When a batch fails, hold it back for review rather than silently dropping it or loading it anyway.
- [3] Reading samples. Automated checks catch broken structure, not wrong meaning. Read samples of each source regularly, such as parsed documents, extracted fields and labels, and look for what the checks can't see.
- [3] Models as checkers. A model can flag records that look contradictory, outdated, duplicated or off-topic, at a scale people can't read. Check its flags against human judgment before relying on them, as you would any judge (see 3.8 LLM judges).
- [3] Fixing where the data lives. When data is wrong, get it fixed at the source, by its owner, rather than patching around it in prompts. Workarounds in prompts pile up, hide the problem from everyone else who uses the data, and break when the data changes. Track data problems like bugs, with owners and dates.
- [2] Quality signals the system can use. Store signals such as how authoritative a source is, how fresh a record is and whether a person confirmed a value, as metadata. Retrieval, ranking and the model itself can then favor better data (see 6.6 Query handling and reranking and 6.14 Data quality in context, memory and retrieval).

Check: Add automated checks to one pipeline that feeds an AI feature, and hold back batches that fail. Run a model-based check over one source, compare its flags with 50 labels you made yourself, and get one real problem fixed by the source's owner.

### 11.10 Lineage and impact [3, some topics 4]

From the Data quality concern.

Lineage records where data came from and what was built from it. Lineage for training and evaluation data is covered in 11.4 Data quality in the data layer, lineage for the chunks in an index in 6.14 Data quality in context, memory and retrieval, versions and IDs on every trace in 2.7 What to record, and records of where models, datasets and tools came from in 10.14 Audit trails and documentation. This section covers connecting them.

- [4] From answer to source, and back. Link each answer to the context it used, each piece of context to the document or record it came from and that source's version, and each dataset example to the trace and source behind it. Then you can answer two questions quickly: where did this wrong answer's information come from, and which answers, datasets and generated values used this source after it went wrong?
- [3] When a source turns out to be bad. A document may prove outdated, a dataset unlicensed, a customer's data due for deletion, or a tool's results wrong for a week. Use lineage to find everything built from it, such as index entries, memories, summaries, evaluation cases, training examples and fine-tuned models, and fix or remove each one (see 10.12 Privacy and personal data and 10.14 Audit trails and documentation).
- [3] Lineage you can afford. Full lineage for every value is expensive. Record IDs and versions at each step, as traces already do (see 2.7 What to record), rather than copies of the data, and keep more detail for data that feeds training, decisions about people or regulated features.
- [2] Lineage tools. Data teams track lineage between tables and pipelines with data catalogs and with standards such as OpenLineage. Find out whether your company has one, and connect your AI pipelines to it where you can.

Check: Take one wrong answer from production and trace it back to the source record and version it came from. Then go the other way: from that source, find every answer, index entry, memory and dataset example that used it after it went wrong.

### 11.11 Security in data quality [3]

From the Data quality concern.

- [3] Bad data on purpose. Some bad data is planted: poisoned documents, memories, feedback and training examples. Quality checks catch some of it, but treat any source that outsiders can write to as untrusted (see 5.1 Security foundations).

### 11.12 Governance and safety in data quality [3]

From the Data quality concern.

- [3] Wrong data about people. A wrong address, a mistaken flag or a confused identity harms the person it's about, and privacy laws give people the right to have it corrected (see 10.12 Privacy and personal data). Make each correction reach every copy, including values the model wrote and memories.

The other concerns each meet data quality in one place: wrong labels in evaluation sets in 3.16 Governance and safety in evaluation, complete traces in 9.19 Reliability in observability, sources that stop updating without an error in 9.15 Governance and safety in reliability, and what duplicate and outdated documents cost in 8.20 Governance and safety in cost and latency.

### Key reading for Stage 11

- Kohavi, Tang and Xu, *Trustworthy Online Controlled Experiments* (Cambridge, 2020).
- Gebru et al., "Datasheets for Datasets" (2021).
- Breck et al., "Data Validation for Machine Learning" (Google, 2019).
- Northcutt, Athalye and Mueller, "Pervasive Label Errors in Test Sets Destabilize Machine Learning Benchmarks" (2021).
- Chip Huyen, *Designing Machine Learning Systems* (O'Reilly, 2022). Chapter 8 covers changes in data over time and how to monitor them.
- The OpenLineage documentation (openlineage.io), on recording lineage across pipelines.
- The documentation for one data validation tool, such as Great Expectations, Soda or pandera.

## Stage 12. Adapting models

Changing a model's weights is the most powerful way to change how it behaves, and the most expensive to build and keep up. This stage covers deciding whether to adapt, the datasets adaptation needs, fine-tuning, distillation, and learning from preferences and automated rewards.

| Section | Level | Why this level | From |
|---|---|---|---|
| 12.1 Deciding whether to adapt | 3 | Most problems have cheaper fixes, so adapting needs a clear case | Layer 7 |
| 12.2 Fine-tuning and preference datasets | 3 | When you fine-tune, the dataset is what you control | Layer 10 |
| 12.3 Supervised fine-tuning | 3 | The adaptation method app teams actually run | Layer 7 |
| 12.4 Distillation | 3 | The most common reason app teams fine-tune | Layer 7 |
| 12.5 Owning an adapted model | 3 | A fine-tune must be retrained, re-tested and eventually retired | Layer 7 |
| 12.6 Learning from human preferences | 2 | Occasionally useful for tone and behavior; labs use it at scale | Layer 7 |
| 12.7 RL environments | 2 (graders 3) | Mostly a lab tool, but the graders double as your eval graders | Layer 10 |
| 12.8 Learning from automated rewards | 2 | Powerful for checkable tasks, but costly and easy to game | Layer 7 |
| 12.9 Evaluation in model adaptation | 3 | An adapted model must beat the baseline without breaking anything else | Layer 7 |
| 12.10 Cost and latency in model adaptation | 3 | Adapting pays off only when savings outweigh training and upkeep | Layer 7 |
| 12.11 Governance and safety in model adaptation | 2–3 | Fine-tuning can weaken safety, and licenses follow the model you change | Layer 7 |
| 12.12 Security in model adaptation | 2 | Anyone who can supply training data can change the model | Layer 7 |
| 12.13 Observability in model adaptation | 2 | Training curves only mean something next to real outputs | Layer 7 |

### About Layer 7: Model adaptation

Adaptation changes a model's weights. It's the most powerful way to change how a model behaves, and the most expensive to build and keep up. Most application teams rarely need it, because better prompts, retrieval and stronger models solve most problems more cheaply. Teams that do adapt usually have one of two reasons:
- to make a small model do a narrow job as well as a large one, for less money and latency
- to lock in a format or behavior that prompting can't hold

Go deep (level 3) on four things: deciding whether to adapt, supervised fine-tuning, distillation, and owning a fine-tuned model over time. These are the parts application teams actually do. Learn preference tuning and reinforcement learning to level 2; you'll use them occasionally, mostly through provider APIs. How labs run these methods at scale is level 1. No concept in this layer needs level 4. If you need that depth, you're doing a model-training job, not an application one.

Before you adapt a model, answer three questions:

1. Have you tried the cheaper fixes? A clearer prompt, examples, retrieval and a stronger model come first.
2. Can you measure the gap? You need an evaluation set that shows where the current model falls short. The same set will show whether adaptation closed the gap.
3. Can you get the data? You need enough good examples of the behavior you want, or a grader that can score it. Building both is covered in 12.2 Fine-tuning and preference datasets and 12.7 RL environments.

If any answer is no, don't adapt yet.

### 12.1 Deciding whether to adapt [3]

From Layer 7, Model adaptation. Most of the value is in knowing when not to.

Knowing when not to adapt is most of the value in this layer.

- [3] The order of fixes. Try fixes in order of cost: a clearer prompt, examples in the prompt, retrieval for missing facts, splitting the task into steps, and a stronger or reasoning model. Each one is cheaper to try and easier to undo than training. Adapt only when they all fall short and your evaluation set shows the gap.
- [3] Good reasons to adapt. Four situations justify it:
  - A small model needs to match a large one on a narrow task, for lower cost or latency.
  - Output format, style or behavior needs to be more consistent than prompting can make it.
  - Long instructions and examples repeated in every prompt could be trained in instead, which shortens prompts.
  - The model has to run on a device or offline.

  What fine-tuning can and can't teach is covered in 12.2 Fine-tuning and preference datasets.
- [3] The full cost. Count the data work, training runs, evaluation and hosting, plus retraining every time the base model changes. Some providers also charge more per token to run a fine-tuned model than the base model.
- [2] Where the weights live. When you fine-tune a provider's proprietary model, the result stays with the provider. You can call it but not download it, and it lasts only as long as the provider supports its base model. When you fine-tune an open-weight model, you control the weights but have to host them yourself.

Check: For one problem in your product, write a one-page case for or against adapting a model. Include the cheaper fixes you tried, the evaluation results that show the remaining gap, where the training data would come from, and the full cost weighed against the expected savings.

### 12.2 Fine-tuning and preference datasets [3]

From Layer 10, Data. You need evaluation sets before you can tell whether fine-tuning helped.

When prompting and retrieval aren't enough, app teams fine-tune models. They either use a provider's API or apply a lightweight method such as LoRA to an open-weight model. The training method is mostly handled for you, and how it works is covered in 12.3 Supervised fine-tuning and 12.6 Learning from human preferences. What you control is the dataset. If your team never fine-tunes, level 2 is enough, but these dataset skills still carry over to evaluation.

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

### 12.3 Supervised fine-tuning [3]

From Layer 7, Model adaptation. Learn it hands-on, through a provider's API or with LoRA.

Supervised fine-tuning (SFT) trains a model on examples of inputs paired with ideal outputs, so that it learns to produce outputs like them. It's the method application teams use most. Building the datasets is covered in 12.2 Fine-tuning and preference datasets.

- [3] How it works. For each example, training adjusts the model's weights slightly to make the ideal output more likely. Training passes over the dataset a few times, and each pass is called an epoch. A validation set, held out from training, shows whether the model is learning the task or just memorizing the examples.
- [3] Two ways to run it. One way is a provider's fine-tuning API: you upload a dataset, start a job and get back a new model name to call. The other is to fine-tune an open-weight model yourself with a library such as Hugging Face TRL, Unsloth or Axolotl. Get hands-on with at least one of them.
- [3] LoRA. Instead of changing all the weights, LoRA trains small add-on matrices, called an adapter, and leaves the original weights frozen. It's the most common of several parameter-efficient methods. It needs far less memory, trains faster and produces small files. On narrow tasks it usually comes close to full fine-tuning: it learns less from big changes, but it also forgets less. QLoRA saves more memory by storing the frozen weights at lower precision. The memory numbers are in 13.2 Compute arithmetic.
- [2] Full fine-tuning. Updating every weight can teach the model more. But it costs far more memory and compute and is more likely to damage the model's general abilities. Application teams rarely need it.
- [3] Reading loss curves. Training loss should fall. Validation loss should fall and then flatten. If validation loss starts rising while training loss keeps falling, the model is overfitting, so stop at the point where validation loss was lowest. Loss doesn't measure quality directly, so always confirm with your evaluation set.
- [2] Settings. The learning rate matters most: too high damages the model, and too low changes nothing. A few epochs are usually enough. LoRA adds a rank setting that controls the adapter's size. Start from the provider's or library's defaults and change one setting at a time.
- [3] Starting point. Fine-tune the instruction-tuned version of a model, not the base version, and train in the chat format the model already uses. Try the smallest model that might work first, since it's the cheapest to train and to run.
- [2] Embedding models. Fine-tuning an embedding model on real queries paired with the right documents can noticeably improve search in specialized domains. Libraries such as Sentence Transformers make this a small project. Learn this to level 3 if you own search.
- [2] Small classifiers. You can fine-tune a small encoder model to classify text, for example for routing, intent or safety checks. At high volume, this often beats prompting a large model on cost and speed.
- [1] Merging models. Weights from separately fine-tuned models can sometimes be combined into one model without further training.

Check: Fine-tune a small model on a narrow task, with LoRA or through a provider's API. Plot training and validation loss and choose where to stop. Show that the result beats the prompted baseline on your evaluation set without getting worse on a general test set.

### 12.4 Distillation [3]

From Layer 7, Model adaptation. It builds on supervised fine-tuning.

Distillation trains a smaller, cheaper model (the student) to imitate a larger one (the teacher). It's the most common reason application teams fine-tune.

- [3] The workflow. Run the teacher on real inputs and keep only its good outputs. Fine-tune the student on those outputs with SFT. Compare the student with the teacher on a held-out evaluation set, then move traffic to the student gradually. Building the dataset is covered in 12.2 Fine-tuning and preference datasets.
- [3] What to expect. On a narrow task, a well-distilled student can come close to the teacher at a fraction of the cost and latency. It rarely beats the teacher, and it handles unusual inputs worse, so keep the teacher available for hard cases. Sending hard cases to the teacher is covered in 8.6 Routing.
- [2] Provider tools. Some providers let you store a large model's production outputs and use them directly to fine-tune one of their smaller models.
- [2] Outputs versus probabilities. Labs often distill using the probabilities the teacher assigns to every possible next token, which carry more information than its text alone. With a provider's model you only get the text, so you distill from outputs alone.
- [2] Distilling reasoning. Training a small model on a reasoning model's step-by-step answers can pass on reasoning ability for a narrow task. With proprietary teachers, two things limit this: the reasoning may be hidden or summarized, and provider terms may forbid it.

Check: Distill a large model into a small one for one task. Report the student's quality, cost and latency next to the teacher's on the same held-out set. List the kinds of input where the student falls short.

### 12.5 Owning an adapted model [3]

From Layer 7, Model adaptation.

A fine-tuned model is software you have to maintain. It doesn't get better when the provider releases a better base model.

- [3] Base model upgrades. When a better base model arrives, your fine-tune doesn't carry over to it. You can retrain on the new base, or first check whether the new model with only a prompt already beats your fine-tune. Expect to face this decision every few months.
- [3] Retirement. Providers eventually retire the base models they offer for fine-tuning, and fine-tuned models built on them are retired too. Open-weight fine-tunes don't expire, but they fall behind newer models.
- [3] Reproducibility. Keep everything you need to retrain: the dataset version, the base model and its version, the settings and the evaluation results. Automate the pipeline so that retraining is routine, not a project.
- [3] Fallback. Keep a working prompt-based version of every fine-tuned feature. Then you can switch back if the fine-tune breaks or its base model is retired.
- [2] Serving. You call a hosted fine-tune like any other model. Open-weight fine-tunes need hosting, and many LoRA adapters can share one base model on the same hardware. How that works is covered in 13.5 Serving engines and batching.

Check: Retrain one fine-tuned model on a newer base model using your automated pipeline. Compare the result with the old fine-tune and with the new base model using only a prompt. Show that you can switch to the prompt-based fallback.

### 12.6 Learning from human preferences [2]

From Layer 7, Model adaptation.

Preference tuning teaches a model which of two answers is better. Labs use it to make models helpful and safe. Application teams use it occasionally, after SFT, to change tone, length or specific unwanted behaviors.

- [2] RLHF. Reinforcement learning from human feedback works in two steps. First, a reward model is trained to predict which answers people prefer. Then reinforcement learning pushes the model toward answers the reward model scores highly. A penalty keeps the model from drifting too far from where it started. This is how chat assistants were first made helpful.
- [2] DPO and related methods. Direct preference optimization (DPO) learns straight from preference pairs, without a separate reward model or reinforcement learning loop. It's simpler and more stable than RLHF, and some providers offer it through their fine-tuning APIs. Related methods include KTO, and ORPO, which combines SFT and preference tuning in one step. The data each method needs is covered in 12.2 Fine-tuning and preference datasets.
- [2] When to use it. Use preference tuning when the model already does the task but you want to change how it does it: shorter answers, a different tone or fewer unnecessary refusals. Pairs of an original output and a corrected one from your product work well.
- [2] Over-optimization. Pushed too far, a model learns to please the reward signal rather than the user, for example by writing longer or more flattering answers. A strength setting, often called beta, limits how far the model can move from its starting point.
- [1] AI feedback. Labs increasingly have models judge answers against written principles instead of asking people. Anthropic's Constitutional AI is one example.

Check: Explain how DPO differs from RLHF. Describe a problem in your product that preference tuning could fix better than SFT, and say where the preference pairs would come from.

### 12.7 RL environments [2, graders 3]

From Layer 10, Data. They build on graders and evaluation.

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

### 12.8 Learning from automated rewards [2]

From Layer 7, Model adaptation. Learn this after you've built graders (see 12.7 RL environments).

In this method, the model learns by attempting tasks that are scored automatically, by tests, checks or a grading model. This is how labs trained today's reasoning models. Some providers offer it as reinforcement fine-tuning (RFT).

- [2] How it works. For each task, the model makes several attempts, and a grader scores each one. Training makes the higher-scoring attempts more likely. Methods such as GRPO compare each attempt with the others in its group, so there's no need to train a separate model that estimates how good each attempt is. A penalty keeps the model close to where it started.
- [2] Verifiable rewards. Some answers can be checked automatically, for example by running tests or comparing with a known answer. For these tasks, rewards are cheap and hard to dispute. This is a big reason models have improved fastest at math and code.
- [2] Reinforcement fine-tuning through a provider. The provider runs the training loop on your tasks and grader. It can work with relatively few tasks. But every training step needs many model attempts, so it costs far more than SFT. What you supply, and how to build the grader, is covered in 12.7 RL environments.
- [2] Sharpening, not creating. Reinforcement learning mostly makes a model more reliable at things it can already do some of the time. If the base model never succeeds at a task, there's nothing to reinforce. Measure its success rate across several attempts first, and use SFT to get it started if needed.
- [2] Watching the outputs. Models find ways to raise their score without doing the task. During training, read samples of what the model produces, not just the reward curve.
- [1] Agentic training. Labs use reinforcement learning to train agents on long, multi-step tasks with real tools. This is a big reason recent models handle agentic work better, but application teams rarely run it themselves.

Check: Pick a task in your product that has a checkable answer. Measure how often the base model succeeds across several attempts. Decide whether reinforcement fine-tuning, SFT or neither is the right next step, and explain why.

### 12.9 Evaluation in model adaptation [3]

From Layer 7, Model adaptation.

- [3] Beating the baseline. Compare every adapted model with two alternatives on the same held-out evaluation set: the best prompt on the base model, and a stronger model. If the adapted model doesn't clearly win on quality, cost or latency, don't ship it.
- [3] Regressions. Adaptation can weaken abilities you didn't train on, such as general knowledge, instruction following, and formats outside your task. Test on more than the target task.

Building held-out evaluation sets and keeping them out of training data is covered in 12.2 Fine-tuning and preference datasets and 11.4 Data quality in the data layer.

### 12.10 Cost and latency in model adaptation [3]

From Layer 7, Model adaptation.

- [3] Break-even. Adapting pays off when the savings on each request, multiplied by your volume, outweigh the full cost of adapting. The savings come from a smaller model, shorter prompts and lower latency. Work this out before you start. It usually takes steady, high volume.
- [2] How training is billed. Hosted fine-tuning usually charges per training token, which is dataset size times the number of epochs. Reinforcement fine-tuning is billed differently, for example by training time, and usually costs much more. Running training yourself costs GPU hours.

### 12.11 Governance and safety in model adaptation [2–3]

From Layer 7, Model adaptation.

- [3] Weakened safety. Fine-tuning can weaken a model's safety training even when the training data is harmless. Re-test safety behavior after every fine-tune, and add your own guardrails where needed.
- [2] Licenses. Check that the base model's license allows fine-tuning and commercial use of the result. Also check what you must pass on, such as attribution or the original license terms.
- [2] Provider review. Providers check fine-tuning data and fine-tuned models against their usage policies, and they can reject a job.
- [2] Regulation. Under laws such as the EU AI Act, modifying a general-purpose model heavily enough can make you legally responsible for it as its provider. Typical fine-tunes fall well short of that line, but check before large projects.
- [2] Model documentation. Write a short card for each adapted model that covers the base model, data sources, method, evaluation results and known limits.

### 12.12 Security in model adaptation [2]

From Layer 7, Model adaptation.

- [2] Fine-tuning as an attack. Your product may let users fine-tune models or supply training data. If so, a handful of harmful examples can strip safety training or plant hidden behaviors. Screen the data and test the result before deploying it.

Poisoned training data and personal data leaking into fine-tuned models are covered in 11.6 Security in the data layer. Trusting downloaded model files is covered in 1.10 Security in foundation models.

### 12.13 Observability in model adaptation [2]

From Layer 7, Model adaptation.

- [2] Training metrics. Watch loss curves (and reward curves, for reinforcement learning) alongside samples of real outputs. Record every run, with its data and settings, in an experiment tracker such as Weights & Biases or MLflow.

Reliability is covered in 12.5 Owning an adapted model: fallbacks, retraining and retirement. Data quality for training data is covered in 11.4 Data quality in the data layer.

### Key reading for Stage 12

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 7 covers when and how to fine-tune, including LoRA and model merging.
- Hu et al., "LoRA: Low-Rank Adaptation of Large Language Models" (2021).
- Dettmers et al., "QLoRA: Efficient Finetuning of Quantized LLMs" (2023).
- Biderman et al., "LoRA Learns Less and Forgets Less" (2024).
- Ouyang et al., "Training language models to follow instructions with human feedback" (2022), the InstructGPT paper on RLHF.
- Rafailov et al., "Direct Preference Optimization: Your Language Model Is Secretly a Reward Model" (2023).
- DeepSeek-AI, "DeepSeek-R1" (2025), on reinforcement learning for reasoning with GRPO, and on distilling that reasoning into smaller models.
- Hinton, Vinyals and Dean, "Distilling the Knowledge in a Neural Network" (2015).
- Qi et al., "Fine-tuning Aligned Language Models Compromises Safety, Even When Users Do Not Intend To!" (2023).
- Nathan Lambert, *Reinforcement Learning from Human Feedback* (rlhfbook.com), on preference data, preference tuning, reward models and reinforcement learning.
- The Hugging Face TRL documentation and Unsloth's guides, for hands-on fine-tuning.
- Zhou et al., "LIMA: Less Is More for Alignment" (2023), on small, high-quality fine-tuning sets.

## Stage 13. Under the hood

Model APIs hide most of these layers. This stage covers open-weight models, running models yourself, how serving engines work, and the hardware, clusters and data centers underneath. It explains how providers behave and price, and when self-hosting makes sense.

| Section | Level | Why this level | From |
|---|---|---|---|
| 13.1 Open-weight models | 2 (some topics 3) | Worth it when control, privacy or cost at scale matter | Layer 8 |
| 13.2 Compute arithmetic | 3 | Answers most compute questions: will it fit, how fast will it run, what will it cost | Layer 9 |
| 13.3 Accelerators | 2 | You choose hardware, but you don't program it | Layer 9 |
| 13.4 Running models yourself | 2 (some topics 3) | Local models for development, production hosting if you self-host | Layer 6 |
| 13.5 Serving engines and batching | 2 | Explains how providers behave and price | Layer 6 |
| 13.6 Quantization | 2 | Decides what fits, how fast it runs and what quality you lose | Layer 6 |
| 13.7 Structured generation | 2 | Explains the limits of schema enforcement | Layer 6 |
| 13.8 Faster generation | 2 | Speeds up some tasks, such as edits | Layer 6 |
| 13.9 Multi-GPU and clusters | 1 (some topics 2) | Platform teams run clusters; you only need to know why big models need them | Layer 9 |
| 13.10 Security in inference and serving | 2–3 | Self-hosted endpoints get exposed by mistake | Layer 6 |
| 13.11 Governance and safety in inference and serving | 2 | Serving logs hold user data | Layer 6 |
| 13.12 Cost and latency in compute | 3 | Choosing between an API and self-hosting is your biggest compute decision | Layer 9 |
| 13.13 Reliability in compute | 2 | Capacity shortages and hardware failures reach your users | Layer 9 |
| 13.14 Observability in compute | 2 | You'll read GPU metrics when you self-host | Layer 9 |
| 13.15 Governance and safety in compute | 2 | Where the hardware sits decides where data is processed | Layer 9 |
| 13.16 Evaluation in compute | 2 | The same model can give different results on different hardware | Layer 9 |
| 13.17 Security in compute | 1 | Providers handle most of it | Layer 9 |
| 13.18 Power and data centers | 1 | Explains prices, shortages and regions; you won't build any of it | Layer 9 |

### 13.1 Open-weight models [2, some topics 3]

From Layer 8, Foundation models. Learn this when control, privacy or cost at scale matter.

Open-weight models are models whose trained weights are published, so anyone can download and run them.

- [2] The landscape. Major families include Meta's Llama, Alibaba's Qwen, DeepSeek, Mistral, Google's Gemma and OpenAI's gpt-oss. Sizes range from under a billion parameters to hundreds of billions. The best open-weight models trail the frontier by months, not years.
- [3] When open weights win. Open-weight models are the better choice in five cases:
  - Data can't leave your control.
  - You need deep fine-tuning.
  - Your volume is high enough that self-hosting is cheaper.
  - You need to run offline or on devices.
  - You can't accept a provider retiring your model.

  Otherwise, APIs are usually simpler and more capable.
- [2] Open weights versus open source. Most open models publish their weights but not their training data or code. You can run and fine-tune them, but you can't fully inspect or reproduce them.
- [2] Licenses. Some licenses are permissive, such as Apache 2.0 and MIT. Others are custom and add conditions, such as limits for very large user bases or restrictions on certain uses. Check the license before you ship, with legal help if needed.
- [2] Variants. Open models come in base and instruction-tuned versions, in several sizes, and in lower-precision (quantized) versions, which the community often makes. Quality differs between variants, so check exactly which one you're running.
- [2] Where to find them. Hugging Face hosts most open-weight models and their model cards. You can run them yourself (see 13.4 Running models yourself and 13.2 Compute arithmetic) or call them through hosted providers.
- [2] Safety differences. Some open models have lighter safety training than API models, and anyone can fine-tune that training away. If you ship an open model, you may need your own guardrails.

Check: Pick an open-weight model that could replace an API model in one feature. Check its license and run your evaluation set on it. Then compare the cost of hosting it with the cost of the API.

### 13.2 Compute arithmetic [3]

From Layer 9, Compute.

These calculations come up in every other section, so learn them first.

- [3] Weight memory. Multiply the number of parameters by the bytes per parameter. At 16-bit precision each parameter takes 2 bytes, at 8-bit it takes 1 byte, and at 4-bit it takes half a byte. A 70-billion-parameter model at 16-bit needs about 140 GB for its weights alone, so it doesn't fit on one 80 GB GPU.
- [3] KV cache memory. The model stores intermediate results for every token in a conversation so it doesn't have to recompute them. This store is called the KV cache. It grows with context length and with the number of conversations running at once, and on long contexts it can take more memory than the weights. This is why long contexts cost more, and why a GPU can serve fewer long conversations at once. How serving engines manage the KV cache is covered in 2.4 KV-cache management.
- [3] Generation speed. For one conversation at a time, the top speed is memory bandwidth divided by the model's size in bytes. The H100 SXM, a common data center GPU, has about 3.35 TB/s of memory bandwidth. A model whose weights take 70 GB can therefore generate at most about 48 tokens per second on it. Real systems stay below this ceiling.
- [3] Cost per token. Divide the hourly cost of the hardware by the tokens it produces per hour. For example, suppose a GPU rents for $2.50 an hour and produces 1,000 tokens per second when fully busy. It makes 3.6 million tokens an hour, so each million tokens costs about $0.70. If it's busy only a quarter of the time, each million tokens costs about $2.80.
- [2] Memory-bound and compute-bound work. The model processes all the tokens in a prompt at once, so that step is limited by raw compute. It generates the answer one token at a time, so that step is limited by memory bandwidth. This is why output tokens cost several times more than input tokens, and why cached input is cheaper still. It is also why serving many conversations at once lowers the cost per token.
- [2] FLOPs per token. A model does about 2 floating-point operations (FLOPs) per parameter for each token it processes. Training takes about 6 per parameter for each training token. Use these numbers to estimate how long a long prompt takes to process, or roughly what a training run costs.
- [2] Mixture-of-experts models. These models use only some of their parameters for each token. They need memory for all of their parameters, but they generate at roughly the speed of the parameters they actually use.
- [2] Fine-tuning memory. Full fine-tuning needs about 16 bytes per parameter for weights, gradients and optimizer state, before counting activations. LoRA freezes the original weights and trains a small add-on. QLoRA also stores the original weights at 4-bit, so a model of about 8 billion parameters can be fine-tuned on a single 24 GB GPU.
- [2] Test-time compute. Reasoning models spend more compute on each answer by generating many more tokens before they reply. That makes them slower and more expensive, but more accurate on hard tasks. Choosing how much reasoning to allow for each request is covered in 8.6 Routing.

Check: Take a model's size, its precision and a context length. Estimate whether it fits on a given GPU, its top generation speed for one user, and its cost per million tokens at 30% utilization. Then measure a real run and explain the gap.

### 13.3 Accelerators [2]

From Layer 9, Compute. Focus on memory-bound work, precision and spec sheets. These explain the arithmetic.

Accelerators are chips built for the matrix math that models run on. GPUs are the most common kind.

- [2] What a GPU is good at. Thousands of small cores work in parallel. Dedicated units called tensor cores do matrix math quickly at low precision.
- [2] Memory. Each GPU has a large pool of high-bandwidth memory (HBM) and a small amount of much faster memory on the chip itself. For running models, HBM capacity and bandwidth usually matter more than raw compute. Many speedups, such as FlashAttention, come from moving data between these two kinds of memory less often.
- [2] Precision. Numbers can be stored in 32, 16, 8 or 4 bits. Each halving of precision halves memory use and roughly doubles the math the chip can do, if the chip supports that format. Recent generations added 8-bit and then 4-bit floating-point formats. Choosing a quantization method is covered in 13.6 Quantization.
- [2] Reading a spec sheet. The numbers to look for are memory capacity (GB), memory bandwidth (TB/s), compute (TFLOPS) at each precision, link speed between GPUs, and power draw (watts). Headline compute figures are often quoted "with sparsity". That doubles them, and it rarely applies in practice.
- [2] Generations. Each generation adds memory, bandwidth and support for lower precision, so the same model gets cheaper to run. Check which generation a price or benchmark refers to. NVIDIA now releases a new generation roughly every year.
- [1] The chip landscape. NVIDIA dominates, with data center GPUs such as the A100, H100, H200 and the Blackwell generation. AMD's Instinct GPUs often have more memory per chip. Google's TPUs and AWS's Trainium and Inferentia are custom chips from cloud providers. Makers of specialized inference chips, such as Cerebras and Groq, sell very fast generation.
- [2] The software stack. This covers drivers, CUDA and its libraries, AMD's ROCm, and frameworks such as PyTorch and JAX. CUDA's maturity is a big reason NVIDIA dominates. The most common setup failure is a mismatch between the versions of the driver, CUDA and PyTorch.
- [2] CPUs versus GPUs. Small models, such as embedding models, classifiers and rerankers, often run well enough on CPUs, especially after quantization. A GPU pays off once volume is high.
- [2] Local and edge hardware. Apple Silicon Macs share one pool of memory between the CPU and GPU. A Mac with a lot of memory can therefore run large quantized models, though more slowly than data center GPUs. High-end consumer GPUs have 24 to 32 GB. Phones and laptops have neural processing units (NPUs) that run small models on the device. On-device models help with privacy, cost and offline use, but memory, battery and heat limit their size.

Check: Read the spec sheets of two data center GPUs. Explain which would serve a given model more cheaply, and why.

### 13.4 Running models yourself [2, some topics 3]

From Layer 6, Inference and serving. Start with a local model.

Running models yourself means running them on your laptop for development, or on your own or rented infrastructure in production. Whether self-hosting is worth it is covered in 13.12 Cost and latency in compute.

- [3] Local development. You can run open models on your laptop with Ollama, llama.cpp or LM Studio, or with MLX on Apple Silicon. They expose an OpenAI-compatible endpoint. This is useful for offline work, fast experiments, and tests that don't cost API calls.
- [2] Managed endpoints. Inference platforms and cloud services can host an open-weight or fine-tuned model on dedicated GPUs for you. They handle the engine, scaling and updates. This is often the right middle ground between a shared API and full self-hosting.
- [2] Packaging and startup. A serving image bundles the engine and its settings. The weights are downloaded or mounted when a server starts. Loading tens or hundreds of gigabytes of weights makes up most of the startup time, so keep weights close to the servers and in a format that loads quickly. Cold starts are covered in 8.12 Cloud capacity.
- [2] Autoscaling signals. Scale on queue length, time to first token or KV cache use. CPU use says little about how busy a GPU server is. Adding GPU servers is slow, so scale early.
- [2] Health and rollouts. A server is ready only after its model has loaded and warmed up. Roll out new models or settings gradually, and warm them up before they receive traffic.
- [2] Admission control. Cap the number of concurrent requests, and the prompt length and output length of each request. When load goes beyond capacity, reject or queue the extra requests, so the rest don't all slow down.
- [2] Serving smaller models. Embedding models, rerankers and classifiers need their own serving setup. That can be a specialized server, such as Hugging Face's Text Embeddings Inference, or CPUs. These models are usually limited by throughput, so send many inputs in each call.
- [2] On-device runtimes. Runtimes such as Core ML, ONNX Runtime, ExecuTorch and LiteRT run models on phones and laptops, and WebGPU runs them in browsers. Some operating systems and browsers also offer built-in small models through their own APIs.

Check: Deploy an open-weight model behind an autoscaling endpoint, on a managed platform or your own cluster. Show three things: it scales on queue length, it turns away excess load cleanly, and it rolls out a new version without failed requests.

### 13.5 Serving engines and batching [2]

From Layer 6, Inference and serving.

A serving engine is the software that runs a model on GPUs and answers requests. Providers run engines like these behind their APIs.

- [2] What an engine does. It loads the model onto the GPUs and accepts requests. It schedules and batches those requests, manages the KV cache and runs the model efficiently. Widely used engines include vLLM, SGLang and NVIDIA's TensorRT-LLM for production, and llama.cpp and Ollama for local and smaller-scale use. Most expose an OpenAI-compatible API.
- [2] Continuous batching. Older servers waited to fill a batch, then ran it until every request finished. Modern engines add and remove requests from the running batch at every step. This is called continuous (or in-flight) batching. It keeps the GPU busy and greatly raises throughput.
- [2] Scheduling. Engines decide which requests run next and how many run together. When the KV cache is full, they queue new requests or pause running ones. Long prompts can be processed in chunks (chunked prefill) so they don't stall other users' output.
- [2] Parallelism for serving. A model too big for one GPU is split across several with tensor parallelism, which also cuts latency. More copies of the model (replicas) add throughput. Very large mixture-of-experts models spread their experts across GPUs. The hardware side of this is covered in 13.9 Multi-GPU and clusters.
- [2] Serving many adapters. One base model can serve many LoRA adapters at once, loading the right adapter for each request. This makes it cheap to host a separate fine-tune for each customer or task.
- [2] Engine settings. These are the settings that matter most:
  - maximum context length, which limits KV cache use
  - maximum concurrent requests
  - how much GPU memory the engine may use
  - the number of GPUs to split the model across
  - quantization
  - prefix caching

  Start from the defaults, and change settings while you load-test.
- [1] Separate prefill and decode. Large deployments run prompt processing and output generation on different machines, because the two phases stress hardware differently.
- [1] Kernels and compilers. Engines speed up models with optimized GPU code, such as FlashAttention for the attention step, and by combining many small operations into fewer, larger ones.

Check: Serve an open-weight model with vLLM or SGLang, and call it through its OpenAI-compatible API. Load-test it at increasing concurrency and plot throughput and p95 time to first token. Explain where latency starts to climb, and why.

### 13.6 Quantization [2]

From Layer 6, Inference and serving.

Quantization shrinks a model so it fits in less memory and runs faster, at some cost in quality. The number formats themselves are covered in 13.3 Accelerators.

- [2] What it is. Quantization stores weights at lower precision. Some methods also quantize activations and the KV cache. It cuts memory use and usually speeds up generation, but it can lower quality.
- [2] Common formats:
  - GGUF, used by llama.cpp and Ollama, with named levels such as Q4_K_M and Q8_0
  - GPTQ and AWQ, for 4-bit weights on GPUs
  - FP8, on recent data center GPUs
  - 4-bit floating-point formats, on the newest GPUs
- [2] Quality impact. 8-bit is usually close to the original. 4-bit loses a little more, and the loss varies by model and task. Small models, math, long contexts and less common languages tend to suffer most. Always evaluate a quantized model on your own task.
- [2] Weight-only versus full quantization. Quantizing only the weights saves memory and speeds up generation when the model serves one user at a time. Quantizing activations as well speeds up the math itself, which matters most when the model serves many users at once.
- [2] Quantization-aware training. Some models are trained for low precision or released at it, which loses less quality than quantizing a finished model.
- [2] Whose quantization. Community-made quantized versions vary in quality. Providers may also serve quantized versions of open models. This is one reason the same model can score differently across providers (see 13.16 Evaluation in compute).

Check: Run one open-weight model at 16-bit, 8-bit and 4-bit precision. Compare memory use, output speed and quality on your evaluation set, and choose the precision you would ship.

### 13.7 Structured generation [2]

From Layer 6, Inference and serving.

This section covers how engines guarantee output formats. How to use structured output in a request is covered in 1.1 Working with a model.

- [2] How schema enforcement works. At each step, the engine blocks any token that would break the required format, such as a JSON schema, a regular expression or a grammar. This guarantees the output matches the format, unless generation is cut off. Libraries such as XGrammar, Outlines and llguidance do this inside engines.
- [2] Limits. Providers support only part of the JSON Schema standard, so they reject or simplify some schemas. A new schema can add delay to its first request while the provider prepares it. Forcing a format can also lower answer quality if the model has no room to reason first.
- [2] Tool-call parsing. Tool calls use the same machinery: the engine turns the model's output into structured calls. A self-hosted engine needs the right parser for each model family. With the wrong one, tool calls come back as plain text.

Check: Call a model with a JSON schema enforced, either through a provider or on your own engine. Measure the extra delay on the first request with a new schema. Compare answer quality with and without enforcement on 50 examples.

### 13.8 Faster generation [2]

From Layer 6, Inference and serving.

These techniques speed up output without changing the model.

- [2] Speculative decoding. A small, fast draft model guesses the next several tokens, and the large model checks all of them in one step, keeping the ones it agrees with. The output is exactly what the large model would have produced alone, but it arrives faster when the guesses are good. It helps most when the output is predictable, such as code or text that repeats the input. It speeds up single requests the most; under heavy load, the gains shrink.
- [2] Predicted outputs. Some providers let you pass in the output you expect, such as the file you're editing. The model can then move quickly through the parts that don't change, which speeds up edits a lot.
- [1] Multi-token prediction. Some models are trained to predict several tokens at once, and engines can use this to generate faster.

Check: For an editing task, try speculative decoding on a self-hosted engine or predicted outputs with a provider. Measure how output speed changes at low load and at high load.

### 13.9 Multi-GPU and clusters [1, some topics 2]

From Layer 9, Compute.

Large models don't fit on one GPU, so they run across several. Training runs use thousands.

- [2] Why models span GPUs. When the weights plus the KV cache exceed one GPU's memory, the model is split across several GPUs. Splitting within one server works far better than splitting across servers, because the links inside a server are much faster.
- [1] Interconnects. NVLink connects GPUs inside a server or rack, and it is many times faster than PCIe or the network between servers. InfiniBand or high-speed Ethernet connects servers to each other. Rack-scale systems, such as NVIDIA's NVL72 racks, link 72 GPUs so they work almost like one large GPU.
- [1] Ways to split work. Tensor parallelism splits each layer across GPUs and needs the fastest links. Pipeline parallelism gives each GPU a block of layers. Data parallelism runs copies of the model on different data. Expert parallelism spreads a mixture-of-experts model's experts across GPUs. How serving and training use these methods is covered in 13.5 Serving engines and batching and 12.3 Supervised fine-tuning.
- [1] Servers. A typical AI server holds 8 GPUs with fast links between them, plus CPUs, system memory and local storage. Loading weights from storage takes up a large part of a model's startup time.
- [2] GPUs in Kubernetes. GPU node pools, requesting GPUs for a container, and NVIDIA's GPU Operator. Research and batch clusters often use Slurm or Ray instead. Learn this to level 2 if your team self-hosts; otherwise level 1 is enough.
- [1] Sharing a GPU. One GPU can be split into isolated slices (NVIDIA's Multi-Instance GPU, or MIG) or shared over time. This is useful for small models that don't need a whole GPU.
- [1] Failures at scale. In large clusters, GPUs and links fail routinely. Long training jobs therefore save checkpoints often and restart automatically.

Check: Work out how many 80 GB GPUs a 405-billion-parameter model needs at 8-bit precision, leaving room for the KV cache. Explain why those GPUs should all be in one server.

### 13.10 Security in inference and serving [2–3]

From Layer 6, Inference and serving.

- [3] Exposed endpoints. Self-hosted engines often have no authentication by default. Never expose one directly to the internet. Put it behind your own authentication and network rules.
- [2] Shared caches. A prefix cache shared between users can leak information through timing: a fast response shows that someone recently sent the same prefix. Providers typically keep caches separate for each organization. Self-hosted systems that serve several customers should do the same.

### 13.11 Governance and safety in inference and serving [2]

From Layer 6, Inference and serving.

- [2] Serving logs. Engines and proxies can log full prompts and outputs, which contain user data. Decide what gets logged and how long it's kept.

Where inference runs, and the data residency that follows from it, is covered in 13.15 Governance and safety in compute.

Cost and latency have no separate section in Layer 6. Latency is covered in 1.5 Inference performance. Cost per token and self-hosting costs are covered in 13.2 Compute arithmetic and 13.12 Cost and latency in compute, and model prices and cost per task in 1.7 Cost and latency in foundation models. For Evaluation, changes to how a model is served can change its quality. This is covered in 13.16 Evaluation in compute and 13.6 Quantization. Data quality has nothing specific to this layer.

### 13.12 Cost and latency in compute [3]

From Layer 9, Compute. Focus on self-hosting versus APIs.

- [3] Self-hosting versus APIs. Compare an API's price per million tokens with your own cost per million tokens at realistic utilization, and add engineering and operations time. Self-hosting usually wins only in four cases: steady high volume, strict data rules, a model that no API offers, or latency needs that an API can't meet.
- [3] Utilization. An idle GPU costs as much as a busy one. The utilization you can sustain decides your real cost per token.
- [3] Right-sizing. Pick the cheapest hardware that fits the model and meets your latency target. For small models, older or smaller GPUs are often cheaper per token.
- [2] Benchmarking. Measure throughput, time to the first token and cost using your own prompts and traffic pattern. Don't rely on the peak figures in spec sheets or vendor charts.
- [2] Price trends. GPU rental prices have fallen as supply has grown. The price of a given level of model capability has dropped steeply year over year. Plan on features that are too expensive today becoming affordable later.

### 13.13 Reliability in compute [2]

From Layer 9, Compute. Learn this if you self-host.

- [2] Provider capacity incidents. A provider can run short of capacity or go down entirely. Spreading traffic across providers or regions helps; how to do that is covered in 8.3 Fallbacks and retries and 8.6 Routing.
- [2] Headroom. GPUs are scarce and slow to start, so you can't add them in minutes the way you can add ordinary servers. Keep spare capacity for traffic peaks.
- [2] Spot interruptions. Batch and fine-tuning jobs on spot capacity must save checkpoints and resume after an interruption.
- [1] Hardware failures. GPUs fail more often than ordinary servers. Managed services hide this from you, but self-hosting doesn't.

### 13.14 Observability in compute [2]

From Layer 9, Compute. Learn this if you self-host.

- [2] GPU metrics. Memory use, utilization, power and temperature, from tools such as nvidia-smi and NVIDIA's DCGM. "GPU utilization" only reports the share of time the GPU was running any work, not how hard it was working. Track tokens per second and cost per token as well.

### 13.15 Governance and safety in compute [2]

From Layer 9, Compute.

- [2] Data residency. Where the hardware sits decides where data is processed. Check which regions a provider runs a model in. Some providers can route requests to other regions to handle load unless you choose a regional option.
- [1] Sovereign and on-premises requirements. Some governments and industries require the hardware to be in a particular country or on their own premises.

Energy, carbon and export controls are covered in 13.18 Power and data centers.

### 13.16 Evaluation in compute [2]

From Layer 9, Compute.

- [2] Same model, different results. The same open-weight model can score differently on different hardware, at different precision, or from different providers, because of differences in quantization and implementation. Re-run your evaluations whenever you change where a model runs.

### 13.17 Security in compute [1]

From Layer 9, Compute.

- [1] Shared hardware. On cloud and serverless platforms, your workloads share physical machines with other customers. Know what isolation your provider guarantees.
- [1] Confidential computing. Recent GPUs can run workloads in encrypted enclaves whose setup can be verified, so even the cloud operator can't see the data. Some teams use this for very sensitive data.

Data quality has nothing specific to this layer.

### 13.18 Power and data centers [1]

From Layer 9, Compute. You can read about power and data centers at any point. The material is light, and it explains the prices and shortages you'll see along the way.

This is the physical base under everything else. You won't build any of it, but it explains prices, shortages and where capacity exists.

- [1] Power. Power is now the main limit on new AI capacity. Large AI data center campuses need hundreds of megawatts to gigawatts, and new grid connections can take years. This shapes where capacity appears and how fast it grows.
- [1] Density and cooling. The newest racks each draw over 100 kW and need liquid cooling, so many older data centers can't host them.
- [1] Location. Capacity clusters where power is cheap and available. Distance to users matters for real-time features, but not for training or batch work.
- [1] Supply chain. Advanced chips depend on a few suppliers. TSMC makes most of them, and three companies make HBM: SK hynix, Samsung and Micron. Advanced packaging capacity is also limited. A bottleneck at any of these causes shortages.
- [1] Export controls. Governments restrict where advanced chips can be sold. This affects which providers and models can operate in which countries.
- [1] Energy, carbon and water. AI data centers use a lot of electricity, and some also use a lot of water for cooling. Estimates of energy per query vary widely. Know enough to give a stakeholder a rough, honest answer that includes the uncertainty.
- [1] Scale trends. The compute used to train frontier models has grown several-fold each year, and the largest clusters now hold hundreds of thousands of GPUs. This affects how often new models arrive and how much better they get.

Check: Explain to a stakeholder why the newest GPUs are scarce, and why the price of a given level of model capability keeps falling.

### Key reading for Stage 13

- Horace He, "Making Deep Learning Go Brrrr From First Principles" (2022), on compute-bound versus memory-bound work.
- Carol Chen, "Transformer Inference Arithmetic" (kipp.ly, 2022), on memory, bandwidth and KV-cache math.
- EleutherAI, "Transformer Math 101" (2023), on memory and compute for training and inference.
- Google DeepMind, "How to Scale Your Model" (2025), on rooflines, accelerators and parallelism.
- Modal, "GPU Glossary", a plain-language guide to GPU hardware and software terms.
- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 9 covers inference optimization, including AI accelerators.
- Stas Bekman, *Machine Learning Engineering Open Book* (GitHub), a practical reference on accelerators, networks and debugging.
- International Energy Agency, "Energy and AI" (2025), on data center power demand.
- Epoch AI's data on hardware, training compute and power trends.
- Yu et al., "Orca: A Distributed Serving System for Transformer-Based Generative Models" (2022), which introduced continuous batching.
- Zheng et al., "SGLang: Efficient Execution of Structured Language Model Programs" (2024), on prefix caching and structured generation.
- Leviathan et al., "Fast Inference from Transformers via Speculative Decoding" (2023).
- Maarten Grootendorst, "A Visual Guide to Quantization" (2024).

## Stage 14. Each concern across the stack

Every cross-cutting concern applies at every layer, and the stages above cover each one where it first matters. This stage brings each concern back together, with a map of where it's covered in every layer. Each map ends with a check that follows one feature through every layer it touches.

### 14.1 Evaluation at each layer

From the Evaluation concern.

Each layer's sections cover evaluation as it applies to that layer. This table shows where.

| Layer | What to evaluate there | Level | Where |
|---|---|---|---|
| 1. Product and UX | The whole experience as users get it, product metrics, experiments, and whether offline gains reach users | 3 | 4.4 Feedback and measurement, 4.10 Evaluation in product and UX |
| 2. Agents and orchestration | Task success checked in the end state, success on every one of several runs, trajectories, each step on its own, cost per success | 3 (some topics 4) | 7.4 Evaluation in agents and orchestration |
| 3. Tools and protocols | The right tools with the right arguments, mock tools and replayed responses, cases with injected instructions | 3 | 5.13 Evaluation in tools and protocols |
| 4. Context, memory and retrieval | Which step failed, retrieval metrics, faithfulness and citations, memory across sessions | 3 (some topics 4) | 6.3 Evaluation in context, memory and retrieval |
| 5. Model gateway and routing | Each route and fallback, on the traffic it gets, slice by slice | 3 | 8.8 Evaluation in model gateway and routing |
| 6. Inference and serving | Quality lost to quantization or to schema enforcement | 2 | 13.6 Quantization, 13.7 Structured generation |
| 7. Model adaptation | Beating the best prompt and a stronger model, regressions outside the task, safety after fine-tuning | 3 | 12.9 Evaluation in model adaptation, 12.11 Governance and safety in model adaptation |
| 8. Foundation models | Choosing and switching models on your own task, choosing a judge model | 3 (choosing models 4) | 3.17 Choosing models, 3.18 Evaluation in foundation models |
| 9. Compute | The same model on different hardware or providers | 2 | 13.16 Evaluation in compute |
| 10. Data | Evaluation datasets, error analysis, labeling, judge calibration sets, synthetic cases, graders | 4 | 3.3 Human labeling, 3.4 Evaluation: building evaluation datasets, 3.11 Synthetic data, 12.7 RL environments |

Check: For one feature, list every layer it touches and what you would evaluate at each. Mark which of those your evaluation covers today, and add cases for the most important gap.

### 14.2 Observability at each layer

From the Observability concern.

Each layer's sections cover observability as it applies to that layer. This table shows where.

| Layer | What to observe there | Level | Where |
|---|---|---|---|
| 1. Product and UX | What users experienced in the client, and links from complaints and feedback to traces | 3 | 4.12 Observability in product and UX |
| 2. Agents and orchestration | Whole runs traced as trees, reading runs, task status, replaying runs | 3 | 7.10 Observability in agents and orchestration |
| 3. Tools and protocols | Every tool call in the trace, audit logs of actions, tool usage, redaction | 3 | 5.15 Observability in tools and protocols |
| 4. Context, memory and retrieval | The assembled context of every call, retrieval in production, index health | 3 | 6.16 Observability in context, memory and retrieval |
| 5. Model gateway and routing | Routes, retries, fallbacks, cache hits, cost tags and trace IDs, recorded at the gateway | 3 | 8.9 Observability in model gateway and routing |
| 6. Inference and serving | Time to first token and output speed as percentiles, serving-engine metrics | 2–3 | 1.12 Observability in inference and serving |
| 7. Model adaptation | Training metrics next to real outputs, experiment tracking | 2 | 12.13 Observability in model adaptation |
| 8. Foundation models | What to log for each model call: model, snapshot, settings, tokens, stop reason, latency | 3 | 1.11 Observability in foundation models |
| 9. Compute | GPU metrics | 2 | 13.14 Observability in compute |
| 10. Data | Full traces tied to feedback, turning traces into datasets, drift in what users ask | 3 (some topics 4) | 11.2 Product feedback |

Check: For one feature, list every layer it touches and what its traces record at each. Mark what's missing, and add the most important gap.

### 14.3 Security at each layer

From the Security concern.

Each layer's sections cover security as it applies to that layer. This table shows where.

| Layer | What to secure there | Level | Where |
|---|---|---|---|
| 1. Product and UX | Rendering output safely, abuse through the product, untrusted uploads, shared links | 3 | 4.8 Security in product and UX |
| 2. Agents and orchestration | Trust between agents, separating reading from acting, permissions for each agent, goal drift | 3 | 7.13 Security in agents and orchestration |
| 3. Tools and protocols | Untrusted tool results, the dangerous combination, tool poisoning, designs that resist injection, permissions, sandboxes | 3 (some topics 4) | 5.10 Security in tools and protocols, 5.7 Permissions, 5.12 Code execution and sandboxes |
| 4. Context, memory and retrieval | Indirect injection, permission-aware retrieval, keeping customers apart, memory poisoning, leaks through output | 3 | 6.10 Security in context, memory and retrieval |
| 5. Model gateway and routing | Provider keys and secrets management, limits per user, denial of wallet, the gateway as a checkpoint | 2–3 | 8.2 API keys and access, 8.4 Rate limits and quotas, 8.10 Security in model gateway and routing |
| 6. Inference and serving | Exposed self-hosted endpoints, caches shared between customers | 2–3 | 13.10 Security in inference and serving |
| 7. Model adaptation | Fine-tuning as an attack | 2 | 12.12 Security in model adaptation |
| 8. Foundation models | Why models fall for injection, jailbreaks, system prompt leaks, trusting model files | 2–3 | 1.10 Security in foundation models |
| 9. Compute | Shared hardware, confidential computing | 1 | 13.17 Security in compute |
| 10. Data | Secrets and personal data leaking into datasets, data poisoning, feedback manipulation | 2 | 11.6 Security in the data layer |

Check: For one feature, list every layer it touches and the risks that layer's sections name. Mark which have a control in code today, and add one for the most serious gap.

### 14.4 Governance and safety at each layer

From the Governance and safety concern.

Each layer's sections cover governance and safety as it applies to that layer. This table shows where.

| Layer | What to govern there | Level | Where |
|---|---|---|---|
| 1. Product and UX | Users' controls over their data, sensitive situations, telling people they're dealing with AI, respecting users, accessibility | 2–3 | 4.7 Governance and safety in product and UX, 4.6 Setting expectations |
| 2. Agents and orchestration | Where people step in, stopping agents, records of decisions, unattended agents | 3 (some topics 4) | 7.6 Handoffs to people, 7.14 Governance and safety in agents and orchestration |
| 3. Tools and protocols | Which actions need approval, rules for human oversight, approved tools and servers, accountability, terms of the sites agents use | 3 (some topics 4) | 5.5 Actions and side effects, 5.17 Governance and safety in tools and protocols |
| 4. Context, memory and retrieval | Personal data in indexes and memories, deletion, consent for memory, rights to indexed content, where data lives | 3 | 6.15 Governance and safety in context, memory and retrieval |
| 5. Model gateway and routing | Approved models and regions, removing personal data before it leaves, a record of which provider handled each request | 2–3 | 8.11 Governance and safety in model gateway and routing, 8.10 Security in model gateway and routing |
| 6. Inference and serving | What serving logs keep | 2 | 13.11 Governance and safety in inference and serving |
| 7. Model adaptation | Safety after fine-tuning, licenses, provider review, becoming a provider under the law, model cards | 2–3 | 12.11 Governance and safety in model adaptation |
| 8. Foundation models | Usage policies, provider and deployer roles, bias, licenses of open models, who owns outputs | 2 | 1.13 Governance and safety in foundation models, 13.1 Open-weight models |
| 9. Compute | Data residency, sovereign hosting, energy and carbon | 1–2 | 13.15 Governance and safety in compute, 13.18 Power and data centers |
| 10. Data | Personal data in datasets, consent and retention, provider data policies, dataset licenses, data cards | 2–3 | 11.5 Governance and safety in the data layer, 11.4 Data quality in the data layer |

Check: For one feature, list every layer it touches and the governance duties that layer's sections name. Mark which are met today, with evidence for each, and close the most important gap.

### 14.5 Reliability at each layer

From the Reliability concern.

Each layer's sections cover reliability as it applies to that layer. This table shows where.

| Layer | What to keep working there | Level | Where |
|---|---|---|---|
| 1. Product and UX | The product working without AI, never losing the user's work, clear messages when AI can't help, variety or consistency | 3 | 4.11 Reliability in product and UX, 4.2 Designing for wrong answers |
| 2. Agents and orchestration | Failing well, timeouts for whole tasks, falling back, resuming after crashes, compounding errors | 3 | 7.12 Reliability in agents and orchestration, 7.5 State and long-running tasks, 7.1 Workflows and agents |
| 3. Tools and protocols | Retrying tools, protecting downstream systems, tools that are down, actions that are safe to repeat, changing tools safely | 3 | 5.14 Reliability in tools and protocols, 5.5 Actions and side effects |
| 4. Context, memory and retrieval | When search fails, rebuilding indexes without downtime | 2–3 | 6.18 Reliability in context, memory and retrieval |
| 5. Model gateway and routing | Retries, fallbacks, circuit breakers, hedged requests, rate limits, gradual rollouts | 3 (some topics 4) | 8.3 Fallbacks and retries, 8.4 Rate limits and quotas, 8.6 Routing |
| 6. Inference and serving | How calls fail, timeouts and cancellation, failures of self-hosted engines, health checks and turning away excess load | 2–3 | 1.9 Reliability in inference and serving, 1.6 Calling models through an API, 13.4 Running models yourself |
| 7. Model adaptation | A prompt-based fallback for every fine-tune, retraining and retirement | 3 | 12.5 Owning an adapted model |
| 8. Foundation models | Why outputs vary, stop reasons, pinned versions and retirement dates, a tested second-choice model | 3 | 1.8 Reliability in foundation models, 1.1 Working with a model, 1.4 Frontier proprietary models |
| 9. Compute | Provider capacity incidents, headroom, spot interruptions | 2 | 13.13 Reliability in compute |
| 10. Data | Datasets and environments that can be reproduced | 2–3 | 3.2 Data foundations, 12.7 RL environments |

Check: For one feature, list every layer it touches and how that layer's sections say to handle failure there. Mark which are in place and tested, and fix the most important gap.

### 14.6 Cost and latency at each layer

From the Cost and latency concern.

Each layer's sections cover cost and latency as they apply to that layer. This table shows where.

| Layer | What to make cheaper or faster there | Level | Where |
|---|---|---|---|
| 1. Product and UX | Speed for each kind of interaction, cost per user, limits users understand, AI nobody asked for, precomputing | 3 | 4.9 Cost and latency in product and UX |
| 2. Agents and orchestration | Cost per task, budgets the agent knows about, a model for each step, steps in sequence | 3 | 7.11 Cost and latency in agents and orchestration |
| 3. Tools and protocols | Round trips, tool definitions and results in the context, slow tools, sandbox and API costs | 3 | 5.16 Cost and latency in tools and protocols |
| 4. Context, memory and retrieval | Context size, conversation history that grows, retrieval latency, indexing and storage costs | 3 (some topics 4) | 6.17 Cost and latency in context, memory and retrieval, 2.2 Context engineering, 2.3 Conversation history |
| 5. Model gateway and routing | Routing, cascades, caching, reasoning effort for each request, cost attribution, budgets and caps | 3 (some topics 4) | 8.6 Routing, 8.7 Caching, 8.5 Spending caps and cost control |
| 6. Inference and serving | Time to first token, output speed, percentiles, latency budgets, prompt caching, faster generation | 3 | 1.5 Inference performance, 2.4 KV-cache management, 13.8 Faster generation |
| 7. Model adaptation | When adapting a model pays off, how training is billed, distillation | 3 | 12.10 Cost and latency in model adaptation, 12.4 Distillation |
| 8. Foundation models | How models are priced, cost per task, prompt caching, latency by model, choosing models | 3 (choosing models 4) | 1.7 Cost and latency in foundation models, 3.17 Choosing models |
| 9. Compute | Cost per token, self-hosting versus APIs, utilization, batch and reserved capacity | 3 | 13.2 Compute arithmetic, 13.12 Cost and latency in compute, 8.12 Cloud capacity |
| 10. Data | Costs of labeling, synthetic data and fine-tuning runs, the smallest dataset that works | 2–3 | 11.7 Cost in the data layer |

Check: For one feature, list every layer it touches and the cost and latency levers that layer's sections name. Mark which you've used, and try the most promising one you haven't.

### 14.7 Data quality at each layer

From the Data quality concern.

Four layers' sections cover data quality as it applies to that layer. This table shows where.

| Layer | What to keep trustworthy there | Level | Where |
|---|---|---|---|
| 2. Agents and orchestration | Checks between steps, so one step's bad output doesn't spread to the next | 3 | 7.2 Workflow patterns |
| 3. Tools and protocols | Checking tool arguments and results, and dating results where freshness matters | 3 | 5.2 Function calling, 5.17 Governance and safety in tools and protocols |
| 4. Context, memory and retrieval | Freshness, duplicates and versions, source quality, lineage for chunks, gaps, parsing, memories that go wrong | 3 | 6.14 Data quality in context, memory and retrieval, 6.4 Preparing documents, 6.9 Long-term memory |
| 10. Data | The parts of quality, validation, lineage for examples, data cards, decontamination, staleness, wrong labels, drift | 3 | 11.4 Data quality in the data layer, 11.2 Product feedback |

Layers 1, 7 and 8 point to Layer 10's sections for their data quality, and Layers 5, 6 and 9 have nothing specific to it.

Check: For one feature, list every layer it touches and the data each layer passes on. Mark which data is checked where it enters today, and add a check at the most important gap.

## Outside this curriculum

Each layer and concern names some work it leaves to other people. This list collects those items.

### Lab work

- Pre-training models from scratch and running RLHF at lab scale are lab work. Awareness is enough. (Layer 7)
- Training foundation models from scratch is lab work. An application engineer only needs to know how it's done in outline. (Layer 8)
- Building public benchmarks and testing frontier models for dangerous capabilities are lab work. Awareness is enough. (Evaluation)
- Safety training of models and testing frontier models for dangerous capabilities are lab work. Awareness is enough. (Governance and safety)
- Attacks on models during training, such as poisoning pre-training data or stealing model weights, are lab and provider work. Awareness is enough. (Security)

### Infrastructure and platform work

- Writing GPU kernels and building serving engines are infrastructure work. Awareness is enough. (Layer 6)
- Writing GPU kernels (in CUDA or Triton) and designing clusters are infrastructure jobs. An application engineer only needs to know they exist. (Layer 9)
- Running the observability pipeline itself, such as collectors and storage clusters, is platform work. Knowing how data flows through it is enough. (Observability)
- Running infrastructure, such as clusters, networks and data centers, and company-wide incident processes belong to site reliability engineering, a discipline of its own. This curriculum covers what changes when AI is involved. (Reliability)

### Other disciplines

- Visual design, front-end engineering and product management are disciplines of their own. This curriculum covers what changes when AI is involved. (Layer 1)
- Classic application and network security, and company-wide security programs, are disciplines of their own. This curriculum covers what changes when AI is involved. (Security)
- Interpreting laws and contracts is legal work. This curriculum covers recognizing when they apply and building what they require. (Governance and safety)
- Pricing the product itself is a product and finance decision. This curriculum covers the costs that the pricing has to cover. (Cost and latency)
- Building data pipelines, warehouses and catalogs is data engineering, a discipline of its own. This curriculum covers what changes when AI reads and writes the data. (Data quality)
