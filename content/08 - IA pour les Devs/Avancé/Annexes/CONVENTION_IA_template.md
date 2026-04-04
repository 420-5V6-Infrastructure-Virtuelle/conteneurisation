# Convention d'Équipe IA

> Template de convention pour l'usage de l'IA dans une équipe de développement

---

## 1. Usage acceptable

### Interdit

- ☠️ Commit de code non compris
- ☠️ Utiliser l'IA pour des décisions d'architecture sans consensus équipe
- ☠️ Partager des secrets/clés API avec l'IA
- ☠️ Générer du code sensible sans review expert
- ☠️ Désactiver les tests pour "faire passer" une PR

### Obligatoire

- ✅ Documenter les prompts significatifs (> 10 tours)
- ✅ Review humaine obligatoire pour tout code IA
- ✅ Maintenir couverture de tests > X%
- ✅ Signaler les limitations connues

### Recommandé

- 💡 Utiliser AGENTS.md dans chaque projet
- 💡 Préférer modèles frugaux (Haiku/Flash) pour tâches simples
- 💡 Valider les dépendances générées sur npm/pypi
- 💡 Monitorer les coûts mensuellement

---

## 2. Processus de Review

### Pour le reviewer

**Questions à se poser :**

1. Le code est-il compréhensible sans explication ?
2. Les tests sont-ils significatifs ?
3. Y a-t-il des credentials en clair ?
4. Les dépendances sont-elles légitimes ?
5. Le style est-il cohérent ?

### Label `ai-generated`

**Pourquoi :**
- Transparence sur l'origine du code
- Review plus approfondie
- Collecte de métriques

**Configuration GitHub :**

1. Settings → Labels → New label
2. Name : `ai-generated`
3. Color : `B8B8B8`
4. Description : "Code généré par IA - review approfondie requise"

**Utilisation :**

```bash
gh pr edit <number> --add-label ai-generated
```

---

## 3. Junior vs Senior

### Règle fondamentale

> **Le junior DOIT expliquer le code généré avant de pouvoir le commit.**

### Questions types pour validation

- "Pourquoi cette approche plutôt qu'une autre ?"
- "Quelles sont les alternatives possibles ?"
- "Comment fonctionne ce module ?"
- "Où sont les tests associés ?"
- "Quels sont les edge cases gérés ?"

### Validation senior

Pour les juniors (< 1 an dans l'équipe) :
- [ ] Code expliqué à un senior
- [ ] Senior signale la compréhension
- [ ] Tests validés ensemble

---

## 4. Transparence

### Communication client

| Type de projet | Obligation |
|----------------|------------|
| Projet sensible (banque, santé) | Déclarer l'usage IA |
| Code standard (CRUD, admin) | Non requis |
| Code open source | Mentionner dans README |

### Responsabilité

> Le développeur qui commit est **responsable** du code.
> L'équipe qui review est **coresponsable**.

---

## 5. Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                    WORKFLOW AVEC IA                          │
│                                                              │
│  1. Développeur génère avec IA                               │
│         │                                                    │
│         ▼                                                    │
│  2. Développeur COMPREND le code                             │
│     (sinon, pas de commit)                                   │
│         │                                                    │
│         ▼                                                    │
│  3. Tests + Lint OK                                          │
│         │                                                    │
│         ▼                                                    │
│  4. PR avec label "ai-generated"                             │
│         │                                                    │
│         ▼                                                    │
│  5. Review approfondie                                       │
│         │                                                    │
│         ▼                                                    │
│  6. Merge si approuvé                                        │
│         │                                                    │
│         ▼                                                    │
│  7. Documenter les learnings                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Gestion des conflits

### Situation type

- Dev A utilise IA pour générer → code rapide mais "standard"
- Dev B préfère code manuel → code plus "personnel"
- Conflit de style en review

### Résolution

1. **Comparer objectivement :**
   - Lisibilité
   - Maintenabilité
   - Performance
   - Tests
   - Documentation

2. **Privilégier la compréhension :**
   > Le code que personne ne comprend = supprimer

3. **Documenter le choix :**
   - Pourquoi cette approche a été retenue
   - Quelle convention pour les cas similaires

---

## 7. Métriques

### Quoi mesurer

| Métrique | Objectif | Comment mesurer |
|----------|----------|-----------------|
| % code avec label IA | Pas de cible, mais transparence | Compter PRs labelisées |
| Temps de review IA vs humain | Comparable | GitHub insights |
| Test coverage | > 70% | pytest --cov / jest --coverage |
| Bug rate IA vs humain | Comparable | Issues tracker |
| Cache hit rate | > 85% | `opencode stats` |
| Coût mensuel | < Budget | API dashboard |

### Tableau de bord

```markdown
# Metrics IA - [Mois/Année]

## Volume
- PRs totales : X
- PRs AI-generated : Y (Z%)

## Qualité
- Bugs en prod : X (Y% du total)
- Review time moyen : Xh (vs Yh baseline)

## Coûts
- Budget : $X
- Réel : $Y
- Cache hit rate : Z%

## Learnings
- [Ce qui a fonctionné]
- [Ce qui n'a pas fonctionné]
```

---

## 8. Formation

### Onboarding nouveaux développeurs

**Session 1 : Introduction (1h)**
- Démonstration d'OpenCode
- Exemples de prompts efficaces
- Pièges à éviter

**Session 2 : Pratique (2h)**
- Exercice guidé sur un projet
- Génération de tests
- Review de code IA

**Session 3 : Conventions (1h)**
- Cette convention
- AGENTS.md
- Workflow équipe

### Ressources

- [Lien vers documentation OpenCode]
- [Lien vers cours interne]
- [Lien vers AGENTS.md template]

---

## 9. Révision de la convention

### Fréquence

- Review trimestrielle
- Mise à jour selon retours équipe
- Alignement avec nouvelles versions outils

### Processus

1. Collecter les retours (issues, discussions)
2. Proposer modifications
3. Vote équipe
4. Mise à jour document

---

## 10. Annexes

### Annex A : Label GitHub

```yaml
name: ai-generated
color: B8B8B8
description: Code généré par IA - review approfondie requise
```

### Annex B : PR Template

Voir `PR_template_ai.md`

### Annex C : Checklist Review

Voir `REVIEW_checklist_ai.md`

---

# Historique des modifications

| Date | Version | Changements |
|------|---------|-------------|
| [Date] | 1.0 | Version initiale |

---

*Document maintenu par [Équipe] - Dernière mise à jour : [Date]*