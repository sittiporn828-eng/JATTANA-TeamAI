---
name: doggy-vault-search
description: Use when answering questions about prior JATTANA work, decisions, projects, or user preferences. Search the Doggy knowledge vault before answering.
---

# Doggy vault search

Before answering a question about prior work, decisions, project history, or persistent user context, run the local vault query script:

```bash
python "C:/Users/Acer/Desktop/JATANA GROUP/query_vault.py" "<question>" --top 5
```

Use returned chunks as evidence. Prefer the most recent note when entries conflict, say when there is no reliable match, and cite the source path. Do not dump all results into the response.

This search is read-only; never modify the vault or `vault.db` during a query.
