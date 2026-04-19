# Flow + Token Optimization

```
1. At startup → scan ~/.hermes/skills/ folder
2. Read only name + description (not full content)  saves tokens
3. Add short list to system prompt ("available: A, B, C") 
4. User sends message
5. AI matches message to a skill name
6. If match → call skill_view("skill-name") → load FULL recipe ONLY now 
7. AI follows recipe steps using tools
8. Task done
9. If complex task + no skill existed → create new SKILL.md
10. If skill existed but failed → patch the SKILL.md file
11. Saved skills available from next chat onward
```

---

## Token Optimizations in Skills

| # | Trick | How it saves |
|---|---|---|
| 1 | **Lazy loading** | Only names loaded at start, not full content |
| 2 | **Load on demand** | Full recipe loaded only when AI needs it |
| 3 | **Skills replace thinking** | Next time, AI follows recipe instead of reasoning from scratch (fewer LLM calls) |
| 4 | **Short descriptions** | Each skill described in 1 line, not paragraphs |
| 5 | **Conditional loading** | Platform-specific skills (e.g. macOS) hidden on other OS |

---

##  Example of Saving

**Without lazy loading:**
```
Prompt = system + memory + 40 FULL skills (50,000 tokens)
Every turn = 50,000 tokens
```

**With lazy loading:**
```
Prompt = system + memory + 40 skill NAMES (500 tokens)
Every turn = 500 tokens
Full skill loaded only when needed (once, not every turn)
```

**Savings: ~100x less tokens per turn.** 

---

**Bottom line:**
- Lazy loading = huge win
- Skills themselves = fewer re-thinking calls
- Combined with caching = massive cost cut
