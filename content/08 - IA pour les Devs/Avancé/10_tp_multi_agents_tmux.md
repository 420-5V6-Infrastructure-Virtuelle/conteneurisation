---
title: "10 - TP Orchestration Multi-Agents"
weight: 2032
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

# Mise en place

## Prérequis

Un projet avec du code à refactorer ou une feature à implémenter. Utilisez votre propre projet ou le projet exemple fourni.

## Étape 1 : Ajouter le skill "smux"

<https://github.com/ShawnPana/smux>

---

## Etape 2 : demander à un premier agent de déléguer
- soit avec smux
- soit avec les subagents de Codex

---

# Worktrees pour vraiment paralléliser

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

---

# Livrable

À la fin de ce TP :

- [ ] `TASKS.md` généré par l'orchestrateur avec au moins 2 tâches atomiques
- [ ] `STATUS.md` rempli par le worker après implémentation
- [ ] `REVIEW.md` rempli par le reviewer avec un verdict
- [ ] `git diff --stat` montre des changements cohérents avec la tâche
- [ ] Les 3 sessions tmux ont tourné (`tmux ls`)
