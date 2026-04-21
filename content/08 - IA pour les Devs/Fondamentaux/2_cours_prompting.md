---
title: "2 - Prompt Engineering pour Devs"
weight: 1030
---

## _Parler efficacement à l'IA_

---

# Ce que les gens croient

> "Il suffit d'être clair et l'IA comprendra."

**Réalité :** La qualité de la réponse dépend de la structure de votre demande.

---

# Les 4 principes

## 1. Contexte explicite

```markdown
#❌Prompt vague
"Fix the bug in my code"

#✅Prompt structuré
"Contexte: API FastAPI avec endpoints REST.
Problème: Le endpoint POST /users retourne 500.
Fichier: src/api/routes/users.py, ligne 42.
Erreur: IntegrityError sur email duplicata.
Objectif: Ajouter une validation avant l'insertion."
```

---

## 2. Découper les tâches

```markdown
#❌Tout en un
"Refactor toute l'application pour ajouter l'authentification"

#✅Étapes distinctes
"Étape 1: Analyser le système d'auth actuel
Étape 2: Proposer une architecture JWT
Étape 3: Implémenter le middleware d'auth
Étape 4: Ajouter les tests"
```

---

## 3. Spécifier le format de sortie

```markdown
#✅Format demandé
"Réponds en format tableau:

| Fichier | Changement | Complexité |
|---------|------------|------------|"
```

---

## 4. Contraintes explicites

```markdown
#✅Contraintes claires
"- Ne pas utiliser de librairies externes
- Garder la compatibilité Python 3.11
- Préserver les tests existants
- Maximum 50 lignes ajoutées"
```

---

# Pattern : AGENTS.md

**Le fichier qui change tout.**

À la racine de votre projet, un fichier `AGENTS.md` donne le contexte à l'IA :

```markdown
# AGENTS.md - Contexte pour les agents IA

## Project
API REST pour gestion d'utilisateurs.

## Stack
- FastAPI 0.104+
- SQLAlchemy ORM
- PostgreSQL 15
- Pytest pour les tests

## Conventions
- Snake_case pour les variables
- docstrings Google style
- Tests dans tests/ avec pytest

## Architecture
- src/api/routes/ : endpoints REST
- src/models/ : modèles SQLAlchemy
- src/services/ : logique métier

## À NE PAS FAIRE
- Ne pas créer de nouvelles migrations DB
- Ne pas modifier .env
- Ne pas ajouter de dépendances sans validation
```

**Pourquoi ça marche ?**
- L'IA a le contexte permanent
- Pas besoin de répéter à chaque prompt
- Réduit les tokens consommés
- **Plus de cohérence** dans les réponses

---

# Pattern : README.md par dossier

Chaque dossier important devrait avoir un `README.md` :

```
src/
├── api/
│   ├── README.md      # "Ce dossier contient les endpoints..."
│   └── routes/
├── models/
│   └── README.md      # "Modèles SQLAlchemy..."
└── services/
    └── README.md      # "Logique métier isolée..."
```

**Contenu type :**
```markdown
# Services Layer

Contient la logique métier isolée des controllers.

## Conventions
- Un service par domaine métier
- Injection de dépendances via __init__
- Pas d'imports circulaires

## Exemple
```python
# services/user_service.py
class UserService:
    def __init__(self, db: Session):
        self.db = db
```
```

---

# Gestion du contexte de session

L'IA ne "se souvient" de rien entre les sessions — et dans une même session, **trop de contexte dégrade la qualité des réponses**. Ce n'est pas seulement une question de coût : un contexte pollué (nombreux fichiers lus, chemins abandonnés, erreurs accumulées) crée du bruit qui fait dériver l'agent.

Signal d'alarme : si l'agent propose des solutions déjà essayées, oublie des contraintes données en début de session, ou semble "confus" — c'est le signe que le contexte est saturé.

**Règles pratiques :**

- **Une session = un problème.** Ne mélangez pas deux bugs ou deux features dans la même session.
- **Plusieurs sessions courtes > une longue session.** Des sessions ciblées donnent de meilleurs résultats qu'une session marathon.
- **Commencez propre régulièrement.** Plutôt que continuer une session qui accumule trop, ouvrez-en une nouvelle — ou utilisez `/compact` (résume sans perdre le fil) quand le contexte dépasse 70%, `/clear` quand il dépasse 85%.

## Le cas particulier du "reasoning"

Certains modèles (o3, Claude avec extended thinking…) génèrent une chaîne de réflexion interne avant de répondre. Ces tokens de raisonnement sont invisibles dans la réponse mais comptent dans la facture — et peuvent multiplier le coût par 5 sur une tâche complexe.

C'est une **feature propriétaire et opaque** : chaque provider l'implémente différemment (`thinking_budget`, `reasoning_effort`, mode automatique…), vous ne voyez pas ce qui a été raisonné, et certains outils l'activent silencieusement. À utiliser intentionnellement pour des problèmes qui le méritent, pas comme réglage par défaut.

---

# Anti-patterns courants

| Anti-pattern | Conséquence | Solution |
|--------------|-------------|----------|
| Prompt trop long | Confusion, tokens gâchés | Découper en étapes |
| Pas de contraintes | Code non idiomatique | Spécifier les conventions |
| Oublier les tests | Code non testé | Demander tests explicites |
| Ignorer l'existant | Duplication | Référencer les fichiers existants |

---

# TP : Créer votre AGENTS.md

Le TP fil rouge continue : créer un `AGENTS.md` pour votre projet démo.

Voir `2_tp_agents.md` →