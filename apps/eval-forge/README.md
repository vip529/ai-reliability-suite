# 🧠 EvalForge — Structured LLM Evaluation Framework

EvalForge is a full-stack framework for evaluating large-language-model (LLM) outputs using
structured rubrics, schema validation, and metamorphic testing.

Built with **Next.js**, **Vercel AI SDK**, and **Supabase**, it lets developers define evaluation
tasks (prompt + rubric + expected schema) and automatically compute rubric-weighted scores and
explanations for model responses.

## ✨ Features
- 🔹 Rubric-based and metamorphic evaluation of model outputs  
- 🔹 JSON Schema & Zod-based structured output validation  
- 🔹 Edge-deployed routes via **Vercel AI SDK**  
- 🔹 **Supabase Postgres** for storing tasks, runs, and reports  
- 🔹 Markdown & JSON report generation for reproducible scoring

## 🧱 Tech Stack
Next.js (App Router) • Vercel AI SDK • TypeScript • Supabase • Zod • Edge Functions

## 🧩 Why It Matters
EvalForge provides a foundation for consistent, reproducible LLM benchmarking and reliability
tracking—critical for evaluating agentic systems and structured outputs.