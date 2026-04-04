---
title: "2 - TP AGENTS.md et Prompts"
weight: 1035
---

## _Structurer le contexte pour l'IA_

---

# Objectif

Créer un fichier `AGENTS.md` complet pour votre application démo et tester l'efficacité des prompts structurés.

---

# Étape 1 : Analyser le projet

**AvecOpenCode, analyser la structure :**

```bash
cd mon-app-demo
opencode
```

**Prompt :**
```
>Analyse ce projet et liste:
>1. La stack technique
>2. Les conventions de nommage
>3. L'architecture des dossiers
>4. Les patterns utilisés
```

**Noter les réponses dans un fichier temporaire.**

---

# Étape 2 : Créer AGENTS.md

**Structure minimale :**

```markdown
# AGENTS.md

## Project
[Description en 1-2 phrases]

## Stack
- Langage: ...
- Framework: ...
- DB: ...
- Tests: ...

## Architecture
```
[Diagramme ASCII de l'architecture]
```

## Conventions
- Naming: ...
- Style: ...
- Tests: ...

## À NE PAS FAIRE
- ...
```

---

# Étape 3 : README par dossier

**Pour chaque dossier important, créer un README.md :**

```markdown
# [Dossier Name]

[Description]

## Responsabilités
- ...

## Fichiers
- `fichier1.py` : ...
- `fichier2.py` : ...

## Exemple d'usage
```python
# Example code snippet
```
```

**LancerOpenCode pour générer :**

```
>Pour chaque dossier important de ce projet, génère un README.md qui explique son rôle et ses conventions.
```

---

# Étape 4 : Tester l'impact

**Faire le même prompt avant et après AGENTS.md :**

```markdown
Prompt: "Ajoute un endpoint pour supprimer un utilisateur."
```

**Grille de comparaison contexte riche vs pauvre :**

| Critère | Prompt naïf (sans AGENTS.md) | Prompt structuré (avec AGENTS.md) |
|---------|------------------------------|-----------------------------------|
| **Compréhension du contexte** | | |
| **Identification des impacts** | | |
| **Respect des conventions** | | |
| **Temps de réponse** | | |
| **Tokens consommés** | | |
| **Itérations nécessaires** | | |

**Analyse qualitative à documenter :**

1. **Bugs créés :** L'agent a-t-il introduit des erreurs sans AGENTS.md ?
2. **Complétude :** A-t-il pensé aux cas limites (soft delete, permissions, tests) ?
3. **Contexte manquant :** Quelles informations aurait-il fallu ajouter ?

**Question clé :** Combien de tokens avez-vous économisés avec un bon AGENTS.md ?

---

# Étape 5 : Prompts structurés

**Exercice : refactoriser un module**

**❌Prompt non structuré :**
```
Refactor la gestion des utilisateurs
```

**✅Prompt structuré :**
```markdown
Contexte: API REST FastAPI avec SQLAlchemy.

Objectif: Refactoriser src/services/user_service.py.

Problème actuel:
- Logique DB mélangée avec logique métier
- Pas de gestion d'erreurs
- Tests couvrent 60%

Contraintes:
- Garder la même interface publique
- Ajouter des exceptions custom
- Monter la couverture à 80%+

Format de sortie:
- Liste des fichiers modifiés
- Diff pour chaque fichier
- Nouveaux tests ajoutés
```

**Essayer les deux et comparer.**

---

# Étape 6 : Pattern de validation

**Toujours demander validation avant application :**

```
>Propose 3 façons de refactoriser ce module avec les pros/cons de chaque.
>Attends ma validation avant d'implémenter.
```

**Pourquoi ?**
- Évite les catastrophes
- Permet de choisir parmi les options
- Garde le contrôle du développeur

---

# Livrable

À la fin de ce TP :

- [ ] `AGENTS.md` à la racine du projet
- [ ] `README.md` pour chaque dossier important
- [ ] Mesure de l'économie de tokens
- [ ] Comparaison prompt vague vs structuré

---

# Checkpoint

**Question clé :** Combien de tokens avez-vous économisés avec un bon AGENTS.md ?

**Pattern retenu :** Toujours structurer ses prompts avec contexte, objectif, contraintes, format de sortie.

---

# Prochain module

Module 3 : Tool Calling et MCP - comprendre ce que fait réellement l'agent.