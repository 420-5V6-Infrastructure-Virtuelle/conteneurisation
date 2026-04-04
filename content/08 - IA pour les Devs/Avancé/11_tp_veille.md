---
title: "11 - TP Veille et Écosystème"
weight: 2051
---

## _Structurer une veille efficace_

---

# Objectif

Créer un système de veille IA personnel et/ou équipe.

---

# Partie 1 : Audit des sources

##Étape 1 : Évaluer ses sources actuelles

**Questions :**

1. D'où vient votre info IA aujourd'hui ?
2. Combien de temps par jour/semaine ?
3. Quelle est la qualité (signal/bruit) ?

---

##Étape 2 : Sélectionner les sources

**Créer un fichier `veille.md` :**

```markdown
# VEILLE_IA.md

## Sources quotidiennes (15 min max)
- [ ] Hacker News - https://news.ycombinator.com/
- [ ] Twitter list \"AI twitter\"

## Sources hebdomadaires (1h)
- [ ] The AI Epoch newsletter
- [ ] Simon Willison's blog

## Sources mensuelles
- [ ] Papers With Code trending

## Outils à tester ce mois
- [ ] [Nouvel outil de la liste]

## Notes
- [DD/MM] Test de [outil] : [résultat]
```

---

# Partie 2 : Automatiser

## Option 1: Agréger avec RSS

```bash
# Utiliser un lecteur RSS (Feedly, Inoreader)
# Ajouter les flux :
- https://simonwillison.net/atom/everything/
- https://www.anthropic.com/blog/rss.xml
- https://openai.com/blog/rss.xml
```

---

## Option 2: Script de récupération

```bash
#!/bin/bash
# veille.sh

echo "=== Veille IA du $(date) ==="

# Hacker News top stories
curl -s "https://hacker-news.firebaseio.com/v0/topstories.json" | \
  jq '.[0:5]' | \
  jq -r '.[]' | \
  while read id; do
    curl -s "https://hacker-news.firebaseio.com/v0/item/$id.json" | \
      jq -r '"- "+.title+" ("+.url+")"'
  done

echo ""
echo "=== Fin veille ==="
```

---

# Partie 3 : Routage intelligent

## Créer des catégories

```markdown
# CATEGORIES

## À tester immédiatement
- Nouveaux modèles
- Nouveaux outils OpenCode

## À surveiller
- Papers majeurs
- Nouvelles features

## Pour plus tard
- News business
- Spéculation

## Ignorer
- Hype sans fondement
- Marketing empty
```

---

## Filtres Twitter/X

```
# Créer une liste \"AI twitter\" avec :
@karpathy
@simonw
@svpino
@sama
@gdb
@anthropicai
@mistral_ai

# Utiliser la liste pour ne voir que ces comptes
twitter.com/i/lists/XXXXX
```

---

# Partie 4 : Communautés

## Discord/Slack essentiels

| Communauté | Focus |
|------------|-------|
| OpenCode Discord | Agent discussions |
| Anthropic Discord | Claude usage |
| LocalLLaMa Reddit | Open source models |

---

## Template de partage

```markdown
## Format de partage #veille-ia

### Lien
[URL]

### TL;DR
[1-2 phrases max]

### Pourquoi ça compte
[1-2 phrases max]

### Action
À tester / À discuter / Juste pour info
```

---

# Partie 5 : Test d'un nouvel outil

## Process de test

```markdown
# TEST_OUTIL.md

## Outil
[Nom de l'outil]

## Date
[DD/MM/YYYY]

## Installation
[Résumé de l'installation]

## Test 1: Tâche simple
[Tâche] → [Résultat] → [Note /5]

## Test 2: Tâche complexe
[Tâche] → [Résultat] → [Note /5]

## Test 3: Edge case
[Tâche] → [Résultat] → [Note /5]

## Coût
[Coût estimé]

## Verdict
- À adopter ?
- À surveiller ?
- À rejeter ?

## Pourquoi
[Explication]
```

---

# Livrable

À la fin de ce TP :

- [ ] Fichier `VEILLE_IA.md` créé
- [ ] Sources sélectionnées
- [ ] Routine définie (15 min/jour)
- [ ] Premier partage effectué

---

# Checkpoint

**Pattern retenu :** La veille est un investissement, pas une distraction.

**Question clé :** Comment évitez-vous le bruit pour vous concentrer sur le signal ?

---

# Prochain module

Module 12 : Projet final - Intégration complète.