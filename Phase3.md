# Session History — Flow + Token Optimization

```
1. Every chat saved to ~/.hermes/state.db (SQLite)
2. Stored as: date + your message + AI reply
3. FTS5 index makes it searchable (like Google for your chats)
4. User asks about past → "what did we do with Docker last month?"
5. AI calls session_search("Docker")  💰 not loaded by default
6. SQLite returns matching old chats in milliseconds
7. Cheap small model summarizes results  💰 saves tokens
8. Summary injected into current chat
9. AI answers using the summary
10. No size limit — thousands of chats, still fast
```

---

## 💰 Token Optimizations in Diary

| # | Trick | How it saves |
|---|---|---|
| 1 | **Not loaded by default** | Past chats NOT in prompt unless needed |
| 2 | **Search before load** | Only matching chats retrieved, not all |
| 3 | **Small model summarizes** | Cheap model (e.g. Gemini Flash) compresses results |
| 4 | **Summary, not raw text** | Inject 200-word summary instead of 5,000-word chat |
| 5 | **SQLite = free** | No database server cost, just a file |

---

## 🎯 Example of Saving

**Without diary search:**
```
Load last 1000 chats into prompt = 500,000 tokens 💸💸💸💸
Impossible — exceeds context window
```

**With diary search:**
```
Search "Docker" → finds 3 relevant chats (5,000 tokens)
Cheap model summarizes → 200 tokens
Inject summary into main model ✅
```

**Savings: ~2,500x less tokens.** 🚀

---

## 🔑 Key Concept

**Diary is "cold storage."**
- Always saved
- Never loaded unless asked
- Search → summarize → inject

Like Google Drive for conversations — searchable, huge, but doesn't clutter your current work.

---

Say **"next"** for Type 4 (Intuition / Honcho). 🧠
