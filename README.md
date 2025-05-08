# DogBot

**DogBot** is an early-stage AI project that helps dog owners understand problematic behavior through instinct theory, emotional intelligence, and dialogue — powered by GPT-4 and retrieval-augmented generation (RAG).

It is designed as an interactive coach that simulates the dog's perspective, asks meaningful questions, and helps humans reframe behavior with empathy. The system combines AI reasoning with structured data and behavior logic to support real change in human-canine relationships.

---

## Repository Structure

This meta-repo provides an overview of the project components. Each folder listed here is a separate Git repository and not tracked by this meta-repo.

| Repository | Description |
|------------|-------------|
| [`dogbot-ui`](https://github.com/kemperfekt/dogbot-ui) | React frontend with slot-style UX simulating a dialogue between human and dog |
| [`dogbot-agent`](https://github.com/kemperfekt/dogbot-agent) | Web-accessible backend (wrapper around `dogbot-app`) |
| [`dogbot-ops`](https://github.com/kemperfekt/dogbot-ops) | Scripts for generating and managing structured knowledge (e.g., symptoms, behavior patterns) |

---

## Technologies Used

- Python, Pydantic
- OpenAI GPT-4 API
- React (frontend)
- Weaviate (vector search & RAG)

---

## Current Status

DogBot is in development/ research phase.
---

## Author

Developed by [Philipp Kemper](https://github.com/kemperfekt), with a background in product strategy, cross-cultural teams, and behavior-first system design. This project draws on personal interest in dogs, group dynamics, and how communication shapes relationships — both human and machine.

---

*This project was created with GPT-4 — prompts engineered by a humble human.*
