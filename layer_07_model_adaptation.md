# Layer 7, Model adaptation: what an application-layer AI engineer needs to know

This file covers the Model adaptation layer from `ai_stack.md`. That layer specializes a base model for a domain or task, through fine-tuning, reinforcement learning from human or automated feedback, and distillation into smaller, cheaper models. This file is for an AI engineer who builds products on existing models (layers 1 to 5) and doesn't train models from scratch. It lists the concepts in this layer and how deeply to learn each one.

## The rule for depth

Adaptation changes a model's weights. It's the most powerful way to change how a model behaves, and the most expensive to build and keep up. Most application teams rarely need it, because better prompts, retrieval and stronger models solve most problems more cheaply. Teams that do adapt usually have one of two reasons:
- to make a small model do a narrow job as well as a large one, for less money and latency
- to lock in a format or behavior that prompting can't hold

Go deep (level 3) on four things: deciding whether to adapt, supervised fine-tuning, distillation, and owning a fine-tuned model over time. These are the parts application teams actually do. Learn preference tuning and reinforcement learning to level 2; you'll use them occasionally, mostly through provider APIs. How labs run these methods at scale is level 1. No concept in this layer needs level 4. If you need that depth, you're doing a model-training job, not an application one.

Before you adapt a model, answer three questions:

1. Have you tried the cheaper fixes? A clearer prompt, examples, retrieval and a stronger model come first.
2. Can you measure the gap? You need an evaluation set that shows where the current model falls short. The same set will show whether adaptation closed the gap.
3. Can you get the data? You need enough good examples of the behavior you want, or a grader that can score it. Building both is covered in `layer_10_data.md`.

If any answer is no, don't adapt yet.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Deciding whether to adapt | 3 | Most problems have cheaper fixes, so adapting needs a clear case |
| Supervised fine-tuning | 3 | The adaptation method app teams actually run |
| Learning from human preferences | 2 | Occasionally useful for tone and behavior; labs use it at scale |
| Learning from automated rewards | 2 | Powerful for checkable tasks, but costly and easy to game |
| Distillation | 3 | The most common reason app teams fine-tune |
| Owning an adapted model | 3 | A fine-tune must be retrained, re-tested and eventually retired |
| Evaluation | 3 | An adapted model must beat the baseline without breaking anything else |
| Cost and latency | 3 | Adapting pays off only when savings outweigh training and upkeep |
| Governance and safety | 2–3 | Fine-tuning can weaken safety, and licenses follow the model you change |
| Security | 2 | Anyone who can supply training data can change the model |
| Observability | 2 | Training curves only mean something next to real outputs |

## Deciding whether to adapt [3]

Knowing when not to adapt is most of the value in this layer.

- [3] The order of fixes. Try fixes in order of cost: a clearer prompt, examples in the prompt, retrieval for missing facts, splitting the task into steps, and a stronger or reasoning model. Each one is cheaper to try and easier to undo than training. Adapt only when they all fall short and your evaluation set shows the gap.
- [3] Good reasons to adapt. Four situations justify it:
  - A small model needs to match a large one on a narrow task, for lower cost or latency.
  - Output format, style or behavior needs to be more consistent than prompting can make it.
  - Long instructions and examples repeated in every prompt could be trained in instead, which shortens prompts.
  - The model has to run on a device or offline.

  What fine-tuning can and can't teach is covered in `layer_10_data.md`.
- [3] The full cost. Count the data work, training runs, evaluation and hosting, plus retraining every time the base model changes. Some providers also charge more per token to run a fine-tuned model than the base model.
- [2] Where the weights live. When you fine-tune a provider's proprietary model, the result stays with the provider. You can call it but not download it, and it lasts only as long as the provider supports its base model. When you fine-tune an open-weight model, you control the weights but have to host them yourself.

Check: For one problem in your product, write a one-page case for or against adapting a model. Include the cheaper fixes you tried, the evaluation results that show the remaining gap, where the training data would come from, and the full cost weighed against the expected savings.

## Supervised fine-tuning [3]

Supervised fine-tuning (SFT) trains a model on examples of inputs paired with ideal outputs, so that it learns to produce outputs like them. It's the method application teams use most. Building the datasets is covered in `layer_10_data.md`.

- [3] How it works. For each example, training adjusts the model's weights slightly to make the ideal output more likely. Training passes over the dataset a few times, and each pass is called an epoch. A validation set, held out from training, shows whether the model is learning the task or just memorizing the examples.
- [3] Two ways to run it. One way is a provider's fine-tuning API: you upload a dataset, start a job and get back a new model name to call. The other is to fine-tune an open-weight model yourself with a library such as Hugging Face TRL, Unsloth or Axolotl. Get hands-on with at least one of them.
- [3] LoRA. Instead of changing all the weights, LoRA trains small add-on matrices, called an adapter, and leaves the original weights frozen. It's the most common of several parameter-efficient methods. It needs far less memory, trains faster and produces small files. On narrow tasks it usually comes close to full fine-tuning: it learns less from big changes, but it also forgets less. QLoRA saves more memory by storing the frozen weights at lower precision. The memory numbers are in `layer_09_compute.md`.
- [2] Full fine-tuning. Updating every weight can teach the model more. But it costs far more memory and compute and is more likely to damage the model's general abilities. Application teams rarely need it.
- [3] Reading loss curves. Training loss should fall. Validation loss should fall and then flatten. If validation loss starts rising while training loss keeps falling, the model is overfitting, so stop at the point where validation loss was lowest. Loss doesn't measure quality directly, so always confirm with your evaluation set.
- [2] Settings. The learning rate matters most: too high damages the model, and too low changes nothing. A few epochs are usually enough. LoRA adds a rank setting that controls the adapter's size. Start from the provider's or library's defaults and change one setting at a time.
- [3] Starting point. Fine-tune the instruction-tuned version of a model, not the base version, and train in the chat format the model already uses. Try the smallest model that might work first, since it's the cheapest to train and to run.
- [2] Embedding models. Fine-tuning an embedding model on real queries paired with the right documents can noticeably improve search in specialized domains. Libraries such as Sentence Transformers make this a small project. Learn this to level 3 if you own search.
- [2] Small classifiers. You can fine-tune a small encoder model to classify text, for example for routing, intent or safety checks. At high volume, this often beats prompting a large model on cost and speed.
- [1] Merging models. Weights from separately fine-tuned models can sometimes be combined into one model without further training.

Check: Fine-tune a small model on a narrow task, with LoRA or through a provider's API. Plot training and validation loss and choose where to stop. Show that the result beats the prompted baseline on your evaluation set without getting worse on a general test set.

## Learning from human preferences [2]

Preference tuning teaches a model which of two answers is better. Labs use it to make models helpful and safe. Application teams use it occasionally, after SFT, to change tone, length or specific unwanted behaviors.

- [2] RLHF. Reinforcement learning from human feedback works in two steps. First, a reward model is trained to predict which answers people prefer. Then reinforcement learning pushes the model toward answers the reward model scores highly. A penalty keeps the model from drifting too far from where it started. This is how chat assistants were first made helpful.
- [2] DPO and related methods. Direct preference optimization (DPO) learns straight from preference pairs, without a separate reward model or reinforcement learning loop. It's simpler and more stable than RLHF, and some providers offer it through their fine-tuning APIs. Related methods include KTO, and ORPO, which combines SFT and preference tuning in one step. The data each method needs is covered in `layer_10_data.md`.
- [2] When to use it. Use preference tuning when the model already does the task but you want to change how it does it: shorter answers, a different tone or fewer unnecessary refusals. Pairs of an original output and a corrected one from your product work well.
- [2] Over-optimization. Pushed too far, a model learns to please the reward signal rather than the user, for example by writing longer or more flattering answers. A strength setting, often called beta, limits how far the model can move from its starting point.
- [1] AI feedback. Labs increasingly have models judge answers against written principles instead of asking people. Anthropic's Constitutional AI is one example.

Check: Explain how DPO differs from RLHF. Describe a problem in your product that preference tuning could fix better than SFT, and say where the preference pairs would come from.

## Learning from automated rewards [2]

In this method, the model learns by attempting tasks that are scored automatically, by tests, checks or a grading model. This is how labs trained today's reasoning models. Some providers offer it as reinforcement fine-tuning (RFT).

- [2] How it works. For each task, the model makes several attempts, and a grader scores each one. Training makes the higher-scoring attempts more likely. Methods such as GRPO compare each attempt with the others in its group, so there's no need to train a separate model that estimates how good each attempt is. A penalty keeps the model close to where it started.
- [2] Verifiable rewards. Some answers can be checked automatically, for example by running tests or comparing with a known answer. For these tasks, rewards are cheap and hard to dispute. This is a big reason models have improved fastest at math and code.
- [2] Reinforcement fine-tuning through a provider. The provider runs the training loop on your tasks and grader. It can work with relatively few tasks. But every training step needs many model attempts, so it costs far more than SFT. What you supply, and how to build the grader, is covered in `layer_10_data.md`.
- [2] Sharpening, not creating. Reinforcement learning mostly makes a model more reliable at things it can already do some of the time. If the base model never succeeds at a task, there's nothing to reinforce. Measure its success rate across several attempts first, and use SFT to get it started if needed.
- [2] Watching the outputs. Models find ways to raise their score without doing the task. During training, read samples of what the model produces, not just the reward curve.
- [1] Agentic training. Labs use reinforcement learning to train agents on long, multi-step tasks with real tools. This is a big reason recent models handle agentic work better, but application teams rarely run it themselves.

Check: Pick a task in your product that has a checkable answer. Measure how often the base model succeeds across several attempts. Decide whether reinforcement fine-tuning, SFT or neither is the right next step, and explain why.

## Distillation [3]

Distillation trains a smaller, cheaper model (the student) to imitate a larger one (the teacher). It's the most common reason application teams fine-tune.

- [3] The workflow. Run the teacher on real inputs and keep only its good outputs. Fine-tune the student on those outputs with SFT. Compare the student with the teacher on a held-out evaluation set, then move traffic to the student gradually. Building the dataset is covered in `layer_10_data.md`.
- [3] What to expect. On a narrow task, a well-distilled student can come close to the teacher at a fraction of the cost and latency. It rarely beats the teacher, and it handles unusual inputs worse, so keep the teacher available for hard cases. Sending hard cases to the teacher belongs to Layer 5.
- [2] Provider tools. Some providers let you store a large model's production outputs and use them directly to fine-tune one of their smaller models.
- [2] Outputs versus probabilities. Labs often distill using the probabilities the teacher assigns to every possible next token, which carry more information than its text alone. With a provider's model you only get the text, so you distill from outputs alone.
- [2] Distilling reasoning. Training a small model on a reasoning model's step-by-step answers can pass on reasoning ability for a narrow task. With proprietary teachers, two things limit this: the reasoning may be hidden or summarized, and provider terms may forbid it.

Check: Distill a large model into a small one for one task. Report the student's quality, cost and latency next to the teacher's on the same held-out set. List the kinds of input where the student falls short.

## Owning an adapted model [3]

A fine-tuned model is software you have to maintain. It doesn't get better when the provider releases a better base model.

- [3] Base model upgrades. When a better base model arrives, your fine-tune doesn't carry over to it. You can retrain on the new base, or first check whether the new model with only a prompt already beats your fine-tune. Expect to face this decision every few months.
- [3] Retirement. Providers eventually retire the base models they offer for fine-tuning, and fine-tuned models built on them are retired too. Open-weight fine-tunes don't expire, but they fall behind newer models.
- [3] Reproducibility. Keep everything you need to retrain: the dataset version, the base model and its version, the settings and the evaluation results. Automate the pipeline so that retraining is routine, not a project.
- [3] Fallback. Keep a working prompt-based version of every fine-tuned feature. Then you can switch back if the fine-tune breaks or its base model is retired.
- [2] Serving. You call a hosted fine-tune like any other model. Open-weight fine-tunes need hosting, and many LoRA adapters can share one base model on the same hardware. How that works belongs to Layer 6.

Check: Retrain one fine-tuned model on a newer base model using your automated pipeline. Compare the result with the old fine-tune and with the new base model using only a prompt. Show that you can switch to the prompt-based fallback.

## Cross-cutting concerns in this layer

### Evaluation [3]

- [3] Beating the baseline. Compare every adapted model with two alternatives on the same held-out evaluation set: the best prompt on the base model, and a stronger model. If the adapted model doesn't clearly win on quality, cost or latency, don't ship it.
- [3] Regressions. Adaptation can weaken abilities you didn't train on, such as general knowledge, instruction following, and formats outside your task. Test on more than the target task.

Building held-out evaluation sets and keeping them out of training data is covered in `layer_10_data.md`.

### Cost and latency [3]

- [3] Break-even. Adapting pays off when the savings on each request, multiplied by your volume, outweigh the full cost of adapting. The savings come from a smaller model, shorter prompts and lower latency. Work this out before you start. It usually takes steady, high volume.
- [2] How training is billed. Hosted fine-tuning usually charges per training token, which is dataset size times the number of epochs. Reinforcement fine-tuning is billed differently, for example by training time, and usually costs much more. Running training yourself costs GPU hours.

### Governance and safety [2–3]

- [3] Weakened safety. Fine-tuning can weaken a model's safety training even when the training data is harmless. Re-test safety behavior after every fine-tune, and add your own guardrails where needed.
- [2] Licenses. Check that the base model's license allows fine-tuning and commercial use of the result. Also check what you must pass on, such as attribution or the original license terms.
- [2] Provider review. Providers check fine-tuning data and fine-tuned models against their usage policies, and they can reject a job.
- [2] Regulation. Under laws such as the EU AI Act, modifying a general-purpose model heavily enough can make you legally responsible for it as its provider. Typical fine-tunes fall well short of that line, but check before large projects.
- [2] Model documentation. Write a short card for each adapted model that covers the base model, data sources, method, evaluation results and known limits.

### Security [2]

- [2] Fine-tuning as an attack. Your product may let users fine-tune models or supply training data. If so, a handful of harmful examples can strip safety training or plant hidden behaviors. Screen the data and test the result before deploying it.

Poisoned training data and personal data leaking into fine-tuned models are covered in `layer_10_data.md`. Trusting downloaded model files is covered in `layer_08_foundation_models.md`.

### Observability [2]

- [2] Training metrics. Watch loss curves (and reward curves, for reinforcement learning) alongside samples of real outputs. Record every run, with its data and settings, in an experiment tracker such as Weights & Biases or MLflow.

Reliability is covered under Owning an adapted model: fallbacks, retraining and retirement. Data quality for training data is covered in `layer_10_data.md`.

## Not in this layer

- Building fine-tuning, preference and reinforcement learning datasets, graders and environments belongs to Layer 10 (`layer_10_data.md`). So does continued pre-training on domain text.
- Memory and compute for training belong to Layer 9 (`layer_09_compute.md`).
- What each training stage does to a model's behavior is covered in `layer_08_foundation_models.md`.
- Serving fine-tuned models and adapters, and quantizing them, belong to Layer 6 (Inference and serving).
- Routing requests between a teacher and a student model, and fallbacks, belong to Layer 5 (Model gateway and routing).
- Pre-training models from scratch and running RLHF at lab scale are lab work. Awareness is enough.

## Learning order

1. Deciding whether to adapt. Most of the value is in knowing when not to.
2. Supervised fine-tuning, hands-on through a provider's API or with LoRA.
3. Distillation, which builds on SFT.
4. Owning an adapted model.
5. Learning from human preferences.
6. Learning from automated rewards, after you've built graders (see `layer_10_data.md`).

## Key reading

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 7 covers when and how to fine-tune, including LoRA and model merging.
- Hu et al., "LoRA: Low-Rank Adaptation of Large Language Models" (2021).
- Dettmers et al., "QLoRA: Efficient Finetuning of Quantized LLMs" (2023).
- Biderman et al., "LoRA Learns Less and Forgets Less" (2024).
- Ouyang et al., "Training language models to follow instructions with human feedback" (2022), the InstructGPT paper on RLHF.
- Rafailov et al., "Direct Preference Optimization: Your Language Model Is Secretly a Reward Model" (2023).
- DeepSeek-AI, "DeepSeek-R1" (2025), on reinforcement learning for reasoning with GRPO, and on distilling that reasoning into smaller models.
- Hinton, Vinyals and Dean, "Distilling the Knowledge in a Neural Network" (2015).
- Qi et al., "Fine-tuning Aligned Language Models Compromises Safety, Even When Users Do Not Intend To!" (2023).
- Nathan Lambert, *Reinforcement Learning from Human Feedback* (rlhfbook.com), on preference tuning, reward models and reinforcement learning.
- The Hugging Face TRL documentation and Unsloth's guides, for hands-on fine-tuning.
