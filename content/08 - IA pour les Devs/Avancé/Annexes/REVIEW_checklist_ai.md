# Checklist Review - Code généré par IA

> Checklist pour reviewer du code généré par IA (OpenCode, Claude Code, Cursor, etc.)

---

## Comment utiliser cette checklist

1. **Avant la review** : Vérifier le label `ai-generated` sur la PR
2. **Pendant la review** : Passer par chaque section
3. **Après la review** : Valider que tous les items critiques sont cochés

---

## 🎯 PHILOSOPHIE

> "Treat AI-generated code like junior developer work—helpful, but needs guidance."
> — [techdebt.best](https://techdebt.best/ai-code-review/)

**Key differences reviewing AI code :**

| Code humain | Code IA |
|-------------|---------|
| Incohérences style = signature humaine | Style parfait peut masquer bugs |
| Typos dans comments = normal | Comments "too perfect" = peut être halluciné |
| Approche personnelle | Approche "standard" | Possible manque de contexte |

---

## 1. COMPRÉHENSION

### Questions fondamentales

- [ ] **Le code est-il compréhensible ?**
  - Sans explication supplémentaire
  - Nommage clair
  - Logique evidente ou bien commentée

- [ ] **Le développeur peut-il expliquer ce code ?**
  - Demander une explication rapide
  - Vérifier qu'il comprend les edge cases
  - S'assurer qu'il connaît les alternatives possibles

- [ ] **Y a-t-il du code "magique" ?**
  - Fonctions qui semblent faire ce qu'elles ne font pas
  - Side effects cachés
  - Bugs potentiels masqués par un style parfait

### Red flags

```python
# RED FLAG: Variable jamais utilisée mais générée
unused_var = get_something()  # Hallucination?

# RED FLAG: API trop parfaite
result = api.get_user_posts(user_id)  # Cette API existe vraiment?

# RED FLAG: Commentaire trop générique
# Get all users from database
def get_users():  # Quelque soit le projet, ce pattern apparaît
```

---

## 2. SÉCURITÉ

### Credentials

- [ ] **Pas de secrets en clair**
  ```bash
  # Pattern à détecter
  API_KEY = "sk-..."
  PASSWORD = "..."
  SECRET = "..."
  ```

- [ ] **Pas de secrets dans la config**
  - Vérifier `.env.example` vs `.env`
  - Vérifier que `.env` est dans `.gitignore`

### APIs et dépendances

- [ ] **Les APIs appelées existent vraiment**
  - L'IA peut halluciner des méthodes
  - Vérifier la documentation officielle
  
- [ ] **Pas d'imports suspects**
  ```python
  # RED FLAG: Bibliothèque inventée
  from magic_auth import validate_token  # Ça existe?
  ```

### Inputs

- [ ] **Inputs validés**
  - Pas de blind trust des données
  - Validation des types
  - Sanitization si nécessaire

---

## 3. DÉPENDANCES

### Nouvelles dépendances

- [ ] **Chaque nouvelle dépendance est nécessaire**
  - Pourquoi celle-ci et pas une autre ?
  - Est-ce la meilleure option ?
  - Y a-t-il une alternative standard ?

- [ ] **Vérification sur le registry**
  - `npm view package` / `pip show package`
  - Dernière mise à jour ?
  - Téléchargements/mainteneurs ?

- [ ] **License compatible**
  - MIT, Apache, BSD → généralement OK
  - GPL → attention (copyleft)
  - Proprietary → vérifier

- [ ] **Pas de vulnérabilités connues**
  ```bash
  npm audit
  pip-audit
  cargo audit
  ```

### Dépendances suspectes

```json
// RED FLAG: Package typosquatted
{
  "react-native": "^18.0.0",  // Correct
  "react-nativ": "^1.0.0"     // Typosquatting!
}
```

---

## 4. TESTS

### Qualité des tests

- [ ] **Tests significatifs**
  - Pas juste `expect(true).toBe(true)`
  - Assertions qui testent vraiment
  
- [ ] **Edge cases couverts**
  - Valeurs limites
  - Cas d'erreur
  - Inputs nuls/vides

- [ ] **Tests d'erreur**
  - Exception handling
  - Error messages appropriés

### Coverage

- [ ] **Coverage > 70%** pour le nouveau code
  ```bash
  pytest --cov --cov-report=term-missing
  jest --coverage
  ```

- [ ] **Branches couvertes**
  - If/else
  - Try/catch
  - Switch cases

### Tests halllucinés

```python
# RED FLAG: Test qui ne teste rien
def test_function():
    result = my_function()
    assert result is not None  # Trop vague

# GOOD: Test explicite
def test_function_returns_sum():
    assert my_function(2, 3) == 5
    assert my_function(-1, 1) == 0
    assert my_function(0, 0) == 0
```

---

## 5. PERFORMANCE

### Patterns à vérifier

- [ ] **Pas de N+1 queries**
  ```python
  # RED FLAG
  for user in users:
      posts = get_posts(user.id)  # N queries!
  
  # BETTER
  posts = get_posts_batch([u.id for u in users])
  ```

- [ ] **Pas de boucles inutiles**
  - Peut-on utiliser une compréhension de liste ?
  - Y a-t-il une fonction built-in ?

- [ ] **Ressources libérées**
  - Connections fermées
  - Files fermés
  - Context managers utilisés

### Complexité

- [ ] **Complexité temporelle acceptable**
  - O(n²) justifié ?
  - Peut-on optimiser ?

- [ ] **Complexité spatiale acceptable**
  - Pas de listes infinies en mémoire
  - Generators quand approprié

---

## 6. STYLE ET CONVENTIONS

### Cohérence

- [ ] **Style cohérent avec le projet**
  - Même indentation
  - Même façon de nommer
  - Même façon de commenter

- [ ] **Conventions AGENTS.md respectées**
  - Patterns spécifiés
  - Outils configurés

### Code mort

- [ ] **Pas de code commenté**
  ```python
  # def old_function():
  #     ...  # Supprimer!
  ```

- [ ] **Pas de variables inutilisées**
- [ ] **Pas d'imports non utilisés**

---

## 7. DOCUMENTATION

### Code

- [ ] **Fonctions complexes documentées**
  - Docstrings
  - Inline comments pour logique tricky

- [ ] **README à jour**
  - Nouvelles fonctionnalités
  - Nouvelles dépendances
  - Nouveaux scripts

### Contexte IA

- [ ] **Prompts documentés** (si significatifs)
- [ ] **Limitations connues** mentionnées
- [ ] **Alternatives envisagées** documentées

---

## 8. LOGIQUE

### Bugs courants

- [ ] **Conditions correctes**
  ```python
  # RED FLAG: Off-by-one
  for i in range(len(items)):  # vs range(len(items) - 1)
  ```

- [ ] **Types corrects**
  - Comparaisons de types compatibles
  - Retours de types attendus

- [ ] **Mutation vs immutabilité**
  - La fonction modifie-t-elle ses inputs ?
  - Side effects attendus ?

### Hallucinations logiques

```python
# HALLUCINATION: API n'existe pas
my_list.sort(reverse=True, key=lambda x: x.id)  # OK
my_list.sort_ascending()  # N'existe pas!

# HALLUCINATION: Logique incorrecte
if user.is_admin or user.is_moderator:  # OK
if user.is_admin == True or user.is_moderator == True:  # Redondant
```

---

## 9. SPÉCIFIQUE OPENCODE/CLAUDE CODE

### Prompt Caching

- [ ] **Cache non cassé**
  - Même modèle pendant la session
  - Pas d'ajout/suppression d'outils
  - Cache hit rate > 80%

### Outils

- [ ] **Bash commands validées**
  - Pas de `rm -rf` sans vérification
  - Pas de commandes destructrices
  
- [ ] **Edit commands applicées correctement**
  - Vérifier les diffs
  - S'assurer que les bons fichiers sont modifiés

---

## 10. APPROBATION

### Conditions d'approbation

**OBLIGATOIRE :**
- [ ] Code compris par le développeur
- [ ] Tests passent
- [ ] Pas de secrets en clair
- [ ] Dépendances validées

**RECOMMANDÉ :**
- [ ] Coverage > 70%
- [ ] Documentation à jour
- [ ] Style cohérent

### Conditions de rejet

**REJETER SI :**
- ❌ Développeur ne comprend pas son code
- ❌ Tests insuffisants ou non significatifs
- ❌ Dépendances non vérifiées
- ❌ API hallucinée non corrigée

---

## RÉSUMÉ

| Catégorie | Items | Critiques |
|-----------|-------|-----------|
| Compréhension | 3 | ✅ Code compris |
| Sécurité | 5 | ✅ Pas de secrets |
| Dépendances | 4 | ✅ Vérifiées |
| Tests | 6 | ✅ Coverage > 70% |
| Performance | 5 | - |
| Style | 5 | - |
| Documentation | 5 | - |
| Logique | 5 | ✅ Pas d'hallucinations |
| OpenCode | 2 | - |

---

## Quick Reference Card

```
REVIEW RAPIDE CODE IA
═══════════════════════════════════════

1. COMPRÉHENSION → Demander explication
2. SÉCURITÉ → Pas de secrets
3. DÉPENDANCES → Vérifier existence
4. TESTS → Coverage > 70%
5. APIs → Vérifier documentation

RED FLAGS:
☠️ API non documentée
☠️ Variable jamais utilisée
☠️ Commentaire trop générique
☠️ Test qui ne teste rien
☠️ Dépendance suspecte
```

---

*Checklist basée sur les retours de [news.ycombinator.com](https://news.ycombinator.com/item?id=47460525) et [codeintelligently.com](https://codeintelligently.com/blog/ai-code-review-checklist)*