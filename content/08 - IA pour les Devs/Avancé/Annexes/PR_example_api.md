# Pull Request - API Tasks

> **Exemple concret de PR avec code généré par IA**

---

## Description

Ajout de l'endpoint `GET /tasks/me` pour récupérer les tâches de l'utilisateur connecté.

La route utilise l'authentification OAuth2 et filtre automatiquement les tâches par utilisateur.

---

## Changements

- `src/routes/tasks.py` : Nouvel endpoint `GET /tasks/me`
- `src/services/task_service.py` : Méthode `get_by_user(user_id)`
- `tests/test_tasks.py` : Tests pour le nouvel endpoint

---

## AI Usage ✅

- [x] Code généré avec IA (Claude Sonnet)
- [x] Code reviewed par humain
- [x] Compris et validé

### Prompts utilisés

```markdown
1. "Add GET /tasks/me endpoint that returns only the authenticated user's tasks.
    Use the existing OAuth2 authentication.
    Return 401 if not authenticated.
    Write tests with mocked authentication."

2. "Review the code for security issues:
    - Ensure no SQL injection
    - Check authentication flow
    - Verify the user can only access their own tasks"
```

### Modifications humaines

- Renommé `_get_user_tasks` → `get_by_user` pour cohérence
- Ajouté docstring manquante sur l'endpoint
- Corrigé un edge case (utilisateur sans tâches)

---

## Tests

- [x] Tests unitaires passent (`make test`)
- [x] Coverage: 87% (objectif > 70%)
- [x] Tests d'intégration avec auth mockée

```
$ make test
============================= test session starts ==============================
tests/test_tasks.py::test_get_my_tasks PASSED                         [ 33%]
tests/test_tasks.py::test_get_my_tasks_empty PASSED                   [ 66%]
tests/test_tasks.py::test_get_my_tasks_unauthenticated PASSED         [100%]

============================== 3 passed in 0.45s ===============================

Coverage: 87%
```

---

## Checklist Review Humaine

### Sécurité

- [x] Pas de credentials en clair
- [x] Pas d'injection SQL (utilisation de SQLAlchemy ORM)
- [x] Authentification correctement vérifiée
- [x] User isolation: chaque utilisateur ne voit que ses tâches

### Code Quality

- [x] Compréhensible sans explication
- [x] Style cohérent (formatting, naming)
- [x] Docstrings sur les fonctions publiques
- [x] Type hints présents

### Dépendances

- [ ] Aucune nouvelle dépendance ajoutée

### Tests

- [x] Tests pertinents (happy path + edge cases)
- [x] Pas de `@skip` sur les tests nouveaux
- [x] Fixtures réutilisables (conftest.py)

---

## Captures d'écran (si UI)

N/A - API REST

---

## Questions pour les reviewers

1. Est-ce que le nom `get_by_user` est assez descriptif ou préférer `get_tasks_for_user` ?
2. Pensez-vous qu'on devrait paginer cet endpoint ou le laisser sans pagination pour MVP ?

---

## Coût IA

- Tokens input: ~45,000
- Tokens output: ~8,200
- Cache hit rate: 89%
- Coût estimé: $0.12

---

*PR générée avec le template `PR_template_ai.md`*