# Layer 4, Context, memory and retrieval: what an application-layer AI engineer needs to know

This file covers the Context, memory and retrieval layer from `ai_stack.md`. That layer decides what information the model sees on each call. It includes retrieval-augmented generation (RAG), vector and keyword search, knowledge graphs, and long-term memory that lasts across sessions. This file also covers writing and managing prompts, because the prompt is part of what the model sees. It is for an AI engineer who builds products on existing models (layers 1 to 5). It lists the concepts in this layer and how deeply to learn each one.

## The rule for depth

A model works from two things: what it learned in training, and what's in its context on the current call. You can change the first only by training (see `layer_07_model_adaptation.md`). You control the second completely, on every call. So when an answer is wrong, the first question is whether the context held what the model needed.

Everything the model sees comes from four sources:

1. Instructions you write: the system prompt, examples and tool definitions. (Prompts)
2. The work so far: the conversation, tool results and notes from earlier steps. (Conversation history)
3. Knowledge you look up: documents, records and search results. (Retrieval-augmented generation to Knowledge graphs)
4. What you saved from earlier sessions: facts about users and past work. (Long-term memory)

Context engineering decides how much of each goes in, and in what order, within one context window. Retrieval only reads. Memory also writes, deciding what to store, update and forget, so its mistakes outlast the call that made them.

Like the gateway in Layer 5, this layer is yours. No provider knows what your model should see, so no one else will build it for you. That makes it the deepest layer in this curriculum for an application engineer. Most concepts are level 3, and four things are level 4:
- context engineering: deciding what goes into each call
- writing clear instructions
- managing prompts as code
- finding which step caused a bad answer

Learn how search works inside, such as index algorithms, only to level 2, because search engines and vector databases handle it. Knowledge graphs are level 2 unless your questions depend on relationships across many records.

## Levels

1. Aware. You can say what it is and why it matters to your product. You recognize it when it comes up in a model card, a paper or a vendor pitch.
2. Conversant. You understand how it works and what it trades off. That is enough to make decisions, to judge vendor and research claims, and to tell when a problem comes from it.
3. Practitioner. You do it yourself in production and know the common ways it goes wrong.
4. Expert. You design the process, set your team's standards and teach others.

Every concept below has its level in brackets. Each section ends with a check. Once you can do what the check describes, you have reached that section's level.

## Summary

| Area | Level | Why this level |
|---|---|---|
| Context engineering | 4 | What the model sees decides most of what it does |
| Prompts | 3 (some topics 4) | You write and change them more than anything else |
| Conversation history | 3 | Long conversations and agent runs outgrow the window |
| Retrieval-augmented generation | 3 | The main way to give a model knowledge it lacks |
| Preparing documents | 3 | Search can't find what parsing and chunking lost |
| Search | 3 (internals 2) | Keyword and vector search each miss what the other finds |
| Query handling and reranking | 3 | Often the biggest gains once a basic pipeline works |
| Answering from retrieved context | 3 | Retrieval helps only if answers stick to the sources |
| Agentic retrieval | 3 | Agents increasingly search for themselves, in several steps |
| Knowledge graphs | 2 | Worth the cost only when questions follow relationships |
| Long-term memory | 3 | Memory writes, so its mistakes last |
| Evaluation | 3 (some topics 4) | You can't fix a bad answer until you know which step failed |
| Security | 3 | Every retrieved document and memory is untrusted input |
| Data quality | 3 | Stale or wrong documents produce confident wrong answers |
| Governance and safety | 3 | Indexes and memories copy personal data that must stay deletable |
| Observability | 3 | You can't debug an answer without the context it saw |
| Cost and latency | 3 | Context size drives the cost of most calls |
| Reliability | 2–3 | Search is one more service that can fail |

## Context engineering [4]

Context engineering means choosing what goes into the model's context on each call, and in what order. How models use long inputs is covered in `layer_08_foundation_models.md`. This section covers what to put in them.

- [4] Check the context first. When an answer is wrong, look at the context before blaming the model. Was the needed information there? Was it buried, cut off, contradicted or out of date? Many failures that look like model failures are context failures. You can only check if you log the full context of each call (see Observability below).
- [4] The smallest context that works. More context isn't better. Irrelevant material distracts the model, quality tends to fall as the context grows, and every extra token adds cost and latency. Aim for the smallest set of information that lets the model do the job well.
- [4] Long context or retrieval. If everything relevant fits comfortably in the window, such as one contract or a small handbook, put it all in. It's simpler, it avoids retrieval errors, and prompt caching makes repeated use cheaper. Loading a whole knowledge base into a cached prompt is sometimes called cache-augmented generation. Retrieve when the material is too large, changes often or depends on user permissions, or when cost and latency per call matter. Many products retrieve a few whole documents rather than many small fragments. Test the options on your evaluation set.
- [4] A budget for each part. The system prompt, tools, examples, documents, history, memories and the user's request all share one window, and the output needs room too. For each feature, decide how many tokens each part may use and what gets cut first when a call runs over, such as the oldest history or the lowest-ranked documents. Count tokens before sending.
- [3] How context goes wrong. Four common patterns:
  - A mistake or made-up fact enters the context, and the model keeps building on it (poisoning).
  - So much material piles up that the model loses track of what matters (distraction).
  - Irrelevant documents or tools lead the model astray (confusion).
  - Parts of the context contradict each other, such as an old and a new version of a policy (clash).
- [3] Order and structure. Put stable content first and changing content last, even small things like the current date, so prompt caching keeps working (see `layer_06_inference_and_serving.md`). With long documents, put the question after them. Provider guides differ on the details, so test the order on your model. Separate instructions, documents and user input with clear markers, such as XML tags or Markdown headings, and label each document with its source and date.
- [3] Tools in the context. Every tool definition takes tokens, and dozens of similar tools make wrong choices more likely. Load only the tools a task needs, or let the model search for the tool it needs. Designing tools belongs to Layer 3.
- [3] Fetching context when needed. Instead of loading everything up front, let the model fetch information as it goes: through a search tool, a file it can open, or detailed instructions it loads only for the task at hand. This keeps the context small, at the cost of extra steps.

Check: Log 50 real calls for one feature and break each context into its parts in tokens: instructions, tools, history, documents and memories. Remove or shrink the part that adds least, and show on your evaluation set that quality holds while cost falls. Then sort 20 failures into missing, buried or conflicting context, or model errors.

## Prompts [3, some topics 4]

This section covers writing and managing the instructions you give the model. How models respond to instructions and examples is covered in `layer_08_foundation_models.md`.

- [4] Clear instructions. Write the system prompt as a brief for a capable new colleague who knows nothing about your product. Say what the task is, who the users are, what a good result looks like, and what to do when information is missing or a request is out of scope. Explain the reason behind each rule, because models apply rules better when they know why. Recent models follow instructions literally, so say exactly what you want.
- [3] Structure. Break long prompts into labeled sections, such as task, rules, output format and examples. Keep instructions apart from the data they act on. Say what to do, not only what to avoid. A role such as "You are a support agent for Acme" sets tone, but specific instructions matter more.
- [3] Examples. Choose a few varied, realistic examples that include hard cases, since models copy their format, length and mistakes. For tasks with many kinds of input, keep a bank of good examples and retrieve the most similar ones for each request (dynamic few-shot prompting).
- [3] Output instructions. Say what format and length you want. Use structured output when code will read the result (see `layer_08_foundation_models.md`). When quality matters, let the model reason before its final answer.
- [3] Reasoning models. Reasoning models need less guidance on how to think. Give them the goal, the constraints and what a good result looks like. Detailed step-by-step instructions can do worse than a high-level request, so test both.
- [3] Templates. Code assembles most prompts from a template, filling in user data, documents and memories. Test templates with real values, including empty, very long and oddly formatted ones, because a broken variable fails without an error. Mark inserted content clearly, so the model can tell it apart from your instructions.
- [4] Prompts as code. Keep prompts in version control or a prompt registry, not scattered through the code or edited live in a dashboard. Give each version an ID and log it with every call. Review prompt changes like code, and run the evaluation set before any change ships. Roll versions out and back by configuration (see `layer_05_model_gateway_and_routing.md`).
- [3] Maintaining prompts. Improve prompts from error analysis (see `layer_10_data.md`): read failures, change one thing, re-run the evaluation set. A special-case rule for every failure makes prompts long and contradictory, so rewrite them from scratch now and then. Prompts tuned for one model often do worse on another, so re-test them when you switch.
- [2] Automatic prompt optimization. Tools such as DSPy search for better instructions and examples against a metric on your evaluation set. The results are only as good as the metric, and they can overfit the evaluation set.

Check: Put one production prompt under version control, with its version ID logged on every call. Rewrite it from error analysis, and show on your evaluation set that the new version beats the old one. Roll it out and back by changing only configuration.

## Conversation history [3]

Most model APIs are stateless, so your application resends the conversation on every call (see `layer_08_foundation_models.md`). This section covers keeping that history useful as it grows, within one session or agent task. Memory across sessions is covered under Long-term memory.

- [3] Storing conversations. Keep each conversation in your own database, with messages, tool calls and results in order. Some providers can store conversation state and continue from an ID. That saves work, but ties you to the provider and its data retention rules.
- [3] Why history gets expensive. Every call resends all earlier turns, so each turn costs more than the last, and old material crowds the window. Agent runs grow fastest, because tool results are often long.
- [3] Trimming history. The main ways are:
  - keep only the most recent turns
  - summarize older turns into a running summary
  - shorten or drop old tool results once the model has used them
  - compact: near the limit, replace the history with a summary and carry on

  Some providers and agent frameworks do this for you. Whichever you use, test what gets lost.
- [3] What summaries lose. Summaries drop exact numbers, names, decisions and constraints the user set earlier. Keep what must survive, such as the user's goal and constraints, in a separate block that's never summarized.
- [3] Notes outside the context. Long agent tasks keep notes outside the context, such as a to-do list or a progress file, and read them back when needed. This lets a task continue across many context windows, even after the history has been compacted.
- [3] History and caching. Adding messages to the end keeps the prompt cache working. Editing or removing earlier messages breaks it from that point on (see `layer_06_inference_and_serving.md`). So trim in occasional large steps, not a little on every call.
- [2] Sub-agents. A sub-agent can do a search-heavy piece of work in its own context and return only a short result, which keeps the main context clean. Coordinating sub-agents belongs to Layer 2.

Check: Run a 50-turn conversation or a long agent task through your product, and chart tokens and cost per call. Add a trimming or compaction strategy. Show that cost levels off while questions about facts from early turns are still answered correctly.

## Retrieval-augmented generation [3]

Retrieval-augmented generation (RAG) finds relevant information and adds it to the context before the model answers. It's the main way to give a model knowledge it lacks: private data, recent events and anything after its knowledge cutoff (see `layer_10_data.md`). This section covers the whole pipeline, and the sections after it cover the parts.

- [3] When to retrieve. Retrieve when the model needs facts that are private, recent, too large for the context, or must be cited. Fine-tuning is unreliable for teaching facts (see `layer_10_data.md`), and retrieval stays current without retraining. Whether to retrieve or load everything is covered under Context engineering.
- [3] The two halves. Offline, you prepare knowledge: collect documents, parse them, split them into chunks, then embed and index the chunks. Online, for each request, you turn it into search queries, search, rerank and filter, and put the best results in the context. A failure at any step shows up as a bad answer at the end.
- [3] Start simple. Build the simplest complete pipeline first, with an evaluation set. Then improve whichever step the evaluation shows is weakest. Most gains come from better parsing, chunking, hybrid search and reranking, not from elaborate techniques.
- [3] Frameworks and managed services. Libraries such as LlamaIndex and LangChain, and managed services such as the file search built into provider APIs or Amazon Bedrock Knowledge Bases, handle much of the pipeline. They're a quick start, but their default parsing and chunking often suit your documents poorly. Know what each step does, so you can inspect and change it.
- [2] Named techniques. Papers and vendors use many names, such as HyDE, RAG-Fusion, Self-RAG, corrective RAG and GraphRAG. Most combine the steps covered below. Know them well enough to tell what a claim actually changes.

Check: Build a complete RAG pipeline over at least 1,000 of your own documents, with an evaluation set of 50 real questions. Measure retrieval and answer quality separately, and name the step that most needs improving.

## Preparing documents [3]

Search can only find what this step kept. It runs offline, before anyone asks a question, and many retrieval failures start here.

- [3] Sources and connectors. Knowledge lives in files, wikis, ticket systems, databases, email and chat. Pull it in with connectors, and keep each document's ID, source, owner, dates and access permissions.
- [3] Parsing. Turn each document into clean text that keeps its structure: headings, lists, tables and reading order. PDFs are hardest, especially scans, multi-column layouts and tables. Tools range from plain text extraction to layout-aware parsers such as Docling or Unstructured, optical character recognition (OCR) and vision models that read page images. Bad parsing fails silently, so inspect the output from real documents by eye. Sending whole documents to a multimodal model is covered in `layer_08_foundation_models.md`.
- [3] Chunking. Split documents into pieces small enough to retrieve precisely but large enough to make sense alone. The main approaches are:
  - fixed size in tokens, with some overlap between neighboring chunks
  - by structure, such as sections, paragraphs, or functions in code
  - by meaning, starting a new chunk where the topic changes

  Splitting by structure is usually the best start. Test chunk sizes on your evaluation set rather than copying a default.
- [3] Context for each chunk. A chunk cut from its document can lose what it refers to, such as which product "it" means. Add the document title and section heading to each chunk. Or have a model write a sentence placing the chunk in its document, and index that with the chunk (contextual retrieval).
- [3] Metadata. Store fields such as source, date, author, product, language and permissions with each chunk. You'll use them to filter searches, favor newer documents, cite sources and enforce permissions.
- [3] Search small, read big. Search over small chunks for precision, then give the model the surrounding section or the whole document (parent-document retrieval).
- [3] Keeping the index in sync. Documents are added, changed and deleted. Update the index incrementally, remove deleted documents promptly, and record when each document was last indexed. A deleted document that search still finds is both a quality bug and a compliance problem.
- [2] Structured data. Tables and databases are usually better queried than embedded. Let the model write SQL or call an API for them (text-to-SQL). A generated query can run without error and still answer the wrong question, so test it like any other retrieval. Learn this to level 3 if your knowledge lives mainly in databases.
- [2] Images and diagrams. When figures carry the meaning, describe them with a vision model and index the descriptions, or embed page images directly with a multimodal embedding model (see `layer_08_foundation_models.md`).

Check: Parse and chunk 100 of your hardest real documents in two different ways. Inspect a sample of chunks by eye, then compare retrieval on your evaluation set. Show that deleting a source document removes it from search results.

## Search [3, internals 2]

Search finds the chunks most likely to help with a request. What embedding models are and how to choose one is covered in `layer_08_foundation_models.md`. This section covers building search with them.

- [3] Keyword search. Keyword search matches the words in the query, usually scored with BM25. That formula gives rare words more weight, and gives each extra repeat of a word less credit. It's fast and cheap, and best at exact terms such as names, product codes, error messages and IDs. It misses synonyms and rephrasings.
- [3] Vector search. Vector search embeds the query and finds the chunks with the most similar vectors. It matches meaning, so it finds rephrasings, but it handles exact terms and negation poorly (see `layer_08_foundation_models.md`).
- [3] Hybrid search. Run both searches and merge the results, often with reciprocal rank fusion, which combines the two rankings without needing comparable scores. Hybrid search is the usual default, because each method finds what the other misses.
- [3] Filters. Narrow results by metadata, such as date, product or customer. If a database filters only after the vector search, a strict filter can leave too few results, so check how yours does it.
- [3] Where to store vectors. The main options are:
  - a vector extension to a database you already run, such as pgvector for PostgreSQL
  - a search engine with vector support, such as Elasticsearch or OpenSearch
  - a dedicated vector database, such as Pinecone, Qdrant, Weaviate or Milvus
  - an embedded library, such as Chroma or LanceDB, for prototypes and small collections

  Choose on scale, filtering, hybrid search, separating customers' data, operations and cost. For many products, a database you already run is enough.
- [2] Approximate search. Comparing a query with every vector is exact but slow at scale. Vector databases use approximate nearest-neighbor (ANN) indexes, such as HNSW or IVF, that find nearly all the closest vectors far faster, and some compress vectors too. Their settings trade recall for speed and memory, so measure recall at the settings you use.
- [2] Search you already have. Many companies already run a search engine over their content, and web search APIs cover public information. Calling an existing service can be quicker than building an index, but you inherit its ranking and its gaps.

Check: For 50 real queries with known correct documents, compare keyword, vector and hybrid search. Report recall at 10 for each (see Evaluation below), and find one kind of query that each method misses.

## Query handling and reranking [3]

User messages are rarely good search queries, and first-pass results are rarely in the best order. These steps fix both.

- [3] Rewriting queries. Have a model turn the user's message into one or more search queries. In a conversation, rewrite follow-ups so they stand alone: "what about the second one?" means nothing to a search engine. Pull out filters too, such as dates or product names.
- [3] Several queries. Break a complex question into sub-questions and search for each, or search with several phrasings and merge the results. This finds more, at the cost of extra calls and latency.
- [3] Whether and where to search. Not every message needs a search: a greeting or a question about the last answer doesn't. With several sources, such as documentation, tickets and a database, decide which to search. Rules, a small classifier or the model itself can decide.
- [3] Reranking. Retrieve a generous candidate list, such as the top 50 to 100 chunks, then rerank it and keep the best few. Rerankers are described in `layer_08_foundation_models.md`. You can use a hosted reranker, an open-weight one, or a model prompted to score relevance, which is slower and costs more. Reranking is often one of the largest improvements to a basic pipeline.
- [3] How many results. Choose how many chunks to pass to the model, or a reranker score to cut off at, by testing on your evaluation set. Too few misses information. Too many adds noise and cost.
- [2] Searching with a draft answer. Have a model write a plausible answer and search with it, since answers look more like documents than questions do (HyDE). It helps some searches and hurts others, so test it.
- [2] Diversity and other signals. Top results are often near-copies, so remove duplicates or use maximal marginal relevance (MMR), which favors results that add something new. You can also weigh freshness or source authority, for example ranking the current version of a policy above older ones.

Check: Add query rewriting and then a reranker to a basic pipeline. After each change, measure recall, answer quality and added latency on your evaluation set. Keep only the changes that pay for themselves.

## Answering from retrieved context [3]

Retrieval helps only if the model uses what was retrieved and sticks to it. This section covers the answering step of RAG.

- [3] Grounding instructions. Tell the model to answer from the documents provided and to say when they don't contain the answer. Decide for each feature whether the model may add general knowledge: some features should refuse without a source, others can fill small gaps.
- [3] Citations. Give each document an ID, and ask the model to cite the source of each claim. Check that each citation exists and supports its claim. Some providers offer built-in citations that return the exact passages used. Show citations so users can check answers themselves.
- [3] Presenting documents. Give each chunk its title, source, date and ID in a consistent format. Put the most relevant material where the model is least likely to miss it, since the middle of a long context is easiest to miss (see `layer_08_foundation_models.md`).
- [3] Conflicting and outdated sources. Retrieved documents can disagree, such as an old and a new version of a policy. Filter out superseded documents where you can, include dates, and tell the model how to resolve conflicts, such as trusting the newest source or reporting the disagreement.
- [3] When nothing relevant is found. The model should say so, ask a clarifying question or search again, rather than answer from weak results. Weak results can be worse than none, because models tend to use whatever they're given.
- [2] Checking answers. A second model call can check that each claim is supported by the retrieved text before the answer is shown. It catches some unsupported claims, at extra cost and latency.

Check: Take 50 answers from a RAG feature and check each claim against its cited source. Report how many claims are unsupported and how many citations are wrong. Make one change that lowers both, and confirm it on your evaluation set.

## Agentic retrieval [3]

In a fixed pipeline, your code searches once and the model answers. In agentic retrieval, the model decides when to search, what to search for and when it has enough, calling search tools as often as it needs. As models have got better at using tools, this has become common, especially in coding agents and research tools. How tool calling works belongs to Layer 3. This section covers search as an agent uses it.

- [3] Search as a tool. Give the agent search tools with clear descriptions and useful parameters, such as filters, that return short results labeled with their sources. The agent can then search, read, refine its query and search again.
- [3] Pipeline or agent. A fixed pipeline is faster, cheaper and more predictable. An agent handles vague or multi-part questions better, and can recover from a bad first search. Use a pipeline for simple lookups where speed matters, and an agent where questions need several steps. Many products route each request to one or the other.
- [3] Searching without an index. Agents can search the way people do: list files, search for exact strings with a tool such as grep, open files and follow references. Coding agents often work this way, with no embedding index at all. It works well on content with clear structure and names, such as code, and there's no index to keep in sync.
- [3] Web search. Agents can search the web through a provider's built-in tool or a search API, then fetch and read pages. Web results can be wrong, outdated or hostile, like any retrieved text (see Security below).
- [3] Knowing when to stop. Agents can stop early on thin evidence, or keep searching long after they stop learning anything. Tell the agent what "enough" looks like, and cap its searches and tokens.
- [2] Deep research. Research agents run many searches over several minutes, read dozens of sources and write a cited report. Know how they work and what they cost, as a pattern for long questions that need many sources.
- [2] Connecting data sources. Protocols such as MCP expose many data sources to an agent as tools. Which sources to connect, and how much of their output enters the context, belong to this layer. The protocol itself belongs to Layer 3.

Check: For 30 multi-step questions from your product, compare a fixed RAG pipeline with an agent that has the same search as a tool. Report answer quality, number of searches, latency and cost for each, and decide which kinds of question should go to which.

## Knowledge graphs [2]

A knowledge graph stores entities, such as people, products and companies, and the relationships between them. It helps with questions that follow relationships across many records, which chunk search handles poorly.

- [2] What they're good for. A question such as "which customers use a product that depends on this part?" follows a chain of connections across many records. Chunk search finds passages that mention the words, not the chain. Graphs also help with questions about a whole collection, such as the main themes across a year of reports.
- [2] Building one. You can build a graph from structured data you already have, or have a model extract entities and relationships from documents. Model extraction is costly and imperfect: it misses relationships, and the same entity turns up under different names. Merging those duplicates (entity resolution) is much of the work.
- [2] Querying one. Graph databases such as Neo4j are queried with languages such as Cypher. A model can write the query, or your code can fetch an entity and its neighbors and add them to the context as text.
- [2] GraphRAG. Microsoft's GraphRAG and similar methods build a graph from documents, group related entities into communities and summarize each one. They answer broad questions about a whole collection from those summaries, which chunk search can't do, but indexing costs far more.
- [2] When to use one. Start with hybrid search and metadata filters. Add a graph only when your evaluation shows that relationship questions fail and matter. Good metadata and a relational database you already have often give most of the benefit.

Check: Find a kind of question in your product that depends on relationships between records. Show whether hybrid search answers it, and estimate what building and maintaining a graph for it would cost.

## Long-term memory [3]

Long-term memory lets a product remember users and past work across sessions. Unlike retrieval, memory writes: it decides what to store, update and forget. So a wrong memory keeps causing wrong answers until something corrects it.

- [3] Kinds of memory. Each kind is stored and used differently:
  - facts, such as the user's role, preferences and current projects (semantic memory)
  - past events, such as earlier conversations and what was decided in them (episodic memory)
  - ways of working, such as how this user likes a task done, or notes an agent writes for itself (procedural memory)
- [3] Writing memories. A model can write memories in the background after a session, the agent can write them during the conversation with a memory tool, or the user can ask ("remember this"). Save facts that will matter later, not everything that was said. Store each memory with its source and date.
- [3] Updating and forgetting. New information can contradict old memories, for example when a user changes jobs. Update or replace the old memory instead of keeping both. Merge duplicates, and let memories expire when they go stale. Forgetting is part of the design, not a failure.
- [3] Reading memories. Retrieve the memories relevant to the current request, or load a short profile into every call. Too many memories distract the model, and bringing up unrelated personal details makes users uneasy.
- [3] Storage. Memories can live in a vector store, a database table, a knowledge graph that records when each fact was true, or plain files that an agent reads and edits. Records and files are easier to inspect, correct and delete than embeddings alone. Services and frameworks such as Mem0, Zep and Letta manage memory for you. Some providers offer a memory tool, where the model decides what to save and your application stores it.
- [3] Scope. Decide whether each memory belongs to a user, a team, a project or the whole organization. Keep scopes separate, so one user's memories never reach another user's context.
- [3] Memory for agents. Agents that return to the same project can keep notes: what they learned about a codebase, which approaches failed and the user's conventions. Many coding agents read a project instructions file, such as AGENTS.md or CLAUDE.md, at the start of every session, and both people and agents edit it.
- [3] User control. Let users see what the product remembers about them, correct it, delete it and turn memory off. Show when a memory is used. This is also a privacy requirement (see Governance and safety below).
- [3] How memory goes wrong. A memory can be wrong from the start because the model misunderstood. It can go stale, or be applied in the wrong place, such as a preference from one project applied to another. An attacker can also plant one (see Security below). Test memory with scripted scenarios that span several sessions.

Check: Build memory for one feature, covering writing, updating, retrieval and deletion. Run a scripted scenario across five sessions in which the user's preferences change. Show that the product uses the new preference, not the old one, and that a deleted memory never comes back.

## Cross-cutting concerns in this layer

### Evaluation [3, some topics 4]

Building retrieval test sets is covered in `layer_10_data.md`. This part covers what to measure.

- [4] Finding which step failed. A bad answer can come from retrieval (the right information wasn't found), from context assembly (it was found but cut, buried or contradicted), or from generation (it was in the context, but the model ignored or misused it). Measure each step separately, and label each failure by step during error analysis. Otherwise you'll tune the wrong part.
- [3] Retrieval metrics. For queries whose relevant chunks you know, measure:
  - recall at k: the share of relevant chunks that appear in the top k results
  - precision at k: the share of the top k results that are relevant
  - ranking scores, such as mean reciprocal rank (MRR) and normalized discounted cumulative gain (nDCG), which reward putting the best results first

  Recall matters most for the candidate list before reranking. Precision and ranking matter most for what reaches the model.
- [3] Answer metrics. Faithfulness: is every claim supported by the retrieved context? Relevance: does the answer address the question? Correctness: does it match the reference answer, where there is one? Also check that citations are accurate. Model judges can score these, but calibrate them against human labels first (see `layer_10_data.md`). Libraries such as Ragas implement common versions.
- [3] Testing memory. Memory needs tests that span sessions. Check that the product recalls what it should, updates what changed, forgets what was deleted, and doesn't use memories where they don't apply.
- [3] Testing every change. Changes to prompts, chunking, embedding models, retrieval settings and memory rules all change answer quality. Run the evaluation set before each one ships, as you would for a model change.
- [2] Public benchmarks. Retrieval benchmarks such as BEIR, memory benchmarks such as LongMemEval, and long-context tests such as needle in a haystack rarely predict results on your data. Use them to shortlist, not to decide.

### Security [3]

- [3] Indirect prompt injection. Every retrieved document, web page, email, tool result and memory is untrusted text that can contain instructions aimed at the model (see `layer_08_foundation_models.md`). Retrieval widens the risk: anyone who can get text into your index can try to steer your model. Mark retrieved content as data, limit what an agent that reads untrusted content can do, and never let retrieved text grant permissions. General defenses are covered in `concern_03_security.md`.
- [3] Permission-aware retrieval. Users should retrieve only what they may see in the source system. Store permissions with each chunk, filter every search by the user's identity inside the search itself, and keep permissions in sync as they change. Asking the model to hold back something it has already been shown doesn't work.
- [3] Keeping customers apart. When you serve many customers, keep each one's data in a separate index, namespace or partition, or filter every query by customer. One missing filter can leak one customer's documents to another.
- [3] Memory poisoning. Anyone who can get text into a conversation or document can try to plant false or malicious memories that affect later sessions. Review what gets written to memory, and keep memories scoped to the user they came from.
- [2] Leaking data through output. Injected instructions can make the model leak retrieved data, for example inside a link or image address that your product then loads. Block or check links and images in model output. The risk is highest when one agent has private data, reads untrusted content and can send data out.
- [2] Embeddings aren't anonymous. Text can be partly reconstructed from its embedding, so protect vector stores as carefully as the documents they came from.

### Data quality [3]

In this layer, data quality means the quality of what you retrieve and remember.

- [3] Freshness. Stale documents produce confident, wrong answers. Track when each source was last updated and last indexed, and refresh each source as often as it changes.
- [3] Duplicates and versions. Near-duplicates crowd out other results, and old versions contradict new ones. Remove duplicates during ingestion, and mark or remove superseded versions.
- [3] Source quality. Some sources are authoritative. Others are drafts, opinions or simply wrong. Rank or filter by source, and exclude sources that answers shouldn't come from.
- [3] Lineage. For each chunk, record the source document and version, the parser, the chunking settings and the embedding model. Then you can trace a bad answer to its origin, and rebuild the index when any step changes.
- [3] Gaps. Log the questions that retrieval can't answer. They show what content is missing, and give content owners a list of what to write.

### Governance and safety [3]

- [3] Personal data in indexes and memories. Indexes and memories copy data out of its original systems. Know what personal data they hold, remove or mask what you don't need, and apply the source's retention rules.
- [3] Deletion. When a user asks for deletion, or a source document is deleted, remove it from indexes, caches and memories too, including anything derived from it, such as summaries and extracted facts. Design for this from the start, because finding every copy later is hard.
- [3] Consent and transparency for memory. Tell users what the product remembers and why, and get consent where the law requires it. Don't store sensitive personal data, such as health, religion or political views, unless the product needs it and the user agrees.
- [2] Rights to the content. Check that you may index and show each source, especially web content and licensed material. Quote only as much as a citation needs.
- [2] Where the data lives. Vector databases and memory stores hold customer data too, so they must follow the same regional and retention rules as your other data stores.

### Observability [3]

- [3] Logging the context. For every call, log the assembled context, or enough to rebuild it exactly: the prompt version, the search queries, the retrieved chunk IDs with their scores and ranks, the memories used, and the token count of each part. You can't debug an answer without the context it saw. Linking traces to user feedback is covered in `layer_10_data.md`.
- [3] Retrieval in production. Track how often searches return nothing useful, and which documents are retrieved most and which never. These show gaps and drift before users complain.
- [2] Index health. Track index size, the last successful sync for each source, ingestion failures and search latency.

### Cost and latency [3]

- [3] Context size drives cost. Input tokens are most of the cost of many RAG and agent calls. A smaller, better-chosen context is usually the biggest saving, and prompt caching the next (see `layer_06_inference_and_serving.md`).
- [3] Retrieval latency. Rewriting queries, embedding them, searching and reranking all happen before the model starts. Measure each step, run independent steps in parallel, and drop steps that don't pay for themselves. Latency budgets are covered in `layer_06_inference_and_serving.md`.
- [3] Indexing costs. Embedding a large collection, re-embedding it when you change models, adding context to every chunk with a model, and building a knowledge graph all cost money before any user benefits. Estimate these costs before you start, and use batch APIs for bulk work (see `layer_09_compute.md`).
- [2] Storage costs. Vector storage grows with the number of chunks and the size of each vector. Smaller vectors and compression cut it (see `layer_08_foundation_models.md`).

### Reliability [2–3]

- [3] When search fails. Decide what the product does when search is slow or down: answer without retrieval and say so, use cached results, or tell the user. Give retrieval its own timeout, so a slow search doesn't stall the whole request.
- [2] Rebuilding indexes. Changing the embedding model or chunking means rebuilding the index. Build the new index alongside the old one, test it, then switch over, so search never goes down. Record the index version in each trace, so you can reproduce what the model saw.

## Not in this layer

- What embedding models and rerankers are, how to choose them, and how models use long context and examples are covered in `layer_08_foundation_models.md`. This file covers building search and context with them.
- How prompt caching works, and serving embedding models and rerankers, are covered in `layer_06_inference_and_serving.md`. This file covers laying out prompts so caching works.
- Response caching, including semantic caching, belongs to Layer 5 (`layer_05_model_gateway_and_routing.md`).
- Fine-tuning embedding models is covered in `layer_07_model_adaptation.md`. Building retrieval test sets and training data is covered in `layer_10_data.md`.
- How tool calling works, designing tools, and protocols such as MCP belong to Layer 3 (Tools and protocols). This file covers search tools as a source of context.
- Planning, chaining model calls into workflows, coordinating agents and tracking task progress belong to Layer 2 (Agents and orchestration). This file covers what each agent sees and remembers.
- How answers, citations and memory settings are shown to users belongs to Layer 1 (Product and UX).
- General defenses against prompt injection are covered in `concern_03_security.md`, and guardrails on outputs in `concern_04_governance_and_safety.md`.

## Learning order

1. Prompts, including managing them as code. You'll write them before anything else.
2. Context engineering, starting with logging and reading the full context of real calls.
3. Conversation history.
4. Retrieval-augmented generation: a simple pipeline end to end, with an evaluation set that measures retrieval and answers separately. Learn embedding models first (see `layer_08_foundation_models.md`).
5. Preparing documents, search, and query handling and reranking. Improve whichever step your evaluation shows is weakest.
6. Answering from retrieved context.
7. Agentic retrieval, once you've built agents that use tools (Layer 3).
8. Long-term memory.
9. Knowledge graphs, if your evaluation shows relationship questions failing.

## Key reading

- Chip Huyen, *AI Engineering* (O'Reilly, 2025). Chapter 5 covers prompt engineering, and chapter 6 covers RAG, agents and memory.
- Anthropic, "Effective context engineering for AI agents" (2025), and Yichao Ji, "Context Engineering for AI Agents: Lessons from Building Manus" (2025).
- Drew Breunig, "How Long Contexts Fail" and "How to Fix Your Context" (2025), on the ways context goes wrong.
- Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (2020), the paper that named RAG.
- Anthropic, "Introducing Contextual Retrieval" (2024), on adding context to chunks, hybrid search and reranking.
- Manning, Raghavan and Schütze, *Introduction to Information Retrieval* (2008, free online), on keyword search, ranking and retrieval metrics.
- Jason Liu's writing on systematically improving RAG applications.
- Es et al., "RAGAS: Automated Evaluation of Retrieval Augmented Generation" (2023).
- Edge et al., "From Local to Global: A Graph RAG Approach to Query-Focused Summarization" (Microsoft, 2024).
- Packer et al., "MemGPT: Towards LLMs as Operating Systems" (2023), and Park et al., "Generative Agents: Interactive Simulacra of Human Behavior" (2023), on memory for agents.
- Greshake et al., "Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection" (2023), and Simon Willison, "The lethal trifecta for AI agents" (2025).
- Your model providers' prompting guides, and the documentation for your vector database or search engine.
