# 🧩 AI Reliability Suite

A full-stack, open-source suite of tools for evaluating, stress-testing, and improving the reliability of large language models (LLMs) and agentic AI systems.

Built using **Next.js (App Router)**, **Vercel AI SDK**, and **Supabase**, the suite demonstrates
cutting-edge practices in AI tooling, structured evaluation, and developer-focused UX — all designed for Edge-first environments.

---

## 🧠 Projects Overview

| Project | Description | Stack |
|----------|--------------|--------|
| 🧩 [EvalForge](./apps/evalforge) | Structured LLM evaluation framework supporting rubric-based, schema-validated, and metamorphic testing. | Next.js • Vercel AI SDK • Supabase • Zod |
| ⚙️ [SynthBench](./apps/synthbench) | Synthetic data and edge-case generator with automated validation and integration into EvalForge. | Next.js • Vercel AI SDK • Supabase • OpenAI API |
| 🔍 [AgentReliabilityLab](./apps/agentreliabilitylab) | Agent reliability sandbox featuring tool-calling, schema contracts, retry loops, and trace visualization. | Next.js • Vercel AI SDK • Supabase • React Flow |

Each module can run independently but together they form a complete workflow for
**testing, evaluating, and debugging LLM-based systems**.

---

## 🏗️ Architecture
ai-reliability-suite/
│
├── apps/
│   ├── evalforge/              # Structured evaluation system
│   ├── synthbench/             # Synthetic data generator
│   └── agentreliabilitylab/    # Agent reliability dashboard
│
├── packages/
│   ├── core-eval/              # Rubric & schema scoring logic
│   ├── core-data/              # DSL fuzzers & validation utilities
│   ├── core-agent/             # Planner, executor, and tool orchestration
│   ├── db/                     # Prisma + Supabase schema
│   └── ui/                     # Shared React & styling components
│
└── deployment/
├── vercel.json             # Vercel config for monorepo deploy
├── .env.example            # Environment variables
└── seed/                   # Demo datasets & tasks

---

## 🚀 Tech Stack

**Core:**  
- Next.js (App Router)  
- Vercel AI SDK  
- TypeScript + Zod  
- Supabase (Postgres + Auth)  
- Prisma (ORM)  

**Supporting:**  
- React Flow (visualization)  
- Recharts / Chart.js (metrics)  
- Tailwind + shadcn/ui (UI system)  
- Better-Auth or NextAuth (optional authentication)  
- Edge Functions for fast AI route execution  

---

## ⚡ Key Highlights
- 🧱 **Full-stack AI tooling** — end-to-end architecture with reusable core packages  
- ⚙️ **Edge-native execution** via Vercel AI SDK and serverless routes  
- 📊 **Structured evaluation** with schema and rubric validation  
- 🔁 **Agent reliability framework** for multi-step tool-using systems  
- 🧩 **Modular monorepo** built for composability, scalability, and open contribution  

---

## 🌐 Live Deployments (Planned)
| App | URL | Status |
|------|-----|---------|
| EvalForge | [evalforge.vercel.app](#) | 🚧 In Development |
| SynthBench | [synthbench.vercel.app](#) | 🚧 In Development |
| AgentReliabilityLab | [agentreliabilitylab.vercel.app](#) | 🚧 In Development |

---

## 📦 Getting Started

### 1️⃣ Clone the Repository
```bash
git clone https://github.com/vip529/ai-reliability-suite.git
cd ai-reliability-suite
```

### 2️⃣ Install Dependencies
```bash
pnpm install
```

3️⃣ Set Up Environment Variables

Copy .env.example → .env.local
Add your keys:

OPENAI_API_KEY=...
SUPABASE_URL=...
SUPABASE_ANON_KEY=...
DATABASE_URL=...

4️⃣ Run in Development

pnpm dev

Each app can be started independently:

pnpm --filter evalforge dev

🧩 Roadmap
	•	EvalForge MVP — Rubric & schema evaluation
	•	SynthBench MVP — Synthetic data generator
	•	AgentReliabilityLab MVP — Tool orchestration sandbox
	•	Supabase integration for unified storage
	•	Auth + user profiles
	•	Public dashboards & reports

⸻

🧠 Vision

AI systems are only as reliable as their evaluation and observability frameworks.
The AI Reliability Suite aims to provide a modular, open-source foundation for
developers and researchers to:
	•	Evaluate LLMs under structured, consistent criteria
	•	Generate adversarial and edge-case datasets automatically
	•	Monitor and visualize agentic reasoning reliability

All while staying Edge-first, type-safe, and developer-friendly.

⸻

👨‍💻 Author

Vipin Yadav
Senior Software Engineer — Full Stack & AI Product Developer
LinkedIn • GitHub
⸻
