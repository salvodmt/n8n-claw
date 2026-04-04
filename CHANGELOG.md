# Changelog

All notable changes to this project will be documented in this file.
Format based on [Keep a Changelog](https://keepachangelog.com/).

---

## [1.0.0] — 2026-04-05

### Multi-Provider, Zero Config

n8n-claw is now fully model-agnostic. Choose your LLM provider during setup and everything works out of the box — no manual node swapping, no credential juggling. This release also streamlines the installation to a 2-step process: clone & run, then chat.

### Added
- **LLM Provider Abstraction** — setup.sh automatically patches all LLM nodes in every workflow to match your chosen provider before importing
- **8 supported providers**: Anthropic, OpenAI, OpenRouter, DeepSeek, Google Gemini, Mistral, Ollama, OpenAI-compatible
- **Mistral AI** as new provider option
- **Provider switching** via `./setup.sh --force` — re-imports all workflows with new provider nodes
- **Credential PATCH** — existing credentials are updated with current API keys on re-run (instead of reusing stale data)
- **Telegram webhook fix** — second deactivate/activate cycle at end of setup ensures reliable webhook registration on fresh install
- **Ollama model prompt** — interactive model selection during setup
- **File delivery pipeline** — agent can send files (PDFs, images, documents) back to users via `[send_file:]` markers

### Changed
- **Default models updated**: OpenAI → `gpt-5.4`, Gemini → `gemini-3-flash-preview`, Ollama → `glm-4.7-flash`
- **README simplified** — installation reduced to 2 steps (clone & run → chat), removed manual credential setup instructions
- **Memory Consolidation** reads LLM provider config from `tools_config` at runtime (works with any provider)

### Fixed
- Gemini credential type corrected to `googlePalmApi` (matches n8n node expectation)
- OpenRouter default model corrected to `anthropic/claude-sonnet-4-6`
- Connection traversal in LLM node patch for nested workflow structures

### Upgrade from v0.17.0
```bash
cd n8n-claw && git pull && ./setup.sh --force
```
Choose your provider when prompted. All workflows will be re-imported with the correct LLM nodes.

---

## Previous Releases (v0.1.0 – v0.17.0)

### [0.17.0] — 2026-04-03 — File Bridge: Binary File Passthrough
New File Bridge microservice for binary file handling between Telegram, cloud storage, and the agent. Skills (Seafile, Google Drive, Nextcloud) now support upload and download of actual files.

### [0.16.0] — 2026-03-27 — Google OAuth2 & Google Skills
OAuth2 authorization flow via Telegram. Four new Google skills: Gmail, Calendar, Analytics, Ads. Fixed cartesian product bug in agent workflow.

### [0.15.0] — 2026-03-23 — OpenClaw Integration & New MCP Skills
OpenClaw integration (autonomous Linux agent), NocoDB CRM, Vikunja task management. Logo and social preview added.

### [0.14.0] — 2026-03-20 — Webhook API & External Integrations
HTTP webhook endpoint for Slack, Teams, Paperclip, and custom apps. Unified adapter workflow with multi-system support.

### [0.13.0] — 2026-03-19 — Heartbeat Extension
Recurring scheduled actions, Background Checker for silent monitoring, notify_mode control. Email Bridge with IMAP search.

### [0.12.0] — 2026-03-15 — Expert Agents
Sub-agent system with dynamic personas. Agent Library Manager for installing expert agents from catalog. 85+ expert agents available.

### [0.11.0] — 2026-03-14 — Crawl4AI Web Reader
Self-hosted web reader with JavaScript rendering. New MCP skills.

### [0.10.0] — 2026-03-10 — Project Memory & Scheduled Actions
Project document management, scheduled agent actions, reminder system rewrite, Email Bridge microservice, dynamic MCP server loading.

### [0.9.0] — 2026-03-10 — Scheduled Actions & Reminders
Single reminder workflow, auto-cleanup, dynamic MCP loading.

### [0.8.0] — 2026-03-10 — Reminder System
Unified reminder workflow replacing per-reminder approach.

### [0.7.0] — 2026-03-08 — Credential Flow & MCP Templates
Secure credential form for MCP skill API keys. One-time tokens with 10-min TTL. MCP template registry via CDN.

### [0.6.0] — 2026-03-07 — MCP Template Registry
Skill catalog with CDN delivery. Library Manager for install/remove.

### [0.5.0] — 2026-03-06 — Self-Hosted Web Search
SearXNG integration for private web search.

### [0.4.0] — 2026-03-06 — Media Handling
Photo, document, voice message, and location support in Telegram.

### [0.3.0] — 2026-03-06 — Heartbeat & Task Management
Proactive heartbeat, task management, morning briefing.

### [0.2.0] — 2026-03-06 — RAG Pipeline & Memory
Vector embeddings for semantic memory search. Memory consolidation workflow.

### [0.1.0] — 2026-03-05 — First Release
Core agent with Telegram interface, long-term memory, conversation history, MCP Builder, personality system.
