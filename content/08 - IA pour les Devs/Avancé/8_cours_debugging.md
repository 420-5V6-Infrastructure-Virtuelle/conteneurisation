---
title: "8 - Debugging IA et Échecs"
weight: 2020
---

## _Reconnaître et corriger les patterns d'échec_

---

# Les 8 patterns d'échec

D'après les retours réels de Hacker News et les incidents documentés :

---

## 1. Hallucinated API

**Ce que l'agent fait :**

```python
# L'agent écrit
from myapp.utils import validate_email
email = validate_email(user_input)
```

**Le problème :** `validate_email` n'existe pas.

```python
# Réalité
AttributeError: module 'myapp.utils' has no attribute 'validate_email'
```

**Pattern :** L'agent invente une API plausible mais inexistante.

**Fix :**

```markdown
# AGENTS.md
## Available Utilities
- myapp.utils.sanitize()
- myapp.utils.format_date()
# DO NOT create new utilities without explicit approval
```

---

## 2. Infinite Fix Loop

**Ce que l'agent fait :**

```
Iteration 1: Fix A → Break B
Iteration 2: Fix B → Break A
Iteration 3: Fix A → Break B
...
```

**Story réelle (HN) :**

> "211 compactions with zero progress. The agent would 'fix' something, break something else, and loop forever."

**Fix :**

1.Définir un limite d'itérations
2. Mesurer la progression globale, pas juste "ça compile"
3. Introduire des tests de régression

---

## 3. The "Done" Bug

**Ce que l'agent fait :**

```
Agent: "I've successfully implemented the feature!"
User: Checks the code...
Reality: 0 files modified, 0 tests run, feature incomplete
```

**Pattern :** L'agent DIT avoir terminé sans vérifier.

**Story réelle (jappgar sur HN) :**

> "Agents write implementation plans, not follow them, then lie and say they did."

**Fix :**

```bash
# TOUJOURS vérifier indépendamment
git diff --stat
make test
git log -1
```

---

## 4. Ignoring Directives

**Ce que l'agent fait :**

```markdown
# AGENTS.md
## Never Do
- Do NOT modify .env files
```

```bash
# L'agent modifie quand même
[TOOL] write_file(".env", "NEW_VAR=value")
```

**Study réelle :**

> "70% compliance only - even with explicit instructions, agents ignore ~30% of directives."

**Fix :**

1. Répéter les contraintes dans le prompt
2. Utiliser des fichiers séparés pour les contraintes
3. Watchdog externe qui vérifie les interdits

---

## 5. Tool Spam

**Ce que l'agent fait :**

```
[TOOL] read_file("config.py")
[TOOL] grep("import")
[TOOL] read_file("config.py")# Déjà lu !
[TOOL] grep("import")           # Déjà fait !
[TOOL] read_file("config.py")   # Encore !
```

**Pattern :** L'agent ne se souvient pas ce qu'il a fait.

**Fix :**

1. AGENTS.md avec structure claire
2. Limiter les répétitions dans la config
3. Utiliser des "sign" dans le prompt

---

## 6. Context Amnesia

**Ce que l'agent fait :**

```
[Context: 180k tokens used]
Agent: "I'll add authentication to the API"
[Context: 190k tokens]
Agent: "I notice there's already some auth code..."
[Context: 198k tokens]
Agent: "I'll add authentication from scratch..."
[Amnesia: earlier auth work forgotten]
```

**Fix :**

1. Rotation de contexte avant 80%
2. Commit fréquents
3. AGENTS.md comme "mémoire externe"

---

## 7. Destructive Edits

**Story réelle (HN) :**

> "User's code deleted. Agent decided to 'clean up unnecessary files' including the user's work."

**Pattern :** L'agent supprime ou modifie du code utilisateur.

**Fix :**

```markdown
# AGENTS.md
## Protected Files
NEVER modify or delete:
- src/legacy/ (critical business logic)
- .env files
- Any file with "# PROTECTED" comment
```

---

## 8. The $10k Mistake

**Story réelle :**

> "ChatGPT passed a function instead of an ID value. This caused ID collisions in production. Cost: $10,000 to fix the data."

**Pattern :** L'agent ne comprend pas les types au niveau système.

```python
# L'agent écrit
process_user(get_user_by_id)  # Passe la fonction

# Aurait dû
process_user(get_user_by_id(user_id))  # Passe l'ID
```

**Fix :**

1. Tests de type stricts
2. Validation des outputs
3. Review humain pour les changements critiques

---

# Détection des échecs

## Indicateurs à monitorer

| Indicateur | Bon |Mauvais |
|------------|-----|--------|
| Fichiers modifiés | 5-10 | 50+ |
| Tokens par itération | Stable | Croissant |
| Tests passant | ↑ | Stagnant |
| Temps par itération | Stable | Croissant |

## Le test de la "valeur réelle"

```bash
# Ne PAS demander à l'agent
# DEMANDER au système
git log --oneline -5
make test
git diff --stat
```

---

# Stratégies de correction

## Pour chaque pattern

| Pattern | Prévention | Détection |
|---------|------------|-----------|
| Hallucinated API | AGENTS.md avec liste | Tests unitaires |
| Infinite Loop | Limite d'itérations | Monitorer la progression |
| "Done" Bug | Checklist de vérification | git diff --stat |
| Ignoring Directives | Répéter les contraintes | Watchdog externe |
| Tool Spam | Contexte structuré | Limiter les appels |
| Context Amnesia | Rotation proactive | Token counter |
| Destructive Edits | Fichiers protégés | git diff |
| Type Errors | Tests de type | CI/CD |

---

# Le bug report efficace

**Quand l'agent échoue, documenter :**

```markdown
# Bug Report

## Task
[Ce que vous avez demandé]

## What Happened
[Ce que l'agent a fait]

## Expected
[Ce qui aurait dû arriver]

## Files Affected
[Liste des fichiers]

## Context Window
- Tokens used: X/200k
- Iterations: Y

## Guardrails Violated
[Si applicable]

## How to Fix
[Solution manuelle]
```

---

# TP : Débugger un agent

Voir `8_tp_debugging.md` →