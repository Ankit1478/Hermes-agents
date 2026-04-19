# Intuition (Honcho) — Flow + Token Optimization

```
1. Honcho runs as SEPARATE service (cloud or self-hosted)
2. After every chat turn → Hermes sends conversation to Honcho (background)  💰 async
3. Honcho's AI analyzes: "What does this reveal about the user?"
4. Honcho saves insights (preferences, patterns, goals)
5. Next chat starts → Hermes asks Honcho: "What should I know?"
6. Honcho returns short summary of user
7. Summary injected into system prompt  💰 small
8. AI responds with personalized understanding
9. Cycle repeats — insights deepen over time
```

---

## 💰 Token Optimizations in Honcho

| # | Trick | How it saves |
|---|---|---|
| 1 | **Async writes** | Analysis happens in background, doesn't slow your chat |
| 2 | **Separate LLM** | Honcho uses its own model for analysis, not your main one |
| 3 | **Only summary injected** | Small snippet added to prompt, not full history |
| 4 | **Appended AFTER cache** | Injected outside cached section so cache still works |
| 5 | **Cadence control** | Not every turn — runs every N turns to save calls |

---

## 🎯 Example of Saving

**Without Honcho's smart design:**
```
Every turn = analyze full history + inject everything 💸💸💸
```

**With Honcho:**
```
Turn 1-2: chat happens (fast, cached)
Background: Honcho analyzes last turn (separate)
Turn 3: Honcho injects small summary "user prefers X, Y, Z"
Main prompt stays cached ✅
```

**Savings:** Zero latency impact + cache stays intact.

---

## 🔑 Key Concept

**Honcho is "slow thinking" behind the scenes.**
- Main chat = fast, cached, cheap
- Honcho = slow analysis in background
- Only the *output* (small summary) reaches your main chat

Like a personal assistant who studies you in their spare time, then whispers useful insights to you later.

---

## 🏁 All 4 Memory Types Summary

| Type | What | Where | Loaded When |
|---|---|---|---|
| **1. Sticky Notes** | Facts | MEMORY.md + USER.md | Every chat start (frozen) |
| **2. Recipes** | How-to skills | skills/ folder | Names always, full on demand |
| **3. Diary** | Past chats | SQLite + FTS5 | On search only |
| **4. Intuition** | User model | Honcho service | Small snippet per chat |

---

**You now understand the complete Hermes memory architecture.** 🎯
