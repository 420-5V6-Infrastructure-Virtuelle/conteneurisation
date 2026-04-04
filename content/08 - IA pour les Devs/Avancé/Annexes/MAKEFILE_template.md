# Makefile Template - Projet IA

> Template de Makefile avec cibles pour projets utilisant l'IA

---

## Structure de base

```makefile
# Makefile pour projet IA
.PHONY: help install test lint run clean ai-stats

# Variables
PYTHON := python3
PIP := pip
PYTEST := pytest
BLACK := black
RUFF := ruff

# Dossiers
SRC_DIR := src
TEST_DIR := tests
DOCS_DIR := docs

# Fichiers
AGENTS_FILE := AGENTS.md
COVERAGE_FILE := .coverage

# ===========================================

help: ## Afficher cette aide
	@echo "Makefile pour projet avec IA"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

# ===========================================
# DÉVELOPPEMENT
# ===========================================

install: ## Installer les dépendances
	$(PIP) install -r requirements.txt
	$(PIP) install -r requirements-dev.txt

install-ai: ## Installer les dépendances pour IA
	$(PIP) install -r requirements.txt
	$(PIP) install -r requirements-ai.txt

dev: ## Mode développement
	@echo "🔍 Lancement en mode développement..."
	$(PYTHON) -m $(SRC_DIR).main

run: ## Lancer l'application
	$(PYTHON) -m $(SRC_DIR).main

# ===========================================
# TESTS
# ===========================================

test: ## Lancer les tests
	$(PYTEST) $(TEST_DIR) -v

test-coverage: ## Lancer les tests avec coverage
	$(PYTEST) $(TEST_DIR) --cov=$(SRC_DIR) --cov-report=term-missing --cov-report=html

test-coverage-min: ## Tests avec coverage minimum (70%)
	$(PYTEST) $(TEST_DIR) --cov=$(SRC_DIR) --cov-fail-under=70

test-fast: ## Tests rapides (skip slow)
	$(PYTEST) $(TEST_DIR) -v -m "not slow"

test-all: ## Tous les tests (dont slow)
	$(PYTEST) $(TEST_DIR) -v --runslow

# ===========================================
# QUALITÉ
# ===========================================

lint: ## Lancer le linter
	$(RUFF) check $(SRC_DIR) $(TEST_DIR)

lint-fix: ## Corriger automatiquement
	$(RUFF) check --fix $(SRC_DIR) $(TEST_DIR)

format: ## Formater le code
	$(BLACK) $(SRC_DIR) $(TEST_DIR)

format-check: ## Vérifier le formatage
	$(BLACK) --check $(SRC_DIR) $(TEST_DIR)

check: lint format-check ## Vérifier lint + format
	@echo "✅ Code prêt pour commit"

# ===========================================
# IA
# ===========================================

ai-stats: ## Afficher les stats d'usage IA
	@echo "📊 Cache hit rate:"
	@opencode stats --cache
	@echo ""
	@echo "💰 Coûts du mois:"
	@opencode stats --costs

ai-cache-clear: ## Vider le cache IA
	@echo "🗑️  Clearing prompt cache..."
	@opencode cache clear

ai-version: ## Vérifier la version OpenCode
	@opencode --version

ai-config: ## Afficher la config IA
	@echo "📁 Config location:"
	@echo "  - Global: ~/.opencode/config.json"
	@echo "  - Local: .opencode/config.json"
	@echo ""
	@echo "📄 AGENTS.md location:"
	@if [ -f $(AGENTS_FILE) ]; then \
		echo "  ✅ $(AGENTS_FILE) found"; \
		cat $(AGENTS_FILE); \
	else \
		echo "  ❌ No AGENTS.md found"; \
	fi

# ===========================================
# VALIDATION FINALE
# ===========================================

validate: test-coverage-min lint format-check ## Validation complète (tests, lint, format)
	@echo "✅ Validation terminée avec succès"

pre-commit: check validate ## Hook pre-commit
	@echo "✅ Pre-commit checks passés"

# ===========================================
# DOCUMENTATION
# ===========================================

docs: ## Générer la documentation
	$(PYTHON) -m pdoc $(SRC_DIR) -o $(DOCS_DIR)

docs-serve: ## Servir la documentation
	$(PYTHON) -m pdoc $(SRC_DIR) --http :8080

# ===========================================
# NETTOYAGE
# ===========================================

clean: ## Nettoyer les fichiers générés
	rm -rf __pycache__
	rm -rf .pytest_cache
	rm -rf .ruff_cache
	rm -rf htmlcov
	rm -f $(COVERAGE_FILE)
	find . -type d -name "__pycache__" -exec rm -rf {} +
	find . -type f -name "*.pyc" -delete

clean-all: clean ## Nettoyer tout (dont env)
	rm -rf .venv
	rm -rf venv

# ===========================================
# SETUP
# ===========================================

setup: ## Setup initial
	python3 -m venv .venv
	@echo "Activate with: source .venv/bin/activate"
	$(PIP) install --upgrade pip
	$(PIP) install -r requirements.txt
	$(PIP) install -r requirements-dev.txt

setup-ai: setup ## Setup avec support IA
	$(PIP) install -r requirements-ai.txt
	@echo ""
	@echo "📋 Prochaines étapes:"
	@echo "  1. Copier AGENTS_template.md vers AGENTS.md"
	@echo "  2. Personnaliser AGENTS.md pour votre projet"
	@echo "  3. Lancer 'opencode'"

# ===========================================
# CI/CD
# ===========================================

ci: validate ## Commande CI complète
	@echo "✅ CI checks passés"

ci-test: test-coverage-min ## Tests pour CI
	@echo "✅ Tests OK (coverage > 70%)"

ci-lint: lint format-check ## Lint pour CI
	@echo "✅ Lint OK"

# ===========================================
# UTILITAIRES
# ===========================================

tree: ## Afficher la structure
	@tree -I '__pycache__|*.pyc|.venv|venv|.git|.pytest_cache|.ruff_cache|htmlcov'

info: ## Informations sur le projet
	@echo "📦 Projet: $(shell basename $(CURDIR))"
	@echo "📁 Structure:"
	@make tree --no-print-directory | head -20
	@echo ""
	@echo "📄 Fichiers importants:"
	@if [ -f $(AGENTS_FILE) ]; then echo "  ✅ $(AGENTS_FILE)"; else echo "  ❌ Pas de $(AGENTS_FILE)"; fi
	@if [ -f README.md ]; then echo "  ✅ README.md"; else echo "  ❌ Pas de README.md"; fi
	@if [ -f Makefile ]; then echo "  ✅ Makefile"; else echo "  ❌ Pas de Makefile"; fi

todo: ## Afficher les TODOs
	@grep -r "TODO\|FIXME\|XXX" $(SRC_DIR) || echo "Aucun TODO trouvé"

```

---

## Variante JavaScript/Node.js

```makefile
# Makefile pour projet Node.js avec IA
.PHONY: help install test lint run clean ai-stats

# Variables
NPM := npm
NODE := node

# ===========================================

help: ## Afficher cette aide
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

# ===========================================
# DÉVELOPPEMENT
# ===========================================

install: ## Installer les dépendances
	$(NPM) install

install-dev: install ## Installer dépendances dev
	$(NPM) install --include=dev

dev: ## Mode développement
	$(NPM) run dev

run: ## Lancer l'application
	$(NPM) run start

build: ## Build production
	$(NPM) run build

# ===========================================
# TESTS
# ===========================================

test: ## Lancer les tests
	$(NPM) test

test-coverage: ## Tests avec coverage
	$(NPM) test -- --coverage --coverageThreshold='{"global":{"lines":70}}'

test-watch: ## Tests en watch mode
	$(NPM) test -- --watch

# ===========================================
# QUALITÉ
# ===========================================

lint: ## Lancer le linter
	$(NPM) run lint

lint-fix: ## Corriger automatiquement
	$(NPM) run lint:fix

format: ## Formater le code
	$(NPM) run format

format-check: ## Vérifier le formatage
	$(NPM) run format:check

check: lint format-check ## Vérifier lint + format

# ===========================================
# IA
# ===========================================

ai-stats: ## Stats d'usage IA
	opencode stats --cache
	opencode stats --costs

ai-config: ## Afficher config IA
	@echo "📁 Config: .opencode/config.json"
	@cat .opencode/config.json 2>/dev/null || echo "Pas de config locale"

# ===========================================
# VALIDATION
# ===========================================

validate: test-coverage lint format-check ## Validation complète
	@echo "✅ Validation OK"

# ===========================================
# CI/CD
# ===========================================

ci: validate ## CI checks
	@echo "✅ CI OK"
```

---

## Fichiers requirements

### requirements.txt

```
# Core
fastapi>=0.100.0
uvicorn>=0.23.0
pydantic>=2.0.0

# Database (si besoin)
# sqlalchemy>=2.0.0
# alembic>=1.12.0

# Utils
python-dotenv>=1.0.0
```

### requirements-dev.txt

```
# Testing
pytest>=7.0.0
pytest-cov>=4.0.0
pytest-asyncio>=0.21.0

# Linting
ruff>=0.0.280
black>=23.0.0
mypy>=1.0.0

# Docs
pdoc>=14.0.0
```

### requirements-ai.txt

```
# OpenCode requirements (optionnel)
# L'IA utilise son propre environnement

# Pour tests avec IA
pytest-timeout>=2.2.0  # Timeout pour éviter boucles infinies
```

---

## Utilisation

```bash
# Setup initial
make setup-ai

# Développement quotidien
make dev

# Avant commit
make validate

# Tests avec coverage
make test-coverage

# Stats IA
make ai-stats

# Aide
make help
```

---

## Intégration pre-commit

### .pre-commit-config.yaml

```yaml
repos:
  - repo: local
    hooks:
      - id: make-validate
        name: Make validate
        entry: make validate
        language: system
        pass_filenames: false

      - id: check-agents-file
        name: Check AGENTS.md exists
        entry: /bin/bash -c 'test -f AGENTS.md'
        language: system
        pass_filenames: false
```

---

## CI/CD minimal

### GitHub Actions

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt -r requirements-dev.txt
      - run: make ci-test
      - run: make ci-lint
```

---

*Makefile template pour projets avec OpenCode/Claude Code*