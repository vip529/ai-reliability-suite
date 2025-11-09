# 🔍 AgentReliabilityLab — Agentic System Reliability Sandbox

AgentReliabilityLab is an experimental sandbox for analyzing and improving the reliability of
tool-using and multi-step AI agents.

It implements **planner/executor logic**, **tool-calling** via **Vercel AI SDK**, and
**schema-based output contracts** with automatic repair and retry mechanisms. Each run is
recorded as a structured trace with detailed metrics and visualized in an interactive dashboard.

## ✨ Features
- 🔹 Tool-calling & schema-validated agent execution  
- 🔹 Self-repair, retry, and consistency verification loops  
- 🔹 Trace visualization (React Flow) with success/latency metrics  
- 🔹 Reliability dashboards backed by **Supabase**  
- 🔹 Edge-function orchestration for low-latency runs

## 🧱 Tech Stack
Next.js (App Router) • Vercel AI SDK • TypeScript • Supabase • React Flow • Zod

## 🧩 Why It Matters
AgentReliabilityLab enables controlled testing of reasoning consistency, cascading-error
handling, and self-verification—key to building trustworthy agentic AI systems.