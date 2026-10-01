# Governance and safety: what an application-layer AI engineer needs to know

This file covers the Governance and safety concern from `ai_stack.md`. Governance and safety makes sure the system behaves acceptably even when no one is attacking it. It covers guardrails against harmful or biased output, privacy and handling of personal data, regulatory compliance, and audit trails. It also covers intellectual property, where data came from, and human oversight of what agents do. It is one of the seven cross-cutting concerns: not a layer, but something every layer needs. This file is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this concern and how deeply to learn each one.

## The rule for depth

Security asks what happens when someone attacks the system. Governance and safety asks whether it behaves acceptably when nobody does, and whether you can show that it does. AI features can harm people with no attacker involved: a confident wrong answer about a medication, a hiring screen that favors one group, a memory that keeps what a user asked to forget, or an agent that acts without approval.

What counts as acceptable comes from laws, contracts, your company's policies and what the people affected can reasonably expect. Legal, privacy and policy teams interpret those. You build the controls that make the product follow them, and you're often the first to see where it doesn't.

For every AI feature, governance and safety answers four questions:

1. What must it never do, and what must it handle with care? (Foundations, Content policy and guardrails, Fairness)
2. What happens to people's data? (Privacy and personal data)
3. Which laws apply, and whose rights are involved? (Regulation, Intellectual property)
4. Who is responsible, and can you show what happened? (Oversight and accountability, Audit trails and documentation)

Most concepts in this concern are level 3, and two things are level 4:
- writing each feature's content policy precisely enough to enforce and test
- mapping everywhere personal data goes, since only engineers know every copy

Learn regulation and intellectual property to level 2: know when a law or a right applies and what it asks of the product, and let legal decide how it applies to you.

The layer files cover governance and safety as it applies to each layer, such as human approval for agent actions in Layer 3 and deleting data from indexes and memories in Layer 4. This file covers what they share, and Governance and safety at each layer maps where the rest is covered.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Foundations | 3 | Acceptable is set by laws, contracts, policies and the people affected |
| Content policy and guardrails | 3 (some topics 4) | Every feature needs written rules and checks that enforce them |
| Fairness | 3 | AI can work worse for some groups without anyone noticing |
| Privacy and personal data | 3 (some topics 4) | AI systems copy personal data into many places |
| Regulation | 2 (some topics 3) | Laws change fast, and legal decides how they apply |
| Intellectual property | 2 | Rights to inputs and outputs are partly unsettled |
| Oversight and accountability | 3 | Someone must own what each feature does |
| Audit trails and documentation | 3 | You must be able to show what happened and why |
| Security | 3 | Most privacy breaches start as security failures |
| Observability | 3 | Harm often shows up in production before anyone reports it |
| Reliability | 2–3 | Safety checks must fail safe |
| Cost and latency | 3 | Every guardrail adds cost and delay |

## Foundations [3]

These ideas come up in every other section, so learn them first.

- [3] Where "acceptable" comes from. It comes from laws and regulations, from contracts (your terms of service, your customers' agreements and your model providers' usage policies), from your company's own policies, and from what the people your product affects can reasonably expect. When these conflict or are unclear, bring in legal, privacy and policy colleagues. They decide what's acceptable, and you make the product follow it.
- [3] Naming the harms. For each feature, list what could go wrong when no one is attacking it: harmful or dangerous content, wrong information that people act on in health, law or money, unfair treatment of some groups, privacy violations, infringing others' rights, manipulation or unhealthy dependence, and agents taking actions nobody approved. Name who could be harmed: users, people mentioned in the data, your customers and the public.
- [3] Effort matched to risk. A tool that drafts internal meeting notes needs lighter controls than one that screens job applicants or answers medical questions. Rate each feature by how badly and how many people a failure could hurt, whether the harm can be undone, and whether its output feeds a decision about a person. The EU AI Act sorts systems into risk tiers in a similar way (see Regulation).
- [3] Showing, not just doing. Governance means being able to show that the system behaves acceptably, through written policies, test results, records of who approved what and logs of what the system did. Regulators, auditors and customers ask for evidence, not assurances.
- [2] Who does what. Legal interprets laws and contracts, privacy teams own data protection, security teams defend against attacks (see `concern_03_security.md`), and trust and safety or policy teams write content rules. You build the controls, explain how the system actually works, and raise problems early.

Check: For one AI feature, list who could be harmed and how, when no one is attacking it, and rate the feature's risk. For each harm, name the policy that covers it, the control that enforces it and the evidence that shows the control works.

## Content policy and guardrails [3, some topics 4]

A content policy says what a feature may and may not produce. Guardrails are the checks that enforce it. Testing guardrails is covered in `concern_01_evaluation.md`, and defending them against jailbreaks in `concern_03_security.md`.

- [4] A written policy for each feature. Write down what the feature must never produce, what it must handle with care, and what it should do even though it looks risky. For example, a pharmacy assistant should discuss drug doses, must point anyone in crisis to help, and must never give instructions for misuse. Make the policy specific enough that two reviewers would label an output the same way, because it's what your guardrails enforce and your tests check (see `concern_01_evaluation.md`).
- [3] Layers of guardrails. The model's safety training and the provider's filters come first (see `layer_08_foundation_models.md`). Your system prompt sets the feature's rules. Separate checks on inputs and outputs catch what gets past both, and checks on actions stop what agents shouldn't do (see `layer_03_tools_and_protocols.md`). No single layer is enough.
- [3] Guardrail tools. Options include provider moderation endpoints and safety settings, open classifiers such as Llama Guard, frameworks such as NeMo Guardrails and Guardrails AI, cloud services such as Amazon Bedrock Guardrails and Azure AI Content Safety, and small classifiers you train yourself (see `layer_07_model_adaptation.md`). Most let you choose which categories to block and how strictly.
- [3] Tuning for both kinds of mistake. A guardrail that blocks too little lets harm through. One that blocks too much refuses legitimate requests and drives users away. Measure both on labeled examples (see `concern_01_evaluation.md`), and set each category's threshold by the harm it guards against.
- [3] When a guardrail fires. Tell the user plainly what the product can't help with, without accusing them, and offer an alternative or a person where it makes sense (see `layer_01_product_and_ux.md`). Record the event, so you can review patterns and requests that were blocked by mistake.
- [3] Streaming and output checks. When output streams to the user, a harmful passage can appear before the output check finishes. For high-risk features, check the output in chunks as it streams, or hold it back until it passes.
- [3] Users at risk. Some users are in crisis, some are children, and some form attachments to the product. Decide how the product responds to them (see sensitive situations in `layer_01_product_and_ux.md`), and design against flattery and dependence (see respecting users in the same file). Laws increasingly set rules here, especially for products that children may use or that act as companions.

Check: Write the content policy for one feature, with examples of what it must never produce, what it must handle with care and what it should help with anyway. Put an input and an output guardrail in place, and measure on 100 labeled examples how many harmful outputs they miss and how many legitimate requests they block. Change one threshold, and show that both numbers move as you expected.

## Fairness [3]

AI features can work worse for some groups, or treat them unfairly. How models reflect bias is covered in `layer_08_foundation_models.md`, gaps in datasets in `layer_10_data.md`, and testing for fairness in `concern_01_evaluation.md`. This section covers deciding what fairness means for a feature and acting on it.

- [3] Kinds of unfairness. A feature can make worse decisions about some groups, such as when ranking job applicants or flagging fraud. It can work less well for some users, such as speakers of other languages or dialects, or people with accents in voice products. And it can describe groups through stereotypes in what it writes or draws.
- [3] Choosing what to check. Decide with people who know the feature's users which groups and kinds of unfairness matter. Cover characteristics that the law protects, such as gender, ethnicity, age and disability, and the languages and regions you serve.
- [3] Decisions about people. When AI helps decide something about a person, such as hiring, lending, insurance, housing, education or access to services, the stakes and the legal duties rise sharply. Keep a person responsible for each decision, check outcomes by group, and expect the rules in Regulation to apply.
- [3] Fixing what you find. Change the prompt or examples, add data for groups the feature serves poorly, send some cases to a stronger model or to people, or narrow what the feature does. Then measure again. Removing a protected attribute from the input rarely removes the bias, because other details stand in for it.
- [2] Laws on bias. Anti-discrimination law applies to decisions made with AI just as it does to any other. Some places add specific rules, such as New York City's requirement for yearly bias audits of automated hiring tools.

Check: For one feature, list the groups and kinds of unfairness that matter. Measure quality or outcomes for each group on your evaluation set, find the largest gap, and make one change that narrows it.

## Privacy and personal data [3, some topics 4]

Personal data is any information about a person who can be identified. AI systems copy it into many places: prompts, provider logs, traces, search indexes, memories, caches, datasets and fine-tuned models. Personal data in datasets is covered in `layer_10_data.md`, in indexes and memories in `layer_04_context_memory_and_retrieval.md`, in traces in `concern_02_observability.md`, and users' controls over their data in `layer_01_product_and_ux.md`. This section covers the whole picture.

- [4] A map of where personal data goes. For each feature, list every place personal data is sent or stored: the model provider, the gateway's logs (see `layer_05_model_gateway_and_routing.md`), traces, indexes, memories, caches, evaluation sets, training data and every vendor in between. Note how long each keeps it, who can see it and which country it's in. Deletion, retention, consent and privacy reviews all depend on this map, and only engineers know every copy.
- [3] Only what the task needs. Send the model only the personal data the task needs, mask or remove the rest before it leaves (see `layer_05_model_gateway_and_routing.md`), and keep it only as long as needed. Data you never collect can't leak or be misused.
- [3] Purpose and consent. Use personal data only for the purposes users were told about and, where the law requires it, agreed to. Using conversations to improve the product or train models is a separate purpose that needs its own basis (see `layer_10_data.md`).
- [3] People's rights. Privacy laws such as GDPR give people rights to see, correct and delete their data, and to object to some uses. A request must reach every copy on your map, including derived ones such as summaries and extracted facts. Data inside a fine-tuned model can't be reliably removed without retraining, so keep personal data out of training sets.
- [3] Special categories. Health, biometric, financial and children's data, and data about beliefs or sexuality, carry stricter rules, such as HIPAA for health data in the US. Check that every provider that handles such data has agreed in its contract to the rules that apply.
- [2] Privacy reviews. A feature that processes personal data in a new way often needs a formal assessment, such as a data protection impact assessment under GDPR. The privacy team runs it, and your data map is its starting point.
- [2] What models can infer. Models can guess sensitive facts, such as a health condition or political views, from ordinary data, and memory features can bring those guesses up later. Treat inferred sensitive facts like collected ones (see `layer_04_context_memory_and_retrieval.md`).

Check: Draw the personal-data map for one feature, with retention, access and location for each place. Then delete one test user's data, and show that it's gone from every place on the map, including traces, indexes, memories and evaluation sets.

## Regulation [2, some topics 3]

AI law changes quickly and differs by country. This section aims at recognizing when a law applies and what it asks of the product. Legal decides how it applies to you.

- [2] The EU AI Act. It sorts AI systems by risk. A few uses are banned, such as social scoring and manipulation that exploits people's vulnerabilities. High-risk uses, such as hiring, credit, education and access to essential services, carry duties for risk management, data quality, documentation, logging, human oversight, accuracy and security. Systems that talk to people or generate content have transparency duties, and most others have few or none. Duties also differ between companies that make AI systems and companies that use them (see `layer_08_foundation_models.md` and `layer_07_model_adaptation.md`). Its obligations phase in over several years, and a 2026 amendment pushed back the dates for high-risk systems, so check the current schedule.
- [3] Transparency you build. Tell people when they're talking to an AI, label content the AI made where the law requires it (see `layer_01_product_and_ux.md`), and mark generated content in a machine-readable way where required. Standards such as C2PA content credentials attach a record of origin to media, and watermarks such as Google's SynthID hide a mark in the content itself. Metadata is easily stripped and watermarks can be weakened, so treat both as signals, not proof.
- [3] Duties that land in the product. Most legal duties for application teams turn into product features: disclosures, logs, documentation, human review steps, ways for people to contest decisions, data deletion and storing data in the right region. Build them so that legal can point to them.
- [2] Other laws. Data protection laws such as GDPR apply to AI like any other processing, including limits on decisions made solely by automated means. Sector rules apply in health, finance and employment. Many other places regulate AI too, such as China's rules for generative AI services and US state laws on automated decisions, chatbots and deepfakes. Know which places your product operates in.
- [2] Staff knowledge. Some laws, such as the EU AI Act, expect companies to take steps so that staff who build or use AI systems understand them well enough for their roles. Teams often meet this with training on how their AI systems work and fail.
- [2] Frameworks and standards. NIST's AI Risk Management Framework and its profile for generative AI, and ISO/IEC 42001 for AI management systems, give companies a structure for governing AI. Enterprise customers increasingly ask whether you follow one.

Check: For one feature, list the places it's offered and the laws that may apply there, and confirm the list with legal. For each duty that applies, show the product feature that meets it, such as a disclosure, a log or a review step.

## Intellectual property [2]

Intellectual property covers who owns what goes into the system and what comes out. Licenses for models are covered in `layer_07_model_adaptation.md` and `layer_08_foundation_models.md`, rights to indexed content in `layer_04_context_memory_and_retrieval.md`, and dataset licenses and provider terms on outputs in `layer_10_data.md`. This section covers the questions they share.

- [2] Copying in outputs. Models can reproduce text, code or images they were trained on, sometimes word for word (see `layer_10_data.md`). A copied code snippet can bring its license with it. Some providers offer filters that block outputs matching public code, and some promise to defend customers against copyright claims over outputs (see `layer_08_foundation_models.md`).
- [2] Who owns the output. In many places, including the US, material made by AI without meaningful human creative input can't be copyrighted. If your product sells generated content, tell customers what rights they get, and check that your provider's terms allow it.
- [2] What you put in. Check that you may use every input: documents you index, data you fine-tune on, sites your agents read or operate (see `layer_03_tools_and_protocols.md`), and other providers' model outputs used for training (see `layer_10_data.md`).
- [2] Real people and brands. Generated images, voices and text can imitate real people, artists' styles or trademarks. Get consent before cloning anyone's voice or likeness, and block requests to impersonate real people where the harm or legal risk is high.
- [1] Unsettled law. Courts are still deciding lawsuits over training data and outputs, and the rules may shift. Know that the questions are open, and bring in legal before launching features that depend on the answers.

Check: For one feature, list every input it uses and every kind of output it produces, with the license or terms that cover each. Find one input whose rights you can't confirm, and resolve it with legal.

## Oversight and accountability [3]

Someone must be responsible for what each AI feature does, and people must be able to see and correct what agents do. Where people step into agent workflows is covered in `layer_02_agents_and_orchestration.md`, which actions need approval in `layer_03_tools_and_protocols.md`, and the screens people use in `layer_01_product_and_ux.md`. This section covers the governance around them.

- [3] An owner for every feature. Name one person responsible for each AI feature's behavior: its policy, its guardrails, its evaluation results and its incidents. Without an owner, problems that cross teams, such as a provider change that weakens a guardrail, belong to nobody.
- [3] Oversight that works. A person reviewing AI output helps only if they have the time, the information, the authority and the skill to disagree. Reviewers who approve nearly everything give false comfort (see review fatigue in `layer_02_agents_and_orchestration.md`). Measure how often they change or reject what the AI proposes.
- [3] Contesting decisions. When AI contributes to a decision about a person, tell them, explain the main reasons in plain words, and give them a way to have a person review it. Some laws require this.
- [3] Reviews scaled to risk. Before a feature launches, review its purpose, harms, data, evaluation results, guardrails, oversight and disclosures, with more depth and more sign-offs as the risk rises. For a low-risk feature, a one-page review is enough.
- [3] Approving what you build on. Review new model providers, tools, MCP servers and vendors for their data terms, security, licenses and regions before anyone uses them, and enforce the approved lists in code (see `layer_03_tools_and_protocols.md` and `layer_05_model_gateway_and_routing.md`).
- [3] Changes that need review. Decide which changes need a fresh review, such as a new model, a new tool that can act, a new kind of data or user, or a change to a guardrail. A model upgrade can change behavior without any change to your code (see `layer_08_foundation_models.md`).
- [3] Safety incidents. Define what counts as an incident, such as harmful output reaching users, a privacy leak, an unfair decision or an agent acting without approval. Give users a way to report problems, decide who responds and who must be told, including regulators and customers where laws or contracts require it, and turn each incident into test cases (see `concern_01_evaluation.md`).
- [2] Staff use of AI tools. Employees use AI tools with company data whether or not the tools are approved. Publish which tools staff may use with which data, and make the approved ones easy to get.

Check: Name the owner of one feature and run a launch review scaled to its risk. Define what counts as an incident for it, then run a drill: report a harmful output, follow it through response and notification, and add it to the evaluation set.

## Audit trails and documentation [3]

Audit trails record what the system did. Documentation records what the system is and how it was tested. Audit logs of agent actions are covered in `layer_03_tools_and_protocols.md`, records of agent decisions in `layer_02_agents_and_orchestration.md`, and why traces aren't audit logs in `concern_02_observability.md`. This section covers what they have in common.

- [3] What an audit trail shows. For every consequential output or action, it shows who asked, what the system did and why (the inputs, sources, model and prompt versions, and tool calls), what changed and who approved it. Unlike traces, audit trails must be complete, protected from changes and kept for a set period.
- [3] Records about the system. Keep records of decisions about the system itself: which models, tools and prompts were approved and by whom, evaluation results at each release (see `concern_01_evaluation.md`), launch reviews, policy versions and incidents. Months later, someone will ask why a version shipped.
- [3] A card for each feature. Keep a short document for each AI feature that covers its purpose, users, models and providers, data, evaluation results, known limits, guardrails, human oversight and owner. Update it with each release. Cards for adapted models and datasets are covered in `layer_07_model_adaptation.md` and `layer_10_data.md`.
- [3] Where things came from. Record the source and terms of every model, dataset, document collection and tool you use (see lineage in `layer_10_data.md` and the AI bill of materials in `concern_03_security.md`). When a source turns out to be unlicensed, biased or compromised, the record shows what it affected.
- [2] Logging the law requires. The EU AI Act requires high-risk systems to log events automatically, and companies using them to keep those logs for at least six months. Other laws and contracts set their own retention rules.
- [2] Keeping records versus privacy. Audit trails favor keeping records for a long time, and privacy favors deleting them early. Resolve this by type of data: keep the record of what was done and by whom for longer, and keep personal content for as short a time as you can, or replace identities with pseudonyms.

Check: For one consequential action or output, show its audit record: who asked, what the system saw and did, which versions were used and who approved it. Then show the feature's card, and the evaluation results from the release that produced the output.

## Governance and safety at each layer

Each layer file covers governance and safety as it applies to that layer. This table shows where.

| Layer | What to govern there | Level | Where |
|---|---|---|---|
| 1. Product and UX | Users' controls over their data, sensitive situations, telling people they're dealing with AI, respecting users, accessibility | 2–3 | `layer_01_product_and_ux.md` |
| 2. Agents and orchestration | Where people step in, stopping agents, records of decisions, unattended agents | 3 (some topics 4) | `layer_02_agents_and_orchestration.md` |
| 3. Tools and protocols | Which actions need approval, rules for human oversight, approved tools and servers, accountability, terms of the sites agents use | 3 (some topics 4) | `layer_03_tools_and_protocols.md` |
| 4. Context, memory and retrieval | Personal data in indexes and memories, deletion, consent for memory, rights to indexed content, where data lives | 3 | `layer_04_context_memory_and_retrieval.md` |
| 5. Model gateway and routing | Approved models and regions, removing personal data before it leaves, a record of which provider handled each request | 2–3 | `layer_05_model_gateway_and_routing.md` |
| 6. Inference and serving | What serving logs keep | 2 | `layer_06_inference_and_serving.md` |
| 7. Model adaptation | Safety after fine-tuning, licenses, provider review, becoming a provider under the law, model cards | 2–3 | `layer_07_model_adaptation.md` |
| 8. Foundation models | Usage policies, provider and deployer roles, bias, licenses of open models, who owns outputs | 2 | `layer_08_foundation_models.md` |
| 9. Compute | Data residency, sovereign hosting, energy and carbon | 1–2 | `layer_09_compute.md` |
| 10. Data | Personal data in datasets, consent and retention, provider data policies, dataset licenses, data cards | 2–3 | `layer_10_data.md` |

Check: For one feature, list every layer it touches and the governance duties that layer's file names. Mark which are met today, with evidence for each, and close the most important gap.

## Other cross-cutting concerns in governance and safety

### Security [3]

- [3] Privacy depends on security. Most privacy breaches start as security failures, such as an exposed storage bucket or a missing permission check. Make sure the security controls (see `concern_03_security.md`) cover every place on your personal-data map.

### Observability [3]

- [3] Watching for harm. Track guardrail blocks, user reports of harmful output and conversations on sensitive topics, by feature and model version. A rise after a model upgrade is a safety regression, even if nothing else changed (see `concern_02_observability.md`).

### Reliability [2–3]

- [3] Failing safe. If a guardrail fails or times out, block high-risk output rather than show it unchecked. For low-risk features, you may choose to carry on and log the failure.
- [2] Consistent decisions. When AI helps decide something about people, similar cases should get similar outcomes. Answers that change from run to run (see `layer_08_foundation_models.md`) are a fairness problem as well as a quality one.

### Cost and latency [3]

- [3] What safety costs. Every guardrail adds cost and delay. Run cheap checks on every request and expensive ones only where the risk calls for them, and run checks alongside the main call where you can (see `layer_02_agents_and_orchestration.md`).

Testing for harms, refusals of reasonable requests and fairness, and keeping records of testing, are covered in `concern_01_evaluation.md`.

Data quality meets governance in documentation and lineage: data cards, records of where each example came from, and whether datasets represent the people the product serves (see `layer_10_data.md`).

## Not in this concern

- Governance duties specific to each layer, such as which agent actions need approval or deleting data from indexes, are covered in each layer's file (see Governance and safety at each layer).
- Defending against attackers, including jailbreaks and prompt injection, is covered in `concern_03_security.md`. This file covers acceptable behavior when no one is attacking.
- Testing for harms, refusals of reasonable requests and fairness is covered in `concern_01_evaluation.md`. This file covers the policies those tests check.
- Personal data in traces, and choosing observability vendors, are covered in `concern_02_observability.md`.
- Designing handoffs between agents and people is covered in `layer_02_agents_and_orchestration.md`, and approval screens in `layer_01_product_and_ux.md`.
- Personal data in datasets, permission to use product data, and dataset licenses are covered in `layer_10_data.md`.
- Interpreting laws and contracts is legal work. This file covers recognizing when they apply and building what they require.
- Safety training of models and testing frontier models for dangerous capabilities are lab work. Awareness is enough.

## Learning order

1. Foundations, especially matching effort to risk.
2. Content policy and guardrails.
3. Privacy and personal data. Draw the data map before your first launch.
4. Oversight and accountability.
5. Audit trails and documentation.
6. Fairness, before any feature that affects decisions about people.
7. Regulation, with legal, for the places your product operates.
8. Intellectual property, before features that generate content others will publish or sell.

Learn each layer's part of governance and safety when you work on that layer.

## Key reading

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
