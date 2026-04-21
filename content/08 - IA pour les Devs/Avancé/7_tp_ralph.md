---
title: "7 - TP Sandboxing & Exécution Autonome"
weight: 2015
---

## _Faire tourner un agent sans supervision sans se tirer une balle dans le pied_

> ⏱ **1h30**

---

# Le problème : autonomie = surface d'attaque

Pour qu'un agent tourne sans cliquer "oui" à chaque action, il faut lui donner les clés. Mais lui donner les clés, c'est aussi lui donner la capacité de tout casser.

## Pourquoi les incidents arrivent

| Incident | Impact |
|----------|--------|
| Deux agents LangChain qui se chattaient en boucle | **$47 000** sur 11 jours |
| Agent qui ignore la commande STOP | 9,6 M emails supprimés |
| Copilot crée des worktrees en boucle | 1 526 worktrees, 800 Go sur disque |
| Agent Terraform sans supervision | 2,5 ans de données perdues |

Dans chaque cas : l'agent avait trop de permissions et pas de cage.

---

# Skipper les permissions : comment ça marche

## Claude Code

```bash
# Mode interactif normal — Claude demande avant chaque action sensible
claude

# Skipper TOUTES les permissions — à n'utiliser qu'en sandbox
claude --dangerously-skip-permissions

# En mode non-interactif (pour scripts et boucles)
claude -p "$(cat TASK.md)" --dangerously-skip-permissions
```

`--dangerously-skip-permissions` approuve automatiquement : lecture/écriture de fichiers, exécution de commandes shell, appels réseau. Le nom est volontairement alarmant.

## OpenAI Codex CLI

```bash
# Mode suggestion (défaut) — demande approbation à chaque action
codex "ajoute la pagination"

# Auto-edit — approuve les modifications de fichiers, demande pour les commandes shell
codex --approval-mode auto-edit "$(cat TASK.md)"

# Full-auto — approuve tout, y compris les commandes shell arbitraires
codex --approval-mode full-auto "$(cat TASK.md)"
```

`full-auto` ne s'utilise qu'à l'intérieur d'un sandbox. Jamais sur votre machine principale.

---

# Le pattern tmux

Les agents autonomes peuvent tourner des heures. tmux vous donne des sessions persistantes qui survivent aux déconnexions — indispensable pour superviser sans bloquer.

```bash
# Créer une session dédiée
tmux new -s agent

# Lancer l'agent dans la session
claude --dangerously-skip-permissions -p "$(cat TASK.md)"

# Détacher sans tuer la session : Ctrl+B puis D

# Revenir plus tard
tmux attach -t agent

# Voir toutes les sessions actives
tmux ls
```

## Deux agents en parallèle avec worktrees

Au lieu de deux branches sur le même checkout, créez un worktree par agent — chacun a son propre filesystem, zéro conflit.

```bash
# Créer un worktree pour la feature B
git worktree add ../project-feature-b feature/feature-b

# Agent A : dossier principal, session tmux dédiée
tmux new -s agent-a
# Dans agent-a :
# cd ~/project && claude --dangerously-skip-permissions -p "$(cat TASK_A.md)"

# Agent B : worktree isolé, autre session
tmux new -s agent-b
# Dans agent-b :
# cd ~/project-feature-b && claude --dangerously-skip-permissions -p "$(cat TASK_B.md)"
```

Les deux agents travaillent en simultané sur des fichiers distincts — pas de `git stash`, pas de `git checkout`.

---

# Les niveaux de sandbox

Whitelister les outils un par un dans les settings est fastidieux — et incomplet. L'approche correcte : isoler l'agent dans un environnement où même s'il déraille, les dégâts restent contenus.

| Niveau | Mécanisme | Ce que ça protège | Effort | Quand l'utiliser |
|--------|-----------|-------------------|--------|-----------------|
| **Soft : AGENTS.md / CLAUDE.md** | Instructions texte | Rien — l'agent peut ignorer | Minimal | Toujours, mais jamais seul |
| **Built-in sandbox** | `--sandbox` (Claude), mode Codex natif | Filesystem (partiel) | Minimal | Exploratoire, dev local |
| **Session Linux** | User dédié sans sudo | Filesystem hors projet | Moyen | Serveur, setup permanent |
| **Docker container** | Container isolé, `--network none` | Filesystem + réseau | Moyen | CI/CD, agents longue durée |
| **Org level** | Politiques GitHub, RBAC | Accès ressources externes | Élevé | Équipes, production |

**Règle de base :** au minimum Docker ou user Linux dédié dès qu'on utilise `--dangerously-skip-permissions` ou `full-auto`.

---

# Partie 1 : Sandbox avec un user Linux dédié

```bash
# Créer un user sans sudo
sudo useradd -m -s /bin/bash agentuser
sudo chown -R agentuser:agentuser /path/to/project

# Lancer l'agent en tant que agentuser
sudo -u agentuser bash
cd /path/to/project
claude --dangerously-skip-permissions -p "$(cat TASK.md)"
```

L'agent ne peut pas toucher à `~`, `/etc`, `/usr`, ni lire les credentials dans `~/.ssh` ou `~/.aws`.

**Limite :** si votre projet contient des secrets (`.env`), l'agent peut les lire. Sortez-les du projet ou passez à Docker.

---

# Partie 2 : Sandbox Docker

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN useradd -m agentuser
USER agentuser

COPY --chown=agentuser:agentuser . .
RUN pip install --user -r requirements.txt

CMD ["bash"]
```

```bash
docker build -t agent-sandbox .
docker run -it --rm \
  --network none \
  -v $(pwd)/output:/app/output \
  agent-sandbox \
  bash -c "claude --dangerously-skip-permissions -p '$(cat TASK.md)'"
```

`--network none` est le flag le plus important : l'agent ne peut pas exfiltrer de données, appeler des APIs externes, ni télécharger de packages.

---

# Partie 3 : Soft guardrails — AGENTS.md / CLAUDE.md

Les guardrails texte ne sont **pas** une protection de sécurité — ce sont des instructions de comportement. Un agent respecte les bonnes intentions, pas les contraintes dures.

Leur valeur : cadrer le comportement nominal, documenter les contraintes d'équipe.

```markdown
# AGENTS.md

## Scope
- Modifier uniquement les fichiers dans src/ et tests/
- Ne jamais supprimer de fichiers — déplacer vers .archive/ si nécessaire
- Ne jamais modifier .env ou tout fichier contenant des secrets

## Shell Commands
- Ne jamais exécuter de commandes système (apt, pip install --system)
- Ne jamais pusher directement sur main — toujours créer une branche

## Context Management
- Si le contexte dépasse 70%, exécuter /compact avant de continuer
- Committer après chaque phase majeure (Plan, Build, Test)
```

**Quand AGENTS.md suffit :** dev local supervisé, vous regardez les outputs régulièrement.

**Quand ça ne suffit pas :** `full-auto` sur tâche longue, agents parallèles, CI/CD automatisé.

---

# Partie 4 : Org-level controls


## GitHub

```bash
# Token GitHub fine-grained — lecture seule sur le repo
# L'agent peut lire le code mais pas pusher
export GITHUB_TOKEN="github_pat_read_only_xxx"
```

- Branch protection rules : l'agent ne peut pas pusher sur `main`
- Fine-grained tokens : limiter les repos et les permissions
- Environments avec required reviewers : les deploys passent par un humain

**Pattern recommandé pour CI/CD :** token read-only pour analyse, PR ouverte automatiquement, merge manuel obligatoire.

---

# Livrable

À la fin de ce TP :

- [ ] Avoir lancé un agent avec `--dangerously-skip-permissions` dans un container Docker
- [ ] Avoir testé le pattern tmux pour superviser un agent longue durée
- [ ] Avoir un `AGENTS.md` avec des contraintes de scope claires
- [ ] Savoir choisir le bon niveau de sandbox pour un use case donné

---

# Checkpoint

**Règle retenue :** `--dangerously-skip-permissions` et `full-auto` ne s'utilisent qu'à l'intérieur d'un sandbox. Le niveau minimum viable est un user Linux dédié ou Docker.

**Question clé :** Pour votre projet, quel niveau de sandbox est réaliste à mettre en place aujourd'hui ?

---

# Prochain module

Module 8 : Debugging IA — reconnaître et corriger les échecs.
