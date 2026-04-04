# Formation OpenAI Codex - Human Coders

**Source:** https://www.humancoders.com/formations/openai-codex  
**Duration:** 2 days  

## Description

Cette formation OpenAI Codex vous permettra de maîtriser la génération de code assistée par IA avec Codex.

Nous aborderons la prise en main de Codex, la configuration dans vos IDE, la création de contextes multi-fichiers, l'utilisation des actions avancées, l'exécution sécurisée de code et l'orchestration multi-agents.

### Objectifs pédagogiques

- Structurer un prompt efficace pour générer du code fiable
- Configurer Codex dans son IDE et gérer un projet multi-fichiers
- Utiliser les actions de Codex pour documenter, tester et refactorer
- Créer des Tools personnalisés avec function calling
- Mettre en place des agents spécialisés pour automatiser le développement
- Sécuriser l'exécution du code et valider les résultats

### Public

Développeur·se·s souhaitant accélérer leur productivité avec Codex tout en gardant la maîtrise du code.

### Pré-requis

- Connaître les bases d'un langage de programmation (Python, JS, Java…)
- Savoir utiliser un IDE (VS Code, JetBrains…)
- Ordinateur portable à apporter

---

## Programme

### Jour 1 — Prise en main de Codex & génération assistée

#### Introduction à OpenAI Codex
- Qu'est-ce que Codex, son positionnement dans la gamme OpenAI
- Comparaison avec GitHub Copilot, Cursor, Claude Code
- Cas d'usage : génération de code, refactor, documentation, tests, outils internes

#### Comprendre le prompting pour le code
- Notions essentielles : prompt, instructions, context window, tokens
- Structurer un prompt technique (objectifs, contraintes, style, format attendu)
- Les règles d'or du prompt engineering pour le code
- Création d'un contexte persistant pour un projet multi-fichiers
- Gestion de l'historique, filesets, patterns de prompts

#### Installation & configuration
- Accès à OpenAI (API, Playground, intégrations)
- Installation de l'extension Codex dans différents IDE
- Configuration de VS Code / JetBrains / Neovim
- Paramétrage des modèles (gpt-4.1, o3-mini, o3-code, etc.)

#### Codex en interaction
- Utiliser le chat de Codex pour générer, expliquer ou corriger du code
- Appel de fichiers : analyse et modification du code existant
- Navigation intelligente : résumé multi-modules, refactor global
- Premier ensemble de paramètres : model, temperature, system prompt

#### Les Actions
- "Create File", "Edit File", "Fix Problems"
- "Review", "Explain", "Find Bugs"
- "Document", "Generate README"
- "Plan", "Todo", "Refactor"
- Suivi des coûts, logs, historique et diffs

***Mises en pratique:***

- *Initialiser un projet et générer la structure complète avec Codex*
- *Appeler un prompt en ligne de commande avec l'API*
- *Générer un module + tests + documentation cohérente*

---

### Jour 2 — Fonctions avancées, Tools & Agents spécialisés

#### Actions avancées & Customisation
- Personnaliser les actions
- Créer des templates d'instructions pour automatiser les workflows
- Organisation des prompts : librairie interne, patterns, conventions d'équipe
- Configurations avancées : permissions, accès au code, restrictions

#### Les Tools OpenAI
- Notion de Tool / Function calling
- Créer son propre tool (ex : accès base de données, API interne)
- Déclarer un schéma, définir des contraintes, orchestrer Codex + Tools
- Cas d'usage (génération automatique de migrations, requêtes SQL sécurisées, manipulation de données, automatisation de tâches de build)

#### Agents & sous-agents
- Qu'est-ce qu'un agent spécialisé ?
- Agents assistants vs agents exécutants
- Orchestration multi-agents (ex : "reviewer", "coder", "architect")
- Création d'un écosystème d'agents pour un projet complet
- Bonnes pratiques & limites actuelles

#### Exécution sécurisée du code
- Code Interpreter / Sandbox / execution plan
- Tracer l'exécution, comprendre les résultats, réduire les hallucinations
- Produire du code fiable : tests, benchmarks, validations automatiques

***Mises en pratique:***

- *Créer une action personnalisée pour organiser une feature*
- *Ajouter un tool personnalisé (API interne) à Codex*
- *Mettre en place un duo d'agents (architecte + développeur) pour implémenter une nouvelle fonctionnalité*
- *Générer un plan de refactor automatisé sur un projet existant*

---

## Points clés à retenir pour ma formation

### Ce qui fonctionne bien dans ce plan:
- Structure claire Jour 1 (basics) → Jour 2 (avancé)
- TP fil rouge progressif
- Actions concrètes listées explicitement
- Focus sur le function calling et les tools personnalisés

### Ce que je veux adapter:
- **Outil principal: OpenCode** (agnostique, open source) au lieu de Codex
- Vue comparative: Claude Code, Codex, OpenCode comme catégorie TUI agentique
- Cursor/Copilot comme catégorie "assistant autocomplete"
- Ajouter MCP (Model Context Protocol)
- Ajouter AGENTS.md et conventions d'équipe
- Ajouter gestion des coûts / modèles frugaux (Gemini, GLM-4, Qwen)
- Pattern Ralph Loop pour agents autonomes parallèles