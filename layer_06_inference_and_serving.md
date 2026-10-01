# Layer 6, Inference and serving: what an application-layer AI engineer needs to know

This file covers the Inference and serving layer from `ai_stack.md`. That layer runs models in production, whether through a provider's API or on your own infrastructure. It covers serving engines, request batching, quantization, KV-cache management and the endpoints that expose models. This file is for an AI engineer who builds products on existing models (layers 1 to 5) and doesn't build serving infrastructure. It lists the concepts in this layer and how deeply to learn each one.

## The rule for depth

Every model call is an inference request, so this layer is where your product meets the model. Most application teams get inference from a provider's API, which hides the serving machinery. But this layer decides what your users experience: how long they wait for the first word, how fast the answer streams, and what happens when a call is slow or fails.

Go deep (level 3) on what you touch on every call: measuring latency, calling endpoints well, and how prompt caching works. Learn how serving works inside to level 2. That covers engines, batching, KV-cache management, quantization and faster decoding. It's enough to understand how providers behave and price, to run a model locally, and to judge when self-hosting makes sense. If your team self-hosts, take the self-hosting topics to level 3. No concept in this layer needs level 4. Building or tuning serving engines is infrastructure work.

Three numbers describe how a model call performs:

1. Time to first token: how long until the answer starts. Users feel this as waiting.
2. Output speed: how fast tokens arrive once the answer starts, in tokens per second.
3. Throughput: how many tokens or requests a server handles per second across all users. Your costs depend on this.

Total response time is roughly the time to first token plus the number of output tokens divided by the output speed. Serving engines trade these numbers against each other: running more requests together raises throughput and lowers cost, but it slows each request.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Inference performance | 3 | Users feel latency on every call |
| Calling models through an API | 3 | You write this code for every feature |
| Serving engines and batching | 2 | Explains how providers behave and price |
| KV-cache management | 2 (some topics 3) | Prompt caching is the part you use daily |
| Quantization | 2 | Decides what fits, how fast it runs and what quality you lose |
| Structured generation | 2 | Explains the limits of schema enforcement |
| Faster generation | 2 | Speeds up some tasks, such as edits |
| Running models yourself | 2 (some topics 3) | Local models for development, production hosting if you self-host |
| Reliability | 2–3 | Every call can be slow, fail or stop early |
| Observability | 2–3 | Time to first token and output speed must be tracked separately |
| Security | 2–3 | Self-hosted endpoints get exposed by mistake |
| Governance and safety | 2 | Serving logs hold user data |

## Inference performance [3]

These numbers describe what users experience, so learn them first.

- [3] Time to first token. The time from sending a request to receiving the first output token. It includes network time, waiting in a queue, and processing the prompt. Long prompts and reasoning make it longer.
- [3] Output speed. How fast tokens arrive once output starts, in tokens per second. It is also measured as the time per output token. When text streams to a reader, anything faster than reading speed feels fast. When the output feeds other code or an agent's next step, only the total time matters.
- [3] Total response time. Roughly the time to first token plus the output tokens divided by the output speed. Output length usually dominates: 1,000 tokens at 50 tokens per second take 20 seconds. Hidden reasoning tokens count too. Shorter outputs are often the biggest latency win.
- [3] Percentiles. Averages hide slow requests. Track the median and the 95th and 99th percentiles (p50, p95, p99). Users remember the slow requests, and multi-step agents multiply the chance of hitting one. If each step of a 10-step agent has a 5% chance of being slow, about 40% of runs hit at least one slow step.
- [3] Latency budgets. Decide how long each feature may take, and split that time across its steps: retrieval, each model call and each tool call. Then design so that users see progress while they wait.
- [2] Throughput. How many tokens or requests a system serves per second across all users. When you self-host, it sets your costs. When you use an API, it's what rate limits cap. Running more requests together raises throughput but slows each request. Why this happens is covered in `layer_09_compute.md`.
- [2] Prefill and decode. Every request has two phases. Processing the prompt (prefill) sets the time to first token. Generating the output (decode) sets output speed. The two phases stress hardware differently, and engines schedule them differently.
- [2] Load and latency. Latency stays flat as load grows, then rises sharply as a server nears capacity, because requests start to queue. Test at the concurrency you expect, measured from the client. Benchmarking on your own traffic is covered in `layer_09_compute.md`.

Check: For one feature, measure time to first token, output speed and total time at p50 and p95, from the client side. Break the total time into its parts. Make one change that cuts p95 total time, such as shorter outputs, parallel calls or a faster model.

## Calling models through an API [3]

The endpoint is how your code reaches a model. You write this code for every feature.

- [3] API styles. Most providers offer a chat-style endpoint that takes a list of messages and returns a response. Many providers, and nearly all self-hosted engines, also accept the OpenAI API format, which has become a common standard. Code written against it can switch providers, or point at a local engine, by changing a URL. Features and parameters still differ between providers.
- [3] Streaming. The server sends tokens as it generates them, usually as server-sent events. Your code has to assemble partial text, tool calls and structured output as they arrive. It also has to handle errors that arrive partway through, and pass the stream through your own backend to the user.
- [3] Timeouts and cancellation. Set timeouts that fit each request: short for small classification calls, long for reasoning or agent steps that can take minutes. Cancel requests the user no longer needs, for example when they leave or send a new message, so those requests don't keep using capacity.
- [3] Concurrency. Run independent model calls in parallel instead of one after another, using async clients and reused connections. Parallel calls cut total time, but they count against rate limits together.
- [2] Background requests. Some providers let you start a long request, such as a deep research task, and collect the result later instead of holding a connection open. Batch endpoints for bulk work are covered in `layer_09_compute.md`.
- [2] Realtime connections. Voice and other live interactions use persistent connections, such as WebSockets or WebRTC, that stream audio both ways. Turn detection decides when the user has finished speaking. Voice feels natural only when responses start within about a second. Learn this to level 3 if you build voice products.

Check: Build a small backend endpoint that streams a model's answer to a browser and cancels the model request when the user disconnects. Have it run two independent model calls in parallel. Then switch it to a local model by changing only the base URL.

## Serving engines and batching [2]

A serving engine is the software that runs a model on GPUs and answers requests. Providers run engines like these behind their APIs.

- [2] What an engine does. It loads the model onto the GPUs and accepts requests. It schedules and batches those requests, manages the KV cache and runs the model efficiently. Widely used engines include vLLM, SGLang and NVIDIA's TensorRT-LLM for production, and llama.cpp and Ollama for local and smaller-scale use. Most expose an OpenAI-compatible API.
- [2] Continuous batching. Older servers waited to fill a batch, then ran it until every request finished. Modern engines add and remove requests from the running batch at every step. This is called continuous (or in-flight) batching. It keeps the GPU busy and greatly raises throughput.
- [2] Scheduling. Engines decide which requests run next and how many run together. When the KV cache is full, they queue new requests or pause running ones. Long prompts can be processed in chunks (chunked prefill) so they don't stall other users' output.
- [2] Parallelism for serving. A model too big for one GPU is split across several with tensor parallelism, which also cuts latency. More copies of the model (replicas) add throughput. Very large mixture-of-experts models spread their experts across GPUs. The hardware side of this is covered in `layer_09_compute.md`.
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

## KV-cache management [2, some topics 3]

The KV cache stores the intermediate results for each conversation. How much memory it takes is covered in `layer_09_compute.md`. This section covers how engines manage it.

- [2] Paged memory. Early servers reserved one large block of memory per request for its maximum possible length, and most of that block went unused. PagedAttention, introduced by vLLM, stores the KV cache in small fixed-size blocks, like an operating system's virtual memory. This fits many more requests on a GPU at once.
- [2] Prefix caching. When requests start the same way, such as with a shared system prompt, the engine can reuse the KV cache it already computed for that shared part. This cuts both time to first token and compute.
- [3] How provider prompt caching works. Provider prompt caching is prefix caching sold as a feature. The match has to be exact from the first token, so anything that changes early in the prompt, even a timestamp, breaks the cache for everything after it. Cached entries expire, often after a few minutes without use. Providers report cache hits in each response, so track your hit rate. Where to put content in a prompt is covered in `layer_08_foundation_models.md`.
- [2] Cache pressure. When the KV cache is full, the engine can't take on more requests, so latency rises sharply. Long contexts and many simultaneous conversations fill it fastest.
- [1] Cache-aware routing. With several servers, sending requests that share a prefix to the same server raises cache hits. Some serving systems do this automatically.
- [1] Shrinking and moving the cache. Newer attention designs, such as grouped-query attention and multi-head latent attention, shrink the KV cache. Engines can also store the cache at lower precision, or move it to CPU memory or disk and back.

Check: Measure time to first token for a long prompt twice: once with a cold cache and once with a warm one. You can do this on a provider or on your own engine. Then change one early token, such as a timestamp, and show that the cache hit disappears.

## Quantization [2]

Quantization shrinks a model so it fits in less memory and runs faster, at some cost in quality. The number formats themselves are covered in `layer_09_compute.md`.

- [2] What it is. Quantization stores weights at lower precision. Some methods also quantize activations and the KV cache. It cuts memory use and usually speeds up generation, but it can lower quality.
- [2] Common formats:
  - GGUF, used by llama.cpp and Ollama, with named levels such as Q4_K_M and Q8_0
  - GPTQ and AWQ, for 4-bit weights on GPUs
  - FP8, on recent data center GPUs
  - 4-bit floating-point formats, on the newest GPUs
- [2] Quality impact. 8-bit is usually close to the original. 4-bit loses a little more, and the loss varies by model and task. Small models, math, long contexts and less common languages tend to suffer most. Always evaluate a quantized model on your own task.
- [2] Weight-only versus full quantization. Quantizing only the weights saves memory and speeds up generation when the model serves one user at a time. Quantizing activations as well speeds up the math itself, which matters most when the model serves many users at once.
- [2] Quantization-aware training. Some models are trained for low precision or released at it, which loses less quality than quantizing a finished model.
- [2] Whose quantization. Community-made quantized versions vary in quality. Providers may also serve quantized versions of open models. This is one reason the same model can score differently across providers (see `layer_09_compute.md`).

Check: Run one open-weight model at 16-bit, 8-bit and 4-bit precision. Compare memory use, output speed and quality on your evaluation set, and choose the precision you would ship.

## Structured generation [2]

This section covers how engines guarantee output formats. How to use structured output in a request is covered in `layer_08_foundation_models.md`.

- [2] How schema enforcement works. At each step, the engine blocks any token that would break the required format, such as a JSON schema, a regular expression or a grammar. This guarantees the output matches the format, unless generation is cut off. Libraries such as XGrammar, Outlines and llguidance do this inside engines.
- [2] Limits. Providers support only part of the JSON Schema standard, so they reject or simplify some schemas. A new schema can add delay to its first request while the provider prepares it. Forcing a format can also lower answer quality if the model has no room to reason first.
- [2] Tool-call parsing. Tool calls use the same machinery: the engine turns the model's output into structured calls. A self-hosted engine needs the right parser for each model family. With the wrong one, tool calls come back as plain text.

Check: Call a model with a JSON schema enforced, either through a provider or on your own engine. Measure the extra delay on the first request with a new schema. Compare answer quality with and without enforcement on 50 examples.

## Faster generation [2]

These techniques speed up output without changing the model.

- [2] Speculative decoding. A small, fast draft model guesses the next several tokens, and the large model checks all of them in one step, keeping the ones it agrees with. The output is exactly what the large model would have produced alone, but it arrives faster when the guesses are good. It helps most when the output is predictable, such as code or text that repeats the input. It speeds up single requests the most; under heavy load, the gains shrink.
- [2] Predicted outputs. Some providers let you pass in the output you expect, such as the file you're editing. The model can then move quickly through the parts that don't change, which speeds up edits a lot.
- [1] Multi-token prediction. Some models are trained to predict several tokens at once, and engines can use this to generate faster.

Check: For an editing task, try speculative decoding on a self-hosted engine or predicted outputs with a provider. Measure how output speed changes at low load and at high load.

## Running models yourself [2, some topics 3]

Running models yourself means running them on your laptop for development, or on your own or rented infrastructure in production. Whether self-hosting is worth it is covered in `layer_09_compute.md`.

- [3] Local development. You can run open models on your laptop with Ollama, llama.cpp or LM Studio, or with MLX on Apple Silicon. They expose an OpenAI-compatible endpoint. This is useful for offline work, fast experiments, and tests that don't cost API calls.
- [2] Managed endpoints. Inference platforms and cloud services can host an open-weight or fine-tuned model on dedicated GPUs for you. They handle the engine, scaling and updates. This is often the right middle ground between a shared API and full self-hosting.
- [2] Packaging and startup. A serving image bundles the engine and its settings. The weights are downloaded or mounted when a server starts. Loading tens or hundreds of gigabytes of weights makes up most of the startup time, so keep weights close to the servers and in a format that loads quickly. Cold starts are covered in `layer_09_compute.md`.
- [2] Autoscaling signals. Scale on queue length, time to first token or KV cache use. CPU use says little about how busy a GPU server is. Adding GPU servers is slow, so scale early.
- [2] Health and rollouts. A server is ready only after its model has loaded and warmed up. Roll out new models or settings gradually, and warm them up before they receive traffic.
- [2] Admission control. Cap the number of concurrent requests, and the prompt length and output length of each request. When load goes beyond capacity, reject or queue the extra requests, so the rest don't all slow down.
- [2] Serving smaller models. Embedding models, rerankers and classifiers need their own serving setup. That can be a specialized server, such as Hugging Face's Text Embeddings Inference, or CPUs. These models are usually limited by throughput, so send many inputs in each call.
- [2] On-device runtimes. Runtimes such as Core ML, ONNX Runtime, ExecuTorch and LiteRT run models on phones and laptops, and WebGPU runs them in browsers. Some operating systems and browsers also offer built-in small models through their own APIs.

Check: Deploy an open-weight model behind an autoscaling endpoint, on a managed platform or your own cluster. Show three things: it scales on queue length, it turns away excess load cleanly, and it rolls out a new version without failed requests.

## Cross-cutting concerns in this layer

### Reliability [2–3]

- [3] How calls fail. Any inference call can fail in several ways. It can time out. It can return a rate-limit or overload error. It can fail partway through a stream. Or it can return an empty or cut-off answer. Handle each case and show users something sensible. Retries and fallbacks belong to Layer 5.
- [2] Self-hosted failures. Engines crash when they run out of GPU memory, and they can hang on unusual inputs. They also slow down when the KV cache fills. Monitor them, and restart unhealthy servers automatically.

Why the same request can give different outputs is covered in `layer_08_foundation_models.md`.

### Observability [2–3]

- [3] Latency metrics. Record time to first token and output speed, not just total time. Track them as percentiles by model and provider. What else to log for each call is covered in `layer_08_foundation_models.md`.
- [2] Engine metrics. Self-hosted engines report queue length, running requests, KV cache use and cache hit rate, usually in Prometheus format. These metrics warn you before latency climbs. GPU metrics are covered in `layer_09_compute.md`.

### Security [2–3]

- [3] Exposed endpoints. Self-hosted engines often have no authentication by default. Never expose one directly to the internet. Put it behind your own authentication and network rules.
- [2] Shared caches. A prefix cache shared between users can leak information through timing: a fast response shows that someone recently sent the same prefix. Providers typically keep caches separate for each organization. Self-hosted systems that serve several customers should do the same.

### Governance and safety [2]

- [2] Serving logs. Engines and proxies can log full prompts and outputs, which contain user data. Decide what gets logged and how long it's kept.

Where inference runs, and the data residency that follows from it, is covered in `layer_09_compute.md`.

Cost and latency have no separate section here. Latency is covered under Inference performance. Cost per token and self-hosting costs are covered in `layer_09_compute.md`, and model prices and cost per task in `layer_08_foundation_models.md`. For Evaluation, changes to how a model is served can change its quality. This is covered in `layer_09_compute.md` and under Quantization. Data quality has nothing specific to this layer.

## Not in this layer

- Hardware limits, the arithmetic of memory and speed, buying capacity, and batch pricing belong to Layer 9 (`layer_09_compute.md`).
- Sampling settings, structured output and prompt caching as you use them in requests, and what to log for each call, are covered in `layer_08_foundation_models.md`.
- Creating fine-tuned models and adapters belongs to Layer 7 (`layer_07_model_adaptation.md`).
- Choosing a model or provider for each request, retries, fallbacks, response caching, and handling rate limits belong to Layer 5 (Model gateway and routing).
- Writing GPU kernels and building serving engines are infrastructure work. Awareness is enough.

## Learning order

1. Inference performance. Learn the three numbers first.
2. Calling models through an API.
3. KV-cache management, especially how prompt caching works.
4. Running models yourself, starting with a local model.
5. Serving engines and batching.
6. Quantization.
7. Structured generation and faster generation.

## Key reading

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 9 covers inference metrics, batching, caching, quantization and speculative decoding.
- Databricks, "LLM Inference Performance Engineering: Best Practices" (2023), on latency metrics and how batching trades latency for throughput.
- Kwon et al., "Efficient Memory Management for Large Language Model Serving with PagedAttention" (2023), the vLLM paper.
- Yu et al., "Orca: A Distributed Serving System for Transformer-Based Generative Models" (2022), which introduced continuous batching.
- Zheng et al., "SGLang: Efficient Execution of Structured Language Model Programs" (2024), on prefix caching and structured generation.
- Leviathan et al., "Fast Inference from Transformers via Speculative Decoding" (2023).
- Maarten Grootendorst, "A Visual Guide to Quantization" (2024).
- The vLLM documentation, plus your providers' guides to latency, streaming and prompt caching.
