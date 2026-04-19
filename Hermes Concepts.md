# The Remaining Hermes Concepts 

---

## 1. Context Compression

```
1. Chat keeps going, messages pile up
2. Token count reaches ~50% of model's limit
3. Hermes triggers compression
4. Old messages sent to CHEAP model
5. Cheap model writes a short summary
6. Old messages replaced with summary in context
7. Recent messages kept word-for-word (tail protection)
8. Chat continues, tokens low again
```

**Token optimization:**
- Cheap model for summary (not main model)
- Only triggers when needed (not every turn)
- Keeps last few turns exact (for continuity)

---

## 2. Subagent Delegation

```
1. Main agent gets complex task
2. Task has independent parts (research A, research B, research C)
3. Main agent spawns 3 subagents
4. Each subagent runs in ISOLATED conversation
5. Each has its own context, tools, terminal
6. All 3 run in parallel
7. Each returns final result (not all chat history)
8. Main agent combines results
9. Subagents destroyed after task
```

**Token optimization:**
- Subagent context never touches main context
- Only final answer returned (not full work)
- Parallel = faster completion

---

## 3. Gateway System

```
1. Start one Hermes process on your server
2. Gateway connects to Telegram, Discord, Slack, etc. (all at once)
3. Message arrives from any platform
4. Gateway routes to agent
5. Agent processes (same memory, same skills everywhere)
6. Reply sent back to original platform
7. Chat history unified across platforms
```

**Key point:** One agent, many platforms. Same brain everywhere.

---

## 4. Sandboxing & Security

```
1. AI wants to run a command
2. Security layer checks first:
   - Is this command dangerous? (rm, sudo, etc.)
   - If yes → ask user for approval
3. Credential filter scrubs API keys from context
4. Command runs in sandbox:
   - Docker container (isolated)
   - Read-only root filesystem
   - Dropped Linux capabilities
5. If command fails → rollback to snapshot
6. Logs everything for audit
```

**Layers of defense:**
- Approval for risky commands
- Credentials hidden from AI
- Sandboxed execution
- Auto-rollback on failure

---

## 5. Cron & Scheduling

```
1. User says: "Every Monday 8am, send me briefing"
2. Hermes creates cron job (stored as JSON)
3. Scheduler runs in background continuously
4. Time hits → job triggers
5. Fresh agent session starts
6. Agent runs task (reads calendar, emails, etc.)
7. Result delivered to chosen platform (Telegram, email)
8. Session ends, agent sleeps
```

**Token optimization:**
- Each cron run = fresh session (no baggage)
- Idle time = zero cost (agent sleeps)

---

## 6. MCP (Model Context Protocol)

```
1. External services expose MCP servers (GitHub, Notion, database)
2. Hermes connects as MCP client
3. External tools appear in Hermes like native tools
4. Agent calls them same way: mcp_github_create_issue(...)
5. Request sent to external server
6. Server handles it, returns result
7. Result goes back to agent
```

**Key point:** Infinite tool extension without changing Hermes code.

---

## 7. Profiles

```
1. Create multiple profiles: coder, writer, researcher
2. Each profile has its own:
   - Memory files
   - Skills folder
   - Session database
   - Config
3. Run them separately: hermes --profile coder
4. Each is a totally separate agent
5. Can share external memory (Honcho) if desired
6. Each maintains own personality
```

**Use case:** Specialized agents for different work domains.

---

## 8. Voice & Multimodal

```
1. Voice input:
   - User sends voice note (Telegram, Discord)
   - Whisper transcribes to text
   - Agent processes as normal message
2. Voice output:
   - Agent generates text response
   - TTS converts to audio
   - Sent back as voice note
3. Vision:
   - User sends image
   - Vision model analyzes
   - Description added to chat context
   - Agent responds using description
```

**Token optimization:**
- Vision uses separate cheap model
- Only description injected (not raw image)

---

## 9. Model Routing

```
1. Agent has multiple model options configured:
   - Main model (expensive, smart) → for reasoning
   - Auxiliary model (cheap, fast) → for side tasks
2. Routing decisions:
   - Complex reasoning → main model
   - Summarization → cheap model
   - Vision → vision model
   - Embedding → embedding model
3. Each task matched to right model
4. Costs optimized automatically
```

**Example:**
- Main chat: Claude Sonnet ($$$)
- Context compression: Gemini Flash ($)
- Embeddings: OpenAI ada ($)

---

## 10. Prompt Injection Defense

```
1. User sends content (email, web page, file)
2. Scanner runs first, looks for:
   - "Ignore previous instructions"
   - Hidden prompts in text
   - Suspicious command patterns
3. If injection detected → content flagged
4. Agent warned before processing
5. Clean content passed to agent
6. Agent treats flagged content as data, not instructions
```

**Key point:** Stops attackers from hijacking the agent.

---

## 11. Checkpoints & Rollback

```
1. Before destructive operation (delete, overwrite):
   - Hermes takes filesystem snapshot
   - Saves state to checkpoint folder
2. Operation runs
3. If successful → keep going
4. If failed or user says "undo":
   - Rollback command restores snapshot
   - System back to previous state
5. Old checkpoints auto-cleaned after N days
```

**Use case:** Safety net for agent mistakes.

---

# Quick Summary Table

| # | Concept | Main benefit |
|---|---|---|
| 1 | Context Compression | Long chats stay cheap |
| 2 | Subagent Delegation | Parallel work, isolated context |
| 3 | Gateway | One agent, many platforms |
| 4 | Sandboxing | Safe command execution |
| 5 | Cron | Scheduled automation |
| 6 | MCP | Infinite tool extension |
| 7 | Profiles | Multiple specialized agents |
| 8 | Voice/Multimodal | Beyond text input/output |
| 9 | Model Routing | Cheap model for cheap tasks |
| 10 | Injection Defense | Security against attacks |
| 11 | Checkpoints | Undo button for agents |

---

**That's the complete Hermes architecture.** You now have the full picture — memory, skills, tools, learning, security, scheduling, and scaling patterns.

These are all reusable patterns for your own projects. Want me to compile this into a single architecture cheat sheet you can keep?
