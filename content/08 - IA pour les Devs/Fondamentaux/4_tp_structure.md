---
title: "4 - TP Structuration Projet"
weight: 1055
---

## _Préparer le terrain pour l'agent_

---

# Objectif

Transformer un projet existant en projet "AI-ready" avec les 4 piliers.

---

# Étape 1 : Audit du projet

**Lister les éléments manquants :**

```bash
cd mon-app-demo

# Vérifier la présence des éléments
ls -la | grep -E "Makefile|AGENTS.md|README.md|docker"
```

**Checklist:**
- [ ] AGENTS.md à la racine ?
- [ ] Makefile avec commandes standard ?
- [ ] Docker et docker-compose ?
- [ ] README.md dans les dossiers clés ?

---

## Checklist de découverte des conventions

**Utiliser OpenCode pour analyser le projet :**

```
>Analyse ce projet et réponds aux questions suivantes:
>1. Quelle est la stack technique exacte ?
>2. Quelle est la structure des dossiers ?
>3. Quelles sont les conventions de nommage ?
>4. Quelles sont les commandes de build/test/run ?
```

**Remplir la checklist :**

| Élément | Convention identifiée | Source |
|---------|----------------------|--------|
| **Stack** | Ex: FastAPI + PostgreSQL + Redis | requirements.txt, docker-compose.yml |
| **Structure** | Ex: src/api/, src/models/, tests/ | Arborescence |
| **Commandes** | Ex: make test, make dev, make lint | Makefile ou package.json |
| **Commits** | Ex: feat:, fix:, docs: | git log --oneline |
| **Tests** | Ex: pytest, coverage > 80% | tests/, pytest.ini |
| **Imports** | Ex: imports groupés, ordre alphabétique | src/*.py |
| **Nommage** | Ex: snake_case pour fonctions | Analyse de code |
| **Gestion erreurs** | Ex: exceptions custom, HTTPException | patterns trouvés |
| **Base de données** | Ex: SQLAlchemy ORM, migrations Alembic | models/ |
| **API** | Ex: REST, versionné /api/v1/ | routes/ |
| **Config** | Ex: .env, config.py, Pydantic settings | config/ |

**Questions d'analyse approfondie :**

1. **Commits :** Quels préfixes sont utilisés ? Y a-t-il des scopes ?
2. **Style de code :** Y a-t-il un formatter configuré (black, prettier) ?
3. **Tests :** Quelle est la structure des tests ? Couverture actuelle ?
4. **Dépendances :** Comment sont-elles gérées ? Versions fixées ou flexibles ?
5. **Patterns récurrents :** Quels design patterns sont utilisés ?

---

# Étape 2 : Créer le Makefile

**Identifier les commandes du projet :**

```bash
# Comment lance-t-on les tests ?
# Comment lance-t-on le serveur de dev ?
# Comment build-t-on l'image ?
```

**Créer le Makefile :**

```makefile
# Makefile
.PHONY: help install test lint dev build clean

help:
	@echo "Commands:"
	@echo "  make install  - Install dependencies"
	@echo "  make test      - Run tests"
	@echo "  make lint      - Run linters"
	@echo "  make dev       - Start dev server"
	@echo "  make build     - Build Docker image"

install:
	pip install -r requirements.txt

test:
	pytest tests/ -v --cov=src --cov-report=term-missing

lint:
	black src/ tests/
	ruff check src/ tests/ --fix

dev:
	uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

build:
	docker build -t mon-app:latest .

clean:
	find . -type d -name "__pycache__" -exec rm -rf {} +
	rm -rf .pytest_cache htmlcov .coverage
```

**Tester :**
```bash
make help
make test
```

---

# Étape 3 : Docker et docker-compose

**Créer Dockerfile :**

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY src/ ./src/
COPY tests/ ./tests/

EXPOSE 8000

CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Créer docker-compose.yml :**

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=sqlite:///./data.db
      - DEBUG=true
    volumes:
      - ./src:/app/src
  
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: monapp
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

**Mettre à jour le Makefile :**

```makefile
dev-docker:
	docker-compose up -d
	docker-compose logs -f app

test-docker:
	docker-compose exec app pytest tests/
```

---

# Étape 4 : AGENTS.md complet

**Utiliser OpenCode pour générer une base :**

```
>Analyse ce projet et propose un AGENTS.md complet avec:
>- Stack technique
>- Architecture des dossiers
>- Conventions de code
>- Commandes disponibles (via Makefile)
>- Contraintes à respecter
```

**Vérifier et ajuster manuellement.**

**Points clés à inclure :**
- Stack exacte avec versions
- Commandes `make ...` pour chaque action
- Conventions de nommage
- Ce qu'il ne faut JAMAIS faire

---

# Étape 5 : README par dossier

**Pour chaque dossier important :**

```
>Pour chaque dossier de src/, génère un README.md qui explique:
>- Le rôle du dossier
>- Les fichiers qu'il contient
>- Les conventions à respecter
>- Un exemple de code
```

**Valider les READMEs générés.**

---

# Étape 6 : Validation finale

**Lancer un prompt complexe et observer :**

```
>Add a new endpoint to delete a user.
>The endpoint should:
>- Require authentication
>- Soft delete (set deleted_at)
>- Return 204 on success
```

**Vérifier :**
- L'agent a-t-il utilisé `make test` ?
- A-t-il respecté les conventions ?
- A-t-il évité les contraintes listées ?

---

# Étape 7 : Mesure d'efficacité

**Comparer avant/après :**

| Métrique | Avant structuration | Après |
|----------|---------------------|-------|
| Tokens consommés | ? | ? |
| Iterations nécessaires | ? | ? |
| Fichiers modifiés correctement | ? | ? |
| Respect des conventions | ? | ? |

---

# Étape 8 : Feature complète — Plan, Build, PR review

Maintenant que le projet est structuré (Makefile, Docker, AGENTS.md, READMEs), réalisez une feature non triviale sur Comparia en suivant le cycle complet.

## Choisir la feature

Quelque chose qui nécessite plusieurs fichiers et au moins un test. Par exemple :
- Export CSV des comparaisons
- Historique des sessions avec persistance
- Mode "personnage" persistant entre les messages (reprend l'idée funky de TP2 mais côté backend)

## Phase 1 — PLAN (pas de code encore)

```
> Je veux implémenter [feature] dans Comparia.
  Analyse le projet et propose un plan :
  - Quels fichiers créer ou modifier ?
  - Quelle est l'architecture proposée ?
  - Quels sont les risques ?
  Ne commence pas à coder. Attends ma validation.
```

Lisez le plan, questionnez les choix. Validez ou demandez des ajustements.

## Phase 2 — BUILD

```
> Plan validé. Implémente étape par étape.
  Lance make test après chaque étape.
  Commits atomiques.
```

## Phase 3 — Cleanup

```
> /simplify
```

## Phase 4 — PR review

Créez la PR et demandez une review :

```bash
git checkout -b feature/[nom-de-la-feature]
git push -u origin feature/[nom-de-la-feature]
gh pr create --title "[feat] [description]" --body "..."
```

Puis dans Claude Code :

```
> /review
```

Ou demandez à l'agent de jouer le reviewer :

```
> Review cette PR comme un senior dev sceptique.
  Identifie les problèmes avant que ça parte en prod.
```

**Ce qu'on valide ici :** le projet bien structuré + le cycle Plan/Build/Review fonctionne comme un workflow d'équipe réel.

---

# Étape 9 : Hygiène de session

## /simplify avant review

Claude a tendance à over-engineer. Avant de soumettre une PR ou de demander un review :

```
> /simplify
```

L'agent passe en revue le code modifié et supprime les abstractions inutiles, les fonctions intermédiaires superflues, les patterns sur-conçus.

> Règle : run `/simplify` avant tout review. Claude nettoie sa propre surenchère.

## Retro en fin de session → mise à jour AGENTS.md

À la fin de chaque session de travail, prenez 5 minutes :

```
> Résume ce qu'on vient de faire. Qu'est-ce qui t'a manqué comme contexte ?
  Je veux mettre à jour AGENTS.md.
```

L'agent identifie les lacunes de contexte qu'il a rencontrées. Vous les ajoutez à `AGENTS.md`. La prochaine session part d'une base plus solide.

**Ce qu'on ajoute typiquement après une retro :**
- Contraintes oubliées ("NE PAS modifier les migrations")
- Patterns récurrents du projet ("toujours utiliser le service layer")
- Outils disponibles qu'il ne connaissait pas

---

# Livrable

À la fin de ce TP :

- [ ] Makefile avec commandes standard
- [ ] Docker et docker-compose configurés
- [ ] AGENTS.md complet et validé
- [ ] README.md dans chaque dossier important
- [ ] Mesure de l'amélioration
- [ ] Une feature complète avec Plan + PR review

---

# Checkpoint

**Pattern retenu :** Un projet bien structuré = moins de tokens = moins d'erreurs.

**Question clé :** Le Makefile a-t-il réduit les erreurs de commandes ?

---

# Prochain module

Module 5 : Coûts et modèles frugaux - optimiser sa facture.