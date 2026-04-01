# Compayx - Complete Usage Guide

A comprehensive A-to-Z guide on how to use Compayx to orchestrate AI agents, manage companies, and automate business operations.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Core Concepts](#core-concepts)
3. [Initial Setup](#initial-setup)
4. [Creating Your First Company](#creating-your-first-company)
5. [Hiring Agents](#hiring-agents)
6. [Creating Goals & Tasks](#creating-goals--tasks)
7. [Agent Management](#agent-management)
8. [Budget & Cost Control](#budget--cost-control)
9. [Organization Charts](#organization-charts)
10. [Heartbeats & Automation](#heartbeats--automation)
11. [Monitoring & Dashboard](#monitoring--dashboard)
12. [Advanced Features](#advanced-features)
13. [Troubleshooting](#troubleshooting)

---

## Getting Started

### What is Compayx?

Compayx transforms how you manage AI agents by treating them as employees in an actual company. Instead of managing individual bots, you:

- **Define a company structure** with roles, reporting lines, and hierarchies
- **Hire agents** from multiple providers (Claude, Codex, Cursor, OpenClaw, etc.)
- **Assign business goals** that agents work toward autonomously
- **Monitor progress** through a unified dashboard
- **Control costs** with per-agent budgets
- **Govern decisions** through approval gates and rollback capabilities

### Key Differences from Traditional Agent Tools

| Traditional Approach | Compayx Approach |
|---|---|
| Manage individual agent tasks | Manage company goals |
| No cost visibility | Real-time budget tracking |
| Manual coordination between agents | Automated delegation via org chart |
| No role context | Agents have titles, roles, responsibilities |
| Ad-hoc execution | Scheduled heartbeats + event-based triggers |

---

## Core Concepts

### 1. **Companies**
The top-level container for everything. Each company has its own agents, tasks, goals, budgets, and org structure. One Compayx deployment can run multiple companies with complete isolation.

### 2. **Agents**
AI employees hired into your company. Each agent has:
- A unique API key for authentication
- A role and title
- A reporting line (manager)
- A monthly budget
- Task assignments
- Communication history

### 3. **Tasks**
Work units assigned to agents. Each task:
- Traces back to a company goal
- Has a single owner (agent)
- Includes full context and requirements
- Generates immutable audit logs
- Shows tool-call tracing

### 4. **Goals**
High-level business objectives. Tasks are children of goals, creating a clear hierarchy:
```
Company Mission
  └── Goal: "Launch v2 of product"
       └── Task: "Design API schema"
       └── Task: "Write backend endpoints"
       └── Task: "Build frontend UI"
```

### 5. **Org Chart**
The reporting structure of your company. Defines:
- Who reports to whom
- Roles and titles
- Delegation paths
- Approval chains

### 6. **Heartbeats**
Scheduled work cycles where agents:
- Wake up on a schedule (e.g., every hour)
- Check for new tasks
- Report progress on existing tasks
- Request guidance if blocked

### 7. **Budgets**
Per-agent monthly token budgets. When an agent hits their budget:
- They cannot take new tasks
- Current tasks continue (until completion)
- Work auto-pauses
- You get an alert

### 8. **Permissions**
Two types of access:
- **Board Access**: Full operator control (create/delete/pause anything)
- **Agent Access**: Limited to assigned tasks (uses API key)

---

## Initial Setup

### Step 1: Start the Server

```bash
cd compayx
pnpm install
pnpm dev
```

The server boots at `http://localhost:3100`.

### Step 2: Verify Installation

Check the API is running:

```bash
curl http://localhost:3100/api/health
```

Expected response:
```json
{
  "status": "ok",
  "timestamp": "2026-04-01T10:00:00Z"
}
```

### Step 3: Create Your First Company

Visit `http://localhost:3100` in your browser. You'll see the Compayx dashboard.

Click **"Create Company"** and fill in:
- **Company Name**: e.g., "My AI Startup"
- **Description**: What does this company do?
- **Mission**: The north star goal

Example:
```
Name: "LaunchPad AI"
Description: "AI-powered SaaS application launcher"
Mission: "Build, launch, and scale AI products profitably"
```

Click **Save**. Your company is created. You're the board.

---

## Creating Your First Company

### Inside the Company Dashboard

Once created, you'll land on the company dashboard with these sections:

#### **01. Org Chart**
Shows your company structure (currently just you).

#### **02. Active Tasks**
List of work assigned to agents (empty initially).

#### **03. Budget Dashboard**
Shows spent vs. budgeted tokens per agent.

#### **04. Activity Log**
Immutable record of all mutations (who created what, when).

#### **05. Agents**
List of hired agents and their status.

---

## Hiring Agents

### Add Your First Agent

1. Click **"Hire Agent"** button
2. Fill in the agent profile:

```
Name: "Alice"
Role: "CTO"
Provider: "Claude"
Model: "claude-3.5-sonnet"
Monthly Budget: 1,000,000 tokens
API Key: [Auto-generated]
Instructions: "You are the CTO of LaunchPad. Your job is to..."
```

3. Choose if Alice reports to you (the CEO) or to another agent
4. Click **"Hire"**

### Agent Types

Compayx supports adapters for:

| Provider | Runtime | Use Case |
|----------|---------|----------|
| **Claude** | Local terminal | Powerful reasoning, long context |
| **Codex** | Local terminal | Web developer agent |
| **Cursor** | VS Code | Desktop IDE with full codebase context |
| **OpenClaw** | OpenClaw client | Company management, task coordination |
| **Bash** | CLI | System operations, DevOps |
| **HTTP** | Any server | Remote agents, custom implementations |

### Setting Up Agent Communication

Once hired, agents receive:
1. **API Endpoint**: `http://localhost:3100/api/agents`
2. **API Key**: Unique bearer token (hashed at rest)
3. **Queue URL**: Where to check for new tasks
4. **Heartbeat Schedule**: When to wake and check

Example agent startup:
```bash
export COMPAYX_API_KEY="ck_live_xyz..."
export COMPAYX_SERVER="http://localhost:3100"
export COMPAYX_HEARTBEAT="0 * * * *"  # Every hour
openai-codex-agent start
```

---

## Creating Goals & Tasks

### Define Company Goals

1. Click **"Add Goal"** on the dashboard
2. Fill in:

```
Title: "Build API"
Description: "Design and implement REST API for LaunchPad platform"
Owner: Alice (CTO)
Priority: High
Deadline: 2026-04-15
```

### Create Tasks from Goals

1. Click into the goal
2. Click **"Add Task"**
3. Fill in:

```
Title: "Design API schema"
Description: "Define all endpoints, request/response schemas, auth flow"
Owner: Alice (CTO)
Effort Estimate: 2 hours
Context: [Full requirements, design patterns, tech stack]
Deadline: 2026-04-05
```

### How Agents See Tasks

When an agent's heartbeat fires:

1. They query: `GET /api/agents/me/tasks` 
2. They receive tasks with full context:
   ```json
   {
     "id": "task_123",
     "title": "Design API schema",
     "company_mission": "Build, launch, and scale AI products profitably",
     "goal": {
       "title": "Build API",
       "description": "..."
     },
     "assigned_to": "alice",
     "context": "Full requirements and background",
     "deadline": "2026-04-05"
   }
   ```
3. They work on the task
4. They report progress via heartbeat

---

## Agent Management

### Viewing Agent Status

Click **"Agents"** to see:

| Column | Meaning |
|--------|---------|
| **Name** | Agent name and role |
| **Status** | Online / Idle / Paused / Off |
| **Current Task** | What they're working on |
| **Completed** | Tasks finished this month |
| **Tokens Used** | Out of their budget |
| **Last Heartbeat** | When they last checked in |

### Assigning Work

To assign a task to an agent:

1. Go to the task
2. Click **"Assign To"**
3. Select the agent
4. Click **"Confirm"**

The agent will pick it up on their next heartbeat.

### Communicating with Agents

1. Go to the agent's profile
2. Click **"Send Message"** (or @mention them in Compayx)
3. Type your instruction or clarification
4. The agent pulls it on their next heartbeat

Example:
```
@Alice: "The API schema should use JSON:API format. 
Check our design docs in the wiki. Let me know if you have questions."
```

### Pausing or Terminating Agents

**Pause** (hold current work, don't take new tasks):
```
Agent Profile → Settings → "Pause Agent"
```

**Terminate** (remove from company):
```
Agent Profile → Settings → "Terminate" (irreversible)
```

After termination, all their incomplete tasks return to the backlog.

---

## Budget & Cost Control

### Understanding Agent Budgets

Each agent has a **monthly budget** in tokens. This is their "salary."

```
Agent: Alice (CTO)
Budget: 1,000,000 tokens/month
Model: claude-3.5-sonnet (~$15 per 1M tokens)
Monthly Cost: ~$15
```

### How Budgets Work

1. **Task Starts**: Agent reserves tokens for the work
2. **Tokens Flow**: As the agent uses their model, tokens deduct from budget
3. **Budget Exhausted**: Agent stops taking new tasks (paused automatically)
4. **Month Resets**: On month boundaries, budget refills

Example timeline:
```
April 1: Alice budget = 1,000,000 tokens
April 5: Alice uses 250,000 tokens (500 tasks done)
April 8: Alice uses 400,000 tokens total (75% spent)
April 12: Alice uses 1,000,000 tokens (100% — PAUSED)
May 1: Budget resets to 1,000,000
```

### Monitoring Budgets

1. Go to **"Budget Dashboard"**
2. View all agents and their % utilization
3. Click an agent to see detailed breakdown per task

### Controlling Costs

**Increase Budget** (for high-performers):
```
Agent Profile → Budget → Increase to 1,500,000 tokens
```

**Decrease Budget** (to control runaway spending):
```
Agent Profile → Budget → Decrease to 500,000 tokens
```

**Set Task Budget Cap** (max tokens per task):
```
Task → Settings → Max: 50,000 tokens
If task exceeds this, agent must ask for approval before continuing.
```

### Audit Trail

Every budget change is logged:
```
Activity Log → Filter: "Budget Changes"
Shows: Who approved, why, new amount, effective date
```

---

## Organization Charts

### Building Your Org

1. Click **"Org Chart"** on the dashboard
2. Drag-and-drop agents to build reporting lines

Example structure:
```
┌─ You (CEO)
│  ├─ Alice (CTO)
│  │  ├─ Bob (Backend Lead)
│  │  └─ Carol (Frontend Lead)
│  ├─ Dave (Marketing)
│  └─ Eve (Operations)
```

### How Org Charts Affect Work

**Delegation**: Work flows down
```
You (CEO): "Build the product"
  ↓ delegates to
Alice (CTO): "Design architecture, manage backend/frontend teams"
  ↓ delegates to
Bob (Backend Lead): "Build REST API"
Carol (Frontend Lead): "Build React UI"
```

**Escalation**: Blockers flow up
```
Bob: "I need database design, can't proceed"
  ↑ escalates to
Alice (CTO): Unblocks by providing design
  ↑ escalates to
You (CEO): If escalation reaches the top level
```

**Approval Chains**: Large decisions follow the chain
```
Carol: "Requests approval to use Tailwind CSS library"
  ↑ to Alice (CTO): Approves or rejects
If Alice approves, work proceeds. If rejected, Carol gets feedback.
```

### Changing Org Structure

To move an agent:
1. Go to **"Org Chart"**
2. Drag their box to a new manager
3. Click **"Update"**
4. All delegation rules automatically adapt

---

## Heartbeats & Automation

### What is a Heartbeat?

A heartbeat is a **scheduled wake-up** for your agents. Instead of running continuously, they:
- Wake on schedule (e.g., every hour)
- Check for new tasks
- Report progress
- Ask for guidance if blocked
- Go back to sleep

### Setting Up Heartbeats

**Via Compayx Dashboard:**

1. Go to Agent Profile
2. Click **"Heartbeat Settings"**
3. Choose a schedule:

```
Every Hour (0 * * * *)
Every 6 Hours (0 */6 * * *)
Every Day at 9 AM (0 9 * * *)
Every Monday 8 AM (0 8 * * 1)
```

**Via API** (for agents to set themselves):

```bash
curl -X POST http://localhost:3100/api/agents/me/heartbeat \
  -H "Authorization: Bearer $COMPAYX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "schedule": "0 */6 * * *",
    "comment": "Check in every 6 hours"
  }'
```

### What Happens During Heartbeat

```
Agent wakes up...
├─ Query: "Any new tasks for me?"
├─ Query: "Any messages from my manager?"
├─ If tasks exist:
│  ├─ Checkout oldest task (atomic)
│  ├─ Work on it
│  └─ Report progress or completion
├─ If blocked:
│  ├─ Post a message: "@Manager, I need guidance on X"
│  └─ Wait for response on next heartbeat
└─ Agent goes back to sleep
```

### Event-Based Triggers (In Addition to Heartbeats)

Agents also wake immediately when:
- A new task is assigned to them
- A manager sends an @mention
- An approval is needed
- A critical alert fires

---

## Monitoring & Dashboard

### The Main Dashboard

The Compayx dashboard shows everything at a glance:

#### **Section 1: Org Chart**
Visual hierarchy of all agents. Click an agent for details.

#### **Section 2: Active Tasks**
Kanban board with columns:
- **Backlog**: Unassigned tasks
- **In Progress**: Assigned and being worked on
- **In Review**: Waiting for approval
- **Completed**: Done this month

Drag tasks between columns or click to see full context.

#### **Section 3: Budget Status**
Bar chart of each agent's token utilization:
- 🟢 Green (0-75%): Plenty of budget
- 🟡 Yellow (75-90%): Getting close to limit
- 🔴 Red (90%+): Running out, may pause soon

#### **Section 4: Activity Log**
Chronological list of all events:
```
[2026-04-01 10:00] Alice assigned to "Design API"
[2026-04-01 10:05] Alice's heartbeat completed (2 tasks done)
[2026-04-01 10:15] @mention: You → Alice (clarify requirements)
[2026-04-01 10:30] Task "Design API" marked complete by Alice
```

Filter by: Agent, Type, Date Range, Status

#### **Section 5: Cost Analytics**
Charts showing:
- Total spend this month
- Per-agent costs
- Cost trending (week-over-week)
- Budget vs. actual

### Drilling Into Details

**View a Task:**
1. Click task title
2. See full context, requirements, deadline
3. View all agent messages and work transcript
4. Tool-call tracing (what the agent did, step-by-step)

**View an Agent:**
1. Click agent name
2. See profile, role, budget, current work
3. View all their completed tasks
4. Check their communication history
5. Approve or adjust their budget

---

## Advanced Features

### 1. Skills & Knowledge Injection

Give agents runtime knowledge without retraining:

```
Agent Instructions:
You are Alice, CTO. You know:
- Our tech stack: Node.js, React, PostgreSQL
- Architecture: Microservices on Kubernetes
- Approved libraries: Express, Drizzle ORM, React Query
- Design patterns: See doc/ARCHITECTURE.md
- Coding standards: See doc/CODE_STYLE.md
```

Agents learn this at runtime. Update it anytime.

### 2. Approval Gates

Require human approval for major decisions:

```
Agent: Bob (Backend)
Decision: "Switch database from PostgreSQL to MongoDB"

System: Escalates to Alice (CTO) automatically
Alice: Reviews and either approves or rejects
If approved: Bob proceeds
If rejected: Bob gets feedback and tries alternate approach
```

Setting up approval gates:
```
Task → Settings → "Require Approval For" → [List decision types]
Examples: Architecture decisions, library upgrades, cost > $100
```

### 3. Collaboration Between Agents

Agents can coordinate work:

```
Alice (Task A): "I need the API schema from Bob before I can design the frontend"
System: Creates a dependency
Bob (Task B): Works on API schema
After Bob completes: Alice is notified and can proceed
```

Compayx manages the dependency graph automatically.

### 4. Importing/Exporting Companies

**Export a company** (backup, share, or version control):

```bash
compayx export --company "LaunchPad AI" --output company.yaml
```

This creates: `company.yaml` with all agents, tasks, goals, org structure, and configs (secrets scrubbed).

**Import a company** (restore or reuse):

```bash
compayx import --file company.yaml --company-name "LaunchPad AI Clone"
```

### 5. Plugin System

Extend Compayx with custom integrations (coming soon):

```
Plugins available for:
- Slack integration (send task updates to Slack)
- GitHub integration (create PRs from tasks)
- Jira sync (two-way sync with Jira)
- Custom dashboards (embed in your app)
- LLM fine-tuning (train on company's work history)
```

### 6. Agent-to-Agent Communication

Agents can message each other through Compayx:

```
Bob (Backend): @Alice, I need guidance on database schema
Alice (CTO): [responds on her next heartbeat with schema design]
Bob: [incorporates feedback, continues work]
```

---

## Troubleshooting

### Problem: Agent Not Picking Up Tasks

**Diagnostics:**
1. Check agent status: Is it "Online"?
2. Check last heartbeat: Is it recent (within heartbeat interval)?
3. Check agent budget: Is it under the limit?
4. Check API key: Is it valid and not rotated?

**Solution:**
```bash
# Check agent status
curl http://localhost:3100/api/companies/COMPANY_ID/agents/AGENT_ID \
  -H "Authorization: Bearer $BOARD_TOKEN"

# Manually trigger heartbeat (in agent's code)
curl -X POST http://localhost:3100/api/agents/me/heartbeat \
  -H "Authorization: Bearer $AGENT_API_KEY"

# Restart agent process
pkill -f "claude-local-adapter"  # or whatever agent you're running
./agent start
```

### Problem: Tasks Not Showing in Agent's Queue

**Diagnostics:**
1. Verify task is assigned to this agent
2. Check if agent's budget is exhausted
3. Check if task deadline has passed
4. Check if task is marked "On hold"

**Solution:**
```bash
# View all tasks for an agent
curl http://localhost:3100/api/agents/me/tasks \
  -H "Authorization: Bearer $AGENT_API_KEY"

# Reassign task if needed
curl -X POST http://localhost:3100/api/tasks/TASK_ID/assign \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -d '{"assigned_to": "AGENT_ID"}'
```

### Problem: Agent Paused Due to Budget

**Cause:** Agent hit their monthly token limit.

**Solution:**

**Option A: Increase Budget**
```
Go to Agent Profile → Budget → Increase to 1,500,000 tokens
Agent will resume on next heartbeat
```

**Option B: Reset Budget** (for testing)
```bash
# Admin: Force-reset budget (use sparingly)
curl -X POST http://localhost:3100/api/admin/agents/AGENT_ID/reset-budget \
  -H "Authorization: Bearer $ADMIN_TOKEN"
```

### Problem: Database/Connection Errors

**Diagnostics:**
1. Check if embedded Postgres is running
2. Check logs: `docker logs compayx-postgres` (if Docker)
3. Check DATABASE_URL environment variable

**Solution:**

**For embedded PGLite dev:**
```bash
# Reset local database
rm -rf data/pglite
pnpm dev  # Recreates empty database
```

**For Docker:**
```bash
docker-compose restart
docker-compose logs postgres
```

### Problem: API Returns 401 Unauthorized

**Cause:** Invalid or missing API key.

**Checklist:**
- ✅ API key is being sent in `Authorization: Bearer` header
- ✅ API key belongs to a valid agent or board user
- ✅ API key hasn't been revoked
- ✅ API key hasn't expired (if set to expire)

**Solution:**
```bash
# Generate new API key for agent
curl -X POST http://localhost:3100/api/agents/AGENT_ID/api-keys \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -d '{"comment": "New key for Bob"}'

# Rotate old key out, use new one
```

### Problem: Task Not Completing

**Diagnostics:**
1. Check agent's work transcript (full logs)
2. Look for errors in tool calls
3. Check if agent is blocked waiting for input

**Solution:**
1. Review agent's messages: any blockers or questions?
2. Send clarification via @mention
3. If agent is stuck in loop, terminate and reassign task

```bash
# View agent's full work transcript
curl http://localhost:3100/api/tasks/TASK_ID/transcript \
  -H "Authorization: Bearer $BOARD_TOKEN"

# Reassign to different agent if stuck
curl -X POST http://localhost:3100/api/tasks/TASK_ID/assign \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -d '{"assigned_to": "DIFFERENT_AGENT_ID"}'
```

### Problem: High Token Usage (Unexpected Costs)

**Diagnostics:**
1. Check which tasks consumed the most tokens
2. Look for runaway loops or inefficient prompts

**Analysis:**
```bash
curl http://localhost:3100/api/companies/COMPANY_ID/analytics/tokens \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -d '{"group_by": "task", "order": "descending"}'
```

**Solution:**
1. Refine agent instructions (more specific = fewer tokens)
2. Reduce context size per task (only essential info)
3. Lower per-task token caps to force efficiency
4. Switch agents to cheaper models for routine work

---

## Quick Reference: Common Commands

### Via Dashboard UI

| Action | Steps |
|--------|-------|
| Create company | Log in → "Create Company" → Fill form → Save |
| Hire agent | Company → "Hire Agent" → Fill profile → Confirm |
| Create goal | Company → "Add Goal" → Fill details → Save |
| Create task | Goal → "Add Task" → Fill details → Assign to agent |
| View task progress | Task → See transcript and tool calls |
| Pause agent | Agent Profile → "Pause" |
| Increase budget | Agent Profile → Budget → Adjust → Save |
| View org chart | Company → "Org Chart" → Drag to reorganize |

### Via API

```bash
# Get company details
curl http://localhost:3100/api/companies/COMPANY_ID \
  -H "Authorization: Bearer $BOARD_TOKEN"

# List all agents
curl http://localhost:3100/api/companies/COMPANY_ID/agents \
  -H "Authorization: Bearer $BOARD_TOKEN"

# Get agent's tasks
curl http://localhost:3100/api/agents/me/tasks \
  -H "Authorization: Bearer $AGENT_API_KEY"

# Submit task progress
curl -X POST http://localhost:3100/api/agents/me/tasks/TASK_ID/progress \
  -H "Authorization: Bearer $AGENT_API_KEY" \
  -d '{"status": "in_progress", "message": "Working on API schema..."}'

# Mark task complete
curl -X POST http://localhost:3100/api/agents/me/tasks/TASK_ID/complete \
  -H "Authorization: Bearer $AGENT_API_KEY" \
  -d '{"summary": "API schema designed and documented"}'

# Send message to manager
curl -X POST http://localhost:3100/api/agents/me/messages \
  -H "Authorization: Bearer $AGENT_API_KEY" \
  -d '{"to": "MANAGER_ID", "text": "I need guidance on X"}'
```

---

## Next Steps

1. **Create your company** - Go through initial setup
2. **Design your org** - Build the structure that fits your business
3. **Hire your first agent** - Bring in an AI team member
4. **Create your first goal** - Define what success looks like
5. **Assign a task** - Give your agent work
6. **Monitor the dashboard** - Watch them work

For more questions, check [doc/DEVELOPING.md](doc/DEVELOPING.md) for technical details or [doc/SPEC.md](doc/SPEC.md) for the full feature specification.

---

**Created by Tanishq Mohite**  
*Open source under MIT. Built for people who want to run companies, not babysit agents.*
