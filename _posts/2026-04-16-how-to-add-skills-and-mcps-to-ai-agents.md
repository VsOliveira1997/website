---
title: "How to Add Skills and MCPs to AI Agents"
date: 2026-04-16
categories: [AI, Tooling]
tags: [ai, skills, mcp, agents, automation]
image: /assets/img/posts/skills-mcp-linkedin.png
---

Most people still solve everything with a prompt. That works for short tasks, but it becomes very confusing when the AI needs to repeat a workflow, follow a pattern, or interact with external systems.

To improve an agent, there are usually two complementary paths:

- `skills`: teach the AI *how* to handle a class of tasks
- `MCPs`: a bridge between the AI and the program that is running, as in the Ghidra MCP example

In short, a skill organizes behavior, and an MCP exposes capability.

## What a skill is

A skill is a reusable instruction package. Instead of relying on a long prompt every time, you define a focused operating guide for a specific job:

- when the skill should be used
- which steps the agent should follow
- which constraints matter
- what kind of output should come back

A good skill reduces randomness. If you keep asking the AI to perform the same type of task, that pattern should probably be turned into a skill.

Examples:

- reviewing code for regressions
- triaging bug bounty findings
- writing reports in a fixed structure
- analyzing logs and returning prioritized hypotheses

A simple `SKILL.md` can look like this:

```md
---
name: api-review
description: Review API endpoints for auth, validation, and data exposure issues.
---

Use this skill when the user asks for API review, quick threat modeling, or logic flaw hunting.

Workflow:
1. Map routes and HTTP verbs.
2. Identify authentication and authorization per endpoint.
3. Check for IDOR, BFLA, mass assignment, and weak validation.
4. Prioritize findings by impact.
5. Respond with findings, evidence, and next tests.

Rules:
- Do not assume controls without evidence.
- Separate facts from hypotheses.
- Keep the output concise and direct.
```

That alone already changes how an agent behaves, because it stops improvising from scratch and starts following a playbook.

## What an MCP is

MCP, or `Model Context Protocol`, is a standard way to connect models and agents to external tools and data sources. Instead of building a custom integration for every environment, MCP gives you a consistent interface for exposing things like:

- tools
- resources
- prompts

In practice, that means the AI can fetch live context, call actions, and interact with real systems with less glue code and less ad hoc behavior.

If you built your own MCP, it can act as the bridge between the model and whatever already exists in your environment:

- an internal API
- an admin panel
- a read-only database
- repositories
- documentation
- systems for CRM, ads, observability, or automation

That is the real jump in usefulness: the AI stops depending only on the current conversation and starts working with live context.

## Skills and MCPs solve different problems

This is where a lot of people get confused.

A skill does not replace an MCP, and an MCP does not replace a skill.

- A `skill` defines when to use an approach and how to execute it.
- An `MCP` provides the tools that make the approach actionable.

Together, they make an agent far more consistent.

A simple way to think about it:

- without a skill, the AI may have access to a tool but use it poorly
- without an MCP, the AI may know the process but have no reliable way to act outside the chat
- with both, it knows what to do and has the means to do it

## How to add skills to an AI

The implementation varies by product, but the pattern is usually the same:

1. Create a skill file or skill directory.
2. Write the description, usage triggers, and workflow.
3. Define scope, rules, and output expectations.
4. Test it on real tasks and refine it.

The most common mistake is making the skill too generic. Good skills are narrow enough to activate clearly and broad enough to be reused.

A minimal structure usually includes:

- name
- description
- when to use it
- steps
- constraints
- response format

If your agent supports local skills, the structure often ends up looking like this:

```text
~/.codex/skills/
  api-review/
    SKILL.md
```

or inside the project itself:

```text
.ai/
  skills/
    api-review.md
```

The exact folder name matters less than the quality of the instruction. What matters is that it is actionable and specific. It is also worth being careful with the number of skills: depending on the AI, too many skills can consume too many tokens and hurt the response.

## How to add an MCP to an AI

This tends to be more straightforward than people expect. You usually need three pieces:

1. An MCP server.
2. A client that supports MCP.
3. A configuration telling the AI where that server lives.

Depending on your stack, the MCP server might be a local binary, a Node.js script, a Python process, or a remote service.

One real example is [GhidraMCP](https://github.com/lauriewired/ghidramcp), which connects an AI client to Ghidra for reverse engineering workflows. A configuration can look like this:

```toml
[mcp_servers.ghidra]
command = "python"
args = [
  "/path/to/bridge_mcp_ghidra.py",
  "--ghidra-server",
  "http://127.0.0.1:8080/"
]
```

After that, the AI can discover and use the tools exposed by that server.

In `Claude Code`, the same integration pattern can be shared in the repository with a `.claude.json` file:

```json
{
  "mcpServers": {
    "ghidra": {
      "command": "python",
      "args": [
        "/path/to/bridge_mcp_ghidra.py",
        "--ghidra-server",
        "http://127.0.0.1:8080/"
      ]
    }
  }
}
```

In the case of `GhidraMCP`, this becomes easier to visualize because it exposes operations tied directly to the reverse engineering workflow, for example:

- decompiling and analyzing binaries in Ghidra
- automatically renaming methods and data
- listing methods, classes, imports, and exports

This kind of organization makes it easier to understand what the AI can call, which parameters each operation expects, and what kind of response comes back from the MCP server.

## Closing

At the end of the day, the idea is simple: `skills` help the AI follow a process, and `MCPs` help the AI access real tools and data. When both work together, the agent stops being just an improved chatbot and becomes something genuinely useful in day-to-day work.

## References

- [Anthropic: Intro to MCP](https://modelcontextprotocol.io/introduction)
- [Anthropic: Model Context Protocol overview](https://www.anthropic.com/news/model-context-protocol)
- [Subframe Docs: Agent skills](https://docs.subframe.com/guides/skills)
