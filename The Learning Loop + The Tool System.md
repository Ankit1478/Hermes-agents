# The Learning Loop + The Tool System

Let me explain both — they work together tightly.

---

# Part 1: The Learning Loop

## What it is
The system that lets Hermes **write and update its own skills** automatically.

## The Flow

```
1. User gives task
2. AI plans and executes (using tools)
3. Task completes
4. Agent asks itself: "Was this complex?"
   └─ Counts: did I use 5+ tool calls?
5. Agent asks: "Does a skill for this already exist?"

   ┌──────────────────┬─────────────────┐
   NO skill exists    Skill exists      Skill failed/wrong
   ↓                  ↓                 ↓
   CREATE new         LEAVE alone       PATCH it
   SKILL.md                             (update SKILL.md)

6. Write markdown file to ~/.hermes/skills/
7. Skill available from next chat
```

---

## Decision Logic

```python
# Simplified pseudo-code
def after_task_completes(task, tool_calls):
    
    if len(tool_calls) < 5:
        return  # Too simple, not worth saving
    
    existing_skill = find_matching_skill(task)
    
    if not existing_skill:
        # CREATE
        new_skill = llm.generate_skill_doc(task, tool_calls)
        save_to_disk(new_skill)
    
    elif skill_failed_or_incomplete:
        # PATCH
        updated = llm.patch_skill(existing_skill, what_went_wrong)
        save_to_disk(updated)
    
    else:
        # LEAVE ALONE
        pass
```

---

## The Three Outcomes

### CREATE
First time solving a complex task → generate new SKILL.md

### PATCH
Used a skill, something failed → fix the broken step

### SKIP
Task was simple OR skill worked perfectly → do nothing

---

## Real Example

**Task:** "Deploy my site"

**Execution:** 7 tool calls (SSH, git pull, npm install fails, fix Node version, retry, build, restart nginx)

**After completion:**
```
- Complex? YES (7 tool calls ≥ 5)
- Skill exists? NO
- Create skill!

Agent writes:
─── deploy-my-site/SKILL.md ───
Procedure:
1. SSH to server
2. Check Node v20+ (IMPORTANT — use nvm)
3. git pull
4. npm install
5. npm run build
6. Restart nginx

Pitfalls:
- Node v18 breaks npm install
────────────────────────────────
```

Next deploy → loads this skill → done in 3 tool calls instead of 7.

---

# Part 2: The Tool System

## What it is
The system that lets the AI **actually do things** (run commands, read files, search web, etc.) instead of just talking.

## The Flow

```
1. AI decides: "I need to run a command"
2. AI outputs a tool call (structured format)
3. Hermes intercepts the tool call
4. Hermes runs the actual function
5. Result returned to AI
6. AI continues with the result
```

---

## Architecture

```
┌─────────────────────────────────────┐
│         AI MODEL                     │
│  "I'll call bash tool with 'ls -la'" │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│      TOOL REGISTRY                   │
│  Look up 'bash' → found handler      │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│      TOOL HANDLER                    │
│  Runs on your chosen backend:        │
│  • local  • docker  • ssh            │
│  • modal  • daytona • singularity    │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│      RESULT                          │
│  Output sent back to AI              │
└──────────────┬──────────────────────┘
               │
               ▼
         AI continues...
```

---

## How Tools Are Organized

Hermes has **47 tools grouped into 19 toolsets:**

| Toolset | Tools inside |
|---|---|
| **Terminal** | bash, python, execute_code |
| **File** | read_file, write_file, patch, list_dir |
| **Web** | web_search, web_fetch, web_extract |
| **Browser** | navigate, click, screenshot, fill_form |
| **Vision** | analyze_image, generate_image |
| **Memory** | memory_add, memory_replace, memory_list |
| **Skills** | skill_view, skill_create, skill_manage |
| **Delegation** | spawn_subagent, rpc_call |
| **Planning** | create_task, schedule_cron |
| **MCP** | dynamic tools from external servers |

---

## Tool Registration (smart trick)

```python
# Each tool self-registers when imported
@register_tool
def bash(command: str) -> str:
    """Run a shell command"""
    return subprocess.run(command, shell=True).stdout

# At startup:
tool_registry = {
    "bash": bash_handler,
    "read_file": read_file_handler,
    "web_search": web_search_handler,
    ...
}
```

When AI wants to use a tool:
```python
# AI outputs
{"tool": "bash", "args": {"command": "ls -la"}}

# Hermes dispatches
result = tool_registry["bash"](**args)

# Return to AI
```

---

## Parallel Execution (speed trick)

If AI calls **multiple independent tools**, Hermes runs them **in parallel**:

```python
# AI calls 3 tools at once:
- read_file("a.py")
- read_file("b.py")  
- web_search("fastapi docs")

# Hermes runs all 3 simultaneously (ThreadPool)
# Total time = slowest one (not sum of all)
```

**Result:** Multi-step tasks complete 2-3x faster.

---

## Execution Backends (security trick)

Same tool can run in different **environments**:

| Backend | Use case |
|---|---|
| **local** | Run on your machine |
| **docker** | Sandboxed container (safer) |
| **ssh** | Run on remote server |
| **modal** | Serverless (sleeps when idle) |
| **daytona** | Cloud dev environment |
| **singularity** | HPC / research clusters |

Switch with one config change — tool code stays the same.

---

# How They Work Together

Here's the magic — the learning loop and tool system **feed each other**:

```
1. User gives task
2. AI uses TOOLS to execute (tool system)
3. Tool results come back
4. Task completes
5. Learning loop checks: was it complex?
6. If yes → CREATE SKILL describing which tools to use
7. Next time, skill tells AI exactly which tools to call
8. AI uses TOOLS again (faster now, follows recipe)
9. Loop repeats, skills get smarter
```

**Tools = the hands.**
**Skills = the memory of which hands to use and when.**
**Learning loop = writes the instructions for future-you.**

---

## Real Example Combining Both

**Turn 1 (first time):**
```
User: "Set up a new React project"

AI thinks → calls tools:
  bash("npx create-react-app myapp")       [tool]
  bash("cd myapp && npm install tailwind") [tool]
  write_file("tailwind.config.js", ...)    [tool]
  bash("npm run dev")                      [tool]
  web_search("tailwind setup errors")      [tool]
  patch("src/index.css", ...)              [tool]

6 tool calls → complex!

Learning loop creates:
  skills/setup-react-tailwind/SKILL.md
```

**Turn 2 (next week):**
```
User: "Set up another React project"

AI sees skill name in index → loads it → follows recipe:
  Runs 4 tool calls directly (skips trial-and-error)
  Done in 30 seconds instead of 5 minutes
```

---

## Core Insight

| System | Role |
|---|---|
| **Tools** | Let AI *do* things |
| **Skills** | Remember *how* to use tools |
| **Learning loop** | *Writes* the skills automatically |

Without tools → AI can only talk.
Without skills → AI repeats mistakes.
Without learning loop → skills stay static.

**All three together = agent that acts, remembers, and improves.**

---
