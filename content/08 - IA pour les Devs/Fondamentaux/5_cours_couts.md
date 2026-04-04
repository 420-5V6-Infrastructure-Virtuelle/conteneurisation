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
| **Gemini Pro** | Gratuit* | Gratuit* | Planification, reflection |
| **Claude Sonnet** | $3.00 | $15.00 | Décisions critiques |
| **GPT-4o** | $2.50 | $10.00 | Alternatif premium |

*\*Gemini Pro: plan gratuit généreux via Google AI Studio*

**Modèles ultra-frugaux sur OpenRouter :**

| Modèle | Input/1M | Output/1M | Usage |
|--------|----------|-----------|-------|
| **Minimax 2.5** | $0.10 | $0.10 | Implémentation simple |
| **GLM-4.7** | $0.08 | $0.08 | Code trivial |
| **NanoFlash** | $0.03 | $0.03 | Micro-tâches |

**200+ modèles disponibles sur OpenRouter.**

---

# Le pattern "Réfléchir puis implémenter"

**Le concept : Utiliser les modèles gratuits pour la réflexion, les modèles frugaux pour l'implémentation.**

```python
# Workflow économique
1. Réflexion/Planification → Gemini Pro (GRATUIT via Google AI Studio)
2. Décision/Architecture → Claude Sonnet (premium, mais occasionnel)
3. Implémentation simple → OpenRouter Minimax/GLM (~$0.08/1M)
4. Review final → Gemini Flash (quasi-gratuit)
```

**Pourquoi ça marche :**

| Phase | Temps | Modèle | Coût |
|-------|------|--------|------|
| **Réflexion** | 60% du temps | Gemini Pro free | $0 |
| **Décision critique** | 10% | Claude Sonnet | $0.30 |
| **Implémentation** | 25% | Minimax 2.5 | $0.10 |
| **Review** | 5% | Gemini Flash | $0.01 |

**Résultat :** Même workflow, 90% d'économie.

**Offres gratuites à connaître :**

| Provider | Plan gratuit | Limitations |
|----------|--------------|-------------|
| **Google AI Studio** | Gemini Pro illimité* | Rate limits, pas d'usage commercial |
| **Nvidia NIM** | Modèles open source | 40 req/min, rate limits |
| **OpenRouter** | Crédits initiaux | Variables |

**Pattern alternatif - Nvidia Free Tier :**

Nvidia propose des modèles open source gratuits :
- Llama, Mistral, etc. via Nvidia NIM
- 40 requêtes/minute
- Idéal pour les tâches batch ou exploration
- Gratuit, mais rate limits stricts

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