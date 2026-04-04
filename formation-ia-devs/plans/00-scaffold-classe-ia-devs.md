# Formation "IA pour les Développeurs" - Structure Scaffold

**Duration:** 2 jours  
**Format:** 70% TP fil rouge, 30% théorie  
**Outil principal:** OpenCode (agnostique, open source, tool calling transparent)

---

## Prérequis & Questionnaire Pré-Formation

### À envoyer aux participants 1 semaine avant

**Questionnaire d'évaluation:**
1. Quel langage de programmation préférez-vous ? (Python/JS/TS/Rust/Go/Java/Autre)
2. Quels outils IA avez-vous déjà utilisés ? (Copilot/Cursor/Claude Code/Aucun/Autre)
3. Avez-vous un accès API existant ? (OpenAI/Anthropic/OpenRouter/Aucun)
4. Quel type de projet vous intéresse pour les exercices ? (Web/API/Jeu vidéo/Mobile)
5. Qu'est-ce qui vous inquiète le plus dans l'IA pour devs ?

### Apps démo candidates (choix selon profil):

| App | Stack | Complexité | Usage idéal |
|-----|-------|------------|-------------|
| `betagouv/comparia` | FastAPI + Svelte | ~2000 lignes | Full stack réaliste |
| `luchog01/minimalistic-fastapi-template` | FastAPI + Docker | ~500 lignes | Focus patterns |
| Jeu vidéo Bevy (Rust) | Rust + Bevy | Variable | Public Rust/game |
| Quiz infographie Marne | Svelte + API | Simple | Frontend focus |
| Bridge LQDN | Python + API | Moyen | Backend focus |

---

# JOUR 1 - Fondamentaux & Productivité

## Module 1: Typologie des Outils IA (55 min)

### Théorie (20 min)

#### Les 3 catégories d'outils

**1. Agents TUI (Terminal User Interface)**
- *Caractéristiques:* Tool calling transparent, autonomie élevée, cycle complet
- *Outils:*
  - **OpenCode** ← Notre focus (open source, agnostique, MCP natif)
  - Claude Code (Anthropic, performant mais fermé)
  - Gemini CLI (Google, gratuit avec limites)
  - Codex CLI (OpenAI, orienté GitHub)
- *Usage:* Refactoring complexe, génération de features, debugging approfondi

**2. Assistants IDE**
- *Caractéristiques:* Intégration éditeur, suggestions temps réel, moins d'autonomie
- *Outils:* Cursor, GitHub Copilot, Cline, Roo Code
- *Usage:* Complétion, quick fixes, navigation code

**3. Bots CI/CD (PR Reviewers)**
- *Caractéristiques:* Automatisés, review de PR, pas de dev direct
- *Outils:* Google Jules, PR Agent, Devin
- *Usage:* Review automatique, détection de bugs

#### Providers LLM & Modèles à la mode (via OpenRouter)

| Provider | Points forts | Modèles notables | Coût relatif |
|----------|--------------|------------------|--------------|
| OpenAI | Standard, fiable | GPT-4o, o1, o3-mini | $$$$ |
| Anthropic | Reasoning, code | Claude 4 Sonnet, Opus | $$$$ |
| Google | Multimodal, gratuit | Gemini 2.0 Flash, 2.5 Pro | $-$$ |
| DeepSeek | Frugal, performant | DeepSeek V3, R1 | $ |
| OpenRouter | Agrégateur | Tous les modèles | Variable |

**Point clé:** Presque toujours possibilité de mettre sa propre clé API.

### TP Fil Rouge - Setup (35 min)

**Exercice 1.1: Installation OpenCode (10 min)**
```bash
# Via npm
npm install -g opencode

# Configuration initiale
opencode config

# Ajouter clé API OpenRouter
export OPENAI_API_KEY=sk-or-...
```

**Exercice 1.2: Premier prompt - Découverte de l'app (15 min)**
- Lancer OpenCode sur l'app démo choisie (ex: comparia)
- Prompt: "Analyse ce projet et résume l'architecture"
- **Observer le tool calling transparent** (grep, read, glob)
- Noter la différence avec un assistant IDE classique

**Exercice 1.3: Comparaison rapide de modèles (10 min)**
- Tester avec 2 modèles différents via OpenRouter
- Suggestion: DeepSeek V3 (frugal) vs Claude Sonnet (premium)
- Noter: vitesse, qualité de l'analyse, différence de "style"

**Grille d'évaluation qualité:**
| Critère | DeepSeek V3 | Claude Sonnet | Notes |
|---------|-------------|---------------|-------|
| Vitesse de réponse | /5 | /5 |Temps jusqu'à première token |
| Pertinence architecture | /5 | /5 | A-t-il identifié les composants clés? |
| Précision technique | /5 | /5 | Stack, frameworks, dépendances corrects? |
| Suggestions pratiques | /5 | /5 | Problèmes identifiés, améliorations proposées? |
| Tokens consommés | | | Input + Output tokens |
| Coût estimé | | | $ calculé via pricing OpenRouter |

**Score qualité globale:** (Pertinence + Précision + Suggestions) / 3

---

## Module 2: Prompt Engineering pour Devs (55 min)

### Théorie (15 min)

#### Principes de base
- **Contexte** > Instructions vagues
- **Exemples** > Théorie seule (few-shot prompting)
- **Décomposition** > Une tâche complexe = plusieurs tâches simples
- **Feedback** > Itérer sur les résultats

#### Patterns efficaces

**Pattern 1: Contexte structuré**
```
[ROLE] Tu es un expert FastAPI
[CONTEXT] Projet de gestion de tâches avec auth JWT
[TASK] Ajouter endpoint POST /tasks/{id}/assign
[CONSTRAINTS] 
- Valider user_id existe
- Notifier par email
- Retourner 404 si task absente
```

**Pattern 2: Chain of Thought**
```
Avant de coder:
1. Analyse les fichiers pertinents
2. Identifie les dépendances
3. Propose un plan détaillé
4. Code étape par étape
```

### TP Fil Rouge (40 min)

**Exercice 2.1: Refactoring avec contexte riche vs pauvre (20 min)**
- **Tâche:** Ajouter un système de tags aux éléments de l'app
- **Essai 1:** Prompt naïf "Ajoute des tags" → observer le résultat
- **Essai 2:** Prompt structuré avec contexte → comparer
- **Analyse:** Qu'est-ce qui a manqué dans le prompt naïf ?

**Grille de comparaison:**
| Critère | Prompt Naïf | Prompt Structuré | Différence |
|---------|-------------|------------------|------------|
| Compréhension de la tâche | /5 | /5 | |
| Identification des impacts | /5 | /5 | Tables, API, tests modifiés? |
| Respect des conventions | /5 | /5 | Nommage, structure? |
| Temps jusqu'à solution viable | min | min | |
| Tokens totaux | | | |
| Itérations nécessaires | | | Corrections demandées |

**Analyse qualitative:**
- Le prompt naïf a-t-il créé des bugs? Lesquels?
- Le prompt structuré a-t-il tout prévu du premier coup?
- Quel contexte manquait encore?

**Exercice 2.2: Décomposition de tâche complexe (20 min)**
```
Tâche: "Implémenter un système de notifications email"
```
- Décomposer en sous-tâches avec l'agent
- Prompts séparés pour chaque sous-tâche
- Orchestration manuelle des étapes

---

## Module 3: Tool Calling et MCP (55 min)

### Théorie (20 min)

#### Qu'est-ce que le Tool Calling ?
- Le LLM "appelle" des outils de manière autonome
- Cycle: LLM décide → Appelle outil → Reçoit résultat → Continue
- **Transparence:** Voir ce que l'IA fait en temps réel (vs. "boîte noire")

#### MCP (Model Context Protocol)

**Concept:** Standard pour connecter des outils aux LLMs

**Serveurs MCP populaires:**
| Serveur | Usage |
|---------|-------|
| filesystem | Lire/écrire fichiers |
| github | Opérations Git/GitHub |
| postgres | Requêtes BDD |
| brave-search | Recherche web |
| puppeteer/playwright | Automation navigateur |

**Écosystème:**
- OpenClaw et alternatives (registry de serveurs MCP)
- Context7 (documentation à jour)
- Serveurs custom

#### Pourquoi c'est important ?
- **Auditabilité:** Voir chaque action
- **Contrôle:** Valider/refuser les opérations
- **Extensibilité:** Ajouter vos propres outils

### TP Fil Rouge (35 min)

**Exercice 3.1: Observer le tool calling (15 min)**
```
Prompt: "Trouve tous les endpoints API et leur méthode HTTP"
```
- Observer les appels: `grep`, `read`, `lsp_symbols`
- Comprendre le cycle de décision
- Noter: l'agent "pense" avant d'agir

**Grille d'évaluation:**
| Observation | Oui/Non | Exemple observé |
|-------------|---------|-----------------|
| Lit les fichiers avant de répondre | | |
| Utilise plusieurs tools en séquence | | |
| Vérifie ses résultats | | |
| Explique son raisonnement | | |

**Exercice 3.2: MCP en action (10 min)**
- Demander à l'agent de modifier un fichier
- Voir la structure de l'appel tool
- Analyser le pattern: read → edit → verify

**Exercice 3.3: L'erreur classique (10 min)**

**Prompt délibérément vague:**
```
"Ajoute un système de notifications pour les nouveaux utilisateurs"
```

**Ce qui va arriver:**
- L'agent va immédiatement coder sans lire le projet
- Va créer `send_email()`, `send_sms()`, `send_push()`
- Sans vérifier: existe-t-il déjà un système de notifications ? Un service email configuré ?

**Observer le pattern:**
1. Conjecture: "Je vais créer un service de notifications"
2. Spéculation: "Probablement qu'il faut utiliser SendGrid"
3. Code généré sans lecture préalable

**Correction guidée:**
1. Arrêter l'agent
2. Forcer: "Quels services de notification existent déjà dans le projet ?"
3. Observer l'agent lire: `find . -name "*notification*"`, `grep -r "email"`, `read config.py`
4. Redémarrer avec contexte réel

**Leçon:**
- Toujours lire avant de coder
- Un prompt vague ≠ liberté totale
- Demander explicitement la vérification

---

## Module 4: Bonnes Pratiques de Projet (55 min)

### Théorie (25 min)

#### Fichier AGENTS.md

**Quoi:** Document qui guide l'IA sur votre projet

**Pourquoi:**
- Contexte persistant
- Conventions explicites
- Gain de tokens (évite les répétitions)

**Pourquoi c'est crucial:**
- "Context forgetting" - l'agent oublie les instructions distantes
- Un AGENTS.md = ancrage permanent

**Structure recommandée (versions):**

**Minimal (10-20 lignes):**
```markdown
# AGENTS.md

## Stack
FastAPI + SQLAlchemy + PostgreSQL

## Commands
- `make dev` - Run dev server
- `make test` - Run tests  
- `make lint` - Lint check

## Conventions
- Conventional commits
- Tests for new features
```

**Complet (50+ lignes):** Voir annexe

#### Makefile / run.sh

**Pattern classique:**
```makefile
.PHONY: dev test lint build clean

dev:
	docker-compose up -d && docker-compose logs -f

test:
	pytest tests/ -v --cov=src

lint:
	ruff check src/
	mypy src/
```

#### Docker + docker-compose

**Pourquoi l'évoquer:**
- L'agent doit connaître les commandes
- Pattern dev vs. prod
- Volumes, networks, dépendances

### TP Fil Rouge (30 min)

**Exercice 4.1: Créer AGENTS.md pour le projet (15 min)**

**Checklist de découverte des conventions:**
- [ ] Stack: frameworks, langages, versions
- [ ] Structure: organisation des dossiers (src/app/lib?)
- [ ] Commands: dev, test, lint, build, clean
- [ ] Commits: conventional commits? Scope? Breaking changes?
- [ ] Tests: framework, coverage minimum, fixtures location
- [ ] Imports: relatifs ou absolus? Ordre des imports?
- [ ] Naming: camelCase/snake_case, fichiers singular/plural
- [ ] Error handling: exceptions custom? logging pattern?
- [ ] Database: migrations? ORM? raw SQL?
- [ ] API: REST? GraphQL? Versioning?
- [ ] Config: .env? config files? secrets management?

**Questions à se poser:**
1. Regarder 3-5 commits récents → pattern de messages?
2. Regarder 3-5 fichiers de code → style commun?
3. Regarder les tests existants → structure, naming?
4. Lister les dépendances dans requirements.txt/package.json
5. Identifier les patterns récurrents (auth, validation, error handling)

**Exercice 4.2: Vérifier l'impact (15 min)**
- Lancer une tâche complexe **sans** AGENTS.md
- Même tâche **avec** AGENTS.md
- Comparer: qualité, tokens, temps

---

## Module 5: Coûts et Modèles Frugaux (55 min)

### Théorie (20 min)

#### Coûts des modèles

| Modèle | Input ($/1M) | Output ($/1M) | Profil |
|--------|--------------|---------------|--------|
| Claude 4 Opus | $15 | $75 | Premium max |
| Claude 4 Sonnet | $3 | $15 | Excellent rapport |
| GPT-4o | $2.50 | $10 | Standard |
| Gemini 2.0 Flash | $0.10 | $0.40 | Frugal |
| DeepSeek V3 | $0.27 | $1.10 | Frugal performant |
| GLM-4.7 | Variable | Variable | Alternative chinoise |

#### Stratégies d'économie

**1. L'IA pingre:** Gemini Flash pour exploration/simple
**2. L'IA frugale:** DeepSeek V3 ou GLM-4.7 pour dev quotidien
**3. L'IA multimodale:** Qwen pour vision/audio

#### Patterns d'optimisation

- **Context compressing:** Résumer le contexte inutile
- **Alertes de coûts:** Configurer limites sur OpenRouter
- **Arrêter les boucles:** Erreurs répétées = arrêter et changer d'approche

#### Alertes réelles
- Claude Code: $500-2000/mois en usage intensif
- App complexe: 800K tokens en une session

### TP Fil Rouge (35 min)

**Exercice 5.1: Comparaison coût/perf (20 min)**
- Même tâche avec 3 modèles: Gemini Flash / DeepSeek V3 / Claude Sonnet
- Mesurer: temps, qualité estimée, coût calculé
- **Révélation:** Souvent, le modèle "cheap" suffit

**Exercice 5.2: Arrêter une boucle de gaspillage (15 min)**
- Lancer un prompt ambigu qui génère des itérations
- Reconnaître le pattern de gaspillage
- Intervenir pour arrêter et reformuler

---

## Module 6: Multimodal et Cas Avancés (55 min)

### Théorie (15 min)

#### Modèles multimodaux
- **Vision:** Claude, GPT-4o, Gemini, Qwen
- **Audio:** GPT-4o-audio, Gemini
- **Usage:** Analyser screenshots, diagrammes, UI

#### Le quirk Playwright
**Point contre-intuitif:**
- Un screenshot Playwright n'a pas besoin d'être une image
- `browser_snapshot` = accessibility tree (texte)
- Beaucoup plus efficace et précis

#### Patterns spécifiques
- Screenshot → HTML/CSS
- Diagramme → Code
- UI → Tests E2E

### TP Fil Rouge (40 min)

**Exercice 6.1: Screenshot vers code (20 min)**
- Capturer une UI existante de l'app
- Demander à l'IA de reproduire
- Utiliser le bon type de "screenshot"

**Exercice 6.2: UI Testing multimodal (20 min)**
- Tester une interface web de l'app
- Générer des tests E2E basés sur l'analyse visuelle

---

# JOUR 2 - Workflows Autonomes & Bonnes Pratiques

## Module 7: Ralph Loop et Workflows Autonomes (55 min)

### Théorie (25 min)

#### Le pattern Ralph Loop

**Concept:** Cycle autonome avec persistance d'état externe

```
Init → Task Selection → Execute → Verify → Persist → Loop
         ↑                                         ↓
         └─────────────────────────────────────────┘
```

**Composants:**
- `prd.json` - Spécifications
- `progress.txt` - État d'avancement
- `git history` - Historique des tentatives

**Quand l'utiliser:**
- Tâches longues (features complètes)
- Parallélisation d'agents
- "Laisser mouliner" pendant qu'on fait autre chose

**Ressources:**
- GitHub: `snarktank/ralph`
- Blog: `ghuntley.com/loop/`

#### Human-in-the-Loop (HITL)

**Approche moderne:** "On-the-loop"
- L'agent avance en autonomie
- Points de contrôle humains
- Seuils de confiance

### TP Fil Rouge (30 min)

**Exercice 7.1: Implémenter Ralph basique (15 min)**
- Créer la structure de fichiers `.project/`
- Lancer un cycle simple sur une feature mineure
- Observer la persistance d'état

**Exercice 7.2: Parallélisation (15 min)**
- Lancer 2 agents en parallèle sur 2 features
- Utiliser le pattern de branches Git séparées
- Comparer avec le travail séquentiel

**Workflow branches Git parallèles:**
```bash
# Agent 1 - dans un terminal/tmux pane
git checkout -b feature/auth-improvements
# Lancer agent avec prompt spécifique

# Agent 2 - dans un autre terminal/tmux pane
git checkout -b feature/email-notifications
# Lancer agent avec prompt différent

# Après complétion des deux:
git checkout main
git merge feature/auth-improvements
git merge feature/email-notifications
```

**Gestion des conflits:**
- Si les deux branches touchent le même fichier → conflit inévitable
- Solution: partitionner les fichiers par agent
  - Agent 1: backend API, models, services
  - Agent 2: frontend components, routes
- Alternative: agents séquentiels si overlap important

**Critères de succès:**
- Les deux features fonctionnent indépendamment
- Les deux peuvent être mergées sans conflit majeur
- Gain de temps vs séquentiel ≥ 30%

---

## Module 8: Debugging du Code Généré par IA (55 min)

### Théorie (25 min)

#### Les patterns d'échec typiques

**1. Happy Path Bias**
- L'IA code le cas nominal, oublie les edge cases
- *Fix:* Demander explicitement la gestion d'erreurs

**2. Context Missing**
- L'IA ne voit pas les dépendances, code incompatible
- *Fix:* Fournir le contexte requis (AGENTS.md)

**3. Fix Loop Infini**
- Erreur → Fix → Nouvelle erreur → Fix → ...
- *Fix:* Arrêter, analyser root cause, changer d'approche

**4. Spéculation sans vérification**
- L'IA devine au lieu de vérifier
- *Fix:* Forcer lecture avant écriture

**5. Hallucination de dépendances**
- Importe des libs qui n'existent pas
- *Fix:* Vérifier pip/npm avant

**6. Suppression de tests**
- Pour "faire passer" le build
- *Fix:* Règle stricte dans AGENTS.md

#### Cas d'études réels
- "47 Subtle Bugs" (Devrim Ozcay)
- "Bankrupted in 47 Hours"

### TP Fil Rouge (30 min)

**Exercice 8.1: Diagnostiquer un fix loop (15 min)**
- Plonger l'agent dans une boucle de correction
- Analyser les causes
- Briser le cycle

**Exercice 8.2: Code review d'IA (15 min)**
- Générer une feature complète
- Faire une revue systématique
- Identifier les risques (edge cases, sécurité)

---

## Module 9: Tests et Qualité avec IA (55 min)

### Théorie (20 min)

#### TDD assisté par IA
1. Décrire la feature
2. IA génère les tests d'abord
3. IA implémente pour passer
4. Refactor ensemble

#### Coverage improvement
```
Prompt: "Analyse la couverture et identifie les branches manquantes"
```

### TP Fil Rouge (35 min)

**Exercice 9.1: TDD complet (20 min)**
- Feature: Export CSV des données de l'app
- Écrire tests avec l'IA d'abord
- Implémenter pour passer
- Refactor

**Exercice 9.2: Coverage boost (15 min)**
- Analyser coverage actuel
- Générer tests pour branches manquantes
- Atteindre 80%+

---

## Module 10: Conventions d'Équipe et Tensions (55 min)

### Théorie (25 min)

#### Les 4 tensions typiques

**1. Vitesse vs Qualité**
- Certains veulent aller vite avec l'IA
- D'autres veulent du code "propre"

**2. Apprentissage vs Dépendance**
- L'IA empêche-t-elle d'apprendre ?
- Junior vs Senior usage

**3. Transparence vs Magie**
- Certains veulent tout voir (tool calling)
- D'autres veulent du "ça marche"

**4. Coûts partagés**
- Qui paie les tokens ?
- Budget individuel vs collectif

#### Recommandations pratiques

- **Charter IA:** Documenter les choix d'équipe
- **Tag "AI-generated":** Dans les PRs
- **Code review adapté:** Points d'attention spécifiques
- **Formation continue:** Veille partagée

### TP Fil Rouge (30 min)

**Exercice 10.1: Draft charter équipe (15 min)**
- Identifier les conventions pour le projet
- Rédiger les règles
- Définir le process de validation

**Exercice 10.2: Simulation de conflit (15 min)**
- Scénario: bug en prod causé par code IA
- Débrief: qui est responsable ?
- Process: comment éviter

---

## Module 11: Veille et Analyse de Runs (55 min)

### Théorie (30 min)

#### Sources de veille

**Pour les devs:**
- HN (tag LLM) - léger
- r/LocalLLaMA - avancé
- Simon Willison's Blog
- Discord OpenCode/Claude Code

**Documentations à jour:**
- Context7 pour libs
- DeepWiki pour repos populaires

#### Analyser un "run" existant

**Pourquoi:**
- Apprendre des erreurs
- Identifier les améliorations possibles
- "Est-ce que mouliner plus longtemps est utile ?"

**Méthode:**
1. Regarder le transcript
2. Identifier les patterns récurrents
3. Noter les moments d'inefficacité
4. Proposer des améliorations

### TP Fil Rouge (25 min)

**Exercice 11.1: Analyse de run (15 min)**
- Prendre un run précédent de la journée
- Faire commenter par un autre participant
- Identifier: qu'aurait-on pu améliorer ?

**Exercice 11.2: Setup veille personnelle (10 min)**
- Choisir 3 sources
- Configurer les alertes

---

## Module 12: Synthèse et Projet Final (55 min)

### Théorie (10 min)

#### Récapitulatif

**Jour 1:**
- Typologie → Savoir choisir
- Prompt engineering → Savoir demander
- Tool calling → Savoir observer
- AGENTS.md → Savoir documenter
- Coûts → Savoir économiser
- Multimodal → Savoir élargir

**Jour 2:**
- Ralph Loop → Savoir automatiser
- Debugging → Savoir identifier les échecs
- Tests → Savoir valider
- Conventions → Savoir collaborer
- Veille → Savoir apprendre

### TP Final (45 min)

**Projet: Refactor une feature complète**

**Étape 1: Setup (5 min)**
- Choisir une feature existante de l'app
- Vérifier/créer AGENTS.md

**Étape 2: Planification (10 min)**
- Expliquer le comportement actuel
- Proposer une amélioration
- Écrire les tests first

**Étape 3: Exécution (20 min)**
- Implémentation avec l'agent
- Validation continue
- Itérations

**Étape 4: Review (10 min)**
- Self-review systématique
- Identifier les risques
- Livraison (PR ou commit)

**Objectifs par niveau (gestion du temps):**

| Niveau | Fonctionnel | Tests | Code Quality | Livrable |
|--------|-------------|-------|--------------|----------|
| Minimum (45 min) | Feature améliorée fonctionne | ≥1 test passe | Lint clean | Commit |
| Bon | Feature complète | Tests ≥80% coverage | Refactor appliqué | PR créée |
| Excellent | + Edge cases gérés | Tests ≥90% + edge cases | + Documentation | PR review-ready |

**Checkpoints de progression:**
- Minute 5: AGENTS.md validé
- Minute 15: Tests écrits, plan clair
- Minute 35: Implémentation fonctionnelle
- Minute 45: Review et livraison

**Si temps manquant:**
- Priorité: Feature fonctionne avec tests minimaux
- Reporter: Documentation, coverage >80%, refactor avancé

---

## Annexes

### A. AGENTS.md Template (Version complète)

```markdown
# AGENTS.md - [Nom du Projet]

## Role
Tu es un développeur senior sur ce projet.

## Stack Technique
- Backend: [Framework] ([Langage] [Version])
- Frontend: [Framework]
- DB: [Base de données]
- Tests: [Frameworks de test]

## Commands
- Run dev: `make dev` ou `docker-compose up`
- Run tests: `make test`
- Lint: `make lint`
- Build: `docker-compose build`

## Project Structure
[Description rapide de la structure]

## Conventions
- Commits: conventional commits (feat, fix, docs)
- Branches: feature/, fix/, refactor/
- PR: vers develop

## Always
- Créer des tests pour nouvelles features
- Suivre patterns existants
- Valider avec lint avant commit

## Ask First
- Modifier la structure DB
- Ajouter des dépendances
- Changer les configs Docker

## Never
- Commit des secrets (.env, credentials)
- Supprimer des tests existants
- Utiliser `as any`, `@ts-ignore`, `@ts-expect-error`
- Supprimer du code sans comprendre pourquoi
```

### B. Astuces du Formateur

#### tmux + OpenCode
```bash
# Lancer un agent dans un pane dédié
tmux new-session -d -s ia-agent
tmux send-keys -t ia-agent 'opencode' Enter

# Basculer entre agent et code
tmux switch-client -t ia-agent
```

#### Vector-based search vs clever grep
- Vector search: bon pour découverte sémantique
- Clever grep: bon pour patterns précis
- OpenCode utilise les deux intelligemment

---

## Notes pour l'Implémentation

### Timing
- Modules de 55 min avec pauses 10-15 min
- Total: 7h de formation par jour

### Adaptations public

**Public senior:**
- Réduire Module 1 & 2
- Approfondir Module 7 & 8
- Ajouter cas d'architecture

**Public junior:**
- Plus de temps sur Module 2 & 9
- Exercices plus guidés
- Moins d'autonomie attendue

**Format remote:**
- Sessions plus courtes (45 min)
- Plus de breaks
- Partage d'écran obligatoire

### Apps démo alternatives

**Pour publics spécifiques:**
- Jeu vidéo Bevy (Rust) → Public Rust/game
- Quiz infographie Marne → Frontend focus
- Système bridge LQDN → Backend/API focus
- Minimags Quadrature/Reflets → Projets LQDN
- Urbania Flutter → Mobile focus

---

*Structure v1.0 - Scaffold pour développement ultérieur*
