---
title: "12 - TP Projet Final"
weight: 2056
---

## _Intégrer l'IA dans un projet réel_

---

# Objectif

Appliquer toutes les connaissances acquises dans un projet complet en utilisant OpenCode et ses extensions.

---

# Partie 1 : Configuration avancée

> **📌 Git Workflow:** Avant de commencer, créez une branche :
> ```bash
> git checkout main && git pull
> git checkout -b feature/opencode-setup
> ```

## Étape 1 : Choisir son setup OpenCode

**Options de base :**

| Configuration | Usage | Coût |
|---------------|-------|------|
| OpenCode nu | Découverte | Gratuit (modèles free tier) |
| OpenCode + provider | Production | Selon provider |
| Oh My OpenCode | Projects complexes | + fonctionnalités avancées |

---

## Oh My OpenCode : L'extension incontournable

**Source :** [github.com/code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) (48K+ stars)

> Oh My OpenCode transforms OpenCode into a multi-agent coding harness: an orchestrator delegates work to specialist agents that run in parallel.

**Caractéristiques principales :**

1. **Orchestrateur multi-agents**
   - Délègue automatiquement aux agents spécialistes
   - Exécution parallèle pour throughput maximal
   - Agents curratés avec modèles optimaux

2. **Outils avancés**
   - LSP intégrés (go-to-definition, find-references)
   - AST-grep pour refactorings sécurisés
   - MCPs intégrés (Playwright, contexte, etc.)

3. **Compatible Claude Code**
   - Layer de compatibilité
   - Architecture interchangeable

**Installation :**

```bash
# Via OpenCode
opencode install oh-my-opencode

# Ou manuellement
git clone https://github.com/code-yeongyu/oh-my-openagent ~/.opencode/plugins/oh-my-opencode
```

**Configuration type :**

```json
{
  "agents": {
    "build": { "model": "claude-sonnet-4", "tools": ["write", "edit", "bash"] },
    "explore": { "model": "claude-haiku", "tools": ["read", "grep", "glob"] },
    "oracle": { "model": "claude-opus-4", "tools": ["read"] }
  }
}
```

> **📌 Git:** Committez votre configuration :
> ```bash
> git add .opencode/ && git commit -m "feat: add OpenCode configuration"
> ```

---

## Étape 2 : Comprendre le Prompt Caching

**Source :** [claudecodecamp.com/p/how-prompt-caching-actually-works-in-claude-code](https://www.claudecodecamp.com/p/how-prompt-caching-actually-works-in-claude-code)

> "Prompt caching is the architectural constraint around which Claude Code is built. They declare SEVs when cache hit rates drop." — Thariq Shihipar, Claude Code team

### Pourquoi ça compte

**Sans caching :**
- Session 100 turns = $50-100 en input tokens
- ~20M tokens pour une session heavy avec compaction

**Avec caching (90% hit rate) :**
- Même session = $10-19
- **Économie de 80-90%**

### Comment ça fonctionne

```
Request 1 (cold start):
  input_tokens: 2854
  cache_creation_input_tokens: 2727  ← écrit dans le cache
  cache_read_input_tokens: 0

Request 2 (warm):
  input_tokens: 127
  cache_read_input_tokens: 2727      ← lu depuis le cache (90% moins cher)
```

### Règles d'or

| ✅ Faire | ❌ Ne pas faire |
|---------|----------------|
| Contenu statique en premier | Timestamp en haut du system prompt |
| Données dynamiques en messages | Modifier le system prompt |
| Garder les mêmes outils | Ajouter/retirer des MCP mid-session |
| Garder le même modèle | Switcher modèle mid-session |

**Cache TTL :** 5 minutes d'inactivité. Chaque hit reset le timer.

### Breakpoints

```json
{
  "system": [
    {
      "type": "text",
      "text": "You are a senior developer...",
      "cache_control": { "type": "ephemeral" }
    }
  ]
}
```

---

# Partie 2 : Retours d'expérience

> **📌 Note:** Cette partie est théorique. Pas de code à commit, mais prenez des notes.

## Ce que dit Hacker News

**Source :** [news.ycombinator.com/item?id=47460525](https://news.ycombinator.com/item?id=47460525) (1274 points, 621 comments)

### Points de vigilance

1. **Sécurité**
   > "The default model sends prompts to Grok's free tier, which trains on your data. You need to set an explicit 'small_model' in OpenCode to disable that." — HN comment
   
   > "Provider auth shell commands executed without validation — potential RCE vulnerability." — HN comment

2. **Pratiques de développement**
   > "They're constantly releasing at extremely high cadence, where they don't even spend time to test or fix things... they add, remove, refine, change, fix, and break features constantly." — logicprog on HN

3. **Performance**
   > "1GB+ RAM for a TUI. The TypeScript codebase is probably larger than it needs to be." — HN comment
   
   > "Codex (Rust) uses 80MB and 6% CPU vs Claude Code at multiple GBs and 100% CPU for the same task." — jmmv on HN

### Bonnes pratiques suggérées

```markdown
# RECOMMANDATIONS HN

## Avant de commencer
1. Configurer explicitement le modèle principal ET le small_model
2. Utiliser un user dédié sans accès aux fichiers sensibles
3. Vérifier les permissions avant exécution

## Pendant l'usage
1. Tester dans un container sans internet pour projets sensibles
2. Monitorer les appels réseau (mitmproxy)
3. Piner les versions pour éviter les breaking changes
```

---

## Code Review avec IA

**Source :** [codeintelligently.com/blog/ai-code-review-checklist](https://codeintelligently.com/blog/ai-code-review-checklist)

> "I've reviewed over 500 AI-assisted PRs in the past year. Every team I worked with asked for the same thing: a concrete checklist."

### Checklist validée par la communauté

```markdown
## Review Checklist (AI-generated code)

### Compréhension
- [ ] Le code est compréhensible sans explication
- [ ] Je peux expliquer chaque fonction à un collègue
- [ ] Pas de "magic" que personne ne comprend

### Qualité
- [ ] Style cohérent avec le reste du code
- [ ] Pas de code mort
- [ ] Dépendances légitimes (vérifier sur npm/pypi)

### Sécurité
- [ ] Pas de credentials en clair
- [ ] Pas d'APIs hallucinées
- [ ] Inputs validate

### Tests
- [ ] Couverture des cas critiques
- [ ] Tests significatifs (pas juste "expect true")
- [ ] Edge cases couverts
```

### Label IA obligatoire

```bash
# Créer le label
gh label create ai-generated \
  --color B8B8B8 \
  --description "Code généré par IA - review approfondie requise"

# L'appliquer
gh pr edit <number> --add-label ai-generated
```

---

# Partie 3 : Projet à réaliser

> **📌 Git Workflow (OBLIGATOIRE):**
> ```bash
> # Créer la branche projet
> git checkout main && git pull
> git checkout -b feature/projet-final
> ```
> 
> **⚠️ Travaillez TOUJOURS sur une branche. Ne jamais commit sur main.**

## Choix du projet

### 🟢 Projets simples (recommandés pour débuter)

Les LLM excellent sur ces types de projets :

| Projet | Stack suggérée | Pourquoi c'est facile |
|--------|----------------|----------------------|
| **Lecteur RSS** | Python/FastAPI ou Node/Express | CRUD simple, API standard |
| **Lecteur de musique** | Python/Flask + SQLite | Entités simples, peu de logique |
| **Todo app avec auth** | React + Express + SQLite | Patterns bien documentés |
| **Blog statique** | Hugo/Next.js | Génération de fichiers, sans backend |

### 🟡 Projets intermédiaires

| Projet | Stack suggérée | Défis |
|--------|----------------|-------|
| **Chat temps réel** | Elixir/Phoenix LiveView + WebSocket | Concurrency, state management |
| **Dashboard analytics** | React + FastAPI + PostgreSQL | Data viz, requêtes complexes |
| **API gateway** | Go ou Rust | Performance, error handling |

### 🔴 Projets avancés (attention aux guardrails)

> ⚠️ **Les applications "métier" sont AVANCÉES car les agents cassent sans guardrails.**

| Projet | Pourquoi c'est difficile |
|--------|-------------------------|
| **E-commerce complet** | Beaucoup de règles métier, edge cases |
| **Système de facturation** | Calculs précis, erreurs coûteuses |
| **CRM personnalisé** | Logique métier complexe, workflows |

**Si vous choisissez un projet avancé :**
1. Commencez par un POC minimal
2. Utilisez Docker ou user Linux sans permissions (voir `00_workflow_transversal.md`)
3. Divisez en micro-features commitables

### 💡 Options alternatives : Technologies inconnues

**Rust + Bevy (jeu vidéo 2D) :**
```bash
# L'agent découvre Rust et Bevy en même temps que vous
cd mon-jeu-bevy
opencode

> I'm new to Rust and Bevy. Suggest a simple 2D game demo I can build.
> Start with movement and collision detection.
```

Idées de demos Bevy simples :
- Pong (taintedcoders.com/bevy-pong-tutorial)
- Flappy Bird clone
- Simple platformer avec gravité
- 2D shooter basique

**Elixir + Phoenix LiveView (app temps réel) :**
```bash
# L'agent apprend Elixir avec vous
cd mon-app-phoenix
opencode

> I'm learning Elixir/Phoenix. Help me build a real-time chat app
> with LiveView. What's the simplest starting point?
```

Exemples Phoenix LiveView :
- Gurujada (plateforme éducative, 147K lignes de LiveView)
- Flashcards avec IA
- Chat temps réel sans JavaScript

---

## Structure minimale

> **📌 Git:** Committez la structure initiale :
> ```bash
> git add . && git commit -m "chore: init project structure"
> ```

```bash
mkdir mon-projet-ia && cd mon-projet-ia

# Structure
mkdir -p src tests docs
touch AGENTS.md Makefile README.md

# Fichiers IA
touch .opencode/config.json
touch CLAUDE.md  # Optionnel, compatible Claude Code
```

---

## AGENTS.md amélioré

```markdown
# AGENTS.md

## Contexte
Ce projet [description]. L'objectif est [objectif].

## Stack technique
- Langage : [Python/JS/Rust]
- Framework : [FastAPI/Express]
- Tests : [pytest/jest]

## Structure
- `src/` : Code source
- `tests/` : Tests
- `docs/` : Documentation

## Conventions
- Style : ruff/black/prettier
- Commits : conventional commits
- PRs : label `ai-generated` obligatoire

## Workflow IA
1. Utiliser Haiku pour exploration/recherche
2. Utiliser Sonnet pour implémentation standard
3. Réserver Opus pour debugging/architecture
4. Générer tests après chaque feature
5. Valider manuellement avant commit

## Prompt Cache Awareness
- Garder le même modèle pendant la session
- Ne pas ajouter/retirer d'outils mid-session
- Timestamp et données dynamiques → en messages

## Coûts
- Budget : $X/mois
- Monitorer usage avec `opencode stats`
- Préférer modèles frugaux

## Guardrails
- ❌ Jamais commit sans comprendre
- ❌ Jamais de credentials en clair
- ❌ Jamais valider dépendances sans vérifier
- ✅ Toujours label `ai-generated` sur PRs
- ✅ Toujours générer tests
```

---

# Partie 4 : Modes de l'IA

## Mode Anonyme (Blind)

**Objectif : Comparer des modèles sans biais.**

Inspiré de [ComparIA](https://github.com/betagouv/comparia) (gouvernement français, DINUM) :
- 600K+ prompts, 250K+ votes
- Comparaison à l'aveugle entre modèles
- Données ouvertes sur Hugging Face

**Comment utiliser :**

```markdown
YOU: "Je vais te donner deux modèles A et B de façon anonyme.
     Tu dois générer la même réponse avec chacun, puis je voterai."

# Pattern pour choisir entre modèles
YOU: "I need to compare approaches. Give me 3 options:
      1. A simple solution
      2. A robust solution
      3. An experimental solution
      Just list the approaches, don't implement."
```

**Usage recommandé :**
- Évaluer des modèles OpenRouter
- Choisir entre plusieurs architectures
- A/B testing de prompts

---

## Mode "Un seul LLM"

**Objectif : Économiser en utilisant un seul modèle.**

```json
// .opencode/config.json
{
  "model": "openrouter/auto",  // Routage automatique vers le meilleur modèle
  // OU
  "model": "anthropic/claude-3-haiku",  // Le moins cher
}
```

**Pattern : Reflection → Implementation**

```markdown
# ÉTAPE 1 : Reflection (modèle gratuit/cheap)
YOU: "Plan this feature. Don't implement yet.
      Think through edge cases, error handling, file structure.
      Output a detailed plan."

# ÉTAPE 2 : Implementation (modèle frugal)
YOU: "Now implement according to your plan. 
      Use the structure you defined.
      One file at a time, with tests."
```

**Modèles recommandés par phase :**

> **📌 Voir `5_cours_couts.md` pour les détails complets sur l'optimisation des coûts et le pattern Reflection-then-Implement.**

| Phase | Free Tier | Cheap Alternative |
|-------|-----------|-------------------|
| **Reflection/Planning** | Gemini Pro (free) | Haiku ($0.25/M tokens) |
| **Implementation simple** | Gemini Flash (free) | Minimax 2.5 via OpenRouter |
| **Code review** | Claude Code free tier | GLM-4.7 via OpenRouter |
| **Debugging** | Claude Code free tier | Opus (cher mais précis) |

**Offres gratuites à connaître :**
- **Gemini Code**: Plan gratuit généreux (Google AI Studio)
- **Nvidia**: 40 requêtes/minute sur modèles open source
- **Claude Code**: Free tier avec limites

---

## Mode "Yolo" ⚠️

> **⚠️ Voir `00_workflow_transversal.md` pour les détails sur Yolo Mode et Catastrophic Forgetting.**

**Rappel :** Ne pas laisser l'agent travailler sans supervision. Si vous utilisez ce mode, obligatoire : Docker OU user Linux sans permissions.

```bash
# Vérifier TOUJOURS après
git diff --stat
make test
```

---

## Mode "Orchestrateur" (Oh My OpenCode)

**Objectif : Déléguer automatiquement aux spécialistes.**

```json
{
  "agents": {
    "explore": { "model": "haiku", "tools": ["read", "grep", "glob"] },
    "build": { "model": "sonnet", "tools": ["write", "edit", "bash"] },
    "oracle": { "model": "opus", "tools": ["read"] }
  }
}
```

**Pattern recommandé :**

```markdown
YOU: "Add authentication to this app using the orchestrator pattern"

# Oh My OpenCode distribue automatiquement:
# 1. explore agent: lit le code existant
# 2. build agent: implémente l'auth
# 3. oracle agent: review finale
```

---

## Monitoring des coûts

**Suivi des tokens :**

```python
# Script de tracking (à ajouter dans Makefile)
def track_costs():
    """Monitor OpenCode costs from usage logs"""
    # Check cache hit rate
    # Alert if < 80%
    # Budget vs actual
```

**Alertes :**

```yaml
# Cost thresholds
budget_monthly: 50  # USD
alert_threshold: 0.8  # 80% of budget
cache_hit_min: 0.85  # 85% minimum
```

---

# Partie 5 : Git Workflow Détaillé

> **📌 Consulter `00_workflow_transversal.md` pour les règles complètes.**
> Cette section donne les git consignes spécifiques à chaque étape du TP.

## Phase par phase avec Git

**Phase 1 : Setup initial**
```bash
# Créer la branche projet
git checkout main && git pull
git checkout -b feature/projet-final

# Après création de la structure
git add . && git commit -m "chore: init project structure"
```

**Phase 2 : Configuration OpenCode**
```bash
# Après configuration de .opencode/
git add .opencode/ AGENTS.md
git commit -m "chore: add opencode configuration"
```

**Phase 3 : Implémentation feature**
```bash
# APRÈS chaque étape logique de l'agent
git diff                    # Vérifier les changements
git diff --stat             # Vue d'ensemble
git add src/feature/ && git commit -m "feat: add feature types"
# Slop removal avant commit si nécessaire
```

**Phase 4 : Tests**
```bash
# Après génération des tests
git add tests/
git commit -m "test: add tests for feature"
```

**Phase 5 : Review finale**
```bash
# Si corrections après review
git add src/
git commit -m "fix: address review feedback"

# Push final
git push origin feature/projet-final
gh pr create --label ai-generated
```

## Checklist Git par TP

```markdown
- [ ] Branche créée (pas sur main)
- [ ] Commits atomiques (un concept = un commit)
- [ ] git diff vérifié avant chaque commit
- [ ] Pas de slop (console.log, TODO, code mort)
- [ ] Messages descriptifs (feat/fix/test/docs)
- [ ] Push régulier vers remote
```

---

# Partie 6 : Debugging avancé

## Patterns de échecs courants

| Symptôme | Cause probable | Solution |
|----------|----------------|----------|
| Cache hit < 50% | Timestamp/prompts dynamiques | Déplacer en messages |
| Coût élevé | Mauvais modèle | Utiliser Haiku/Flash pour simple |
| Code incompréhensible | Prompt vague | Être plus spécifique |
| Tests qui passent mais bug | Tests incomplets | Ajouter edge cases |
| Boucle infinie | Tool spam | Simplifier le prompt |

---

## Quand demander de l'aide

**À l'oracle (Opus) :**

```markdown
> I've been stuck on this for 3 attempts.
> Here's what I tried:
> 1. [approche 1] → [résultat]
> 2. [approche 2] → [résultat]
> 
> The error is: [erreur complète]
> 
> What should I try next?
```

---

# Partie 7 : Intégration équipe

## PR template

```markdown
## Description
[Description de la feature]

## Changements
- [Liste des changements]

## AI Usage
- [x] Code généré avec IA
- [ ] Code reviewed par humain
- [ ] Compris et validé

## Tests
- [ ] Tests unitaires passent
- [ ] Couverture > 70%

## Checklist Review
- [ ] Compréhensible sans explication
- [ ] Pas de credentials
- [ ] Dépendances vérifiées
- [ ] Style cohérent
```

---

## Metrics à suivre

| Métrique | Cible | Mesure |
|----------|-------|--------|
| Cache hit rate | > 85% | `cache_read_input_tokens / total` |
| PR review time | Comparable | Temps moyen de review |
| Bug rate IA vs humain | Comparable | Bugs en prod |
| Test coverage | > 70% | pytest --cov |

---

# Livrables

## À remettre

1. **Code source** avec AGENTS.md complet
2. **Tests** passants avec coverage > 70%
3. **Documentation** à jour (README + docs/)
4. **Makefile** fonctionnel (test, lint, run)
5. **Rétrospective** :
   - Prompts efficaces documentés
   - Modèles utilisés par tâche
   - Coûts réels vs budget
   - Difficultés rencontrées

---

## Critères de succès par niveau

**Objectifs par niveau :**

| Niveau | Critères | Points |
|--------|----------|--------|
| **Minimum** | Code fonctionne, tests basiques, documentation minimale | 50% |
| **Bon** | + Coverage > 70%, AGENTS.md complet, workflow Git propre | 75% |
| **Excellent** | + Prompt caching optimisé, costs tracking, retours équipe | 100% |

## Checkpoints temporels

| Temps | Checkpoint | Action si retard |
|-------|-----------|------------------|
| **5 min** | Structure créée, branche initialisée | Simplifier le scope |
| **15 min** | AGENTS.md terminé, premier prompt lancé | Vérifier les guardrails |
| **35 min** | Feature principale implémentée | Passer en mode "Yolo" avec Docker |
| **45 min** | Tests écrits, documentation à jour | Reviewer puis merger |

---

# Grille d'évaluation

| Critère | Points | Détails |
|---------|--------|---------|
| Qualité code | 30% | Lisibilité, style, structure |
| Tests | 25% | Coverage, pertinence |
| Documentation | 15% | AGENTS.md, README, comments |
| Conventions IA | 15% | Labels, workflow, guardrails |
| Présentation | 15% | Rétrospective, learnings |

---

# Checklist finale

Avant de soumettre :

- [ ] `make test` passe
- [ ] `make lint` passe
- [ ] AGENTS.md présent et complet
- [ ] PRs labelisées `ai-generated`
- [ ] Code compris et validé
- [ ] Coûts documentés
- [ ] README à jour

---

# Références

## Documentation officielle
- OpenCode : [opencode.ai](https://opencode.ai)
- Oh My OpenCode : [ohmyopencode.com](https://ohmyopencode.com)
- DeepWiki : [deepwiki.com](https://deepwiki.com) - Documentation structurée de repos GitHub

## Articles et discussions
- Prompt Caching : [claudecodecamp.com](https://www.claudecodecamp.com/p/how-prompt-caching-actually-works-in-claude-code)
- HN Discussion : [news.ycombinator.com/item?id=47460525](https://news.ycombinator.com/item?id=47460525)
- AI Code Review : [codeintelligently.com](https://codeintelligently.com/blog/ai-code-review-checklist)
- GitHub : [github.com/code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent)
- ComparIA (gouv français) : [github.com/betagouv/comparia](https://github.com/betagouv/comparia)

## Exemples de projets
- Bevy (Rust game engine) : [bevyengine.org](https://bevyengine.org)
- Phoenix LiveView : [phoenixframework.org](https://phoenixframework.org)
- Gurujada (Edu platform) : exemple Phoenix LiveView 147K lignes

---

# Checkpoint

**Pattern retenu :** L'IA génère, l'humain valide et commit.

**Question clé :** Comment allez-vous mesurer le ROI de l'IA dans votre workflow ?

---

# Conclusion du cours

Vous avez maintenant toutes les clés pour :
- Configurer OpenCode avec les extensions appropriées
- Optimiser les coûts avec le prompt caching
- Intégrer l'IA dans un workflow équipe
- Éviter les pièges documentés par la communauté

**Rappel :** L'IA est un outil. Votre responsabilité de développeur reste entière.

---

# Pour aller plus loin

## Extensions à explorer
- MCP servers personnalisés
- Playwright pour tests E2E
- Intégrations CI/CD

## Veille continue
- Module 11 : Veille technologique
- Communautés : opencode.cafe, awesome-opencode
