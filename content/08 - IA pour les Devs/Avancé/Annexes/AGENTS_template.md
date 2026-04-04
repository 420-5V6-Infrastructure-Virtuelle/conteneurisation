# AGENTS.md Template

> Template de base pour configurer un projet avec OpenCode/Claude Code

---

## Contexte

Ce projet [BREVE DESCRIPTION]. L'objectif est [OBJECTIF PRINCIPAL].

---

## Stack technique

| Composant | Technologie |
|-----------|-------------|
| Langage | [Python/JavaScript/Rust/Go/etc.] |
| Framework | [FastAPI/Express/React/etc.] |
| Base de données | [PostgreSQL/MongoDB/etc.] |
| Tests | [pytest/jest/etc.] |
| Lint | [ruff/black/prettier/etc.] |

---

## Structure du projet

```
projet/
├── src/           # Code source
│   ├── [module1]/
│   ├── [module2]/
│   └── main.py
├── tests/         # Tests
│   ├── test_[module1].py
│   └── test_[module2].py
├── docs/          # Documentation
├── AGENTS.md      # Ce fichier
├── Makefile       # Commandes
└── README.md      # Présentation
```

---

## Conventions

### Style de code

- [Linter utilisé] pour le formatting
- [Convention de nommage]
- [Patterns préférés]

### Commits

```
type(scope): description courte

type = feat | fix | docs | style | refactor | test | chore
```

### Branches

- `main` : Production
- `feature/*` : Nouvelles fonctionnalités
- `fix/*` : Corrections de bugs

---

## Workflow IA

### Modèles à utiliser

| Tâche | Modèle | Raison |
|-------|--------|--------|
| Exploration code | Haiku/Flash | Rapide, peu coûteux |
| Implémentation standard | Sonnet | Bon ratio qualité/coût |
| Architecture/Décisions | Opus | Capacités avancées |
| Debugging complexe | Opus | Raisonnement approfondi |

### Guardrails

**OBLIGATOIRE :**
- [ ] Générer tests pour chaque feature
- [ ] Comprendre le code avant commit
- [ ] Utiliser label `ai-generated` sur PRs
- [ ] Valider dépendances générées

**INTERDIT :**
- [ ] Commit de code non compris
- [ ] Credentials en clair
- [ ] Ignorer les warnings de linter
- [ ] Désactiver les tests

### Prompt Caching

Pour optimiser les coûts :
- ✅ Garder le même modèle pendant la session
- ✅ Ne pas ajouter/retirer d'outils mid-session
- ✅ Mettre les données dynamiques en messages
- ❌ Ne pas modifier le system prompt
- ❌ Ne pas inclure timestamp en début de prompt

---

## Coûts

### Budget

- Budget mensuel : $[X]
- Alert threshold : [Y]%

### Suivi

```bash
# Vérifier le cache hit rate (objectif > 85%)
opencode stats --cache

# Vérifier les coûts du mois
opencode stats --costs
```

---

## Commandes utiles

```bash
# Développement
make install     # Installer dépendances
make test        # Lancer tests
make lint        # Linter
make run         # Lancer l'app

# IA
opencode         # Démarrer session
opencode stats   # Stats usage
```

---

## Patterns spécifiques au projet

### [Pattern 1]

```python
# Exemple de pattern utilisé dans ce projet
def example():
    pass
```

### [Pattern 2]

```python
# Autre pattern important
def another_example():
    pass
```

---

## Points d'attention

### Ce qui fonctionne bien

- [Liste des approches validées]

### Ce qui ne fonctionne pas

- [Liste des pièges à éviter]

---

## Contacts

- Mainteneur principal : [Nom]
- Responsable technique : [Nom]

---

# Références

- [Lien vers documentation framework]
- [Lien vers conventions équipe]
- [Lien vers ARCHITECTURE.md si existant]