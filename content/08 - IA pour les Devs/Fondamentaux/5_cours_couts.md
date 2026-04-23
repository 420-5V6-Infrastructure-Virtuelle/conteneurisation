---
title: "5 - Cours Coûts"
weight: 1064
---

## _Utiliser le bon modèle pour la bonne tâche_

> ⏱ **45 min**

> **Outil principal :** Codex CLI. Remplacer `codex` par `opencode` ou `claude` selon votre outil.

---
# Objectif

Comprendre pourquoi un agent ne devrait pas utiliser le même modèle pour planifier et pour coder — et savoir configurer ses modes en pratique.

## La réalité des coûts

Mise en perspective du Hacker News :

| Usage | Coût mensuel | Profil |
|-------|-------------|--------|
| Casual | $10–20 | Copilot, abonnement basique |
| Actif | $40–100 | Cursor + Claude/GPT régulier |
| Power user | $100–700 | Usage API intensif |
| Extrême | $24 000 | Claude Code sans limite (cas réel HN) |

<!-- Ce n'est pas une fatalité — c'est une question de stratégie. -->

**Prix des modèles courants sur OpenRouter (output tokens) :**

| Modèle | Coût / 1M tokens | Pour quoi |
|--------|-----------------|-----------|
| Gemini Flash | ~$0.30 | Tâches mécaniques, exploration |
| Claude Haiku | ~$1.25 | Usage courant |
| Claude Sonnet | ~$15.00 | Architecture, sécurité, décisions |
| Minimax / GLM |  | Usage courant |

La différence entre Flash et Sonnet sur de l'implémentation mécanique : souvent nulle. Sur de l'architecture : souvent décisive.

---

#  Mesurer sa consommation

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

# Comprendre ce qu'on paie

La facturation se fait au token. Un token ≈ 4 caractères en anglais, un peu moins en français.

**Tokens d'entrée (input)**

Tout ce que le modèle lit avant de répondre : le prompt système, l'historique de la conversation, les résultats des tool calls (contenu des fichiers lus, résultats de commandes…). Plus le contexte accumulé est grand, plus chaque échange coûte cher — même si vous ne demandez qu'une petite chose.

**Tokens de sortie (output)**

Tout ce que le modèle génère : texte de réponse, appels d'outils, code produit. Les tokens de sortie coûtent généralement 3 à 5× plus cher que les tokens d'entrée.

**Tokens de raisonnement (reasoning)**

Les modèles "thinking" (o3, claude-sonnet avec extended thinking…) génèrent une chaîne de réflexion interne avant de répondre. Ces tokens comptent comme des tokens de sortie — ils peuvent multiplier le coût par 5 sur une tâche complexe.

```
Input tokens  → $0.003 / 1M tokens  (ex. Claude Sonnet)
Output tokens → $0.015 / 1M tokens
Reasoning     → $0.015 / 1M tokens (inclus dans output)
```

## Où part l'argent en pratique

Sur une session de codage typique, l'essentiel des tokens d'entrée vient des tool calls : l'agent lit des fichiers, exécute des commandes, lit les résultats — tout ça s'accumule dans le contexte.

**Pour minimiser :**

- `/compact` avant que le contexte dépasse 70% — résume l'historique sans le perdre
- Demandez des réponses concises : `Réponds en 3 bullet points max`
- Préférez `grep` à `lire tout le fichier` quand c'est possible — l'agent fait pareil si AGENTS.md le précise

---

#  Plan vs Act — deux phases différentes

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

#  Configurer les phases par outil

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

# Choisir son modèle selon la tâche

| Type de tâche | Exigence | Modèle adapté |
|--------------|----------|---------------|
| Architecture, sécurité, refactoring complexe | Raisonnement fort | Sonnet, o3, Gemini Pro |
| Écriture de code selon un plan | Vitesse, coût | Gemini Flash, Haiku |
| Review de code avec screenshot UI | Vision | Claude Sonnet, Gemini |
| Génération de tests unitaires répétitifs | Coût minimal | Flash, Haiku |
| Debugging d'une erreur obscure | Raisonnement fort | Sonnet, o3 |

**La règle pratique :** frugal par défaut, premium uniquement pour refactoring, architecture, sécurité.

---

#  Le pattern pingre — réflexion gratuite, implémentation frugale

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

<!-- ## Curiosité : Nvidia NIM async

Nvidia propose des modèles open source (Llama, Mistral, etc.) **gratuitement** via [build.nvidia.com](https://build.nvidia.com), mais en mode asynchrone — jusqu'à 3h d'attente en période de charge. Inutilisable en session interactive, mais intéressant pour des tâches batch overnight. -->

---

---

# Étape 8 : Plans, limites et cache

## Définir ses limites de dépense

Sur **OpenRouter**, configurez des limites avant de lancer un agent autonome :

- **Limite quotidienne** : coupe la clé si vous dépassez X$ en 24h — indispensable avant un Ralph Loop
- **Limite hebdomadaire** : plafond global pour éviter les surprises de fin de semaine
- **Limite par requête** : force l'agent à rester concis

> Réglage dans le dashboard OpenRouter → Settings → Limits. Fixez une limite quotidienne dès l'installation — pas après le premier incident.

## Token limit par requête

Certains outils permettent de limiter les tokens de sortie par appel :

```bash
# Codex CLI — via variable d'environnement
export OPENAI_MAX_TOKENS=4096

# OpenCode — dans config.yaml
max_tokens: 4096
```

Un token limit trop bas casse les réponses longues. Un token limit absent laisse l'agent produire 10 000 tokens pour une réponse de 50 lignes.

## Le cache de prompt

Certains providers (Anthropic, OpenAI) permettent de mettre en cache les tokens d'entrée répétitifs. Résultat : si vous relancez une session avec le même AGENTS.md + les mêmes fichiers, les tokens déjà vus ne sont pas refacturés au plein tarif.

**Chez Anthropic :** tokens mis en cache coûtent ~10% du prix normal à la relecture (après 5 min de TTL).

```
Session 1 : 10 000 tokens d'entrée → $0.03
Session 2 (même contexte) : 10 000 tokens → $0.003 (cache hit)
```

**Quand c'est utile :** sessions longues sur le même projet, Ralph Loop avec AGENTS.md stable.

**Quand ça n'aide pas :** prompts qui changent à chaque fois, modèles sans support du cache (la plupart des modèles frugaux sur OpenRouter).

---

# Étape 9 : Visualiser sa consommation avec codeburn

[codeburn](https://github.com/AgentSeal/codeburn) est un dashboard TUI qui lit les fichiers de session de vos outils IA directement sur disque — sans wrapper, sans clé API — et vous montre où partent vos tokens : par projet, par modèle, par type d'activité.

## Installation

```bash
npm install -g codeburn
# Ou sans installer :
npx codeburn
```

Prérequis : Node.js 22+.

## Exercice

**1. Lancer le dashboard sur vos 7 derniers jours :**

```bash
codeburn
```

Naviguez avec les flèches pour changer la période. Tapez `c` pour comparer les modèles, `o` pour les suggestions d'optimisation.

**2. Résumé rapide en une ligne :**

```bash
codeburn status
```

**3. Identifier les commandes qui brûlent le plus de tokens :**

```bash
codeburn optimize
```

Repérez les tool calls inutilement coûteux (lecture de fichiers entiers, commandes verbose…).

**4. Export pour archiver :**

```bash
codeburn report -p 30days
codeburn export
```

**Ce que vous cherchez :** quel outil / quel projet consomme le plus ? Y a-t-il des pics inexpliqués ? Les suggestions `optimize` correspondent-elles à ce que vous observez ?

---

# Étape 10 : Réduire sa consommation avec rtk

[rtk](https://github.com/rtk-ai/rtk) est un proxy CLI qui filtre et compresse les sorties de commandes avant qu'elles n'entrent dans le contexte de votre agent — 60 à 90% de tokens en moins sur les commandes courantes (`git status`, `ls`, `diff`, `pytest`…).

## Installation

```bash
# Linux/macOS :
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh

# macOS via Homebrew :
brew install rtk

# Vérifier :
rtk --version
```

## Initialisation

```bash
# Pour Claude Code :
rtk init -g

# Pour un autre outil :
rtk init --gemini       # Gemini CLI
rtk init --copilot      # GitHub Copilot
rtk init --agent cursor # Cursor
```

Redémarrez votre outil IA après l'init — les commandes se réécrivent automatiquement via un hook Bash.

## Exercice

**1. Comparer la sortie brute vs. filtrée :**

Dans un repo avec de l'activité Git :

```bash
# Sortie brute :
git status

# Sortie filtrée pour l'agent :
rtk git status
```

Observez la différence de volume. Faites pareil avec `rtk git log` et `rtk git diff`.

**2. Lancer une session avec rtk actif :**

Lancez votre agent sur une tâche courante (ajout d'une feature, correction d'un bug). Comparez le coût affiché avec une session équivalente sans rtk.

**3. Voir les économies accumulées :**

```bash
rtk gain
```

**4. Découvrir les opportunités d'optimisation :**

```bash
rtk discover
```

**Commandes disponibles :**

| Catégorie | Exemples |
|-----------|---------|
| Fichiers | `rtk ls`, `rtk read`, `rtk find`, `rtk grep`, `rtk diff` |
| Git | `rtk git status/log/diff/push/pull` |
| Tests | `rtk pytest`, `rtk jest`, `rtk cargo test` |
| Build/Lint | `rtk tsc`, `rtk ruff check`, `rtk cargo build` |
| Containers | `rtk docker ps`, `rtk kubectl pods` |

**Ce que vous cherchez :** sur quel type de commande le gain est-il le plus fort ? Est-ce que la sortie filtrée reste suffisante pour que l'agent travaille correctement ?

---

# Livrable

À la fin de ce TP :

- [ ] Mesuré sa consommation sur une tâche réelle
- [ ] Identifié deux tâches : une "Plan" (raisonnement), une "Act" (exécution)
- [ ] Configuré un profil frugal dans son outil
- [ ] Essayé le pattern pingre (Gemini gratuit → modèle cheap)
- [ ] Visualisé sa consommation avec codeburn (`codeburn optimize`)
- [ ] Réduit les tokens d'une session avec rtk (`rtk gain`)

---

# Checkpoint

**Question clé :** Pour quelle tâche d'aujourd'hui auriez-vous pu utiliser un modèle moins cher ?

**Pattern retenu :** Phase Plan = raisonnement fort. Phase Act = modèle frugal.

---

# Prochain module

Module 6 : Multimodal - screenshots, images, et au-delà.
