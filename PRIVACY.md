# Privacy Policy

**Service:** Agent Reliability (agentreliability.dev), including its MCP endpoint at
`https://agentreliability.dev/mcp`.
**Data controller:** Santiago Santa María Morales.
**Contact:** santismm@gmail.com.
**Last updated:** 2026-08-18.

This service is a public, read-only knowledge source. It requires no
account, asks for no personal information, sets **no cookies**, and shows
no advertising. This policy describes the only processing that happens:
anonymous usage measurement.

## What we collect

When you (or an AI agent acting for you) request a page, a machine-readable
file, or an MCP tool call, we record:

- **Aggregate counters** — the kind of client (human browser, AI crawler,
  agent, bot), which surface was used (web page, discovery file, API, MCP),
  the outcome, the arrival channel, and which knowledge objects were
  requested. These are counts per day, not per-visitor records.
- **A raw event log of agent and crawler traffic only.** Requests
  classified as coming from a human browser are counted in the aggregates
  but are **never written to this log**.
- **Latency and error data** for MCP tool calls, to keep the service
  answering well.

## What we do NOT collect

- **Your IP address is never stored.** It arrives with the request, is
  used in the moment, and is discarded. To group requests into short-lived
  sessions, measured traffic is marked with a one-way SHA-256 hash of the
  IP and user-agent, salted with a secret that **rotates every day** and
  truncated to 16 hex characters. Because the salt changes daily, the same
  visitor produces unrelated markers on different days, so the marker
  cannot be used to build a profile over time. If the salt secret is not
  configured, no marker is computed at all.
- No names, emails, or account data — there are no accounts.
- No cookies, no fingerprinting, no cross-site tracking.
- No conversation content: an MCP tool call contains only the tool's
  arguments (for example a search query), and that is all we see.

## Storage, processors and retention

Measurement data is stored in a Redis database operated by **Upstash**
(storage processor). The website and MCP endpoint are hosted on **Vercel**
(hosting processor). Both process data on our behalf.

All counters and logs are written with an expiry of **200 days** and are
deleted automatically after it.

## Third-party sharing

We do not sell, rent, or share usage data with third parties. Aggregate,
non-identifying statistics (for example "how many agent requests answered
successfully this month") may be published as part of the project's public
reporting.

## Your rights

Since we store no direct identifiers, we generally cannot link stored data
to a person. You can still contact santismm@gmail.com with any request or
question about this policy — including access, deletion, or complaint —
and we will answer.

## Content licence

The knowledge content this service returns is licensed CC BY 4.0; that is
a content licence, separate from this privacy policy.

## Changes

Material changes to this policy will be published here with a new date.
