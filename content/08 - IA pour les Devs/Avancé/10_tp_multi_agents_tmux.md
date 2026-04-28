---
title: "10 - TP Orchestration Multi-Agents"
weight: 2032
draft: false
---

## _Faire travailler plusieurs agents en parallèle_

> ⏱ **1h**

---

# Le problème de la session unique

Un seul agent sur une grosse tâche : contexte qui grossit, qualité qui chute, goulot d'étranglement sur les tâches parallèles. Et tout sur un gros modèle = facture qui explose.

La solution : **un agent "cerveau"** (gros modèle, qui réfléchit et découpe) qui lance lui-même **des subagents "exécutants"** (petit modèle économique type `gpt-4o-mini`, qui appliquent le plan) dans des fenêtres tmux. L'humain ne touche jamais à tmux directement — il parle à l'orchestrateur, point.

---

# Architecture

```
HUMAIN
   │
   ▼
orchestrateur (Claude / gros modèle)  ← réflexion, plan, review
   │
   │  lance via `tmux new-window`
   │
   ├──▶ worker-1 (codex -m gpt-4o-mini)  ← exécute la tâche 1
   ├──▶ worker-2 (codex -m gpt-4o-mini)  ← exécute la tâche 2
   └──▶ worker-3 (codex -m gpt-4o-mini)  ← exécute la tâche 3
            │
            ▼
       fichiers TASKS.md / STATUS.md / REVIEW.md
```

**Règles :**
- L'humain ne crée aucune fenêtre tmux à la main.
- L'orchestrateur (gros modèle) **réfléchit** : découpe, planifie, review.
- Les workers (petit modèle) **exécutent** : pas de réflexion globale, juste appliquer une tâche cadrée.
- Communication uniquement par fichiers.

---

# Pourquoi ce pattern ?

- **Coût** : un gros modèle qui pense, des petits modèles qui tapent du code. Diviser la facture par 5 à 10 sur les phases d'exécution.
- **Parallélisme réel** : 3 workers qui tournent en même temps dans 3 fenêtres tmux indépendantes.
- **Pas d'a2a** : le protocole Agent-to-Agent de Google reste du vaporware en prod en 2025-2026. Fichiers + tmux, c'est ce que font vraiment les équipes.

---

# Mise en place

## Prérequis

```bash
# Claude Code (orchestrateur)
npm install -g @anthropic-ai/claude-code

# Codex CLI (workers économiques)
npm install -g @openai/codex

# tmux
sudo apt install tmux   # ou équivalent
```

Un projet avec du code à refactorer ou une feature à implémenter.

<!-- ## Étape 1 : Démarrer la session tmux unique

Une seule fois, par l'humain :

```bash
tmux new -s multi-agents
``` -->

<!-- C'est tout. À partir d'ici, **l'orchestrateur s'occupe du reste**. -->

## Lancer l'orchestrateur

Dans la fenêtre principale :

```bash
claude
```

Puis lui donner la consigne :

```
Tu es l'orchestrateur d'un système multi-agents.

Ton rôle :
1. Analyser le code dans src/ et le décomposer en 2 ou 3 tâches atomiques
   (indépendantes, < 15 min chacune).
2. Écrire ces tâches dans TASKS.md.
3. Pour CHAQUE tâche, lancer un worker dans une nouvelle fenêtre tmux avec :

   tmux new-window -t multi-agents -n worker-N \
     "codex -m gpt-4o-mini --dangerously-bypass-approvals-and-sandbox \
       'Implémente UNIQUEMENT la Tâche N de TASKS.md.
        Quand fini, ajoute une section ## Tâche N à STATUS.md avec :
        - fichiers modifiés
        - points d''attention pour le reviewer'"

4. Surveiller STATUS.md (re-lire toutes les 30s) jusqu'à ce que toutes
   les tâches y soient marquées comme terminées.
5. Une fois tout fini, lancer un git diff et écrire un REVIEW.md
   (verdict ✅/⚠️/❌, observations, suggestions max 3).

Tu ne tapes JAMAIS de code toi-même. Tu réfléchis et tu délègues.
Commence.
```

## Étape 3 : Observer

Pendant que l'orchestrateur travaille, naviguer entre les fenêtres tmux :

```
Ctrl+B 0    → orchestrateur (Claude)
Ctrl+B 1    → worker-1 (codex gpt-4o-mini)
Ctrl+B 2    → worker-2 (codex gpt-4o-mini)
Ctrl+B 3    → worker-3 (codex gpt-4o-mini)
Ctrl+B w    → liste interactive des fenêtres
```

Vérifier la progression sans interrompre :

```bash
# Depuis n'importe quelle fenêtre
cat TASKS.md
cat STATUS.md
```

## Étape 4 : Récupérer le review

Quand l'orchestrateur a fini :

```bash
cat REVIEW.md
git diff --stat
```

---

# Variante : workers en worktrees pour vraiment isoler

Si les tâches touchent des fichiers communs, demander à l'orchestrateur d'ajouter cette consigne :

```
Avant de lancer chaque worker, crée un worktree git dédié :

  git worktree add ../projet-taskN feature/task-N

Et lance le worker DANS ce worktree :

  tmux new-window -t multi-agents -n worker-N \
    "cd ../projet-taskN && codex -m gpt-4o-mini ..."

À la fin, fusionne les branches feature/task-* dans la branche courante.
```

Plus aucun conflit de fichiers entre workers, même sur du code partagé.

---

# Ce qu'on observe

- Le découpage en tâches atomiques par le gros modèle vaut son prix : un mauvais `TASKS.md` plombe tout le reste.
- Les workers `gpt-4o-mini` sont étonnamment bons sur des tâches **bien cadrées** — mais s'effondrent dès qu'on leur demande de "comprendre l'archi". D'où la division du travail.
- L'orchestrateur qui review du code qu'il n'a pas écrit attrape plus de bugs qu'un agent qui se review lui-même (moins d'angle mort).
- Le coût total est dominé par les tokens du gros modèle (réflexion + review), pas par les workers. Cible : 80% des tokens "code généré" sur le petit modèle.

<!-- ---

# Livrable

À la fin de ce TP :

- [ ] `TASKS.md` généré par l'orchestrateur (≥ 2 tâches atomiques)
- [ ] `STATUS.md` rempli par les workers (une section par tâche)
- [ ] `REVIEW.md` rempli par l'orchestrateur avec un verdict
- [ ] `tmux list-windows -t multi-agents` montre N+1 fenêtres (1 orchestrateur + N workers)
- [ ] L'humain n'a tapé AUCUNE commande `tmux new-window` lui-même -->
