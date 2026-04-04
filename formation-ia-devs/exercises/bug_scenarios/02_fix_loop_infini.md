# Bug Scenario 2: Fix Loop Infini

## Contexte
L'agent essaie d'ajouter une fonctionnalité de pagination à une API. Il fait une erreur, essaie de corriger, fait une nouvelle erreur, et ainsi de suite... pendant 15 minutes.

## Transcript de la boucle

```
[00:00] USER: "Ajoute une pagination à l'endpoint /users"

[00:01] AGENT: 
✓ Read app/routes/users.py
✓ Edit app/routes/users.py
Added: limit and offset parameters

[00:02] AGENT: Running tests...
✗ Test failed: "TypeError: 'NoneType' object is not iterable"

[00:03] AGENT: 
✓ Read app/services/user_service.py
✓ Edit app/services/user_service.py  
Fixed: return empty list instead of None

[00:04] AGENT: Running tests...
✗ Test failed: "AssertionError: Expected 10 users, got 0"

[00:05] AGENT:
✓ Read test_api.py
✓ Edit test_api.py
Fixed: mocked data

[00:06] AGENT: Running tests...
✗ Test failed: "ImportError: cannot import name 'paginate'"

[00:07] AGENT:
✓ Pip install paginate
✗ Command failed: "No package named 'paginate'"

[00:08] AGENT:
✓ Edit requirements.txt
Added: fastapi-pagination

[00:09] AGENT: Running tests...
✗ Test failed: "ValidationError: 'page' must be >= 1"

[00:10] AGENT:
✓ Edit app/routes/users.py
Fixed: default page=1 instead of page=0

[00:11] AGENT: Running tests...
✗ Test failed: "TypeError: 'Page' object is not iterable"

[00:12] AGENT: [continues trying fixes...]
```

## Problème fondamental

L'agent **tourne en rond** sans comprendre la cause racine. Chaque correction introduit un nouveau problème ou ne résout pas le vrai.

**Signaux d'alerte:**
- 3+ corrections sur le même fichier en moins de 10 minutes
- Tests qui échouent avec des erreurs différentes à chaque fois
- Type errors qui changent de nature (NoneType → ValidationError → TypeError)

## Exercice

1. **Identifier le moment d'arrêt** (2 min)
   - À quelle itération aurait-on dû arrêter l'agent ?
   
2. **Diagnostiquer la cause racine** (5 min)
   - Qu'est-ce qui ne va pas vraiment ?
   - Regarder les imports, les types de retour, les dépendances
   
3. **Proposer le bon prompt** (3 min)
   - Comment reformuler pour éviter la boucle ?

## Diagnostic attendu

**Le vrai problème:**
1. L'agent a utilisé `fastapi-pagination` sans vérifier les versions compatibles
2. Le type de retour `Page[User]` n'est pas itérable directement
3. Les tests mockaient mal les données paginées

**L'arrêt aurait dû se faire à [00:06]** (3ème erreur différente)

## Comment briser la boucle

### Méthode 1: Arrêter et analyser

```
USER: STOP. Résume les 3 dernières erreurs et leur pattern commun.
```

L'agent doit **reculer** avant d'avancer.

### Méthode 2: Changement de perspective

```
USER: Oublie les corrections précédentes. 
      Lis le code de fastapi-pagination dans node_modules.
      Montre-moi la signature correcte de paginate().
```

### Méthode 3: Revenir à un état propre

```
USER: git checkout -- app/routes/users.py app/services/user_service.py
      Recommence avec ce prompt:
      "Ajoute pagination en suivant EXACTEMENT la doc de fastapi-pagination.
       Commence par: pip show fastapi-pagination && lire la doc."
```

## Correction attendue

```python
# app/routes/users.py (CORRECT)

from fastapi import APIRouter, Query
from fastapi_pagination import Page, add_pagination
from fastapi_pagination.ext.sqlalchemy import paginate
from sqlalchemy import select

from app.models import User
from app.database import get_session

router = APIRouter()

@router.get("/users", response_model=Page[User])
async def list_users(
    page: int = Query(1, ge=1),
    size: int = Query(10, ge=1, le=100)
):
    """List users with pagination using fastapi-pagination."""
    async with get_session() as session:
        query = select(User).order_by(User.created_at.desc())
        return await paginate(session, query)

# NE PAS OUBLIER: add_pagination(app) dans main.py
```

```python
# tests/test_api.py (CORRECT)

import pytest
from httpx import AsyncClient
from fastapi_pagination import Page

from app.main import app

@pytest.mark.asyncio
async def test_list_users_paginated():
    """Test that /users returns paginated results."""
    async with AsyncClient(app=app, base_url="http://test") as client:
        # Create some users first
        for i in range(25):
            await client.post("/users", json={"name": f"User{i}", "email": f"user{i}@example.com"})
        
        # Test pagination
        response = await client.get("/users?page=1&size=10")
        assert response.status_code == 200
        
        data = response.json()
        assert "items" in data
        assert "total" in data
        assert "page" in data
        assert "size" in data
        
        # Verify structure matches Page model
        assert len(data["items"]) == 10
        assert data["page"] == 1
        assert data["size"] == 10
```

```
# requirements.txt (CORRECT)

fastapi>=0.100.0
fastapi-pagination>=0.12.0  # Version compatible avec votre FastAPI
sqlalchemy>=2.0.0
```

## Points clés à retenir

1. **Signaux de boucle:**
   - 3+ erreurs différentes en <10 min
   - Type errors qui changent de nature
   - Corrections sur corrections
   
2. **Quand arrêter:**
   - Dès que le pattern d'erreurs change de nature
   - Après 3 tentatives sur un même problème
   
3. **Comment repartir:**
   - `git checkout` pour revenir à un état propre
   - Demander à l'agent de **lire la doc** avant de coder
   - Réduire le scope (un fichier à la fois)