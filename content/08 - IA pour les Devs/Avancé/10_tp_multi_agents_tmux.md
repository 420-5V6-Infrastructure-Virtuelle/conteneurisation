---
title: "10 - TP Orchestration Multi-Agents"
weight: 2032
draft: true
---

## _Faire travailler plusieurs agents en parallèle_

> ⏱ **1h**

---

# Le problème de la session unique

Un seul agent sur une grosse tâche : contexte qui grossit, qualité qui chute, goulot d'étranglement sur les tâches parallèles.

La solution : plusieurs agents spécialisés qui se passent le travail via des fichiers. Pas de réseau inter-machines, pas de protocole complexe. Des fichiers texte et tmux.

---

# Architecture

```
orchestrateur (agent A)
    └── écrit TASKS.md
            └── worker (agent B)
                    └── lit TASKS.md
                    └── implémente
                    └── écrit STATUS.md
                            └── reviewer (agent C)
                                    └── lit git diff
                                    └── écrit REVIEW.md
```

**Règle unique :** la communication se fait uniquement par fichiers. Chaque agent a un rôle, un contexte minimal, et une sortie définie.

---

# Pourquoi pas a2a ?

Le protocole Agent-to-Agent de Google existe, mais c'est encore du vaporware en production en 2025-2026. Pas de support natif dans Claude Code ou Codex. Ce pattern fichiers+tmux est ce que les équipes font vraiment aujourd'hui — et ça marche.

---

# Mise en place

## Prérequis

Un projet avec du code à refactorer ou une feature à implémenter. Utilisez votre propre projet ou le projet exemple fourni.

## Étape 1 : Créer les 3 sessions tmux

```bash
tmux new -s orchestrateur
# Dans orchestrateur, on lancera l'agent A

# Ctrl+B C pour créer une nouvelle fenêtre
# ou dans un autre terminal :
tmux new -s worker
tmux new -s reviewer
```

Vérifier que les sessions existent :
```bash
tmux ls
# orchestrateur: 1 windows
# worker: 1 windows
# reviewer: 1 windows
```

## Étape 2 : L'agent orchestrateur

Dans la session `orchestrateur` :

```bash
claude -p "
Analyse le code dans src/ et décompose le travail à faire en tâches atomiques.

Écris le résultat dans TASKS.md avec ce format exact :

# TASKS.md

## Tâche 1 : [titre court]
**Fichiers concernés :** src/...
**Objectif :** [une phrase]
**Critère de succès :** [comment savoir que c'est fait]

## Tâche 2 : ...

Chaque tâche doit être indépendante et réalisable en moins de 15 minutes.
Maximum 3 tâches.
"
```

Lire le résultat :
```bash
cat TASKS.md
```

Ajuster si nécessaire (les tâches doivent être vraiment indépendantes).

## Étape 3 : L'agent worker

Dans la session `worker`, lancer en mode autonome dans Docker :

```bash
docker run -it --rm \
  -v $(pwd):/app \
  -w /app \
  --network none \
  node:20 bash

# Dans le container :
npm install -g @anthropic-ai/claude-code
claude --dangerously-skip-permissions -p "$(cat TASKS.md)

Implémente uniquement la Tâche 1.
Quand tu as fini, écris dans STATUS.md :
- Ce que tu as fait
- Les fichiers modifiés
- Les points d'attention pour le reviewer
"
```

Pendant que le worker tourne (Ctrl+B D pour détacher) :

```bash
# Surveiller depuis l'extérieur
tmux attach -t worker   # pour reattacher
watch -n 5 cat STATUS.md   # voir la progression
```

## Étape 4 : L'agent reviewer

Quand STATUS.md indique que la tâche est terminée :

```bash
# Dans la session reviewer :
git diff | claude -p "
Tu es un senior developer qui review du code.

Voici le diff à reviewer :
$(git diff)

Et voici les notes du développeur :
$(cat STATUS.md)

Écris ton review dans REVIEW.md avec :
## Verdict
✅ OK / ⚠️ Points d'attention / ❌ À revoir

## Observations
(bugs potentiels, edge cases manqués, qualité du code)

## Suggestions
(max 3, concrètes et actionnables)
"

cat REVIEW.md
```

---

# Variante : worktrees pour vraiment paralléliser

Si les tâches sont indépendantes, lancer workers A et B en simultané sur des worktrees différents :

```bash
# Worktree pour la tâche 2
git worktree add ../projet-task2 feature/task-2

# Session tmux worker-b
tmux new -s worker-b
# Dans worker-b :
cd ../projet-task2
claude --dangerously-skip-permissions -p "$(cat TASKS.md)
Implémente uniquement la Tâche 2."
```

Workers A et B tournent en parallèle sans conflit de fichiers.

---

# Ce qu'on observe

- L'orchestrateur produit mieux quand sa seule responsabilité est de découper
- Le worker produit mieux quand son contexte est petit (juste la tâche, pas l'historique)
- Le reviewer est plus pertinent qu'un agent qui a fait le code lui-même — moins d'angle mort
- La qualité du `TASKS.md` détermine 80% du résultat final

<!-- ---

# Livrable

À la fin de ce TP :

- [ ] `TASKS.md` généré par l'orchestrateur avec au moins 2 tâches atomiques
- [ ] `STATUS.md` rempli par le worker après implémentation
- [ ] `REVIEW.md` rempli par le reviewer avec un verdict
- [ ] `git diff --stat` montre des changements cohérents avec la tâche
- [ ] Les 3 sessions tmux ont tourné (`tmux ls`) -->
