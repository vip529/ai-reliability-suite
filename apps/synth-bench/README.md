# ⚙️ SynthBench — Synthetic Data & Edge-Case Generator

SynthBench is a data-generation and fuzz-testing platform designed to create challenging
synthetic datasets for evaluating and fine-tuning LLMs.

It uses **Vercel AI SDK**, **OpenAI/Hugging Face APIs**, and a custom **DSL fuzzer** to produce
adversarial, negated, and boundary-case prompts. Each dataset passes strict validation before
export to **EvalForge** for scoring.

## ✨ Features
- 🔹 Automatic edge-case & counterfactual generation for LLMs  
- 🔹 Mini-DSL fuzzing engine (regex, JSON, SQL-like grammars)  
- 🔹 Schema & semantic validation with auto-filtering  
- 🔹 Seamless integration with EvalForge evaluation pipeline  
- 🔹 Dataset storage & export via **Supabase**

## 🧱 Tech Stack
Next.js (App Router) • Vercel AI SDK • Supabase • TypeScript • Zod • OpenAI API

## 🧩 Why It Matters
SynthBench helps teams identify model blind spots and robustness gaps without manual data
creation, improving evaluation coverage for production LLMs.