# Makefile - Projet API FastAPI

> **Exemple concret avec cibles pour IA**

---

.PHONY: help install test test-coverage lint format dev build clean validate ai-stats

# Variables
SRC_DIR := src
TEST_DIR := tests
VENV := .venv
PYTHON := $(VENV)/bin/python
PIP := $(VENV)/bin/pip
PYTEST := $(VENV)/bin/pytest
RUFF := $(VENV)/bin/ruff
BLACK := $(VENV)/bin/black

# Couverture minimum
COV_THRESHOLD := 70

# ============================================
# DÉVELOPPEMENT
# ============================================

help:  ## Afficher cette aide
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

install: $(VENV)/bin/activate  ## Installer les dépendances
	$(PIP) install -r requirements.txt
	$(PIP) install -r requirements-dev.txt

$(VENV)/bin/activate:
	python -m venv $(VENV)

dev:  ## Lancer le serveur de développement
	$(PYTHON) -m uvicorn $(SRC_DIR).main:app --reload --port 8000

# ============================================
# TESTS
# ============================================

test:  ## Lancer les tests
	$(PYTEST) $(TEST_DIR)/ -v

test-coverage:  ## Lancer les tests avec coverage
	$(PYTEST) $(TEST_DIR)/ -v --cov=$(SRC_DIR) --cov-report=term-missing --cov-fail-under=$(COV_THRESHOLD)

test-watch:  ## Lancer les tests en mode watch
	$(PYTEST) $(TEST_DIR)/ -v --watch

# ============================================
# QUALITÉ
# ============================================

lint:  ## Linter le code
	$(RUFF) check $(SRC_DIR)/ $(TEST_DIR)/
	$(BLACK) --check $(SRC_DIR)/ $(TEST_DIR)/

format:  ## Formater le code
	$(BLACK) $(SRC_DIR)/ $(TEST_DIR)/
	$(RUFF) check $(SRC_DIR)/ $(TEST_DIR)/ --fix

validate: lint test-coverage  ## Validation complète (lint + coverage)

# ============================================
# BUILD
# ============================================

build:  ## Construire l'image Docker
	docker build -t api-tasks:latest .

clean:  ## Nettoyer les artefacts
	rm -rf .pytest_cache htmlcov .coverage $(VENV)
	find . -type d -name "__pycache__" -exec rm -rf {} +

# ============================================
# IA
# ============================================

ai-stats:  ## Afficher les stats d'usage IA
	@echo "=========================================="
	@echo "IA Usage Statistics"
	@echo "=========================================="
	@opencode stats --cache 2>/dev/null || echo "Cache stats: install opencode CLI"
	@opencode stats --costs 2>/dev/null || echo "Cost stats: install opencode CLI"
	@echo ""
	@echo "Target cache hit rate: >85%"
	@echo "Current budget: $$30/month"

ai-budget:  ## Vérifier le budget IA
	@echo "Budget: $$30/month"
	@echo "Check opencode stats for actual usage"

# ============================================
# SETUP NOUVEAU PROJET
# ============================================

setup-ai:  ## Configurer AGENTS.md pour un nouveau projet
	@echo "Creating AGENTS.md from template..."
	@cp Annexes/AGENTS_example_api.md AGENTS.md 2>/dev/null || echo "Template not found, create manually"
	@echo "Creating .opencode/config.json..."
	@mkdir -p .opencode
	@echo '{"model": "anthropic/claude-3-sonnet"}' > .opencode/config.json
	@echo "Setup complete. Edit AGENTS.md with your project details."

# ============================================
# CI
# ============================================

ci: validate  ## Commande CI complète
	$(MAKE) validate

# ============================================
# UTILITAIRES
# ============================================

reset-db:  ## Réinitialiser la base de données (dev seulement)
	@echo "Warning: This will delete all data!"
	@read -p "Are you sure? " -n 1 -r; echo
	@if [[ $$REPLY =~ ^[Yy]$$ ]]; then \
		$(PYTHON) -c "from src.db import reset; reset()"; \
	fi

migrate:  ## Lancer les migrations
	$(PYTHON) -m alembic upgrade head

migrate-create:  ## Créer une migration (USAGE: make migrate-create MSG="description")
	$(PYTHON) -m alembic revision --autogenerate -m "$(MSG)"