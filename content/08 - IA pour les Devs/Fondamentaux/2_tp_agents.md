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

Ne remplissez pas ce fichier à la main — laissez l'agent le générer à partir de l'étape 1, puis corrigez les inexactitudes. Un AGENTS.md écrit par un humain à partir d'un template vide sera moins précis qu'un AGENTS.md généré par un LLM qui a lu le projet.

```
> À partir de ton analyse du projet, génère un fichier AGENTS.md complet.
  Inclus la stack, l'architecture, les conventions, les commandes disponibles,
  et une section "À NE PAS FAIRE" avec les contraintes critiques.
```

**Ce que ça doit ressembler pour Comparia :**

```markdown
# AGENTS.md

## Project
Comparia est une interface de comparaison de modèles LLM développée par beta.gouv.fr.
Elle soumet le même prompt à plusieurs modèles et compare les réponses côte à côte.

## Stack
- Backend: FastAPI (Python 3.11)
- Frontend: Svelte + TypeScript
- Infra: Docker Compose
- Tests: pytest (backend), vitest (frontend)

## Architecture
backend/
├── app/
│   ├── routers/     # Endpoints FastAPI
│   ├── services/    # Appels LLM
│   └── models/      # Pydantic schemas
frontend/
└── src/
    ├── lib/         # Composants Svelte réutilisables
    └── routes/      # Pages SvelteKit

## Conventions
- Python: snake_case, black formatter, docstrings Google style
- TypeScript: camelCase, eslint
- Commits: feat:, fix:, docs:, refac:

## Commandes
- make dev    # Backend + frontend
- make test   # pytest + vitest
- make lint   # black + ruff + eslint

## À NE PAS FAIRE
- Ne pas modifier .env directement
- Ne pas ajouter de dépendances sans mettre à jour requirements.txt ET pyproject.toml
- Ne pas commit sans passer make test
```

Corrigez ensuite ce que l'agent a mal compris ou oublié.

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

# Étape 7 : Feature funky sur Comparia

Maintenant que votre AGENTS.md existe, testez-le en conditions réelles : ajoutez une petite feature originale à Comparia.

**L'idée :** modifier le prompt système des modèles comparés pour leur donner un rôle absurde.

Quelques pistes :
- Un agent spécialisé dans les **conseils médicaux catastrophiques**
- Un **coach sportif bidon** 

**Démarche :**

```
> Je veux adapter Comparia pour [votre idée].
  Quels fichiers modifier pour changer le prompt système des modèles ?
  Respecte les conventions définies dans AGENTS.md.
```

**Ce qu'on observe ici :**
- L'agent lit-il AGENTS.md avant de proposer ?
- Respecte-t-il les conventions de nommage ?
- Modifie-t-il uniquement les fichiers pertinents ?

Comparez avec ce que vous auriez obtenu sans AGENTS.md (étape 4).

---

# Étape 8 : Patterns de workflow

## Todo list pour les tâches complexes

Pour toute tâche comportant plusieurs étapes, demandez explicitement une todo list :

```
> Avant de commencer, crée une todo list des étapes pour implémenter
  cette feature dans Comparia. On validera chaque étape ensemble.
```

L'agent coche les étapes au fur et à mesure — vous gardez une vue d'ensemble et pouvez réorienter à tout moment.

## Plan → Build → Test → Plan (modestly)

Le cycle recommandé pour toute feature non triviale :

```
1. PLAN  — "Propose une architecture pour [feature]. Pas de code encore."
2. BUILD — "Implémente l'étape 1 seulement."
3. TEST  — "Lance les tests. Qu'est-ce qui casse ?"
4. PLAN  — "On révise le plan avec ce qu'on a appris."
```

> Ne demandez pas à l'agent de tout faire d'un coup. Le cycle court force la vérification à chaque étape.

## Laissez l'agent corriger ses propres erreurs

Quand l'agent génère une erreur, résistez à l'envie de corriger vous-même dans le code :

```
# ❌ Vous corrigez silencieusement dans le code
# → L'agent ne comprend pas pourquoi ça marche

# ✅ Vous montrez l'erreur à l'agent
> "make test" échoue avec ce message : [copier l'erreur]
  Analyse et corrige.
# → L'agent construit une représentation mentale du projet
```

Si vous corrigez vous-même, soit vous dites à l'agent ce que vous avez fait, soit vous le laissez corriger — dans les deux cas, il doit comprendre pourquoi.

---

# Livrable

À la fin de ce TP :

- [ ] `AGENTS.md` à la racine du projet
- [ ] `README.md` pour chaque dossier important
- [ ] Mesure de l'économie de tokens
- [ ] Comparaison prompt vague vs structuré
- [ ] Une feature funky ajoutée à Comparia via AGENTS.md

---

# Checkpoint

**Question clé :** Combien de tokens avez-vous économisés avec un bon AGENTS.md ?

**Pattern retenu :** Toujours structurer ses prompts avec contexte, objectif, contraintes, format de sortie.

---

# Prochain module

Module 3 : Tool Calling et MCP - comprendre ce que fait réellement l'agent.