# AI stack: ten layers and seven cross-cutting concerns

Read the stack from top to bottom. It starts with what users touch and ends with the raw inputs everything is built from. Each layer depends on the layers below it. The cross-cutting concerns are not layers. They apply to every layer and have to be handled at each one.

## Layers, top to bottom

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

## Across every layer

- Evaluation. This measures whether the system does its job, using test sets, automated graders (including models that judge other models), human review, and A/B tests in production. It applies everywhere, from choosing a model to checking retrieval quality and whether an agent finished its task.
- Observability. This shows what the system is actually doing. It uses traces that follow a request through agents, tools and model calls, plus logs and metrics for finding and fixing failures.
- Security. This defends against attackers. It covers prompt injection, authentication and authorization (including what each agent is allowed to do), secrets management, and preventing data from being stolen.
- Governance and safety. This makes sure the system behaves acceptably even when no one is attacking it. It covers guardrails against harmful or biased output, privacy and handling of personal data, regulatory compliance, and audit trails. It also covers intellectual property, where data came from, and human oversight of what agents do.
- Reliability. This keeps the system working when parts fail. It covers retries, switching between models and providers, uptime targets, falling back to reduced functionality instead of failing outright, and keeping model outputs consistent.
- Cost and latency. This keeps the system affordable and fast. It covers cost per request or task, token and GPU spending, and response time, all weighed against quality.
- Data quality. This keeps the information moving through the system trustworthy. It covers accuracy, freshness, lineage (where data came from) and how data changes over time. It applies equally to training data, retrieved documents, tool outputs and evaluation sets.

## How the parts connect

A request enters through Product and UX. The agent layer plans the work, calling tools and pulling in context as needed. Each model call goes through the gateway to inference, which runs a foundation model or an adapted model on compute. Feedback travels the other way: what users do in the product becomes data, and that data improves the models through adaptation.
