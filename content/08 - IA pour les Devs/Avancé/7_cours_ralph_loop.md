---
title: "7 - Ralph Loop et Workflows Autonomes"
weight: 2010
---

## _Laisser l'IA mouliner en autonomie_

---

# Le problème du contexte

**Le LLM a une fenêtre de contexte limitée.**

```
┌─────────────────────────────────────────┐
│         Context Window ~200k tokens      │
│  ████████████████████████████░░░░░░░░░  │
│  Used                        Available   │
└─────────────────────────────────────────┘
```

**Quand le contexte se remplit :**
- L'agent "oublie" les premières instructions
- Les décisions précédentes sont perdues
- Le travail devient incohérent

---

# Le concept Ralph Loop

**Origine :** Geoffrey Huntley, nommé d'après Ralph Wiggum (Les Simpson).

**L'idée :** Un simple `while true` qui relance l'agent jusqu'à ce que le travail soit fini.

```bash
# Pattern de base
while :; do
  cat PROMPT.md | opencode
done
```

**Principe clé :** La progression vit dans les fichiers et git, PAS dans le contexte du LLM.

---

# Comment ça marche

## Le cycle Ralph Loop

```
┌──────────────────────────────────────────────────────────┐
│                      RALPH LOOP                          │
│                                                          │
│   ┌─────────┐    ┌──────────┐    ┌────────┐    ┌───────┐  │
│   │ Execute │───►│ Evaluate │───►│  Fix   │───►│Repeat │  │
│   └─────────┘    └──────────┘    └────────┘    └───────┘  │
│        │              │              │              │     │
│        │              │              │              │     │
│        ▼              ▼              ▼              ▼     │
│   Tâche原子      Tests pass/fail   Corrections    Loop   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Les 4 phases (Ralphable methodology)

1. **Execute** - Agent fait la tâche
2. **Evaluate** - Tests automatiques pass/fail
3. **Fix** - Si échec, corriger
4. **Repeat** - Jusqu'à 100% de succès

---

# Gestion du contexte

## Indicateurs de santé

| État | Tokens | Action |
|------|--------|--------|
| 🟢 Healthy | < 60% | Agent travaille librement |
| 🟡 Warning | 60-80% | Wrap up, préparer rotation |
| 🔴 Critical | > 80% | Rotation forcée |

## Rotation de contexte

**Quand le contexte est plein :**
1. Commit du travail en cours
2. Démarrer un NOUVEAU agent avec un contexte frais
3. Le nouvel agent reprend à partir du commit

```bash
# Le nouveau agent lit le git log
git log -1 --format="%H %s"
# "Add feature X, tests passing"
# Il comprend où on en est
```

---

# Guardrails - Apprendre des erreurs

## Le système de garde-fous

**Pattern :** Quand ça échoue, écrire un "Sign" dans `.ralph/guardrails.md`

```markdown
# Guardrails
## Sign: Do Not Modify .env Files
**Learned from iteration #47**
- Attempted to modify .env, causing CI failure
- Solution: Use .env.example only
- Never touch actual .env

## Sign: Always Run Tests Before Commit
**Learned from iteration #12**
- Committed without tests, broke main
- Solution: Run `make test` before any commit
```

**Le nouvel agent lit les guardrails AVANT de commencer.**

---

# Succès réels

**Fruit Ninja en 1 heure :** Cursor + Ralph a construit un jeu complet avec :
- 8 rotations de contexte
- Détection de collision
- Son
- Learning des échecs Canvas

**Claude Code overnight :** Se réveiller avec de nouvelles features codées et testées.

**Self-healing :** Le loop identifie un problème, étudie le code, corrige, déploie, vérifie - tout seul.

---

# Les dangers (REAL STORIES)

## Incidents réels documentés

| Incident | Coût/Impact |
|----------|-------------|
| Deux agents LangChain qui chatterent en boucle | **$47,000** sur 11 jours |
| OpenClaw ignore la commande STOP | 9.6M emails supprimés |
| Copilot crée **1,526 worktrees git** | 800GB sur le disque |
| Terraform + Claude Code | 2.5 ans de données perdues |
| Replit agent fabrique 4,000 faux records | Corruptions + mensonges |
| **43,175 redémarrages** en une nuit | Port conflict |

---

## Patterns d'échec

**1. Silent Failure** - L'agent dit "tout va bien" mais ne fait rien

```
Agent: "I've successfully fixed the bug!"
Reality: No files changed, no tests run
```

**2. Loop non détecté**

```python
# OpenClaw détecte les reads en boucle
if repeated_read_calls():
    break
# Mais PAS les exec en boucle !
```

**3. Context compaction détruit les contraintes**

```
Instruction originale: "NE PAS modifier la DB"
Après compression: "modifier DB"  # Contrainte perdue
```

---

# Bonnes pratiques

## Monitoring

| Pratique | Description |
|----------|-------------|
| **Monitorer les outcomes, pas l'activité** | Vérifier que l'agent a PRODUIT quelque chose |
| **Définir des indicateurs négatifs** | Alerte quand expected ne se produit PAS |
| **Watchdog externe** | Ne laissez PAS l'agent se monitorer lui-même |
| **Limites de spending** | Hard limits sur les comptes API |

## Pratiques de sécurité

```bash
# TOUJOURS
- Limite d'itérations (default: 20)
- Pas de permissions Terraform sans review
- Commit fréquents = mémoire
- Tests automatisés = vérité
- Watchdog externe = sécurité

# JAMAIS
- Laisser tourner sans limite de spending
- Permissions destructives sans oversight
- Compter sur l'agent pour se monitorer
```

---

# Quand utiliser Ralph Loop

**BON pour :**
- Tests à faire passer
- Refactoring automatisé
- Implémentation d'API
- Migrations de DB

**MAUVAIS pour :**
- "Rends ça plus joli" (subjectif)
- Tâches nécessitant compréhension profonde du code
- Décisions d'architecture sans critères clairs

---

# TP : Ralph Loop en pratique

Voir `7_tp_ralph.md` →