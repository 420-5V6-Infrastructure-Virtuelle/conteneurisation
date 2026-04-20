---
title: "5 - TP Modes et Modèles"
weight: 1065
---

## _Utiliser le bon modèle pour la bonne tâche_

> ⏱ **45 min**

> **Outil principal :** Codex CLI. Remplacer `codex` par `opencode` ou `claude` selon votre outil.

---

# Objectif

Comprendre pourquoi un agent ne devrait pas utiliser le même modèle pour planifier et pour coder — et savoir configurer ses modes en pratique.

---

# Étape 1 : Mesurer sa consommation

Chaque outil expose sa consommation différemment — utilisez l'option native plutôt qu'un pipe.

**Codex CLI** — affichage intégré dans le TUI, résumé tokens/coût à la fin de chaque session.

**OpenCode** — mode verbose :
```bash
opencode --verbose
```
Tokens et coût apparaissent dans les logs après chaque échange.

**Claude Code** — statusline en temps réel (configurée en TP1) + verbose :
```bash
claude --verbose
```

**Pour aller plus loin — outils tiers :**

- **[claude-devtools](https://github.com/matt1398/claude-devtools)** — UI visuelle pour inspecter les sessions Claude Code : tool calls, token usage, sous-agents, fenêtre de contexte.
- **[codeburn](https://github.com/AgentSeal/codeburn)** — visualise où partent vos tokens par type de tool call. Utile pour repérer ce qui consomme inutilement.

**Travailler sur une tâche :**

```
> Add pagination to the GET /users endpoint
```

Notez les tokens d'entrée, de sortie, et le coût estimé affiché.

---

# Étape 2 : Plan vs Act — deux phases différentes

Un agent qui reçoit "Refactor the auth service" fait en réalité deux choses très différentes :

**Phase Plan**
- Comprendre le codebase et ses contraintes
- Identifier les impacts sur les autres modules
- Décider de l'architecture
- Décomposer en étapes exécutables

→ Tâche cognitive dense. Un modèle avec un fort raisonnement (Claude Sonnet, o3, Gemini Pro) fait ici une vraie différence.

**Phase Act**
- Écrire le code selon le plan établi
- Refactorer fichier par fichier
- Générer des tests boilerplate
- Appliquer les conventions mécaniquement

→ Tâche répétitive et prévisible. Un modèle cheap et rapide (Gemini Flash, Haiku) suffit largement.

**L'insight :** payer le modèle cher uniquement pour la réflexion, pas pour l'exécution mécanique.

---

# Étape 3 : Configurer les phases par outil

**Codex CLI** — switcher de modèle entre les phases :

```bash
# Phase Plan : modèle avec forte capacité de raisonnement
OPENAI_MODEL="anthropic/claude-3.5-sonnet" codex "Analyse l'architecture auth et propose un plan de refactoring. Pas de code."

# Phase Act : modèle frugal pour l'implémentation
OPENAI_MODEL="google/gemini-flash-1.5" codex "Implémente ce plan : [coller le plan]"
```

**OpenCode** — deux profils dans config.yaml :

```yaml
# ~/.config/opencode/config.yaml
models:
  plan:
    model: anthropic/claude-3.5-sonnet
  act:
    model: google/gemini-2.0-flash
```

Sélectionnez le profil selon la phase en cours.

**Claude Code** — Plan mode et switch de modèle :

```
Shift+Tab   # Active le Plan mode : l'agent réfléchit avant d'agir
```

```bash
# Changer de modèle en cours de session :
/model claude-haiku-4-5   # Passer en frugal pour l'implémentation
```

---

# Étape 4 : Choisir son modèle selon la tâche

| Type de tâche | Exigence | Modèle adapté |
|--------------|----------|---------------|
| Architecture, sécurité, refactoring complexe | Raisonnement fort | Sonnet, o3, Gemini Pro |
| Écriture de code selon un plan | Vitesse, coût | Gemini Flash, Haiku |
| Review de code avec screenshot UI | Vision | Claude Sonnet, Gemini |
| Génération de tests unitaires répétitifs | Coût minimal | Flash, Haiku |
| Debugging d'une erreur obscure | Raisonnement fort | Sonnet, o3 |

**La règle pratique :** frugal par défaut, premium uniquement pour refactoring, architecture, sécurité.

---

# Étape 5 : Le pattern pingre — réflexion gratuite, implémentation frugale

L'idée : utiliser un modèle **gratuit** pour la phase Plan, puis fournir ce plan à un modèle **ultra-frugal** pour l'implémentation mécanique.

**Phase Plan — Gemini Pro gratuit via Google AI Studio**

[Google AI Studio](https://aistudio.google.com) offre un plan gratuit généreux sur Gemini Pro (rate limits, pas d'usage commercial, mais parfait pour la réflexion) :

```
> Analyse ce besoin et propose un plan d'implémentation détaillé
  pour ajouter [feature] à une API FastAPI.
  Liste les fichiers à modifier, les étapes, les risques.
  Ne génère pas de code.
```

**Phase Act — modèle frugal sur OpenRouter**

Copiez le plan dans votre agent configuré sur un modèle cheap :

```bash
OPENAI_MODEL="google/gemini-flash-1.5" codex "Voici le plan validé : [coller le plan]. Implémente étape par étape."
```

**Résultat :** la partie coûteuse (raisonnement, architecture) est gratuite ; la partie mécanique coûte quasi-rien.

## Curiosité : Nvidia NIM async

Nvidia propose des modèles open source (Llama, Mistral, etc.) **gratuitement** via [build.nvidia.com](https://build.nvidia.com), mais en mode asynchrone — jusqu'à 3h d'attente en période de charge. Inutilisable en session interactive, mais intéressant pour des tâches batch overnight.

---

# Étape 6 : Seuils de contexte à surveiller

Le contexte consommé, c'est aussi des tokens payants. Gardez un œil sur `Ctx(u)` dans la statusline (configurée en TP1) :

| Contexte % | État | Action |
|------------|------|--------|
| 0–50% | Vert | Travaillez librement |
| 50–70% | Jaune | Soyez sélectif dans les lectures |
| 70–90% | Orange | `/compact` maintenant |
| 90%+ | Rouge | `/clear` requis |

Chaque token non consommé est un token économisé.

---

# Livrable

À la fin de ce TP :

- [ ] Mesuré sa consommation sur une tâche réelle
- [ ] Identifié deux tâches : une "Plan" (raisonnement), une "Act" (exécution)
- [ ] Configuré un profil frugal dans son outil
- [ ] Essayé le pattern pingre (Gemini gratuit → modèle cheap)

---

# Checkpoint

**Question clé :** Pour quelle tâche d'aujourd'hui auriez-vous pu utiliser un modèle moins cher ?

**Pattern retenu :** Phase Plan = raisonnement fort. Phase Act = modèle frugal.

---

# Prochain module

Module 6 : Multimodal - screenshots, images, et au-delà.
