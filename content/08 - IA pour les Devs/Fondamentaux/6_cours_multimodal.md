---
title: "6 - Multimodal et Frugal"
weight: 1060
---

## _Images, audio, et IA économique_

---

# IA Multimodal

**Qu'est-ce que la multimodalité ?**

Un modèle multimodal peut traiter :
- **Texte** (le standard)
- **Images** (screenshots, diagrammes, photos)
- **Audio** (transcription, analyse)
- **Vidéo** (en développement)

---

# Use cases pour les devs

## 1. Analyser des screenshots d'erreurs

```
Prompt: "Voici un screenshot de mon erreur. Explique le problème."
[Image attachée]
```

**Gain :** Pas besoin de copier-coller des stack traces de 50 lignes.

## 2. Transformer des mockups en code

```
Prompt: "Voici le design Figma en image. Génère le CSS correspondant."
[Image du design]
```

**Gain :** De l'image au CSS en un prompt.

## 3. Analyser des diagrammes d'architecture

```
Prompt: "Ce diagramme AWS est-il correct ? Que manque-t-il ?"
[Image CloudWatch/Draw.io]
```

**Gain :** Validation d'architecture par l'IA.

## 4. Debug via screenshot

```
Prompt: "La page s'affiche mal, voici le screenshot et le CSS."
[Screenshot + CSS]
```

---

# Limites importantes

| Limite | Description |
|--------|-------------|
| **Coût en tokens** | Les images consomment beaucoup de tokens |
| **Précision** | L'IA peut mal interpréter une image |
| **Hallucination visuelle** | Inventer des détails qui n'existent pas |
| **Résolution** | Images trop petites = détails perdus |

**Conseil :** Utilisez la multimodalité pour des tâches spécifiques, pas par défaut.

---

# Le Playwright Quirk

## Un screenshot n'a PAS besoin d'être une image

**Le pattern clé :** Playwright peut générer des "snapshots textuels" du DOM.

```python
# PAS ça (lourd)
screenshot = page.screenshot()  # 50KB+ en base64

# Mais ça (léger)
snapshot = page.accessibility.snapshot()
# Retourne une représentation textuelle du DOM
```

**Exemple de snapshot textuel :**

```
- Button "Login" [focused]
- TextBox "Email" [empty]
- TextBox "Password" [empty]
- Link "Forgot password?"
- Heading "Welcome"
```

**Avantages :**
- **Tokens économisés :** 100x moins de tokens qu'une image
- **Exploitable directement :** Le LLM comprend le texte
- **Pas besoin de vision :** N'importe quel modèle peut l'utiliser

**Dans OpenCode :**

```
[TOOL] playwright_snapshot()  
# Pas de screenshot PNG !
# Retourne une representation textuelle
```

---

# IA Frugale

## La facture arrive vite

**Scénarios réels (HN 2024-2025) :**

| Profil | Coût mensuel |
|--------|-------------|
| Casuel | $10-20 |
| Développeur actif | $40-100 |
| Power user | $100-700 |
| Extrême | $2,000-24,000 |

**Story réaliste :** Un développeur a eu une facture de $2,400/mois avant d'optimiser.

---

# Stratégies d'économie

## 1. Modèles frugaux par défaut

| Modèle | Coût/1M tokens | Usage |
|--------|---------------|-------|
| **Gemini Flash** | ~$0.07 | Tâches simples, brainstorming |
| **DeepSeek** | ~$0.10-0.50 | Math, coding avancé |
| **GLM-4** | ~$0.15 | Alternative chinoise |
| **Claude Haiku** | ~$0.25 | Tâches rapides |

## 2. Modèles premium pour les décisions critiques

| Modèle | Coût/1M tokens | Usage |
|--------|---------------|-------|
| **Claude Sonnet** | $3 / $15 | Architecturedécisions |
| **GPT-4o** | $2.50 / $10 | Code complexe |
| **Claude Opus** | $15 / $75 | Review final |

## 3. Pattern de routage intelligent

```yaml
# Dans OpenCode, configurer le routage
routing:
  brainstorm:
    model: gemini-flash
    max_tokens: 1000
  
  code_generation:
    model: claude-sonnet
    
  final_review:
    model: claude-opus
```

---

# Token Management

## Stratégies de compression

| Technique | Économie | Description |
|-----------|---------|-------------|
| **AGENTS.md** | 30-50% | Contexte permanent, pas répété |
| **Prompt compression** | 50-80% | LLMLingua supprime le bruit |
| **Semantic caching** | 40-60% | Cache des requêtes similaires |
| **Context rotation** | Variable | Ralph Loop avec rotation |

**LLMLingua et outils similaires :**

```python
from llmlingua import PromptCompressor

compressor = PromptCompressor()
compressed = compressor.compress_prompt(long_prompt)
# Réduction de 50-80%
```

---

# TP : Modèles frugaux

Le TP fil rouge continue : comparer les modèles.

Voir `6_tp_multimodal.md` →