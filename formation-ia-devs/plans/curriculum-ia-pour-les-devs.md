# Formation "IA pour les Développeurs" - Curriculum Complet

## Objectifs Pédagogiques

À l'issue de cette formation de 2 jours, les participants seront capables de :

1. **Choisir** les outils IA appropriés selon leur contexte (projet, équipe, budget)
2. **Produire, corriger et tester** du code de manière autonome avec l'IA
3. **Structurer** leur workflow avec une intégration cohérente de l'IA

---

## Public Cible

- Développeurs curieux de l'IA mais ne sachant pas par où commencer
- Développeurs peu familiers des bonnes pratiques actuelles
- Développeurs inquiets pour leur avenir professionnel

---

## Format

- **70% pratique** (TP fil rouge)
- **30% théorie** (concepts, patterns, retours d'expérience)
- Modules de **55 minutes** avec pauses de 10-15 min
- **OpenCode** comme outil principal (agnostique, open source, tool calling transparent)

---

# JOUR 1 : Fondamentaux et Productivité

## Module 1 : Introduction et Écosystème (55 min)

### Théorie (20 min)

#### Objectifs de la formation
- Démystifier l'IA pour les devs
- Comprendre ce que l'IA fait bien / mal
- Acquérir des réflexes pratiques

#### Typologie des outils IA

**Agents TUI (Terminal User Interface)**
- *Caractéristiques*: Tool calling transparent, autonomie élevée, cycle complet
- *Outils*:
  - **OpenCode** - Open source, agnostique, MCP natif ← **Notre focus**
  - Claude Code - Anthropic, très performant, coûteux
  - Gemini CLI - Google, gratuit avec limites
  - Codex CLI - OpenAI, orienté GitHub Copilot
- *Usage idéal*: Refactoring complexe, génération de features, debugging profond

**Assistants IDE**
- *Caractéristiques*: Intégration éditeur, suggestions temps réel, moins d'autonomie
- *Outils*:
  - Cursor - Fork VSCode, contexte projet entier
  - GitHub Copilot - Intégré partout, suggestions inline
  - Cline/Roo Code - Extension VSCode, mode agent
- *Usage idéal*: Complétion, quick fixes, navigation code

**Bots CI/CD (PR Reviewers)**
- *Caractéristiques*: Automatisés, review de PR, pas de dev direct
- *Outils*:
  - Google Jules - Crée des PR automatiquement
  - PR Agent - Review automatisée
  - Devin (Cognition) - Agent full-stack (très coûteux)
- *Usage idéal*: Review automatique, détection de bugs, suggestions d'amélioration

**Écosystème des providers LLM**
| Provider | Points forts | Modèles notables |
|----------|--------------|------------------|
| OpenAI | Standard, fiable | GPT-4o, o1 (reasoning) |
| Anthropic | Reasoning, code | Claude 4 Sonnet, Claude 4 Opus |
| Google | Multimodal, gratuit | Gemini 2.0 Flash, Gemini 2.5 Pro |
| DeepSeek | Frugal, performant | DeepSeek V3, DeepSeek R1 |
| OpenRouter | Agrégateur, choix | Accès à tous les modèles |

### TP Fil Rouge - Setup (35 min)

**App démo au choix :**

*Option A - Minimaliste (~500 lignes)*
- `luchog01/minimalistic-fastapi-template` - FastAPI + Docker + structure clean
- Idéal pour focus sur les patterns sans overhead

*Option B - Full Stack (évolutif)*
- `fastapi/full-stack-fastapi-template` - FastAPI + React + PostgreSQL + Docker
- Plus réaliste, permet cas complexes

*Option C - SvelteKit*
- `swyxio/swyxkit` - SvelteKit minimal, ~800 lignes
- Alternative frontend-focused

*Option D - Express TypeScript*
- `edwinhern/express-typescript-2024` - Express + TypeScript + Docker
- Stack JS/TS alternative

**Exercice 1.1 : Installation OpenCode**
```bash
# Via npm
npm install -g opencode

# Configuration initiale (utiliser OpenRouter)
opencode config

# Ajouter clé API OpenRouter
export OPENAI_API_KEY=sk-or-...
```

**Exercice 1.2 : Premier prompt**
- Lancer OpenCode sur l'app démo
- Prompt simple : "Analyse ce projet et résume l'architecture"
- Observer le tool calling transparent (grep, read, glob)

**Exercice 1.3 : Comparaison de modèles**
- Tester avec différent modèles via OpenRouter
- Comparer : Claude 4 Sonnet vs Gemini 2.0 Flash vs DeepSeek V3
- Noter : vitesse, qualité, coût

---

## Module 2 : Prompt Engineering pour Devs (55 min)

### Théorie (15 min)

#### Principes de base
- **Contexte** > Instructions vagues
- **Exemples** > Théorie seule (few-shot prompting)
- **Décomposition** > Une tâche complexe = plusieurs tâches simples
- **Feedback** > Itérer sur les résultats

#### Patterns efficaces

**Pattern 1 : Contexte structuré**
```
[ROLE] Tu es un expert FastAPI
[CONTEXT] Projet de gestion de tâches avec auth JWT
[TASK] Ajouter endpoint POST /tasks/{id}/assign
[CONSTRAINTS] - Valider user_id existe
             - Notifier par email
             - Retourner 404 si task absente
```

**Pattern 2 : Few-shot**
```
Exemple 1:
Input: "fix login bug"
Output: "Investigation: found missing token validation in auth.py:42"

Exemple 2:
Input: "add pagination"
Output: "Investigation: need to modify get_items() and add skip/limit params"

Maintenant:
Input: "add rate limiting"
Output:
```

**Pattern 3 : Chain of Thought**
```
Avant de coder:
1. Analyse les fichiers pertinents
2. Identifie les dépendances
3. Propose un plan détaillé
4. Code étape par étape
```

### TP Fil Rouge (40 min)

**Exercice 2.1 : Refactoring avec contexte riche**
- Objectif : Ajouter un système de tags aux tâches
- Prompt naïf → résultat décevant
- Prompt structuré → résultat optimal
- Comparer les outputs

**Exercice 2.2 : Décomposition de tâche complexe**
```
Tâche: "Implémenter un système de notifications email pour les tâches assignées"
```
- Décomposer en sous-tâches
- Prompts séparés pour chaque sous-tâche
- Orchestration manuelle

**Exercice 2.3 : Debugging avec collaboration**
- Introduire un bug intentionnel
- Demander à l'IA de le trouver
- Observer le processus d'investigation

---

## Module 3 : Tool Calling et MCP (55 min)

### Théorie (20 min)

#### Qu'est-ce que le Tool Calling ?
- Le LLM "appelle" des outils (fonctions) de manière autonome
- Cycle : LLM décide → Appelle outil → Reçoit résultat → Continue
- Transparence: voir ce que l'IA fait en temps réel

#### MCP (Model Context Protocol)

**Concept** : Standard pour connecter des outils aux LLMs

**Serveurs MCP populaires :**
| Serveur | Usage |
|---------|-------|
| filesystem | Lire/écrire fichiers |
| github | Opérations Git/GitHub |
| postgres | Requêtes BDD |
| brave-search | Recherche web |
| puppeteer | Automation navigateur |
| slack/notion | Intégrations team |

**Écosystème MCP :**
- *OpenClaw* - Registry de serveurs MCP
- *Context7* - Documentation à jour des bibliothèques
- *Serveurs custom* - Créer le vôtre

#### Pourquoi c'est important ?
- **Auditabilité** : Voir chaque action
- **Contrôle** : Valider/refuser les opérations
- **Extensibilité** : Ajouter vos propres outils

### TP Fil Rouge (35 min)

**Exercice 3.1 : Observer le tool calling**
```
Prompt: "Trouve tous les endpoints API et leur méthode HTTP"
```
- Observer `lsp_diagnostics`, `grep`, `read`
- Comprendre le cycle de décision

**Exercice 3.2 : MCP filesystem en action**
- Modifier un fichier via l'agent
- Voir la structure de l'appel tool
- Analyser le pattern read → edit → verify

**Exercice 3.3 : Ajouter un MCP custom (optionnel)**
- Installer `mcp-server-postgres`
- Configurer une BDD de test
- Faire des requêtes via l'agent

---

## Module 4 : Bonnes Pratiques de Projet (55 min)

### Théorie (25 min)

#### Fichier AGENTS.md

**Quoi** : Document qui guide l'IA sur votre projet

**Pourquoi** : Contexte persistant, conventions explicites, gain de tokens

**Structure recommandée :**
```markdown
# AGENTS.md - Projet XYZ

## Role
Tu es un développeur senior sur ce projet.

## Stack Technique
- Backend: FastAPI (Python 3.11)
- Frontend: SvelteKit
- DB: PostgreSQL
- Tests: pytest, vitest

## Commands
- Run dev: `make dev`
- Run tests: `make test`
- Lint: `make lint`
- Build: `docker-compose build`

## Conventions
- Commits: conventional commits (feat, fix, docs)
- Branches: feature/, fix/, refactor/
- PR: vers develop

## Always
- Créer des tests pour nouvelles features
- Suivre patterns existants
- Valider avec `make lint` avant commit

## Ask First
- Modifier la structure DB
- Ajouter des dépendances
- Changer les configs Docker

## Never
- Commit des secrets (.env, credentials)
- Supprimer des tests existants
- Utiliser `as any`, `@ts-ignore`
```

#### Makefile / run.sh

**Makefile classique :**
```makefile
.PHONY: dev test lint build clean

dev:
	docker-compose up -d && docker-compose logs -f

test:
	pytest tests/ -v --cov=src

lint:
	ruff check src/
	mypy src/

build:
	docker-compose build

clean:
	docker-compose down -v
```

**run.sh alternatif :**
```bash
#!/bin/bash
set -e

case "$1" in
  dev)
    docker-compose up -d
    ;;
  test)
    pytest tests/ -v
    ;;
  *)
    echo "Usage: $0 {dev|test|lint|build}"
    exit 1
    ;;
esac
```

#### Docker Config

**docker-compose.yml standard :**
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/app
    volumes:
      - .:/app
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

#### README.md par dossier

**Structure de projet avec READMEs :**
```
project/
├── README.md           # Vue d'ensemble
├── AGENTS.md           # Pour l'IA
├── src/
│   ├── api/
│   │   ├── README.md   # "Module API: endpoints REST"
│   │   └── routes.py
│   ├── models/
│   │   ├── README.md   # "Models SQLAlchemy"
│   │   └── user.py
│   └── services/
│       ├── README.md   # "Logique métier"
│       └── auth.py
└── tests/
    └── README.md       # "Tests: conventions et coverage"
```

### TP Fil Rouge (30 min)

**Exercice 4.1 : Créer AGENTS.md**
- Analyser l'app démo
- Identifier les conventions implicites
- Rédiger AGENTS.md complet

**Exercice 4.2 : Vérifier l'impact**
- Lancer une tâche complexe sans AGENTS.md
- Même tâche avec AGENTS.md
- Comparer : qualité, tokens, temps

**Exercice 4.3 : Structurer un projet legacy**
- Prendre un projet sans structure
- Ajouter Makefile, READMEs, AGENTS.md
- Tester l'efficacité des agents

---

## Module 5 : Gestion des Coûts et Modèles Frugaux (55 min)

### Théorie (20 min)

#### Coûts des modèles

| Modèle | Input ($/1M tokens) | Output ($/1M tokens) | Notes |
|--------|---------------------|----------------------|-------|
| Claude 4 Opus | $15 | $75 | Ultra-performant |
| Claude 4 Sonnet | $3 | $15 | Excellent rapport Q/P |
| GPT-4o | $2.50 | $10 | Standard |
| Gemini 2.0 Flash | $0.10 | $0.40 | Très économique |
| DeepSeek V3 | $0.27 | $1.10 | Frugal et performant |
| DeepSeek R1 | $0.55 | $2.19 | Reasoning |

#### Stratégies d'optimisation

**1. Choisir le bon modèle pour la tâche**
- *Navigation/Exploration* → Gemini Flash, DeepSeek V3
- *Code simple* → Claude Sonnet, GPT-4o
- *Architecture complexe* → Claude Opus, o1

**2. Gérer le contexte**
- Compresser les conversations
- Résumer le contexte inutile
- Utiliser AGENTS.md pour éviter les répétitions

**3. Éviter les pièges**
- Erreurs en boucle → changer de modèle ou arrêter
- Recherche exhaustive non ciblée → affiner le prompt
- Tests itératifs sans borne → fixer un budget

#### Alertes de coûts réels

**Retours d'expérience :**
- Claude Code : $500-2000/mois en usage intensif
- App complexe : 800K tokens dépensés
- Gaspillage typique : $2847/mois sans monitoring

### TP Fil Rouge (35 min)

**Exercice 5.1 : Comparaison coût/perf**
- Même tâche avec 3 modèles différents
- Mesurer : temps, tokens, qualité
- Calculer le coût total

**Exercice 5.2 : Optimisation de contexte**
- Lancer une session longue (30+ messages)
- Identifier le contexte redondant
- Compresser et comparer

**Exercice 5.3 : Budget et limites**
- Configurer des alertes OpenRouter
- Fixer un budget quotidien
- Simuler un dépassement

---

## Module 6 : Multimodal et Cas Avancés (55 min)

### Théorie (15 min)

#### Modèles multimodaux
- **Vision** : Claude, GPT-4o, Gemini, Qwen
- **Audio** : GPT-4o-audio, Gemini
- **Usage** : Analyser screenshots, diagrammes, UI

#### Patterns spécifiques
- Screenshot → HTML/CSS
- Diagramme → Code
- UI → Tests E2E

#### Quirk Playwright
- Screenshot n'est pas forcément une image
- Utiliser `browser_snapshot` pour l'accessibilité
- Éviter de manipuler visuellement sans vérification

### TP Fil Rouge (40 min)

**Exercice 6.1 : Screenshot vers code**
- Capturer une UI existante
- Demander à l'IA de reproduire
- Comparer avec l'original

**Exercice 6.2 : UI Testing multimodal**
- Tester une interface web
- Capturer les états
- Générer des tests visuels

**Exercice 6.3 : Analyse de diagrammes**
- Fournir un diagramme d'architecture
- Demander l'implémentation
- Valider la corrélation

---

# JOUR 2 : Workflows Autonomes et Bonnes Pratiques

## Module 7 : Ralph Loop et PIV Pattern (55 min)

### Théorie (25 min)

#### Ralph Loop

**Concept** : Cycle autonome avec persistance d'état externe

```
Init → Task Selection → Execute → Verify → Persist → Loop
         ↑                                         ↓
         └─────────────────────────────────────────┘
```

**Composants clés :**
- `prd.json` - Spécifications
- `progress.txt` - État d'avancement
- `git history` - Historique des tentatives

**Implémentation :**
```bash
# Structure
.project/
├── prd.json          # Quoi faire
├── progress.txt      # Où on en est
├── state.json        # État courant
└── checkpoints/      # Points de sauvegarde
```

**Ressources :**
- GitHub: `snarktank/ralph`
- Blog: `ghuntley.com/loop/`

#### PIV Loop (Plan-Implement-Validate)

**Phase 1 - Prime**
- Charger le contexte projet
- Scanner la codebase
- Identifier les patterns existants

**Phase 2 - Implement**
- TDD strict : Red → Green → Refactor
- Écrire les tests d'abord
- Code minimal pour passer

**Phase 3 - Validate**
- Tests passent
- Coverage threshold atteint
- Pas de vulnérabilités sécurité

**Ressources :**
- GitHub: `galando/piv-speckit`

#### Human-in-the-Loop (HITL)

**Approche moderne : "On-the-loop"**
- L'agent avance en autonomie
- Points de contrôle humains
- Seuils de confiance
- Intervention et reprise

**Harness nécessaire :**
- Specs claires (PRD)
- Quality checks (tests, lint)
- Workflow guidance (AGENTS.md)

### TP Fil Rouge (30 min)

**Exercice 7.1 : Implémenter Ralph basique**
- Créer la structure de fichiers
- Lancer un cycle simple
- Observer la persistance

**Exercice 7.2 : PIV sur une feature**
- Feature: Système de commentaires
- Phase Prime: Analyse
- Phase Implement: TDD
- Phase Validate: Tests + lint

**Exercice 7.3 : HITL checkpoint**
- Configurer un seuil de confiance
- Laisser l'agent travailler
- Intervenir au checkpoint

---

## Module 8 : Debugging du Code Généré par IA (55 min)

### Théorie (25 min)

#### Les 8 patterns d'échec

**1. Happy Path Bias**
- L'IA code le cas nominal
- Oublie les edge cases
- *Fix*: Demander explicitement la gestion d'erreurs

**2. Context Missing**
- L'IA ne voit pas les dépendances
- Code incompatible avec le reste
- *Fix*: Fournir le contexte requis

**3. Fix Loop Infini**
- Erreur → Fix → Nouvelle erreur → Fix → ...
- Boucle sans fin
- *Fix*: Arrêter, analyser, changer d'approche

**4. Spéculation sans vérification**
- L'IA devine au lieu de vérifier
- Code basé sur des hypothèses fausses
- *Fix*: Forcer la lecture avant écriture

**5. Tool Calling Rogue**
- L'IA appelle des outils non pertinents
- Détruit des fichiers
- *Fix*: Valider avant exécution

**6. Hallucination de dépendances**
- Importe des libs qui n'existent pas
- *Fix*: Vérifier pip/npm avant

**7. SQL incorrect**
- Requêtes mal formées
- Jointures inexistantes
- *Fix*: Schéma explicite

**8. Suppression de tests**
- Pour "faire passer" le build
- *Fix*: Règle stricte dans AGENTS.md

#### Cas d'études réels

**"47 Subtle Bugs" (Devrim Ozcay)**
- Le code passe les tests
- Échec en production
- *Leçon*: Tests insuffisants, pas d'edge cases

**"Bankrupted in 47 Hours"**
- Code validé fonctionnellement
- Perte financière majeure
- *Leçon*: Business logic non testée

**Claude Code Issues:**
- SQL incorrect sur schéma complexe
- Rejet d'hypothèse utilisateur juste
- Fix spéculatif sans root cause
- Suppression de fichiers en cascade

### TP Fil Rouge (30 min)

**Exercice 8.1 : Diagnostiquer un fix loop**
- Plonger l'IA dans une boucle de correction
- Analyser les causes
- Briser le cycle

**Exercice 8.2 : Edge case injection**
- Code généré par l'IA
- Identifier les edge cases manquants
- Demander l'ajout

**Exercice 8.3 : Code review d'IA**
- Générer une feature complète
- Faire une revue systématique
- Identifier les risques

---

## Module 9 : Tests et Qualité avec IA (55 min)

### Théorie (20 min)

#### Test Generation Patterns

**TDD assisté par IA :**
1. Décrire la feature
2. IA génère les tests d'abord
3. IA implémente pour passer
4. Refactor ensemble

**Coverage improvement :**
```
Prompt: "Analyse la couverture de tests et identifie les branches manquantes dans src/auth/"
```

**Test types :**
- Unit tests → IA excelle
- Integration tests → Context nécessaire
- E2E tests → Multimodal utile

#### Qualité globale

**Lint et format :**
- Ruff, ESLint, Prettier
- Configurer dans AGENTS.md
- Validation automatique

**Security checks :**
- Détection de secrets
- Vulnérabilités dépendances
- Patterns dangereux

### TP Fil Rouge (35 min)

**Exercice 9.1 : TDD complet**
- Feature: Export CSV des tâches
- Écrire tests avec l'IA
- Implémenter pour passer
- Refactor

**Exercice 9.2 : Coverage boost**
- Analyser coverage actuel
- Générer tests pour branches manquantes
- Atteindre 80%+

**Exercice 9.3 : Integration tests**
- Scénario utilisateur complet
- Générer les tests
- Valider l'end-to-end

---

## Module 10 : Conventions d'Équipe et Tensions (55 min)

### Théorie (25 min)

#### Conventions collectives

**Code ownership :**
- Qui valide le code généré ?
- Responsabilité en cas de bug ?
- Review process adapté ?

**Standards de qualité :**
- Tests obligatoires ?
- Coverage minimum ?
- Lint strict ?

**Outils homogènes :**
- Même AGENTS.md pour tous ?
- Même modèle ?
- Même workflow ?

#### Tensions typiques

**Tension 1 : Vitesse vs Qualité**
- Certains veulent aller vite avec l'IA
- D'autres veulent du code "propre"

**Tension 2 : Apprentissage vs Dépendance**
- L'IA empêche-t-elle d'apprendre ?
- Junior vs Senior usage

**Tension 3 : Transparence vs Magie**
- Certains veulent tout voir (tool calling)
- D'autres veulent du "ça marche"

**Tension 4 : Coûts partagés**
- Qui paie les tokens ?
- Budget individuel vs collectif

#### Recommandations

**1. Documenter les choix**
- Charter IA dans l'équipe
- AGENTS.md unique et versionné

**2. Code review adapté**
- Tag "AI-generated" dans les PRs
- Points d'attention spécifiques

**3. Formation continue**
- Veille partagée
- Retours d'expérience réguliers

### TP Fil Rouge (30 min)

**Exercice 10.1 : Draft charter équipe**
- Identifier les conventions
- Rédiger les rules
- Définir le process de validation

**Exercice 10.2 : Simulation de conflit**
- Scénario: bug en prod causé par code IA
- Débrief: qui est responsable ?
- Process: comment éviter

**Exercice 10.3 : Workflow collaboratif**
-Configurer AGENTS.md pour une équipe
- Tester avec différents profils
- Ajuster

---

## Module 11 : Veille et Ressources (55 min)

### Théorie (30 min)

#### Sources de veille

**Blogs et newsletters :**
- *Anthropic Blog* - Prompt engineering
- *OpenAI Cookbook* - Patterns pratiques
- *Simon Willison's Blog* - Veille technique
- *Last Week in AI* - Newsletter hebdo

**Communautés :**
- *r/LocalLLaMA* - Modèles open source
- *r/ClaudeAI* - Usage Claude
- *Discord OpenCode/Claude Code*

**GitHub Trending :**
- Suivre `anthropics/*`, `openai/*`
- Projets `mcp-server-*`
- Awesome lists: `awesome-ai-tools`

#### Outils de veille

**Automatisés :**
- RSS feeds agrégés
- GitHub notifications ciblées
- Alertes Reddit/Hacker News

**Manuels :**
- Revue hebdomadaire
- Sharing session équipe

#### Ressources pédagogiques

**Tutoriels :**
- `microsoft/mcp-for-beginners` - 11 modules MCP
- `anthropics/prompt-eng-interactive-tutorial` - 34k stars
- `lm-academy/prompt-engineering-developers` - Labs

**Documentations :**
- Context7 pour docs à jour
- DeepWiki pour répos populaires

### TP Fil Rouge (25 min)

**Exercice 11.1 : Setup veille personnelle**
- Choisir 3 sources
- Configurer les alertes
- Premier tour de veille

**Exercice 11.2 : Analyser un nouvel outil**
- Prendre un projet récent
- Évaluer la pertinence
- Tester rapidement

**Exercice 11.3 : Partage équipe**
- Présenter une découverte
- Démonstration live
- Discussion

---

## Module 12 : Synthèse et Projet Final (55 min)

### Théorie (10 min)

#### Récapitulatif

**Jour 1 :**
- Typologie des outils
- Prompt engineering
- Tool calling & MCP
- Bonnes pratiques projet
- Coûts et modèles frugaux
- Multimodal

**Jour 2 :**
- Ralph Loop & PIV
- Debugging code IA
- Tests et qualité
- Conventions équipe
- Veille

#### Checklist post-formation

- [ ] AGENTS.md dans mon projet principal
- [ ] Workflow établi (modèle, tools)
- [ ] Alertes de coûts configurées
- [ ] Convention équipe discutée
- [ ] Sources de veille choisies

### TP Final (45 min)

**Projet : Refactor une feature complète**

**Étape 1 : Setup**
- Choisir une feature existante
- Lire toute la codebase pertinente
- Créer AGENTS.md si absent

**Étape 2 : Analyse**
- Documenter le comportement actuel
- Identifier les problèmes
- Proposer une amélioration

**Étape 3 : Planification**
- Écrire le PRD (prd.json)
- Définir les tests
- Plan d'implémentation

**Étape 4 : Exécution**
- TDD: tests d'abord
- Implémentation incrémentale
- Validation continue

**Étape 5 : Review**
- Self-review avec l'IA
- Identifier les risques
- Corriger

**Étape 6 : Livraison**
- PR bien documentée
- Changes expliqués
- Tests passent

---

## Annexes

### A. Modèles de AGENTS.md

#### Minimal (10-20 lignes)
```markdown
# AGENTS.md

## Stack
FastAPI + SQLAlchemy + PostgreSQL

## Commands
- `make dev` - Run dev server
- `make test` - Run tests
- `make lint` - Lint check

## Conventions
- Conventional commits
- Tests for new features
```

#### Complet (100+ lignes)
Voir exemples :
- `agentsmd/agents.md`
- `streamlit/e2e_playwright/AGENTS.md`

### B. Makefile complet

```makefile
.PHONY: help dev test lint build clean install

help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-15s\033[0m %s\n", $$1, $$2}'

install: ## Install dependencies
	pip install -r requirements.txt
	cd frontend && npm install

dev: ## Run development servers
	docker-compose up -d db redis
	uvicorn src.main:app --reload &
	cd frontend && npm run dev

test: ## Run all tests
	pytest tests/ -v --cov=src --cov-report=html

test-unit: ## Run unit tests only
	pytest tests/unit -v

test-integration: ## Run integration tests
	pytest tests/integration -v

lint: ## Run linters
	ruff check src/
	mypy src/
	cd frontend && npm run lint

build: ## Build production
	docker-compose build

clean: ## Clean up
	docker-compose down -v
	rm -rf .pytest_cache htmlcov

migration: ## Create migration
	alembic revision --autogenerate -m "$(MSG)"

upgrade: ## Upgrade DB
	alembic upgrade head

downgrade: ## Downgrade DB
	alembic downgrade -1
```

### C. Docker patterns

#### Multi-stage build
```dockerfile
# Build stage
FROM python:3.11-slim as builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Runtime stage
FROM python:3.11-slim

WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .

ENV PATH=/root/.local/bin:$PATH

CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]
```

#### Dev environment
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      target: builder
    volumes:
      - .:/app
      - venv:/app/.venv
    command: uvicorn main:app --reload
    environment:
      - PYTHONUNBUFFERED=1
    ports:
      - "8000:8000"

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  venv:
  pgdata:
```

### D. Ressources externes

**Tutoriels :**
- https://github.com/microsoft/mcp-for-beginners
- https://github.com/anthropics/prompt-eng-interactive-tutorial
- https://github.com/lm-academy/prompt-engineering-developers

**Patterns et workflows :**
- https://github.com/snarktank/ralph (Ralph Loop)
- https://github.com/galando/piv-speckit (PIV Loop)
- https://ghuntley.com/loop/ (Ralph Loop docs)

**AGENTS.md examples :**
- https://github.com/streamlit/e2e_playwright/blob/main/AGENTS.md
- https://github.com/anthropics/agents.md-patterns

**Veille :**
- https://www.reddit.com/r/LocalLLaMA/
- https://simonwillison.net/
- https://openai.com/blog/

**Outils mentionnés :**
- OpenCode: https://github.com/opencode-ai/opencode
- OpenRouter: https://openrouter.ai/
- Context7: https://context7.com/
- DeepWiki: https://deepwiki.com/

### E. Grille de coûts détaillée

| Contexte Claude | Coût estimé |
|-----------------|-------------|
| Session simple (10 msg) | ~$0.05 |
| Feature complexe (50 msg) | ~$0.50 |
| Refactor complet (200 msg) | ~$5-10 |
| Projet entier (1000+ msg) | ~$50-200 |

| Contexte DeepSeek | Coût estimé |
|-------------------|-------------|
| Session simple | ~$0.01 |
| Feature complexe | ~$0.10 |
| Refactor complet | ~$1-2 |
| Projet entier | ~$10-30 |

### F. FAQ

**Q: L'IA va-t-elle remplacer les développeurs ?**
R: Non, mais elle change la nature du travail. Les devs qui maîtrisent ces outils seront plus productifs et pourront se concentrer sur l'architecture, la logique métier et les décisions complexes.

**Q: Quel modèle choisir ?**
R: Pour apprendre: DeepSeek V3 ou Gemini Flash (économiques). Pour la production: Claude Sonnet (équilibre) ou Claude Opus (qualité max).

**Q: Comment convaincre mon équipe ?**
R: Commencez par un pilote sur un projet non critique. Mesurez les gains de productivité. Partagez les résultats concrets.

**Q: Comment gérer les bugs générés par l'IA ?**
R: Toujours review le code, écrire des tests, valider en staging. L'IA accélère mais ne remplace pas la validation humaine.

---

## Notes Implementateur

### Apps démo recommandées

**Option A - Full Stack (recommandé):**
- `fastapi/full-stack-fastapi-template` - 42k stars, React + PostgreSQL, complet

**Option B - Progressif:**
- Jour 1: `luchog01/minimalistic-fastapi-template` (~500 lignes)
- Jour 2: `edwinhern/express-typescript-2024` (stack différente)

**Option C - Frontend focus:**
- `swyxio/swyxkit` - SvelteKit minimal (~800 lignes)

### Pré-requis technique

**Pour les participants :**
- Python 3.11+ ou Node.js 18+
- Docker & docker-compose
- Git
- Éditeur de code (VSCode recommandé)
- Compte OpenRouter avec crédit ($10 minimum)

**Pour le formateur :**
- Tous les pré-requis participants
- Accès aux multiples modèles LLM
- Démos pré-préparées
- Slides(optionnel, préférer live coding)

### Timing suggéré

**Jour 1 (7h de formation):**
- 09:00-09:55: Module 1
- 09:55-10:10: Pause
- 10:10-11:05: Module 2
- 11:05-11:20: Pause
- 11:20-12:15: Module 3
- 12:15-14:00: Déjeuner
- 14:00-14:55: Module 4
- 14:55-15:10: Pause
- 15:10-16:05: Module 5
- 16:05-16:20: Pause
- 16:20-17:15: Module 6

**Jour 2 (7h de formation):**
- 09:00-09:55: Module 7
- 09:55-10:10: Pause
- 10:10-11:05: Module 8
- 11:05-11:20: Pause
- 11:20-12:15: Module 9
- 12:15-14:00: Déjeuner
- 14:00-14:55: Module 10
- 14:55-15:10: Pause
- 15:10-16:05: Module 11
- 16:05-16:20: Pause
- 16:20-17:15: Module 12

### Adaptations possibles

**Pour un public senior :**
- Réduire Module 1 & 2
- Approfondir Module 7 & 8
- Ajouter des cas d'architecture

**Pour un public junior :**
- Plus de temps sur Module 2 & 9
- Exercices plus guidés
- Moins d'autonomie attendue

**En format远程 :**
- Plus de breaks
- Sessions plus courtes (45 min)
- Partage d'écran obligatoire
- Chat actif pour questions

---

*Document généré avec l'aide d'OpenCode - Curriculum v1.0*
