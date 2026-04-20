---
title: "3 - TP Tool Calling et MCP"
weight: 1045
---

## _Observer ce que fait l'agent_

> ⏱ **1h**

> **Outil principal :** Codex CLI. Remplacer `codex` par `opencode` ou `claude` selon votre outil. La config MCP diffère selon l'outil — voir les notes en contexte.

---

# Objectif

Comprendre le pattern tool calling et configurer un MCP basique.

---

# Étape 1 : Observer les appels

**Lancer l'agent :**

```bash
codex       # Codex : tool calls visibles dans le TUI nativement
            # OpenCode : opencode --verbose
            # Claude Code : claude --verbose
```

**Prompt simple :**
```
>Liste tous les endpoints de l'API
```

**Observer dans les logs :**
```
[TOOL] lsp_symbols(filePath="src/api/routes.py", scope="document")
[TOOL] read_file(filePath="src/api/routes.py")
[TOOL] grep(pattern="@app\.(get|post|put|delete)", output="content")
```

**Grille d'observation du tool calling :**

| Critère | Oui/Non | Notes |
|---------|---------|-------|
| Lit les fichiers avant de répondre | | |
| Utilise plusieurs outils en séquence | | |
| Vérifie les résultats | | |
| Explique son raisonnement | | |
| Demande clarification si ambigu | | |

**Noter :**
- Quels outils ont été utilisés ?
- Dans quel ordre ?
- Combien de tokens ?

---

# Étape 2 : Compter les appels

**Prompt complexe :**
```
>Ajoute un endpoint GET /users/{id}/posts
>qui retourne les posts d'un utilisateur
>avec pagination (limit, offset)
```

**Tracer les appels :**
```
┌─────────────────────────────────────────────┐
│ Appel #1: read_file(src/api/routes.py)      │
│ Appel #2: read_file(src/models/post.py)     │
│ Appel #3: read_file(src/models/user.py)     │
│ Appel #4: write_file(src/api/routes.py)     │
│ Appel #5: run_command(pytest tests/)        │
└─────────────────────────────────────────────┘
```

**Question :** L'agent a-t-il modifié les bons fichiers ?

---

# Étape 3 : MCP basique

**Configurer un MCP simple :**

```yaml
# OpenCode : ~/.config/opencode/mcp.yaml
mcpServers:
  filesystem:
    command: mcp-filesystem
    args: ["/home/user/mon-app-demo"]
```

> **Claude Code :** la config MCP va dans `~/.claude/settings.json` sous la clé `mcpServers` (même format).
>
> **Codex CLI :** config MCP via `~/.codex/config.toml` ou variables d'environnement selon la version.

**Redémarrer l'agent :**

```bash
codex   # OpenCode : opencode | Claude Code : claude
```

**Vérifier que le MCP est chargé :**
```
>Liste les outils disponibles
```

---

# Étape 4 : MCP GitHub (optionnel)

**Pour les projets hébergés sur GitHub :**

```yaml
mcpServers:
  github:
    command: mcp-github
    env:
      GITHUB_TOKEN: ${GITHUB_TOKEN}
```

**Usage :**
```
>Crée une issue pour le bug que tu viens de trouver
>Liste les PRs ouvertes sur ce repo
```

---

# Étape 5 : L'anti-pattern "tool spam"

**Observer un agent qui boucle :**

```
[TOOL] read_file(src/api/routes.py)
[TOOL] grep(pattern="auth")
[TOOL] read_file(src/api/routes.py)  # <-- Déjà lu !
[TOOL] grep(pattern="auth")           # <-- Déjà fait !
[TOOL] read_file(src/api/routes.py)  # <-- Encore !
```

**Cause :** L'agent ne "souvient" pas ce qu'il a fait.
**Solution :** AGENTS.md clair, prompts structurés.

---

# Étape 6 : Playwright MCP

**Installer le MCP Playwright :**

```yaml
mcpServers:
  playwright:
    command: mcp-playwright
```

**Usage :**
```
>Ouvre l'app localement et vérifie que la page d'accueil s'affiche
```

**Observer :**
```
[TOOL] playwright_navigate(url="http://localhost:8000")
[TOOL] playwright_snapshot()  # Pas de screenshot PNG !
[TOOL] playwright_click(ref="button.login")
```

**Noter le format texte du snapshot :**
- Pas d'image base64
- Représentation accessible du DOM
- Tokens économisés

---

# Étape 7 : L'exercice de spéculation

**Objectif : Identifier quand l'agent "devine" au lieu de vérifier.**

**Prompt volontairement vague :**
```
>Optimise les performances de l'application
```

**Observer le comportement :**

| Action | Attendu | Observé |
|--------|---------|---------|
| A lu les fichiers de configuration | | |
| A vérifié les métriques actuelles | | |
| A demandé des clarifications | | |
| A proposé des solutions spécifiques | | |
| A identifié les bottlenecks réels | | |

**Questions à se poser :**
1. L'agent a-t-il lu les fichiers avant de proposer ?
2. A-t-il identifié le contexte (DB, backend, frontend) ?
3. Les suggestions sont-elles génériques ou ciblées ?

**Correction :**
```markdown
>Avant de proposer des optimisations:
>1. Lis vite-fait -l pour voir les processus actifs
>2. Lis le docker-compose.yml pour identifier les services
>3. Lis les logs récents pour les erreurs/perfs
>4. Seulement ensuite, propose 3 optimisations ciblées
```

**Pattern retenu :** Toujours forcer la lecture avant l'action.

---

# Étape 8 : MCP context7 — vérifier la doc d'une librairie

Quand un agent travaille avec une librairie dont il peut avoir une connaissance périmée, le MCP context7 permet de lui injecter la documentation réelle à jour.

**Exemple de pattern (FastAPI 0.115 — `lifespan` remplace `on_event`) :**

```
> FastAPI 0.115 deprecated the on_event startup/shutdown hooks.
  Let me check the current API:

[TOOL] context7_resolve-library-id [libraryName=fastapi, query=lifespan startup shutdown]
[TOOL] context7_query-docs [libraryId=/tiangolo/fastapi, query=lifespan context manager app startup]

Now I understand the new lifespan pattern. Let me update the code:
```

**Le pattern :**
1. L'agent résout l'ID de la librairie dans le registre context7
2. Il interroge la doc pour la version précise
3. Il corrige ou implémente avec la vraie API

**Applicable à n'importe quelle librairie** : FastAPI, SQLAlchemy, Next.js… Dès qu'une version récente casse une API connue.

**Configurer context7 :**

```yaml
# ~/.config/opencode/mcp.yaml
mcpServers:
  context7:
    command: npx
    args: ["-y", "@upstash/context7-mcp@latest"]
```

---

# Étape 9 : Comparaison des approches

**Sans MCP (fichiers locaux uniquement) :**
- L'agent lit les fichiers
- Modifie le code
- Exécute des commandes

**Avec MCP (outils externes) :**
- L'agent interagit avec GitHub
- Peut tester via Playwright
- Connecté à la DB via postgres MCP
- Vérifie la doc à jour via context7

**Question :** Quel MCP serait le plus utile pour votre stack actuelle ?

---

# Livrable

À la fin de ce TP :

- [ ] Comprendre les appels d'outils dans les logs
- [ ] Avoir configuré au moins un MCP
- [ ] Connaître les outils disponibles par défaut
- [ ] Identifier les risques potentiels

---

# Checkpoint

**Pattern retenu :** Toujours lire les logs pour comprendre ce que l'agent a fait.

**Question clé :** Combien de tokens auraient été économisés avec un AGENTS.md ?

---

# Prochain module

Module 4 : Bonnes Pratiques - AGENTS.md, Makefile, Docker, README.