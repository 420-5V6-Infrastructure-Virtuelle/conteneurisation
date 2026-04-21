---
title: "6 - TP Multimodal et Modèles Frugaux"
weight: 1065
draft: true
---

## _Comparer et optimiser les coûts_

---

# Objectif

Comparer différents modèles, mesurer les coûts, et expérimenter la multimodalité.

## Quand utiliser le multimodal

| Cas d'usage | Description |
|-------------|-------------|
| **Screenshot d'erreur** | Montrer l'erreur en UI plutôt que de la décrire |
| **Mockup → code** | Transformer un design en HTML/CSS |
| **Diagramme d'archi** | Analyser un schéma et suggérer une implémentation |
| **Debug visuel** | "Pourquoi la page s'affiche comme ça ?" |

## Limitations à connaître

| Limitation | Impact |
|------------|--------|
| Coût élevé | Une image = 500–2000 tokens selon la résolution |
| Hallucination visuelle | Le modèle peut "lire" du texte qui n'existe pas |
| Résolution limitée | Les détails fins sont souvent manqués |

**L'astuce Playwright :** au lieu d'envoyer une image (lourd), le MCP Playwright retourne une représentation textuelle du DOM. Aucun token d'image, même précision pour naviguer la structure. Voir TP3 Étape 6.

---

# Partie 1 : Comparaison de modèles

## Étape 1 : Préparer le test

**Prompt de référence :**

```markdown
Analyse le fichier src/api/routes/users.py et propose 3 façons
d'améliorer la gestion des erreurs. Pour chaque proposition,
donne le code modifié et explique les avantages.
```

**Garder ce prompt identique pour tous les modèles.**

---

## Étape 2 : Tester les modèles frugaux

**Configurer Gemini Flash :**

```yaml
#~/.config/opencode/config.yaml
default_model: google/gemini-2.0-flash
```

**Exécuter le prompt et noter :**
- Temps de réponse
- Qualité de la réponse (1-5)
- Tokens consommés (input/output)
- Coût estimé

---

## Étape 3 : Tester DeepSeek

```yaml
default_model: deepseek/deepseek-chat
```

**Même prompt, mêmes mesures.**

---

## Étape 4 : Tester Claude Sonnet

```yaml
default_model: anthropic/claude-3.5-sonnet
```

**Même prompt, mêmes mesures.**

---

## Étape 5 : Comparer

**Tableau à remplir :**

| Modèle | Temps | Qualité | Tokens In | Tokens Out | Coût |
|--------|-------|---------|-----------|------------|------|
| Gemini Flash | ? | ? | ? | ? | ? |
| DeepSeek | ? | ? | ? | ? | ? |
| Claude Haiku | ? | ? | ? | ? | ? |
| Claude Sonnet | ? | ? | ? | ? | ? |
| GPT-4o | ? | ? | ? | ? | ? |

**Question :** Quel est le meilleur rapport qualité/prix ?

---

# Partie 2 : Multimodal

## Étape 1 : Screenshot d'erreur

**Prendre un screenshot d'une erreur dans votre app :**

```bash
# Si l'app tourne localement
# Capturer un screenshot de l'erreur avec un outil de screenshot
```

**Prompt multimodal :**

```
Voici un screenshot d'erreur dans mon application.
Analyse le problème et propose une solution.
[Image attachée]
```

---

## Étape 2 : Playwright Snapshot

**Utiliser le snapshot textuel :**

```bash
# Avec OpenCode configuré avec Playwright MCP
opencode
```

```
>Lance l'app localement avec playwright et prends un snapshot
>de la page d'accueil. Analyse la structure.
```

**Observer :**
- Le snapshot est-il textuel ou image ?
- Combien de tokens consommés ?
- Est-ce que l'agent comprend la structure ?

---

## Étape 3 : Transformer un mockup

**Trouver un mockup simple (ou en créer un) :**

```
Voici le design de la page de profil.
Génère le HTML/CSS correspondant.
[Image du mockup]
```

**Comparer :**
- La fidélité du résultat
- Le temps de génération
- Les tokens consommés

---

# Partie 3 : Économie de tokens

## Étape 1 : Prompt mal structuré

**Exécuter ce prompt :**

```
Je veux améliorer mon code mais je sais pas trop quoi faire
en fait j'ai des bugs et des problèmes de perf peux-tu m'aider
à tout régler s'il te plaît merci.
```

**Noter les tokens consommés.**

---

## Étape 2 : Prompt bien structuré

**Exécuter ce prompt :**

```markdown
Contexte: API FastAPI avec endpoint GET /users
Problème: Temps de réponse > 2s pour 100 utilisateurs
Fichier concerné: src/api/routes/users.py
Objectif: Optimiser pour < 500ms

Contraindre:
- Garder la même signature
- Ne pas ajouter de dépendances
- Documenter les changements
```

**Comparer les tokens consommés.**

---

## Étape 3 : Calculer l'économie

| Prompt | Tokens | Coût |
|--------|--------|------|
| Mal structuré | ? | ? |
| Bien structuré | ? | ? |
| Économie | ? | ? |

---

# Partie 4 : Stratégie de routage

**Proposer un routage intelligent :**

```yaml
# Noter votre stratégie
brainstorming:
  model: ???  # Votre choix
  raison: ???

code_simple:
  model: ???
  raison: ???

architecturedécision:
  model: ???
  raison: ???

review_final:
  model: ???
  raison: ???
```

---

# Livrable

À la fin de ce TP :

- [ ] Tableau comparatif des modèles
- [ ] Test multimodal (screenshot et/ou mockup)
- [ ] Mesure d'économie avec prompts structurés
- [ ] Stratégie de routage personnelle

---

# Checkpoint

**Pattern retenu :** Frugal par défaut, premium pour les décisions critiques.

**Question clé :** Quel modèle utilisez-vous le plus pour le travail quotidien ?

---

# Prochain module

Jour 2 - Module 7 : Ralph Loop et workflows autonomes.