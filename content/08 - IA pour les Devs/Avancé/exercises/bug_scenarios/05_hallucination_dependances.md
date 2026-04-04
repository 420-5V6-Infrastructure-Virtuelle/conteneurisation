---
title: "Hallucination de Dépendances"
weight: 5
---

# Bug Scenario 5: Hallucination de Dépendances

## Contexte
L'agent importe une librairie qui n'existe pas ou qui n'est pas compatible avec le projet.

## Exemple typique

```python
# app/services/analytics.py (GÉNÉRÉ PAR L'AGENT)

from flask_jwt_extended import jwt_required  # ❌ Projet FastAPI, pas Flask!
from mongoengine import Document  # ❌ Projet PostgreSQL, pas MongoDB!
from fastapi_cache import cache  # ❌ Package inexistant
import pendulum  # ❌ Pas dans requirements.txt
from typing_extensions import Final  # ❌ Python 3.8, pas besoin

@jwt_required()  # ❌ Décorateur Flask dans projet FastAPI
async def get_user_analytics(user_id: int):
    # Fetch from MongoDB... ❌ Projet utilise PostgreSQL
    user = await User.objects(id=user_id).first()
    
    # Use pendulum for dates... ❌ Aucune raison de ne pas utiliser datetime
    last_month = pendulum.now().subtract(months=1)
    
    return {"analytics": "data"}
```

## Ce qui s'est passé

### Scénario 1: Librairie inexistante

```
[00:01] AGENT: 
✓ Edit app/services/analytics.py
   from fastapi_cache import cache

[00:02] AGENT: 
✗ Running tests...
ImportError: cannot import name 'cache' from 'fastapi_cache'

[00:03] AGENT:
✓ Pip install fastapi-cache
✗ Command failed: No matching distribution found

[00:04] AGENT: [ESSAIE D'AUTRES CHOSES...]
```

**Le problème:** `fastapi-cache` vs `fastapi-cache2` vs `fastapi-caching` → noms confus, l'agent devine.

### Scénario 2: Librairie incompatible

```
[00:01] AGENT:
✓ Edit app/services/auth.py
   from flask_jwt_extended import jwt_required

[00:02] ✓ Tests pass (mocks!)
[00:03] ✓ Lint clean
[00:04] ✓ Commit
```

**Le problème:** Les tests passent avec des mocks, mais l'import échouera en production.

```python
# tests/conftest.py (LE VRAI PROBLÈME)

# Les mocks cachent les imports incorrects!
@pytest.fixture
def mock_jwt():
    return MagicMock()  # Pas de validation réelle de l'import
```

### Scénario 3: Stack technique incorrecte

```
[00:01] AGENT: [N'A PAS LU LE PROJET]
✓ Edit app/models/user.py
   from mongoengine import Document

[00:02] AGENT:
✓ Models look good
✓ Tests pass (avec fixtures mockées)
```

**Le problème:** L'agent a assumé MongoDB alors que le projet utilise SQLAlchemy/PostgreSQL.

## Exercice

### Partie 1: Identifier les hallucinations (5 min)

Regarder ce code et lister les imports problématiques:

```python
# app/services/notification.py

from celery import Celery  # ❓
from sendgrid import SendGridAPIClient  # ❓
from twilio.rest import Client as TwilioClient  # ❓
import arrow  # ❓
from pydantic import BaseModel  # ✓ Probablement OK

app = Celery('tasks', broker='redis://localhost:6379')  # ❓

async def send_notification(user_id: int, message: str):
    # Use arrow for timezone handling  # ❓
    sent_at = arrow.now().isoformat()
    
    # Send via SendGrid  # ❓
    sg = SendGridAPIClient(os.environ.get('SENDGRID_API_KEY'))
    ...
```

Questions:
1. Comment savoir si celery/sendgrid/twilio/arrow sont dans le projet ?
2. Quels fichiers lire pour vérifier ?
3. Comment éviter ce problème ?

### Partie 2: Corriger les imports (10 min)

```python
# requirements.txt (EXISANT)

fastapi>=0.100.0
sqlalchemy>=2.0.0
psycopg2-binary>=2.9.0
pydantic>=2.0.0
python-jose[cryptography]>=3.3.0
redis>=4.5.0
```

```python
# app/services/notification.py (À CORRIGER)

# Remplacer les imports par des alternatives valides
# Exemple: arrow → datetime (déjà dans stdlib)
# Exemple: SendGrid → utiliser l'API REST directement ou fastapi-mail

from datetime import datetime, timezone
from typing import Optional
import httpx
from pydantic import BaseModel

from app.config import settings

async def send_notification(user_id: int, message: str) -> dict:
    """Send notification using configured provider."""
    sent_at = datetime.now(timezone.utc).isoformat()
    
    # Use fastapi-mail (si disponible) ou httpx direct
    async with httpx.AsyncClient() as client:
        # Appel API direct au lieu de dépendre de SendGrid SDK
        response = await client.post(
            f"{settings.notification_api_url}/send",
            json={"user_id": user_id, "message": message}
        )
        return response.json()
```

### Partie 3: Validation des dépendances (5 min)

Créer un script de validation:

```python
# scripts/check_imports.py

"""Check that all imports in the codebase are in requirements.txt."""

import ast
import sys
from pathlib import Path
from typing import Set

def get_imports_from_file(filepath: Path) -> Set[str]:
    """Extract all import names from a Python file."""
    with open(filepath) as f:
        tree = ast.parse(f.read())
    
    imports = set()
    for node in ast.walk(tree):
        if isinstance(node, ast.Import):
            for alias in node.names:
                imports.add(alias.name.split('.')[0])
        elif isinstance(node, ast.ImportFrom):
            if node.module:
                imports.add(node.module.split('.')[0])
    
    return imports

def get_stdlib_modules() -> Set[str]:
    """Return Python standard library module names."""
    # Simplified - in production, use stdlib_list package
    return {
        'abc', 'asyncio', 'collections', 'concurrent', 'contextlib',
        'datetime', 'functools', 'itertools', 'json', 'logging',
        'os', 'pathlib', 're', 'sys', 'typing', 'unittest', 'uuid',
    }

def get_requirements(filepath: Path) -> Set[str]:
    """Parse requirements.txt."""
    requirements = set()
    with open(filepath) as f:
        for line in f:
            line = line.strip()
            if line and not line.startswith('#'):
                # Extract package name (before version specifier)
                pkg = line.split('>=')[0].split('==')[0].split('<')[0].split('>')[0].split('[')[0]
                requirements.add(pkg.lower().replace('-', '_'))
    return requirements

def main():
    project_root = Path(__file__).parent.parent
    stdlib = get_stdlib_modules()
    
    # Get all imports
    all_imports = set()
    for py_file in project_root.rglob('*.py'):
        if 'venv' not in str(py_file) and '__pycache__' not in str(py_file):
            all_imports.update(get_imports_from_file(py_file))
    
    # Get requirements
    req_file = project_root / 'requirements.txt'
    requirements = get_requirements(req_file) if req_file.exists() else set()
    
    # Find missing
    third_party = all_imports - stdlib - {'app'}  # 'app' is local
    missing = third_party - requirements
    
    if missing:
        print("❌ Missing dependencies:")
        for pkg in sorted(missing):
            print(f"  - {pkg}")
        print("\nAdd to requirements.txt:")
        print(f"  {' '.join(sorted(missing))}")
        sys.exit(1)
    else:
        print("✓ All dependencies are declared")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

## Prévention: Règles AGENTS.md

```markdown
## Dependencies

### Before Adding New Imports
1. ✓ Check requirements.txt for installed packages
2. ✓ Check if stdlib equivalent exists (datetime vs arrow)
3. ✓ Check if alternative from existing packages works
4. ✓ Only then, propose adding new dependency

### Standard Library Preferences
- datetime over arrow, pendulum
- asyncio over trio (unless explicitly async project)
- logging over structlog (unless already in project)
- json over orjson (unless perf-critical)

### Never Assume
- Check requirements.txt, pyproject.toml, or package.json
- Ask: "What packages are installed?" before adding imports
```

## Points clés à retenir

1. **Toujours vérifier les imports**
   - L'agent "devine" des packages qui n'existent pas
   - Regex sur requirements.txt avant d'importer
   
2. **Les mocks cachent les erreurs d'import**
   - Tests passent en local, échouent en production
   - Utiliser des tests d'intégration sans mocks
   
3. **Préférer la stdlib**
   - datetime, asyncio, logging, json → toujours disponibles
   - arrow, pendulum, structlog, orjson → nécessitent installation
   
4. **Script de validation**
   - `check_imports.py` en CI
   - Détecte les imports non déclarés avant déploiement