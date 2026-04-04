# AGENTS.md - API REST Python

> **Exemple concret pour un projet FastAPI**

---

## Contexte

API REST pour gestion de tâches (todo app). L'objectif est de fournir un backend JSON pour une application mobile.

---

## Stack technique

| Composant | Technologie |
|-----------|-------------|
| Langage | Python 3.11 |
| Framework | FastAPI 0.104 |
| ORM | SQLAlchemy 2.0 |
| Base de données | PostgreSQL 15 |
| Tests | pytest + pytest-asyncio |
| Lint | ruff + black |
| CI | GitHub Actions |

---

## Structure du projet

```
api/
├── src/
│   ├── main.py           # Point d'entrée FastAPI
│   ├── config.py         # Configuration (env vars)
│   ├── models/           # Modèles SQLAlchemy
│   │   ├── __init__.py
│   │   ├── task.py
│   │   └── user.py
│   ├── schemas/          # Pydantic schemas
│   │   ├── __init__.py
│   │   ├── task.py
│   │   └── user.py
│   ├── routes/           # Endpoints REST
│   │   ├── __init__.py
│   │   ├── tasks.py
│   │   └── auth.py
│   └── services/         # Logique métier
│       ├── __init__.py
│       └── task_service.py
├── tests/
│   ├── conftest.py       # Fixtures pytest
│   ├── test_tasks.py
│   └── test_auth.py
├── AGENTS.md             # Ce fichier
├── Makefile              # Commandes
├── pyproject.toml        # Dépendances
└── README.md
```

---

## Conventions

### Style de code

- **Black** pour le formatting (line length 88)
- **Ruff** pour le linting
- **Type hints** obligatoires sur toutes les fonctions publiques
- **Docstrings** format Google style

```python
# BON
def get_task(task_id: int, db: Session) -> Task | None:
    """Récupère une tâche par son ID.
    
    Args:
        task_id: L'identifiant de la tâche
        db: Session SQLAlchemy
    
    Returns:
        La tâche ou None si non trouvée
    """
    return db.query(Task).filter(Task.id == task_id).first()
```

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
| Exploration code | Haiku | Rapide, peu coûteux |
| Implémentation standard | Sonnet | Bon ratio qualité/coût |
| Architecture/Décisions | Opus | Capacités avancées |
| Debugging complexe | Opus | Raisonnement approfondi |

### Pattern Reflection → Implementation

```markdown
# ÉTAPE 1 : Reflection (modèle gratuit/cheap)
YOU: "Plan this feature. Don't implement yet.
     Think through edge cases, error handling.
     Output a detailed plan."

# ÉTAPE 2 : Implementation (modèle frugal)
YOU: "Now implement according to your plan."
```

### Guardrails

**OBLIGATOIRE :**
- [x] Générer tests pour chaque feature
- [x] Comprendre le code avant commit
- [x] Utiliser label `ai-generated` sur PRs
- [x] Passer par Makefile (`make test`, `make lint`)

**INTERDIT :**
- [ ] Commit de code non compris
- [ ] Credentials en clair dans le code
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

- Budget mensuel : $30
- Alert threshold : 80% ($24)

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
make test        # Lancer tests avec coverage
make lint        # Ruff + Black
make run         # uvicorn src.main:app --reload

# IA
opencode         # Démarrer session
opencode stats   # Stats usage
```

---

## Patterns spécifiques au projet

### Pagination

```python
# TOUJOURS utiliser ce pattern pour la pagination
@router.get("/tasks/")
async def list_tasks(
    skip: int = Query(0, ge=0),
    limit: int = Query(100, le=100),
    db: Session = Depends(get_db)
) -> List[TaskSchema]:
    return task_service.get_all(db, skip=skip, limit=limit)
```

### Authentification

```python
# Utiliser OAuth2PasswordBearer pour l'auth
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@router.get("/tasks/me")
async def get_my_tasks(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
):
    user = auth_service.get_current_user(db, token)
    return task_service.get_by_user(db, user.id)
```

---

## Points d'attention

### Ce qui fonctionne bien

- Demander des tests avant l'implémentation
- Faire générer les schemas Pydantic d'abord
- Demander "quels edge cases?" avant de coder

### Ce qui ne fonctionne pas

- Laisser l'ajouter des dépendances sans validation
- Générer des migrations DB sans review
- Grosses features en une seule demande → diviser

---

## Catastrophic Forgetting

**Tip du nom :** Si je m'appelle "Hadrien" dans ce fichier, l'agent doit m'appeler par mon nom au début de chaque message. Si l'agent arrête → contexte compressé = répéter les guardrails.

---

## Contacts

- Mainteneur principal : Hadrien
- Responsable technique : À définir

---

# Références

- FastAPI docs : https://fastapi.tiangolo.com
- SQLAlchemy 2.0 : https://docs.sqlalchemy.org/en/20/
- Ruff : https://docs.astral.sh/ruff/