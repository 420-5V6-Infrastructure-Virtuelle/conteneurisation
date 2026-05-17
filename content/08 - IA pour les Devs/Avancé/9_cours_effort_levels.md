---
title: "9 - Effort & Autonomie : choisir son niveau"
weight: 2028
---

## _Quand faire le full patate ?_

---

# Le problème

Tout le monde utilise le même réglage pour tout. Sonnet + interactif + vérification à chaque fichier pour corriger un typo. Opus + full-auto + Docker pour écrire un `console.log`. C'est soit trop lent, soit trop risqué.

L'effort doit être proportionnel à la complexité et à l'impact de la tâche.

---

# Les 4 niveaux

| Niveau | Modèle | Tools | Supervision | Coût indicatif |
|--------|--------|-------|-------------|----------------|
| **Low** | Haiku / Flash | Aucun | N/A | ~$0.01 |
| **Mid** | Sonnet | read, grep, edit | Active | ~$0.10–0.50 |
| **High** | Sonnet + extended thinking | Tous | Intermittente | ~$1–5 |
| **Full patate** | Opus + `--dangerously-skip-permissions` | Tous | Sandbox + tmux | ~$5–20 |

---

# Matrice de décision rapide

```
C'est un bug ?
├── Isolé, fichier connu → Mid
├── Intermittent, multi-système → High
└── "Je ne sais même pas d'où ça vient" → High + extended thinking

C'est une feature ?
├── < 5 fichiers → Mid
├── > 10 fichiers, logique complexe → High pour l'architecture, Mid pour l'implémentation
└── Refacto de masse, migration → Full patate en sandbox

C'est une question ?
├── Syntaxe / API standard → Low
└── "Explique-moi comment marche X dans notre codebase" → Mid (l'agent lit le code)
```

---

# Le coût réel de l'extended thinking

Un cas documenté : debugging d'un race condition sur une API Node.js.

| Tentative | Niveau | Tokens | Coût | Résultat |
|-----------|--------|--------|------|----------|
| 1 | Mid (Sonnet) | 12k | $0.06 | Mauvaise piste |
| 2 | Mid (Sonnet) | 18k | $0.09 | Mauvaise piste |
| 3 | High (Sonnet + thinking) | 45k | $0.90 | Fix correct |

**Total : $1.05 pour résoudre quelque chose qui aurait pris 3h à la main.**

Le coût n'est pas le sujet. Le sujet c'est de choisir le bon niveau au bon moment — pas de bruler des tokens High sur du Low, pas de s'obstiner en Mid quand il faut passer en High.

---

# Résumé

**Low** = question dans le vide. Rapide, pas de contexte.

**Mid** = agent dans le repo, vous supervisez. C'est là que vous passez 80% du temps.

**High** = problème dur, raisonnement profond, 3ème tentative. Intentionnel, pas par défaut.

**Full patate** = autonomie maximale, sandbox obligatoire, résultats au matin.
