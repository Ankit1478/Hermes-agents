1. At startup → scan ~/.hermes/skills/ folder
2. Read only name + description (not full content)  💰 saves tokens
3. Add short list to system prompt ("available: A, B, C")  💰
4. User sends message
5. AI matches message to a skill name
6. If match → call skill_view("skill-name") → load FULL recipe ONLY now  💰
7. AI follows recipe steps using tools
8. Task done
9. If complex task + no skill existed → create new SKILL.md
10. If skill existed but failed → patch the SKILL.md file
11. Saved skills available from next chat onward
