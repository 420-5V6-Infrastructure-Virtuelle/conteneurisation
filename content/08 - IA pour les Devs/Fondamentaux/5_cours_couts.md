---
title: "5 - Coûts et Modèles Frugaux"
weight: 1060
---

## _Optimiser sa facture IA_

---

# La réalité des coûts

**Ce que disent les vendors :** "À partir de $20/mois"

**La réalité (Hacker News) :**

| Usage | Coût mensuel | Profil |
|-------|-------------|--------|
| Casual | $10-20 | GitHub Copilot, abonnement basique |
| Actif | $40-100 | Cursor + Claude/GPT |
| Power user | $100-700 | Usage API intensif |
| Extreme | $24,000 | Claude Code sans limite (HN user jbentley1) |

**Source :** [HN: How much are you paying for AI coding tools?](https://news.ycombinator.com/item?id=45091878)

---

# Le piège du pricing vendor

**Le problème :**

```
$20/mois pour Claude Pro
+ $20/mois pour Cursor
+ $30/mois pour GitHub Copilot
+ API calls supplémentaires
= $70-100/mois en abonnements
```

**La solution : OpenRouter**

Un seul abonnement, accès à 200+ modèles :

```yaml
# ~/.config/opencode/config.yaml
providers:
  openrouter:
    api_key: ${OPENROUTER_API_KEY}
    
# Un modèle frugal par défaut
default_model: google/gemini-2.0-flash  # ~$0.07/1M tokens
```

---

# Stratégies d'économie

## 50-85% de réduction possible

| Technique | Économie | Description |
|-----------|----------|-------------|
| **Prompt compression** | 30-50% | Réduire les tokens d'entrée |
| **Semantic caching** | 40-60% | Cacher les requêtes similaires |
| **Model routing** | 50-85% | Router vers le modèle optimal |
| **Context management** | 20-40% | Gérer la fenêtre de contexte |

**Source :** [Token optimization saves up to 80%](https://www.obviousworks.ch/en/token-optimization-saves-up-to-80-percent-llm-costs/)

---

# Modèles frugaux vs premium

| Modèle | Input/1M | Output/1M | Usage recommandé |
|--------|----------|-----------|------------------|
| **Gemini Flash** | $0.07 | $0.30 | Exploration, drafts |
| **DeepSeek** | Gratuit | Gratuit | Math, code simple |
| **Claude Haiku** | $0.25 | $1.25 | Tâches courantes |
| **Claude Sonnet** | $3.00 | $15.00 | Décisions critiques |
| **GPT-4o** | $2.50 | $10.00 | Alternatif premium |

**25+ modèles sous $1/M token disponibles.**

---

# Le pattern "frugal first"

**Recommandation :**

```python
# Workflow frugal
1. Exploration → Gemini Flash (quasi-gratuit)
2. Draft initial → Claude Haiku (économique)
3. Itérations → Claude Haiku
4. Décision finale → Claude Sonnet (premium)
5. Review → Gemini Flash
```

**Économie réaliste : 60-80% de réduction**

---

# Token management en pratique

## Context window en 2026

| Modèle | Context max |
|--------|-------------|
| Claude Opus 4.6 | 1M tokens |
| Gemini 3.1 Pro | 1M tokens |
| Llama 4 Scout | 10M tokens |

**Mais attention :** Plus de contexte ≠ gratuit. Chaque token compte.

## Compression tools

| Outil | Réduction | Description |
|-------|-----------|-------------|
| LLMLingua | 50-80% | Prompt compression |
| Hybrid Context Optimizer | 89-99% | MCP server |
| Auto-compaction | Variable | Intégré dans OpenCode |

---

# Cas concret : $650/mois au lieu de $2400

**HN user (ianberdin) :**

> "My AI costs peaked at $2,400/month. After systematic optimization, I'm down to $650 for the same workload."

**Facteurs clés :**
- Prompt engineering (50% d'économie)
- Model routing (30% supplémentaire)
- Semantic caching (20% restant)

**Source :** [HN: AI Tool Briefing](https://news.ycombinator.com/item?id=44782790)

---

# Pièges à éviter

## 1. Hallucinations des modèles cheap

Les modèles frugaux peuvent halluciner plus :

```python
# Gemini Flash peut inventer des APIs
result = api.fakeMethod()  # N'existe pas !

# Solution : Vérifier contre la doc
```

## 2. Perte de contexte

Compression agressive = perte d'information critique.

## 3. Coût caché des itérations

```
Prompt 1: 10k tokens
Prompt 2: 15k tokens (context repris)
Prompt 3: 20k tokens
...
Total: beaucoup plus que prévu
```

---

# TP : Optimiser ses coûts

Voir `5_tp_couts.md` →