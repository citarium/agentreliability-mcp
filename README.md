# Agent Reliability — MCP server

> Testing, benchmarking and auditing autonomous AI agents — methods, harnesses, evidence

A **remote MCP server** over a curated knowledge graph. Every claim it
returns is bound to a registered source: the tools hand back claims *with*
their citations and a confidence value, so an agent can show its work
instead of asserting.

Nothing to install. It is a hosted streamable-HTTP endpoint:

```
https://agentreliability.dev/mcp
```

## Add it to a client

**Claude Code**

```bash
claude mcp add --transport http agent-reliability https://agentreliability.dev/mcp
```

**Claude Desktop / any client reading `mcpServers`**

```json
{
  "mcpServers": {
    "agent-reliability": {
      "type": "streamable-http",
      "url": "https://agentreliability.dev/mcp"
    }
  }
}
```

No API key, no account, no auth. Read-only.

**Check it answers, without any client at all:**

```bash
curl -s https://agentreliability.dev/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H 'mcp-protocol-version: 2025-06-18' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Tools

Eight, each with an `outputSchema`, each returning `structuredContent`.

| tool | arguments | what it does |
|---|---|---|
| `get_overview` | — | Corpus overview: what this instance knows, counts by type, published tags, freshness. **Start here** when you land and do not yet know whether this corpus can answer your question. |
| `search` | `query`, `limit?` | Full-text search over the knowledge graph. Accent- and apostrophe-insensitive, so query in the user's own words; every hit carries its relevance score and the fields it matched. |
| `answer` | `question` | Answer a question from the corpus. Returns the matched object's claims with sources and confidence — **never an unsourced answer**. |
| `get_entity` | `id` | Fetch one knowledge object by id, with its claims and the sources each claim cites. |
| `get_topic` | `tag` | List the knowledge objects carrying a tag (topics are content-backed tags). |
| `get_related` | `id` | Graph neighbours of an object: outgoing and incoming relations, each with its relation type. |
| `get_sources` | `object_id?` | The whole source registry, or just the sources cited by one object. Use it to judge the corpus before trusting it. |
| `get_latest` | `limit?` | Most recently verified knowledge objects — a freshness signal. |

The intended path is `get_overview` → `search` or `answer` → `get_entity`
→ `get_related`. `get_overview` exists because an agent that has just
arrived needs to know whether this corpus can help *before* it spends a
call guessing.

## What is in the corpus

| | |
|---|---|
| knowledge objects | **38** |
| registered sources | **31** |
| published topics | **93** |

| type | objects |
|---|---|
| entity | 25 |
| guide | 9 |
| comparison | 2 |
| faq | 1 |
| glossary | 1 |

Subject matter: evals and benchmarks (GAIA, AgentBench, Inspect), LLM-as-judge and its failure modes, Goodhart and benchmark contamination, fault injection and chaos testing, approval gates and autonomy levels, grounding and faithfulness.

### Questions it is built to answer

- *How do I tell a real eval from a benchmark my agent has memorised?*
- *What does calibration mean for an LLM judge, and how is it measured?*
- *Which failure modes does fault injection actually catch?*

## What an answer actually looks like

A real call against the live endpoint — `answer` with
*"how do I tell a real eval from benchmark contamination"* — returns this `structuredContent`, trimmed:

```json
{
  "answered": true,
  "entity": {
    "id": "agent-reliability-glossary",
    "name": "Agent reliability glossary",
    "evidence_tier": "secondary",
    "confidence": 0.85,
    "last_verified": "2026-08-08",
    "canonical_url": "https://agentreliability.dev/k/agent-reliability-glossary"
  },
  "claims": [
    {
      "text": "An eval is a structured, repeatable test that measures an LLM or LLM-based system against a defined dimension; frameworks package evals as registries of reusable templates.",
      "sources": [{ "title": "openai/evals — framework for evaluating LLMs and LLM systems" }]
    }
  ]
}
```

Note what travels with the answer: the **evidence tier**, a **confidence**,
the date it was **last verified**, and the **source behind the claim** — not
as prose an agent has to parse, but as fields it can act on. An agent can
decline to use a weak claim, or cite the primary source directly.

When the corpus cannot answer, `answered` is `false`. It does not
improvise, and the miss is recorded so the gap can be filled.

## Machine-readable surfaces

The MCP endpoint is one of several. The same corpus is served as plain
files an agent can read directly:

| surface | what it is |
|---|---|
| [`/llms.txt`](https://agentreliability.dev/llms.txt) | the index, as `text/plain` |
| [`/llms-full.txt`](https://agentreliability.dev/llms-full.txt) | the whole corpus in one file |
| [`/ai-index.json`](https://agentreliability.dev/ai-index.json) | every surface this instance publishes, with its content type |
| [`/api/index.json`](https://agentreliability.dev/api/index.json) | one JSON document per knowledge object |
| [`/api/sources.json`](https://agentreliability.dev/api/sources.json) | the source registry, in full |
| [`/.well-known/mcp/server.json`](https://agentreliability.dev/.well-known/mcp/server.json) | this server's manifest |

Each knowledge object has a human page and a machine twin at the same id,
with a canonical URL that agrees across all of them.

## Behaviour worth knowing before you integrate

- **`POST` only.** Every other method answers `405` with an `Allow: POST, OPTIONS` header.
- **Rate limit:** 120 requests per minute per client, counted in a shared
  store, published on every response as `RateLimit-Limit`,
  `RateLimit-Remaining` and `RateLimit-Reset` (all three exposed via CORS).
  It fails **open**: if the store is unreachable the request is served.
- **Malformed input** gets a spec-correct JSON-RPC error — `-32700` for
  unparseable bodies, `-32602` for an unknown tool — never an HTML error page.
- **Request bodies are capped** and validated before transport.

## Privacy

No accounts, no cookies, no ads. Usage is measured in aggregate with
daily-rotating hashed identifiers and a 200-day retention; raw IPs are
never stored. Full policy: [PRIVACY.md](./PRIVACY.md).

## Provenance and licence

Knowledge content is **CC-BY-4.0**: use it, cite it. The source registry is
public precisely so a claim can be checked rather than trusted —
`get_sources` returns what any given claim rests on.

Claims carry an evidence tier and a `last_verified` date. Where the
evidence is weaker, the object says so rather than rounding up.

## How it is built

Compiled and served by [Citarium](https://github.com/citarium/citarium), an
open-source framework for turning a knowledge graph into a website, an API,
an MCP server and agent-readable files from a single source — under
external evaluation, with the guardians and the falsification record in the
open.

This repository is the server's public face: its manifest and its
documentation. The corpus itself lives at [agentreliability.dev](https://agentreliability.dev).
