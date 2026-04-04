---
title: "3 - Tool Calling et MCP"
weight: 1040
---

## _Comprendre ce que fait réellement l'agent_

---

# La boîte noire

**Problème avec certains outils :** Vous ne voyez pas ce que l'agent fait.

```
Utilisateur: "Ajoute l'authentification"
┌─────────────────────────────────────┐
│         BOÎTE NOIRE                 │
│   ??? fichiers modifiés ???          │
│   ??? commandes exécutées ???        │
│   ??? accès réseau ???              │
└─────────────────────────────────────┘
Résultat: "C'est fait !"
```

**Risque :** L'agent peut modifier des fichiers critiques, exécuter des commandes dangereuses, ou faire des appels réseau non désirés.

---

# Tool Calling transparent

**OpenCode montre chaque action :**

```
Utilisateur: "Ajoute l'authentification"

[ACTION] Reading file: src/api/routes.py
[ACTION] Reading file: src/models/user.py
[ACTION] Creating file: src/middleware/auth.py
[ACTION] Running command: pytest tests/
[ACTION] Writing file: src/api/routes.py (+15 lines)

Résultat: "J'ai ajouté le middleware d'authentification.
Voulez-vous que je crée les tests ?"
```

**Vous gardez le contrôle :**
- Chaque action est visible
- Vous pouvez annuler
- Vous comprends le raisonnement

---

# Le pattern des outils

**Un "outil" est une capacité donnée à l'agent.**

```yaml
# Exemple d'outil: lecture de fichier
tools:
  - name: read_file
    description: "Lit le contenu d'un fichier"
    parameters:
      path: string  # Chemin du fichier
```

**L'agent décide quand l'utiliser :**

```
Agent: "Pour ajouter l'auth, je dois d'abord comprendre
       la structure existante. Je vais lire les fichiers."
       
[APPEL] read_file(path="src/api/routes.py")
[APPEL] read_file(path="src/models/user.py")
```

---

# Les outils courants

| Outil | Description | Risque |
|-------|-------------|--------|
| `read_file` | Lire un fichier | Aucun |
| `write_file` | Créer/modifier fichier | **Modifications** |
| `run_command` | Exécuter shell | **Élevé** |
| `search_code` | Grep/ripgrep | Aucun |
| `lsp_diagnostics` | Erreurs de type | Aucun |
| `web_fetch` | Requêtes HTTP | **Réseau** |

---

# MCP : Model Context Protocol

**Le standard pour connecter des outils aux agents.**

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   OpenCode   │◄───│    MCP     │◄───│  Serveur    │
│   (Agent)   │     │  (Protocole)│     │  (Outils)   │
└─────────────┘     └─────────────┘     └─────────────┘
```

**Avantages :**
- Outils modulaires
- Communauté partage des MCPs
- Séparation des responsabilités

---

# MCP populaires

| MCP | Usage | Installation |
|-----|-------|--------------|
| **filesystem** | Accès fichiers | Intégré |
| **postgres** | Requêtes DB | `mcp-postgres` |
| **github** | Issues, PRs | `mcp-github` |
| **playwright** | Browser automation | `mcp-playwright` |
| **slack** | Messages Slack | `mcp-slack` |

**Configuration dans OpenCode :**

```yaml
# ~/.config/opencode/mcp.yaml
mcpServers:
  postgres:
    command: mcp-postgres
    args: ["postgresql://user:pass@localhost/db"]
    env:
      POSTGRES_URL: postgresql://...
  
  github:
    command: mcp-github
    args: []
    env:
      GITHUB_TOKEN: ${GITHUB_TOKEN}
```

---

# Playwright et le quirk des screenshots

**Un screenshot n'a pas besoin d'être une image.**

```python
# L'agent peut demander un screenshot
# Playwright renvoie une représentation textuelle !

# Pas ça (lourd, tokens) :
screenshot_base64 = "iVBORw0KGgoAAAANSUhEUgAA..."  # 50KB+

# Mais ça (léger, exploitable) :
page_snapshot = """
- Button "Login" [focused]
- TextBox "Email" 
- TextBox "Password"
- Link "Forgot password?"
"""
```

**OpenCode utilise les snapshots textuels :**
- Économie de tokens massive
- Plus exploitable par le LLM
- Pas de vision nécessaire

---

# Vector Search vs Clever Grep

**Deux approches pour chercher du code :**

## Vector Search (sémantique)

```
Query: "où est la validation du mot de passe ?"
→ [Embedding] → Recherche vectorielle
→ Résultat: src/services/auth.py ligne 42
```

**Avantages :**
- Comprend le sens
- Fonctionne sans mots-clés exacts

**Inconvénients :**
- Indexation nécessaire
- Tokens pour l'embedding
- Peut manquer des détails précis

## Clever Grep (AST-aware)

```bash
# L'agent utilise ripgrep avec patterns
rg "password" --type py -A 3 -B 3
rg "def.*valid" --type py
```

**Avantages :**
- Rapide, pas d'indexation
- Précis sur les noms exacts
- Pas de tokens supplémentaires

**Inconvénients :**
- Nécessite les bons mots-clés

**Recommandation :** Les deux sont complémentaires. OpenCode utilise grep par défaut, vector search si configuré.

---

# TP : Tool Calling en pratique

Le TP fil rouge continue : comprendre et tracer les appels d'outils.

Voir `3_tp_tool_calling.md` →