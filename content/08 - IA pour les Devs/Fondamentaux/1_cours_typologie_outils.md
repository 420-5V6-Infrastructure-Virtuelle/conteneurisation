---
title: "1 - Typologie des Outils IA"
weight: 1020
---

## _Comprendre l'écosystème sans se perdre_

---

# La clé : votre propre API key

**Le point essentiel : tous les outils se valent.**

Ce qui compte : **apporter votre propre clé API** (via OpenRouter, OpenAI, Anthropic...) et utiliser n'importe quel outil.

```
┌─────────────────────────────────────────────────────────────┐
│                    Votre API Key                            │
│                  (OpenRouter, Claude, etc.)                 │
└─────────────────────┬───────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   ┌─────────┐   ┌─────────┐   ┌─────────┐
   │OpenCode │   │Claude   │   │Cursor   │
   │  (TUI)  │   │Code(TUI)│   │ (IDE)   │
   └─────────┘   └─────────┘   └─────────┘
```

**Avantage : pas de vendor lock-in.** Vous changez d'outil sans changer de modèle.

---

# Typologie par catégorie

Ne choisissez pas un outil individuel - comprenez les **catégories**.

## Type 1 : Agents TUI (Terminal User Interface)

**Agents quasi-autonomes dans le terminal.**

| Outil | Caractéristiques |
|-------|------------------|
| **OpenCode** | Open source, agnostique, tool calling transparent |
| **Claude Code** | Vendor lock-in Anthropic mais très performant |
| **Codex CLI** | OpenAI, récent |
| **Gemini CLI** | Google, récent |

**Pourquoi OpenCode ?**
- Tool calling **transparent** : vous voyez chaque action
- **Open source** : vous contrôlez
- **Agnostique** : pas lié à un provider

**Usage :** `opencode "add authentication to the API"` puis laissez mouliner.

---

## Type 2 : Assistants IDE (VSCode et apparentés)

**Autocomplete intelligent + agents légers.**
- **Cursor** 
- **GitHub Copilot**
- **Cline**
- **Roo Code**: Fork de Cline

---

## Type 3 : Bots PR (CI/CD)

**Agents qui travaillent sur vos Pull Requests.**

| Outil | Caractéristiques |
|-------|------------------|
| **Jules** | Google, travaille en background |
| **GitHub Copilot for PR** | Revue automatique |
| **Manuellement** | ex: Avec Github Actions |
| **Autres** | Écosystème en croissance rapide |

**Usage :** Créez une PR, le bot commente et propose des fixes.

---

Choisir son outil

**Question clé : quel est votre workflow ?**

- **Terminal-first, vous aimez contrôler** → OpenCode / Claude Code
- **VSCode habituel, autocomplete suffisant** → Cursor / Copilot
- **Équipe établie, CI/CD mature** → Bots PR

## Openrouter

```bash
# Configuration OpenRouter dans OpenCode
export OPENROUTER_API_KEY="sk-or-v1-..."
# Accès à 200+ modèles
# Gemini Flash (frugal), Claude (qualité), Qwen (multimodal)...
```

---

# Coûts et modèles frugaux

**La facture arrive vite : $500-2000/mois** pour un usage intensif.

| Stratégie | Modèle | Coût/Million tokens |
|-----------|--------|---------------------|
| **Pingre** | Gemini Flash | ~$0.07 |
| **Équilibré** | Claude Haiku | ~$0.25 |
| **Qualité** | Claude Sonnet | ~$3.00 |
| **Multimodal** | Qwen-VL | ~$0.30 |

Commencez frugal, passez premium pour les décisions critiques.
**Pour le cours :** il est intéressant d'opérer avec des modèles suboptimaux pour observer les comportements erratiques principaux des agents, causés par les modèles LLM qui sont derrière.


## Tour rapide des modèles du moment

---

# TP : Configuration initiale

Le TP fil rouge commence : configuration de votre environnement.

**Objectif :**
1. Configurer Codex avec votre clé
