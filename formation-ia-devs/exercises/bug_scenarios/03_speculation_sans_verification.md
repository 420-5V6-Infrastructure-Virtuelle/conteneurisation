# Bug Scenario 3: Spéculation sans Vérification

## Contexte
L'agent modifie du code sans avoir lu les fichiers de configuration ou les dépendances, créant une incompatibilité silencieuse.

## Code avant intervention

```python
# app/services/reporting.py

def generate_monthly_report(user_id: int, month: str) -> Report:
    """Generate monthly report for user."""
    # Existing working code
    data = fetch_user_data(user_id, month)
    processed = process_data(data)
    return Report(processed)
```

## L'intervention de l'agent

```
USER: "Optimise generate_monthly_report avec multiprocessing"

[00:01] AGENT:
✓ Edit app/services/reporting.py

Added:
from multiprocessing import Pool
import concurrent.futures

def generate_monthly_report(user_id: int, month: str) -> Report:
    """Generate monthly report for user with parallel processing."""
    data = fetch_user_data(user_id, month)
    
    with concurrent.futures.ProcessPoolExecutor(max_workers=4) as executor:
        # Split data into chunks for parallel processing
        chunks = split_into_chunks(data, 4)
        futures = [executor.submit(process_chunk, chunk) for chunk in chunks]
        results = [f.result() for f in concurrent.futures.as_completed(futures)]
    
    processed = merge_results(results)
    return Report(processed)

✓ Tests passed
✓ Lint clean
✓ Ready to commit
```

## Le bug caché (pas détecté par les tests)

L'agent n'a pas vérifié:

1. **Contexte d'exécution** - L'application tourne dans un conteneur Docker
2. **Type de serveur** - C'est un serveur ASGI (uvicorn) avec workers
3. **Sérialisation** - Les objets `Report` ne sont pas picklables
4. **Infrastructure** - Déjà multi-process au niveau du serveur

```python
# docker-compose.yml (PAS LU PAR L'AGENT!)

services:
  app:
    image: myapp:latest
    command: uvicorn app.main:app --workers 4  # DÉJÀ 4 WORKERS!
    deploy:
      replicas: 3  # 3 CONTENEURS EN PLUS!
```

Résultat: **4 workers uvicorn × 3 replicas × 4 process dans le code = 48 processus**

## Symptômes en production

```
CRITICAL: MemoryUsageError - Container memory exceeded (2Gi > 1.5Gi limit)
CRITICAL: PickleError - Cannot pickle Report object with lock
WARNING: High CPU throttling - requests taking 30s instead of 200ms
```

## Ce que l'agent aurait dû faire

```
USER: "Optimise generate_monthly_report avec multiprocessing"

AGENT: Avant d'optimiser, je dois vérifier le contexte:

[00:01] ✓ Read docker-compose.yml
[00:02] ✓ Read app/main.py
[00:03] ✓ Read requirements.txt

AGENT: J'ai détecté que:
1. L'app tourne déjà avec 4 workers uvicorn
2. Il y a 3 replicas en production
3. Les objets Report contiennent des verrous non-sérialisables

L'optimisation par multiprocessing serait contre-productive car:
- Risque de saturation mémoire (48 processus)
- Échec de sérialisation (Report non picklable)

RECOMMANDATION: Utiliser ThreadPoolExecutor dans le process existant, ou:
- Optimiser process_data avec vectorisation (numpy/pandas)
- Ajouter un cache Redis pour fetch_user_data
- Streamer les résultats au lieu de tout charger en mémoire

Voulez-vous que j'implémente une de ces alternatives?
```

## Exercice

1. **Identifier les fichiers non lus** (3 min)
   - Quels fichiers de configuration devraient être lus avant un refactoring ?
   - Pour chaque fichier, qu'est-ce qu'on cherche ?
   
2. **Écrire le prompt correctif** (2 min)
   - Un prompt qui force la vérification du contexte
   
3. **Proposer une alternative valide** (5 min)
   - Une optimisation qui tient compte du contexte

## Fichiers à vérifier avant optimisation

| Fichier | Ce qu'on cherche | Impact |
|---------|------------------|--------|
| `docker-compose.yml` | Workers, replicas, limits | Multi-process déjà en place |
| `requirements.txt` | Versions, incompatibilités | Dépendances |
| `app/main.py` | Type de serveur, config | ASGI/WSGI, workers |
| `.env` | Variables d'environnement | Secrets, flags |
| `tests/` | Couverture, mocks | Edge cases |

## Alternative correcte

```python
# app/services/reporting.py (CORRECT)

from concurrent.futures import ThreadPoolExecutor
import functools

# Cache avec TTL
@functools.lru_cache(maxsize=128)
def _fetch_user_data_cached(user_id: int, month: str) -> dict:
    """Cached wrapper for fetch_user_data."""
    return fetch_user_data(user_id, month)

def generate_monthly_report(user_id: int, month: str) -> Report:
    """Generate monthly report with optimized I/O parallelization.
    
    Using ThreadPoolExecutor (not ProcessPoolExecutor) because:
    - App already has multiple uvicorn workers
    - I/O-bound (fetch_user_data) benefits from threads
    - No pickling issues with Report objects
    """
    data = _fetch_user_data_cached(user_id, month)
    
    # Parallelize I/O, not CPU (threads, not processes)
    with ThreadPoolExecutor(max_workers=4) as executor:
        processed = process_data_parallel(data, executor)
    
    return Report(processed)

# Alternative: Externaliser à Celery/Redis pour longs rapports
# → Voir module 11 sur l'architecture async
```

## Points clés à retenir

1. **Toujours lire avant de coder**
   - Au minimum: `docker-compose.yml`, `requirements.txt`, tests concernés
   
2. **Poser des questions sur le contexte**
   - "Quel est le contexte d'exécution ?"
   - "Quelles sont les contraintes existantes ?"
   
3. **Recommander au lieu d'agir aveuglément**
   - "J'ai détecté X, voici les options..."
   - Attendre validation pour les actions risquées
   
4. **L'optimisation peut être négative**
   - Plus de processus ≠ meilleure performance
   - Contexte (workers, replicas) > Optimisation locale