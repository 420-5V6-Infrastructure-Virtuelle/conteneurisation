---
title: "4 - TP Délégation entre agents et skills"
weight: 1055
draft: false
---

## _Plan → Build → Review → Retro_

> ⏱ **1h**

<!-- > **Outil principal :** Codex CLI. Les commandes spécifiques à Claude Code sont signalées. -->

---


# Les 4 piliers

Un projet bien structuré pour l'IA a besoin de :

1. **AGENTS.md** - Contexte permanent pour l'agent
2. **Makefile / run.sh** - Commandes reproductibles
3. **Docker** - Environnement isolé et défini
4. **README.md par dossier** - Documentation

---

# Skills & Délégation multi-agents

La plupart des outils proposent d'utiliser plusieurs agents préconfigurés pour certaines tâches, appelés "skill", "mode", "agent" ou "workflow" selon les outils. Dans la pratique, cela revient surtout à spécifier un (pré-)prompt particulier pour qu'un agent classique se concentre sur une problématique particulière. On peut ainsi créer des workflows plus complexes en demandant à un agent de déléguer des sous-tâches à un autre agent. Pour ce TP par exemple : un agent planifie, puis un autre exécute le plan, et un dernier enlève les parties inutiles : le "slop".

## Créer un skill dans OpenCode

Un skill est un dossier contenant un fichier `SKILL.md`. OpenCode cherche les skills dans :
- **Projet :** `.opencode/skills/<nom>/SKILL.md`
- **Global :** `~/.config/opencode/skills/<nom>/SKILL.md`

Format du fichier :

```markdown
---
name: mon-skill
description: Ce que fait ce skill (utilisé par l'agent pour décider de l'activer)
---

## Instructions

Ce que l'agent doit faire quand ce skill est activé.
```

L'agent voit les skills disponibles et les charge à la demande via son outil `skill`. L'invocation est automatique si la tâche correspond à la description du skill.

## Créer un skill dans Codex CLI

Même principe : un dossier avec `SKILL.md`, placé dans `.agents/skills/` à la racine du repo (ou `~/.agents/skills/` pour usage global).

```
.agents/skills/mon-skill/SKILL.md
```

Format identique :

```markdown
---
name: mon-skill
description: Ce que fait ce skill
---

Instructions pour Codex.
```

**Invocation explicite :** tapez `$mon-skill` dans votre prompt.  
**Invocation implicite :** Codex active automatiquement le skill si votre tâche correspond à sa description.

Pour créer un skill interactivement : lancez `$skill-creator`.

---

# Objectif

Réaliser une feature non triviale sur Comparia en suivant le cycle complet : plan → build → PR review → retro AGENTS.md.

C'est le payoff de tous les TPs précédents : AGENTS.md, Makefile, prompts structurés — tout ça ensemble.

---

# Choisir la feature

Quelque chose qui touche plusieurs fichiers et nécessite au moins un test :
- Export CSV des comparaisons
- Historique des sessions avec persistance
- Mode "personnage" persistant entre les messages (reprend l'idée de TP2 côté backend)
- **Une seule conversation** — Comparia affiche actuellement deux conversations en parallèle. Simplifier à une seule réduit la surface d'état côté frontend et backend, sans retirer la comparaison (on peut conserver les deux modèles côte à côte sur un même échange).
- **Historique côté client** — Sauvegarder les comparaisons dans le localStorage pour les retrouver après rechargement.

<!-- Autres idées solides :
- Export de la comparaison en Markdown / JSON (un bouton, un endpoint)
- Partager une comparaison via URL (sérialiser l'état dans les query params)
- Épingler un modèle favori par utilisateur (localStorage)
-->

<!-- Idées plus incertaines :
- Système de notation par réponse (thumbs up/down) — nécessite du persistance côté serveur
- Bibliothèque de prompts système réutilisables
- Comparaison à 3 modèles (UI non triviale)
-->

---

# Phase 1 — PLAN (pas de code encore)

```
> Je veux implémenter [feature] dans Comparia.
  Analyse le projet et propose un plan :
  - Quels fichiers créer ou modifier ?
  - Quelle est l'architecture proposée ?
  - Quels sont les risques ?
  Ne commence pas à coder. Attends ma validation.
```

Lisez le plan, questionnez les choix. Validez ou demandez des ajustements avant de passer à la suite.

---

# Phase 2 — BUILD

```
> Plan validé. Implémente étape par étape.
  Lance make test après chaque étape.
  Commits atomiques.
```

---

# Phase 3 — Cleanup
S'assurer que l'agent lance lui-même le skill "Nettoyage de code".


**Codex / OpenCode :**
```
> Passe en revue le code qu'on vient d'écrire.
  Retire les abstractions superflues, les fonctions intermédiaires inutiles.
  Ne change pas le comportement.
```

**Claude Code :**
```
> /simplify
```

---
<!-- 
# Phase 4 — Review

<!-- ```bash
git checkout -b feature/[nom-de-la-feature]
git push -u origin feature/[nom-de-la-feature]
gh pr create --title "[feat] description" --body "..."
``` -->
<!-- 
**Claude Code :**
```
> /review
``` 

**Codex / OpenCode :**
```
> Review cette PR comme un senior dev sceptique.
  Identifie les problèmes avant que ça parte en prod.
```
 -->

---

<!-- # Phase 5 — Retro → AGENTS.md

```
> Résume ce qu'on vient de faire. Qu'est-ce qui t'a manqué comme contexte ?
  Je veux mettre à jour AGENTS.md.
```

L'agent identifie les lacunes de contexte rencontrées. Vous les ajoutez à `AGENTS.md` — la prochaine session repart d'une base plus solide.

**Ce qu'on ajoute typiquement :**
- Contraintes oubliées ("NE PAS modifier les migrations")
- Patterns récurrents du projet ("toujours utiliser le service layer")
- Outils disponibles qu'il ne connaissait pas

--- -->
<!-- 
# Livrable

- [ ] Feature implémentée et testée (`make test` passe)
- [ ] PR créée avec review documentée
- [ ] `AGENTS.md` mis à jour après retro

---

# Checkpoint

**Pattern retenu :** Plan d'abord, code ensuite, retro toujours.

**Question clé :** Qu'est-ce que la retro a révélé que votre AGENTS.md ne couvrait pas ? -->

---
Ressources :

- <https://opencode.ai/docs/skills/>
- <https://developers.openai.com/codex/skills>

---

# Prochain module

Module 5 : Coûts et modèles frugaux.
