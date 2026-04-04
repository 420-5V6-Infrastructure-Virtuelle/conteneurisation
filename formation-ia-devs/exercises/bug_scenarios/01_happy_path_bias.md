# Bug Scenario 1: Happy Path Bias

## Contexte
L'agent a implémenté une fonctionnalité de "réinitialisation de mot de passe" qui fonctionne parfaitement dans le cas nominal, mais échoue silencieusement dans plusieurs cas limites.

## Code généré par l'IA

```python
# app/services/auth.py

async def reset_password(email: str, new_password: str) -> bool:
    """Reset user password and send confirmation email."""
    user = await get_user_by_email(email)
    
    # Validate password strength
    if len(new_password) < 8:
        raise ValueError("Password must be at least 8 characters")
    
    # Update password
    user.password_hash = hash_password(new_password)
    await save_user(user)
    
    # Send confirmation email
    await send_email(
        to=email,
        subject="Password Reset Complete",
        body=f"Your password has been reset successfully."
    )
    
    return True
```

## Le bug caché

L'agent n'a pas géré ces cas:

1. **Utilisateur inexistant** - `get_user_by_email` retourne `None`, crash sur `user.password_hash`
2. **Email déjà utilisé ailleurs** - Pas de vérification que l'email appartient au contexte
3. **Échec d'envoi email** - Si `send_email` échoue, le mot de passe est déjà changé
4. **Validation incomplète** - Pas de vérification de complexité (majuscules, chiffres, caractères spéciaux)

## Symptômes en production

```
ERROR: AttributeError: 'NoneType' object has no attribute 'password_hash'
  File "app/services/auth.py", line 12, in reset_password
    user.password_hash = hash_password(new_password)
```

## Exercice

1. **Identifier les edge cases manquants** (5 min)
   - Listez tous les cas où ce code peut échouer
   
2. **Corriger le code** (10 min)
   - Ajoutez la gestion d'erreurs appropriée
   - Assurez-vous que chaque erreur retourne un message clair
   
3. **Écrire les tests manquants** (5 min)
   - Tests pour chaque edge case identifié

## Correction attendue

```python
# app/services/auth.py

from typing import Optional
from enum import Enum

class ResetPasswordError(Enum):
    USER_NOT_FOUND = "user_not_found"
    INVALID_PASSWORD = "invalid_password"
    EMAIL_SEND_FAILED = "email_send_failed"

async def reset_password(email: str, new_password: str) -> tuple[bool, Optional[ResetPasswordError]]:
    """Reset user password and send confirmation email.
    
    Returns:
        (True, None) on success
        (False, error_code) on failure
    """
    # 1. Check user exists
    user = await get_user_by_email(email)
    if user is None:
        return False, ResetPasswordError.USER_NOT_FOUND
    
    # 2. Validate password strength (complete)
    if len(new_password) < 8:
        return False, ResetPasswordError.INVALID_PASSWORD
    if not any(c.isupper() for c in new_password):
        return False, ResetPasswordError.INVALID_PASSWORD
    if not any(c.isdigit() for c in new_password):
        return False, ResetPasswordError.INVALID_PASSWORD
    
    # 3. Store old hash for potential rollback
    old_hash = user.password_hash
    
    try:
        # Update password
        user.password_hash = hash_password(new_password)
        await save_user(user)
        
        # Send confirmation email
        email_sent = await send_email(
            to=email,
            subject="Password Reset Complete",
            body="Your password has been reset successfully."
        )
        
        if not email_sent:
            # Rollback on email failure
            user.password_hash = old_hash
            await save_user(user)
            return False, ResetPasswordError.EMAIL_SEND_FAILED
        
        return True, None
        
    except Exception as e:
        # Log and rollback on any unexpected error
        logger.error(f"Password reset failed for {email}: {e}")
        user.password_hash = old_hash
        await save_user(user)
        raise
```

## Tests attendus

```python
# tests/test_auth.py

import pytest
from app.services.auth import reset_password, ResetPasswordError

@pytest.mark.asyncio
async def test_reset_password_user_not_found():
    """Should return USER_NOT_FOUND error for non-existent email."""
    success, error = await reset_password("nonexistent@example.com", "NewPass123")
    assert success is False
    assert error == ResetPasswordError.USER_NOT_FOUND

@pytest.mark.asyncio
async def test_reset_password_weak_password():
    """Should reject passwords without uppercase/numbers."""
    # Too short
    success, error = await reset_password("user@example.com", "short")
    assert success is False
    assert error == ResetPasswordError.INVALID_PASSWORD
    
    # No uppercase
    success, error = await reset_password("user@example.com", "alllowercase123")
    assert success is False
    
    # No digit
    success, error = await reset_password("user@example.com", "NoDigitsHere")
    assert success is False

@pytest.mark.asyncio
async def test_reset_password_email_failure_rollback(db_user, mock_email_failure):
    """Should rollback password change if email fails."""
    old_hash = db_user.password_hash
    
    success, error = await reset_password(db_user.email, "NewPass123")
    
    assert success is False
    assert error == ResetPasswordError.EMAIL_SEND_FAILED
    
    # Verify rollback
    await db_user.refresh()
    assert db_user.password_hash == old_hash

@pytest.mark.asyncio
async def test_reset_password_success(db_user, mock_email_success):
    """Should successfully reset password and send email."""
    success, error = await reset_password(db_user.email, "ValidPass123")
    
    assert success is True
    assert error is None
    assert db_user.password_hash != db_user.original_password_hash
```

## Points clés à retenir

1. **Toujours demander les edge cases explicitement**
   - Prompt: "Quels cas limites dois-je gérer ?"
   
2. **Penser aux rollbacks**
   - Si une étape échoue, comment annuler les précédentes ?
   
3. **Tests = documentation des edge cases**
   - Si ce n'est pas testé, c'est que l'IA l'a oublié
