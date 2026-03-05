---
layout: default
title: "Stop Letting Your Coding Agent Hallucinate Private APIs — Build an MCP Server That Feeds It Real Signatures"
date: 2026-03-06
categories: [ai, mcp, java, coding-agents]
---

# Stop Letting Your Coding Agent Hallucinate Private APIs — Build an MCP Server That Feeds It Real Signatures

*AI coding agents are remarkably capable on public codebases. They've been trained on GitHub, Maven Central, npm, PyPI. But the moment you point them at your team's internal libraries — the proprietary SDKs, the custom frameworks, the artifact zoo that only exists inside your VPN — they start guessing. And they guess wrong.*

*This is the gap Maven Deps Inspector MCP was built to close.*

---

## The Problem Nobody Talks About in Agentic Coding

Most of the conversation around AI coding agents focuses on public code. Claude, Copilot, Cursor — they all have strong priors on Spring Boot, Apache Commons, Guava, Jackson. The signatures are baked into the model weights from training data.

But teams working on large or specialised codebases often build on layers of internal libraries. Custom frameworks. Domain-specific toolkits. Artifacts that live in private Nexus or Artifactory repositories and have no public documentation, no StackOverflow answers, no GitHub issues. The model has never seen them.

When you ask an agent to write code that calls `com.yourcompany.platform.OrderService.submit(Order)`, it either refuses (if it is being honest) or invents a plausible-looking method signature that fails to compile — the method, parameter type, or return type simply does not exist on that class.

The root issue is a **context gap**: the agent knows how to write Java, but it does not know what your library actually exposes.

---

## The MCP Approach: Give the Agent Real Bytecode Knowledge

The [Model Context Protocol](https://modelcontextprotocol.io/) gives AI agents a structured way to call external tools at inference time. Instead of relying solely on training data, the agent can ask a tool server for information and get back grounded, accurate answers.

Maven Deps Inspector MCP is a Java-based MCP server that:

1. Reads your project's `pom.xml` (including transitive dependencies)
2. Resolves JARs from your local Maven repository (`.m2/repository`, Nexus, Artifactory — anywhere `mvn install` has already fetched to)
3. Extracts class, method, and field signatures directly from `.class` bytecode using the ASM library
4. Serves that information to any MCP-capable agent over stdio JSON-RPC

**No source code needed. No Javadoc JARs. No internet access to the artifact registry.** The information comes straight from the compiled bytecode that is already on disk.

---

## What the Agent Can Ask

Once the server is running, the agent gets five tools:

- **`searchDependencies(className)`** — "What classes named `EventHandler` are indexed?" Returns all matching fully-qualified names with their GAV coordinates.
- **`getClassDocumentation(classFqn)`** — "Give me the full class signature for `com.yourcompany.platform.OrderService`." Returns modifiers, inheritance, all fields, all methods.
- **`getMethodDocumentation(classFqn, methodName)`** — Returns the exact method signature, parameter types, return type, and whether it overrides a parent.
- **`getFieldDocumentation(classFqn, fieldName)`** — Returns field type and modifiers.
- **`clearCache`** — Invalidates the on-disk cache when you have rebuilt your artifacts.

A coding agent working on an unfamiliar codebase can now **look before it leaps**. Before writing a method call, it queries the signature. Before constructing an object, it checks the constructor. Before referencing a constant, it verifies the field exists.

---

## Disambiguation: When Names Collide

Larger codebases are full of naming collisions. How many classes named `Logger`, `EventHandler`, `PluginBase`, or `ServiceLocator` does your codebase have across a decade of artifacts?

The server handles this through the MCP **elicitation** capability. When an agent finds multiple classes matching a simple name, it can surface an interactive disambiguation dialog — the user picks the right one by fully-qualified name. Agents that don't support elicitation get a structured error with all candidates listed, so they can retry with `classFqn`.

---

## The Signature-Before-Writing Pattern

Here's the workflow shift this enables:

**Before (without the MCP server):**
1. Agent writes code using guessed or hallucinated API
2. Code fails to compile — method or type does not exist
3. Developer manually looks up the correct API
4. Agent is corrected and rewrites
5. Repeat 2-3 times

**After (with the MCP server):**
1. Agent calls `searchDependencies` to find the class
2. Agent calls `getClassDocumentation` to read the actual fields and methods
3. Agent writes code using the verified signature
4. Code compiles and runs correctly on the first try

This isn't just a quality improvement — it's a **latency reduction**. In a fast-moving agentic workflow, each correction loop costs time, tokens, and human attention. Eliminating the guessing step pays back immediately.

---

## Wiring It Into Your Agent's Instructions

The MCP server is most effective when you also tell your agent to use it. In Claude Code this means adding a line to your `CLAUDE.md`:

```markdown
Before writing code that uses any class from an internal or unfamiliar dependency,
call searchDependencies to locate the class and getClassDocumentation to verify
its fields and methods. Never guess method signatures.
```

With this in place the agent treats signature lookup as a first step, not a fallback. The MCP server provides the capability; the system instruction enforces the habit. Together they turn the agent from a "write then fix" loop into a "check then write" flow that stays accurate from the first edit.

The same instruction works in Claude Desktop system prompts, Cursor rules files, or any agent framework that accepts natural-language guidance.

---

## Setup in Three Commands

```bash
# 1. Build
mvn package -DskipTests

# 2. Wire into Claude Code
claude mcp add maven-deps-inspector \
  java -jar /path/to/maven-deps-inspector-mcp-1.0.0.jar \
       --config /path/to/config.json

# 3. Point it at your project
# config.json:
{
  "cachePath": "~/.maven-deps-inspector-cache",
  "projects": [{ "pomPath": "/path/to/your/project/pom.xml" }]
}
```

The server indexes your dependency graph on startup and refreshes every 5 minutes. Results are cached on disk keyed by SHA-256 of the artifact coordinates, so repeated lookups are instant.

---

## Why Bytecode, Not Source?

Source code is not always available for internal artifacts — especially for older libraries or third-party SDKs distributed as binary-only. Javadoc JARs are even less reliably published. But the compiled `.class` files are always there: if Maven resolved the dependency, the bytecode is in `.m2`.

ASM gives us everything the agent needs — class hierarchy, method descriptors, field types, access modifiers — without requiring a single line of source.

---

## What's Coming Next

The current version surfaces signatures from bytecode. The next step is Javadoc. Many Maven artifacts ship a companion `*-javadoc.jar` alongside the main JAR, and those contain the full prose documentation — parameter descriptions, return value semantics, thrown exceptions, usage notes. A future version of Maven Deps Inspector will extract and serve that documentation alongside the signatures, giving agents not just the shape of an API but the intent behind it.

---

## The Bigger Picture

As agentic coding workflows mature, the bottleneck shifts from "can the agent write code" to "does the agent have accurate context about the system it is writing for." Public library knowledge is largely solved by training data. Private library knowledge is the next frontier.

MCP servers are the right abstraction for this. They are lightweight, stdio-based, easy to wire into any MCP-compatible client (Claude Desktop, Claude Code, Cursor, or any custom agent), and they keep the context window clean by serving only what the agent asks for.

Maven Deps Inspector is one piece of that puzzle — the piece that answers: *"What does this class actually look like?"*

The code is straightforward enough to fork and adapt: swap in a different bytecode parser, add support for non-Maven build systems, or extend the schema with annotation metadata. The MCP transport layer stays the same.

---

*The project is available on GitHub. Prerequisites: Java 17+, Maven 3.8+, a Maven local repository populated with your dependencies.*

---

[← Back to Blog](/blog)
