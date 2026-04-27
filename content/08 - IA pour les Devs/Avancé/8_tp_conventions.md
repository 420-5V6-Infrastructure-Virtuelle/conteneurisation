---
title: "8 - Conventions d'équipe & gestion de projet"
weight: 2025
---

## _Cadrer l'usage de l'IA dans une équipe, et l'utiliser pour piloter_

---

# Deux profils, deux usages

L'IA dans une équipe ne sert pas à la même chose selon où vous êtes assis :

- **Dev** : génération de code, refactor, review, tests, debug. L'agent travaille dans le repo.
- **Manager / lead / chef de projet** : synthèse, suivi, rédaction de specs, préparation de réunions, relecture de PRs sans rentrer dans le code. L'agent travaille sur des notes, des tickets, des comptes-rendus.

Les deux profils peuvent (et devraient) utiliser le **même outil** — Codex, Claude Code, etc. — mais avec des **agents spécialisés** différents. C'est le rôle d'`AGENTS.md`.

---

# AGENTS.md : décrire l'équipe à l'agent

`AGENTS.md` (à la racine du repo, ou dans `~/` pour un usage perso) est un fichier que l'agent lit automatiquement à chaque session. Il y trouve le contexte qui ne change pas : qui vous êtes, qui est l'équipe, quels outils vous utilisez, ce qu'il doit éviter.

Exemple côté **manager** :

```markdown
# AGENTS.md

## Mon rôle
Lead technique d'une équipe de 6 (4 devs back, 2 front).
Je code peu — 80% de mon temps c'est review, specs, suivi.

## Mon équipe
- Alice : senior back, owner du module paiement
- Bob : junior back, en montée en compétence sur Postgres
- Carla : front, owner du design system
- ...

## Outils
- Linear pour les tickets (project "CORE")
- GitHub pour le code (org acme/)
- Slack pour la communication (#team-core)

## Comment je travaille
- Je préfère les comptes-rendus en bullet points, pas en prose
- Quand tu rédiges une spec, structure : contexte / objectif / non-objectifs / risques
- Pour les 1:1, sortir 3 questions max, pas un script complet
```

Avec ce fichier, vous n'avez plus à répéter le contexte à chaque prompt. L'agent sait à qui il parle et comment vous aider.

---

# Agents pour la gestion de projet

Quelques usages concrets côté manager — chaque exemple suppose qu'`AGENTS.md` est en place.

## Préparer une réunion

```
@notes/dernier-1-1-bob.md @linear/bob-tickets.json

Prépare 3 sujets pour mon prochain 1:1 avec Bob.
Cherche les tickets bloqués depuis > 3 jours, les PRs en attente
de review de sa part, et tout signal faible dans les notes précédentes.
```

## Synthèse hebdo

```
Lis tous les commits de la semaine sur acme/core (git log --since "7 days ago"),
les PRs mergées, et les tickets Linear passés en Done.
Sors une synthèse de 10 lignes max pour le standup de lundi.
Format : "Ce qui a avancé / Ce qui est bloqué / Décisions à prendre".
```

## Rédiger une spec

```
@notes/brainstorm-feature-x.md

Transforme ces notes en spec courte (1 page).
Sections : contexte, objectif, non-objectifs, risques, questions ouvertes.
N'invente pas — si une info manque, mets "[À CLARIFIER]".
```

Ce dernier point est important : **un agent invente quand on lui demande d'être complet**. Mieux vaut un trou explicite qu'une réponse hallucinée.

---

# Prompt engineering : ce qui marche vraiment

Quelques règles qui changent les résultats, dev comme manager :

## 1. Donnez du contexte avant la tâche

Mauvais :
```
Réécris ce paragraphe.
```

Bon :
```
Audience : devs juniors qui découvrent Git.
Objectif : comprendre git rebase sans peur.
Réécris ce paragraphe en gardant le ton informel.
```

## 2. Précisez le format de sortie

"Réponds en bullet points", "Tableau markdown", "JSON avec ces clés", "Maximum 5 lignes". L'agent suit ces contraintes — mais il faut les écrire.

## 3. Donnez un exemple

Un seul bon exemple vaut trois paragraphes d'instructions. C'est vrai pour le code comme pour la rédaction.

## 4. Demandez de poser des questions

```
Avant de répondre, pose-moi les 2 questions qui changeraient le plus
ta réponse si tu avais leurs réponses.
```

Évite les réponses génériques.

## 5. Utilisez les fichiers comme mémoire partagée

Les agents ne partagent pas de mémoire entre sessions. Mais ils savent lire et écrire des fichiers. Un compte-rendu de réunion en `.md`, une todo en `.md`, une spec en `.md` — c'est la mémoire de l'équipe **et** de l'agent.

---

# Les tensions à anticiper

| Tension | Ce qu'on entend |
|---------|-----------------|
| **Productivité vs Qualité** | "L'IA code plus vite mais le code est moins maintenable" |
| **Apprentissage vs Dépendance** | "Les juniors ne comprennent pas ce qu'ils committent" |
| **Confidentialité vs Cloud** | "On n'a pas le droit d'envoyer ce code à un modèle externe" |

Ces tensions ne se résolvent pas avec un outil — elles se résolvent avec des **règles d'équipe**.

---

# Trois guardrails minimum

Pas une convention parfaite — une convention que tout le monde a votée et appliquera dès demain.

Exemples qui reviennent souvent :

```
- Commit de code IA non compris = refus de merge
- Toute PR IA-générée porte le label "ai-generated"
- Le reviewer doit valider les dépendances ajoutées par l'IA
- Pas de secrets ni code propriétaire envoyé à un modèle externe non auto-hébergé
- Les juniors expliquent le code généré avant de commit
```

## Le label `ai-generated` sur GitHub

```bash
gh label create "ai-generated" --color "B8B8B8" \
  --description "Code généré par IA - review approfondie requise"

gh pr edit <number> --add-label ai-generated
```

L'intérêt n'est pas de stigmatiser le code IA — c'est de **rendre visible** la part d'IA dans le repo, pour adapter la review.

---

# Scénario classique

**Vendredi 16h47.** Un utilisateur signale que les commandes passées depuis 2h sont doublées en base. `git blame` pointe vers un commit "feat: add order processing" mergé ce matin. Le code a été généré par IA — le reviewer a approuvé sans comprendre la logique de déduplication.

Questions à se poser dans l'équipe :

1. Qui est responsable ? L'auteur, le reviewer, ou personne ?
2. Laquelle de vos 3 guardrails aurait évité ça ?
3. Que manquait-il dans le process de review ?

---

# À retenir

- **AGENTS.md** : décrivez votre rôle, votre équipe, vos outils. L'agent devient utile sans répétition.
- **Dev ou manager** : même outil, agents spécialisés différents.
- **Prompt engineering** : contexte, format, exemples, questions ouvertes — pas de magie.
- **Conventions d'équipe** : 3 règles votées valent mieux qu'un document de 20 pages.
- **Fichiers markdown** : la mémoire partagée entre humains et agents.
