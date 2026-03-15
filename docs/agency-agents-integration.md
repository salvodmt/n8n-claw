# agency-agents Integration — Brainstorming & Implementierungsplan

> **Status:** Implementiert (Option C — Sub-Agents) auf Branch `feature/expert-agents`
> **Datum:** 2026-03-15
> **Betrifft:** n8n-claw (primär), dmo-claw (sekundär)
> **Quelle:** [msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents) (31k Stars)

---

## Inhaltsverzeichnis

1. [Was ist agency-agents?](#1-was-ist-agency-agents)
2. [Warum ist das relevant für n8n-claw?](#2-warum-ist-das-relevant-für-n8n-claw)
3. [Transferability-Analyse: Welche Personas passen?](#3-transferability-analyse-welche-personas-passen)
4. [Mapping: agency-agents → soul/agents-Tabellen](#4-mapping-agency-agents--soulagents-tabellen)
5. [Option A: /persona-Switching via Telegram](#5-option-a-persona-switching-via-telegram)
6. [Option B: Persona-Templates-Repo](#6-option-b-persona-templates-repo)
7. [Option C: Sub-Agents (Delegation statt Persona-Switch)](#7-option-c-sub-agents-delegation-statt-persona-switch)
8. [Vergleich aller drei Optionen](#8-vergleich-aller-drei-optionen)
9. [Empfehlung & Roadmap](#9-empfehlung--roadmap)
10. [dmo-claw: Personas, Rollen & Sub-Agents](#10-dmo-claw-personas-rollen--sub-agents)
11. [Token-Budget & Prompt-Größe](#11-token-budget--prompt-größe)
12. [Offene Fragen](#12-offene-fragen)

---

## 1. Was ist agency-agents?

Eine Open-Source-Sammlung von **144 spezialisierten KI-Agenten-Prompts**, organisiert in 12 Divisions:

| Division | Agents | Beispiele |
|----------|--------|-----------|
| Engineering | 23 | Backend Architect, Security Engineer, AI Engineer, DevOps Automator |
| Marketing | 23 | Instagram Curator, Content Creator, SEO Specialist, Growth Hacker |
| Specialized | 21 | MCP Builder, Automation Governance Architect, Blockchain Auditor |
| Game Development | 14 | Game Designer, Unity Architect, Godot Scripter |
| Design | 8 | UI Designer, UX Researcher, Brand Guardian |
| Sales | 8 | Outbound Strategist, Discovery Coach, Deal Strategist |
| Testing | 8 | Evidence Collector, Performance Benchmarker, API Tester |
| Support | 6 | Support Responder, Analytics Reporter, Finance Tracker |
| Project Management | 6 | Studio Producer, Project Shepherd |
| Paid Media | 7 | PPC Strategist, Search Query Analyst |
| Spatial Computing | 6 | XR Interface Architect, visionOS Engineer |
| Product | 4 | Sprint Prioritizer, Trend Researcher, Feedback Synthesizer |

### Aufbau einer Persona

Jeder Agent ist eine `.md`-Datei mit YAML-Frontmatter und 9 standardisierten Sektionen:

**Frontmatter:**
```yaml
---
name: SEO Specialist
description: Expert search engine optimization strategist...
tools: WebFetch, WebSearch, Read, Write, Edit
color: "#4285F4"
emoji: 🔍
vibe: Drives sustainable organic traffic through technical SEO and content strategy.
services:                    # optional
  - name: Service Name
    url: https://...
    tier: free
---
```

**Sektionen (am Beispiel SEO Specialist / Instagram Curator):**

| # | Sektion | Inhalt | Beispiel (Instagram Curator) |
|---|---------|--------|------------------------------|
| 1 | **Identity & Memory** | Rolle, Persönlichkeit, Erfahrungshintergrund | "Visual storytelling expert, Instagram culture mastery" |
| 2 | **Core Mission** | 3-5 Hauptverantwortlichkeiten mit Unterpunkten | Brand Development, Multi-Format Content, Community Building, Commerce |
| 3 | **Critical Rules** | Nicht-verhandelbare Regeln und Grenzen | "Content-Mix: 1/3 Marke, 1/3 Wissen, 1/3 Community" |
| 4 | **Technical Deliverables** | 2-3 konkrete Templates mit Code-Beispielen | Brand Aesthetic Guide, Content Calendar, Hashtag Strategy |
| 5 | **Communication Style** | Ton, Stimme, Kommunikationsansatz | "Visuell denken, Trends einbeziehen, Ergebnisse messbar machen" |
| 6 | **Workflow Process** | Schritt-für-Schritt-Methodik (3-4 Phasen) | Phase 1: Aesthetic → Phase 2: Strategy → Phase 3: Community → Phase 4: Optimization |
| 7 | **Learning & Memory** | Muster-Erkennung, Expertise-Aufbau | "Track algorithm changes, learn content performance patterns" |
| 8 | **Success Metrics** | Quantifizierbare Ziele mit konkreten Zahlen | "Engagement > 3.5%, Story Completion > 80%, K-Factor > 1.0" |
| 9 | **Advanced Capabilities** | 2-3 spezialisierte Fähigkeiten | Shopping Integration, Algorithm Mastery, Community Management |

### OpenClaw-Konvertierung

Das Repo enthält ein Konvertierungsskript (`scripts/convert.sh`), das jede Persona in drei Dateien aufteilt:

```
~/.openclaw/agency-agents/{agent-slug}/
├── SOUL.md       ← Identity, Communication, Style, Critical Rules
├── AGENTS.md     ← Core Mission, Deliverables, Workflow, Metrics, Capabilities
└── IDENTITY.md   ← Name + Emoji + Vibe (One-Liner)
```

**Klassifizierungslogik (Regex auf `##`-Headern):**
```
Header enthält "identity"          → SOUL.md
Header enthält "communication"     → SOUL.md
Header enthält "style"             → SOUL.md
Header enthält "critical rule(s)"  → SOUL.md
Alles andere                       → AGENTS.md
```

Diese Aufteilung korrespondiert direkt mit n8n-claws Tabellenstruktur.

---

## 2. Warum ist das relevant für n8n-claw?

### Strukturelle 1:1-Übereinstimmung

| agency-agents | n8n-claw | Inhalt |
|---------------|----------|--------|
| SOUL.md | `soul`-Tabelle | Persönlichkeit, Kommunikationsstil, Grenzen |
| AGENTS.md | `agents`-Tabelle | Mission, Tools, Workflow, Metriken |
| IDENTITY.md | `soul.name` + `soul.vibe` | Name und Kurzbeschreibung |

### n8n-claw ist bereits persona-ready

Die bestehende Architektur unterstützt Persona-Switching mit minimalen Änderungen:

1. **Dynamischer System-Prompt:** Der "Build System Prompt" Code-Node (`workflows/n8n-claw-agent.json`, Zeile 934) baut den System-Prompt bei **jeder eingehenden Nachricht** neu aus der Datenbank. Persona-Änderungen wirken sofort — kein Workflow-Neustart nötig.

2. **Flexibles User-Profil:** `user_profiles.preferences` ist ein JSONB-Feld, das beliebige Daten speichern kann — z.B. `{ "current_persona": "technical" }`. Bereits im Schema vorhanden, keine Migration nötig.

3. **Erweiterbare agents-Tabelle:** Die `agents`-Tabelle hat ein simples `key`/`content`-Schema. Neue Einträge mit einem `persona:`-Prefix können neben bestehenden Keys (mcp_instructions, tools, etc.) koexistieren.

4. **Library Manager als Vorbild:** Der `updateAgentConfig()`-Mechanismus im Library Manager (`workflows/mcp-library-manager.json`) aktualisiert bereits dynamisch die `agents`-Tabelle nach jeder Skill-Installation. Dasselbe Pattern funktioniert für Personas.

### Was n8n-claw aktuell fehlt

- **Nur eine Persona:** Die `soul`-Tabelle definiert eine feste Persönlichkeit (beim Setup konfiguriert). Kein Wechsel möglich.
- **Kein Persona-Catalog:** Es gibt keine Sammlung vordefinierter Persönlichkeiten zum Auswählen.
- **Kein Kommando:** Telegram hat kein `/persona`-Kommando — alle Nachrichten gehen direkt an den Agent.

---

## 3. Transferability-Analyse: Welche Personas passen?

### Übersicht

Von 144 agency-agents sind **4 direkt relevant** für n8n-claw/dmo-claw (Tourism/Productivity):

| Agent | Größe | Soul-Anteil | Agents-Anteil | Dev-spezifisch | Direkt nutzbar |
|-------|-------|-------------|---------------|----------------|----------------|
| **Instagram Curator** | ~2.5K chars | 40% | 60% | ~5% | **95%** |
| **Content Creator** | ~2.8K chars | 35% | 65% | ~15% | **85%** |
| **Support Responder** | ~4.2K chars | 35% | 65% | ~45% | **60%** |
| **Analytics Reporter** | ~3.1K chars | 30% | 70% | ~55% | **50%** |

Die restlichen 140 Agents (Solidity Engineer, XR Cockpit Specialist, Godot Scripter etc.) haben keine Relevanz für einen Tourism-Chatbot.

### Detailanalyse pro Agent

#### Instagram Curator (95% nutzbar)

**Direkt übertragbar:**
- Identity: "Visual storytelling expert, Instagram culture mastery"
- Communication: "Visuell denken, Trends einbeziehen, Ergebnisse messbar"
- Critical Rules: Content-Mix (1/3 Marke, 1/3 Wissen, 1/3 Community), CTA-Regeln
- Success Metrics: Engagement > 3.5%, Story Completion > 80%
- Workflow: 4-Phasen-Prozess (Aesthetic → Strategy → Community → Optimization)

**Braucht Adaption:** Fast nichts — kaum Dev-spezifischer Content.

#### Content Creator (85% nutzbar)

**Direkt übertragbar:**
- 8 Kernfähigkeiten (Strategy, Multi-Format, Brand Narrative, SEO, Distribution, Tracking)
- Success Metrics: 25% Engagement, 40% Traffic Growth, 70% Video Completion
- Workflow-Prozess für Content-Erstellung

**Braucht Adaption:**
- "Deploy this agent"-Szenarien → umformulieren zu "Wann nutzen"
- Einige Capabilities setzen Dev-Tools voraus (z.B. "automation pipelines")

#### Support Responder (60% nutzbar)

**Direkt übertragbar:**
- Identity: "Empathisch, lösungsorientiert"
- Communication: 4 Prinzipien (Empathie, Lösungen, Proaktivität, Klarheit)
- Critical Rules: Kundenzufriedenheit vor internen Metriken, Eskalationsregeln
- Success Metrics: CSAT 4.5+, First Contact Resolution 80%+, SLA 95%+

**Braucht schwere Adaption:**
- Omnichannel-Framework (Email/Chat/Phone/Social/In-App) → n8n hat nur Telegram/OpenWebUI
- Python-Klasse `SupportAnalytics` → müsste als n8n-Workflow oder HTTP-Node umgebaut werden
- Python-Klasse `KnowledgeBaseManager` → komplett dev-spezifisch

#### Analytics Reporter (50% nutzbar)

**Direkt übertragbar:**
- Identity: Analytisch, datengetrieben
- Communication: Statistische Signifikanz betonen, Business Outcomes fokussieren
- Workflow: 4-Phasen (Data Discovery → Analysis → Insight → Impact)
- Success Metrics: 95%+ Accuracy, 70%+ Implementation Rate

**Braucht schwere Adaption:**
- SQL-Dashboard-Template → dev-spezifisch
- Python-RFM/KMeans-Analyse → dev-spezifisch
- JavaScript-Attribution-Modeling → dev-spezifisch
- Alle Code-Templates setzen direkten Sprach-Runtime-Zugang voraus

### Muster

**Marketing-Agents** (Instagram Curator, Content Creator) sind fast 1:1 übertragbar — sie beschreiben Kommunikation und Strategie, nicht Code.

**Technische Agents** (Support Responder, Analytics Reporter) haben wertvolle Persönlichkeits- und Kommunikationsfragmente, aber ihre "Technical Deliverables" sind Python/SQL-Code für Dev-Assistenten und müssten für n8n komplett umgeschrieben werden.

---

## 4. Mapping: agency-agents → soul/agents-Tabellen

### Prinzip

```
agency-agents Sektion         →  n8n-claw Tabelle.Key
────────────────────────────────────────────────────────
Frontmatter (name, vibe)      →  soul.name, soul.vibe
Identity & Memory             →  soul.persona
Communication Style           →  soul.communication
Critical Rules                →  soul.boundaries
Core Mission                  →  agents.persona:{id} (Block 1)
Technical Deliverables        →  agents.persona:{id} (Block 2)
Workflow Process              →  agents.persona:{id} (Block 3)
Success Metrics               →  agents.persona:{id} (Block 4)
Advanced Capabilities         →  agents.persona:{id} (Block 5)
Learning & Memory             →  agents.persona:{id} (Block 6)
```

### Konkretes Beispiel: Instagram Curator → n8n-claw

**Variante 1: Alles in einem `agents`-Eintrag** (empfohlen für Einfachheit)

```sql
-- agents-Tabelle
INSERT INTO agents (key, content) VALUES ('persona:instagram-curator', '
# Instagram Curator

## Persönlichkeit
Du bist eine visuelle Storytelling-Expertin mit tiefem Instagram-Verständnis.
Du denkst in Ästhetik, Engagement und Community-Building.

## Kommunikation
- Visuell denken: Immer an die visuelle Wirkung denken
- Trends einbeziehen: Aktuelle Formate und Features nutzen
- Ergebnisse messbar machen: Engagement-Raten, Completion-Rates
- Community-Fokus: Interaktion über Broadcasting

## Regeln
- Content-Mix: 1/3 Marke, 1/3 Wissen/Inspiration, 1/3 Community/UGC
- Jeder Post braucht einen klaren CTA
- Brand-Konsistenz über alle Formate
- Authentizität vor Perfektion

## Workflow
1. Ästhetik definieren (Farben, Fonts, Bildsprache)
2. Content-Strategie (Themenplan, Format-Mix, Posting-Zeiten)
3. Community-Building (Antworten, UGC, Kooperationen)
4. Optimierung (Analytics, A/B-Tests, Algorithmus-Anpassung)

## Erfolgsmetriken
- Engagement Rate > 3.5%
- Story Completion > 80%
- Follower-Wachstum > 5%/Monat
- Regelmäßiger UGC-Input
');
```

**Variante 2: Aufgeteilt in soul + agents** (näher am OpenClaw-Format)

```sql
-- soul-Tabelle: Persönlichkeit überschreiben
UPDATE soul SET content = 'Visuelle Storytelling-Expertin mit tiefem Instagram-Verständnis. Denkt in Ästhetik, Engagement und Community.' WHERE key = 'persona';
UPDATE soul SET content = 'Visuell, trend-bewusst, ergebnisorientiert, community-fokussiert.' WHERE key = 'vibe';

-- agents-Tabelle: Operational
INSERT INTO agents (key, content) VALUES ('persona:instagram-curator', '## Mission\n...\n## Workflow\n...\n## Metriken\n...');
```

**Empfehlung:** Variante 1 (alles in einem `agents`-Eintrag) — einfacher zu verwalten, kein Überschreiben der `soul`-Tabelle nötig, Persona-Wechsel betrifft nur eine Zeile.

---

## 5. Option A: `/persona`-Switching via Telegram

### Konzept

Personas werden als Zeilen in der `agents`-Tabelle gespeichert (Key-Prefix `persona:`). Ein Telegram-Kommando `/persona` wechselt zwischen ihnen. Die aktive Persona wird in `user_profiles.preferences` gespeichert.

### Datenfluss

```
User: /persona list
  → Telegram Trigger
    → Route: erkennt /persona-Kommando
      → DB Query: SELECT key FROM agents WHERE key LIKE 'persona:%'
        → Telegram Reply: "Verfügbare Personas: default, technical, instagram-curator"

User: /persona instagram-curator
  → Telegram Trigger
    → Route: erkennt /persona-Kommando
      → DB Update: UPDATE user_profiles SET preferences = jsonb_set(preferences, '{current_persona}', '"instagram-curator"')
        → Telegram Reply: "Persona gewechselt zu: Instagram Curator 📸"

User: Erstelle einen Post über die Berge
  → Telegram Trigger
    → Load Soul / Load Agents / Load User Profile
      → Build System Prompt:
          - Liest preferences.current_persona = "instagram-curator"
          - Findet agents.key = "persona:instagram-curator"
          - Injiziert Persona-Content in System-Prompt
        → AI Agent (mit Instagram-Curator-Persona)
          → Antwort mit visuellem Fokus, Content-Mix-Regeln, Engagement-Metriken
```

### Betroffene Dateien & Code-Änderungen

#### 1. `workflows/n8n-claw-agent.json` — Build System Prompt

**Aktuelle Logik (Zeile 934):**
```javascript
const agentText = agents.map(a => {
  if (a.key === 'mcp_instructions') { /* MCP-Injektion */ }
  return `## ${a.key}\n${a.content}`;
}).join('\n\n');
```

**Neue Logik (~10 Zeilen hinzufügen):**
```javascript
// ── Persona-Lookup ──
const prefs = user.preferences || {};
const currentPersona = prefs.current_persona || 'default';
const personaEntry = agents.find(a => a.key === `persona:${currentPersona}`);
const personaText = personaEntry
  ? `# ACTIVE PERSONA: ${currentPersona}\n${personaEntry.content}`
  : '';

// ── Reguläre agents (ohne persona:-Prefix) ──
const regularAgents = agents.filter(a => !a.key.startsWith('persona:'));
const agentText = regularAgents.map(a => {
  if (a.key === 'mcp_instructions') { /* bestehende MCP-Injektion */ }
  return `## ${a.key}\n${a.content}`;
}).join('\n\n');

// ── System-Prompt zusammenbauen ──
const systemPrompt = `# CURRENT TIME\n${now}\n\n# SOUL\n${soulText}\n\n${personaText}\n\n# AGENT CONFIG\n${agentText}\n\n# USER PROFILE\n${userText}${scheduledHint}${projectsSection}\n\n# RECENT CONVERSATION\n${historyText}`;
```

#### 2. `workflows/n8n-claw-agent.json` — `/persona`-Kommando-Routing

**Wo:** Neuer IF-Node nach "Normalize Message", vor "Build System Prompt"

**Logik (im Normalize-Message-Code-Node oder als separater Node):**
```javascript
const userMessage = /* ... bestehende Normalisierung ... */;

// /persona Kommando erkennen
if (userMessage.startsWith('/persona')) {
  const arg = userMessage.substring(9).trim();
  return [{
    json: {
      command: 'persona',
      arg: arg || 'list',    // "list" wenn kein Argument
      chatId,
      userId,
      source: 'telegram'
    }
  }];
}

// Normaler Nachrichtenfluss (wie bisher)
return [{ json: { userMessage, chatId, userId, source } }];
```

**Handler-Node (neuer Code-Node für /persona):**
```javascript
const { command, arg, chatId, userId } = $input.first().json;

if (arg === 'list') {
  // Alle verfügbaren Personas laden
  const personas = await $helpers.httpRequest({
    method: 'GET',
    url: SUPABASE_URL + '/rest/v1/agents?key=like.persona:*&select=key',
    headers: pgHeaders
  });
  const names = personas.map(p => p.key.replace('persona:', '')).join(', ');
  return [{ json: { chatId, reply: `📋 Verfügbare Personas: ${names}\n\nWechseln mit: /persona {name}` } }];
}

// Persona wechseln
const personaKey = `persona:${arg}`;
const exists = await $helpers.httpRequest({
  method: 'GET',
  url: SUPABASE_URL + `/rest/v1/agents?key=eq.${personaKey}&select=key`,
  headers: pgHeaders
});

if (exists.length === 0) {
  return [{ json: { chatId, reply: `❌ Persona "${arg}" nicht gefunden. Nutze /persona list` } }];
}

// User-Profil updaten
await $helpers.httpRequest({
  method: 'PATCH',
  url: SUPABASE_URL + `/rest/v1/user_profiles?user_id=eq.telegram:${userId}`,
  headers: { ...pgHeaders, 'Prefer': 'return=minimal' },
  body: JSON.stringify({ preferences: { current_persona: arg } })
});

return [{ json: { chatId, reply: `✅ Persona gewechselt zu: **${arg}**\nAb jetzt antworte ich in diesem Stil.` } }];
```

#### 3. `setup.sh` — Default-Persona seeden

**Wo:** Nach dem bestehenden agents-Seeding (Zeile ~1150)

```python
# Default-Persona (aktuelles Verhalten als Fallback)
cur.execute("""
    INSERT INTO agents (key, content) VALUES ('persona:default', %s)
    ON CONFLICT (key) DO UPDATE SET content = EXCLUDED.content, updated_at = NOW()
""", ("""# Default
Keine spezielle Persona aktiv. Verwende die Standard-Persönlichkeit aus der Soul-Tabelle.
""",))
```

### Vorteile

- **Zero Schema Changes** — nutzt bestehende Tabellen (`agents`, `user_profiles`)
- **Sofort wirksam** — System-Prompt wird bei jeder Nachricht neu aus DB gebaut
- **Minimal invasiv** — ~20 Zeilen im Build System Prompt, ~30 Zeilen für den /persona Handler
- **Kein Workflow-Neustart** nötig nach Persona-Wechsel
- **Fallback eingebaut** — ohne gesetzte Persona greift `persona:default`

### Nachteile

- **Manuelle DB-Pflege** — Personas müssen per SQL oder PostgREST eingefügt werden
- **Kein Versioning** — Persona-Änderungen haben keine Versionierung
- **Nicht distribuierbar** — andere n8n-claw-Instanzen können Personas nicht "installieren"
- **Kein Catalog/Discovery** — es gibt keinen zentralen Katalog verfügbarer Personas
- **Skalierung begrenzt** — bei 20+ Personas wird die `agents`-Tabelle unübersichtlich
- **Kein CDN** — Personas leben nur in der lokalen DB

---

## 6. Option B: Persona-Templates-Repo (analog MCP Skills)

### Konzept

Personas werden als Templates in einem dedizierten Repo (oder als neue Kategorie im bestehenden `n8n-claw-templates`-Repo) verwaltet. Der Library Manager wird um den Typ `"persona"` erweitert und installiert Personas als `agents`-Einträge (statt Workflows zu importieren).

### Architektur

```
n8n-claw-templates/              # bestehendes Repo (ODER neues Repo)
├── templates/
│   ├── index.json               # Katalog — erweitert um Persona-Einträge
│   ├── weather-openmeteo/       # bestehende MCP Skills
│   │   ├── manifest.json
│   │   └── workflow.json
│   ├── persona-instagram-curator/   # NEU: Persona-Template
│   │   ├── manifest.json
│   │   └── persona.json            # NEU: statt workflow.json
│   ├── persona-content-creator/
│   │   ├── manifest.json
│   │   └── persona.json
│   └── ...
```

### Template-Format für Personas

**`manifest.json`:**
```json
{
  "id": "persona-instagram-curator",
  "name": "Instagram Curator",
  "version": "1.0.0",
  "updated": "2026-03-13",
  "type": "persona",
  "category": "personas",
  "description": "Visuelle Storytelling-Expertin mit Instagram-Fokus. Content-Mix, Engagement-Metriken, Community-Building.",
  "source": "agency-agents/marketing-instagram-curator",
  "author": "freddy-schuetz",
  "license": "MIT",
  "tools": [],
  "credentials_required": [],
  "emoji": "📸",
  "color": "#E1306C"
}
```

**`persona.json` (NEU — ersetzt workflow.json):**
```json
{
  "format": "n8n-claw-persona",
  "format_version": 1,
  "persona_key": "persona:instagram-curator",
  "display_name": "Instagram Curator",
  "content": "# Instagram Curator\n\n## Persönlichkeit\nDu bist eine visuelle Storytelling-Expertin...\n\n## Kommunikation\n- Visuell denken...\n\n## Regeln\n- Content-Mix: 1/3 Marke...\n\n## Workflow\n1. Ästhetik definieren...\n\n## Erfolgsmetriken\n- Engagement Rate > 3.5%..."
}
```

### Datenfluss: Installation

```
User (via Telegram): "Installiere die Instagram Curator Persona"
  → AI Agent erkennt Intent → ruft Library Manager Tool auf
    → Library Manager: action = "install_template", templateId = "persona-instagram-curator"

Library Manager Flow:
  1. Fetch manifest.json von CDN
  2. Erkenne type = "persona"
  3. Fetch persona.json von CDN (statt workflow.json)
  4. INSERT INTO agents (key, content) VALUES ('persona:instagram-curator', persona.content)
  5. Optional: INSERT INTO mcp_registry für Tracking
     { template_id: 'persona-instagram-curator', template_type: 'persona', active: true }
  6. updateAgentConfig() — aktualisiert mcp_instructions mit Liste installierter Personas
  7. Return: "Persona 'Instagram Curator' installiert. Aktiviere mit /persona instagram-curator"
```

### Betroffene Dateien & Code-Änderungen

#### 1. `workflows/mcp-library-manager.json` — Persona-Branch

**Wo:** Im Haupt-Code-Node, nach dem bestehenden `install_template`-Flow

```javascript
// ── Bestehender Flow für MCP Skills ──
if (manifest.type === 'native' || manifest.type === 'bridge') {
  // ... bestehende Logik (Import sub + server workflow) ...
}

// ── NEUER Flow für Personas ──
if (manifest.type === 'persona') {
  // 1. persona.json von CDN laden (statt workflow.json)
  const personaData = await helpers.httpRequest({
    method: 'GET',
    url: `${CDN_BASE}/${templateId}/persona.json`
  });
  const persona = JSON.parse(personaData);

  // 2. Validierung
  if (persona.format !== 'n8n-claw-persona') {
    return error('Invalid persona format');
  }

  // 3. In agents-Tabelle einfügen
  await helpers.httpRequest({
    method: 'POST',
    url: `${SUPABASE_URL}/rest/v1/agents`,
    headers: { ...pgHeaders, 'Prefer': 'return=minimal' },
    body: JSON.stringify({
      key: persona.persona_key,
      content: persona.content
    })
  });

  // 4. Optional: mcp_registry für Tracking
  await helpers.httpRequest({
    method: 'POST',
    url: `${SUPABASE_URL}/rest/v1/mcp_registry`,
    headers: { ...pgHeaders, 'Prefer': 'return=minimal' },
    body: JSON.stringify({
      server_name: manifest.name,
      template_id: templateId,
      template_type: 'persona',
      active: true,
      tools: [],
      path: templateId,
      mcp_url: ''        // Personas haben keine MCP-URL
    })
  });

  // 5. Agent-Config updaten (Persona-Liste aktualisieren)
  await updateAgentConfig();

  return success(`Persona "${manifest.name}" installiert. Aktiviere mit /persona ${templateId.replace('persona-', '')}`);
}
```

**Uninstall-Branch:**
```javascript
if (registryEntry.template_type === 'persona') {
  // 1. Aus agents-Tabelle entfernen
  await helpers.httpRequest({
    method: 'DELETE',
    url: `${SUPABASE_URL}/rest/v1/agents?key=eq.persona:${templateId.replace('persona-', '')}`,
    headers: pgHeaders
  });

  // 2. Aus mcp_registry entfernen
  await helpers.httpRequest({
    method: 'DELETE',
    url: `${SUPABASE_URL}/rest/v1/mcp_registry?template_id=eq.${templateId}`,
    headers: pgHeaders
  });

  // 3. Agent-Config updaten
  await updateAgentConfig();

  // 4. Falls ein User diese Persona aktiv hatte → auf default zurücksetzen
  await helpers.httpRequest({
    method: 'PATCH',
    url: `${SUPABASE_URL}/rest/v1/user_profiles?preferences->>current_persona=eq.${templateId.replace('persona-', '')}`,
    headers: { ...pgHeaders, 'Prefer': 'return=minimal' },
    body: JSON.stringify({ preferences: { current_persona: 'default' } })
  });

  return success(`Persona "${registryEntry.server_name}" entfernt.`);
}
```

#### 2. `workflows/n8n-claw-agent.json` — Build System Prompt

Gleiche Änderung wie bei Option A (Persona-Lookup + Filter).

#### 3. `n8n-claw-templates/templates/index.json` — Catalog erweitern

```json
[
  { "id": "weather-openmeteo", "name": "Weather (Open-Meteo)", "type": "native", "category": "weather", "..." },
  { "id": "persona-instagram-curator", "name": "Instagram Curator", "type": "persona", "category": "personas", "description": "..." },
  { "id": "persona-content-creator", "name": "Content Creator", "type": "persona", "category": "personas", "description": "..." }
]
```

#### 4. `updateAgentConfig()` erweitern

Die bestehende Funktion listet nur MCP-Skills. Erweitern um Persona-Auflistung:

```javascript
async function updateAgentConfig() {
  // Bestehend: MCP Skills auflisten
  const mcpServers = await fetchActiveServers();
  const skillsList = mcpServers
    .filter(s => s.template_type !== 'persona')
    .map(s => `- ${s.server_name}: ${s.mcp_url} (Tools: ${s.tools.join(', ')})`)
    .join('\n');

  // NEU: Personas auflisten
  const personas = mcpServers
    .filter(s => s.template_type === 'persona')
    .map(s => `- ${s.template_id.replace('persona-', '')}: ${s.server_name}`)
    .join('\n');

  const personaSection = personas
    ? `\n\n## Installierte Personas\nUser kann mit /persona {name} wechseln:\n${personas}`
    : '';

  const instructions = `## MCP Client\n...\n## Currently installed MCP Skills\n${skillsList}${personaSection}`;

  await patchAgents('mcp_instructions', instructions);
}
```

### Vorteile

- **Nutzt bewährte Infrastruktur** — CDN, Library Manager, Catalog-System, alles vorhanden
- **Versioniert & distribuierbar** — Git-Repo + CDN, pinned commit hashes
- **Konsistentes UX** — "Installiere Instagram Curator" funktioniert wie "Installiere Weather"
- **Catalog-Discovery** — "Zeig mir verfügbare Personas" lädt von CDN
- **Community-fähig** — andere können Personas beitragen (PR auf das Repo)
- **Sauber trennbar** — Uninstall entfernt alles (agents-Eintrag + Registry + User-Reset)
- **Skaliert** — 50+ Personas im Katalog problemlos

### Nachteile

- **Mehr initialer Aufwand** — Library Manager erweitern, Repo-Struktur aufsetzen (~4-6h)
- **Kein workflow.json nötig** — Template-System ist für Workflows gebaut, Personas sind nur Text
- **CDN-Cache** — jsDelivr-Cache muss bei neuen Personas mit Commit-Hash-Update gemanagt werden
- **Overhead** — Für etwas, das letztlich INSERT/DELETE auf einer Tabelle ist
- **mcp_registry Zweckentfremdung** — Tabelle heißt "mcp_registry", speichert aber auch Personas (Naming-Mismatch)

---

## 7. Option C: Sub-Agents (Delegation statt Persona-Switch)

### Konzept

Die Haupt-Persönlichkeit des Agents bleibt **komplett unverändert**. Stattdessen delegiert der Haupt-Agent bestimmte Aufgaben an spezialisierte Sub-Agents, die jeweils mit ihrer eigenen Persona und einem eigenen AI-Agent-Node arbeiten.

**Kern-Idee:** Ein einziger **"Sub-Agent Runner"** Workflow, der dynamisch die gewünschte Persona aus der Datenbank lädt — nicht ein Workflow pro Persona.

### Warum das funktioniert

n8n's AI Agent Node unterstützt dynamische System-Prompts (`={{ $json.systemPrompt }}`). Der Haupt-Agent macht das bereits so (Build System Prompt lädt aus DB). Dasselbe Pattern funktioniert in einem Sub-Workflow: Persona aus DB laden → als System-Prompt setzen → AI Agent ausführen → Ergebnis zurückgeben.

**Bestehende Vorstufe:** Der WorkflowBuilder (`workflows/workflow-builder.json`) hat bereits eine separate Claude-Instanz (via Claude Code CLI Node). Das zeigt, dass das Pattern in n8n funktioniert.

### Architektur

```
Haupt-Agent (soul + agents, UNVERÄNDERT)
│
├── Bestehende Tools (Memory, HTTP, MCP Client, Reminder, etc.)
│
└── toolWorkflow → "Expert Agent" (Sub-Agent Runner)
    │
    Sub-Agent Runner (EIN Workflow für ALLE Personas):
    │
    ├── executeWorkflowTrigger
    │     Empfängt: { agent: "instagram-curator", task: "...", context: "..." }
    │
    ├── Postgres Node (Load Persona)
    │     SELECT content FROM agents WHERE key = 'persona:instagram-curator'
    │
    ├── Code Node (Build Sub-Agent Prompt)
    │     Kombiniert: Persona-Content + Konversations-Kontext + Aufgabe
    │
    ├── AI Agent Node (Sub-Agent)
    │     systemMessage: {{ $json.systemPrompt }}  ← dynamisch!
    │     text: {{ $json.userMessage }}
    │     model: Claude Sonnet 4.6
    │     maxIterations: 5
    │     Tools: [HTTP Request, Web Search, ...]
    │
    └── Return → Ergebnis an Haupt-Agent
```

### Datenfluss (konkretes Beispiel)

```
User: "Erstelle einen Instagram-Post über die Bergsaison"

1. Haupt-Agent erkennt: Marketing-Aufgabe → delegiert an Expert Agent
   Tool-Call: expert_agent({
     agent: "instagram-curator",
     task: "Erstelle einen Instagram-Post über die Bergsaison",
     context: "[letzte 5 Chat-Nachrichten als Kontext]"
   })

2. Sub-Agent Runner:
   → Lädt persona:instagram-curator aus agents-Tabelle
   → Baut System-Prompt: Persona + Kontext
   → AI Agent iteriert mit Instagram-Persona:
     - Analysiert Thema "Bergsaison"
     - Recherchiert Trends (Web Search)
     - Erstellt Caption nach 1/3-Regel
     - Schlägt Hashtags + Posting-Zeit vor
   → Gibt strukturiertes Ergebnis zurück

3. Haupt-Agent:
   → Empfängt Sub-Agent-Ergebnis als Tool-Result
   → Formuliert Antwort in seinem EIGENEN Ton
   → Sendet an User via Telegram
```

**Wichtig:** Der User merkt keinen Bruch — der Haupt-Agent antwortet immer in seiner eigenen Stimme. Der Sub-Agent arbeitet "hinter den Kulissen".

### Betroffene Dateien & Code-Änderungen

#### 1. Neuer Workflow: `workflows/sub-agent-runner.json`

```
Nodes:
  1. Execute Workflow Trigger (empfängt agent, task, context)
  2. Postgres: Load Persona (SELECT FROM agents WHERE key = 'persona:' + agent)
  3. Code: Build Sub-Agent Prompt
  4. AI Agent (Claude Sonnet 4.6, maxIterations: 5)
  5. Claude LLM (Anthropic API Credential)
  6. HTTP Request Tool (für API-Aufrufe)
  7. Web Search Tool (optional)
  8. Code: Format Result (gibt Output zurück)
```

**Build Sub-Agent Prompt (Code-Node):**
```javascript
const input = $input.first().json;
const persona = $('Load Persona').first()?.json;

if (!persona || !persona.content) {
  return [{
    json: {
      systemPrompt: 'Du bist ein hilfreicher Assistent.',
      userMessage: input.task
    }
  }];
}

const systemPrompt = `# EXPERT AGENT: ${input.agent}

${persona.content}

# KONTEXT
Du wurdest als Experten-Agent für eine spezifische Aufgabe aktiviert.
Bearbeite die Aufgabe gründlich und gib ein strukturiertes Ergebnis zurück.
Antworte auf Deutsch, es sei denn die Aufgabe erfordert eine andere Sprache.

${input.context ? '# GESPRÄCHSKONTEXT\n' + input.context : ''}`;

return [{
  json: {
    systemPrompt,
    userMessage: input.task
  }
}];
```

#### 2. `workflows/n8n-claw-agent.json` — Neuer toolWorkflow-Node

```json
{
  "id": "tool-expert-agent",
  "name": "Expert Agent",
  "type": "@n8n/n8n-nodes-langchain.toolWorkflow",
  "typeVersion": 2.2,
  "parameters": {
    "name": "expert_agent",
    "description": "Delegate a task to a specialized expert agent. Use when the task benefits from deep domain expertise (e.g., Instagram content creation, data analysis, customer support writing). The expert works independently and returns a detailed result.\n\nAvailable experts are listed in the agent config. Pass the expert identifier, a detailed task description, and relevant context from the conversation.",
    "workflowId": {
      "__rl": true,
      "value": "REPLACE_SUB_AGENT_RUNNER_ID",
      "mode": "id"
    },
    "workflowInputs": {
      "mappingMode": "defineBelow",
      "value": {
        "agent": "={{ $fromAI('agent', 'Expert agent identifier (e.g. instagram-curator, content-creator)', 'string') }}",
        "task": "={{ $fromAI('task', 'Detailed task description for the expert', 'string') }}",
        "context": "={{ $fromAI('context', 'Relevant context from the current conversation', 'string') }}"
      },
      "schema": [
        { "id": "agent", "displayName": "agent", "type": "string", "description": "Expert agent identifier", "required": true },
        { "id": "task", "displayName": "task", "type": "string", "description": "Task description", "required": true },
        { "id": "context", "displayName": "context", "type": "string", "description": "Conversation context", "required": false }
      ]
    }
  }
}
```

#### 3. `setup.sh` — Sub-Agent Runner importieren

```bash
# In IMPORT_ORDER hinzufügen (vor dem Haupt-Agent):
IMPORT_ORDER=(
  "reminder-factory"
  "reminder-runner"
  "mcp-client"
  "mcp-library-manager"
  "credential-form"
  "project-manager"
  "sub-agent-runner"         # NEU
  "memory-consolidation"
  "n8n-claw-agent"
)

# Neue Placeholder-Ersetzung:
REPLACE_SUB_AGENT_RUNNER_ID → tatsächliche Workflow-ID nach Import
```

#### 4. `agents`-Tabelle — Persona-Einträge + Expert-List

```sql
-- Persona-Definitionen (je eine pro Expert)
INSERT INTO agents (key, content) VALUES ('persona:instagram-curator', '...');
INSERT INTO agents (key, content) VALUES ('persona:content-creator', '...');
INSERT INTO agents (key, content) VALUES ('persona:support-responder', '...');

-- Expert-Liste für den Haupt-Agent (damit er weiß, wen er delegieren kann)
INSERT INTO agents (key, content) VALUES ('expert_agents', '
## Expert Agents
Du kannst spezialisierte Aufgaben an Expert Agents delegieren.
Nutze das expert_agent Tool wenn eine Aufgabe von Spezialwissen profitiert.

Verfügbare Experten:
- instagram-curator: Instagram Content, Visual Storytelling, Community-Building
- content-creator: Multi-Format Content, Editorial Planning, Brand Narrative
- support-responder: Kundenservice, Beschwerdemanagement, Feedback-Verarbeitung
');
```

### Sub-Agent Tools: Welche bekommt er?

| Option | Tools im Sub-Agent | Komplexität | Wann sinnvoll |
|--------|-------------------|-------------|---------------|
| **Basis** | HTTP Request, Web Search | Niedrig | MVP — Sub-Agent kann recherchieren und APIs aufrufen |
| **+ MCP Client** | Basis + alle installierten MCP Skills | Mittel | Sub-Agent kann z.B. Instagram-API direkt nutzen |
| **Persona-definiert** | Tools pro Persona konfigurierbar | Hoch | Volle Flexibilität, aber deutlich mehr Aufwand |

**Empfehlung:** Start mit Basis-Tools (HTTP + Web Search). MCP Client als Phase 2 hinzufügen — dann kann z.B. der Instagram-Curator direkt Instagram-Posts erstellen.

### Design-Entscheidungen

1. **Konversations-Kontext: Ja.** Der Sub-Agent bekommt die bisherige Chat-History als `context`-Parameter übergeben. Der Haupt-Agent entscheidet selbst, wie viel Kontext er weitergibt (typischerweise die letzten 5-10 Nachrichten). Trade-off: Mehr Tokens pro Sub-Agent-Call, aber bessere Ergebnisse.

2. **Antwort-Integration: Haupt-Agent formuliert um.** Da der Sub-Agent als `toolWorkflow`-Tool aufgerufen wird, ist sein Output ein Tool-Result. Der Haupt-Agent verarbeitet dieses Ergebnis und formuliert seine eigene Antwort daraus — automatisch, kein Extra-Code nötig. Der User hört immer die Stimme des Haupt-Agents.

3. **Ein Workflow für alle Personas.** Kein Workflow-Building via Claude Code, keine dynamische Workflow-Erstellung. Der Sub-Agent Runner ist ein fester Bestandteil von n8n-claw, wird bei der Installation ausgeliefert und arbeitet mit beliebigen Persona-Definitionen aus der Datenbank.

### Vorteile

- **Haupt-Persönlichkeit bleibt unverändert** — kein Persona-Switching, kein Identitätsbruch
- **Token-sparend im Haupt-Prompt** — Persona-Content nur geladen wenn gebraucht (im Sub-Agent), nicht dauerhaft im System-Prompt des Haupt-Agents
- **Autonome Iteration** — Sub-Agent kann 5+ Tool-Calls machen ohne den Haupt-Agent zu blockieren
- **Modell-Flexibilität** — Sub-Agent könnte günstigeres Modell nutzen (z.B. Haiku für einfache Aufgaben)
- **Ein Workflow für alle** — neue Persona = nur DB INSERT, kein neuer Workflow
- **Out-of-the-box** — Sub-Agent Runner wird mit n8n-claw ausgeliefert, keine manuelle Einrichtung
- **Kombinierbar mit Option B** — Personas über Template-System installierbar, Ausführung über Sub-Agent Runner

### Nachteile

- **Extra API-Kosten** — Jeder Sub-Agent-Aufruf ist ein separater Claude-API-Call (Input + Output Tokens)
- **Latenz** — +3-10 Sekunden pro Delegation (Sub-Agent muss eigenen AI-Loop durchlaufen)
- **Credential-Sharing** — Sub-Agent Runner braucht eigene Referenz auf die Anthropic-API-Credential
- **Kontext-Isolation** — Sub-Agent hat keinen direkten Zugang zur Conversation History; Haupt-Agent muss relevanten Kontext explizit als Parameter übergeben
- **Debugging komplexer** — Fehler im Sub-Agent sind schwerer nachzuvollziehen (verschachtelte Workflow-Ausführungen)
- **Neuer Workflow** — Sub-Agent Runner muss erstellt, getestet und in setup.sh integriert werden

### Kombination B + C: Die stärkste Variante

Option B (Persona-Templates) + Option C (Sub-Agent Runner) ergänzen sich ideal:

| Komponente | Verantwortung |
|------------|--------------|
| **Option B: Library Manager** | Personas installieren/entfernen (CDN → agents-Tabelle) |
| **Option C: Sub-Agent Runner** | Personas ausführen (agents-Tabelle → AI Agent) |

**Flow:**
1. User: "Installiere die Instagram Curator Persona"
2. Library Manager lädt persona.json von CDN → INSERT INTO agents
3. User: "Erstelle einen Instagram-Post über die Bergsaison"
4. Haupt-Agent delegiert an expert_agent(agent: "instagram-curator", task: "...")
5. Sub-Agent Runner lädt Persona aus agents-Tabelle → führt aus → gibt Ergebnis zurück
6. Haupt-Agent formuliert Antwort in seinem eigenen Ton

---

## 8. Vergleich aller drei Optionen

| Aspekt | A: `/persona`-Switching | B: Persona-Templates | C: Sub-Agent Runner |
|--------|------------------------|---------------------|---------------------|
| **Haupt-Persona bleibt?** | Nein (wird gewechselt) | Nein (wird gewechselt) | **Ja (unverändert)** |
| **Schema-Änderungen** | Keine | Keine | Keine |
| **Neue Workflows** | 0 | 0 | 1 (Sub-Agent Runner) |
| **Aufwand** | ~1-2h | ~4-6h | ~3-5h |
| **Extra API-Kosten** | Nein | Nein | Ja (pro Delegation) |
| **Latenz pro Nutzung** | 0s extra | 0s extra | +3-10s pro Delegation |
| **Token-Budget Haupt-Prompt** | Erhöht (~+25%) | Erhöht (~+25%) | **Unverändert** |
| **Eigene Tool-Sets möglich** | Nein | Nein | **Ja** |
| **Autonome Iteration** | Nein | Nein | **Ja (5+ Schritte)** |
| **Modell-Flexibilität** | Nein | Nein | **Ja (Haiku/Sonnet/Opus)** |
| **Out-of-the-box** | Ja | Ja (nach Install) | **Ja (ships mit n8n-claw)** |
| **Neue Persona hinzufügen** | DB INSERT | CDN + Install | **DB INSERT** |
| **Distribuierbar** | Nein | Ja | Persona: Ja (via B) |
| **Community-fähig** | Nein | Ja | Ja (via B) |
| **Uninstall** | SQL DELETE | Agent-Kommando | SQL DELETE (oder via B) |
| **Skalierbarkeit** | ~10 Personas | ~50+ Personas | ~50+ (mit B) |
| **Kombinierbar** | A ⊂ B | B + C ideal | **C + B ideal** |

### Kernunterschiede

**Option A** = "Persona-Switching als Feature" — ändert wer der Agent ist. Minimal, schnell, lokal.

**Option B** = "Persona-Switching als Plattform" — ändert wer der Agent ist, aber distribuierbar und installierbar.

**Option C** = "Persona-Delegation als Architektur" — der Agent bleibt er selbst, delegiert aber Spezialist-Aufgaben. Architektonisch sauberer, aber mit API-Kosten.

**A und B** ändern die Identität des Agents (der Agent "wird" zum Instagram-Curator).
**C** erhält die Identität (der Agent "fragt" den Instagram-Curator um Hilfe).

---

## 9. Empfehlung & Roadmap

### Empfehlung: C als Ziel-Architektur, mit B als Distribution

Option C (Sub-Agent Runner) ist die architektonisch sauberste Lösung:
- Die Haupt-Persönlichkeit bleibt erhalten (kein Identitätsbruch)
- Token-Budget des Haupt-Prompts wird nicht aufgebläht
- Sub-Agents können autonom iterieren
- Ein Workflow für alle Personas (kein Workflow-Sprawl)

Option B (Templates-Repo) ist der ideale Distributions-Mechanismus für Personas.

Zusammen: **B + C** = Personas über Library Manager installieren, über Sub-Agent Runner ausführen.

Option A bleibt als leichtgewichtige Alternative: Wenn man die Agent-Persönlichkeit global ändern will (statt zu delegieren), ist `/persona`-Switching der einfachste Weg. A und C schließen sich nicht gegenseitig aus.

### Roadmap

#### Phase 1: Sub-Agent Runner MVP (Option C)
- `sub-agent-runner.json` Workflow erstellen (Trigger → Postgres → Code → AI Agent → Return)
- `expert_agent` toolWorkflow-Node im Haupt-Agent hinzufügen
- 2-3 Persona-Einträge in `agents`-Tabelle seeden (instagram-curator, content-creator)
- `expert_agents`-Eintrag in `agents` für Persona-Discovery
- `setup.sh` erweitern (Import + ID-Patching)
- Testen via Telegram: "Erstelle einen Instagram-Post" → Delegation → Ergebnis

#### Phase 2: Personas testen & iterieren
- 2-4 Wochen im Alltag nutzen
- Persona-Texte verfeinern (Token-Budget, Ton, Nützlichkeit)
- Sub-Agent Tools evaluieren: Reicht HTTP + Web Search? MCP Client hinzufügen?
- Bewerten: Delegiert der Haupt-Agent sinnvoll? Sind die Ergebnisse gut?

#### Phase 3: Persona-Templates (Option B)
- Library Manager um `type: "persona"` erweitern
- 3-5 Personas als Templates ins n8n-claw-templates Repo
- Install/Uninstall über "Installiere Instagram Curator" / "Entferne Instagram Curator"

#### Phase 4: Optional — `/persona`-Switching (Option A)
- Nur wenn globales Persona-Switching zusätzlich zu Delegation gewünscht ist
- Build System Prompt erweitern + `/persona`-Kommando
- Sinnvoll für Szenarien wo die Grund-Persönlichkeit dauerhaft geändert werden soll

#### Phase 5: dmo-claw portieren
- Sub-Agent Runner + Rollen-System kombinieren
- Tourism-spezifische Personas erstellen
- Petra bleibt Petra — delegiert aber an Spezialisten

---

## 10. dmo-claw: Personas, Rollen & Sub-Agents

### Aktuelles Rollen-System

dmo-claw hat ein rollenbasiertes System (`roleContextMap` im Build System Prompt Code-Node):

```javascript
const roleContextMap = {
  marketing:         'Rolle: Marketing\nBevorzugte Tools: create_instagram_post, schedule_post...',
  member_relations:  'Rolle: Mitgliederbetreuung\nBevorzugte Tools: review_tools...',
  admin:             'Rolle: Admin\nVollzugriff auf alle Tools...',
  readonly:          'Rolle: Lesezugriff\nEingeschränkter Zugang...'
};
```

### Drei orthogonale Dimensionen (mit Sub-Agents)

| Dimension | Bestimmt durch | Kontrolliert | Beispiel |
|-----------|----------------|--------------|----------|
| **Rolle** | `dmo_users.role` (Admin-zugewiesen) | Tool-Zugriff, Daten-Sichtbarkeit | Marketing-User sieht nur Marketing-Tools |
| **Haupt-Persona** | `soul`-Tabelle (Setup) | Grundton, Identität | "Petra" — freundlich, kollegial |
| **Sub-Agent** | `agents`-Tabelle (Persona-Einträge) | Spezialisiertes Wissen | Instagram-Curator, Analytics-Reporter |

**Konkretes Szenario (dmo-claw mit Option C):**
1. Marketing-User Lisa schreibt an Petra (Haupt-Persona, dmo-claw Agent)
2. Lisa: "Erstelle einen Instagram-Post für das Bergfrühlingsfest"
3. Petra delegiert intern an `expert_agent(agent: "instagram-tourism", task: "...")`
4. Sub-Agent (Instagram-Tourism-Persona) erarbeitet Post mit:
   - Alpine Bildsprache
   - Content-Mix (1/3 Event, 1/3 Natur, 1/3 Community)
   - Saison-spezifische Hashtags
   - Posting-Zeitempfehlung
5. Petra empfängt das Ergebnis und antwortet Lisa in ihrem eigenen Ton:
   "Hier ist mein Vorschlag für den Instagram-Post zum Bergfrühlingsfest! 🏔️ ..."

**Lisa merkt keinen Unterschied** — sie spricht immer mit Petra. Aber Petras Instagram-Vorschläge sind deutlich besser, weil ein Spezialist-Prompt dahinter arbeitet.

### Tourism-spezifische Personas (Ideen)

| Persona | Basiert auf | Tourism-Adaption |
|---------|------------|------------------|
| `persona:instagram-tourism` | Instagram Curator | Tourismus-Content, alpine Ästhetik, Saison-Planung |
| `persona:review-manager` | Support Responder | Google-Bewertungen beantworten, Gäste-Feedback |
| `persona:briefing-analyst` | Analytics Reporter | Morning Briefing optimieren, Tourismus-KPIs |
| `persona:event-promoter` | Content Creator | Veranstaltungs-Marketing, Saison-Highlights |

---

## 11. Token-Budget & Prompt-Größe

### Ist-Zustand

n8n-claws aktueller System-Prompt (soul + agents + user + conversation):

| Teil | Geschätzte Größe |
|------|------------------|
| Soul (6 Keys) | ~800 chars |
| Agents (5 Keys) | ~2.200 chars |
| User Profile | ~200 chars |
| Active Projects | ~100 chars |
| Conversation History (20 msgs) | ~3.000 chars |
| **Gesamt** | **~6.300 chars (~1.800 Tokens)** |

### Persona-Zusatz

Eine ungekürzte agency-agents Persona:

| Agent | Rohgröße | Gekürzt (empfohlen) |
|-------|----------|---------------------|
| Instagram Curator | ~2.500 chars | ~800 chars |
| Content Creator | ~2.800 chars | ~900 chars |
| Support Responder | ~4.200 chars | ~700 chars |
| Analytics Reporter | ~3.100 chars | ~600 chars |

### Empfehlung

- **Personas auf 500-1.000 chars kürzen** (Kernessenz: Persönlichkeit + Regeln + Metriken)
- Dev-spezifische Teile (Python-Code, SQL-Templates) komplett entfernen
- Technical Deliverables auf 2-3 Bullet Points reduzieren
- Das erhöht den System-Prompt um ~15-25% — akzeptabel

### Beispiel: Instagram Curator gekürzt (800 chars)

```
# Instagram Curator 📸

Du bist eine visuelle Storytelling-Expertin. Du denkst in Ästhetik,
Engagement und Community-Building.

## Kommunikation
- Visuell denken: Bildsprache und Ästhetik immer mitdenken
- Trends: Aktuelle Instagram-Features und Formate einbeziehen
- Messbar: Engagement-Raten und Completion-Rates als Orientierung
- Community: Interaktion und UGC über einseitiges Broadcasting

## Regeln
- Content-Mix: 1/3 Marke, 1/3 Wissen/Inspiration, 1/3 Community
- Jeder Post: klarer CTA
- Brand-Konsistenz über alle Formate

## Workflow
1. Ästhetik → 2. Strategie → 3. Community → 4. Optimierung

## Ziele
Engagement > 3.5% · Story Completion > 80% · Follower +5%/Monat
```

### Token-Vergleich: Option A/B vs. Option C

| Szenario | Option A/B | Option C |
|----------|-----------|----------|
| **Haupt-Prompt (jede Nachricht)** | ~2.300 Tokens (+500 Persona) | ~1.800 Tokens (unverändert) |
| **Delegation (wenn genutzt)** | 0 extra | ~1.500 Tokens (Sub-Agent Prompt + History) |
| **10 Nachrichten, 2 Delegationen** | 23.000 Tokens | 18.000 + 3.000 = 21.000 Tokens |

Bei häufiger Delegation ist Option C sogar **günstiger** als dauerhaft aufgeblähte Haupt-Prompts — die Persona-Tokens fallen nur bei tatsächlicher Nutzung an.

---

## 12. Offene Fragen

Diese Fragen sollten vor oder während der Implementierung geklärt werden:

1. **Welche Basis-Tools bekommt der Sub-Agent?**
   - Minimum: HTTP Request + Web Search
   - Ideal: + MCP Client (Zugriff auf installierte Skills)
   - Entscheidung beeinflusst, wie autonom der Sub-Agent arbeiten kann

2. **Persona-Discovery: Wie erfährt der Haupt-Agent, welche Experten verfügbar sind?**
   - Option 1: Statischer `expert_agents`-Eintrag in der `agents`-Tabelle (manuell pflegen)
   - Option 2: Dynamisch aus DB laden (alle `persona:*`-Einträge auflisten)
   - Option 3: Via `updateAgentConfig()` automatisch aktualisieren (wie bei MCP Skills)

3. **Persona-Qualität: Wie viel Adaption brauchen agency-agents Prompts?**
   - Marketing-Agents (Instagram Curator, Content Creator): ~95% direkt nutzbar
   - Technische Agents (Support Responder, Analytics Reporter): ~50% nutzbar, Rest ist Dev-Code
   - Alle Personas sollten auf 500-1.000 chars gekürzt werden

4. **Soll der Sub-Agent auch für dmo-claw rollen-gefilterte Tools bekommen?**
   - Wenn ein Marketing-User den Sub-Agent nutzt: Soll der Sub-Agent nur Marketing-Tools haben?
   - Oder hat der Sub-Agent immer alle Tools (da der Haupt-Agent bereits filtert)?

---

## Anhang: Relevante Dateien

| Datei | Relevanter Inhalt | Zu ändern? |
|-------|-------------------|------------|
| `workflows/n8n-claw-agent.json` (Zeile 934) | Build System Prompt Code-Node | Ja (A: Persona-Lookup, C: expert_agents Listing) |
| `workflows/n8n-claw-agent.json` (Tool-Nodes) | toolWorkflow-Anbindungen | Ja (C: neuer Expert Agent toolWorkflow) |
| `workflows/mcp-library-manager.json` | Install/Remove Flow, updateAgentConfig() | Nur Option B |
| `workflows/sub-agent-runner.json` | **NEU** — Sub-Agent Runner Workflow | Nur Option C |
| `supabase/migrations/001_schema.sql` | soul, agents, mcp_registry, user_profiles Schema | Nein (keine Änderung nötig) |
| `setup.sh` (Zeile 1096-1290) | Soul/Agents Seeding, IMPORT_ORDER | Ja (Persona-Seeding, Runner-Import) |
| `supabase/migrations/002_seed.sql` | Seed-Daten | Ja (Persona-Einträge) |
| `../n8n-claw-templates/templates/index.json` | Template-Katalog | Nur Option B |
| `../dmo-claw/workflows/dmo-claw.json` | roleContextMap, Rollen-Filterung | Phase 5 |
