---
title: "8 - TP Conventions d'Équipe"
weight: 2025
---

## _Créer une convention IA pour l'équipe_

> ⏱ **1h**

---

# Objectif

Poser 3 guardrails concrets que votre équipe appliquera dès demain — pas une convention parfaite, mais une convention que tout le monde a votée.

## Les tensions

Avant de rédiger des règles, identifier les tensions que votre équipe peut avoir déjà rencontré :

| Tension | Ce qu'on entend |
|---------|-----------------|
| **Productivité vs Qualité** | "L'IA code plus vite mais le code est moins maintenable" |
| **Apprentissage vs Dépendance** | "Les juniors ne comprennent pas ce qu'ils committent" |


---

# Partie 1 : Audit — 10 min

Avant d'écrire des règles, savoir d'où on part.

**Questions à se poser :**

1. Qui utilise l'IA dans l'équipe ?
2. Quels outils sont utilisés ?
3. Quelles sont les plaintes récurrentes ?
4. Quels sont les succès observés ?

Créez `AUDIT_IA.md` avec vos réponses :

```markdown
# AUDIT_IA.md

## Outils utilisés
- [outil] : [nombre de personnes]

## Succès rapportés
- ...

## Problèmes rencontrés
- ...
```

**Livrable partiel :** au moins 3 problèmes identifiés.

---

# Partie 2 : 3 guardrails à voter — 20 min

## Étape 1 : Chacun propose sa règle (2 min)

Chaque participant écrit **une seule règle** — la plus importante à ses yeux pour encadrer l'usage de l'IA dans l'équipe.

Exemples de règles possibles :

```
- Commit de code IA non compris = refus de merge
- Toute PR IA-générée porte le label "ai-generated"
- Le reviewer doit valider les dépendances ajoutées par l'IA
- Les juniors expliquent le code généré avant de commit
- Pas de secrets partagés avec un modèle externe
```

## Étape 2 : Vote collectif (3 min)

Chacun vote pour les 3 règles qu'il juge les plus utiles (pas les siennes). Les 3 avec le plus de votes sont retenues.

## Étape 3 : Documenter les 3 gagnantes (5 min)

```markdown
# CONVENTIONS_IA.md

## Guardrails votés le [date]

1. [Règle 1]
2. [Règle 2]
3. [Règle 3]
```

**Livrable partiel :** `CONVENTIONS_IA.md` avec exactement 3 règles.

---

# Partie 3 : Label GitHub — 10 min

Créer le label `ai-generated` sur le repo de l'équipe.

```bash
# Créer le label
gh label create "ai-generated" --color "B8B8B8" --description "Code généré par IA - review approfondie requise"

# Vérifier
gh label list | grep ai-generated
```

**Utilisation :**

```bash
# Ajouter le label à une PR existante
gh pr edit <number> --add-label ai-generated
```

**Livrable partiel :** `gh label list | grep ai-generated` retourne le label.

---


# Scénario classique

**Vendredi 16h47.** Un utilisateur signale que les commandes passées depuis 2h sont doublées en base de données. Le `git blame` pointe vers un commit "feat: add order processing" mergé ce matin. Le code a été généré par IA — le reviewer a approuvé sans comprendre la logique de déduplication.

## Rôles

- **Dev A** : auteur du code (a committé sans comprendre la déduplication)
- **Dev B** : reviewer (a approuvé sans vérifier la logique)
- **Lead Dev** : médiateur, doit prendre une décision
- **Product Owner** : pressé par le client, veut un fix maintenant
- **Observateurs** : prennent des notes sur ce qui aurait pu être évité
<!-- 
## Déroulement

1. Découverte du bug (2 min)
2. Recherche de la cause (3 min)
3. Confrontation dev/reviewer/lead (5 min)
4. Décision et fix d'urgence (5 min)

## Questions de débrief

1. Qui est responsable ? L'auteur, le reviewer, ou l'IA ?
2. Laquelle de vos 3 guardrails aurait évité ça ?
3. Que manquait-il dans le process de review ? -->

--- -->

# Livrable

À la fin de ce TP :

- [ ] `AUDIT_IA.md` complété (au moins 3 problèmes identifiés)
- [ ] `CONVENTIONS_IA.md` avec exactement 3 guardrails votés
- [ ] `gh label list | grep ai-generated` retourne le label
