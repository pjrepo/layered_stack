# Layer 1, Product and UX: what an application-layer AI engineer needs to know

This file covers the Product and UX layer from `ai_stack.md`. That layer is where people use the system: apps, copilots, chat and voice interfaces, and AI built into existing software. It also collects user feedback, which flows back into Data. This file is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this layer and how deeply to learn each one.

## The rule for depth

This layer is where people meet what you've built. Product managers and designers usually lead it, and you share it with them. AI features differ from other software in two ways: their output is sometimes wrong in ways users can't always see, and every use takes time and costs money. You know better than anyone else on the team how and when your model fails, so the parts of the design that depend on that are yours.

For every AI feature, this layer answers four questions:

1. Is AI the right tool here, and what counts as success? (Choosing what to build)
2. How do users work with it? (Interaction patterns, Users in control of agents)
3. What do users see while it works, and when it's wrong? (Streaming and waiting, Designing for wrong answers, Setting expectations)
4. What do users do with the result, and what does that tell you? (Feedback and measurement)

Most concepts in this layer are level 3, and two things are level 4:
- designing every feature for the times the AI is wrong
- recording what users do with each result

Learn product strategy to level 2: you contribute to it, but product managers own it. Visual design and front-end engineering are disciplines of their own, so this file covers only what changes when AI is involved.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Choosing what to build | 3 | AI fits some problems well and others badly |
| Interaction patterns | 3 | A chat box is rarely the only or best interface |
| Streaming and waiting | 3 | Users judge speed by what they see, not by total time |
| Designing for wrong answers | 4 | Every AI feature is sometimes wrong, and users can't always tell |
| Setting expectations | 3 | Users who expect too much are let down, and those who expect too little never try |
| Users in control of agents | 3 | People must be able to see, steer and stop what agents do |
| Feedback and measurement | 3 (some topics 4) | Only your product can record what users do with each result |
| Governance and safety | 2–3 | Laws increasingly require disclosure, user controls and care for vulnerable users |
| Security | 3 | Model output shown in a browser can attack the user |
| Cost and latency | 3 | Heavy users can cost many times the average |
| Evaluation | 3 | Users judge the whole experience, not single outputs |
| Reliability | 3 | The rest of the product must keep working when AI fails |
| Observability | 3 | A user's complaint should lead straight to the trace |

## Choosing what to build [3]

Product managers usually decide what to build. You bring what they can't: how well models handle the task, how they fail and what each use costs.

- [3] Where AI fits. AI fits tasks where a good-enough result is valuable, the result is quick to check or cheap to fix, and the work is slow or tedious for people. It fits poorly where mistakes are costly and hard to spot, or where a simple rule would do the job.
- [3] Doing the task or helping with it. Decide whether the AI does the task for the user (automation) or helps the user do it, with drafts and suggestions (assistance). Assistance tolerates more errors, because a person checks every result. Automation needs higher accuracy and a way to catch mistakes. Start with assistance where mistakes are costly.
- [3] Defining success. Before building, decide what success means for the user and the business, and how you'll measure it, such as tasks completed, time saved or suggestions accepted. Link it to your evaluation set (see `layer_10_data.md`), so improvements you measure offline show up in the product.
- [3] Prototyping with real models. Judge AI features with real models and real data, not mockups, because quality can't be judged from a picture of the interface. A demo that works on a few hand-picked examples is far from a product that works for real users. Put a working prototype in front of real users early.
- [3] The cost of each use. Unlike most features, an AI feature costs money every time it's used. Check that the value of each use covers its cost at the volume you expect, including your heaviest users (see `layer_08_foundation_models.md`). This shapes pricing, such as charging by usage, credits or plan limits, which you'll decide with product and finance.
- [2] Designing for better models. Models improve every few months (see `layer_08_foundation_models.md`). Features that are barely possible today may soon be easy, and workarounds for today's weaknesses may become unnecessary. Design so that a better model makes the product better without a redesign.
- [2] Where lasting advantage comes from. Every competitor can use the same models. Lasting advantage comes from your data, from how deeply you fit into users' work, from distribution, and from the feedback loop that improves your product with use (see `layer_10_data.md`).

Check: For one proposed feature, write a one-page case covering the user's problem, whether AI does the task or assists, how you'll measure success, what happens when the AI is wrong, and the cost of each use. Build a working prototype with a real model and test it with five real users.

## Interaction patterns [3]

How users work with AI shapes what they can do with it. A chat box is the default, but often not the best choice.

- [3] Chat. Conversation is flexible and familiar, and suits open-ended requests. But it leaves users to work out what to ask, and it's slow for structured or repeated tasks.
- [3] AI inside existing software. Autocomplete, a button that drafts or summarizes, suggestions in context and sensible defaults meet users where they already work, without new habits. They're often more useful than a separate chat window.
- [3] Assistants beside the work. A side panel that sees what the user is working on, such as the open document or the selected code, and can act on it, while the user stays in charge. In some products, the user and the AI edit the same document or canvas side by side.
- [3] Handing tasks to agents. The user assigns a task, leaves, and comes back to a result, such as a report or a code change. The interface is about assigning, monitoring, reviewing and correcting, not chatting (see Users in control of agents).
- [3] Beyond free text. Forms, buttons, pickers, templates and file uploads often beat typing: they show users what's possible and give the model clean input. Show results in the form the task needs, such as a table, a diff or a draft in the editor, not only as paragraphs of text.
- [2] Generated interfaces. The model can choose interface components to show, such as a chart, a form or a booking card, instead of replying in text (see `layer_03_tools_and_protocols.md`). Limit it to components you've built and tested.
- [2] Voice. Voice needs fast replies, handling of interruptions and turn-taking, and short answers written to be spoken (see `layer_06_inference_and_serving.md` and `layer_08_foundation_models.md`). Learn this to level 3 if you build voice products.
- [2] Conversation features. Users expect chat products to offer a history they can search, the option to edit an earlier message and try again, and ways to copy, share and delete conversations.

Check: Take one feature that is currently a chat box and build another version as inline suggestions, a button or a form. With real users, compare how often each is used and how often tasks succeed.

## Streaming and waiting [3]

AI features are slow by software standards. Users judge speed by how soon something happens and whether they can see progress, not only by total time. How streaming works at the API level is covered in `layer_06_inference_and_serving.md`.

- [3] Streaming text. Show text as it arrives, so users start reading within a second or two. Render partial Markdown, code and tables without flicker or jumping layouts, and handle a stream that breaks partway.
- [3] Rendering results. Render Markdown, code with a copy button, tables, math and links cleanly, and render them safely (see Security below).
- [3] Something happens at once. Acknowledge every request immediately, then show progress, then the result. A blank screen for five seconds feels broken, even if the answer is good.
- [3] Showing progress. For multi-step work, show what's happening, such as "searching your documents" or "running tests", and let users see the plan. Don't present the model's raw reasoning as a reliable explanation of how it reached its answer (see `layer_08_foundation_models.md`).
- [3] Stop, edit and retry. Let users stop a response, edit their message and try again, or regenerate. Stopping should cancel the model call, not just hide the output.
- [3] Long tasks. For work that takes minutes, move it out of the conversation: show its status, let users leave, and notify them when it's done (see `layer_02_agents_and_orchestration.md`).
- [2] Streaming into components. Structured output can stream into interface components, such as filling in a form or a table field by field as the data arrives.

Check: Measure time to first visible text and total time for one feature, as users experience them. Add streaming and progress updates, and have users rate how fast it feels before and after. Show that the stop button cancels the model call.

## Designing for wrong answers [4]

Every AI feature is sometimes wrong, and users can't always tell. This is the biggest difference between designing AI features and other software, and it shapes most of the decisions in this layer.

- [4] Plan for mistakes. For each feature, decide how users will notice a mistake, how cheaply they can fix it, and what harm a mistake that nobody notices can do. Make mistakes cheap before trying to make them rare.
- [4] Easy to check. Put sources and citations next to the claims they support, link to the exact passage, highlight what changed, and show the inputs the AI used. Checking a result should take much less time than doing the task. Producing citations is covered in `layer_04_context_memory_and_retrieval.md`.
- [4] Easy to fix. Let users edit results in place, undo what the AI did, and tell it what's wrong. Drafts and suggestions that users accept are safer than changes applied directly.
- [4] Trust that matches reality. Users tend to trust AI too much, accepting wrong answers without checking, or too little, ignoring useful help after one mistake. Aim for trust that matches how reliable the feature actually is: show sources and limits, and make checking easy.
- [3] Showing uncertainty. A model's own statement of how sure it is means little (see `layer_08_foundation_models.md`). Show uncertainty through behavior instead: say "I couldn't find this in your documents", ask a clarifying question, offer alternatives or flag items for review.
- [3] When the AI can't help. When it declines, times out, fails or has nothing useful to say, show a clear message and a way forward: try again, a path that doesn't use AI, or a person. Never show a blank screen or a raw error, and never trap users with a bot that can't help them. What the system does when providers fail is covered in `layer_05_model_gateway_and_routing.md`.
- [3] Variety or consistency. The same request can give different answers. Decide what each feature needs: several options for creative work, stable answers for factual questions. Design for it, for example with a regenerate button, or with caching for answers that shouldn't change.

Check: List the five most likely mistakes of one AI feature. For each, show how a user would notice it and how many steps it takes to fix. Redesign the worst case, and test whether users catch a planted mistake before and after the change.

## Setting expectations [3]

Users who expect too much are let down, and users who expect too little never try. Both need an accurate picture of what the AI can do.

- [3] What it can and can't do. At first use, say what the feature is for, what it does well and where it falls short, in concrete terms, with example requests.
- [3] Starting points. An empty text box is hard to start from. Offer example requests, templates and suggested next steps based on what the user is doing.
- [3] What the AI can see and remember. Show what the AI has access to and what it used for each answer: which documents, which page, which memories. Let users see, edit and delete what the product remembers about them, and turn memory off (see `layer_04_context_memory_and_retrieval.md`). Otherwise users guess wrong in both directions.
- [3] Labeling AI. Make clear which content the AI produced, and when users are dealing with an AI rather than a person. Some laws require this (see Governance and safety below).
- [2] Voice and personality. The product's tone, such as formal, friendly or brief, is a product decision. Set it in the system prompt and keep it consistent across features. Don't pretend to be human, and don't flatter users (see `layer_08_foundation_models.md`).

Check: Watch five new users try a feature for the first time without help. Note what they expect it to do and where they get stuck. Change the introduction and starting points, and repeat with five more users.

## Users in control of agents [3]

Where people step into agent workflows is covered in `layer_02_agents_and_orchestration.md`, and which actions need approval in `layer_03_tools_and_protocols.md`. This section covers the screens people use to see, steer and stop agents.

- [3] Approval screens. Show exactly what the agent is about to do, with a preview such as the email text, the diff or the order. Say why approval is needed, and make approving, editing and rejecting equally easy.
- [3] Plans users can steer. Show the agent's plan before and during the work, and let users change it, skip steps or add constraints.
- [3] Stopping and taking over. Let users pause, stop or redirect an agent at any point, and take over the task themselves.
- [3] Reviewing results. When an agent finishes, show what it did, what changed, what it was unsure of and what needs the user's attention, not just "done". Link each claim to the detail behind it.
- [3] How much the agent does alone. Where it makes sense, let users choose how much the agent does without asking: ask before every action, ask only before risky ones, or act freely within set limits.
- [2] Many tasks at once. Users who run several agents need one place to see every task, its status and what's waiting for them.

Check: Build the approval and review screens for one agent. With five users, measure how long an approval takes and whether they catch a mistake planted in a preview. Improve the preview until most of them do.

## Feedback and measurement [3, some topics 4]

This layer collects the feedback that flows back into Data. Turning feedback into datasets and evaluation cases is covered in `layer_10_data.md`. This section covers collecting it and measuring what AI features do for users.

- [4] Recording what users do. Record what users do with every AI result: accept, edit, copy, regenerate, undo, abandon or ask for a person. Link each event to the result's trace, and to the prompt and model versions. Only the product can capture this, and if you don't record it at the time, it's gone.
- [3] Feedback controls. Offer thumbs up and down, ratings or "report a problem" where users judge a result, with an optional reason from a short list plus free text. Few users give feedback this way, and those who do aren't typical, so don't rely on it alone.
- [3] Edits as feedback. When users edit an AI draft, save both versions. The difference shows exactly what was wrong.
- [3] Product metrics. Measure what AI features do for users: how many use them, how often they come back, how often tasks succeed, how many suggestions are accepted, time saved and how often users escalate to a person. Also track guardrail metrics that shouldn't get worse, such as complaints, reports and support tickets.
- [3] Experiments. Test prompt, model and design changes on real users with controlled experiments (see `layer_05_model_gateway_and_routing.md`). AI outputs vary, so experiments can need more users or more time to show a real difference. Watch for novelty: use of a new AI feature often spikes, then falls.
- [3] Watching real use. Watch people use the feature, interview them, and read real conversations where users have agreed to it. Numbers show what happens. Watching shows why.
- [3] Offline and online agreement. Check that improvements on your evaluation set show up in product metrics. If they don't, your evaluation set doesn't reflect real use (see `layer_10_data.md`).

Check: Instrument one feature so that every result records accepts, edits, regenerations and abandons, linked to its trace. Build a weekly report of acceptance and task success. Run one experiment on a prompt or model change, and report whether your offline evaluation predicted the result.

## Cross-cutting concerns in this layer

### Governance and safety [2–3]

- [3] Data controls. Let users see and delete their conversations, choose whether their data is used to improve the product, and export it. Explain these choices in plain words where users make them. Permission to use data is covered in `layer_10_data.md`.
- [3] Sensitive situations. Decide how the product responds when conversations turn to self-harm or to serious medical, legal or financial risk, such as pointing to professional help or crisis lines, and test those responses. Products that children may use need extra care, and laws increasingly require it.
- [2] Transparency laws. The EU AI Act requires telling people when they're interacting with an AI system, and marking some AI-generated content. Other laws set similar rules. Know which apply to your product, and bring in legal.
- [2] Respecting users. Don't design AI features to maximize engagement at users' expense, for example through flattery, false urgency or encouraging emotional dependence.
- [2] Accessibility. Streaming text, voice and generated content must work with screen readers and keyboards, and accessibility laws apply to AI features like any other. Learn this to level 3 if you build the front end.

### Security [3]

- [3] Rendering output safely. Model output is untrusted. Rendered as HTML or Markdown, it can run scripts, or load images and links that send data to an attacker (see `layer_04_context_memory_and_retrieval.md`). Sanitize it, restrict which links and images load, and never run generated code in the user's browser outside a sandbox.
- [3] Abuse through the product. Some users will try jailbreaks, or try to use your product as free general-purpose AI. Limit message length, file size and use per user (see `layer_05_model_gateway_and_routing.md`), and watch for abuse.
- [3] Untrusted uploads. Files users upload can carry injected instructions or malware. Treat them like any untrusted input (see `layer_03_tools_and_protocols.md`).
- [2] Sharing. Shared conversation links can expose private data, and public links have ended up in search results. Make sharing deliberate, show what will be shared, and let users revoke links.

### Cost and latency [3]

- [3] Speed by interaction. Users expect different speeds from different interactions: autocomplete within a fraction of a second, a chat reply that starts within a second or two, a background task within minutes. Choose models and designs to match (see `layer_06_inference_and_serving.md`).
- [3] Cost per user. Heavy users can cost many times the average user. Track cost per active user and per plan, and use plan limits, rate limits and cheaper models for free tiers (see `layer_05_model_gateway_and_routing.md`).
- [3] Limits users understand. When users hit a limit, tell them what happened, when they can continue and what their options are.
- [3] Calls nobody asked for. AI that runs automatically, such as on every keystroke or page view, multiplies cost. Wait for a pause in typing, run on request where the value is low, and reuse results that haven't changed.
- [2] Precomputing. Generating results ahead of time, such as summarizing a document when it's saved rather than when it's opened, hides latency, but you pay for results nobody reads.
- [2] Letting users choose. Some products let users pick a model or a reasoning effort, trading speed and cost for quality. Most users shouldn't have to choose, so route automatically (see `layer_05_model_gateway_and_routing.md`) and offer the choice to those who want it.

### Evaluation [3]

- [3] The whole experience. Evaluation sets test model outputs. Also test the experience as users get it: the rendered answer, citations, error messages, slow networks and broken streams, on the devices your users have. Automate tests of the main flows.
- [3] Using it yourself. Use the product every day, and get colleagues to use it too. People inside the company find rough edges before customers do.

Product metrics and experiments are covered under Feedback and measurement.

### Reliability [3]

- [3] The product works without AI. When the AI is down or slow, the rest of the product should keep working, and AI features should fail clearly (see `layer_05_model_gateway_and_routing.md`).
- [3] Never lose the user's work. Keep the user's message and any partial answer when a stream breaks, reconnect where possible, and save drafts. A failure should never cost users what they typed.

### Observability [3]

- [3] From complaint to trace. A user's report, a support ticket or a thumbs-down should lead straight to the exact trace, with its prompt and model versions (see `layer_10_data.md`).
- [2] What users experienced. Measure speed and errors in the client, as users saw them: time to first visible text, broken streams and rendering failures. Server metrics miss these.

Data quality has little of its own in this layer. The quality of the feedback you collect is covered in `layer_10_data.md`.

## Not in this layer

- How streaming, cancellation and latency work at the API level is covered in `layer_06_inference_and_serving.md`. This file covers what users see.
- What the system does when providers fail, and limits and budgets for each user, are covered in `layer_05_model_gateway_and_routing.md`.
- Producing citations and designing memory are covered in `layer_04_context_memory_and_retrieval.md`. This file covers showing them.
- Which actions need approval is covered in `layer_03_tools_and_protocols.md`, and where people step into agent workflows in `layer_02_agents_and_orchestration.md`. This file covers the screens they use.
- Turning feedback into datasets and evaluation cases is covered in `layer_10_data.md`. This file covers collecting it.
- What models can and can't do, including with speech and images, is covered in `layer_08_foundation_models.md`.
- Visual design, front-end engineering and product management are disciplines of their own. This file covers what changes when AI is involved.

## Learning order

1. Choosing what to build. It decides whether the rest is worth doing.
2. Designing for wrong answers. It shapes most other decisions in this layer.
3. Streaming and waiting.
4. Feedback and measurement. Record what users do from the first release.
5. Interaction patterns.
6. Setting expectations.
7. Users in control of agents, once you ship agents.

## Key reading

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 10 covers AI application architecture and user feedback.
- Amershi et al., "Guidelines for Human-AI Interaction" (Microsoft, 2019).
- Google PAIR, *People + AI Guidebook*.
- Apple, *Human Interface Guidelines*, the section on machine learning.
- Emily Campbell, *The Shape of AI* (shapeof.ai), a library of interface patterns for AI features.
- Nielsen Norman Group's research on AI user experience.
- Parasuraman and Riley, "Humans and Automation: Use, Misuse, Disuse, Abuse" (1997), on trusting automation too much and too little.
- Lisanne Bainbridge, "Ironies of Automation" (1983), on why automation makes human oversight harder.
- Kohavi, Tang and Xu, *Trustworthy Online Controlled Experiments* (Cambridge, 2020).
- OWASP Top 10 for LLM Applications (2025), especially "Improper Output Handling".
- The documentation for a toolkit for AI interfaces, such as the Vercel AI SDK.
