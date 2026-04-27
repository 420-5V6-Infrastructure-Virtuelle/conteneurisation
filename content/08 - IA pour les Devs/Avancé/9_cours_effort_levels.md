---
title: "9 - Effort & Autonomie : choisir son niveau"
weight: 2028
---

## _Quand utiliser un effort de réflexion maximal ?_

---

# Le problème

Tout le monde utilise le même réglage pour tout. Sonnet + interactif + vérification à chaque fichier pour corriger un typo. 

L'effort doit être proportionnel à la complexité et à l'impact de la tâche.

---

# Les 4 niveaux

| Niveau | Modèle | Tools | Supervision | Coût indicatif |
|--------|--------|-------|-------------|----------------|
| **Low** | Haiku / Flash | Aucun | N/A | ~$0.01 |
| **Mid** | Sonnet | read, grep, edit | Active | ~$0.10–0.50 |
| **High** | Sonnet + extended thinking | Tous | Intermittente | ~$1–5 |
| **Max** | Opus + `--dangerously-skip-permissions` | Tous | Sandbox + tmux | ~$5–20 |

---

# Low — La question ciblée

```bash
# Pas d'agent, juste une question
gemini "Quel est le bon code HTTP pour 'resource already exists' ?"
# 409 Conflict. $0.001. 3 secondes.
```

**Quand :** snippet rapide, question de syntaxe, explication d'une erreur connue.

**Ce qu'on perd :** contexte repo, cohérence avec le reste du code, connaissance des conventions de l'équipe. L'agent répond dans le vide.

**Signal d'alarme :** vous avez besoin de coller du code dans la question → passez en Mid.

---

# Mid — Le sweet spot

```bash
# Agent dans le repo, supervision active
claude
> Fix the auth middleware bug in src/middleware/auth.ts — the token expiry check is inverted
```

L'agent lit le fichier, comprend le contexte, propose le fix. Vous vérifiez. Vous validez.

**Quand :** bug isolé, feature ciblée (< 5 fichiers impactés), refacto locale.

**Ce qu'on perd :** vision globale sur des changements larges. Si la tâche touche 15 fichiers, la qualité chute.

**Signal d'alarme :** l'agent modifie plus de 10 fichiers → soit vous découpez la tâche, soit vous montez en High.

---

# High — Le problème difficile

Extended thinking = le modèle raisonne avant de répondre. Tokens invisibles, mais comptent dans la facture.

```bash
# Claude Code avec extended thinking (activé via les settings)
# ou en API : thinking_budget = 10000 tokens

> Our order processing is producing duplicate entries intermittently.
> It only happens under concurrent load. Here's the sequence diagram
> and the relevant logs from the last 3 incidents.
```

**Quand :**
- 3ème tentative sur un bug (les deux premières ont échoué)
- Décision d'architecture avec contraintes contradictoires
- Debug de comportement non-déterministe ou distribué

**Ce qu'on perd :** temps et argent si la tâche ne le mérite pas. Un bug simple avec extended thinking = vous payez 5x pour une réponse identique.

**Signal d'alarme :** ne pas activer par défaut. C'est un outil pour les cas durs, pas un réglage permanent.

---

# Recherche en autonomie surveillée / async

```bash
# Dans un worktree isolé, dans Docker, dans tmux
git worktree add ../project-migration feature/db-migration
cd ../project-migration

tmux new -s migration
docker run -it --rm -v $(pwd):/app agent-sandbox bash
claude --dangerously-skip-permissions -p "$(cat MIGRATION_TASK.md)"
# Ctrl+B D — vous détachez et revenez le lendemain
```

**Quand :**
- Grosse migration (ORM, framework, base de données)
- Refacto de masse sur un dossier entier
- Tâche longue qu'on lance la nuit
- Exploration d'une techno inconnue sur un projet jetable

**Ce qu'on perd :** visibilité complète pendant l'exécution. Le slop s'accumule. L'agent peut partir dans des directions non voulues. C'est pour ça que le sandbox est non-négociable à ce niveau.

**Checklist avant de lancer :**
- [ ] Worktree ou branche dédiée
- [ ] Docker avec `--network none` si tâche sensible
- [ ] `AGENTS.md` avec guardrails clairs
- [ ] tmux pour pouvoir reattacher
- [ ] `git diff --stat` au réveil avant de toucher quoi que ce soit

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
└── Refacto de masse, migration →  en sandbox en autonomie

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
| 3 | High (Sonnet ou Opus) | 45k | $0.90 | Fix correct |

**Total : $1.05 pour résoudre quelque chose qui aurait pris 3h à la main.**

Le coût n'est pas le sujet. Le sujet c'est de choisir le bon niveau au bon moment — pas de bruler des tokens High sur du Low, pas de s'obstiner en Mid quand il faut passer en High.

---

# Résumé

**Low** = question dans le vide. Rapide, pas de contexte.

**Mid** = agent dans le repo, vous supervisez. C'est là que vous passez 80% du temps.

**High** = problème dur, raisonnement profond, 3ème tentative. Intentionnel, pas par défaut.

**Async** = autonomie maximale, sandbox obligatoire, résultats au matin.
