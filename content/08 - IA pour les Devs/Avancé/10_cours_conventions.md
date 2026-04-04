---
title: "10 - Conventions d'Équipe et Tensions"
weight: 2040
---

## _Gérer l'IA dans une équipe de développement_

---

# Les tensions réelles

**Ce que les équipes vivent :**

| Tension | Exemple |
|---------|---------|
| **Productivité vs Qualité** | "L'IA code plus vite mais le code est moins maintenable" |
| **Apprentissage vs Dépendance** | "Les juniors ne comprennent pas ce que l'IA génère" |
| **Transparence vs Secret** | "Faut-il dire au client que l'IA a écrit le code ?" |
| **Standardisation vs Créativité** | "Tous les codeurs génèrent le même style" |

---

# Convention d'équipe IA

## Document à créer

```markdown
# CONVENTIONS_IA.md

## Usage Acceptable

### Interdit
- Commit du code IA sans review humain
- Utiliser l'IA pour des décisions d'architecture sans consensus
- Partager des secrets/API keys avec l'IA
- Générer du code sensible (auth, crypto) sans expert review

### Obligatoire
- Documenter les prompts significatifs dans `prompts/`
- Review IA code comme du code humain
- Maintenir une couverture de tests > 70%
- Signaler les limitations de l'IA dans les commentaires

### Recommandé
- Utiliser AGENTS.md pour le contexte projet
- Préférer les modèles frugaux pour les tâches simples
- Valider les dépendances générées
```

---

# Le "Code Review IA"

## Le reviewer humain doit vérifier

| Aspect | Question |
|--------|----------|
| **Lisibilité** | Le code est-il compréhensible par un humain ? |
| **Maintenabilité** | Un humain peut-il le modifier sans IA ? |
| **Tests** | Les tests testent-ils vraiment quelque chose ? |
| **Sécurité** | Pas de credentials, pas d'injection SQL ? |
| **Dépendances** | Les packages ajoutés sont-ils légitimes ? |

---

# Junior vs Senior avec l'IA

## Le piège

**Junior :**
```
Junior: "L'IA a généré ce code, je le commit."
Senior: "Tu comprends ce qu'il fait ?"
Junior: "Non, mais l'IA l'a dit."
```

**Le problème :** Le junior n'apprend pas, dépend de l'IA.

---

## La solution

```markdown
## Règle Junior-Senior

1. Le junior DOIT expliquer le code généré avant de commit
2. Le senior DOIT questionner les choix de l'IA
3. Les décisions d'architecture restent humaines
4. Le code incomprisn'est pas commit
```

---

# Transparence et Ethics

## Questions à se poser

1. **Faut-il déclarer l'usage de l'IA au client ?**
   - Recommandé pour les projets sensibles
   - Optionnel pour le code standard

2. **Qui est responsable si l'IA génère un bug ?**
   - Le développeur qui a commit
   - L'équipe qui a review

3. **Le code IA appartient-il à qui ?**
   - Au projet, comme tout code
   - Pas de différence légale avec le code humain

---

# Workflow d'équipe

## Processus recommandé

```
┌────────────────────────────────────────────────────────────┐
│                    WORKFLOW ÉQUIPE                          │
│                                                             │
│  1. Dev A génère avec IA                                   │
│          │                                                  │
│          ▼                                                  │
│  2. Dev A comprend le code (sinon refuse)                 │
│          │                                                  │
│          ▼                                                  │
│  3. Tests passent + lint OK                                │
│          │                                                  │
│          ▼                                                  │
│  4. PR avec label "ai-generated"                            │
│          │                                                  │
│          ▼                                                  │
│  5. Reviewer vérifie :                                      │
│     - Code quality                                          │
│     - Tests coverage                                        │
│     - Security                                              │
│          │                                                  │
│          ▼                                                  │
│  6. Merge siapprouvé                                       │
└────────────────────────────────────────────────────────────┘
```

---

# Label "ai-generated"

**Exemple de configuration GitLab/GitHub :**

```yaml
# .github/labeler.yml
ai-generated:
- any: ['**/*']
  all:
    - changed-files:
        any-glob-to-any-file: '**/*'
```

**Utilité :**
- Traçabilité du code IA
- Review plus approfondie
- Statistiques d'usage

---

# PR Review par IA

## Les bots existants

| Outil | Usage |
|-------|-------|
| **Jules** (Google) | Travaille sur les PR en background |
| **GitHub Copilot for PR** | Suggestions automatiques |
| **CodeRabbit** | Review automatique |

---

## Bonnes pratiques

```markdown
## PR Review Bot Rules

1. Bot comments are suggestions, not requirements
2. Human reviewer has final say
3. Bot reviews supplement, not replace human review
4. Security issues flagged by bot = MUST fix
```

---

# Gestion des conflits

## Cas typique : Dev A vs IA

**Situation :**
- Dev A écrit du code "à l'ancienne"
- Dev B génère du code IA
- Les deux approches sont différentes

**Solution :**

```markdown
## Résolution de Conflits

1. Comparer objectivement :
   - Lisibilité
   - Maintenabilité
   - Performance
   - Tests

2. Préférer la compréhension humaine
   - Le code que personne ne comprend = supprimer

3. Documenter le choix
   - Pourquoi cette approche ?
```

---

# TP : Convention d'équipe

Voir `10_tp_conventions.md` →