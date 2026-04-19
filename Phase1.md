When user send prompt for first time 

1. Load `MEMORY.md`
2. Load `USER.md`
3. System prompt (instructions + tools)
4. **Skills index** (just names + descriptions, not full content)
5. User message

# After Loading — What Happens Next

```
5 things loaded → sent to AI
         ↓
    AI thinks
         ↓
    AI decides: answer OR use tool?
         ↓
   ┌─────┴─────┐
   ↓           ↓
ANSWER       TOOL
(done)      (run it → get result → back to AI)
         ↓
   Final answer to you
         ↓
   Save chat to diary (SQLite)
         ↓
   Maybe save new fact to MEMORY.md
         ↓
   Maybe create/update a skill
```

**In short:**

6. AI reads everything
7. AI replies OR calls a tool
8. If tool → run it → send result back → AI continues
9. Final answer shown to you
10. Chat saved to diary
11. New facts saved (if any)
12. New skill created (if task was complex)


# How Sticky Notes Work 

**Two text files:**
- `MEMORY.md` → facts about your stuff
- `USER.md` → facts about you

**The flow:**

1. Chat starts → Hermes reads both files
2. Pastes content into AI prompt
3. Prompt is **frozen** for whole chat (saves money via caching)
4. AI learns new fact → saves to file on disk
5. New fact appears in **next** chat (not current one)

**When file is full:**
- AI merges old entries to make space
- Auto-managed, you don't touch it

**Size limits:**
- MEMORY.md → ~2,200 chars
- USER.md → ~1,375 chars

**That's it.** Plain text files + 4 tools (add, replace, remove, list).


# Frozen Prompt

**What:** System prompt stays **identical** every turn in a chat.

**Why:** LLMs cache repeated prompts → 90% cheaper + faster.

**Rule:** New facts learned mid-chat → saved to disk, but **not** added to current prompt. They appear in next chat.

**Trade-off:** Slight delay in using new info, but huge cost savings.

# How Frozen Prompt Works

**Turn 1:**
```
Prompt: [instructions + MEMORY.md + USER.md]
→ Sent to AI (full price, gets cached)
```

**Turn 2:**
```
Prompt: [SAME instructions + SAME MEMORY.md + SAME USER.md]
→ AI sees "this is identical" → uses cache (10% price)
```

**Turn 3, 4, 5...** same thing → cache hit every time.

**If AI learns new fact mid-chat:**
```
→ Write to MEMORY.md on disk ✅
→ Do NOT rebuild the prompt ❌
→ Current chat keeps using old frozen prompt
→ Next chat reads updated file
```

**Why it works:** Cache matches on **exact text**. Change one word → cache breaks → full price again. So Hermes never touches the prompt mid-chat.

## Your confusion:
*"If the AI sends the same prompt every turn anyway, where's the saving?"*

---

## The answer: **Caching happens on the PROVIDER's side, not yours**

You still **send** the same prompt every turn. But the LLM provider (Anthropic, OpenAI) does this:

---

## Turn 1 (first time)

```
You send: [10,000 tokens of prompt]
         ↓
Provider: "Never seen this before. Let me process it fully."
         ↓
Provider saves it in their cache
         ↓
You pay: FULL price for 10,000 tokens
```

---

## Turn 2 (same prompt)

```
You send: [SAME 10,000 tokens]
         ↓
Provider: "Wait, I already processed this! I have it ready."
         ↓
Provider skips re-processing → uses cached version
         ↓
You pay: 10% price for 10,000 tokens
```

---

## Why the saving is real

Processing tokens is **expensive work** for the provider (GPU compute). If they already did that work, they charge you less to re-use it.

**It's like a coffee shop:**
- First cappuccino → $5 (they grind beans, steam milk, etc.)
- Same order in 2 minutes → $0.50 (they just pour from what's already made)

---

## The key rule

**Cache only works if the text is byte-identical.**

Change one character → provider thinks it's new → full price again.

That's why Hermes **freezes** the prompt. Any mid-chat change would break the cache and cost you full price.

---

## Summary

- You still send the full prompt every turn
- Provider recognizes "I've seen this" → reuses processed version
- You pay ~10% instead of 100%
- If prompt changes → cache breaks → full price

**The saving isn't on your side. It's on the provider's side, passed to you as a discount.**



# Ah, Got It — Only About Sticky Notes

For **Sticky Notes specifically**, cost cutting comes from **3 things**:

---

## 1. Prompt Caching (biggest)
Files stay identical every turn → provider cache hit → 90% cheaper

## 2. Small Size Limits
- MEMORY.md → 2,200 chars max
- USER.md → 1,375 chars max

Smaller file = fewer tokens sent every turn = less cost.

## 3. Frozen During Session
Not re-read or re-built mid-chat → no wasted processing → cache stays valid

---

## That's it for Sticky Notes — just 3 tricks:

Cache it
Keep it small
Don't change it


# Short Answers

### 1. Prompt Caching
Same text every turn → provider recognizes it → skips re-processing → charges 10% instead of 100%.

### 2. How summarization keeps it under limits
- AI tries to add new fact
- If file > limit → tool returns error: *"Memory full, consolidate first"*
- AI merges old lines into one shorter line
- Then adds the new fact

**Example:**
```
Before (3 lines):
- Project uses FastAPI
- Project uses PostgreSQL  
- Project uses Docker

After (1 line):
- Project: FastAPI + PostgreSQL + Docker
```

### 3. Frozen During Session
Files read **once** at chat start. Even if AI adds new facts mid-chat, the prompt **doesn't change**. New facts appear only in next chat.




## 1. Fetch MEMORY.md        (facts)
## 2. Fetch USER.md          (preferences)
## 3. Fetch system prompt    (instructions)
## 4. Fetch skills index     (names only, not full)
## 5. Add user message
## 6. Send → cached by provider (90% cheaper)
## 7. If files full → AI merges old entries
## 8. New facts saved to disk, applied next chat (frozen rule)
