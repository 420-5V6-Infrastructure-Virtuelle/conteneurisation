---
title: "11 - TP Agent sur Unknown Tech"
weight: 2051
---

## _Explorer une technologie inconnue avec un agent LLM_

---

# Objectif

Utiliser un agent IA pour explorer une technologie que vous ne connaissez **pas** : ici **Rust** et le framework de jeux vidéo **Bevy**.

---

# Le concept "Unknown Tech"

## Pourquoi explorer l'inconnu ?

Un des grands pouvoirs des agents LLM : vous guider dans une technologie que vous ne maîtrisez pas.

**Scénario :** Vous êtes développeur Python/JS, vous n'avez jamais fait de Rust. Mais vous voulez créer un petit jeu.

**L'agent peut :**
- Expliquer la syntaxe Rust
- Guider l'installation
- Suggérer des patterns de code
- Proposer des idées de démos simples
- Débugger les erreurs du compilateur Rust

---

# Partie 1 : Setup Rust + Bevy

## Étape 1 : Questionner l'agent

**Prompt initial :**

```
I want to build a simple game demo in Rust using the Bevy game engine.
I have never used Rust or Bevy before.

1. What do I need to install?
2. What's a simple first demo I could build in 2-3 hours?
3. What are common gotchas for beginners?

Give me 3 demo ideas ranked by difficulty.
```

**L'agent va suggérer :**
- Des idées de démos (Pong, snake, particle system, etc.)
- Le setup nécessaire (rustup, cargo, bevy dependencies)
- Les pièges classiques (borrow checker, ECS patterns)

---

## Étape 2 : Installation guidée

**Demander à l'agent de générer les commandes :**

```
Give me the exact commands to:
1. Install Rust on my system (Linux/macOS)
2. Create a new Bevy project
3. Run a minimal window
```

**Notez chaque commande dans votre historique.**  
**Gardez le Git workflow actif - voir section transversale.**

---

# Partie 2 : L'agent propose, vous codez

## Sélectionner une démo

**Exemple de réponse agent :**

```markdown
## Demo Ideas (ranked by difficulty)

1. **Hello Window** (30 min)
   - Just opens a window
   - Teaches: Cargo, dependencies, Bevy App structure
   
2. **Moving Sprite** (2h)
   - A sprite that moves with keyboard
   - Teaches: Systems, Query, Components, Input handling
   
3. **Simple Pong** (3-4h)
   - Two paddles, a ball, collision
   - Teaches: ECS architecture, collision detection, game loops
```

**Choisir le niveau adapté à votre temps disponible.**

---

## Implémenter avec l'agent

**Pattern de travail :**

```markdown
YOU: "I chose the moving sprite demo. Let's start."

AGENT: Explains the structure, provides base code

YOU: Copy-paste into your editor

ERROR: Compiler error appears

YOU: Paste error to agent

AGENT: Explains the error, suggests fix

YOU: Apply fix, test again

[Loop until working]
```

**Important :** Vous ne copiez pas aveuglément. Vous **comprenez** chaque ligne.

---

# Partie 3 : Documenter le processus

## Créer un fichier de suivi

```markdown
# RUST_BEVY_LEARNING.md

## Date: DD/MM/YYYY

## Why Rust + Bevy?
[Ce qui vous a motivé]

## Demo Chosen
[Hello Window / Moving Sprite / Pong]

## Installation Log

```bash
# Commandes exécutées
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo new my_bevy_game
cd my_bevy_game
# ... toutes les commandes ...
```

## Errors Encountered

### Error 1: [Titre]
```
[Error message exact]
```
**Cause:** [Explication agent]
**Fix:** [Solution appliquée]

### Error 2: ...
```

**Ce fichier devient votre mémoire de session.**  
**Utile pour vous ET pour l'équipe.**

---

# Partie 4 : Le check agent

## Critiquer le résultat

**Questions à poser à la fin :**

```
1. What are the limitations of the code we wrote?
2. What would you improve if we had more time?
3. What are the next learning steps for Rust/Bevy?
4. What bad habits did I show that I should fix?
```

**L'agent peut vous surprendre :**
- Code qui fonctionne mais pas idiomatique
- Patterns simplistes qui passent à l'échelle
- Dette technique invisible

---

# Git & Docker Workflow (Transversal)

> **⚠️ Voir `00_workflow_transversal.md` pour le workflow complet.**

**Rappels rapides pour ce TP :**

```bash
# Git : Une branche par feature
git checkout -b feature/bevy-demo
git add src/main.rs && git commit -m "feat: add basic Bevy window"

# Docker : Isolation optionnelle pour Rust/Bevy
docker run -it -v $(pwd):/app rust:latest bash
cargo new my_bevy_game
```

**Pourquoi c'est important :** En unknown tech, Git est votre safety net. Si l'agent suggère du code qui casse tout, vous pouvez `git diff` ou `git revert`.

---

# Yolo Mode ⚠️

> **⚠️ Voir `00_workflow_transversal.md` pour les détails complets sur Yolo Mode et Catastrophic Forgetting.**

**Rappel rapide :** Le "yolo mode" (laisser l'agent faire tout seul) est risqué. En unknown tech, c'est encore plus dangereux car vous ne savez pas si le code généré est correct.

**Pattern recommandé :**
```markdown
BAD: "Just create the game for me."

GOOD: "Explain the code structure first."
      [Ask clarifying questions]
      [Request one function at a time]
```

---

# Livrable

À la fin de ce TP :

- [ ] Rust et Bevy installés (ou dans Docker)
- [ ] Une démo fonctionnelle (au choix)
- [ ] Fichier `RUST_BEVY_LEARNING.md` documenté
- [ ] Commits atomiques avec messages clairs
- [ ] Compréhension du code produit (vous pouvez l'expliquer)

---

# Checkpoint

**Pattern retenu :** L'agent est un guide, pas un développeur remplaçant.

**Question clé :** Si l'agent a écrit du code que vous ne comprenez pas, comment allez-vous le maintenir ?

**Prochain module :** Module 12 - Projet final avec intégration complète du Git workflow.

---

# Ressources

- [Bevy Official Docs](https://bevyengine.org/learn/)
- [Rust Book](https://doc.rust-lang.org/book/)
- [Bevy Examples](https://github.com/bevyengine/bevy/tree/main/examples)
- [r/rust_gamedev](https://www.reddit.com/r/rust_gamedev/) - Pour les idées de démos

---

# Annexe : Liens de veille à connaître

Ces liens sont à intégrer dans votre routine de veille (voir `11_cours_veille.md`).

## Outils de monitoring et d'inspection

- **[claude-devtools](https://github.com/matt1398/claude-devtools)** — Les DevTools manquants pour Claude Code : inspecter les sessions, tool calls, usage de tokens, sous-agents et fenêtre de contexte en UI visuelle.
- **[codeburn](https://github.com/AgentSeal/codeburn)** — Visualise où vont vos tokens session par session (par type de tool call, fichiers lus, etc.). Utile pour identifier ce qui consomme inutilement.
- **[rtk](https://github.com/rtk-ai/rtk)** — Proxy CLI qui réduit la consommation de tokens de 60-90% sur les commandes dev courantes.

## Lectures importantes

- **[HN #47004712](https://news.ycombinator.com/item?id=47004712)** — Discussion HN à lire : retours d'expérience terrain sur l'usage des agents IA.
- **[Reddit ExperiencedDevs — "An AI CEO finally said something honest"](https://www.reddit.com/r/ExperiencedDevs/comments/1r6olcv/an_ai_ceo_finally_said_something_honest/)** — Analyse critique sur le discours des entreprises IA.
- **[Agentic Coding Trends Report 2026](https://resources.anthropic.com/hubfs/2026%20Agentic%20Coding%20Trends%20Report.pdf)** — Rapport Anthropic sur les tendances du coding agentique.
- **[AI Fluency Index](https://www.anthropic.com/research/AI-fluency-index)** — Recherche Anthropic sur l'usage réel de l'IA et le biais des artefacts.

## Répertoires de ressources

- **[Anthropic Skills](https://github.com/anthropics/skills/tree/main/skills)** — Skills officiels Claude Code : simplify, review, security-review, etc.
- **[Claude Code Security Review](https://github.com/anthropics/claude-code-security-review)** — Skill de review sécurité pour Claude Code.
- **[Claude Code Ultimate Guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/guide/cheatsheet.md)** — Cheatsheet community avec bonnes pratiques et quiz.