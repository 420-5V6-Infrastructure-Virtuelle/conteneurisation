---
title: "12 - Projet Final"
weight: 2055
---

## _Intégrer l'IA dans votre workflow de développement_

---

# Objectif du module

Appliquer toutes les connaissances acquises dans un projet réel.

---

# Les ingrédients

## Ce que vous avez appris

| Module | Compétence |
|--------|-----------|
| 1-2 | Typologie d'agents, prompting, AGENTS.md |
| 3 | Tool calling, MCP |
| 4 | Bonnes pratiques, structure |
| 5 | Coûts, modèles frugaux |
| 6 | Multimodal |
| 7 | Ralph Loop |
| 8 | Debugging |
| 9 | Tests |
| 10 | Conventions équipe |
| 11 | Veille |

---

# Projet final

## Choix du projet

**Options :**

1. **Améliorer un projet existant** avec AGENTS.md + tests
2. **Créer un outil CLI** avec OpenCode
3. **Automatiser une tâche** avec un agent

**Critères de succès :**

- [ ] Utilise l'IA de manière substantielle
- [ ] AGENTS.md présent
- [ ] Tests validés par l'IA
- [ ] Documentation à jour
- [ ] Coût contrôlé

---

# Phase 1 : Setup

## Créer la structure

```bash
mkdir mon-projet-ia
cd mon-projet-ia

# Structure minimale
mkdir -p src tests docs
touch AGENTS.md Makefile README.md
```

---

## AGENTS.md pour le projet

```markdown
# AGENTS.md

## Contexte
Ce projet [description]. L'objectif est [objectif].

## Stack technique
- Langage : [Python/JS/Rust/etc.]
- Framework : [FastAPI/Express/etc.]
- DB : [PostgreSQL/etc.]

## Structure
- `src/` : Code source
- `tests/` : Tests
- `docs/` : Documentation

## Conventions
- Style : [ruff/black/prettier/etc.]
- Tests : pytest / jest
- Commit : conventional commits

## Workflow
1. Créer une branch feature/
2. Implémenter avec l'IA
3. Générer les tests
4. Valider tests +lint
5. PR avec description claire

## Coûts
- Budget : $X/mois
- Préférer : Haiku/Flash pour tâches simples
- Réserver Sonnet pour : debugging, complex logic

## Guardrails
- Pas de commit sans comprendre le code
- Pas de credentials en clair
- Valider les dépendances générées
```

---

# Phase 2 : Développement

## Workflow itératif

```
┌─────────────────────────────────────────────┐
│       DÉVELOPPEMENT AVEC IA                 │
│                                             │
│  1. DÉCRIRE dans AGENTS.md ou prompt       │
│         │                                   │
│         ▼                                   │
│  2. GÉNÉRER avec l'IA                      │
│         │                                   │
│         ▼                                   │
│  3. COMPRENDRE le code généré              │
│         │                                   │
│         ▼                                   │
│  4. COMPLÉTER manuellement si besoin       │
│         │                                   │
│         ▼                                   │
│  5. GÉNÉRER les tests                      │
│         │                                   │
│         ▼                                   │
│  6. VALIDER tests + lint                   │
│         │                                   │
│         ▼                                   │
│  7. DOCUMENTER                             │
│         │                                   │
│         ▼                                   │
│  8. COMMIT (seulement si compris)          │
└─────────────────────────────────────────────┘
```

---

## Session type

```bash
# Lancer l'agent
opencode

# Démarrer une tâche
> Implement [feature] following AGENTS.md

# Vérifier comprendre
> Explain this code to me

# Générer tests
> Write tests for [module]

# Valider
make test
make lint

# Documenter
> Update README with [changes]
```

---

# Phase 3 : Tests

## Stratégie

1. **Tests unitaires** : Générés par l'IA
2. **Tests d'intégration** : Scénarios critiques
3. **Validation manuelle** : Edge cases

---

## Exemple de génération

```bash
# Générer des tests pour un module
> Write pytest tests for src/auth.py covering:
- happy path
- invalid credentials
- expired tokens
- rate limiting

# Exécuter et valider
make test
```

---

# Phase 4 : Debugging

## Quand ça ne marche pas

1. **Copier l'erreur** dans le prompt
2. **Demander l'analyse**
3. **Proposer des hypothèses** si l'IA hésite

```markdown
> This test fails with:
[erreur complète]

The expected behavior is:
[comportement attendu]

What's wrong?
```

---

## Patterns de debugging

| Pattern | Solution |
|---------|----------|
| Hallucinated API | Vérifier la signature |
| Infinite loop | Ajouter timeout/guard |
| \"Done\" bug | Vérifier les assertions |
| Tool spam | Simplifier le prompt |

---

# Phase 5 : Intégration équipe

## PR avec label IA

```bash
# Créer la PR
gh pr create --title "feat: [feature]" \
  --body "Description de la feature"

# Ajouter le label
gh pr edit <number> --add-label ai-generated
```

---

## Review template

```markdown
## Review checklist (AI-generated code)

### Compréhension
- [ ] Le code est compréhensible
- [ ] Les tests couvrent les cas critiques
- [ ] Pas decredentials en clair

### Qualité
- [ ] Style cohérent
- [ ] Pas de code mort
- [ ] Dépendances légitimes

### Performance
- [ ] Pas d'inefficacité évidente
- [ ] Ressources correctement gérées
```

---

# Phase 6 : Rétrospective

## Ce qui a fonctionné

**Documenter :**

- Quels prompts ont été efficaces
- Quels modèles pour quelles tâches
- Combien de temps économisé

---

## Ce qui n'a pas fonctionné

**Documenter :**

- Les échecs (bugs, malentendus)
- Les coûts imprévus
- Les limitations

---

# Livrable final

## À remettre

1. **Code source** avec AGENTS.md
2. **Tests** passants
3. **Documentation** à jour
4. **Makefile** fonctionnel
5. **README** avec instructions

---

## Critères d'évaluation

| Critère | Points |
|---------|--------|
| Code qualité | 30% |
| Tests couverture | 25% |
| Documentation | 20% |
| Convention IA | 15% |
| Présentation | 10% |

---

# TP : Projet final

Voir `12_tp_projet.md` →