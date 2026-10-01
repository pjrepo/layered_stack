# Layer 8, Foundation models: what an application-layer AI engineer needs to know

This file covers the Foundation models layer from `ai_stack.md`: the pre-trained models everything else builds on, including frontier proprietary models, open-weight models, multimodal models, and the embedding models used for search. It is for an AI engineer who builds products on existing models (layers 1 to 5) and doesn't train them from scratch. It lists the concepts in this layer and how deeply to learn each one.

## The rule for depth

You don't build foundation models, but everything you build runs on them. The model is your most important dependency. Know it the way a backend engineer knows their database: what it does well, where it breaks, what it costs and how to replace it.

Go deepest on behavior and choice: what models can and can't do, and how to pick one by testing it on your own task. Learn the model's interface to level 3, because you use it in every call. Learn how models work inside, such as attention and training stages, to level 2. That's enough to explain their behavior, not to build one.

Most decisions in this layer come down to one question: which model does this job well enough, cheaply enough and fast enough? Answer it by testing on your own task. Benchmarks and price lists only narrow the list. New models arrive every few months, so you'll answer this question again and again. Build your product so that switching models is routine.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| How models work | 2 | Explains model behavior; labs handle the internals |
| Working with a model | 3 | You use this interface in every call |
| Capabilities and limits | 3 (some topics 4) | Knowing where models break is the core of this layer |
| Frontier proprietary models | 3 | Most products run on them |
| Open-weight models | 2 (some topics 3) | Worth it when control, privacy or cost at scale matter |
| Multimodal models | 2–3 | Level 3 for the input types your product handles |
| Embedding models | 3 | Search quality starts with the embedding model |
| Choosing models | 4 | Your most important recurring decision in this layer |
| Cost and latency | 3 | Price per token misleads; cost per task is what counts |
| Reliability | 3 | The same input can give different outputs |
| Security | 2–3 | Models can't reliably tell instructions from data |
| Evaluation | 3 | Model judges favor answers like their own |
| Observability | 3 | Stop reasons and token counts explain many surprises |
| Governance and safety | 2 | Licenses, usage policies and laws limit what you can ship |

## How models work [2]

You won't build a model, but a working picture of how one works explains most of what it does.

- [2] Next-token prediction. A language model reads a sequence of tokens and gives a probability for every possible next token. It picks one, adds it to the sequence and repeats. Everything a chat model does is produced this way, including answering, reasoning and calling tools.
- [2] The transformer. Tokens are turned into vectors, passed through a stack of layers, and come out as next-token probabilities. Each layer uses attention, which lets every token draw on every earlier token. Attention is why models use context so well, and why cost grows with context length.
- [2] Training stages. Pre-training on huge amounts of text produces a base model that continues text but doesn't follow instructions. Post-training then turns it into an assistant, using three methods:
  - supervised fine-tuning on example conversations
  - preference tuning on human or AI judgments of which answer is better
  - reinforcement learning on tasks with checkable answers

  Safety training teaches the model what to refuse. How these methods work belongs to Layer 7. Here, you only need to know what each stage does to the model's behavior.
- [2] Where quirks come from. Many quirks trace back to a training stage:
  - Preference tuning rewards answers people like. That can make models flatter and agree with the user (sycophancy), and write at length.
  - Reinforcement learning on checkable tasks makes models strong at math and code. It sometimes also teaches them to game tests.
  - Safety training causes refusals, including some unnecessary ones.
- [2] Base and instruction-tuned models. Base models only continue text. Instruction-tuned (chat) models follow instructions in a conversation format. Almost all products use instruction-tuned models.
- [2] Dense and mixture-of-experts models. A dense model uses all its parameters for every token. A mixture-of-experts (MoE) model has many sub-networks, called experts, and sends each token through only a few of them. So it has many parameters in total but uses only some for each token, which gives more quality for the same compute. Many recent large models use this design. What it means for memory and speed is covered in `layer_09_compute.md`.
- [2] Model size. More parameters generally mean more knowledge and better reasoning, at higher cost and latency. Well-trained small models can match much larger older models on many tasks. They are often trained to imitate larger ones (distillation).
- [2] Encoder models. Models such as BERT read text and produce vectors or labels without generating text. They are small, fast and cheap. They are still common for classification and as the base of embedding models.
- [1] Other architectures. State-space and hybrid models handle long inputs more cheaply. Diffusion language models generate many tokens at once instead of one at a time.
- [1] Interpretability. Research into what happens inside models, such as finding the internal features that represent concepts. It explains some behavior but isn't yet a practical tool for application teams.

Check: Pick three quirks you've seen in a model you use, such as flattering the user, refusing a harmless request or gaming a test. For each one, explain which training stage most likely caused it.

## Working with a model [3]

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
- [3] Structured output. Models can return JSON that matches a schema you supply. With strict schema enforcement, the provider guarantees that the output matches, unless the response is cut off or refused. Without it, models usually but not always produce valid JSON. Keep schemas simple. When quality matters, let the model reason before it fills in the structure. How providers enforce schemas belongs to Layer 6.
- [3] Reasoning controls. Reasoning models think before they answer, and you pay for the thinking tokens as output. You can usually set how much they think, with an effort level or a token budget. Some providers show the reasoning, some summarize it and some hide it.
- [3] Stop reasons. Every response reports why it ended: it finished, hit the output limit, called a tool, or was refused or blocked. Check this on every call, because a response cut off by the output limit can look complete.
- [2] Token probabilities. Some APIs return the probability of each output token (logprobs). Where they're available, they're a useful confidence signal for classification.
- [2] Provider-specific features. Providers add built-in tools, such as web search, code execution and file handling. Some can also store conversations on their side. These features save work but tie you to one provider.

Check: Write a small script that calls a model with a system prompt and a JSON schema. Count the tokens before sending, and check the stop reason on every response. Run the same request 10 times at each of two temperatures and compare how much the output varies.

## Capabilities and limits [3, some topics 4]

Knowing what models do well and where they break is the core of this layer. Learn the general patterns below. Then learn the details of the models you ship with from your own evaluations and traces.

- [4] Uneven abilities. Models can be excellent at hard tasks and still fail at easy ones. Doing well on one task doesn't mean a model will do well on a related one. For the models you ship, know their specific strengths and failures from your own tests, not from their reputation.
- [3] Instruction following. Models follow the system prompt and user instructions, but not perfectly. Recent models follow instructions more literally, so vague instructions get vague results, and prompts written for older models may need rewriting. When instructions conflict, models are trained to rank the system prompt above the user, and the user above text in documents and tool results. That ranking doesn't always hold.
- [3] Learning from examples. Models can pick up a task from a few examples in the prompt (in-context learning, or few-shot prompting). They copy the examples' format, length and style, including their mistakes. So choose examples carefully and vary them.
- [3] Reasoning. Working step by step improves results on math, code, planning and other multi-step problems, but it adds cost and latency. Reasoning models do this automatically, so telling them to "think step by step" adds little. For simple lookup, extraction and classification tasks, extra reasoning rarely helps.
- [2] Faithfulness of reasoning. The reasoning a model shows doesn't always match how it actually reached its answer. Treat it as a useful hint, not a reliable explanation.
- [3] Long context. Models accept very long inputs but use them unevenly. Quality drops as the input grows, and material in the middle of a long input is easier to miss. Finding one fact in a long document (a "needle in a haystack") is much easier than reasoning across many parts of it. Test your model at the context lengths you actually use, and keep context lean.
- [3] Knowledge and hallucination. Models store a lot of knowledge but recall it unreliably. They are weakest on rare facts, recent events, exact numbers, quotes and citations. When a model doesn't know something, it often gives a confident, plausible answer anyway. This is called hallucination. Give the model the facts it needs in the context, ask it to cite them, and allow it to say it doesn't know. Knowledge cutoffs are covered in `layer_10_data.md`.
- [3] Common failure patterns:
  - Models agree with the user too readily (sycophancy).
  - Their results can change with small changes in wording (prompt sensitivity).
  - They favor options because of where those options sit in a list.
  - They write longer answers than needed.
  - In long tasks they stop early or skip steps.
  - In long conversations they drift from their instructions.
  - They sometimes refuse reasonable requests.
  - In coding tasks they sometimes game the tests.
- [3] Tool use and agentic tasks. Models differ in how reliably they pick the right tool, fill in its arguments and recover from errors. Recent models can carry out long, multi-step tasks with tools, such as fixing a bug across a codebase. The length of task they can complete reliably has been growing fast, but reliability still drops as tasks get longer. How function calling works belongs to Layer 3.
- [2] Confidence. When a model says how sure it is ("I'm 90% sure"), that number is poorly calibrated. Token probabilities are a better signal where available, but they aren't perfect either.
- [2] Refusals and safety filters. Models refuse requests that their training or their provider's policy treats as harmful, and they sometimes refuse harmless ones too. Providers may also run separate filters that block inputs or outputs. Learn how your models behave close to those limits for your use case.

Check: Write a one-page profile of a model you ship with. Cover its strengths on your task and its common failures, with a real example of each failure. Include the prompt or design changes that work around them.

## Frontier proprietary models [3]

Frontier models are the most capable models. Labs such as OpenAI, Anthropic, Google and xAI offer them through APIs, and most products run on them.

- [3] Families and tiers. Each lab offers a family of models in tiers: a large, most capable model, a balanced mid-sized one, and a small, fast, cheap one. For example, Anthropic's Claude comes in tiers such as Opus, Sonnet and Haiku, and Google's Gemini in tiers such as Pro and Flash. For the families you use, know the current tiers, their strengths, prices and limits.
- [2] The wider landscape. Know the other major families well enough to tell when one is worth testing. That means what each is known for, roughly how they compare, and how fast the rankings change.
- [3] Versions and aliases. Providers release dated snapshots, plus aliases that point to the latest snapshot. When an alias moves to a new snapshot, behavior can change without warning. Pin a snapshot in production and upgrade on purpose.
- [3] Retirement. Providers retire old models on a published schedule, sometimes with only a couple of months' notice. Track the retirement dates of every model you use and plan migrations early.
- [2] Access channels. You can often get the same model from its lab directly or through a cloud platform such as AWS Bedrock, Google Vertex AI or Microsoft Azure. Channels can differ in features, rate limits, regions, data terms and how soon new models arrive.
- [2] System cards. Labs publish a system card for each model, describing its capabilities, test results, known limits and safety testing. Read the card for every model you adopt.

Check: For the model family you use most, list the current tiers with their prices, context windows and main strengths. Add the snapshot you've pinned and its retirement date, if one has been announced.

## Open-weight models [2, some topics 3]

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
- [2] Where to find them. Hugging Face hosts most open-weight models and their model cards. You can run them yourself (see Layers 6 and 9) or call them through hosted providers.
- [2] Safety differences. Some open models have lighter safety training than API models, and anyone can fine-tune that training away. If you ship an open model, you may need your own guardrails.

Check: Pick an open-weight model that could replace an API model in one feature. Check its license and run your evaluation set on it. Then compare the cost of hosting it with the cost of the API.

## Multimodal models [2–3]

Multimodal models take in or produce more than text, such as images, documents, audio and video.

- [3] Image input. Models can read screenshots, photos, charts, diagrams and handwriting. They are weaker at counting objects, reading very small text, locating things precisely and reasoning about space. Images cost tokens, usually more at higher resolution.
- [3] Document input. Many models accept PDFs directly. They read both the text and an image of each page, including tables and charts. This often works better than extracting the text first, though it costs more tokens. Building document-parsing pipelines belongs to Layer 4.
- [2] Speech and audio. Speech-to-text models transcribe audio, and text-to-speech models produce natural voices. Native audio models take in and produce speech directly, which cuts delay in voice conversations and keeps tone of voice. Learn this to level 3 if you build voice products.
- [2] Image generation and editing. These models create or edit images from text instructions. Know what they do well, where their results are inconsistent, and the policies that restrict what they'll produce.
- [2] Computer use. Some models can operate a computer from screenshots, clicking and typing like a person. This is slower and less reliable than calling an API, so use it only when no API exists. The sandboxes these agents run in belong to Layer 3.
- [1] Video. Models can understand video input and generate short clips. Video generation is expensive and still maturing.

Check: Send the same 20 real documents or screenshots from your product to two models. Compare their accuracy and token cost, and list the cases that both get wrong.

## Embedding models [3]

Embedding models turn text, and sometimes images, into lists of numbers called vectors. Similar content gets similar vectors. Search systems are built on embedding models, and how those systems work belongs to Layer 4.

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
- [3] What embeddings miss. Embeddings capture topic and meaning, but they handle exact terms (such as product codes, names and IDs), numbers and negation poorly. Similar isn't the same as relevant. This is why search systems combine embeddings with keyword search and reranking (Layer 4).
- [2] Smaller vectors. Some models produce vectors you can shorten with little loss of quality (Matryoshka embeddings). Vectors can also be stored at lower precision. Both cut storage and search costs.
- [2] Rerankers. A reranker reads a query and a document together and scores how relevant the document is. It is more accurate than comparing embeddings but too slow to run on everything, so it's used to reorder the top results.
- [2] Other kinds of embeddings:
  - Sparse embeddings give weights to important words.
  - Multi-vector models, such as ColBERT, keep a vector for each token to match more finely.
  - Multimodal embeddings put images and text in the same space, so a text query can find images.

Check: Compare two embedding models on 50 real queries from your product whose correct documents you know. Report which model finds more of them and what each costs to store.

## Choosing models [4]

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
- [3] Staying swappable. Keep the choice of model in configuration, and avoid depending on one model's quirks. Keep your evaluation set ready so that switching is routine. Switching models at runtime belongs to Layer 5.
- [2] Leaderboards. Human-vote leaderboards such as LMArena rank models by which answers people prefer. Independent sites compare quality, price and speed, and task leaderboards cover areas such as coding and agents. Use them to build a shortlist. Why public benchmarks rarely predict your results is covered in `layer_10_data.md`.
- [2] Pace of progress. New frontier models arrive every few months, and the price of a given level of capability keeps falling. Expect today's limits to move, and re-test regularly. Avoid building elaborate workarounds for weaknesses the next model may fix.

Check: For one feature, compare three models on your evaluation set, including at least one cheaper tier. Pick one based on quality, cost per task and latency. Write down what would make you switch.

## Cross-cutting concerns in this layer

### Cost and latency [3]

- [3] Pricing structure. Models charge separately for input tokens, output tokens and cached input, and some charge more for very long inputs. Reasoning tokens are billed as output. Images and audio are converted to tokens and billed too.
- [3] Cost per task. Compare models by what a completed task costs, not by price per token. A model with cheaper tokens can cost more overall if it needs longer prompts, more reasoning, more retries or more tool calls. Use each model's own token counts when you compare.
- [3] Prompt caching. Providers charge much less for input that repeats the start of a recent prompt. This happens either automatically or when you mark content for caching. Put stable content first, such as instructions, tool definitions and reference documents, and put changing content last. How caching works belongs to Layer 6.
- [3] Latency. Time to the first token and output speed differ widely between models and providers. Reasoning adds delay before the first visible token. Use small, fast models where users are waiting.

### Reliability [3]

- [3] Consistency. The same input can give different outputs, even at temperature 0, because of how providers batch and compute requests. Use structured output, validate every response, retry failures, and test with repeated runs.
- [2] Outages. Models go down or slow down. Falling back to another model or provider belongs to Layer 5, but you need a tested second-choice model for each feature.

Pinned versions and retirement dates are covered under Frontier proprietary models.

### Security [2–3]

- [3] Prompt injection. Models can't reliably tell instructions from data. So text in a web page, document, email or tool result can take control of the model. Training reduces this but doesn't solve it. Assume any model that reads untrusted content can be hijacked, and limit what it's able to do. Defenses are covered in `concern_03_security.md`.
- [2] Jailbreaks. Users can talk models into breaking their rules. No model fully resists this, so don't rely on the model alone to enforce your policies.
- [2] System prompt leaks. Users can often get a model to reveal its system prompt, so never put secrets in it.
- [2] Trusting model files. Download open-weight models from official sources. Prefer the safetensors format over formats that can run code when loaded. Be aware that weights can also contain hidden, planted behaviors.

### Evaluation [3]

- [3] Choosing a judge model. When models grade outputs, they tend to favor answers from their own model family, longer answers and whichever option is shown first. Use a judge from a different family, or check it against human labels.

Using evaluations to choose models is covered under Choosing models.

### Observability [3]

- [3] What to log for each call. Record the model and snapshot, the settings, token usage (including cached and reasoning tokens), the stop reason and the latency. These explain most surprises in cost and quality.

### Governance and safety [2]

- [2] Usage policies. Every provider restricts what you may build with its models. Read the policy for each model you use.
- [2] Regulation. Laws such as the EU AI Act give different duties to companies that make models and to companies that build products with them. Know which role you're in, and bring in legal when it matters.
- [2] Bias. Models can reflect social and cultural biases, and can perform worse in some languages. Test your product across the user groups and languages you serve.
- [1] Output ownership. Provider terms say who owns the outputs. Some providers also promise to defend customers against copyright claims over outputs.

Data quality has little specific to this layer. Knowledge cutoffs and other effects of training data are covered in `layer_10_data.md`.

## Not in this layer

- Writing prompts and deciding what goes into the context (context engineering) belong to Layer 4 (Context, memory and retrieval), along with retrieval pipelines, vector databases and memory. This file covers how models respond to prompts and context, and what embedding models and rerankers are.
- How function calling works, protocols such as MCP, and the sandboxes agents run in belong to Layer 3 (Tools and protocols). This file covers how reliably models use tools.
- Routing requests between models and providers, fallbacks and response caching belong to Layer 5 (Model gateway and routing).
- Serving engines, quantization, schema enforcement and prompt caching mechanics belong to Layer 6 (Inference and serving).
- Fine-tuning, reinforcement learning and distillation methods belong to Layer 7 (Model adaptation).
- Training data and knowledge cutoffs are covered in `layer_10_data.md`, and hardware limits in `layer_09_compute.md`.
- Training foundation models from scratch is lab work. An application engineer only needs to know how it's done in outline.

## Learning order

1. Working with a model. You need the interface to try anything else.
2. Capabilities and limits. Learn them by running real inputs through the models you use.
3. How models work. This explains the behavior you've just seen.
4. Frontier proprietary models.
5. Choosing models. This needs the evaluation sets described in `layer_10_data.md`.
6. Embedding models, before you build search (Layer 4).
7. Multimodal models, as your product needs them.
8. Open-weight models, when control, privacy or cost at scale matter.

## Key reading

- Andrej Karpathy, "Deep Dive into LLMs like ChatGPT" (video, 2025), on how models are trained and why they behave as they do.
- 3Blue1Brown's videos on transformers and attention (2024), for the mechanics.
- Jay Alammar and Maarten Grootendorst, *Hands-On Large Language Models* (O'Reilly, 2024), on tokens, embeddings, semantic search and multimodal models.
- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 2 covers how foundation models are built, sampling, structured outputs and hallucination.
- Liu et al., "Lost in the Middle" (2023), and Chroma's "Context Rot" report (2025), on how models use long inputs.
- Sharma et al., "Towards Understanding Sycophancy in Language Models" (2023).
- Kalai et al., "Why Language Models Hallucinate" (OpenAI, 2025).
- METR, "Measuring AI Ability to Complete Long Tasks" (2025), on how fast agentic ability is improving.
- Muennighoff et al., "MTEB: Massive Text Embedding Benchmark" (2022), and the MTEB leaderboard.
- The documentation, prompting guides and system cards for the models you use, plus Simon Willison's blog for keeping up with new releases.
