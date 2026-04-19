# Notes: The "Frozen Memory" Trick

## Core Concept

**Keep your system prompt identical across turns in a session so LLM providers can cache it — you pay less and responses are faster.**

---

## The Problem It Solves

Every message you send to an LLM includes:
- System prompt (instructions, context, user info)
- Conversation history
- Current user message

The **system prompt is usually the biggest part** — and it's mostly the same every turn.

Without caching, you pay for those same tokens **every single message**.

---

## How Prompt Caching Works

Major LLM providers (Anthropic, OpenAI, Google) offer prompt caching:

- **Send the same prefix twice within a time window** → provider caches it
- **Cached tokens cost ~10% of normal price** (Anthropic: up to 90% cheaper)
- **Cached tokens are also faster** (no reprocessing)

**Catch:** The cache matches on **exact prefix**. Change one character → cache miss → full price again.

---

## The "Frozen" Rule

> 🔒 **Once a session starts, do not modify the system prompt until the session ends.**

Any updates you learn during the session get:
1. **Written to disk immediately** (so they aren't lost)
2. **Applied only on the next session's startup**

---

## Architecture Pattern

```
┌─────────────────────────────────────┐
│  SESSION START                      │
│  1. Load all context from disk      │
│  2. Build system prompt             │
│  3. FREEZE IT                       │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  DURING SESSION                     │
│  • System prompt = unchanged        │
│  • Cache hits = cheap + fast        │
│  • New facts → write to disk only   │
│  • Do NOT re-inject into prompt     │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  NEXT SESSION                       │
│  New facts are picked up here       │
└─────────────────────────────────────┘
```

---

## Implementation Checklist

When building this into your own system:

### ✅ Do
- Load all "static" context (user profile, facts, preferences, instructions) **once** at session start
- Keep the system prompt **byte-identical** across turns
- Put **stable content first** (it's what gets cached)
- Write new learnings to persistent storage (DB, file) during the session
- Reload everything fresh on next session start

### ❌ Don't
- Don't update the system prompt mid-session (kills cache)
- Don't inject timestamps or changing values into the cached section
- Don't reorder content between turns
- Don't mix dynamic and static content in the same block

---

## Structure Your Prompt in Layers

Put content in order of **how often it changes**:

```
┌────────────────────────────────────┐
│ LAYER 1: Never changes              │ ← cached
│ • System instructions               │
│ • Tool definitions                  │
│ • Agent personality                 │
├────────────────────────────────────┤
│ LAYER 2: Changes per session        │ ← cached
│ • User profile                      │
│ • Known facts                       │
│ • Loaded skills/knowledge           │
├────────────────────────────────────┤
│ LAYER 3: Changes per turn           │ ← not cached
│ • Current conversation              │
│ • Tool results                      │
│ • Latest user message               │
└────────────────────────────────────┘
```

Put Layer 1 + Layer 2 in the cached prefix. Put Layer 3 after the cache breakpoint.

---

## Size Discipline

Because the cached prefix is loaded **every turn**, keep it lean:

| Content Type | Recommendation |
|---|---|
| User profile | ~1,500 chars max |
| Known facts | ~2,200 chars max |
| Instructions | As short as possible |

**When it fills up:** compress/merge entries before adding new ones — don't just keep growing it.

---

## Benefits You Get

| Benefit | Why |
|---|---|
| 💰 **Cheaper** | Cached tokens cost 10–25% of normal |
| ⚡ **Faster** | No reprocessing of prefix |
| 🧠 **Stable behavior** | AI doesn't get confused by mid-session rewrites |
| 🔁 **Consistent context** | Same baseline knowledge throughout session |

---

## Trade-offs to Know

| Pro | Con |
|---|---|
| Big cost savings | Mid-session learnings only apply next time |
| Faster responses | Need clear session boundaries |
| Stable AI behavior | Must design "session end" logic carefully |

---

## When to Use This Pattern

✅ **Good fit:**
- Chatbots / personal assistants
- Agents with recurring sessions
- Long conversations with shared context
- Any system where users return multiple times

❌ **Bad fit:**
- One-shot API calls (no repeat turns)
- Systems where context must update in real-time mid-conversation
- Very short interactions (<3 turns)

---

## Provider-Specific Notes

**Anthropic (Claude):**
- Uses `cache_control` markers
- 5-minute cache TTL (or 1 hour with extended option)
- ~90% cost reduction on cache hits
- Mark up to 4 cache breakpoints

**OpenAI:**
- Automatic caching for prompts >1024 tokens
- ~50% cost reduction
- No manual markers needed

**Google (Gemini):**
- Explicit context caching API
- Set TTL manually
- ~75% cost reduction

---

## Quick Pseudo-Code

```python
class Session:
    def __init__(self, user_id):
        # Load everything ONCE
        self.system_prompt = self.build_frozen_prompt(user_id)
        self.cache_marker = mark_as_cacheable(self.system_prompt)
        self.pending_updates = []  # saved but not injected
    
    def build_frozen_prompt(self, user_id):
        return f"""
        {load_instructions()}
        {load_user_profile(user_id)}
        {load_known_facts(user_id)}
        {load_skills_index(user_id)}
        """
    
    def chat(self, message):
        # System prompt stays identical every turn
        response = llm.call(
            system=self.system_prompt,  # ← cached
            messages=self.history + [message]
        )
        return response
    
    def learn_fact(self, fact):
        # Save to disk, DON'T modify current prompt
        db.save(fact)
        self.pending_updates.append(fact)
    
    def end_session(self):
        # Updates will appear in next session's prompt
        db.commit()
```

---

## Key Takeaway

> **Stability = Savings.**
>
> The more your system prompt stays the same, the more you save. Design your architecture around clear session boundaries where "static context" is loaded fresh, then frozen for the whole session.

---

## Related Patterns to Pair With This

- **Lazy loading** — load details only when needed (like Hermes does with skills)
- **Session summarization** — compress old turns to keep the tail short
- **Write-through cache** — update disk immediately, apply to prompt later
- **Layered prompts** — separate static vs. dynamic content

---

Save these notes. This single pattern can cut LLM costs by 70-90% on any long-running agent or chatbot you build. 💰

Want notes on any other Hermes concept (lazy-loaded skills, session compression, the learning loop)?
