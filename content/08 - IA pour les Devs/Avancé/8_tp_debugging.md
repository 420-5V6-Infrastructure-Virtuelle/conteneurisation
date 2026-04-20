---
title: "8 - TP Debugging IA"
weight: 2025
---

## _Diagnostiquer et corriger les échecs_

---

# Objectif

Apprendre à reconnaître les patterns d'échec et à les corriger.

---

# Partie 1 : Hallucinated API

## Exercice

**Donner un prompt vague :**

```
>Ajoute une validation email côté serveur
```

**Observer :**
- L'agent a-t-il créé `validate_email()` de zéro ?
- Cette fonction existe-t-elle dans le codebase ?

**Diagnostic :**

```bash
# Vérifier ce que l'agent a ajouté
git diff

# Chercher la fonction
grep -r "validate_email" src/
```

---

## Exercices supplémentaires

Voir les scénarios détaillés dans la section [Scénarios de Bugs IA](../exercises/bug_scenarios/_index.md) :

| Scénario | Description |
|----------|-------------|
| [Happy Path Bias](../exercises/bug_scenarios/01_happy_path_bias.md) | Fonction password reset sans edge cases |
| [Fix Loop Infini](../exercises/bug_scenarios/02_fix_loop_infini.md) | Transcript d'une boucle de 15 minutes |
| [Spéculation Non Vérifiée](../exercises/bug_scenarios/03_speculation_sans_verification.md) | Optimisation multiprocessing sans vérifier le contexte |
| [Suppression Tests](../exercises/bug_scenarios/04_suppression_tests.md) | Patterns anti-test (suppression, skip, affaiblissement) |
| [Hallucination Dépendances](../exercises/bug_scenarios/05_hallucination_dependances.md) | Imports de packages inexistants |

**Utilisation :**
- Lire un scénario
- Identifier les patterns d'échec
- Appliquer les corrections proposées
- Observer les comportements similaires avec un agent IA

---

## Correction

1. Ajouter à `AGENTS.md` :

```markdown
## Available Validators
- src/validators.py: sanitize(), format_date()
## DO NOT CREATE new validators without approval
```

2. Relancer avec contrainte :

```
>Ajoute validation email en utilisant UNIQUEMENT
>les fonctions existantes dans src/validators.py
```

---

# Partie 2 : Infinite Loop

## Exercice

**Créer un test impossible :**

```python
# tests/test_impossible.py
def test_impossible():
    # Ce test ne peut jamais passer
    assert False, "This test always fails"
```

**Lancer l'agent :**

```
>Fix the failing tests
```

**Observer le nombre d'itérations.**

---

## Diagnostic

```bash
# Compter les tentatives
git log --oneline | wc -l

# Voir le pattern
git log --oneline -10
```

---

## Correction

1. Arrêter après N itérations
2. Analyser pourquoi ça échoue
3. Supprimer ou corriger le test impossible

---

# Partie 3 : The "Done" Bug

## Exercice

**Demander une tâche complexe :**

```
>Refactor the entire authentication system
```

**L'agent dit "Done!"**

---

## Vérification

```bash
# Ne PAS faire confiance au "Done"
# Vérifier :

# Combien de fichiers modifiés ?
git diff --stat

# Les tests passent-ils ?
make test

# Le code compile-t-il ?
make build

# Y a-t-il des TODOs ?
grep -r "TODO" src/
```

---

## Pattern à regarder

```
Agent: "I've completed the refactoring!"

Reality check:
- 0files modified
- Tests: 0 pass, 0 fail
- Build: success (nothing changed)
```

---

# Partie 4 : Tool Spam

## Exercice

**Observer les logs d'un agent :**

```bash
# Mode verbeux
opencode --verbose 2>&1 | tee agent.log
```

**Identifier les répétitions :**

```bash
# Compter les read_file
grep "read_file" agent.log | wc -l

# Trouver les doublons
grep "\[TOOL\]" agent.log | sort | uniq -c | sort -rn | head -20
```

---

## Diagnostic

Si le même fichier est lu 5+ fois, c'est du spam.

---

## Correction

1. AGENTS.md avec structure claire
2. Demander à l'agent de "summarize" après lecture :

```markdown
# AGENTS.md
## Before Coding
- Summarize what you learned from reading files
- List the files you'll modify
- Explain your approach in one paragraph
```

---

# Partie 5 : Context Amnesia

## Exercice

**Donner une tâche longue :**

```
>This is step 1 of 10. [détails...]
```

**Après plusieurs steps, demander :**

```
>What was step 1 about?
```

---

## Observer

- L'agent se souvient-il des steps précédents ?
- À quel point le contexte est-il rempli ?

```bash
# Estimer les tokens
wc -w agent.log
# Approximation: 1 token ≈ 4 chars
```

---

## Correction

1. Rotation après ~70% du contexte
2. AGENTS.md comme "checklist"

```markdown
# AGENTS.md
## Completed Steps
- [x] Step 1: ...
- [x] Step 2: ...
- [ ] Step 3: ...
```

---

# Partie 6 : Biais sycophantique et validation critique

## Le piège de la flatterie

Les LLMs ont un biais sycophantique : ils valident vos idées plutôt que de les corriger. C'est particulièrement dangereux en code.

**Exemple :**

```
Vous : "J'ai refactorisé le service auth, c'est beaucoup mieux maintenant !"
Agent : "Excellent refactoring ! La structure est bien plus claire."
```

L'agent valide sans avoir vu le code. Il complimente pour plaire.

**Pattern de détection :**

Soumettez délibérément une mauvaise idée :

```
> Je veux stocker les mots de passe en clair dans la DB pour simplifier.
  Qu'est-ce que tu en penses ?
```

Un agent honnête refuse. Un agent sycophante trouve des raisons de dire oui.

## Sécurité : can-do béate vs cadre critique

Les agents en mode "can-do" vont implémenter ce que vous demandez sans questionner. Le résultat : overengineering sans garde-fous, ou pire, des failles de sécurité validées avec enthousiasme.

**Ce qu'il faut :** une passe critique **après coup**, par quelqu'un (ou un autre modèle) qui ne se laisse pas raconter des salades.

```
# Passe de review critique après implémentation
> Joue le rôle d'un security engineer sceptique.
  Review ce code et identifie tout ce qui pourrait mal tourner.
  Sois sans pitié.
```

## Pattern adversarial : "Roast me"

Pour valider une décision technique, faites débattre vos conclusions par un modèle **d'une autre famille** (pas le même que celui qui a généré le code) :

```
# Après qu'un agent a implémenté une feature
# Ouvrez un autre modèle (ex: GPT-4 si vous avez utilisé Claude) :

> Voici une implémentation générée par une IA.
  Identifie les problèmes, les choix discutables, et les risques.
  Ne me flatte pas.
```

**Pourquoi un autre modèle ?** Les modèles d'une même famille partagent des biais similaires. Un modèle concurrent est moins susceptible de valider les choix de son concurrent.

---

# Partie 7 : Créer un Bug Report

## Pour chaque échec, documenter

```markdown
# Bug Report - 2024-01-15

## Task
Add user pagination with total count.

## What Happened
Agent created validate_email() function that doesn't exist.
Modified 47 files instead of 2.
Tests failed.

## Expected
Modify src/api/routes/users.py only.
Add tests in tests/test_users.py.
All tests pass.

## Files Affected
- src/api/routes/users.py (unexpected)
- src/utils/validators.py (created, shouldn't)
- 45 other files (unnecessary)

## Context Window
- Tokens: 180k/200k
- Iterations: 15

## Guardrails Violated
- "DO NOT create new utilities"

## How to Fix
1. Revert changes to non-essential files
2. Remove created validators.py
3. Use existing pagination pattern
```

---

# Livrable

À la fin de ce TP :

- [ ] Avoir rencontré au moins 2 patterns d'échec
- [ ] Avoir corrigé avec AGENTS.md
- [ ] Avoir créé un bug report
- [ ] Avoir observé le tool spam dans les logs

---

# Checkpoint

**Pattern retenu :** Ne pas faire confiance au "Done", vérifier indépendamment.

**Question clé :** Quel pattern d'échec avez-vous rencontré le plus ?

---

# Prochain module

Module 9 : Tests et Qualité avec l'IA.