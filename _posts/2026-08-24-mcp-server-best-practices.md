---
layout: post
title: "Best Practices for Building MCP Servers"
date: 2026-08-24
author: Devansh Dutta
tags:
  [
    mcp,
    ai-agents,
    tool-design,
    context-engineering,
    security,
    error-handling,
    production,
    llm,
  ]
category: AI
categories: AI
giscus_comments: false
related_posts: false
featured: true
toc:
  sidebar: left
pretty_table: true

---

# Best Practices for Building MCP Servers

_By Devansh · August 2026 · ~10 min read_

MCP servers are easy to build, but making them reliable and secure for production is more difficult. The main challenges are security, tool design, context size, performance, error handling, and deployment.

In our experience building and operating MCP servers, a reliable one should:

- Provide a small number of useful tools.
- Design tools around what the user wants to achieve, not around backend APIs.
- Keep tool names, inputs, and outputs simple and consistent.
- Give the AI clear descriptions of when and how to use each tool.
- Avoid sending large amounts of unnecessary data to the AI.
- Handle errors clearly so the AI can recover.
- Use proper authentication and permissions for sensitive actions.

The MCP protocol is also continuing to evolve, with newer versions improving security, scalability, and server management. Therefore, MCP servers should be designed to be secure, maintainable, and easy to update.

## What makes MCP server design different?

Traditional APIs are built for software applications that already know **which API to call and what data to send**. MCP servers are different because they're built for **AI agents** — the AI has to figure out which tool to use, when to use it, what to pass in, how to read the result, and what to do next.

Because of this, MCP tools should be simple, clear, and focused on completing the user's task. The goal isn't to expose every backend API — it's to give the AI the right tools to complete the task reliably.

![Traditional API exposing three separate operations versus an MCP server exposing one outcome-oriented tool](/assets/img/mcp-design-shift.png)  
_Don't make the AI manage your backend workflow when the MCP server can do it internally._

## Best practices for tool design

**Design around outcomes, not operations.** A common implementation mistake is to copy every REST API or database operation into a separate MCP tool. Instead, design tools around the outcome the AI needs to achieve. Combine related steps into one useful tool when the AI doesn't need to handle each step itself, and keep complex processing inside the server. Simple rule: don't make the AI manage your backend workflow when the MCP server can do it internally.

**Keep the tool surface curated.** Too many tools make it harder for the AI to decide which one to use. Keep the server focused on one business or technical domain, avoid tools that do almost the same thing, and test your tools to decide whether some should be combined or separated. As a practical starting point, **5–15 tools** can work well, but this is not an official MCP requirement. If a server exposes a very large number of tools, consider dynamic tool discovery so the AI only loads what it needs. In our own MCP work, an overgrown tool list is consistently one of the first things that hurts tool-selection accuracy — trimming it down tends to pay off quickly.

**Naming consistency is a reliability requirement.** If the same concept has different names across tools, the AI can make incorrect tool calls. Use the same name for the same concept everywhere — don't mix `user_id`, `customer_id`, and `accountId` for the same thing. When multiple servers are in play, prefix tool names with the service, e.g. `github_create_issue`, `slack_send_message`. Consistent naming reduces ambiguity and makes tool selection and argument generation more reliable — it's one of the simplest fixes we've found for cutting down incorrect tool calls.

**Use structured schemas for the contract.** Don't put the entire input/output structure only inside the tool description. Use `inputSchema` and, where useful, `outputSchema`; use enums and constraints; mark required vs. optional parameters clearly; and validate inputs server-side regardless of what the schema says. The description explains how to use the tool — the schema defines what it actually accepts and returns.

## Prompting and context optimization

**Treat tool descriptions as agent UX.** The AI uses the tool description to decide what the tool does and when to use it. Explain what it does, when (and when not) to use it, important formats or naming rules, and what it returns. Instead of _"Searches data,"_ use _"Searches customer records by name or email. Use this tool when you need to find an existing customer. Returns matching customer IDs and basic info."_

**Use behavioral metadata carefully.** Annotations like `readOnlyHint`, `destructiveHint`, and `idempotentHint` are hints, not security controls. A `destructiveHint` doesn't automatically prevent a dangerous action — sensitive or destructive actions still need real authorization and approval behind them.

**Control response size.** The AI's context window is limited. Paginate large lists, let the AI filter results, set sane default limits, return only what's needed rather than the full record, and pass a file/path reference instead of embedding large content directly in tool arguments.

## Error handling and resilience

**Make errors useful to the agent.** Return clear, actionable errors and tell the AI what it can try next — don't expose raw stack traces, and don't return an empty "successful" response when the operation actually failed. Instead of `Error 404`, use _"Customer not found. Try searching by email or customer_id."_

**Design mutations for retries.** Agents and network connections may retry the same call. Use idempotency keys so a retried "create order" doesn't create two orders, make repeated requests safe whenever possible, require proper authorization and confirmation for irreversible actions, and test what happens when the same request is sent multiple times.

**The stdio logging rule.** Over stdio transport, stdout carries MCP protocol messages — writing normal logs there corrupts the stream. Keep stdout for protocol messages only; send logs to stderr or a separate logging system:

```
✗  print("Fetching customer data...")

✓  import logging
   logging.basicConfig(level=logging.INFO)
   logging.info("Fetching customer data...")
```

## Security and production deployment

**Authentication is not authorization.** Authentication asks who's making the request; authorization asks what they're allowed to do. Check permissions at the tool and resource level, use RBAC, mask sensitive data the AI shouldn't see, and keep audit logs of important actions. A user being authenticated to use an MCP server doesn't mean they should be allowed to use every tool on it.

**Remote HTTP needs a different security posture.** Moving from local stdio to remote HTTP is a major security change: use TLS, OAuth-based authentication, validate the OAuth issuer, configure CORS and redirect URLs securely, add rate limiting, keep audit logs, and validate downstream requests to reduce SSRF risk. Note: the 2026-07-28 MCP specification is moving away from OAuth Dynamic Client Registration (DCR) toward Client ID Metadata Documents — not an immediate breaking change, since DCR stays supported for backward compatibility for at least 12 months.

**The Lethal Trifecta.** A risky combination of three capabilities in one agent: access to private data, access to untrusted content, and the ability to communicate externally. With all three, an attacker may be able to trick the agent into sending private information to an external destination — and MCP servers can easily provide all three at once.

![Triangle diagram of the Lethal Trifecta: private data access, untrusted content, and external communication converging on a data exfiltration risk](/assets/img/mcp-lethal-trifecta.png)  
_Reduce the risk by giving the AI only the private data it needs, treating external content as untrusted, limiting outbound actions, and requiring approval for high-impact steps._

**Audit logging.** Log who made the request, which tool was used, success/failure status, duration, and a request/correlation ID to trace an action across systems. Strip sensitive data before writing logs — don't log passwords, API keys, full prompts, or full tool responses by default — and define retention and access policy.

**Containerization and runtime hygiene.** MCP servers still need standard production engineering: containers for consistency, pinned runtime/dependency versions, health checks, safe shutdown, CPU/memory/concurrency limits, and configuration via environment variables so stdio and HTTP can share one codebase instead of separate implementations. In our experience, runtime and dependency mismatches are a common cause of unexpected restarts after deployment, so these controls are worth treating as part of the deployment baseline rather than an afterthought.

## Key challenges

| Challenge                | What goes wrong                                                  | Impact                                                 | Recommended solution                                              |
| ------------------------ | ---------------------------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------- |
| **API–Agent Mismatch**   | Backend APIs expose individual operations instead of user goals. | More tool calls, higher latency, wrong tool selection. | Design tools around user goals; handle complex steps server-side. |
| **Context Bloat**        | Too many tools or too much data given to the AI.                 | Higher token usage, weaker reasoning.                  | Keep tools focused, paginate, use dynamic discovery.              |
| **Naming Inconsistency** | Same concept, different names across tools.                      | Wrong arguments, unreliable calls.                     | Use consistent names across the server.                           |
| **Schema Ambiguity**     | Input/output structure only explained in text.                   | AI generates the wrong structure.                      | Use structured schemas; validate server-side.                     |
| **Silent Failures**      | Failed operation returns an empty/misleading result.             | AI thinks it succeeded.                                | Return clear errors; validate the result.                         |
| **Authentication Gaps**  | Weak or no authentication.                                       | Unauthorized access.                                   | Secure authentication on protected deployments.                   |
| **Authorization Gaps**   | Authenticated users with too many permissions.                   | Unauthorized actions, data leakage.                    | Enforce permissions at the tool/resource level.                   |
| **Prompt Injection**     | Untrusted content treated as instructions.                       | Unsafe actions.                                        | Treat external content as untrusted; restrict capabilities.       |
| **Large Payloads**       | Large files/raw data sent through context.                       | Bloat, truncation, unreliable calls.                   | Use references, pagination, limited responses.                    |
| **Production Issues**    | Runtime/config problems surface after deployment.                | Restarts, outages, inconsistent behavior.              | Health checks, fixed versions, monitoring, load testing.          |

## Testing and evaluation strategy

Testing an MCP server should check not just whether the tools work, but whether the AI can use them correctly, safely, and reliably.

| Test area              | What to test                                   | Example                         | Metric                     |
| ---------------------- | ---------------------------------------------- | ------------------------------- | -------------------------- |
| **Tool Selection**     | Does the AI choose the correct tool?           | Search vs. retrieve             | Selection accuracy         |
| **Arguments**          | Does the AI provide correct parameters?        | IDs, dates, enums               | Valid-call rate            |
| **Task Completion**    | Can the AI complete the entire task?           | Resolve a support request       | Task success rate          |
| **Context Efficiency** | Is unnecessary data or too many calls used?    | Large search result             | Tokens per successful task |
| **Security**           | Can the AI be tricked into an unsafe action?   | Malicious tool output           | Unsafe actions blocked     |
| **Reliability**        | Does it handle failures and retries correctly? | Downstream 429/500 error        | Recovery rate              |
| **Concurrency**        | Does it work under multiple requests?          | 20/50/100 simultaneous requests | Error rate & latency       |
| **Regression**         | Do tool changes affect AI performance?         | Rename a parameter              | Evaluation score change    |

## Note:

Building an MCP server is not just about exposing functions to an AI agent. Reliability depends on how clearly tools are designed, how much context they expose, how failures are handled, and how access is controlled. Keeping the tool surface focused, using explicit schemas, returning actionable errors, and testing agent behavior alongside server functionality provides a practical foundation for production-ready MCP systems — this is the approach we've settled on through our own work building and running MCP servers.

As MCP continues to evolve, these practices also make servers easier to maintain, evaluate, and adapt to changes in models and deployment environments.
