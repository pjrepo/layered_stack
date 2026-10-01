# Security: what an application-layer AI engineer needs to know

This file covers the Security concern from `ai_stack.md`. Security defends against attackers. It covers prompt injection, authentication and authorization (including what each agent is allowed to do), secrets management, and preventing data from being stolen. It is one of the seven cross-cutting concerns: not a layer, but something every layer needs. This file is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this concern and how deeply to learn each one.

## The rule for depth

A model can't reliably tell your instructions from text an attacker wrote, and no one yet knows how to make it. Anyone who can get text in front of the model, by typing it or by planting it in a web page, an email or a document the model will read, can try to steer it. The rest of this curriculum tries to get the most out of the model. Security assumes the model may be working for someone else, and limits what it can do, reach and send, with controls enforced in ordinary code.

For every AI feature, security answers four questions:

1. Who might attack, and where can their text get in? (Foundations, Prompt injection, Jailbreaks and abuse)
2. What could a hijacked model do or reach, and on whose authority? (Access and identity, Secrets)
3. What could leave the system, and through which channel? (Preventing data theft)
4. What are you trusting that you didn't build? (Supply chain)

Security testing and Responding to attacks check the answers and deal with what gets through.

Security teams know attacks and defenses in general, but only you know what your models can read, reach and do. So the AI-specific design is yours. Most concepts in this concern are level 3, and three things are level 4:
- designing each feature so that a hijacked model can do little harm
- enforcing the user's own permissions, in code, on everything the model can reach
- mapping and closing every way data can leave

Know attack techniques well enough to recognize their families, and leave inventing new ones to researchers. Classic application and network security are disciplines of their own, so work with your security team on them. Attacks on models during training, such as poisoning pre-training data, need only awareness, because labs and providers defend against them.

The layer files cover security as it applies to each layer, such as tool poisoning in Layer 3 and permission-aware retrieval in Layer 4. This file covers what they share, and Security at each layer maps where the rest is covered.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Foundations | 3 (some topics 4) | The model can't be trusted to protect anything |
| Prompt injection | 3 (some topics 4) | Anyone who can get text in front of the model can try to steer it |
| Jailbreaks and abuse | 3 | Every model can be talked out of its rules |
| Access and identity | 3 (some topics 4) | The model must never reach more than the user could |
| Secrets | 3 | AI systems copy text, and secrets travel with it |
| Preventing data theft | 3 (some topics 4) | Attackers can get the model to hand data over |
| Supply chain | 3 | Models, tools and packages from others are ways in |
| Security testing | 3 | A defense nobody has attacked is untested |
| Responding to attacks | 3 | Some attacks will get through |
| Governance and safety | 2–3 | Laws treat a leak through AI like any other breach |
| Reliability | 2–3 | Security checks must fail safe |
| Cost and latency | 3 | Every check adds cost and delay |

## Foundations [3, some topics 4]

These ideas come up in every other section, so learn them first.

- [4] The model isn't a security boundary. Models can't reliably tell your instructions from text an attacker wrote (see `layer_08_foundation_models.md`), and no known method fixes this completely. So never rely on the model, or on instructions in its prompt, to protect anything. Treat it like an untrusted user: enforce every limit in code, outside the model, and design each feature so that a model under an attacker's control can do little harm.
- [3] Who attacks. Attackers include users of your product, anyone who can put text where your system will read it, such as in a web page, an email, a shared document or a product review, the publishers of tools, models and packages you use, and insiders. Many of them never use your product directly.
- [3] Threat modeling. For each feature, write down what an attacker could want, such as your data, your users' data, actions in their name or your money, how they could get in, and what stops them. Catalogs such as the OWASP Top 10 for LLM Applications and MITRE ATLAS list common attacks to check against. Update the threat model whenever a feature gets a new tool, data source or kind of user.
- [3] Defense in depth. No single defense stops every attack, so combine several: limit what the model can do, check what goes in and what comes out, require approval for what can't be undone, and watch for attacks. Then an attack has to get past all of them to cause harm.
- [3] Ordinary security still applies. An AI feature is still a web application, and many real incidents come from ordinary mistakes, such as exposed keys, missing authorization checks, public storage buckets and unpatched dependencies. Work with your security team on the basics. This file covers what AI adds.

Check: Write a one-page threat model for one AI feature. List every way outside text reaches the model, everything the model can do and reach, and the worst thing an attacker could achieve through each input. For the worst case, name the control in code that stops it.

## Prompt injection [3, some topics 4]

Prompt injection is text that makes a model follow an attacker's instructions instead of yours. It is direct when the attacker types it, and indirect when it arrives inside content the model reads, such as a web page, a document or a tool result. How models fall for it is covered in `layer_08_foundation_models.md`, injection through retrieved content in `layer_04_context_memory_and_retrieval.md`, and injection through tools in `layer_03_tools_and_protocols.md`. This section covers the defenses they share.

- [4] Limiting the damage first. Filters and instructions reduce injection but don't stop it, so the defenses that matter most limit what a hijacked model can do. Give it only the tools and data the task needs, keep agents that read untrusted content away from powerful tools (see `layer_02_agents_and_orchestration.md`), require a person's approval for actions that can't be undone, and close the channels data could leave through (see Preventing data theft). Where the stakes are high, use agent designs that resist injection (see `layer_03_tools_and_protocols.md`).
- [3] Every entry point. List every way outside text reaches each model call: user messages, uploaded files, retrieved documents, web pages, emails, tool results, memories and other agents' output. Instructions can hide where people won't see them, such as in white text, HTML comments, image text, file metadata or invisible Unicode characters.
- [3] Marking untrusted content. Put untrusted content inside clear markers, label where it came from, and tell the model to treat it as data. Some teams also transform it, such as by inserting a special character between its words, so the model can tell it apart from instructions (spotlighting). This makes injection harder, not impossible.
- [3] Detecting injection. Classifiers can flag text that looks like an injection before it reaches the model. Cloud providers offer some, and open models such as Llama Prompt Guard run on your own servers. They catch common attacks, miss new ones and flag some harmless text, so treat a flag as a signal, not a guarantee.
- [3] Checking actions against the request. Before an agent acts, check the action against what the user actually asked for and against fixed rules, such as which domains email may go to or which records a tool may change. An action the user's request doesn't explain is a warning sign (see goal drift in `layer_02_agents_and_orchestration.md`).
- [2] Model-level defenses. Providers train models to resist injection and to give text in documents and tool results less authority than the system prompt (see `layer_08_foundation_models.md`), and some publish how often attacks succeed. This raises the bar, but it doesn't replace the other defenses.

Check: Plant injected instructions in a document, a web page and a tool result that your agent reads. Have one ask for an action the user didn't request, and another ask for a different user's data. Show that each attempt is stopped by a control in code, such as a permission check or an approval, even when the model follows the instruction.

## Jailbreaks and abuse [3]

A jailbreak is a user talking a model into breaking its rules. Abuse is using your product for something it wasn't meant for. How models respond to jailbreaks is covered in `layer_08_foundation_models.md`, and deciding which content to block in `concern_04_governance_and_safety.md`. This section covers defending the product against both.

- [3] Common techniques. Role play and made-up scenarios, long prompts full of fake examples of the model complying (many-shot jailbreaking), requests hidden in encodings or less common languages, and conversations that escalate a little at a time. New techniques appear constantly, so learn the families rather than a list.
- [3] Refusal isn't a control. Every model can be jailbroken with enough effort. For anything that matters, such as another user's data, money or dangerous content, enforce the limit in code or with a separate check that the conversation can't talk its way past.
- [3] Input and output classifiers. Separate classifiers can screen requests and responses for jailbreak attempts and prohibited content. Options include provider moderation endpoints, open models such as Llama Guard, and small classifiers you train yourself (see `layer_07_model_adaptation.md`). Checking the output catches jailbreaks that the input check missed.
- [3] Abuse of the product. Some users treat your product as free general-purpose AI, generate spam with it, script many accounts to get around limits, or send expensive requests on purpose to run up your bill. Limits per user and per account (see `layer_05_model_gateway_and_routing.md`), checks at sign-up, and watching for unusual use (see `concern_02_observability.md`) keep the cost and damage down.
- [2] Copying your product. Competitors can query your product at scale to extract its prompts, or to collect outputs to train their own models. Rate limits and terms of service are the practical defenses.

Check: Run a jailbreak session against one feature with a colleague, using at least three families of technique. Show which attempts the model resisted on its own, which your classifiers caught, and that none reached data or actions protected in code.

## Access and identity [3, some topics 4]

This section covers who may use the system, and what the model may reach on their behalf. Permissions for agents and their tools are covered in `layer_03_tools_and_protocols.md`, permission-aware retrieval in `layer_04_context_memory_and_retrieval.md`, and provider API keys in `layer_05_model_gateway_and_routing.md`.

- [4] The user's permissions, everywhere. The model should never reach anything the user who asked couldn't reach themselves. Check the requesting user's rights at every point where data or actions are reached: search, tools, memories, caches, files and datasets. Check in code, at the point of access, never by asking the model.
- [3] Every caller identified. Authenticate users, services and agents, and give each its own identity, so every model call, tool call and action traces back to whoever caused it. Never leave a model endpoint, MCP server or internal tool reachable without authentication (see `layer_06_inference_and_serving.md`).
- [3] Customers kept apart. In a product that serves many customers, one missing filter can show one customer's data to another. Check isolation everywhere data is stored: indexes and memories (see `layer_04_context_memory_and_retrieval.md`), response and prompt caches (see `layer_05_model_gateway_and_routing.md` and `layer_06_inference_and_serving.md`), sandboxes (see `layer_03_tools_and_protocols.md`), traces (see `concern_02_observability.md`), and datasets and fine-tuned models (see `layer_10_data.md`). Test it by trying to reach another customer's data on purpose.
- [3] Permissions that change. People leave teams, documents get restricted and access gets revoked. Make sure each change reaches every copy the AI system keeps, such as search indexes, memories and caches, and any long-running task still acting for that user.
- [2] Who can change the AI's behavior. Prompts, tool definitions, model settings and routing rules decide what the product does. Limit who can change them in production, and review changes like code (see `layer_04_context_memory_and_retrieval.md`).

Check: For one feature, list every place the model reads data or takes actions, and show where the user's permissions are checked in code for each. Then, as a test user, try to reach another user's and another customer's data through the feature, including with injected instructions, and show that every attempt fails.

## Secrets [3]

Secrets are API keys, passwords, tokens and signing keys. Storing and rotating provider keys is covered in `layer_05_model_gateway_and_routing.md`, keeping tool credentials out of the context in `layer_03_tools_and_protocols.md`, and keeping secrets out of system prompts in `layer_08_foundation_models.md`. This section covers the other ways secrets leak from AI systems.

- [3] How secrets travel. AI systems copy text into many places, and a secret travels with the text: from a document into a search index, from a tool result into a trace, from a log into a training set (see `layer_10_data.md`), or from a sandbox's environment variables into code the model wrote. Anything in the context can also come out in the output, or be taken by an injected instruction.
- [3] Secrets users share. Users paste keys, passwords and tokens into chats and uploads. Detect and mask them before they're stored, indexed, remembered or logged.
- [3] Scanning beyond code. Scan for secrets wherever text collects, not only in code: documents before indexing, tool results, memories, logs and traces (see `concern_02_observability.md`). Mask what you find, and rotate any real secret that was exposed.
- [3] Developer machines. MCP server configurations, agent tools and local scripts often keep tokens in plain files, where any local program or coding agent can read them. Keep them in the operating system's credential store or a secrets manager, and give each token the narrowest scope that works.

Check: Plant a fake key in a document, a chat message and a tool result. Show that each is caught and masked before it reaches an index, a memory, a log, a trace or the model's output.

## Preventing data theft [3, some topics 4]

Attackers want the data your product can reach: users' conversations and files, customers' documents and your own data. With AI, they often don't need to break in, because they can get the model to hand the data over. Leaks through rendered output are covered in `layer_01_product_and_ux.md` and `layer_04_context_memory_and_retrieval.md`, leaks through tool arguments in `layer_03_tools_and_protocols.md`, and leaks from fine-tuned models in `layer_10_data.md`. This section covers closing every way out.

- [4] Mapping every way out. List every channel data can leave through: rendered output that loads images or links, tools that send email, post messages or fetch URLs, code sandboxes and browsers with network access, webhooks, shared links, and exported logs and traces. For each, decide in code what it may reach, such as a list of allowed domains, no network access by default, or a person's approval before anything goes outside. A channel you didn't list is one nobody is guarding.
- [3] Scanning what goes out. Check outputs and outgoing tool arguments for secrets, personal data and confidential material before they leave, and block or mask what shouldn't go. Pattern matching catches keys and card numbers, and classifiers catch more. Neither catches data the model has summarized or reworded, so treat scanning as one defense among several.
- [3] Slow leaks. An attacker doesn't need one big transfer. An agent can be made to leak a little at a time, such as a few characters in each search query or web address it requests. Limits on volume help, but the reliable fix is keeping sensitive data away from agents that can reach outside (see the dangerous combination in `layer_03_tools_and_protocols.md`).
- [2] Canary data. Plant fake records, such as a fake customer or a fake API key, in sensitive sources, and alert if they ever appear in outputs, tool calls or outside systems. They show that a leak happened, and where it came from.

Check: Map every way data can leave one agent. Use injected instructions to try to send private data out through each channel, including an image link in the output, and show that each attempt is blocked. Then plant a canary record and show that an attempt to leak it raises an alert.

## Supply chain [3]

Your product depends on models, tools, packages and data that other people publish, and each is a possible way in. MCP servers and tool packages are covered in `layer_03_tools_and_protocols.md`, model files in `layer_08_foundation_models.md`, and poisoned training data in `layer_10_data.md`.

- [3] Everything is a dependency. Model providers, open-weight models, MCP servers, agent frameworks, packages, prompt templates from shared hubs and public datasets can all be compromised or change without notice. Use official sources, pin versions, review changes before upgrading, and keep a list of what you depend on.
- [3] Invented packages. Models sometimes suggest packages that don't exist, and attackers register those names with malicious code (slopsquatting). Check that every package a model or coding agent adds is the real one you meant before installing it.
- [3] Coding agents on your machine. Coding agents run commands, read files and install packages with your permissions, and they can be injected through what they read, such as a README, an issue or a web page. Limit what they may run without asking (see `layer_03_tools_and_protocols.md`), keep production credentials off the machines they run on, and review what they change.
- [2] Models from others. Open-weight models and fine-tunes can hide planted behaviors that appear only on certain inputs (see `layer_08_foundation_models.md`), and inspection can't reliably find them. Prefer well-known publishers, and run your security tests on any model you download.
- [2] Knowing what you run. Keep a list of the models, datasets, tools and versions in each product, sometimes called an AI bill of materials. When a model or package turns out to be compromised, it tells you what's affected.

Check: List every model, MCP server, framework and AI-related package one product depends on, with its version and source. Pin anything that isn't pinned. Then show how a package a coding agent adds gets checked before it's installed.

## Security testing [3]

Red-teaming as part of evaluation, and testing that the product refuses harmful requests, are covered in `concern_01_evaluation.md`. This section covers testing security controls.

- [3] Testing the controls, not the model. A security test should pass because a control in code stopped the attack, such as a permission check, a blocked domain or an approval step, not because the model refused. To check, rerun your attack cases with a prompt that tells the model to follow any instruction it reads. If an attack then succeeds, the model was the only thing stopping it.
- [3] Attacks for every entry point. Build test cases with injected instructions for every way outside text reaches the model, including attempts to reach other users' data and to send data out. Run them before every release, like any regression set (see `concern_01_evaluation.md`).
- [3] Attackers adapt. Published defenses often fail once an attacker designs attacks against them. Don't rely only on lists of known attacks: have people, and models prompted to attack, craft attacks aimed at your own product.
- [3] Benchmarks and tools. Benchmarks such as AgentDojo measure how often agents fall for injection, which helps when you compare models and defenses. Tools such as garak, PyRIT and promptfoo run common attacks automatically.
- [2] Specialists. Before launching features that take real actions or reach sensitive data, have security specialists review the design and test it. Some companies also pay outside researchers who report problems (a bug bounty).

Check: Build a security test suite for one agent, with injection cases for every entry point, including cases that try to reach other users' data and to send data out. Run it with a prompt that tells the model to obey every instruction, and show that each attack is still stopped by a control in code.

## Responding to attacks [3]

Some attacks will get through. This section covers noticing them and limiting the damage. Spotting attacks in traces and metrics is covered in `concern_02_observability.md`.

- [3] Logging security events. Record every blocked action, permission denial, classifier flag, approval and rejection, with its trace ID, user and agent. These records are what you investigate after an attack, and patterns in them reveal attacks in progress.
- [3] Kill switches. Be able to turn off a tool, an MCP server, an agent or a whole AI feature quickly, without a deploy, and test that it works (see `layer_02_agents_and_orchestration.md`).
- [3] Cleaning up. After an attack, revoke the keys and tokens involved (see `layer_05_model_gateway_and_routing.md`), remove planted content from indexes, memories and caches (see `layer_04_context_memory_and_retrieval.md`), and use traces and audit logs to find what the attacker saw and did.
- [3] Learning from it. Turn each attack into a test case (see Security testing), and fix the control that failed, not just the prompt. A data leak through an AI feature is a breach like any other, with the same duties to report it (see Governance and safety below).

Check: Run a drill in which an injected instruction in a shared document has made an agent leak data. Use your security events and traces to find every affected request, turn off the feature, revoke the credentials involved, remove the document, and add the attack to your test suite.

## Security at each layer

Each layer file covers security as it applies to that layer. This table shows where.

| Layer | What to secure there | Level | Where |
|---|---|---|---|
| 1. Product and UX | Rendering output safely, abuse through the product, untrusted uploads, shared links | 3 | `layer_01_product_and_ux.md` |
| 2. Agents and orchestration | Trust between agents, separating reading from acting, permissions for each agent, goal drift | 3 | `layer_02_agents_and_orchestration.md` |
| 3. Tools and protocols | Untrusted tool results, the dangerous combination, tool poisoning, designs that resist injection, permissions, sandboxes | 3 (some topics 4) | `layer_03_tools_and_protocols.md` |
| 4. Context, memory and retrieval | Indirect injection, permission-aware retrieval, keeping customers apart, memory poisoning, leaks through output | 3 | `layer_04_context_memory_and_retrieval.md` |
| 5. Model gateway and routing | Provider keys and secrets management, limits per user, denial of wallet, the gateway as a checkpoint | 2–3 | `layer_05_model_gateway_and_routing.md` |
| 6. Inference and serving | Exposed self-hosted endpoints, caches shared between customers | 2–3 | `layer_06_inference_and_serving.md` |
| 7. Model adaptation | Fine-tuning as an attack | 2 | `layer_07_model_adaptation.md` |
| 8. Foundation models | Why models fall for injection, jailbreaks, system prompt leaks, trusting model files | 2–3 | `layer_08_foundation_models.md` |
| 9. Compute | Shared hardware, confidential computing | 1 | `layer_09_compute.md` |
| 10. Data | Secrets and personal data leaking into datasets, data poisoning, feedback manipulation | 2 | `layer_10_data.md` |

Check: For one feature, list every layer it touches and the risks that layer's file names. Mark which have a control in code today, and add one for the most serious gap.

## Other cross-cutting concerns in security

### Governance and safety [2–3]

- [3] Where security ends and safety begins. Security defends against attackers. Governance and safety makes the system behave acceptably when nobody is attacking. Many controls serve both, such as classifiers, approvals and audit logs, so plan them together.
- [2] Rules that require security. The EU AI Act requires high-risk systems to resist attempts to manipulate them, including data poisoning and inputs designed to cause mistakes. Data protection laws such as GDPR treat a leak through an AI feature like any other breach, with deadlines for reporting it. Bring in legal and your security team.
- [2] Customers' security reviews. Enterprise customers increasingly ask how your AI features handle prompt injection, which providers see their data, whether it's used for training, and how they're kept apart from other customers. Keep written answers current.

### Reliability [2–3]

- [3] Failing safe. When a security check fails or times out, block actions that are risky or can't be undone, rather than let them through. For low-risk features, you may choose to carry on and log the failure.
- [2] Attacks on availability. Floods of long or expensive requests can slow the product down for everyone. Limits per user and per key (see `layer_05_model_gateway_and_routing.md`) keep one attacker from affecting others.

### Cost and latency [3]

- [3] Paying for checks. Classifiers, second model calls and approval steps add cost and delay. Match checks to risk: run cheap checks everywhere and expensive ones where the stakes are high, and run checks alongside the main call where you can (see `layer_02_agents_and_orchestration.md`).

Protecting traces, and spotting attacks in traces and metrics, are covered in `concern_02_observability.md`. Red-teaming, and judges that injected text can fool, are covered in `concern_01_evaluation.md`.

Data quality meets security where data is poisoned on purpose: in training data (see `layer_10_data.md`) and in indexes and memories (see `layer_04_context_memory_and_retrieval.md`).

## Not in this concern

- Security risks specific to each layer, such as tool poisoning or permission-aware retrieval, are covered in each layer's file (see Security at each layer).
- Guardrails against harmful or biased output, privacy and personal data, and audit trails are covered in `concern_04_governance_and_safety.md`. This file covers defending against attackers.
- Red-teaming as part of evaluation, and testing that the product refuses harmful requests, are covered in `concern_01_evaluation.md`.
- Protecting traces, and spotting attacks in traces and metrics, are covered in `concern_02_observability.md`.
- Storing and rotating provider keys, and limits on what each user can spend, are covered in `layer_05_model_gateway_and_routing.md`.
- Sandboxes, OAuth, and permissions for agents and their tools are covered in `layer_03_tools_and_protocols.md`.
- Classic application and network security, and company-wide security programs, are disciplines of their own. This file covers what changes when AI is involved.
- Attacks on models during training, such as poisoning pre-training data or stealing model weights, are lab and provider work. Awareness is enough.

## Learning order

1. Foundations, especially why the model can't be a security boundary.
2. Access and identity. Get the user's permissions right before anything else.
3. Prompt injection.
4. Preventing data theft.
5. Secrets.
6. Jailbreaks and abuse, before a public launch.
7. Supply chain, before you connect outside tools, servers or models.
8. Security testing, before launch and after every major change.
9. Responding to attacks.

Learn each layer's part of security when you work on that layer.

## Key reading

- OWASP Top 10 for LLM Applications (2025), and OWASP Top 10 for Agentic Applications (2026).
- Simon Willison's writing on prompt injection, including "The lethal trifecta for AI agents" (2025).
- Greshake et al., "Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection" (2023).
- Beurer-Kellner et al., "Design Patterns for Securing LLM Agents against Prompt Injections" (2025), and Debenedetti et al., "Defeating Prompt Injections by Design" (Google DeepMind, 2025).
- Hines et al., "Defending Against Indirect Prompt Injection Attacks With Spotlighting" (Microsoft, 2024).
- Wallace et al., "The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions" (OpenAI, 2024).
- Nasr et al., "The Attacker Moves Second: Stronger Adaptive Attacks Bypass Defenses against LLM Jailbreaks and Prompt Injections" (2025), on testing defenses against attackers who adapt.
- Debenedetti et al., "AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents" (2024).
- Anil et al., "Many-shot Jailbreaking" (Anthropic, 2024), and Sharma et al., "Constitutional Classifiers: Defending against Universal Jailbreaks across Thousands of Hours of Red Teaming" (Anthropic, 2025).
- Spracklen et al., "We Have a Package for You! A Comprehensive Analysis of Package Hallucinations by Code Generating LLMs" (2024).
- NIST, "Adversarial Machine Learning: A Taxonomy and Terminology of Attacks and Mitigations" (NIST AI 100-2), and MITRE ATLAS.
