# Annexes - Templates et Checklists

> Templates et checklists pour l'utilisation de l'IA dans les projets

---

## Liste des templates

| Fichier | Description | Usage |
|---------|-------------|-------|
| `AGENTS_template.md` | Template de base pour AGENTS.md | Copier et personnaliser pour chaque projet |
| `CONVENTION_IA_template.md` | Convention d'équipe pour l'usage IA | Adapter à votre équipe |
| `PR_template_ai.md` | Template de PR pour code généré par IA | Utiliser pour toutes les PRs avec IA |
| `REVIEW_checklist_ai.md` | Checklist de review pour code IA | Pour les reviewers de PRs |
| `MAKEFILE_template.md` | Makefile avec cibles IA | Personnaliser selon le projet |

---

## Comment utiliser ces templates

### 1. AGENTS.md

```bash
# Copier le template
cp Annexes/AGENTS_template.md AGENTS.md

# Personnaliser
# - Modifier la description et l'objectif
# - Compléter la stack technique
# - Adapter les conventions
# - Définir le workflow IA
```

### 2. Convention d'équipe

```bash
# Créer le fichier
cp Annexes/CONVENTION_IA_template.md docs/CONVENTIONS_IA.md

# Adapter à votre équipe
# - Définir les règles acceptables/interdites
# - Compléter les métriques
# - Personnaliser le workflow

# Partager avec l'équipe
git add docs/CONVENTIONS_IA.md
git commit -m "docs: add AI conventions template"
```

### 3. PR Template

```bash
# Ajouter au projet
cp Annexes/PR_template_ai.md .github/PULL_REQUEST_TEMPLATE.md

# Personnaliser
# - Ajouter les checks spécifiques à votre projet
# - Adapter les questions selon vos besoins
```

### 4. Checklist Review

```bash
# Garder à portée de main
# Pendant les reviews, utiliser la checklist

# Ou intégrer dans votre outil de review
# - GitHub: PR template
# - GitLab: Merge request template
# - Bitbucket: Pull request template
```

### 5. Makefile

```bash
# Copier le template
cp Annexes/MAKEFILE_template.md Makefile

# Personnaliser
# - Adapter les chemins (SRC_DIR, TEST_DIR)
# - Modifier les commandes selon votre stack
# - Ajouter des cibles spécifiques

# Utiliser
make help
```

---

## Workflow recommandé

```
┌─────────────────────────────────────────────────────────────┐
│            WORKFLOW AVEC TEMPLATES                          │
│                                                             │
│  1. NOUVEAU PROJET                                          │
│     │                                                       │
│     ├─ Copier AGENTS_template.md → AGENTS.md                │
│     ├─ Copier MAKEFILE_template.md → Makefile                │
│     └─ Personnaliser selon le projet                         │
│                                                             │
│  2. DÉVELOPPEMENT                                           │
│     │                                                       │
│     ├─ make validate (avant commit)                         │
│     ├─ make test-coverage (vérifier 70%+)                   │
│     └─ make ai-stats (monitorer coûts)                      │
│                                                             │
│  3. PR                                                      │
│     │                                                       │
│     ├─ Utiliser PR_template_ai.md                           │
│     ├─ Ajouter label ai-generated                           │
│     └─ Reviewer utilise REVIEW_checklist_ai.md              │
│                                                             │
│  4. ÉQUIPE                                                   │
│     │                                                       │
│     └─ Partager CONVENTION_IA_template.md                   │
│        et adapter à l'équipe                                 │
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