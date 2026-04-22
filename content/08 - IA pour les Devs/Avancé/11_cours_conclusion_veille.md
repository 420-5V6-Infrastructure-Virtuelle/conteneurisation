---
title: "11 - Veille et Écosystème"
weight: 2050
---

## _Rester à jour dans un écosystème en mouvement_

---

# Le rythme effréné

**L'écosystème IA évolue très vite :**

| Période | Changements majeurs |
|---------|---------------------|
| 2022 | ChatGPT launch |
| 2023 | GPT-4, Claude 1, open source explosion |
| 2024 | Claude 3, Gemini, Sora, multimodal |
| 2025 | Claude 3.5 Sonnet, reasoning models, agents |
| 2026 | Context windows 1M+, autonomous agents |

**Résultat :** Ce qui était impossible hier est standard aujourd'hui.

---

# Perspectives critiques 

## Closed-source : enshittification sans préavis

Les modèles closed-source peuvent se dégrader silencieusement entre deux versions — sans changelog, sans notification. Un modèle qui était bon à l'implémentation peut devenir médiocre sur vos cas d'usage sans que vous le sachiez.

Ex: https://github.com/besimple-oss/broccoli

## AI Fluency Index

Selon l'[AI Fluency Index d'Anthropic](https://www.anthropic.com/research/AI-fluency-index), le mode génération d'artefacts (code, documents) crée un effet wow qui réduit l'esprit critique.

Le modèle de chat back-and-forth préserve davantage l'esprit critique.

---

# Les sources de veille

## Agrégateurs et newsletters

| Source | Fréquence | Focus |
|--------|-----------|-------|
| **Hacker News** | Quotidien | Technique, discussions |
| **Lobste.rs** | Quotidien | Technique, moins de bruit |

### Lobste.rs - Communauté technique

- Signal/bruit meilleur que HN
- Communauté plus restreinte, plus technique
- Tags : `llm`, `machine-learning`, `ai`

---

## Pour une veille avancée : LocalLLM

**r/LocalLLm** (Reddit) - La référence pour les modèles locaux :
- Benchmarks en temps réel
- Quantisation, fine-tuning, local inference
- Nouveaux modèles open source (Llama, Mistral, Qwen, etc.)
- Hardware optimisation

**Quand l'utiliser :**
- Vous voulez self-host vos modèles
- Intérêt pour les détails techniques (GGUF, quantisation)
- Tests de performance avant déploiement

---

# Les Providers et leurs produits

---

## Tendances à surveiller

1. **Context windows** : 200k → 1M+ tokens
2. Prix
3. Multimodal image ou non

---

# Modèles frugaux en 2026

## La guerre des prix

| Modèle | Coût/1M input | 
|--------|---------------|------------------|
| Gemini Flash | $0.07 | 
| MiniMax |  |
| NanoFlash | |
| GLM 4.7 | |

=> Routage intelligent selon la tâche.

---



## Les MCP essentiels

| MCP | Usage |
|-----|-------|
| **filesystem** | Accès fichiers |
| **postgres** | Requêtes DB |
| **github** | Issues, PRs |
| **playwright** | Browser automation |
| **slack** | Messages |

---

## DeepWiki : Documentation structurée

**DeepWiki** transforme n'importe quel repo GitHub en documentation navigable :

```
Repo GitHub → DeepWiki → Markdown structuré
```

**Usage :**
- Comprendre un projet open source rapidement
- Chercher des patterns dans une codebase
- Ancrer un agent dans la documentation d'un projet

**Exemple :**
```bash
# DeepWiki pour Next.js
deepwiki fetch "vercel/next.js"
# Retourne un markdown structuré avec:
# - Architecture
# - API publique
# - Patterns utilisés
```

**Dans le workflow IA :**
```yaml
# L'agent utilise DeepWiki pour:
deepwiki_fetch:
  url: "betagouv/comparia"  # ComparIA repo
  # L'agent comprend le projet sans lire tout le code
```

**Quand l'utiliser :**
- Découvrir un projet open source
- Préparer un TP sur une techno inconnue (Rust, Elixir, etc.)
- Documenter les decisions d'architecture

---
# Annexe : Liens de veille à connaître

## Outils de monitoring et d'inspection

- **[claude-devtools](https://github.com/matt1398/claude-devtools)** — Les DevTools manquants pour Claude Code : inspecter les sessions, tool calls, usage de tokens, sous-agents et fenêtre de contexte en UI visuelle.
- **[codeburn](https://github.com/AgentSeal/codeburn)** — Visualise où vont vos tokens session par session (par type de tool call, fichiers lus, etc.). Utile pour identifier ce qui consomme inutilement.
- **[rtk](https://github.com/rtk-ai/rtk)** — Proxy CLI qui réduit la consommation de tokens de 60-90% sur les commandes dev courantes.

## Lectures importantes

- **[HN #47004712](https://news.ycombinator.com/item?id=47004712)** — Discussion HN à lire : retours d'expérience terrain sur l'usage des agents IA.
- **[Reddit ExperiencedDevs — "An AI CEO finally said something honest"](https://www.reddit.com/r/ExperiencedDevs/comments/1r6olcv/an_ai_ceo_finally_said_something_honest/)** — Analyse critique sur le discours des entreprises IA.
- **[Agentic Coding Trends Report 2026](https://resources.anthropic.com/hubfs/2026%20Agentic%20Coding%20Trends%20Report.pdf)** — Rapport Anthropic sur les tendances du coding agentique.
- **[AI Fluency Index](https://www.anthropic.com/research/AI-fluency-index)** — Recherche Anthropic sur l'usage réel de l'IA et le biais des artefacts.

## Répertoires de ressources

- **[Anthropic Skills](https://github.com/anthropics/skills/tree/main/skills)** — Skills officiels Claude Code : simplify, review, security-review, etc.
- **[Claude Code Security Review](https://github.com/anthropics/claude-code-security-review)** — Skill de review sécurité pour Claude Code.
- **[Claude Code Ultimate Guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/guide/cheatsheet.md)** — Cheatsheet community avec bonnes pratiques et quiz.