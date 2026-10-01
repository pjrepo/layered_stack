# Layer 2, Agents and orchestration: what an application-layer AI engineer needs to know

This file covers the Agents and orchestration layer from `ai_stack.md`. That layer turns a request into a sequence of steps. It covers planning, multi-step workflows, coordination between agents, handoffs to humans, and tracking where each task stands. This file is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this layer and how deeply to learn each one.

## The rule for depth

This layer decides who chooses the steps. In a workflow, your code fixes the sequence of steps, and the model does the work inside each one. In an agent, the model chooses its own next step, in a loop, until it decides the task is done. Every step adds cost, latency and one more chance to fail, and handing control to the model makes the system harder to predict and test. So the central skill in this layer is giving each feature the least autonomy that does the job, and making whatever autonomy you give visible, recoverable and interruptible.

For every feature, this layer answers four questions:

1. Who chooses the steps: your code or the model? (Workflows and agents, Workflow patterns, Agent loops and planning)
2. Is the work done by one agent or several? (Multi-agent systems)
3. Where does the task's state live, and how does it survive failures and waits? (State and long-running tasks)
4. When does a person step in? (Handoffs to people)

Like Layers 3, 4 and 5, this layer is yours. Frameworks supply building blocks, but only you know which tasks need an agent, how they can go wrong, and where a person must decide. So most concepts in this layer are level 3, and three things are level 4:
- choosing between a workflow and an agent, and how much autonomy each feature gets
- deciding where people step in, and when to step back as an agent proves reliable
- measuring whether agents actually finish their tasks

Learn the range of agent frameworks to level 2 and one framework to level 3, because frameworks change quickly and the patterns underneath them don't. Designs where many agents debate or work as peers are level 2, because they rarely beat simpler designs in production.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Workflows and agents | 4 | How much autonomy a feature gets shapes everything else |
| Workflow patterns | 3 | Most features need a workflow, not an agent |
| Agent loops and planning | 3 | An agent is only as good as its goal, checks and stopping rules |
| Multi-agent systems | 3 | Several agents help some tasks and hurt others |
| State and long-running tasks | 3 | Long tasks meet crashes, deploys and waits for people |
| Handoffs to people | 3 (some topics 4) | People catch what agents get wrong, if they're asked at the right points |
| Frameworks and platforms | 2 (one framework 3) | Frameworks change fast, and the patterns underneath them don't |
| Evaluation | 3 (some topics 4) | An agent that works once may fail the next time |
| Observability | 3 | Agent failures are found by reading runs |
| Cost and latency | 3 | Agents multiply calls, tokens and waiting |
| Reliability | 3 | Every extra step is another chance to fail |
| Security | 3 | Injected instructions can pass from one agent to another |
| Governance and safety | 2–3 | Someone must be able to see and stop what agents do |

## Workflows and agents [4]

This section covers the first decision for any feature: who chooses the steps.

- [4] Workflow or agent. In a workflow, your code decides the steps, and each step is a model call or ordinary code. In an agent, the model decides what to do next, calls tools, reads the results and repeats until it judges the task done. Workflows are cheaper, faster, more predictable and easier to test. Agents handle tasks whose steps can't be known in advance.
- [4] The least autonomy that works. Try a single well-prompted call first, then a fixed workflow, and use an agent only where the steps really can't be predicted. Add autonomy when your evaluation shows the simpler design falls short, not because agents are in fashion.
- [4] When agents fit. Agents do best when the task is open-ended, the agent can check its own progress (for example by running tests), mistakes are cheap to catch and undo, and the task is worth the extra cost. Coding and research are the clearest successes so far.
- [3] Compounding errors. Errors multiply across steps. If each step succeeds 95% of the time and nothing catches mistakes, a 10-step task succeeds only about 60% of the time. Shorten paths, check results between steps, and let agents notice and fix their own errors.
- [3] Mixing both. Most real systems combine the two: a workflow with one step handled by an agent, or an agent that calls fixed workflows as tools. Anything that doesn't need judgment belongs in plain code.
- [3] The harness. An agent is a model plus the code around it: the loop, tools, instructions, context management, permissions and stopping rules. This code is called the harness, and the same model can do much better or worse in different harnesses.

Check: Build one feature three ways: as a single call, a fixed workflow and an agent. Compare quality, cost, latency and consistency on your evaluation set, and write down which you'd ship and why.

## Workflow patterns [3]

Most features need a workflow, not an agent. These patterns cover most workflows, and they combine freely.

- [3] Chaining. Split the task into fixed steps, each working on the previous step's output, such as outline, then draft, then edit. Pass structured data between steps, and add checks between them that stop or repair bad output before it spreads.
- [3] Routing. Classify each request and send it down the path built for that kind of request, with its own prompt, tools and model. Choosing which model answers a call is covered in `layer_05_model_gateway_and_routing.md`. This is choosing which path.
- [3] Parallel steps. Run independent pieces at the same time and combine the results, such as checking a contract against several policies at once, or summarizing each of 500 documents and then combining the summaries. Or run the same step several times and take a vote, for a more reliable judgment.
- [3] Orchestrator and workers. A model breaks the task into pieces that code can't predict in advance, hands each piece to a worker, and combines the results. It's like parallel steps, except the model decides the pieces.
- [3] Generate and critique. One call produces a result, and another checks it against clear criteria and gives feedback. The loop repeats until the result passes or hits a limit. It works when you can say what "good" means, and when feedback actually improves the result.
- [3] Checks as steps. Safety and policy checks can run as steps in a workflow, often in parallel with the main call, and stop the run when they fail. Which checks to run is covered in `concern_03_security.md` and `concern_04_governance_and_safety.md`.
- [2] Batch workflows. Some workflows run offline over large volumes, such as tagging a backlog of support tickets. Run them through queues and batch APIs (see `layer_09_compute.md`), and make them safe to restart partway.

Check: Rebuild one multi-step feature as an explicit workflow with checks between steps. Show on your evaluation set that the checks catch bad intermediate outputs, and report quality, cost and latency for each step.

## Agent loops and planning [3]

The loop itself, in which the model calls tools and reads results, is covered in `layer_03_tools_and_protocols.md`. This section covers making that loop reach its goal.

- [3] A clear goal and finish line. Tell the agent what the task is, what finished looks like, and what it must not do. Without a clear finish line, agents stop early or keep going long after they should.
- [3] Checking its own work. Give the agent ways to test its progress against reality: running tests, comparing results with the data, validating output. Agents that get real feedback at each step do far better than agents that only reason about what they did.
- [3] Planning. For long tasks, have the agent write a plan, such as a to-do list, then update it as it works and re-plan when something unexpected happens. A written plan keeps long tasks on track, and lets a person check the approach before the work starts.
- [3] Stopping rules. Stop the agent when it finishes, when it hits a limit on steps, time or cost, when it stops making progress (such as repeating the same call), or when it needs help. Decide what the user gets in each case. Spending caps are enforced in the gateway (see `layer_05_model_gateway_and_routing.md`).
- [3] Recovering from errors. When a step fails, the agent should read the error, try a different approach, and ask for help after repeated failures instead of looping.
- [3] Tasks longer than one context window. Some tasks outlast any single context window. Design them to be resumed: work in small, verified increments, keep a progress file and a list of what's left, and leave things in a clean state after each increment, so a fresh session can pick up. Context techniques such as compaction are covered in `layer_04_context_memory_and_retrieval.md`.
- [2] Reflection and multiple attempts. An agent can review a failed attempt, write down what went wrong and try again with that note. Running several attempts and keeping the best, judged by tests or a grader, can raise success on hard tasks, at a multiple of the cost.

Check: Build an agent for one task with a written plan, a way to check its own work and stopping rules. Run it 20 times, and report its success rate, how often each stopping rule fired, and what it did when a tool failed.

## Multi-agent systems [3]

A multi-agent system splits a task across several agents, each with its own instructions, tools and context. Keeping each agent's context small is covered in `layer_04_context_memory_and_retrieval.md`, and connecting agents across companies in `layer_03_tools_and_protocols.md`.

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

## State and long-running tasks [3]

A single model call finishes in seconds. A task can run for hours, wait days for a person, and meet crashes and deploys along the way. This section covers tracking where each task stands.

- [3] Task state. Give every task an ID and a status, such as queued, running, waiting for input, waiting for approval, completed, failed or cancelled. Store it in a database with the steps taken and their results, so users, operators and the agent itself can see where each task stands.
- [3] Background tasks. Work that takes more than a few seconds shouldn't hang on one web request. Put it in a queue, run it on workers, show progress, and notify the user when it finishes.
- [3] Durable execution. Save the task's state after each step (a checkpoint), so a task that fails resumes where it stopped instead of starting over. Durable execution engines such as Temporal, and the checkpointing built into frameworks such as LangGraph, do this for you. Resuming can repeat the step that was running, so steps that change things must be safe to repeat (see `layer_03_tools_and_protocols.md`).
- [3] Pausing and resuming. A task that waits for a person may wait for hours. Save its state, release its resources, and resume from exactly that point when the answer arrives.
- [3] Cancelling. Users and operators must be able to stop a task at any point. Stop cleanly: finish or undo the current step, record what was done and tell the user.
- [2] Triggers. Not every task starts with a user's message. Agents can start on a schedule, a new email, a webhook or a change in another system; these are sometimes called ambient agents. No one is watching when they run, so their limits, notifications and handoffs matter more.
- [2] Deploying while tasks run. When you release a new prompt, tool or workflow, tasks already running started under the old one. Decide whether they finish on the old version or switch, and record the version in each task's state.
- [2] Conflicts. Two tasks or agents may change the same record or file at once. Use locks, separate working copies such as separate code branches, or a single owner for each resource.

Check: Run a multi-step task as a background job with a status the user can see. Kill the worker partway through and show that the task resumes from its last checkpoint without repeating completed actions. Then pause it for approval, restart all workers while it waits, approve it and show that it finishes.

## Handoffs to people [3, some topics 4]

People catch what agents get wrong, but only if they're asked at the right points. Which actions need approval is covered in `layer_03_tools_and_protocols.md`, and approval screens belong to Layer 1. This section covers where people fit into the flow of work.

- [4] Where people step in. For each workflow, decide the points where a person acts: approving risky actions, answering the agent's questions, reviewing a plan before work starts, checking the result before it's used, and taking over when the agent can't finish. Each point costs someone's time, so put them where they prevent the most harm.
- [4] Autonomy that grows with evidence. Start a new agent with more review. Measure how often reviewers change or reject what it proposes, and remove review steps for kinds of task where it has proven reliable. Keep review where mistakes are costly.
- [3] Asking instead of guessing. An agent should ask a clarifying question when a request is ambiguous and the answer matters, and otherwise proceed on a sensible assumption and say what it assumed. Too many questions annoy users, so test where the line falls.
- [3] Escalation. Set clear triggers for handing a task to a person: the user asks for one, the agent fails repeatedly or is unsure, or the topic is sensitive. Route each handoff to the right person or team, with a target response time.
- [3] Handing over context. Give the person who takes over a summary of the goal, what's been done and why they're needed, with the full history available. Nobody should have to ask the user to repeat themselves.
- [3] Review fatigue. People asked to approve too often start approving without reading. Keep approvals rare, specific and easy to judge: show exactly what will change and why. Group low-risk items, and track how often reviewers change anything.
- [3] Acting on the decision. The person's decision goes back into the task, and the agent continues from there. Treat an edit as the new plan, and ask for a reason with each rejection, so the agent can use it.
- [2] Reviews as data. Every approval, edit and rejection is a label. Collect them for evaluation and improvement (see `layer_10_data.md`).

Check: For one agent, map every point where a person can step in and what they see there. Add an escalation path that hands over a summary. Over a week, measure how often people approve, edit or reject at each point.

## Frameworks and platforms [2, one framework 3]

Agent frameworks package the patterns in this file. They change quickly, so learn the patterns first and the frameworks second.

- [3] Build by hand first. Build a simple workflow and a simple agent with direct API calls before you adopt a framework. Then you'll know what the framework does for you, and you can debug it. Frameworks add layers that can hide the actual prompts and responses.
- [2] What frameworks offer. Agent frameworks provide the loop, tool handling, state and checkpoints, pausing for people, multi-agent patterns, tracing and ready-made integrations. Widely used ones include LangGraph, the OpenAI Agents SDK, the Claude Agent SDK, Google's Agent Development Kit, CrewAI, Microsoft Agent Framework, Pydantic AI and Mastra.
- [3] One framework in depth. Learn one framework well enough to run it in production: how it stores state, how it pauses and resumes, how it traces, and how to see the raw model calls.
- [2] Graphs. Several frameworks describe a workflow as a graph: steps are nodes, edges decide what runs next, and shared state passes between them. Graphs make the flow of control explicit, including loops.
- [2] Managed platforms. Cloud and model providers offer hosted platforms that run agents for you, with memory, tools, identity and monitoring built in. They save infrastructure work, but tie you to one vendor.
- [2] Visual builders. No-code tools such as n8n let people assemble workflows and agents by connecting boxes. They're quick for simple flows, but harder to test, version and review than code.
- [2] Choosing. Weigh language support, control over prompts and the loop, durability, tracing, lock-in and maturity. Keep your prompts, tools and evaluation sets independent of the framework, so you can switch.

Check: Build the same small agent by hand and with one framework. Show that you can see every raw model call and tool call in both, and pause the framework version for approval and resume it. List what the framework saved you and what it hid.

## Cross-cutting concerns in this layer

### Evaluation [3, some topics 4]

- [4] Measuring task success. Judge an agent by whether it finished the task correctly, checked in the end state, such as the ticket resolved, the tests passing or the record updated, not by its final message. Define success for each task before you build the agent.
- [3] Several runs per task. Agents take different paths on different runs, so run each task several times. Report how often the agent succeeds on every run, not just whether it can succeed once. A user who runs a task ten times needs it to work all ten. τ-bench introduced a measure of this, called pass^k.
- [3] Judging the path. Read trajectories, not just outcomes: wasted steps, wrong tools, risky actions that happened to work out, and right answers reached for the wrong reasons. A model judge can score trajectories against a rubric, once it's checked against human reviewers (see `layer_10_data.md`).
- [3] Testing each step. Evaluate workflow steps and agent roles on their own too, such as the router's accuracy, the planner's plans and each worker's output. End-to-end results show that something failed. Step tests show where.
- [3] Test environments. Agents need somewhere safe to run: mock or sandboxed tools, seeded data and simulated users, reset between runs (see `layer_10_data.md`). Run a set of full tasks, several times each, before every release.
- [3] Cost per success. Report cost, time and steps per successful task alongside the success rate. An agent that succeeds a little more often at five times the cost may not be better.
- [2] Benchmarks. Agent benchmarks such as SWE-bench, τ-bench, GAIA, WebArena, OSWorld and Terminal-Bench each measure a different kind of agentic work. Know what each measures, and don't expect their scores to predict results on your tasks.

### Observability [3]

- [3] Tracing whole runs. Trace every run as a tree: the task, each step, and each model call, tool call and sub-agent, linked parent to child. Tools such as LangSmith, Langfuse, Arize Phoenix and Braintrust display agent traces, and OpenTelemetry's conventions for generative AI include agents.
- [3] Reading runs. Agent traces are long. Use views that summarize a run, showing its plan, its steps, the tools it used and where time and tokens went, and that jump straight to failures. Read full runs regularly, not just metrics. Most agent problems are found this way.
- [3] Monitoring tasks. Track tasks by status: how many are running, waiting for people, stuck or failed, and how long each has waited. A task that has waited three days for approval is a product problem.
- [2] Replaying runs. Save enough to replay a run from any checkpoint with a changed prompt or tool, so you can test a fix on the exact case that failed.

### Cost and latency [3]

- [3] Cost per task. Agents make many calls, each resending a growing context, so budget cost per completed task, not per call (see `layer_08_foundation_models.md`). Multi-agent systems use far more: one published research system used about 15 times the tokens of a chat.
- [3] Budgets the agent knows about. Caps on calls, tokens and cost are enforced in the gateway (see `layer_05_model_gateway_and_routing.md`). Also tell the agent its budget, so it can plan within it and wrap up cleanly instead of being cut off partway through a step.
- [3] A model for each step. Use strong models where judgment matters, such as planning and final review, and cheaper, faster models for routing, extraction and routine steps (see `layer_08_foundation_models.md`).
- [3] Latency. Each step in sequence adds its full latency, and slow steps compound (see `layer_06_inference_and_serving.md`). Run independent steps and sub-agents in parallel, stream progress to users, and move long tasks to the background.

### Reliability [3]

- [3] Failing well. When an agent can't finish, it should stop with a clear status, keep partial results and say what's left. Agents sometimes report success they didn't achieve, so confirm completion in code where you can.
- [3] Timeouts for whole tasks. Steps have timeouts (see `layer_03_tools_and_protocols.md`), and so should whole tasks. A stuck task should fail or escalate, not run forever.
- [3] Falling back. When an agent fails, fall back to a simpler workflow, a smaller version of the task, or a person. Fallbacks between models are covered in `layer_05_model_gateway_and_routing.md`.

Resuming after crashes is covered under State and long-running tasks.

### Security [3]

- [3] Trust between agents. Treat another agent's output as data, not instructions, especially if that agent read untrusted content. An injected instruction can travel from a sub-agent that read a web page to a lead agent that can act.
- [3] Separating reading from acting. Keep agents that read untrusted content away from powerful tools, and give agents that act only trusted inputs. Orchestration is where you can build this separation. Designs that resist injection are covered in `layer_03_tools_and_protocols.md`.
- [3] Permissions for each agent. Give each agent in a system the fewest tools and permissions its role needs (see `layer_03_tools_and_protocols.md`). A sub-agent shouldn't inherit everything the lead agent can do.
- [2] Goal drift. Over long runs, injected text or a build-up of context can pull an agent away from its task. Keep the original goal in view, and check actions against it.

### Governance and safety [2–3]

- [3] Stopping agents. Be able to pause or stop one task, all tasks of one agent, or every agent at once, and test that it works.
- [3] Records of decisions. Keep each task's plan, steps, approvals and results, so you can explain afterwards what the agent did, why, and who approved it. Audit logs of individual actions are covered in `layer_03_tools_and_protocols.md`.
- [2] Telling users. Make it clear when users are dealing with an agent rather than a person, especially when it acts or writes on their behalf. Laws such as the EU AI Act require this in some cases.
- [2] Unattended agents. Agents that start on triggers, with no user watching, need tighter limits and regular review of what they did.

Data quality has little of its own in this layer. Each step's output is the next step's input, so validate it at every boundary, as covered under Workflow patterns. Agent trajectories as training data are covered in `layer_10_data.md`.

## Not in this layer

- How the tool loop works, tool design, which actions need approval, and agent-to-agent protocols are covered in `layer_03_tools_and_protocols.md`. This file covers how steps and agents are organized.
- What each agent sees and remembers, including sub-agents as a way to keep context small, is covered in `layer_04_context_memory_and_retrieval.md`.
- Choosing which model answers each call, fallbacks between models and spending caps are covered in `layer_05_model_gateway_and_routing.md`.
- How well models handle long, multi-step tasks is covered in `layer_08_foundation_models.md`.
- Agent environments, simulated users and agent trajectories are covered in `layer_10_data.md`, and training agents with reinforcement learning in `layer_07_model_adaptation.md`.
- How plans, progress and approvals are shown to users belongs to Layer 1 (Product and UX).

## Learning order

1. Workflows and agents. Most of the value is in knowing when not to build an agent.
2. Workflow patterns. Most features need a workflow, not an agent.
3. Agent loops and planning, building one agent by hand.
4. Evaluation, as described in the cross-cutting section, before you rely on any agent.
5. State and long-running tasks.
6. Handoffs to people.
7. Multi-agent systems, once a single agent falls short.
8. Frameworks and platforms, after you've built the basics by hand.

## Key reading

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
