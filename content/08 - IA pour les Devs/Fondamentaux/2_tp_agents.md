---
title: "2 - TP AGENTS.md, Prompts et Script de Commandes"
weight: 1035
---

## _Structurer le contexte pour l'IA_

> ⏱ **1h15**

> **Outil principal :** Codex CLI (`codex`). Remplacer par `opencode` ou `claude` selon votre outil.

---

# Objectif

Rendre Comparia "AI-ready" : donner à l'agent le contexte projet dont il a besoin pour travailler sans approximations.

---

# Étape 1 : Analyser le projet

```bash
cd comparia
codex   # OpenCode : opencode | Claude Code : claude
```

```
> Analyse ce projet et liste :
> 1. La stack technique
> 2. Les conventions de nommage
> 3. L'architecture des dossiers
> 4. Les patterns utilisés
```

Observez ce que l'agent a compris — et ce qu'il a raté. C'est la matière première de l'AGENTS.md.

---

# Étape 2 : Générer AGENTS.md

Ne remplissez pas ce fichier à la main. Demandez à l'agent de le générer à partir de son analyse, puis corrigez les inexactitudes.

```
> À partir de ton analyse, génère un fichier AGENTS.md complet.
  Inclus : stack, architecture, conventions, commandes disponibles,
  et une section "À NE PAS FAIRE" avec les contraintes critiques.
```

Relisez le résultat et corrigez ce qui est faux ou manquant. Un AGENTS.md incorrect est pire qu'aucun AGENTS.md — il mène l'agent dans une mauvaise direction.

**Vérifier dans Git ce que l'agent a écrit :**

```bash
git diff          # Voir le contenu exact
git add AGENTS.md && git commit -m "feat: add AGENTS.md"
```

---

# Étape 3 : Docker et le script de commandes

## Comprendre Docker

Comparia tourne dans Docker. Avant de demander à l'agent de lancer les tests, il faut comprendre ce que ça implique.

**L'image** est une recette figée : système d'exploitation, dépendances, code. Elle se construit une fois avec `docker build`.

**Le container** est l'exécution de cette recette : un processus isolé du reste de votre machine. L'image ne change pas ; vous pouvez lancer dix containers depuis la même image.

```
docker build → Image
                 │
         docker run → Container (processus isolé)
```

<!-- **Les volumes** lient un dossier de votre machine à un dossier du container. Sans ça, vos modifications de code restent sur votre machine et n'existent pas dans le container qui tourne.

```yaml
# docker-compose.yml
volumes:
  - ./backend:/app   # votre code local → visible dans le container
``` -->

C'est pour ça que les agents se marient bien avec Docker : un container est un environnement reproductible où l'agent peut lancer des commandes, casser des choses, et recommencer sans polluer votre machine.

## Ce que le script doit exposer

Pour qu'un agent puisse manier l'application de façon fiable, il lui faut quatre commandes prévisibles :

| Commande | Ce qu'elle fait |
|----------|----------------|
| `make install` | Installe les dépendances |
| `make dev` | Lance l'app en développement |
| `make test` | Lance les tests (pytest, vitest…) |
| `make lint` | Vérifie le style (black, eslint…) |

Ces noms sont une convention — l'important est qu'ils soient stables et documentés dans AGENTS.md. L'agent n'a pas à deviner comment lancer les tests.

Un script bash (`run.sh`) fonctionne aussi bien qu'un Makefile. L'essentiel est la stabilité des noms.

## Demander à l'agent de générer le script

Vous n'avez pas à écrire la syntaxe Makefile vous-même :

```
> Vérifie si ce projet a un Makefile avec les cibles install, test, lint, dev.
  Si non, génère-en un adapté aux outils détectés (Docker Compose, pytest, vitest).
  Ajoute ces commandes dans AGENTS.md sous une section "Commandes".
```

**Vérifier et tester :**

```bash
git diff          # Quels fichiers l'agent a-t-il touchés ?
make test         # Est-ce que ça marche ?
```

---

# Étape 4 : Tester l'impact du contexte

**Même prompt, avec et sans AGENTS.md :**

```
Prompt: "Ajoute un endpoint pour supprimer un utilisateur."
```

**Test 1 — sans AGENTS.md :**

```bash
git stash         # Cacher temporairement l'AGENTS.md
codex             # Lancer l'agent
```

```
> Ajoute un endpoint pour supprimer un utilisateur.
```

**Test 2 — avec AGENTS.md :**

```bash
git stash pop     # Restaurer l'AGENTS.md
codex
```

```
> Ajoute un endpoint pour supprimer un utilisateur.
```

**Après chaque test :**

```bash
git diff --stat   # Combien de fichiers modifiés ?
git diff          # Ce qui a changé ligne par ligne
git stash         # Remettre à zéro pour le prochain test
```

**Ce qu'on observe :**
- L'agent respecte-t-il les conventions de nommage ?
- A-t-il pensé aux cas limites (soft delete, permissions, tests) ?
- Combien d'itérations ont été nécessaires ?

---

# Étape 5 : Prompts structurés

**❌ Prompt non structuré :**
```
Refactor la gestion des utilisateurs
```

**✅ Prompt structuré :**
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
- Nouveaux tests ajoutés
```

Essayez les deux. Comparez avec `git diff` ce qui a réellement changé.

---

# Étape 6 : Pattern de validation

**Toujours demander validation avant application :**

```
> Propose 3 façons de refactoriser ce module avec les pros/cons de chaque.
  Attends ma validation avant d'implémenter.
```

---

# Étape 7 : Feature nouvelle sur Comparia

Testez votre AGENTS.md en conditions réelles : ajoutez une petite feature originale.

**Exemple :** modifier le prompt système des modèles comparés pour leur donner un rôle absurde (coach sportif bidon, conseiller financier catastrophique…)

```
> Je veux adapter Comparia pour [votre idée].
  Quels fichiers modifier pour changer le prompt système des modèles ?
  Respecte les conventions définies dans AGENTS.md.
```

**Ce qu'on observe :**
- L'agent lit-il AGENTS.md avant de proposer ?
- Respecte-t-il les conventions de nommage ?
- Modifie-t-il uniquement les fichiers pertinents ?

```bash
git diff --stat   # Combien de fichiers touchés ?
git diff          # Ce qui a changé ligne par ligne
```

---

# Étape 8 : Patterns de workflow

## Todo list pour les tâches complexes

```
> Avant de commencer, crée une todo list des étapes pour implémenter
  cette feature. On validera chaque étape ensemble.
```

L'agent coche les étapes au fur et à mesure — vous gardez une vue d'ensemble et pouvez réorienter.

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

Si vous corrigez vous-même, dites-le à l'agent — il doit comprendre pourquoi.

---

# Livrable

- [ ] `AGENTS.md` à la racine du projet
- [ ] Script de commandes fonctionnel (`make test` passe)
- [ ] Comparaison prompt vague vs structuré observée dans `git diff`
- [ ] Une feature ajoutée

---

# Checkpoint

**Question clé :** Combien d'itérations de moins avec un bon AGENTS.md ?

**Pattern retenu :** Structurer ses prompts avec contexte, objectif, contraintes, format de sortie. Vérifier systématiquement dans Git.

---

# Prochain module

Module 3 : Tool Calling et MCP — comprendre ce que fait réellement l'agent.
