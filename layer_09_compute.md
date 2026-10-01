# Layer 9, Compute: what an application-layer AI engineer needs to know

This file covers the Compute layer from `ai_stack.md`: GPUs and other accelerators, the clusters and cloud capacity they run in, and the power and data centers behind them. It is for an AI engineer who builds products on existing models (layers 1 to 5) and doesn't manage hardware. It lists the concepts in this layer and how deeply to learn each one.

## The rule for depth

Compute is mostly hidden from you. Model providers own the hardware, and you pay per token. You still run into compute in three places:

1. The limits and prices of model APIs.
2. Running a model yourself: self-hosting an open-weight model, running a fine-tuning job, or trying a model on your laptop.
3. The market forces that set prices and decide where capacity exists.

Learn the arithmetic behind these decisions in depth. Learn how the hardware works well enough to follow that arithmetic. For everything underneath, such as chip design, cluster networking, data centers and power, awareness is enough. No concept in this layer needs level 4. If you need that depth, you're doing an infrastructure job, not an application one.

Most compute questions come down to three calculations:

1. Memory: will the model fit? Multiply the number of parameters by the bytes each one takes. Then leave room for the KV cache (stored intermediate results for each conversation) and for overhead.
2. Speed: how fast can it generate? Each new token requires reading every weight from memory once. So memory bandwidth divided by the model's size in bytes gives the top speed for a single conversation.
3. Cost: what does a token cost? Divide the hourly cost of the hardware by the tokens it produces per hour, at the utilization you'll actually get.

Once you learn these calculations, you use them in every section below.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Compute arithmetic | 3 | Answers most compute questions: will it fit, how fast will it run, what will it cost |
| Accelerators | 2 | You choose hardware, but you don't program it |
| Multi-GPU and clusters | 1 (some topics 2) | Platform teams run clusters; you only need to know why big models need them |
| Cloud capacity | 2–3 | You buy compute, either directly or through a model API |
| Power and data centers | 1 | Explains prices, shortages and regions; you won't build any of it |
| Cost and latency | 3 | Choosing between an API and self-hosting is your biggest compute decision |
| Reliability | 2 | Capacity shortages and hardware failures reach your users |
| Observability | 2 | You'll read GPU metrics when you self-host |
| Governance and safety | 2 | Where the hardware sits decides where data is processed |
| Evaluation | 2 | The same model can give different results on different hardware |
| Security | 1 | Providers handle most of it |

## Compute arithmetic [3]

These calculations come up in every other section, so learn them first.

- [3] Weight memory. Multiply the number of parameters by the bytes per parameter. At 16-bit precision each parameter takes 2 bytes, at 8-bit it takes 1 byte, and at 4-bit it takes half a byte. A 70-billion-parameter model at 16-bit needs about 140 GB for its weights alone, so it doesn't fit on one 80 GB GPU.
- [3] KV cache memory. The model stores intermediate results for every token in a conversation so it doesn't have to recompute them. This store is called the KV cache. It grows with context length and with the number of conversations running at once, and on long contexts it can take more memory than the weights. This is why long contexts cost more, and why a GPU can serve fewer long conversations at once. How serving engines manage the KV cache belongs to Layer 6.
- [3] Generation speed. For one conversation at a time, the top speed is memory bandwidth divided by the model's size in bytes. The H100 SXM, a common data center GPU, has about 3.35 TB/s of memory bandwidth. A model whose weights take 70 GB can therefore generate at most about 48 tokens per second on it. Real systems stay below this ceiling.
- [3] Cost per token. Divide the hourly cost of the hardware by the tokens it produces per hour. For example, suppose a GPU rents for $2.50 an hour and produces 1,000 tokens per second when fully busy. It makes 3.6 million tokens an hour, so each million tokens costs about $0.70. If it's busy only a quarter of the time, each million tokens costs about $2.80.
- [2] Memory-bound and compute-bound work. The model processes all the tokens in a prompt at once, so that step is limited by raw compute. It generates the answer one token at a time, so that step is limited by memory bandwidth. This is why output tokens cost several times more than input tokens, and why cached input is cheaper still. It is also why serving many conversations at once lowers the cost per token.
- [2] FLOPs per token. A model does about 2 floating-point operations (FLOPs) per parameter for each token it processes. Training takes about 6 per parameter for each training token. Use these numbers to estimate how long a long prompt takes to process, or roughly what a training run costs.
- [2] Mixture-of-experts models. These models use only some of their parameters for each token. They need memory for all of their parameters, but they generate at roughly the speed of the parameters they actually use.
- [2] Fine-tuning memory. Full fine-tuning needs about 16 bytes per parameter for weights, gradients and optimizer state, before counting activations. LoRA freezes the original weights and trains a small add-on. QLoRA also stores the original weights at 4-bit, so a model of about 8 billion parameters can be fine-tuned on a single 24 GB GPU.
- [2] Test-time compute. Reasoning models spend more compute on each answer by generating many more tokens before they reply. That makes them slower and more expensive, but more accurate on hard tasks. Choosing how much reasoning to allow for each request belongs to Layer 5.

Check: Take a model's size, its precision and a context length. Estimate whether it fits on a given GPU, its top generation speed for one user, and its cost per million tokens at 30% utilization. Then measure a real run and explain the gap.

## Accelerators [2]

Accelerators are chips built for the matrix math that models run on. GPUs are the most common kind.

- [2] What a GPU is good at. Thousands of small cores work in parallel. Dedicated units called tensor cores do matrix math quickly at low precision.
- [2] Memory. Each GPU has a large pool of high-bandwidth memory (HBM) and a small amount of much faster memory on the chip itself. For running models, HBM capacity and bandwidth usually matter more than raw compute. Many speedups, such as FlashAttention, come from moving data between these two kinds of memory less often.
- [2] Precision. Numbers can be stored in 32, 16, 8 or 4 bits. Each halving of precision halves memory use and roughly doubles the math the chip can do, if the chip supports that format. Recent generations added 8-bit and then 4-bit floating-point formats. Choosing a quantization method belongs to Layer 6.
- [2] Reading a spec sheet. The numbers to look for are memory capacity (GB), memory bandwidth (TB/s), compute (TFLOPS) at each precision, link speed between GPUs, and power draw (watts). Headline compute figures are often quoted "with sparsity". That doubles them, and it rarely applies in practice.
- [2] Generations. Each generation adds memory, bandwidth and support for lower precision, so the same model gets cheaper to run. Check which generation a price or benchmark refers to. NVIDIA now releases a new generation roughly every year.
- [1] The chip landscape. NVIDIA dominates, with data center GPUs such as the A100, H100, H200 and the Blackwell generation. AMD's Instinct GPUs often have more memory per chip. Google's TPUs and AWS's Trainium and Inferentia are custom chips from cloud providers. Makers of specialized inference chips, such as Cerebras and Groq, sell very fast generation.
- [2] The software stack. This covers drivers, CUDA and its libraries, AMD's ROCm, and frameworks such as PyTorch and JAX. CUDA's maturity is a big reason NVIDIA dominates. The most common setup failure is a mismatch between the versions of the driver, CUDA and PyTorch.
- [2] CPUs versus GPUs. Small models, such as embedding models, classifiers and rerankers, often run well enough on CPUs, especially after quantization. A GPU pays off once volume is high.
- [2] Local and edge hardware. Apple Silicon Macs share one pool of memory between the CPU and GPU. A Mac with a lot of memory can therefore run large quantized models, though more slowly than data center GPUs. High-end consumer GPUs have 24 to 32 GB. Phones and laptops have neural processing units (NPUs) that run small models on the device. On-device models help with privacy, cost and offline use, but memory, battery and heat limit their size.

Check: Read the spec sheets of two data center GPUs. Explain which would serve a given model more cheaply, and why.

## Multi-GPU and clusters [1, some topics 2]

Large models don't fit on one GPU, so they run across several. Training runs use thousands.

- [2] Why models span GPUs. When the weights plus the KV cache exceed one GPU's memory, the model is split across several GPUs. Splitting within one server works far better than splitting across servers, because the links inside a server are much faster.
- [1] Interconnects. NVLink connects GPUs inside a server or rack, and it is many times faster than PCIe or the network between servers. InfiniBand or high-speed Ethernet connects servers to each other. Rack-scale systems, such as NVIDIA's NVL72 racks, link 72 GPUs so they work almost like one large GPU.
- [1] Ways to split work. Tensor parallelism splits each layer across GPUs and needs the fastest links. Pipeline parallelism gives each GPU a block of layers. Data parallelism runs copies of the model on different data. Expert parallelism spreads a mixture-of-experts model's experts across GPUs. How serving and training use these methods belongs to Layers 6 and 7.
- [1] Servers. A typical AI server holds 8 GPUs with fast links between them, plus CPUs, system memory and local storage. Loading weights from storage takes up a large part of a model's startup time.
- [2] GPUs in Kubernetes. GPU node pools, requesting GPUs for a container, and NVIDIA's GPU Operator. Research and batch clusters often use Slurm or Ray instead. Learn this to level 2 if your team self-hosts; otherwise level 1 is enough.
- [1] Sharing a GPU. One GPU can be split into isolated slices (NVIDIA's Multi-Instance GPU, or MIG) or shared over time. This is useful for small models that don't need a whole GPU.
- [1] Failures at scale. In large clusters, GPUs and links fail routinely. Long training jobs therefore save checkpoints often and restart automatically.

Check: Work out how many 80 GB GPUs a 405-billion-parameter model needs at 8-bit precision, leaving room for the KV cache. Explain why those GPUs should all be in one server.

## Cloud capacity [2–3]

Every model call runs on someone's hardware. You buy that capacity either indirectly, through a model API, or directly, by renting GPUs.

- [3] Ways to buy compute. You can pay per token through a model API, where the provider owns the hardware, or reserve throughput from a model provider. You can rent serverless GPUs billed by the second, or GPU virtual machines billed by the hour. You can also buy your own hardware. Know which option fits a given workload.
- [3] Provider capacity as you see it. Providers set rate limits and usage tiers. At busy times you may see overload errors and slower responses, and some models are available only in some regions. Before a launch, forecast your peak tokens per minute and ask for higher limits early. Handling these errors in code belongs to Layer 5.
- [3] Batch and priority options. Batch APIs run work on spare capacity at a discount, often 50%, and return results within about a day. Use them for evaluations, backfills and synthetic data. Some providers also sell priority or flexible tiers that trade price against speed or reliability.
- [2] Reserved throughput. Paying for a fixed share of a provider's capacity gives you predictable throughput and latency. It's usually worth it only at steady, high volume.
- [2] GPU pricing models. On-demand capacity is the most expensive. Reserved or committed capacity is cheaper, in return for a commitment of one to three years. Spot or preemptible capacity is much cheaper, but the provider can take it back at short notice. Some clouds also sell short reservations for a fixed window.
- [2] Quotas and scarcity. On a new cloud account, GPU quotas often start at zero. The newest chips are scarce and may require a waitlist or a long commitment. Availability differs by region.
- [2] Serverless GPUs and cold starts. You pay by the second and can scale down to zero. But a cold start has to find a machine and load the weights. That takes seconds for small models and can take minutes for large ones. Keep warm instances for anything latency-sensitive.
- [2] Data gravity. Moving data between clouds costs money (egress fees) and time. Put compute near your data, or budget for the transfer.
- [1] The provider landscape. Hyperscalers (AWS, Google Cloud, Microsoft Azure, Oracle), GPU-focused clouds (for example CoreWeave, Lambda, Nebius and Crusoe), and serverless and inference platforms (for example Modal, RunPod, Baseten and Together AI).

Check: For a feature launching next month, forecast peak tokens per minute and compare the forecast with your rate limits. Move one offline job to a batch API. Write down how you'd buy compute if traffic grew tenfold.

## Power and data centers [1]

This is the physical base under everything else. You won't build any of it, but it explains prices, shortages and where capacity exists.

- [1] Power. Power is now the main limit on new AI capacity. Large AI data center campuses need hundreds of megawatts to gigawatts, and new grid connections can take years. This shapes where capacity appears and how fast it grows.
- [1] Density and cooling. The newest racks each draw over 100 kW and need liquid cooling, so many older data centers can't host them.
- [1] Location. Capacity clusters where power is cheap and available. Distance to users matters for real-time features, but not for training or batch work.
- [1] Supply chain. Advanced chips depend on a few suppliers. TSMC makes most of them, and three companies make HBM: SK hynix, Samsung and Micron. Advanced packaging capacity is also limited. A bottleneck at any of these causes shortages.
- [1] Export controls. Governments restrict where advanced chips can be sold. This affects which providers and models can operate in which countries.
- [1] Energy, carbon and water. AI data centers use a lot of electricity, and some also use a lot of water for cooling. Estimates of energy per query vary widely. Know enough to give a stakeholder a rough, honest answer that includes the uncertainty.
- [1] Scale trends. The compute used to train frontier models has grown several-fold each year, and the largest clusters now hold hundreds of thousands of GPUs. This affects how often new models arrive and how much better they get.

Check: Explain to a stakeholder why the newest GPUs are scarce, and why the price of a given level of model capability keeps falling.

## Cross-cutting concerns in this layer

### Cost and latency [3]

- [3] Self-hosting versus APIs. Compare an API's price per million tokens with your own cost per million tokens at realistic utilization, and add engineering and operations time. Self-hosting usually wins only in four cases: steady high volume, strict data rules, a model that no API offers, or latency needs that an API can't meet.
- [3] Utilization. An idle GPU costs as much as a busy one. The utilization you can sustain decides your real cost per token.
- [3] Right-sizing. Pick the cheapest hardware that fits the model and meets your latency target. For small models, older or smaller GPUs are often cheaper per token.
- [2] Benchmarking. Measure throughput, time to the first token and cost using your own prompts and traffic pattern. Don't rely on the peak figures in spec sheets or vendor charts.
- [2] Price trends. GPU rental prices have fallen as supply has grown. The price of a given level of model capability has dropped steeply year over year. Plan on features that are too expensive today becoming affordable later.

### Reliability [2]

- [2] Provider capacity incidents. A provider can run short of capacity or go down entirely. Spreading traffic across providers or regions helps; how to do that belongs to Layer 5.
- [2] Headroom. GPUs are scarce and slow to start, so you can't add them in minutes the way you can add ordinary servers. Keep spare capacity for traffic peaks.
- [2] Spot interruptions. Batch and fine-tuning jobs on spot capacity must save checkpoints and resume after an interruption.
- [1] Hardware failures. GPUs fail more often than ordinary servers. Managed services hide this from you, but self-hosting doesn't.

### Observability [2]

- [2] GPU metrics. Memory use, utilization, power and temperature, from tools such as nvidia-smi and NVIDIA's DCGM. "GPU utilization" only reports the share of time the GPU was running any work, not how hard it was working. Track tokens per second and cost per token as well.

### Governance and safety [2]

- [2] Data residency. Where the hardware sits decides where data is processed. Check which regions a provider runs a model in. Some providers can route requests to other regions to handle load unless you choose a regional option.
- [1] Sovereign and on-premises requirements. Some governments and industries require the hardware to be in a particular country or on their own premises.

Energy, carbon and export controls are covered under Power and data centers.

### Evaluation [2]

- [2] Same model, different results. The same open-weight model can score differently on different hardware, at different precision, or from different providers, because of differences in quantization and implementation. Re-run your evaluations whenever you change where a model runs.

### Security [1]

- [1] Shared hardware. On cloud and serverless platforms, your workloads share physical machines with other customers. Know what isolation your provider guarantees.
- [1] Confidential computing. Recent GPUs can run workloads in encrypted enclaves whose setup can be verified, so even the cloud operator can't see the data. Some teams use this for very sensitive data.

Data quality has nothing specific to this layer.

## Not in this layer

- Serving engines, request batching, quantization methods, KV-cache management and model endpoints belong to Layer 6 (Inference and serving). This file covers the hardware limits they work within.
- How fine-tuning and other training methods work belongs to Layer 7 (Model adaptation).
- Handling rate limits, falling back between providers, and routing requests (including choosing a reasoning budget) belong to Layer 5 (Model gateway and routing).
- How models are designed, such as dense versus mixture-of-experts, belongs to Layer 8 (Foundation models). This file covers only what the design means for memory and speed.
- Writing GPU kernels (in CUDA or Triton) and designing clusters are infrastructure jobs. An application engineer only needs to know they exist.

## Learning order

1. Compute arithmetic.
2. Accelerators, especially memory-bound work, precision and spec sheets. These explain the arithmetic.
3. Cloud capacity, starting with how model APIs expose capacity.
4. Cost and latency, especially self-hosting versus APIs.
5. Multi-GPU and clusters, then reliability and observability if you self-host.

You can read about power and data centers at any point. The material is light, and it explains the prices and shortages you'll see along the way.

## Key reading

- Horace He, "Making Deep Learning Go Brrrr From First Principles" (2022), on compute-bound versus memory-bound work.
- Carol Chen, "Transformer Inference Arithmetic" (kipp.ly, 2022), on memory, bandwidth and KV-cache math.
- EleutherAI, "Transformer Math 101" (2023), on memory and compute for training and inference.
- Google DeepMind, "How to Scale Your Model" (2025), on rooflines, accelerators and parallelism.
- Modal, "GPU Glossary", a plain-language guide to GPU hardware and software terms.
- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 9 covers inference optimization, including AI accelerators.
- Stas Bekman, *Machine Learning Engineering Open Book* (GitHub), a practical reference on accelerators, networks and debugging.
- International Energy Agency, "Energy and AI" (2025), on data center power demand.
- Epoch AI's data on hardware, training compute and power trends.
