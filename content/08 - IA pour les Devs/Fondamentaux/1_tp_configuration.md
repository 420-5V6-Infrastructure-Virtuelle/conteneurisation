---
title: "1 - TP Configuration et Premier Pas"
weight: 1025
---

## _Mise en route de l'environnement_

> ⏱ **45 min**

> **Outil principal :** Codex CLI. Les variantes OpenCode et Claude Code sont notées où elles diffèrent.
<!-- 
---

# L'écosystème en 3 minutes

Trois catégories d'outils, un seul principe : **apportez votre propre clé API**.

| Catégorie | Exemples | Usage |
|-----------|----------|-------|
| **Agents TUI** (terminal) | Codex, OpenCode, Claude Code | Session autonome dans le projet |
| **Assistants IDE** | Cursor, Copilot, Cline | Autocomplete + chat dans l'éditeur |
| **Bots PR** | Jules, CodeRabbit | Revue automatique sur les Pull Requests |

Cette formation se concentre sur les agents TUI — les plus puissants pour coder et les plus transparents sur ce qu'ils font.

**Pourquoi une clé OpenRouter plutôt qu'un abonnement ?** Accès à 200+ modèles sans vendor lock-in : frugal (Gemini Flash) comme premium (Claude Sonnet). Vous changez de modèle sans changer d'outil. -->

---

# Prérequis

- [ ] Docker & docker-compose
- [ ] Git configuré
- [ ] Un éditeur de code (VSCode recommandé)
- [ ] La clé OpenRouter fournie par le formateur

---

# Étape 1 : Configurer votre clé OpenRouter

Vous avez reçu une clé OpenRouter (`sk-or-v1-...`). Exportez-la :

```bash
export OPENROUTER_API_KEY="sk-or-v1-..."
# Ajouter à ~/.bashrc ou ~/.zshrc pour la rendre persistante
```

**Pourquoi OpenRouter ?**
- Accès à 200+ modèles via une seule clé
- Frugal (Gemini Flash ~$0.10/1M) comme premium (Claude Sonnet ~$3/1M)
- Pas de vendor lock-in

---

# Étape 2 : Installation

- **Codex CLI :**

```bash
npm install -g @openai/codex
codex --version
```

- **OpenCode :** <https://github.com/opencode-ai/opencode>

-  **Claude Code :** `npm install -g @anthropic-ai/claude-code`

---

# Étape 3 : Configuration

**Codex CLI** se configure via variables d'environnement :

```bash
export OPENAI_API_KEY="${OPENROUTER_API_KEY}"
export OPENAI_BASE_URL="https://openrouter.ai/api/v1"
export OPENAI_MODEL="" 
```

> **OpenCode :** fichier `~/.config/opencode/config.yaml`
> ```yaml
> providers:
>   openrouter:
>     api_key: ${OPENROUTER_API_KEY}
>     base_url: https://openrouter.ai/api/v1
> default_provider: openrouter
> default_model: google/gemini-2.0-flash
> ```
>
> **Claude Code :** via `~/.claude/settings.json` ou `ANTHROPIC_API_KEY` pour usage direct Anthropic.

---

# Choisir son app de travail

Comparia est l'app démo de cette formation, mais vous pouvez appliquer les mêmes exercices à votre propre projet.

---

# Étape 4 : Cloner et faire marcher Comparia

**[Comparia](https://github.com/betagouv/comparia)** est un outil de comparaison de modèles d'IA développé par beta.gouv.fr.

```bash
git clone https://github.com/betagouv/comparia
cd comparia
```

**Configurer le token OpenRouter** (fourni par le formateur) :

```bash
export OPENROUTER_TOKEN="hf_..."
```

**Lancer l'app avec l'aide de l'agent :**

```bash
codex
```

```
> Arrive à lancer ce projet en local
```

**Vérification :** l'interface est accessible dans le navigateur.


---
<!-- 
# Livrable

- [ ] Clé OpenRouter configurée
- [ ] Agent installé et fonctionnel (`codex`, `opencode` ou `claude`)
- [ ] Comparia qui tourne en local
- [ ] Token HuggingFace configuré

--- -->

# Ressources

- [Comparia — betagouv](https://github.com/betagouv/comparia)
- [Documentation OpenRouter](https://openrouter.ai/docs)
- [Codex CLI GitHub](https://github.com/openai/codex)
- [OpenCode GitHub](https://github.com/opencode-ai/opencode)
