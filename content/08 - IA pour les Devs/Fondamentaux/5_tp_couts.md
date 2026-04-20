---
title: "5 - TP Optimisation des Coûts"
weight: 1065
---

## _Maîtriser sa facture IA_

---

# Objectif

Comparer différents modèles et mesurer l'impact sur les coûts et la qualité.

---

# Étape 1 : Mesurer sa consommation

**Lancer OpenCode avec verbeux :**

```bash
opencode --verbose 2>&1 | tee session.log
```

**Travailler sur une tâche :**

```
> Add pagination to the GET /users endpoint
```

**Analyser les logs :**

```bash
# Extraire les tokens consommés
grep -i "token" session.log
grep -i "cost" session.log
```

**Noter :**
- Tokens d'entrée (prompt)
- Tokens de sortie (completion)
- Coût estimé

---

# Étape 2 : Comparer les modèles

**Configurer 3 modèles :**

```yaml
# ~/.config/opencode/config.yaml
models:
  frugal:
    model: google/gemini-2.0-flash
  balanced:
    model: anthropic/claude-3.5-haiku
  premium:
    model: anthropic/claude-3.5-sonnet
```

**Même tâche, 3 modèles :**

```
Tâche: "Add input validation to the POST /users endpoint"

Model A (Gemini Flash): ?? tokens, $0.???, qualité ?
Model B (Claude Haiku): ?? tokens, $0.???, qualité ?
Model C (Claude Sonnet): ?? tokens, $0.???, qualité ?
```

**Grille d'évaluation qualité/prix :**

| Critère | Modèle A (Flash) | Modèle B (Haiku) | Modèle C (Sonnet) |
|---------|-----------------|------------------|-------------------|
| **Vitesse** (1-5) | | | |
| **Pertinence** (1-5) | | | |
| **Précision** (1-5) | | | |
| **Suggestions utiles** (1-5) | | | |
| **Tokens consommés** | | | |
| **Coût estimé** | | | |
| **Score qualité** (moyenne) | | | |

**Formula du score qualité :**
```
Score = (Pertinence + Précision + Suggestions) / 3
```

**Critères de qualité détaillés :**
- Code compile sans erreurs
- Tests passent
- Conventions respectées
- Pas de TODOs ou placeholders
- Edge cases couverts

---

# Étape 3 : Prompt compression

**Mesurer le même prompt compressé :**

```markdown
# Prompt long (~500 tokens)
Contexte: Ce projet est une API REST dévelopée avec FastAPI...
Objectif: Ajouter une validation des entrées pour le endpoint...
Contraintes: Respecter les conventions définies dans AGENTS.md...
[etc.]

# Prompt compressé (~100 tokens)
Project: FastAPI REST API
Task: Add input validation to POST /users
Check: AGENTS.md for conventions
Require: email format, password strength, no duplicates
```

**Comparer les tokens et la qualité du résultat.**

---

# Étape 4 : Calculer le ROI

**Scénario A : Tout premium**

```python
Tâches/jour: 20
Tokens/tâche: 5000 input + 2000 output
Coût/tâche: $0.015 (Sonnet)
Coût/jour: $0.30
Coût/mois: $9
```

**Scénario B : Frugal first**

```python
Tâches/jour: 20
Tokens/tâche: 5000 input + 2000 output
Coût/tâche: $0.0005 (Gemini Flash)
Coût/jour: $0.01
Coût/mois: $0.30

Plus 5 tâches critiques en Sonnet: $0.75/jour
Total/mois: $22.50
```

**Économie : 75%**

---

# Étape 5 : Le pattern pingre — réflexion gratuite, implémentation frugale

L'idée : utiliser un modèle **gratuit** pour la phase de réflexion/planification, puis fournir ce plan à un modèle **ultra-frugal** pour l'implémentation mécanique.

## En pratique

**Étape 1 — Plan avec Gemini Pro (gratuit via Google AI Studio)**

[Google AI Studio](https://aistudio.google.com) offre un plan gratuit généreux sur Gemini Pro (rate limits, pas d'usage commercial, mais parfait pour la réflexion) :

```
# Dans Google AI Studio ou via l'API gratuite :
> Analyse ce besoin et propose un plan d'implémentation détaillé
  pour ajouter [feature] à une API FastAPI.
  Liste les fichiers à modifier, les étapes, les risques.
  Ne génère pas de code.
```

**Étape 2 — Implémentation avec un modèle frugal sur OpenRouter**

Copiez le plan dans votre agent OpenCode configuré sur un modèle cheap :

```yaml
# config.yaml
default_model: minimax/minimax-01  # ~$0.10/1M — ou glm-4-9b-chat, nanoflash
```

```
> Voici le plan validé : [coller le plan]
  Implémente étape par étape. Commits atomiques.
```

**Résultat :** la partie coûteuse (raisonnement, architecture) est gratuite ; la partie mécanique (écriture de code répétitive) coûte quasi-rien.

## Curiosité : Nvidia NIM async

Nvidia propose les modèles open source (Llama, Mistral, etc.) **gratuitement** via [build.nvidia.com](https://build.nvidia.com), mais en mode asynchrone — jusqu'à 3h d'attente entre les requêtes en période de charge. Inutilisable en session interactive, mais intéressant pour des tâches batch overnight.

---

# Étape 6 : Implémenter le model routing

**Configuration OpenCode avec routing :**

```yaml
# ~/.config/opencode/config.yaml
default_model: google/gemini-2.0-flash

routing:
  complex_tasks:
    - "refactor"
    - "architecture"
    - "security"
    model: anthropic/claude-3.5-sonnet
  
  simple_tasks:
    - "fix typo"
    - "add comment"
    - "format"
    model: google/gemini-2.0-flash
```

---

# Étape 6 : Caching sémantique

**Identifier les patterns répétitifs :**

```python
# Mêmes prompts, réponses similaires
"Quelle est la structure de ce projet ?"
"Explique ce fichier"
"Ajoute des tests pour cette fonction"
```

**Mettre en cache les réponses :**

```yaml
# OpenCode supporte le caching des conversations
# Les réponses identiques sont servies depuis le cache
```

---

# Étape 7 : Outils d'optimisation des tokens

## rtk — proxy de réduction de tokens

[rtk](https://github.com/rtk-ai/rtk) est un proxy CLI qui réduit la consommation de tokens de 60-90% sur les commandes de dev courantes en compressant le contexte envoyé au modèle.

```bash
# Installation
npm install -g rtk

# Usage : préfixer vos commandes claude
rtk claude "Ajoute un endpoint DELETE /users/:id"
```

Utile pour les tâches répétitives où le contexte projet est volumineux.

## Grille qualité/prix sur Comparia

Appliquez la grille suivante à un usage concret sur Comparia (par exemple : ajouter une feature de filtrage des comparaisons) :

| Critère | Modèle A (Gemini Flash) | Modèle B (Claude Sonnet) |
|---------|------------------------|--------------------------|
| **Vitesse** (1-5) | | |
| **Créativité** (1-5) | | |
| **Cohérence** (1-5) | | |
| **Tokens consommés** | | |
| **Coût estimé** | | |

**Question clé :** Le modèle premium justifie-t-il son prix pour cette tâche ?

## Seuils de contexte à surveiller

Gardez un œil sur `Ctx(u)` dans la statusline (configurée en TP1) :

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

- [ ] Mesuré sa consommation de tokens
- [ ] Comparé 3 modèles sur la même tâche
- [ ] Calculé le coût mensuel estimé
- [ ] Identifié les tâches "frugales" vs "premium"
- [ ] rtk testé sur une commande

---

# Checkpoint

**Question clé :** À quel moment vaut-il la peine de payer premium ?

**Pattern retenu :** Frugal first, premium pour les décisions critiques.

---

# Prochain module

Module 6 : Multimodal - screenshots, images, et au-delà.