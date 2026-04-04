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

**Critères de qualité :**
- Code compile sans erreurs
- Tests passent
- Conventions respectées
- Pas de TODOs ou placeholders

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

# Étape 5 : Implémenter le model routing

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

# Livrable

À la fin de ce TP :

- [ ] Mesuré sa consommation de tokens
- [ ] Comparé 3 modèles sur la même tâche
- [ ] Calculé le coût mensuel estimé
- [ ] Identifié les tâches "frugales" vs "premium"

---

# Checkpoint

**Question clé :** À quel moment vaut-il la peine de payer premium ?

**Pattern retenu :** Frugal first, premium pour les décisions critiques.

---

# Prochain module

Module 6 : Multimodal - screenshots, images, et au-delà.