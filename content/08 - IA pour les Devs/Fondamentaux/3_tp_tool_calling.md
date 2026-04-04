---
title: "3 - TP Tool Calling et MCP"
weight: 1045
---

## _Observer ce que fait l'agent_

---

# Objectif

Comprendre le pattern tool calling et configurer un MCP basique.

---

# Étape 1 : Observer les appels

**Lancer OpenCode en mode verbeux :**

```bash
opencode --verbose
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
# ~/.config/opencode/mcp.yaml
mcpServers:
  filesystem:
    command: mcp-filesystem
    args: ["/home/user/mon-app-demo"]
```

**Redémarrer OpenCode :**

```bash
opencode
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

# Étape 7 : Comparaison des approches

**Sans MCP (fichiers locaux uniquement) :**
- L'agent lit les fichiers
- Modifie le code
- Exécute des commandes

**Avec MCP (outils externes) :**
- L'agent interagit avec GitHub
- Peut tester via Playwright
- Connecté à la DB via postgres MCP

**Question :** Quel MCP serait utile pour votre workflow ?

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