---
title: "3 - TP Tool Calling et MCP"
weight: 1045
---

## _Observer ce que fait l'agent_

> ⏱ **45 min**

> **Outil principal :** Codex CLI. Remplacer `codex` par `opencode` ou `claude` selon votre outil.

---

# Objectif

Configurer des MCPs utiles et observer concrètement leur impact sur le comportement de l'agent.

---

# Étape 1 : Observer les appels

**Lancer l'agent :**

```bash
codex       # tool calls visibles nativement
            # OpenCode : opencode --verbose
            # Claude Code : claude --verbose
```

**Prompt simple :**
```
>Liste tous les endpoints de l'API
```

**Observer dans les logs :**
```
[TOOL] read_file(filePath="src/api/routes.py")
[TOOL] grep(pattern="@app\.(get|post|put|delete)", output="content")
```

**Grille d'observation :**

| Critère | Oui/Non | Notes |
|---------|---------|-------|
| Lit les fichiers avant de répondre | | |
| Utilise plusieurs outils en séquence | | |
| Vérifie les résultats | | |
| Demande clarification si ambigu | | |

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

# Étape 3 : Recherche web — MCPs ou natif ?

Pas d'exercice ici, juste un point de configuration à connaître.

**Claude Code et Codex** ont la recherche web intégrée nativement — rien à faire.

**OpenCode** n'a pas de recherche intégrée. Il faut ajouter un MCP :

```yaml
# ~/.config/opencode/mcp.yaml
mcpServers:
  brave-search:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-brave-search"]
    env:
      BRAVE_API_KEY: ${BRAVE_API_KEY}
```

Les alternatives `websearch` et `ddg_search` existent aussi (pas de clé requise pour ddg). Le résultat côté agent est identique dans les trois cas — c'est juste le moteur qui change.

---

# Étape 4 : context7 — ancrer l'agent dans la vraie doc

Quand l'agent travaille avec une librairie dont il peut avoir une connaissance périmée, context7 lui injecte la documentation réelle à jour.

**Configurer context7 :**

```yaml
# ~/.config/opencode/mcp.yaml  (Claude Code : ~/.claude/settings.json)
mcpServers:
  context7:
    command: npx
    args: ["-y", "@upstash/context7-mcp@latest"]
```

**Redémarrer l'agent, puis tester :**

```
>use context7
>Comment migrer de on_event vers lifespan dans FastAPI ?
```

**Observer le pattern :**
```
[TOOL] context7_resolve-library-id [libraryName=fastapi]
[TOOL] context7_query-docs [libraryId=/tiangolo/fastapi, query=lifespan startup shutdown]

→ L'agent répond avec la vraie API FastAPI 0.115, pas ce qu'il "croit" savoir
```

**Tester avec une librairie de votre choix** — n'importe quelle lib où une version récente a cassé une API connue.

---

# Étape 5 : Playwright — voir et interagir avec Comparia

**Pourquoi Playwright plutôt qu'un screenshot ?**

Playwright renvoie une représentation textuelle du DOM, pas une image. Économie de tokens massive, et le LLM peut raisonner dessus sans vision.

**Configurer le MCP Playwright :**

```yaml
mcpServers:
  playwright:
    command: npx
    args: ["-y", "@playwright/mcp@latest"]
```

**L'exercice :**

Comparia tourne en local. Trouvez le port en lisant le README ou le `docker-compose.yml` du projet, puis donnez ce prompt à l'agent :

```
>Ouvre Comparia sur http://localhost:<PORT>
>Vérifie que la page d'accueil charge correctement.
>Si ce n'est pas le cas, attends et réessaie jusqu'à ce qu'elle soit disponible.
>Une fois chargée, décris ce que tu vois et interagis avec l'interface :
>lance une comparaison entre deux modèles avec le prompt "Explique le tool calling en 2 phrases".
```

**Observer les appels :**
```
[TOOL] playwright_navigate(url="http://localhost:<PORT>")
[TOOL] playwright_snapshot()
→ snapshot textuel du DOM, pas d'image

[TOOL] playwright_navigate(...)   ← si page pas encore dispo, l'agent boucle
[TOOL] playwright_snapshot()
→ "Page loaded: ComparIA — comparer les modèles d'IA"

[TOOL] playwright_click(ref="textarea.prompt-input")
[TOOL] playwright_fill(value="Explique le tool calling en 2 phrases")
[TOOL] playwright_click(ref="button[type=submit]")
[TOOL] playwright_snapshot()
→ l'agent lit les réponses des deux modèles
```

**Points d'attention :**
- Pas de screenshot PNG — représentation DOM accessible
- L'agent boucle naturellement si le serveur n'est pas encore prêt
- Il peut lire les réponses des modèles comme du texte

---

# Étape 6 : LSP — navigation sémantique du code

Le cours couvre les trois situations. En pratique :

**OpenCode :** rien à faire, LSP est natif.

**Claude Code :** installez l'extension dans votre IDE (VS Code ou JetBrains). Claude Code s'y branche automatiquement.

**Codex CLI :** installez [Serena](https://github.com/oraios/serena), le MCP qui enveloppe votre language server :

```yaml
# ~/.codex/config.yaml
mcpServers:
  serena:
    command: uvx
    args: ["serena-mcp-server"]
    env:
      PROJECT_ROOT: /chemin/vers/comparia
```

**Tester la différence :**

```
# Sans LSP — grep approximatif
>Trouve toutes les fonctions qui gèrent l'authentification

# Avec LSP — navigation précise
>Trouve toutes les références à la fonction authenticate()
>et liste leurs fichiers et numéros de ligne
```

Avec LSP, l'agent ne cherche pas par mots-clés — il interroge le language server, qui connaît la structure du code.

---

# Étape 7 : L'anti-pattern "tool spam"

**Observer un agent qui boucle :**

```
[TOOL] read_file(src/api/routes.py)
[TOOL] grep(pattern="auth")
[TOOL] read_file(src/api/routes.py)  # <-- Déjà lu !
[TOOL] grep(pattern="auth")           # <-- Déjà fait !
```

**Cause :** contexte trop chargé, AGENTS.md absent ou vague.
**Solution :** AGENTS.md clair sur la structure du projet, prompts structurés.

---

# Étape 8 : GitHub — gh CLI ou MCP ?

C'est l'exemple parfait de la philosophie bash vs MCP. Les deux fonctionnent, les trade-offs sont réels.

## Approche 1 — gh CLI (bash)

Si `gh` est installé et authentifié sur votre machine, l'agent peut l'utiliser directement sans aucune config :

```
>Crée une issue sur le repo Comparia pour signaler
>que la page d'accueil met plus de 3s à charger.
>Inclure le snapshot Playwright comme description.
```

L'agent exécutera quelque chose comme :
```bash
gh issue create \
  --repo betagouv/comparia \
  --title "Page d'accueil lente (>3s)" \
  --body "..."
```

**Avantages :** zéro config, transparent, aucune surface d'attaque supplémentaire.  
**Limite :** l'agent a accès à tout ce que `gh` peut faire — avec vos permissions complètes.

## Approche 2 — MCP GitHub

Le MCP GitHub expose des outils typés (`create_issue`, `list_pull_requests`, `get_file_contents`…) avec un périmètre configurable.

```yaml
# ~/.config/opencode/mcp.yaml
mcpServers:
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: ${GITHUB_TOKEN}
```

> Pour Claude Code : même format dans `~/.claude/settings.json` sous `mcpServers`.

```
>use mcp github
>Liste les 5 dernières PRs ouvertes sur betagouv/comparia
>et résume les changements de chacune
```

**Avantages :** interface propre, token avec permissions fines (lecture seule si vous voulez), contexte riche.  
**Limite :** le MCP lit les issues et PRs — qui peuvent contenir des tentatives de prompt injection (voir section risques dans le cours).

## L'exercice

Faites les deux, comparez :

1. Avec `gh` : créez une issue sur un de vos repos
2. Avec le MCP : listez les PRs ouvertes et demandez un résumé

**Question :** Laquelle des deux approches vous semble plus adaptée à votre contexte ? Pourquoi ?

---

# Bonus : autres MCPs utiles

Une fois que vous êtes à l'aise avec le pattern MCP, voici ce que la communauté utilise le plus.

## Communication

**Slack**
```yaml
slack:
  command: npx
  args: ["-y", "@modelcontextprotocol/server-slack"]
  env:
    SLACK_BOT_TOKEN: ${SLACK_BOT_TOKEN}
    SLACK_TEAM_ID: ${SLACK_TEAM_ID}
```
L'agent peut envoyer des messages, lire des canaux, rechercher dans l'historique. Utile pour des notifications automatiques en fin de tâche.

**Telegram**  
Plusieurs MCPs communautaires disponibles — pratique si votre équipe est sur Telegram plutôt que Slack.

## Gestion de projet

**Linear** — issues et sprints structurés, API propre, très utilisé dans les startups tech.

**Notion** — lecture et écriture de pages. Utile si votre doc technique est dans Notion.

**Jira** — pour les équipes enterprise. MCP officiel Atlassian disponible.

## Données et infra

**PostgreSQL / SQLite** — l'agent peut requêter directement votre base. Très puissant pour le debug ou l'exploration de données.

```yaml
postgres:
  command: npx
  args: ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
```

**Filesystem étendu** — accès à des dossiers en dehors du projet courant (logs système, exports, etc.).

**Cloudflare** — gestion de DNS, Workers, KV store directement depuis l'agent.

## Pour trouver d'autres MCPs

- [mcp.so](https://mcp.so) — répertoire communautaire
- [MCP Registry officiel](https://github.com/modelcontextprotocol/registry)
- Chercher `mcp-server-*` sur npm ou PyPI

> **Rappel sécurité :** vérifiez toujours le repo GitHub d'un MCP avant de l'installer via `npx -y`. Un package compromis s'exécute avec vos permissions.

---

# Livrable

À la fin de ce TP :

- [ ] Avoir observé et compris les tool calls dans les logs
- [ ] context7 configuré et testé sur une librairie réelle
- [ ] Playwright : Comparia chargée et interagie via l'agent
- [ ] LSP configuré (ou compris pourquoi c'est déjà là)
- [ ] GitHub : les deux approches testées (gh CLI + MCP)
- [ ] Avoir identifié un anti-pattern tool spam dans vos observations

---

# Prochain module

Module 4 : Bonnes Pratiques - AGENTS.md, Makefile, Docker, README.
