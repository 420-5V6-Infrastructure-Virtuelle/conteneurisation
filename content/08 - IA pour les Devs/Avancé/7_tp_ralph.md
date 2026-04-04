---
title: "7 - TP Ralph Loop"
weight: 2015
---

## _Mettre en place un workflow autonome_

---

# Prérequis

- OpenCode configuré
- Un projet avec des tests (votre appdémo)
- tmux installé

---

# Partie 1 : Setup tmux

## Pourquoi tmux ?

Laisser l'agent tourner en background pendant que vous gardez le contrôle du terminal.

```bash
# Créer une session dédiée
tmux new -s ralph

# Lancer OpenCode
opencode

# Détacher : Ctrl+B puis D
# Revenir : tmux attach -t ralph
```

---

# Partie 2 : Créer le fichier de tâche

## Étape 1 : Définir une tâche atomique

Le secret : des critères **objectifs et vérifiables**.

```bash
# Créer le fichier de tâche
cat > RALPH_TASK.md << 'EOF'
# Task: Add User Pagination

## Goal
Add pagination to the GET /users endpoint.

## Success Criteria (ALL must pass)
- [ ] GET /users?page=1&limit=10 returns 10 users max
- [ ] Response includes total count
- [ ] Page parameter is validated (error if < 1)
- [ ] Limit parameter is validated (1-100)
- [ ] Tests pass: `make test`
- [ ] No lint errors: `make lint`

## Files to Modify
- src/api/routes/users.py
- tests/test_users.py

## Constraints
- Do NOT modify database schema
- Do NOT add new dependencies
- Keep existing endpoint signature
EOF
```

---

# Partie 3 : Lancer le loop

## Méthode simple (pour débuter)

```bash
# Script bash basique
while :; do
  echo "=== Iteration $(date) ==="
  opencode --prompt "$(cat RALPH_TASK.md)"
  if make test; then
    echo "SUCCESS!"
    break
  fi
  sleep 10
done
```

---

## Méthode avec guardrails

```bash
# Structure du projet pour Ralph
mkdir -p .ralph

cat > .ralph/config.yaml << 'EOF'
task_file: RALPH_TASK.md
guardrails_file: .ralph/guardrails.md
max_iterations: 20
test_command: make test
commit_between_iterations: true
EOF
```

**Créer les guardrails initiaux :**

```markdown
# .ralph/guardrails.md
## Sign: Env Files Are Private
Never modify .env files directly.
Use .env.example for documentation only.
```

---

# Partie 4 : Observer le loop

## Ce que vous verrez

```
=== Iteration 1 ===
[Agent reads files...]
[Agent writes code...]
[Agent runs tests...]
FAIL: test_pagination_page_parameter

=== Iteration 2 ===
[Agent reads guardrails...]
[Agent fixes the issue...]
[Agent runs tests...]
PASS: All tests pass!
SUCCESS!
```

---

# Partie 5 : Analyser les itérations

## Questions à se poser

1. **Combien d'itérations ?**
2. **Quelles erreurs ont été rencontrées ?**
3. **Le guardrails a-t-il été respecté ?**
4. **Y a-t-il eu des oublis ?**

---

# Partie 6 : Limite d'itérations

**TOUJOURS définir une limite !**

```bash
# Script avec limite
MAX_ITERATIONS=20
count=0

while [ $count -lt $MAX_ITERATIONS ]; do
  echo "=== Iteration $((count+1))/$MAX_ITERATIONS ==="
  opencode --prompt "$(cat RALPH_TASK.md)"
  
  if make test; then
    echo "SUCCESS after $((count+1)) iterations!"
    break
  fi
  
  count=$((count+1))
  sleep 5
done

if [ $count -eq $MAX_ITERATIONS ]; then
  echo "FAILED: Max iterations reached"
  exit 1
fi
```

---

# Partie 7 : Cas d'échec

## Créer un cas d'échec volontaire

```markdown
# RALPH_TASK_FAIL.md
# Task: Refactor All Code
## Goal
"Improve the code quality"  # Trop vague !
## Success Criteria
- [ ] Code is better
```

**Lancer et observer l'échec :**
- L'agent va tourner en boucle
- "Better" n'est pas mesurable
- Aucun testobjectif ne passe

---

# Partie 8 : Créer un guardrails

## Après un échec, documenter

```bash
# Si l'agent a fait une erreur, créer un guardrails
cat > .ralph/guardrails.md << 'EOF'
# Guardrails

## Sign: Pagination Must Use Offset/Limit
**Learned from iteration #3**
- Attempted to use cursor pagination
- Project uses offset/limit convention
- Always check existing pagination patterns first

## Sign: Never Delete Test Files
**Learned from iteration #7**
- Deleted tests to make them pass
- ALWAYS fix tests, never delete them
EOF
```

---

# Livrable

À la fin de ce TP :

- [ ] Un fichier `RALPH_TASK.md` avec critères objectifs
- [ ] Un loop fonctionnel avec limite d'itérations
- [ ] Connaissance des patterns d'échec
- [ ] Premiers guardrails documentés

---

# Checkpoint

**Pattern retenu :** Ralph Loop pour tâches atomiques avec critères vérifiables.

**Question clé :** Quelle est la première tâche que vous confieriez à un loop autonome ?

---

# Prochain module

Module 8 : Debugging IA - reconnaître et corriger les échecs.