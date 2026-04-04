# Annexes - Exemples et Checklists

> **Exemples concrets et checklists pour l'utilisation de l'IA dans les projets**

---

## Liste des fichiers

| Fichier | Description | Usage |
|---------|-------------|-------|
| `AGENTS_example_api.md` | Exemple AGENTS.md pour API FastAPI | S'inspirer pour créer votre AGENTS.md |
| `MAKEFILE_example_api.md` | Exemple Makefile avec cibles IA | S'inspirer pour votre Makefile |
| `PR_example_api.md` | Exemple de PR avec code IA | S'inspirer pour vos PRs |
| `REVIEW_checklist_ai.md` | Checklist de review pour code IA | Utiliser pendant les reviews |
| `CONVENTION_IA_template.md` | Convention d'équipe pour l'usage IA | Adapter à votre équipe |

---

## ⚠️ Les AGENTS.md seront générés par LLM

**Le fichier AGENTS.md sera créé par votre agent IA.** 

Les exemples fournis (`AGENTS_example_api.md`) servent à :
1. Comprendre la structure attendue
2. Montrer le niveau de détail nécessaire
3. Donner des idées de sections à inclure

**Ne copiez pas l'exemple tel quel.** L'agent générera un AGENTS.md adapté à VOTRE projet.

---

## Comment utiliser ces exemples

### 1. AGENTS.md (généré par LLM)

```bash
# LANCER l'agent
opencode

# DEMANDER de créer l'AGENTS.md
> Create an AGENTS.md file for this project.
> Include: project context, stack, conventions, forbidden actions.
> See Annexes/AGENTS_example_api.md for the expected format.

# L'agent génère un AGENTS.md adapté à votre projet
# REVIEW et personnalisez si nécessaire
```

### 2. Makefile (personnaliser)

```bash
# Copier l'exemple comme point de départ
cp Annexes/MAKEFILE_example_api.md Makefile

# Adapter les variables
# - SRC_DIR := votre-répertoire-src
# - TEST_DIR := votre-répertoire-tests
# - COV_THRESHOLD := votre-seuil

# Personnaliser les cibles selon votre stack
# Voir "Personnalisation par stack" ci-dessous
```

### 3. PR Template (s'inspirer)

```bash
# Créer le template GitHub
mkdir -p .github

# Utiliser l'exemple comme référence
# - Sections AI Usage
# - Prompts utilisés
# - Modifications humaines
# - Coût IA

# Personnaliser pour votre équipe
```

---

## Workflow recommandé

```
┌─────────────────────────────────────────────────────────────┐
│            WORKFLOW AVEC EXEMPLES                           │
│                                                             │
│  1. NOUVEAU PROJET                                          │
│     │                                                       │
│     ├─ Lancer opencode                                      │
│     ├─ Demander création AGENTS.md                          │
│     │  (avec référence à AGENTS_example_api.md)             │
│     └─ L'agent génère le fichier adapté                     │
│                                                             │
│  2. DÉVELOPPEMENT                                           │
│     │                                                       │
│     ├─ make validate (avant commit)                         │
│     ├─ make test-coverage (vérifier 70%+)                   │
│     └─ make ai-stats (monitorer coûts)                      │
│                                                             │
│  3. PR                                                      │
│     │                                                       │
│     ├─ Créer PR avec sections AI Usage                      │
│     │  (s'inspirer de PR_example_api.md)                    │
│     ├─ Ajouter label ai-generated                           │
│     └─ Reviewer utilise REVIEW_checklist_ai.md              │
│                                                             │
│  4. ÉQUIPE                                                   │
│     │                                                       │
│     └─ Adapter CONVENTION_IA_template.md                    │
│        et partager avec l'équipe                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Personnalisation par stack

### Python

```makefile
# Ajouter dans Makefile
PYTEST := pytest
RUFF := ruff
BLACK := black

test: $(PYTEST) tests/ -v
lint: $(RUFF) check src/
format: $(BLACK) src/ tests/
```

### JavaScript/TypeScript

```makefile
# Ajouter dans Makefile
NPM := npm

test: $(NPM) test
lint: $(NPM) run lint
format: $(NPM) run format
```

### Rust

```makefile
# Ajouter dans Makefile
CARGO := cargo

test: $(CARGO) test
lint: $(CARGO) clippy
format: $(CARGO) fmt
```

---

## Intégration CI/CD

### GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Validate
        run: make validate
      
      - name: Check coverage
        run: make test-coverage
      
      - name: Check AI stats
        if: github.event_name == 'pull_request'
        run: make ai-stats
```

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - quality

test:
  stage: test
  script:
    - make validate
    - make test-coverage

quality:
  stage: quality
  script:
    - make lint
    - make ai-stats
```

---

## Références rapides

### Commandes essentielles

```bash
# Validation avant commit
make validate

# Tests avec coverage
make test-coverage

# Stats IA
make ai-stats

# Setup nouveau projet
make setup-ai

# Aide
make help
```

### Labels GitHub

```bash
# Créer label IA
gh label create ai-generated \
  --color B8B8B8 \
  --description "Code généré par IA - review approfondie requise"
```

### Fichiers à créer pour nouveau projet

```
projet/
├── AGENTS.md          # Workflow et conventions IA
├── Makefile           # Commandes avec cibles IA
├── README.md          # Documentation projet
├── requirements.txt   # Dépendances Python
├── requirements-dev.txt  # Dépendances dev
├── tests/             # Tests
└── src/               # Code source
```

---

*Annexes du cours "IA pour les Devs" - Module 12*