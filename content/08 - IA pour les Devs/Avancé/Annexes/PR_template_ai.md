# PR Template - Code généré par IA

> Template de Pull Request pour code généré avec assistance IA

---

## Description

[Décrire la fonctionnalité ou le bug fix de manière concise]

### Contexte

- **Issue liée :** #[numéro]
- **Type de changement :** 
  - [ ] Feature
  - [ ] Bug fix
  - [ ] Refactoring
  - [ ] Documentation
  - [ ] Test

---

## Changements

### Fichiers modifiés

| Fichier | Type de changement |
|---------|-------------------|
| `path/to/file.py` | Ajout/Modification/Suppression |
| `path/to/other.py` | Ajout/Modification/Suppression |

### Résumé des modifications

- [Changement 1]
- [Changement 2]
- [Changement 3]

---

## 🤖 Usage de l'IA

### Outils utilisés

- [ ] OpenCode
- [ ] Claude Code
- [ ] Cursor
- [ ] GitHub Copilot
- [ ] Autre : [préciser]

### Modèle principal

- [ ] Claude Haiku (rapide, peu coûteux)
- [ ] Claude Sonnet (standard)
- [ ] Claude Opus (complexe)
- [ ] GPT-4 Flash
- [ ] GPT-4
- [ ] Autre : [préciser]

### Nature de l'assistance

- [ ] Génération de code principal
- [ ] Génération de tests
- [ ] Documentation
- [ ] Refactoring
- [ ] Debugging assisté
- [ ] Review assistée

### Prompts significatifs

> Documenter les prompts importants (> 10 tours) pour partage d'équipe

```markdown
**Prompt :** 
[Le prompt utilisé]

**Résultat :**
[Ce qui a fonctionné ou non]

**Iteraions :**
[Nombre de tentatives nécessaires]
```

---

## ✅ Validation

### Compréhension

- [ ] J'ai lu et compris tout le code de cette PR
- [ ] Je peux expliquer ce code à un collègue
- [ ] Je connais les edge cases gérés

### Tests

- [ ] Tests unitaires passent (`make test`)
- [ ] Coverage > 70% pour le nouveau code
- [ ] Tests d'intégration OK
- [ ] Tests manuels effectués

**Coverage actuel :** [X]%

```bash
# Commande pour vérifier
make test-coverage
```

### Qualité

- [ ] Lint passe (`make lint`)
- [ ] Pas de credentials en clair
- [ ] Pas de TODO non résolus
- [ ] Style cohérent avec le reste du projet

### Dépendances

- [ ] Pas de nouvelles dépendances, OU
- [ ] Nouvelles dépendances vérifiées sur [npm/pypi/crates.io]
- [ ] License compatible
- [ ] Pas de dépendances avec vulnérabilités connues

---

## 📋 Checklist Review (pour le reviewer)

### Compréhension

- [ ] Le code est compréhensible sans explication supplémentaire
- [ ] Les noms de variables/fonctions sont clairs
- [ ] La logique est evidente ou bien commentée

### Qualité

- [ ] Pas de code mort
- [ ] Pas de duplication
- [ ] Gestion d'erreurs appropriée
- [ ] Pas de "magic numbers" sans explication

### Sécurité

- [ ] Pas de credentials/secrets en clair
- [ ] Inputs validés
- [ ] Pas d'APIs "hallucinées" (vérifier la doc)
- [ ] Pas de vulnérabilités évidentes

### Tests

- [ ] Tests significatifs (pas juste `expect(true)`)
- [ ] Edge cases couverts
- [ ] Cas d'erreur testés
- [ ] Mocks/stubs appropriés

### Performance

- [ ] Pas d'inefficacité évidente (N+1, boucles inutiles)
- [ ] Ressources correctement libérées
- [ ] Pas de mémoire leak potentiels

---

## 🧪 Comment tester

### Prérequis

```bash
# Setup nécessaire
[commandes]
```

### Étapes de test

1. [Étape 1]
2. [Étape 2]
3. [Étape 3]

### Résultats attendus

[Description du comportement attendu]

---

## 💰 Coûts (optionnel)

### Session

- **Tours de conversation :** [X]
- **Modèle principal :** [modèle]
- **Coût estimé :** $[X]

### Cache hit rate

- **Cache hit :** [X]%
- **Cache write :** [X] tokens
- **Cache read :** [X] tokens

---

## 📚 Documentation

- [ ] README mis à jour si nécessaire
- [ ] AGENTS.md mis à jour si patterns nouveaux
- [ ] Comments inline pour code complexe
- [ ] API documentation si applicable

---

## ⚠️ Points d'attention

### Risques connus

[Description des risques ou limitations]

### Questions ouvertes

- [Question 1]
- [Question 2]

---

## 🔗 Références

- [Lien vers la documentation]
- [Lien vers l'issue]
- [Lien vers discussion design]

---

## Screenshots (si applicable)

| Avant | Après |
|-------|-------|
| [image] | [image] |

---

## Reviewers

- [ ] @reviewer1
- [ ] @reviewer2

---

**Label à ajouter :** `ai-generated`

```bash
gh pr edit [number] --add-label ai-generated
```

---

*Template basé sur les bonnes pratiques de [codeintelligently.com](https://codeintelligently.com/blog/ai-code-review-checklist)*