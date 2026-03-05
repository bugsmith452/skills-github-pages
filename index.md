---
layout: default
title: Home
---

# About Me

I build AI/LLM‑powered developer tools, agentic workflows, and distributed data platforms. Most of my work sits at the intersection of backend engineering, AI integration, and complex data flows in constrained/regulated environments.

## Recent Work

**AI/LLM Integration & Agentic Workflows**
- Architecting a multi‑agent Copilot‑like SDLC assistant for multiple LIMS platforms (Sapio, Sample Manager, LabVantage), orchestrating flows from requirements → HLD/LLD → code using AutoGen 2.0, Azure OpenAI, AWS Bedrock, and Milvus‑based RAG.
- Building LSP‑ and MCP‑based coding agents by wrapping the Java language server (JDTLS) as MCP tools so agents can navigate project structure, inspect diagnostics, apply code actions, and generate platform‑aware Java code against SDKs/javadocs.

**Distributed Data Platforms**
- Designing and shipping large‑scale data pipelines on Azure Databricks/Delta Lake for 70+ lab instruments, with near–real‑time ingestion, standardized logical data models, and reliable APIs for ELN/LIMS systems.

**Extensible Frameworks & Platforms**
- Implementing extensible Java frameworks and plugins for pharmacokinetic analysis and QC document automation (Decorator/Factory patterns, configuration‑driven behavior, shared charting and conversion utilities).

I enjoy owning components end‑to‑end: understanding requirements, designing systems, making tradeoffs (flexibility vs. complexity, performance vs. maintainability), and stabilizing them in production with good observability and error‑handling.

## Areas of Focus

- AI/LLM integration, RAG, multi‑agent orchestration, MCP tools
- Distributed data processing (Azure, Databricks, Delta Lake, ADLS)
- Extensible frameworks, SDK‑style platforms, design patterns in Java/Python/C#
- Reliability, schema evolution, backward‑compatible APIs, observability

---

## Latest Posts

Check out my [blog](/blog) for technical articles and insights.

**Featured:** [Stop Letting Your Coding Agent Hallucinate Private APIs — Build an MCP Server That Feeds It Real Signatures]({% post_url 2026-03-06-mcp-server-maven-deps %})