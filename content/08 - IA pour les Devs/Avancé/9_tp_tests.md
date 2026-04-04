---
title: "9 - TP Tests et Qualité"
weight: 2035
---

## _Générer et valider des tests_

---

# Objectif

Apprendre à générer des tests de qualité avec l'IA et à les valider.

---

# Partie 1 : Génération de tests

##Étape 1 : Identifier une fonction à tester

```bash
# Choisir une fonction
cat src/services/user_service.py
```

---

##Étape 2 : Prompt structuré pour tests

```
Generate comprehensive tests for user_service.create_user():

Coverage requirements:
1. Happy path: user created successfully
2. Duplicate email: raises ValidationError
3. Invalid email format: raises ValidationError
4. Missing fields: raises ValidationError
5. Database error: handles gracefully

Use pytest with fixtures from tests/conftest.py.
Assert on:
- Return value (User object)
- Database state (user exists)
- Error messages (informative)

File: tests/test_user_service.py
```

---

##Étape 3 : Valider les tests générés

**Checklist de validation :**

- [ ] Les tests compilent-ils ?
- [ ] Les tests passent-ils ?
- [ ] Les assertions sont-elles significatives ?
- [ ] Les cas d'erreur sont-ils testés ?
- [ ] Les tests sont-ils indépendants ?

---

# Partie 2 : Coverage

##Étape 1 : Mesurer lecoverage

```bash
# Lancer avec coverage
pytest tests/ --cov=src --cov-report=html

# Voir le rapport
open htmlcov/index.html
```

---

##Étape 2 : Identifier les manques

**Chercher les lignes non couvertes :**

```bash
# Lignes manquantes
grep "MISSING" htmlcov/index.html
```

---

##Étape 3 : Demander des tests pour les manques

```
Generate tests to cover these uncovered lines:

[Lignes non couvertes]

Focus on edge cases and error paths.
```

---

# Partie 3 : Tests fragiles

##Étape 1 : Créer un test fragile

```python
# tests/test_fragile.py
def test_global_state():
    # Ce test dépend de l'état global
    assert len(User.query.all()) == 0
    create_user("test@example.com")
    assert len(User.query.all()) == 1
```

**Lancer plusieurs fois :**
- Passe-t-il toujours ?
- Change l'ordre des tests -> toujours ?

---

##Étape 2 : Corriger le test

```python
# tests/test_robust.py
def test_with_clean_state(db):
    # Utiliser un fixture qui nettoie
    initial = len(User.query.all())
    create_user("test@example.com")
    assert len(User.query.all()) == initial + 1
    # Cleanup automatique via fixture
```

---

# Partie 4 : TDD assisté

##Étape 1 : Écrire le test d'abord

```python
# tests/test_new_feature.py
def test_user_avatar_upload():
    """Test qu'un utilisateur peut uploader un avatar."""
    user = create_user()
    token = get_auth_token(user)
    
    response = client.post(
        f"/users/{user.id}/avatar",
        headers={"Authorization": f"Bearer {token}"},
        files={"file": ("avatar.png", b"PNGDATA", "image/png")}
    )
    
    assert response.status_code == 201
    assert "url" in response.json()
```

---

##Étape 2 : Demander l'implémentation

```
Make this test pass:

```python
[test content]
```

Constraints:
- Minimal implementation
- No extra features
- Handle file validation
- Store in /uploads/avatars/
```

---

##Étape 3 : Vérifier

```bash
# Les tests passent ?
pytest tests/test_new_feature.py -v

# L'implémentation est propre ?
cat src/api/routes/users.py
```

---

# Partie 5 : CI/CD setup

##Étape 1 : Pre-commit

```bash
# Installer pre-commit
pip install pre-commit

# Créer le fichier
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: local
    hooks:
      - id: test
        name: Run tests
        entry: pytest tests/
        language: system
        pass_filenames: false
      - id: lint
        name: Run linters
        entry: make lint
        language: system
        pass_filenames: false
EOF

# Activer
pre-commit install
```

---

##Étape 2 : Tester le pre-commit

```bash
# Essayer de commit sans tests
git add .
git commit -m "test"
# Le pre-commit devrait bloquer si tests échouent
```

---

# Livrable

À la fin de ce TP :

- [ ] Avoir généré des tests complets
- [ ] Avoir mesuré lecoverage
- [ ] Avoir corrigé des tests fragiles
- [ ] Avoir configuré pre-commit

---

# Checkpoint

**Pattern retenu :** Tests générés = à valider manuellement.

**Question clé :** Quel pourcentage decoverage visez-vous ?

---

# Prochain module

Module 10 : Conventions d'équipe et tensions.