# The Future of MCP: Simple Guide

A plain-English breakdown of the 6 key concepts from David Soria Parra's keynote.

---

## What is MCP?

**MCP (Model Context Protocol)** lets AI agents connect to apps and tools like Gmail, GitHub, and Slack so they can do real work for you.

---

## 1. Progressive Discovery

### What it means
Don't load every tool into the model at once. Let it search for tools when needed.

### Why it matters
If you connect 10 apps, that's about 150 tools, which equals 35,000 tokens wasted before anything happens.

### Simple example
```
You: "Send an email to John"

Without discovery:
  Model sees all 158 tools, slow and expensive

With discovery:
  Model: tool_search("email")
  Loads only gmail_send
  Sends email
```

### Analogy
A waiter asks "What are you in the mood for?" instead of handing you a 500-page menu.

---

## 2. Programmatic Tool Calling

### What it means
Instead of calling tools one by one, the model writes a small script that runs many tools at once.

### Why it matters
- Fewer back-and-forth turns
- Tools run in parallel
- Intermediate results don't clog up context

### Simple example

**Task:** "Check 10 PRs and tell me which have failing tests."

**Without programmatic (slow):**
```
Check PR 1, get result
Check PR 2, get result
... (10 times)
```

**With programmatic (fast):**
```javascript
const prs = await github.listPRs();
const checks = await Promise.all(
  prs.map(pr => github.getChecks(pr.number))
);
return checks.filter(c => c.failed).length;
```
One script. Runs in parallel. Returns just the answer.

### Analogy
Sending a friend one to-do list instead of 20 separate texts.

---

## 3. Stop 1:1 REST to MCP

### What it means
Don't blindly copy your REST API into MCP. Redesign tools for agents, not for frontends.

### Why it matters
Agents don't want to chain 5 calls. They want one tool that does the whole task.

### Simple example

**Bad (REST mirror):**
```
get_user, then get_user_orders, then get_order, then get_order_items
```
4 calls just to answer "What did John order?"

**Good (agent-designed):**
```
find_customer_orders(name="John", date="last week")
```
1 call. Server does the joining.

### Analogy
Ordering a burger instead of buying buns, patties, and lettuce separately.

---

## 4. Skills over MCP

### What it means
Ship instructions on how to use the tools alongside the tools themselves.

### Why it matters
Tools tell the model what it can do. Skills tell it how to do it correctly.

### Simple example

Stripe ships a `refund-handling.md` skill with its MCP server:

```markdown
# How to issue a refund
1. Search by EMAIL (faster than ID)
2. Charge must be less than 90 days old
3. Amount must be in CENTS
4. Always send a receipt, Stripe doesn't auto-notify
```

Now the model follows the playbook instead of guessing.

### Analogy
IKEA box without manual equals chaos. IKEA box with manual equals a chair.

---

## 5. Cross-App Access

### What it means
One login unlocks multiple apps for the agent, like "Sign in with Google" but for AI.

### Why it matters
No more logging into Gmail, Calendar, and Drive separately. One consent gives all access.

### Simple example

**Without cross-app access:**
```
Agent: "Log in to Gmail"
Agent: "Log in to Calendar"
Agent: "Log in to Drive"
You: frustrated
```

**With cross-app access:**
```
Agent: "Sign in with Google once"
You: done
Gmail, Calendar, Drive all unlocked
```

### Analogy
A master keycard that opens every floor in your office building.

---

## 6. Server Discovery

### What it means
Agents auto-find MCP servers via standard URLs on websites.

### Why it matters
No more manually hunting for and configuring MCP endpoints.

### Simple example

Websites publish their MCP info at a standard location:
```
restaurant.com/.well-known/mcp.json
```

Agent flow:
```
You: "Book a table at restaurant.com for 7pm"
Agent: checks /.well-known/mcp.json
Agent: connects and books table
```

### Analogy
Same pattern as `robots.txt` for search engines, a known address everyone checks.

---

## Quick Summary Table

| # | Concept | One-Line Idea |
|---|---------|---------------|
| 1 | Progressive Discovery | Find tools only when needed |
| 2 | Programmatic Tool Calling | Run many tools in one script |
| 3 | Stop 1:1 REST to MCP | Design tools for agents, not APIs |
| 4 | Skills over MCP | Ship the "how to use" guide too |
| 5 | Cross-App Access | One login, many apps |
| 6 | Server Discovery | Agents auto-find servers via URLs |

---

## How They Fit Together

```
Discovery     finds the right server
                    |
Skills        teaches the model how to use it
                    |
Programmatic  uses tools efficiently in one script
                    |
Cross-App     handles auth across many services
```

**Two buckets:**
- **Developer techniques:** #1, #2, #3
- **Protocol upgrades:** #4, #5, #6

---

## Key Takeaway

The future of MCP is about making agents smarter, faster, and easier to connect, with less context bloat, smarter tool design, and internet-scale discovery.

2026 will be the year of general agents.
