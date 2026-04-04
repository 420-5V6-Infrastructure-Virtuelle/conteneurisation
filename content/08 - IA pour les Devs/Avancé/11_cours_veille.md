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

# Les sources de veille

## Agrégateurs et newsletters

| Source | Fréquence | Focus |
|--------|-----------|-------|
| **Hacker News** | Quotidien | Technique, discussions |
| **The AI Epoch** | Hebdo | Agrégateur |
| **Decoder** | Quotidien | News IA |
| **Alpha Signal** | Hebdo | ML/Research |

## Comptes à suivre

| Compte | Plateforme | Intérêt |
|--------|-------------|----------|
| @karpathy | Twitter | ML deep dives |
| @simonw | Blog | Outils pratiques |
| @svpino | Twitter | Engineering IA |
| Official accounts | Twitter | Claude, OpenAI, etc. |

---

# Les Providers et leurs produits

## État des lieux 2026

| Provider | Modèle phare | Usage |
|----------|--------------|-------|
| **Anthropic** | Claude 3.5 Sonnet | Code, reasoning |
| **OpenAI** | GPT-4o, o1 | General, reasoning |
| **Google** | Gemini Pro | Multimodal, frugal |
| **Meta** | Llama 4 | Open source |
| **DeepSeek** | V3 | Frugal, coding |
| **Mistral** | Mistral Large | European, open |

---

## Tendances à surveiller

1. **Context windows** : 200k → 1M+ tokens
2. **Reasoning models** : o1, Claude thinking
3. **Multimodal complet** : Text + image + audio
4. **Autonomous agents** : Ralph loop, self-healing
5. **Cost collapse** : $/token en chute libre

---

# Modèles frugaux en 2026

## La guerre des prix

| Modèle | Coût/1M input | Quand l'utiliser |
|--------|---------------|------------------|
| Gemini Flash | $0.07 | Draft, brainstorming |
| DeepSeek V3 | $0.10 | Coding, math |
| Claude Haiku | $0.25 | Quick tasks |
| GPT-4o-mini | $0.15 | General |

**Stratégie :** Routage intelligent selon la tâche.

---

# Outils à surveiller

## TUI Agents

| Outil | Statut | Particularité |
|-------|--------|---------------|
| **OpenCode** | Actif | Open source, agnostique |
| **Claude Code** | Actif | Vendor lock-in |
| **Cursor** | Actif | IDE intégré |
| **Aider** | Actif | CLI lightweight |
| **Goose** | Nouveau | Open source |

---

## IDE Assistants

| Outil | Statut | Particularité |
|-------|--------|---------------|
| **Copilot** | Mature | IDE intégré |
| **Cursor** | Populaire | Fork VSCode |
| **Cline** | Actif | Extension VSCode |
| **Roo Code** | Fork | Plus de flexibilité |

---

## PR Bots

| Outil | Statut | Particularité |
|-------|--------|---------------|
| **Jules** | Google | Background PR work |
| **CodeRabbit** | Commercial | Review automatique |
| **Copilot for PR** | GitHub | Suggestions |

---

# MCP Ecosystem

## Les MCP essentiels

| MCP | Usage |
|-----|-------|
| **filesystem** | Accès fichiers |
| **postgres** | Requêtes DB |
| **github** | Issues, PRs |
| **playwright** | Browser automation |
| **slack** | Messages |

---

## Comment suivre les MCP

- [MCP Registry](https://github.com/modelcontextprotocol/registry)
- Awesome MCP lists
- Communautés Discord/Slack

---

# Techniques de veille

## Daily workflow

```markdown
## Matin (15 min)
- [ ] Hacker News front page
- [ ] Twitter lists (AI twitter)
- [ ] Discord serveurs actifs

## Semaine (1h)
- [ ] Newsletter(s) en profondeur
- [ ] Un article technique

## Mois (2h)
- [ ] Un paper en détail
- [ ] Test d'un nouvel outil
```

---

## Filtrer le bruit

**Ce qui compte :**
- Nouveaux modèles
- Nouveaux outils
- Failures et lessons learned
- Techniques pratiques

**Ce qui peut attendre :**
- News financières (funding, acquisitions)
- Spéculation sur l'avenir
- Hype sans substance

---

# Partager en équipe

## Créer un canal veille

```markdown
# Slack/Discord : #veille-ia

## Format recommandé
[Lien]
 TL;DR : 1-2 phrases
 Pourquoi ça compte : 1-2 phrases

## Rotation
- Chaque membre partage 1 chose/semaine
- Pas de spam, que le meilleur
```

---

# Anticiper les changements

## Les signaux

| Signal | Implication |
|--------|-------------|
| Nouveau modèle open source | Possibilité de self-host |
| Hausse de contexte window | Plus de code en une fois |
| Nouveau MCP | Nouvelles capacités agents |
| Faille de sécurité | Mise à jour urgente |

---

# TP : Veille technologique

Voir `11_tp_veille.md` →