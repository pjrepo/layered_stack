# Layer 3, Tools and protocols: what an application-layer AI engineer needs to know

This file covers the Tools and protocols layer from `ai_stack.md`. That layer is how agents take action: function calling, APIs, and standard protocols such as MCP, which connects models to tools and data, and A2A, which lets agents communicate with each other. It also covers the sandboxes and browsers where agents run code and take actions. This file is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this layer and how deeply to learn each one.

## The rule for depth

A model on its own can only produce text. Tools let it act: look things up, change records, send messages, run code and operate a browser. On every tool call, the model only asks. Your code decides whether to run the call, runs it, and sends back the result. So this layer is where a wrong answer can become a wrong action, such as an email sent, a record deleted or money spent.

For every tool you give an agent, answer four questions:

1. How does the model know when and how to use it? (Function calling, Designing tools)
2. How does it reach the system behind it? (Connecting to APIs, MCP, Agent-to-agent protocols)
3. What can it change, and on whose behalf? (Actions and side effects, Permissions)
4. Where does it run, and what can it reach from there? (Code execution and sandboxes, Browsers and computer use)

Like Layers 4 and 5, this layer is mostly yours. Providers supply the function-calling mechanism and some built-in tools, but you define the tools, connect them to your systems and decide what agents may do. So most concepts in this layer are level 3, and three are level 4:
- designing tools
- deciding which actions need a person's approval
- deciding what each agent may do, and on whose behalf

Learn protocol internals, such as MCP's message format, to level 2, because SDKs handle them. Agent-to-agent protocols are level 2, because most products still run all their agents in one system. Browsers and computer use are level 2 unless you build agents that operate websites or desktops.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Function calling | 3 | The mechanism under every tool, protocol and agent |
| Designing tools | 4 | The model knows your tools only through their names, descriptions and results |
| Connecting to APIs | 3 | Most tools are thin wrappers around an API |
| Model Context Protocol (MCP) | 3 (internals 2) | The standard way to share tools across AI applications |
| Agent-to-agent protocols | 2 | Needed only when agents cross team or company lines |
| Actions and side effects | 3 (some topics 4) | A wrong action often can't be taken back |
| Permissions | 3 (some topics 4) | An agent can be tricked into misusing anything it can reach |
| Code execution and sandboxes | 3 | Code the model writes is untrusted code |
| Browsers and computer use | 2 | Slow and fragile, so only for systems with no API |
| Security | 3 | Tools turn injected instructions into actions |
| Evaluation | 3 | You must check what agents did, not just what they said |
| Reliability | 3 | Every tool call depends on another system that can fail |
| Observability | 3 | You can't debug or audit actions you didn't record |
| Cost and latency | 3 | Each tool call adds a model round trip |
| Governance and safety | 2–3 | When an agent acts, your company acts |

## Function calling [3]

Function calling, also called tool use, is how a model asks your code to do something. How reliably models choose tools and fill in arguments is covered in `layer_08_foundation_models.md`. This section covers the mechanism.

- [3] The loop. You send tool definitions with the request. The model replies with a tool call, made of a tool name and arguments, instead of or alongside text. Your code runs the tool and sends the result back in a tool message, and the model continues. This repeats until the model answers without calling a tool. The model never runs anything itself. Planning across many steps belongs to Layer 2.
- [3] Tool definitions. Each tool has a name, a description and an input schema written in JSON Schema. Most SDKs can generate the schema from a typed function or data model. Definitions are sent on every call and count as input tokens.
- [3] Tool choice. You can let the model decide whether to call a tool, require it to call one, force a specific tool, or turn tools off. Forcing a specific tool is also a common way to get structured output.
- [3] Parallel calls. A model can request several independent calls in one turn. Run them at the same time and return every result, each matched to its call ID. Turn parallel calls off when order matters.
- [3] Checking arguments. Arguments can break the schema, or fit it and still be wrong, such as a valid date in the wrong year. Use strict schema enforcement where the provider offers it (see `layer_06_inference_and_serving.md`), and validate in your code before running anything. Return validation errors to the model so it can fix the call.
- [3] Returning results. Return results as text or structured data, and as images or documents where the API supports them. When a tool fails, return the error as the result, marked as an error, instead of ending the loop. A clear error lets the model try again or try another way.
- [3] Keeping the history valid. Every tool call must be followed by its result, matched by ID, and APIs reject conversations that break this rule. Reasoning models may think between tool calls, and some APIs need that reasoning passed back unchanged with the results. Watch for both when you trim history (see `layer_04_context_memory_and_retrieval.md`).
- [3] Built-in tools. Providers also offer tools that run on their side, such as web search, code execution and file search. They save work, but you control less of how they behave, your data passes through the provider, and they tie you to it (see `layer_08_foundation_models.md`).
- [2] Tool runners. Provider SDKs and agent frameworks can run the loop for you. Build it by hand once, so you know what they're doing when something goes wrong.

Check: Build a tool loop by hand, without a framework, with three tools, one of which fails at random. Show parallel calls, a validation error that the model recovers from, and a trace of every call and result.

## Designing tools [4]

A model knows a tool only through its name, description, parameters and results. Tool design is to agents what interface design is to people, and it can matter as much as the system prompt.

- [4] Descriptions are prompts. Write each description like instructions for a new colleague: what the tool does, when to use it and when not to, what each parameter means, with formats and examples, and what it returns. Small wording changes can shift behavior as much as prompt changes do.
- [4] Tools for tasks, not endpoints. Don't expose every API endpoint as a tool. Build tools around what the agent needs to get done. For example, one tool that finds a free slot and books a meeting works better than separate tools to list calendars, check availability and create events. Fewer, task-level tools mean fewer steps and fewer wrong choices.
- [4] Results the model can use. Return only what the next step needs, not the whole API response. Prefer readable names to opaque IDs, but include the IDs that later calls need. Cap long results, say when you've cut them and how to get more, and consider a parameter that asks for a short or a detailed result.
- [4] Improving tools from transcripts. Run realistic tasks, read the transcripts, and note where the model picks the wrong tool, fills in arguments badly or wastes calls. Revise the tools and measure again on your evaluation set. A model can help: give it the failing transcripts and ask it to suggest better descriptions.
- [3] Helpful errors. Say what went wrong and what to do instead. "No customer named Jon Smith. Did you mean John Smith (ID 4521)?" is far more useful to a model than "404 Not Found".
- [3] Names and parameters. Give tools distinct names, and group related ones with a shared prefix, such as the service name. Avoid tools that overlap. Use few parameters, with clear names, explicit types, fixed lists of allowed values and sensible defaults. Make mistakes hard: if relative file paths cause errors, require full paths.
- [3] Reading apart from changing. Keep tools that only read separate from tools that change things. Then you can give them different permissions and approval rules (see Actions and side effects, and Permissions).

Check: Take one agent's tool set and redesign it: merge low-level endpoints into task-level tools, and rewrite descriptions, errors and results. Show on an evaluation set of tasks that the success rate rises and the tokens per task fall.

## Connecting to APIs [3]

Most tools are thin wrappers around an API: your own backend, a database or another company's service. This section covers that wiring.

- [3] Wrapping an API. The tool translates between what the model sees and what the API needs. It handles pagination, rate limits, retries and error codes, so the model doesn't have to.
- [3] Credentials. Tools need API keys, service accounts or OAuth tokens. Keep them in a secrets manager and add them inside the tool, never in the prompt or anywhere else in the context (see Permissions). Secrets management in general is covered in `layer_05_model_gateway_and_routing.md`.
- [3] OAuth. OAuth lets a user grant your app limited access to their account on another service, such as their calendar or code repository, without sharing their password. Learn the authorization flow, scopes, access and refresh tokens, how to store and refresh tokens, and what happens when a user revokes access. Agents that act for users need it.
- [3] Databases. When a tool queries a database, use read-only credentials where possible, limit rows and query time, and expose views rather than raw tables. Make changes through narrow tools, not free-form SQL. Querying databases for retrieval is covered in `layer_04_context_memory_and_retrieval.md`.
- [2] Generating tools from API descriptions. OpenAPI specifications describe REST APIs, and tools can be generated from them automatically. That's a quick start, but the result is usually one tool per endpoint, which is too low-level for agents (see Designing tools).
- [2] Integration platforms. Services such as Composio, Arcade, Zapier and Pipedream offer ready-made tools for many apps, with OAuth handled for you. They save a lot of work, but their tools may suit your agent poorly, and your users' data and tokens pass through another company.

Check: Connect an agent to a third-party service with OAuth, so each user's agent sees only that user's data. Show that tokens never appear in prompts or logs, and that revoking access stops the agent cleanly.

## Model Context Protocol (MCP) [3, internals 2]

MCP is an open standard for connecting AI applications to tools and data. A service describes its tools once, in an MCP server, and any application that supports MCP can use them. MCP started at Anthropic in late 2024, and most major model providers and agent tools now support it.

- [3] Hosts, clients and servers. The host is the AI application, such as a chat app, a code editor or your own agent. It opens one client connection to each server. Servers expose capabilities to the host.
- [3] What servers offer. Servers offer three kinds of capability:
  - tools: actions the model can call, by far the most used
  - resources: data the application can read and add to the context
  - prompts: ready-made templates a user can pick

  Clients can offer features back to servers, such as asking the user for input (elicitation) or asking the host's model to generate text (sampling). Support for these varies by client.
- [3] Local and remote servers. A local server runs on the user's machine as a separate process and talks to the host over standard input and output (stdio). A remote server runs as a web service over HTTP. Remote servers are how companies offer MCP access to their products.
- [3] Authorization. Remote servers use OAuth, so users can grant a client access to their own accounts (see Connecting to APIs). Build and debug the flow once, end to end.
- [3] Using existing servers. Many companies publish official servers for their products, and registries list thousands more. Before connecting one, check who maintains it, what access it asks for and what it does with your data (see Security below).
- [3] Building a server. Use an official SDK, available for Python, TypeScript and other languages, and test the server with the MCP Inspector. The protocol is the easy part. Most of the work is tool design (see Designing tools).
- [3] When to use MCP. MCP helps when tools must work across many applications, or when you want to use tools that others have built. For tools that live inside your own application, plain function calling is simpler. Some provider APIs can call remote MCP servers directly.
- [2] Protocol details. Messages use JSON-RPC. A session starts with a handshake in which client and server agree on a protocol version and on the features each supports. Servers can tell clients when their list of tools changes. The specification is versioned by date and still changing, so check which versions your clients and servers support.
- [2] Many servers, many tools. Every connected server adds tool definitions to the context, and connecting many servers can crowd the window and confuse the model. Load tools only when needed (see `layer_04_context_memory_and_retrieval.md`), or let the agent write code that calls them (see Code execution and sandboxes).
- [1] Interactive results. Some hosts let tools return small interactive interfaces, such as a form or a chart, instead of only text. How they're shown belongs to Layer 1.

Check: Build an MCP server with two task-level tools for one of your systems, test it with the MCP Inspector, and use it from two different MCP hosts. Deploy it as a remote server with OAuth, and show that each user sees only their own data.

## Agent-to-agent protocols [2]

Sometimes an agent needs to hand work to another agent, built by another team or run by another company. Agent-to-agent protocols standardize this. Coordinating agents inside your own system belongs to Layer 2.

- [2] A2A. The Agent2Agent protocol (A2A) lets agents from different vendors work together. It started at Google in 2025 and is now run by the Linux Foundation. MCP connects an agent to tools. A2A connects an agent to another agent, which does its own reasoning and keeps its internals private.
- [2] Agent cards. Each agent publishes a card that describes what it can do, where to reach it and how to authenticate, so other agents can find and call it.
- [2] Tasks and results. Work is organized as tasks that move through states, such as working, waiting for input, completed and failed. Results come back as artifacts, such as a document or a set of data. Long tasks can stream updates, or send a notification when they finish.
- [2] When you need it. Most products run all their agents in one system, where ordinary function calls or an orchestration framework are simpler (Layer 2). A2A matters when agents cross team or company boundaries.
- [2] An agent as a tool. The simplest way to connect two agents is to expose one as a tool of the other, through function calling or MCP. A2A adds discovery, task tracking and streaming on top.
- [1] Newer protocols. Other protocols cover payments and checkout by agents, such as AP2 and the Agentic Commerce Protocol, and streaming an agent's progress to a user interface, such as AG-UI. Know they exist, and check how widely each is used before building on it.

Check: Explain the difference between MCP and A2A with an example from your product. For one idea that involves several agents, decide whether A2A, MCP or plain function calls fit best, and explain why.

## Actions and side effects [3, some topics 4]

A wrong answer can be corrected. A wrong action, such as a sent email, a deleted record or a payment, often can't. This section covers tools that change things.

- [4] Which actions need approval. Sort every tool by what it can change: nothing; something you can undo; or something you can't undo or that reaches outside your system, such as sending, paying, deleting or posting. Require a person's approval for the last group, show them exactly what will happen, and decide who may approve. Designing handoffs between agents and people belongs to Layer 2, and approval screens to Layer 1.
- [3] Safe to repeat. Networks fail and agents retry, so a tool can run twice. Make changes safe to repeat (idempotent): pass an idempotency key that the API uses to ignore duplicates, check whether the action already happened, or set values rather than add to them. This is what makes retries safe (see `layer_05_model_gateway_and_routing.md`).
- [3] Previews and dry runs. Let risky tools show what they would do before they do it, such as a draft email, a diff or a list of rows to delete. Previews make approvals meaningful and testing safer.
- [3] Undo. Prefer actions that can be reversed: drafts, soft deletes and staging environments. When a multi-step action fails partway, know how to undo the steps already done, or report exactly what changed.
- [3] Limits. Cap what one action or one run can do, such as the number of emails sent, rows changed or money spent. Runaway limits on model calls are covered in `layer_05_model_gateway_and_routing.md`. These limits apply to actions.
- [3] Long-running tools. Give every tool a timeout. For work that takes minutes, have the tool start a job and return its ID, so the agent can check back later instead of waiting.

Check: For one agent, list every tool with what it changes, whether that can be undone, and whether it needs approval. Make one tool safe to call twice, and add an approval step with a preview to one irreversible action. Show both working in a test.

## Permissions [3, some topics 4]

An agent can misuse anything it can reach, because it can be wrong or be tricked by injected instructions. Permissions decide what it can reach. Authentication and authorization in general are covered in `concern_03_security.md`. This section covers them for agents and their tools.

- [4] Least privilege. Give each agent only the tools and access its task needs: read-only where reading is enough, one project rather than the whole account, a narrow tool rather than a general one. Assume the agent will eventually misuse whatever it has.
- [4] Whose authority. Decide for each tool whether the agent acts with the user's own access, delegated through OAuth, or with its own identity and limited permissions. The agent should never be able to do more than the user who asked. Record which identity each action used.
- [3] Enforced in code. Check permissions in the tool or in the API behind it, never in the prompt. An instruction such as "don't delete records" is a request to the model, not a control.
- [3] Credentials out of the context. The model should never see tokens, passwords or keys. Tools add credentials when they call an API. Anything in the context can leak through output or logs, or be stolen by injected instructions.
- [3] Tool sets per agent. Different agents, and different users of the same agent, get different tools. Keep an explicit list of the tools each may use, and review changes to it like code.
- [3] The confused deputy. An agent with broad access of its own can be talked into using it for someone who lacks that access. Checking the requesting user's own permissions on every action prevents this.
- [2] Agent identities. Agents are increasingly given their own identities and credentials, separate from both users and ordinary services, so their actions can be limited, audited and revoked on their own. Standards for this are still settling.

Check: For one agent, write down each tool with the identity it acts as and the narrowest access that works, and remove anything broader. Then show that an injected instruction to act outside the user's permissions is blocked by the tool layer, not merely refused by the model.

## Code execution and sandboxes [3]

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

## Browsers and computer use [2]

Agents can operate websites and desktop apps the way people do. This is the last resort, for systems with no API. Models that operate computers are covered in `layer_08_foundation_models.md`. This section covers the environments they run in. Learn it to level 3 if you build agents that operate websites or desktops.

- [2] Browser automation. Libraries such as Playwright let an agent open pages, click, type and read. The agent can read a page as structure, through its HTML or its accessibility tree (a list of the page's elements), or as screenshots. Structure is faster and cheaper, and screenshots catch what the structure misses.
- [2] Desktop environments. Agents that use a whole computer need a virtual machine with a virtual screen. Run it isolated, with only the apps, files and access the task needs.
- [2] Hosted browsers. Services such as Browserbase run many isolated browsers for agents, with a recording of each session for debugging.
- [2] Logins. Never give the model passwords. Use saved sessions, or a credential store that fills them in outside the model's view. Let a person take over for logins, CAPTCHAs and payments.
- [2] Reliability and cost. Browser agents are slow, expensive and fragile: pages change, load slowly and show pop-ups. Use an API whenever one exists.
- [2] Hostile pages. Any page can contain injected instructions, and a browser agent can act on them. Limit the sites it may visit, and require confirmation before it submits forms, buys or sends anything (see Security below).

Check: Build a browser agent that completes one task on a test site. Over 20 runs, measure its success rate, time and cost, and compare them with an API if one exists. Show that it stops for confirmation before submitting a form.

## Cross-cutting concerns in this layer

### Security [3]

Tools turn prompt injection from a risk of wrong answers into a risk of wrong actions. How models fall for injected instructions is covered in `layer_08_foundation_models.md`, and injection through retrieved content in `layer_04_context_memory_and_retrieval.md`.

- [3] Every tool result is untrusted. Emails, web pages, files, API responses and other agents' replies can all contain instructions aimed at the model. Limit what an agent can do after reading them.
- [3] The dangerous combination. An agent that can reach private data, reads untrusted content and can send data out can be made to leak that data. For any agent that handles sensitive data, remove at least one of the three. Data can leave through any tool argument, such as a URL, an email address or a search query.
- [3] Tool poisoning. A malicious MCP server can hide instructions in its tool descriptions, or change them after you approved it. Connect only servers you trust, pin their versions, and review tool descriptions when they change.
- [3] Supply chain. MCP servers, tool packages and integration platforms are dependencies. Check who publishes them, pin versions and limit what they can reach. A local MCP server runs with the user's own permissions on their machine.
- [2] Designs that resist injection. Some agent designs limit what injected text can do:
  - fix the plan of actions before the agent reads any untrusted content
  - have a separate model with no tools read the untrusted content, and pass on only limited, checked results
  - track which data came from untrusted sources, and block it from sensitive actions

  They reduce the risk, at some cost in flexibility.

### Evaluation [3]

- [3] Checking actions, not just answers. Check whether the agent called the right tools, with correct arguments and without wasted calls, and whether the task succeeded. Where possible, check the end state, such as the record in the database, rather than the agent's final message.
- [3] Mock tools and staging. Test against mock tools or staging systems that return realistic results, including errors, slow responses and empty results. Record real tool responses and replay them for repeatable tests. Building full environments is covered in `layer_10_data.md`.
- [3] Tool changes are prompt changes. Every change to a tool's name, description, parameters or results can change agent behavior. Run the evaluation set before shipping it.
- [3] Safety cases. Include tasks where the right move is to ask for approval, refuse or stop, and tasks with injected instructions in tool results. Check that the agent doesn't take the bait.
- [2] Benchmarks. Public benchmarks such as the Berkeley Function Calling Leaderboard (BFCL) and τ-bench compare how well models use tools. Use them to shortlist models (see `layer_08_foundation_models.md`), not to predict results with your tools.

### Reliability [3]

- [3] Tools fail. APIs behind tools time out, hit rate limits and return errors. Retry temporary errors with backoff inside the tool. When retries run out, return a clear error, so the model can try another way or tell the user. Retries of model calls are covered in `layer_05_model_gateway_and_routing.md`.
- [3] Protecting downstream systems. An agent can call an API far faster than a person. Limit tool calls per user and per run, so a looping agent can't overload a system or use up its limits for everyone else.
- [2] When a tool is down. Decide whether the agent should continue without it, use an alternative, or stop and tell the user.
- [2] Changing tools safely. Renaming a tool or changing its parameters or results can break agents, like any API change. Version tools and roll out changes gradually. Pin the versions of MCP servers you depend on, so they don't change under you.

### Observability [3]

- [3] Tracing tool calls. Record every tool call in the trace: the tool, its arguments, its result or a summary, errors and latency. Link each one to the model call that requested it and to the request's trace ID (see `layer_05_model_gateway_and_routing.md`). OpenTelemetry's conventions for generative AI include standard names for tool calls.
- [3] Audit logs. For every action that changes something, keep a lasting record: which agent, for which user, with which inputs, what changed and who approved it. Traces are for debugging. Audit logs are for accountability, and are kept longer.
- [2] Tool usage. Track which tools are used most, which fail most and which are never used. Unused tools cost tokens and add confusion, and failing ones point to design or reliability problems.
- [2] Redaction. Tool arguments and results often contain personal data and secrets. Remove them before logging.

### Cost and latency [3]

- [3] Round trips. Each tool call adds a model call to read its result, and each of those resends the growing context. A task with ten tool calls in sequence pays for about ten model calls. Task-level tools, parallel calls and code that calls tools all cut the count.
- [3] Tokens. Tool definitions are sent on every call, and long results fill the context (see `layer_04_context_memory_and_retrieval.md`). Keep definitions stable, so they stay in the prompt cache (see `layer_06_inference_and_serving.md`).
- [3] Slow tools. One slow tool stalls the whole agent. Measure each tool's latency, include it in the latency budget (see `layer_06_inference_and_serving.md`), and show users progress while tools run.
- [2] Execution costs. Sandboxes and browsers bill for running time, and many third-party APIs charge per call. Include these in cost per task (see `layer_08_foundation_models.md`), and shut down idle sandboxes and browsers.

### Governance and safety [2–3]

- [3] Human oversight. Write down which actions agents may take alone, which need approval and which they may never take. Enforce it in code (see Actions and side effects, and Permissions).
- [3] Approved tools and servers. Keep a list of the third-party tools and MCP servers your company has approved, with what data each receives and under what terms. Block the rest, as you would unapproved models (see `layer_05_model_gateway_and_routing.md`).
- [3] Accountability. When an agent acts, your company acts. Every action should trace back to the user who asked, the agent that acted and the rules it followed (see Audit logs above).
- [2] Terms and law. Automating a website or API can break its terms of service. Agents that send messages, buy, sign or publish can create legal obligations. Bring in legal before agents act in these ways.
- [2] Regulation. Laws such as the EU AI Act require human oversight for some high-risk uses. Know enough to recognize when your agents fall under them.

Data quality has little of its own in this layer. Tool results feed straight into the agent's next decision, so validate them against a schema, and include timestamps where freshness matters.

## Not in this layer

- How reliably models choose tools and fill in arguments, and models that operate computers, are covered in `layer_08_foundation_models.md`.
- How engines parse tool calls and enforce schemas, and streaming, are covered in `layer_06_inference_and_serving.md`.
- Retries and fallbacks for model calls, runaway limits on model calls, and provider API keys are covered in `layer_05_model_gateway_and_routing.md`.
- Search tools as a source of context, how many tools to load into the context, and injection through retrieved content are covered in `layer_04_context_memory_and_retrieval.md`.
- Planning, multi-step workflows, coordinating agents, handoffs to people and tracking task progress belong to Layer 2 (Agents and orchestration). This file covers the tools and protocols those agents use.
- Approval screens, and how agent actions are shown to users, belong to Layer 1 (Product and UX).
- Building RL environments, and agent trajectories as training data, are covered in `layer_10_data.md`.
- General defenses against prompt injection are covered in `concern_03_security.md`. This file covers the risks that tools add.

## Learning order

1. Function calling. Build a tool loop by hand before you use a framework.
2. Designing tools. Most tool problems are design problems.
3. Connecting to APIs, including OAuth.
4. Actions and side effects, and permissions, before any agent changes anything real.
5. MCP. Use existing servers first, then build one.
6. Code execution and sandboxes, when your agents need to run code.
7. Browsers and computer use, only for systems with no API.
8. Agent-to-agent protocols, when your agents need to work with agents outside your system.

## Key reading

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 6 covers agents, tools and how agents fail.
- Anthropic, "Building effective agents" (2024), especially its appendix on designing tools, and "Writing effective tools for agents" (2025).
- Yang et al., "SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering" (2024), on why tool design matters.
- The Model Context Protocol specification and documentation (modelcontextprotocol.io).
- The A2A protocol specification (a2a-protocol.org).
- Anthropic, "Code execution with MCP" (2025), and Cloudflare, "Code Mode: the better way to use MCP" (2025), on agents that write code to call tools.
- Stripe, "Designing robust and predictable APIs with idempotency" (2017).
- Beurer-Kellner et al., "Design Patterns for Securing LLM Agents against Prompt Injections" (2025), and Debenedetti et al., "Defeating Prompt Injections by Design" (Google DeepMind, 2025).
- Invariant Labs, "MCP Security Notification: Tool Poisoning Attacks" (2025).
- OWASP Top 10 for LLM Applications (2025), especially "Excessive Agency".
- Yao et al., "τ-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains" (2024), and the Berkeley Function Calling Leaderboard.
- Your model provider's tool use documentation, and the documentation for the sandbox and browser tools you use.
