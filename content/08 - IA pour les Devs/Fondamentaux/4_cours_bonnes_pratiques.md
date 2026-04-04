---
title: "4 - Bonnes Pratiques et Conventions"
weight: 1050
---

## _Structurer un projet pour l'IA_

---

# Les 4 piliers

Un projet bien structuré pour l'IA a besoin de :

1. **AGENTS.md** - Contexte permanent pour l'agent
2. **Makefile / run.sh** - Commandes reproductibles
3. **Docker** - Environnement isolé et défini
4. **README.md par dossier** - Documentation locale

---

# AGENTS.md approfondi

## Structure recommandée

```markdown
# AGENTS.md

## Project Overview
[1-2 phrases décrivant le projet]

## Tech Stack
- Language: Python 3.11
- Framework: FastAPI
- Database: PostgreSQL 15
- Testing: pytest
- CI: GitHub Actions

## Architecture
```
src/
├── api/          # REST endpoints
├── models/       # SQLAlchemy models
├── services/     # Business logic
└── tests/        # Test suite
```

## Conventions
### Naming
- snake_case for variables and functions
- PascalCase for classes
- UPPER_CASE for constants

### Code Style
- Docstrings: Google style
- Max line length: 88 characters (black)
- Type hints required

### Testing
- Tests in tests/
- Fixtures in tests/conftest.py
- Coverage minimum: 80%

## Commands
- `make test` - Run tests
- `make lint` - Run linters
- `make dev` - Start dev server
- `make build` - Build Docker image

## Constraints (NEVER DO)
- Do NOT modify .env files
- Do NOT create new DB migrations
- Do NOT add dependencies without review
- Do NOT commit secrets

## Current Focus
[Zone ciblée pour le travail en cours]
```

---

## Pattern minimal vs comprehensif

**Projet simple :** AGENTS.md de 20 lignes suffit.

**Projet complexe :** AGENTS.md de 100+ lignes avec détails.

**Règle :** Plus l'agent a de contexte, moins il fait d'erreurs.

---

# Makefile / run.sh

## Pourquoi ?

L'agent doit pouvoir exécuter des commandes sans deviner.

```makefile
# Makefile
.PHONY: test lint dev build clean

test:
	pytest tests/ -v --cov=src

lint:
	black src/ tests/
	ruff check src/ tests/

dev:
	uvicorn src.main:app --reload

build:
	docker build -t myapp:latest .

clean:
	rm -rf .pytest_cache htmlcov
```

**Usage par l'agent :**
```
>Exécute les tests
[TOOL] run_command("make test")
```

**Sans Makefile :**
```
>Exécute les tests
[TOOL] run_command("pytest tests/ -v --cov=src --cov-report=html")
# Erreur si pytest pas installé globalement
# Erreur si paramètres incorrects
```

---

# Docker et docker-compose

## Pourquoi Docker ?

**L'agent doit travailler dans un environnement reproductible.**

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
    depends_on:
      - db
  
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

**Commandes dans le Makefile :**
```makefile
dev-docker:
	docker-compose up -d
	docker-compose exec app uvicorn src.main:app --reload

test-docker:
	docker-compose exec app pytest tests/
```

---

# README.md par dossier

Chaque dossier important a son propre README :

```
src/
├── api/
│   ├── README.md          # "Endpoints REST..."
│   └── routes/
│       ├── README.md      # "Routes organisées par domaine..."
│       └── users.py
├── models/
│   └── README.md          # "Modèles SQLAlchemy..."
└── services/
    └── README.md          # "Logique métier..."
```

**Contenu type :**
```markdown
# API Routes

Ce dossier contient les endpoints REST de l'application.

## Structure
- `users.py` - Routes utilisateur (CRUD)
- `auth.py` - Authentification JWT
- `posts.py` - Gestion des posts

## Conventions
- Une route = un fichier
- Préfixer par le domaine : `/api/v1/users`
- Middleware d'auth sur les routes protégées

## Exemple
```python
from fastapi import APIRouter, Depends
from src.services.auth import get_current_user

router = APIRouter(prefix="/users", tags=["users"])

@router.get("/me")
async def get_me(user = Depends(get_current_user)):
    return user
```
```

---

# OpenClaw et alternatives

**Mention rapide :** L'écosystème des outils pour "ouvrir" les modèles locaux.

| Outil | Usage | Attention |
|-------|-------|------------|
| **OpenClaw** | Utiliser des modèles locaux | Configuration complexe |
| **Ollama** | Run local models | Simplicité vs performance |
| **LocalAI** | API locale type OpenAI | Compatibilité variable |

**Pour cette formation :** On reste sur OpenRouter pour la simplicité.L'idée est de mentionner l'existence sans s'y attarder.

---

# TP : Structurer son projet

Le TP fil rouge continue : refactoriser le projet pour l'IA.

Voir `4_tp_structure.md` →