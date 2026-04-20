---
title: "10 - TP Conventions d'Équipe"
weight: 2045
---

## _Créer une convention IA pour l'équipe_

> ⏱ **1h**

---

# Objectif

Créer un document de convention pour l'usage de l'IA dans une équipe.

## Les tensions réelles

Avant de rédiger des règles, identifier les tensions que votre équipe vit déjà :

| Tension | Ce qu'on entend |
|---------|-----------------|
| **Productivité vs Qualité** | "L'IA code plus vite mais le code est moins maintenable" |
| **Apprentissage vs Dépendance** | "Les juniors ne comprennent pas ce qu'ils committent" |
<!-- | **Transparence vs Secret** | "Faut-il dire au client que l'IA a écrit le code ?" |
| **Standardisation vs Créativité** | "Tout le monde génère le même style fade" | -->

Une convention efficace adresse ces tensions — elle ne les ignore pas.

---

# Partie 1 : Audit de l'existant

##Étape 1 : Identifier les pratiques actuelles

**Questions à se poser :**

1. Qui utilise l'IA dans l'équipe ?
2. Quels outils sont utilisés ?
3. Quelles sont les plaintes ?
4. Quels sont les succès ?

---

##Étape 2 : Documenter les usages

```markdown
# AUDIT_IA.md

## Outils utilisés
- OpenCode : 3 personnes
- Cursor : 2 personnes
- Copilot : 5 personnes

## Succès rapportés
- Gain de temps sur les tests
- Documentation générée rapidement
- Refactoring assisté

## Problèmes rencontrés
- Code non compris par les juniors
- Styles hétérogènes
- Dépendances non validées
```

---

# Partie 2 : Rédiger la convention

## Template à compléter

```markdown
# CONVENTIONS_IA.md

## 1. Usage Acceptable

### Interdit
- [ ] Commit de code non compris
- [ ] Utiliser l'IA pour des décisions d'architecture sans consensus
- [ ] Partager des secrets avec l'IA
- [ ] Générer du code sensible sans expert review

### Obligatoire
- [ ] Documenter les prompts significatifs
- [ ] Review humaine obligatoire
- [ ] Maintenir couverture de tests > X%
- [ ] Signaler les limitations

### Recommandé
- [ ] Utiliser AGENTS.md
- [ ] Préférer modèles frugaux pour tâches simples
- [ ] Valider les dépendances générées

## 2. Processus de Review

### Pour le reviewer
- [ ] Le code est-il compréhensible ?
- [ ] Les tests sont-ils significatifs ?
- [ ] Pas decredentials ?
- [ ] Dépendances légitimes ?

### Label
- [ ] Ajouter label \"ai-generated\" sur les PR
- [ ] Review plus approfondie pour ce label

## 3. Junior vs Senior

### Règle
Le junior DOIT expliquer le code généré avant de commit.

### Questionstypes
- \"Pourquoi cette approche ?\"
- \"Quelles sont les alternatives ?\"
- \"Comment fonctionne ce module ?\"

## 4. Transparence

### Communication client
- Projets sensibles : déclarer l'usage IA
- Code standard : non requis

### Responsabilité
- Le développeur qui commit est responsable
- L'équipe qui review est coresponsable
```

---

# Partie 3 : Workflow

##Étape 1 : Définir le process

```
┌─────────────────────────────────────────────┐
│           WORKFLOW AVEC IA                  │
│                                             │
│  Dev génère avec IA                         │
│         │                                   │
│         ▼                                   │
│  Dev comprend le code                       │
│  (sinon refuse de commit)                   │
│         │                                   │
│         ▼                                   │
│  Tests + Lint OK                           │
│         │                                   │
│         ▼                                   │
│  PR avec label \"ai-generated\"              │
│         │                                   │
│         ▼                                   │
│  Review approfondie                         │
│         │                                   │
│         ▼                                   │
│  Merge siapprouvé                          │
└─────────────────────────────────────────────┘
```

---

##Étape 2 : Créer le label

**GitHub :**

```yaml
# Créer un label dans les settings
name: ai-generated
color: B8B8B8
description: Code généré parIA - review approfondie requise
```

**Utilisation :**

```bash
# Ajouter le label à une PR
gh pr edit <number> --add-label ai-generated
```

---

# Partie 4 : Gestion des conflits

## Scénario : Deux approches

**Situation :**
- Dev A utilise IA pour générer
- Dev B préfère code manuel
- Conflit de style

**Résolution :**

1. Comparer objectivement :
   - Lisibilité
   - Maintenabilité
   - Performance
   - Tests

2. Préférer la compréhension
   - Le code que personne comprend = supprimer

3. Documenter le choix

---

# Partie 5 : Jeu de rôle - Bug en production

## Exercice de mise en situation

**Scénario :** Vendredi 16h47, un bug critique est découvert en production. Le code coupable a été généré par IA.

Voir le [scénario complet](../exercises/roleplay_scripts/prod_bug_scenario.md).

**Participants :**
- **Dev A** : Auteur du code (commit sans comprendre)
- **Dev B** : Reviewer (a approuvé sans vérifier)
- **Lead Dev** : Médiateur
- **Product Owner** : Pressé par le client
- **Observateurs** : Prennent des notes

**Déroulement (15 minutes) :**
1. Découverte du bug (2 min)
2. Recherche du coupable (3 min)
3. Confrontation (5 min)
4. Résolution (5 min)
5. Débrief (hors temps)

**Questions à traiter :**
1. Qui est responsable ? (Auteur, Reviewer, IA)
2. Que dit la charte ?
3. Comment éviter la prochaine fois ?

---

# Partie 6 : Intégration GitHub

## Workflow PR avec un agent

L'intégration IA dans les PRs est pénible à configurer une fois, puis elle devient transparente. Le pattern est simple : vous taguez l'agent dans un commentaire GitHub, il lit la PR et répond (ou agit).

<!-- **Avec Codex CLI** — via l'intégration GitHub Actions :

```yaml
# .github/workflows/codex-review.yml
on:
  pull_request_review_comment:
    types: [created]

jobs:
  codex:
    if: contains(github.event.comment.body, '@codex')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          echo "${{ github.event.comment.body }}" | codex
``` -->

Ensuite dans une PR :
```
@codex What do you think about this approach?
@codex Are there edge cases I'm missing in this function?
```

L'agent lit le diff, le contexte de la PR, et répond dans les commentaires.

**Avec Claude Code** — via l'app GitHub officielle :

```bash
# Configurer depuis Claude Code :
/install-github-app
```

```
# Dans une issue ou PR GitHub :
@claude Fix this bug and create a PR
@claude Review this PR and flag security issues
```

Une fois installé, l'agent peut répondre aux issues, créer des branches, et ouvrir des PRs directement depuis GitHub — sans quitter l'interface.

**Mise en garde :** Ces agents lisent vos PRs et issues, qui peuvent contenir des données sensibles ou des tentatives de prompt injection (voir TP3). Limitez les permissions de l'app GitHub à ce qui est nécessaire.

---

# Partie 7 : Perspectives critiques sur l'IA en équipe

## Closed-source : enshittification sans préavis

Les modèles closed-source peuvent se dégrader silencieusement entre deux versions — sans changelog, sans notification. Un modèle qui était bon à l'implémentation peut devenir médiocre sur vos cas d'usage sans que vous le sachiez.

**En pratique :**
- Gardez un benchmark reproductible sur vos cas d'usage réels

## AI Fluency Index : le piège des artefacts

Selon l'[AI Fluency Index d'Anthropic](https://www.anthropic.com/research/AI-fluency-index), les gens "se shootent aux artefacts" — le mode génération d'artefacts (code, documents) crée un effet wow qui réduit l'esprit critique.

Le modèle de chat back-and-forth garde davantage d'esprit critique que la fuite en avant du "et rajoute ça aussi".

**Pour l'équipe :**
- Promouvoir le dialogue itératif plutôt que la génération en bloc
- Questionner chaque artefact plutôt que de l'accepter comme output final
- L'enthousiasme de l'agent n'est pas une validation

---

# Partie 8 : Metrics

## Quoi mesurer

| Métrique | Objectif |
|----------|-----------|
| % code IA | Pas de cible, mais transparence |
| Review time IA vs humain | Comparable |
| Test coverage | > 70% |
| Bug rate IA vs humain | Comparable |

---

# Livrable

À la fin de ce TP :

- [ ] Audit des pratiques documenté
- [ ] Convention IA rédigée
- [ ] Label \"ai-generated\" configuré
- [ ] Processus de review défini

---

# Checkpoint

**Pattern retenu :** L'IA ne commit pas, l'humain commit.

**Question clé :** Comment gérez-vous les désaccords sur le code IA ?

---

# Prochain module

Module 11 : Veille technologique.