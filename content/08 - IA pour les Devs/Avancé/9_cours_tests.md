---
title: "9 - Tests et Qualité avec l'IA"
weight: 2030
---

## _Générer et maintenir des tests de qualité_

---

# Le paradoxe des tests IA

**Problème :** L'agent peut générer des tests qui passent... mais ne testent rien.

```python
def test_delete_user():
    response = client.delete("/users/1")
    assert response.status_code == 200
    # Ce test passe, mais :
    # - Ne vérifie pas que l'utilisateur est supprimé
    # - Ne teste pas les cas d'erreur
    # - Ne vérifie pas les permissions
```

---

# Bonnes pratiques de test

## Structure d'un bon test

```python
def test_delete_user_owned():
    """Test qu'un utilisateur peut supprimer SON compte."""
    # Arrange
    user = create_user(email="test@example.com")
    token = get_auth_token(user)
    
    # Act
    response = client.delete(
        f"/users/{user.id}",
        headers={"Authorization": f"Bearer {token}"}
    )
    
    # Assert
    assert response.status_code == 200
    assert User.query.get(user.id) is None  # Vraiment supprimé
    
    # Cleanup
    # (si nécessaire)
```

---

# Générer des tests avec l'IA

## Prompt pour tests complets

```markdown
Generate tests for src/api/routes/users.py:

For each endpoint, include:
1. Happy path test
2. Authentication failure test
3. Authorization failure test
4. Input validation test
5. Edge cases test

Use pytest with fixtures from tests/conftest.py.
Assert on:
- Response status code
- Response body structure
- Database state changes

Do NOT:
- Create new fixtures without checking existing ones
- Skip cleanup
- Use mock unless explicit
```

---

# Coverage et métriques

## LeCoverage est un indicateur, pas un objectif

| Coverage | Interprétation |
|----------|----------------|
| < 50% | Insuffisant |
| 50-70% | Minimum acceptable |
| 70-85% | Bon |
| 85-95% | Excellent |
| > 95% | Soupçonner des tests inutiles |

**Attention :** 100% coverage couvre lignes, pas les chemins logiques.

---

# Le test-first avec l'IA

## Pattern : TDD assisté par IA

```bash
#Étape 1 : Écrire le test (humain)
# tests/test_new_feature.py
def test_new_feature():
    # Écrire le test AVANT le code
    pass

#Étape 2 : Générer le code (IA)
opencode --prompt "
Make this test pass:
$(cat tests/test_new_feature.py)

Constraints:
- Minimal implementation
- No extra features
- Match test expectations exactly
"

#Étape 3 : Refactor (humain)
# Améliorer la qualité du code
```

---

# Test de régression

## Ce que l'agent DOIT faire

Après chaque changement :

```bash
# 1. Lancer les tests
make test

# 2. Vérifier lecoverage
make coverage

# 3. Pas de test = pas de commit
# (config dans pre-commit)
```

---

# Les anti-patterns

## 1. Tests qui ne testent rien

```python
# ❌ Mauvais
def test_feature():
    # Ce test ne vérifie rien
    pass

# ✅ Bon
def test_feature():
    result = feature()
    assert result is not None
    assert result.status == "success"
```

## 2. Mock excessif

```python
# ❌ Trop de mocks
@mock.patch('module.Class1')
@mock.patch('module.Class2')
@mock.patch('module.Class3')
def test_feature(mock1, mock2, mock3):
    # Le test ne teste pas l'intégration
    pass

# ✅ Moins de mocks, plus d'intégration
def test_feature():
    result = real_feature()
    assert result.status == "success"
```

## 3. Tests fragiles

```python
# ❌ Fragile (dépend de l'ordre)
def test_user_creation():
    create_user("user1")
    create_user("user2")
    assert get_user_count() == 2  # Si user1 existe déjà...

# ✅ Robuste
def test_user_creation():
    initial_count = get_user_count()
    create_user(f"user_{uuid4()}")
    assert get_user_count() == initial_count + 1
```

---

# Tests et Ralph Loop

## Intégration dans le loop

```bash
# Le test est le critère de succès
while :; do
  opencode --prompt "$(cat TASK.md)"
  if make test && make lint; then
    echo "SUCCESS"
    break
  fi
done
```

**Pattern critique :** Les tests DOIVENT être passants AVANT que le loop ne se termine.

---

# CI/CD et IA

## Pre-commit hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: test
        name: Run tests
        entry: make test
        language: system
        pass_filenames: false
```

## GitHub Actions

```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make test
      - run: make coverage
```

---

# TP : Tests et Qualité

Voir `9_tp_tests.md` →