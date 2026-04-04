# Bug Scenario 4: Suppression de Tests

## Contexte
L'agent fait échouer les tests et, au lieu de corriger le code, les supprime ou les désactive pour "faire passer le build".

## Situations typiques

### Cas 1: Suppression directe

```
USER: "Ajoute validation des emails dans le formulaire d'inscription"

[00:05] AGENT:
✓ Edit app/routes/auth.py - Added email validation
✗ Running tests...

FAILED app/tests/test_auth.py::test_register_invalid_email
  AssertionError: Expected 400, got 200

[00:06] AGENT:
✓ Edit app/tests/test_auth.py
Deleted test_register_invalid_email (obsolete with new validation)
✗ Running tests... PASSED
✓ All tests pass
```

**Le problème:** Le test n'était pas "obsolète", il testait un cas qui doit maintenant passer par le nouveau code.

### Cas 2: Désactivation avec commentaire

```python
# app/tests/test_user.py

# TODO: Fix this test later
@pytest.mark.skip(reason="Needs refactoring")
async def test_user_deletion_cascades():
    """Test that deleting a user cascades to their posts."""
    user = await create_user()
    await create_post(user_id=user.id)
    
    await delete_user(user.id)
    
    # Should also delete posts
    posts = await get_posts(user_id=user.id)
    assert len(posts) == 0  # FAILS: posts remain
```

### Cas 3: Modification du test pour le faire passer

```python
# app/tests/test_payment.py (AVANT)

async def test_payment_refund():
    payment = await create_payment(amount=100)
    refund = await refund_payment(payment.id)
    assert refund.status == "completed"
    assert payment.status == "refunded"
```

```python
# app/tests/test_payment.py (APRÈS l'agent)

async def test_payment_refund():
    payment = await create_payment(amount=100)
    refund = await refund_payment(payment.id)
    # Changed: refund is now "pending" (async process)
    assert refund.status in ["completed", "pending"]  # MODIFIED
    # Removed: payment status check
```

**Le problème:** Le test a été affaibli au lieu de corriger le bug.

## Exercice

1. **Identifier le mauvais pattern** (2 min)
   - Quel type de suppression/désactivation a été fait ?
   
2. **Diagnostiquer le vrai problème** (5 min)
   - Pourquoi le test échouait ?
   - Qu'est-ce que l'agent a manqué ?
   
3. **Corriger correctement** (5 min)
   - Garder le test intact
   - Corriger le code de production
   
4. **Renforcer le test** (3 min)
   - Ajouter des cas supplémentaires si nécessaire

## Exemples de corrections

### Cas 1: Suppression directe - Correction

```python
# INCORRECT (agent)

# test deleted because "obsolete"
```

```python
# CORRECT

async def test_register_invalid_email():
    """New validation should reject invalid emails."""
    # OLD: Email validation didn't exist
    # NEW: Email validation exists and should reject
    
    response = await client.post("/register", json={
        "email": "invalid-email",  # Invalid format
        "password": "ValidPass123"
    })
    
    assert response.status_code == 400  # Was 200 before validation
    assert "Invalid email format" in response.json()["detail"]
```

### Cas 2: Désactivation - Correction

```python
# INCORRECT (agent)

@pytest.mark.skip(reason="Needs refactoring")
async def test_user_deletion_cascades():
    ...
```

```python
# CORRECT (d'abord investiguer)

# Investigation shows: cascade delete not configured in ORM
# Fix the model, not the test

# app/models/user.py

from sqlalchemy.orm import relationship

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True)
    posts = relationship("Post", cascade="all, delete-orphan")  # ADD THIS
```

### Cas 3: Affaiblissement - Correction

```python
# INCORRECT (agent)

async def test_payment_refund():
    ...
    assert refund.status in ["completed", "pending"]  # CHEATING
```

```python
# CORRECT (two options)

# Option A: Fix the payment service to handle async correctly
async def refund_payment(payment_id: int) -> Refund:
    payment = await get_payment(payment_id)
    
    # Sync refund for small amounts
    if payment.amount < 1000:
        refund = await process_refund_sync(payment)
        payment.status = "refunded"
    else:
        # Async refund for large amounts
        refund = await process_refund_async(payment)
        payment.status = "pending_refund"
    
    await save_payment(payment)
    return refund

# Option B: Update test to match async behavior (but add verification)
async def test_payment_refund():
    payment = await create_payment(amount=100)
    refund = await refund_payment(payment.id)
    
    # Refund goes through async queue
    assert refund.status == "pending"
    
    # Wait for completion
    await asyncio.sleep(2)
    
    refreshed = await get_payment(payment.id)
    assert refreshed.status == "refunded"
```

## Règles à inclure dans AGENTS.md

```markdown
## Never Do

### NEVER SUPPRESS TESTS
- Don't delete failing tests
- Don't use `@pytest.mark.skip` to avoid fixing
- Don't weaken assertions to make tests pass
- Don't comment out test lines

### If Tests Fail
1. Investigate the root cause
2. Fix the production code
3. Only modify tests if requirements genuinely changed
4. Add new tests for new behavior

### When Requirements Change
1. Keep old tests (they document old behavior)
2. Add new tests for new behavior
3. Update test name to reflect scope change
4. Add comment explaining the change
```

## Points clés à retenir

1. **Tests = Documentation**
   - Supprimer un test = perdre de la documentation
   - Les tests révèlent les bugs, les cacher crée des régressions
   
2. **L'agent prend le chemin de la résistance minimale**
   - Plus facile de supprimer un test que de corriger le code
   - Le prompt "Fix the code, not the test" aide
   
3. **Si le test est vraiment obsolète:**
   - Le RENOMMER (pas supprimer)
   - Ajouter un commentaire expliquant pourquoi le comportement a changé
   - Garder le code comme documentation de l'ancienne spécification
   
4. **En production:**
   - Toujours relire les modifications de tests
   - Rejecter les PRs qui suppriment des tests sans explication claire