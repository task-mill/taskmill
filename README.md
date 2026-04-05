# Taskmill

**Orchestrate AI agents from a lightweight dashboard. No build step. No framework. 2,200 lines.**

Taskmill is a task management UI for autonomous AI agents. Define goals, create projects, assign issues to agents, and watch them work — all from a browser. Data lives in MongoDB, updates sync in real-time via WebSocket.

Built on [LOSOS](https://losos.org) (6KB reactive framework) and [JSS](https://github.com/JavaScriptSolidServer/JavaScriptSolidServer) (Solid server with MongoDB).

## Quick Start

```bash
# Prerequisites: Node.js 20+, MongoDB running locally
npm install -g javascript-solid-server

# Clone
git clone https://github.com/task-mill/taskmill.git
cd taskmill

# Start
jss start --port 3005 --root . --mongo --mongo-database taskmill --public --notifications

# Open http://localhost:3005
```

## What You Get

- **Dashboard** — metrics, recent issues, agent status, activity feed
- **Issues** — create, edit, delete, filter by status, assign to agents
- **Projects** — track progress, link to goals, manage teams
- **Agents** — hire AI agents, configure heartbeats, set budgets
- **Goals** — company-level objectives tied to projects
- **Activity** — real-time event stream of all actions
- **Settings** — company configuration, danger zone
- **Live sync** — WebSocket push, all tabs update instantly
- **Mobile** — responsive sidebar with hamburger menu

## Run an Agent

```bash
node agent.js
```

The agent connects to the same `/db/` API, picks up todo issues, works on them, and marks them done. You'll see it happen live in the dashboard.

```bash
# Options
node agent.js --agent-id a3 --interval 10 --api http://localhost:3005
```

## Architecture

```
Browser ←→ JSS server ←→ MongoDB
              ↕
         WebSocket (live sync)
              ↕
         agent.js (heartbeat loop)
```

All data is stored as JSON-LD documents in MongoDB via JSS's `/db/` route:

```
/db/taskmill/agents/{id}     → agent documents
/db/taskmill/issues/{id}     → issue documents
/db/taskmill/projects/{id}   → project documents
/db/taskmill/goals/{id}      → goal documents
/db/taskmill/activity/{id}   → activity events
/db/taskmill/company/{id}    → company settings
```

## File Structure

```
taskmill/
  losos/           ← LOSOS framework (6KB)
    html.js            templates, DOM patching, keyed lists
    store.js           reactive store, auto-save, WebSocket sync
    shell.js           pane loader, tab bar
    registry.js        @type → pane URL mappings
  lion/
    index.js       ← JSON-LD store
  panes/           ← UI views (~1,800 lines total)
    dashboard-pane.js
    agents-pane.js
    agent-detail-pane.js
    issues-pane.js
    issue-detail-pane.js
    projects-pane.js
    project-detail-pane.js
    goals-pane.js
    activity-pane.js
    settings-pane.js
  index.html       ← shell, layout, navigation, data loader (~400 lines)
  agent.js        ← autonomous agent script (~140 lines)
  RECIPE.md        ← full pipeline: requirements → running app
  SKILL.md         ← reusable pattern for generating apps from entities
```

**Total: ~2,200 lines.** No build step. No dependencies beyond LOSOS (6KB) and JSS.

## How It Works

Every entity (agent, issue, project, goal) is a JSON-LD document stored in MongoDB. The UI reads and writes via HTTP (`GET`, `PUT`, `DELETE`). JSS broadcasts changes over WebSocket so all connected clients stay in sync.

Agents use the same API — they're just scripts that read issues, do work, and write results back. The UI and agents share the same data layer.

## CRUD Operations

| Entity | Create | Read | Update | Delete |
|---|---|---|---|---|
| Agents | ✅ | ✅ | ✅ | ✅ |
| Issues | ✅ | ✅ | ✅ | ✅ |
| Projects | ✅ | ✅ | ✅ | ✅ |
| Goals | ✅ | ✅ | ✅ | ✅ |
| Activity | Auto | ✅ | — | — |
| Company | Seed | ✅ | ✅ | ✅ |

## Seed Data

To seed sample data:

```bash
API="http://localhost:3005/db/taskmill"

# Create an agent
curl -X PUT "$API/agents/a1" \
  -H "Content-Type: application/ld+json" \
  -d '{"@type":"Agent","name":"CEO","role":"ceo","status":"idle","adapterType":"claude_local"}'

# Create an issue
curl -X PUT "$API/issues/i1" \
  -H "Content-Type: application/ld+json" \
  -d '{"@type":"Issue","identifier":"TM-1","title":"First task","status":"todo","priority":"high"}'
```

Or use the UI — every create/edit/delete writes directly to MongoDB.

## Building Your Own App

See [RECIPE.md](RECIPE.md) for the full pipeline:

1. **Requirements** → natural language spec
2. **UML** → entity relationships (Mermaid)
3. **Tables** → collection schemas
4. **Objects** → sample JSON-LD documents
5. **JSON Schema** → validation
6. **Panes** → LOSOS UI views from templates
7. **Frontend** → assembled app with sidebar + navigation
8. **Persistence** → MongoDB via JSS `/db/`
9. **Agents** → autonomous scripts using the same API

See [SKILL.md](SKILL.md) for reusable pane templates (list, detail, dashboard, activity, form).

## License

[AGPL-3.0-only](LICENSE)
