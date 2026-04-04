---
title: "1 - TP Configuration et Premier Pas"
weight: 1025
---

## _Mise en route de l'environnement_

---

# Prérequis

Avant de commencer, assurez-vous d'avoir :

- [ ] Python 3.11+ ou Node.js 18+
- [ ] Docker & docker-compose
- [ ] Git configuré
- [ ] Un éditeur de code (VSCode recommandé)
- [ ] Un compte OpenRouteravec ~$10 de crédit

---

# Étape 1 : OpenRouter

**Créer votre compte et récupérer la clé API :**

1. Aller sur [openrouter.ai](https://openrouter.ai)
2. Créer un compte
3. Générer une clé API
4. Ajouter du crédit ($10 minimum)

**PourquoiOpenRouter ?**
- Accès à 200+ modèles
- tarifs à la token
- pas de vendor lock-in
- modèles frugaux (Gemini Flash) comme premium (Claude Opus)

---

# Étape 2 : Installation d'OpenCode

```bash
# Via npm
npm install -g @opencode-ai/opencode

# Ou via le script officiel
curl -fsSL https://opencode.ai/install.sh | bash
```

**Vérification :**
```bash
opencode --version
```

---

# Étape 3 : Configuration

Créer le fichier de configuration :

```bash
#~/.config/opencode/config.yaml
providers:
  openrouter:
    api_key: ${OPENROUTER_API_KEY}
    base_url: https://openrouter.ai/api/v1
    
default_provider: openrouter
default_model: google/gemini-2.0-flash  # Frugal
```

**Variable d'environnement :**
```bash
export OPENROUTER_API_KEY="sk-or-v1-votre-clé"
```

---

# Étape 4 : Choisir l'application démo

Choisissez selon votre confort et intérêts :

| Option | Stack | Complexité | Repo |
|--------|-------|------------|------|
| **Minimal Python** | FastAPI + SQLite | ~300 lignes | `minimal-fastapi` |
| **Full Python** | FastAPI + PostgreSQL | Réaliste | `full-stack-fastapi` |
| **SvelteKit** | SvelteKit minimal | ~800 lignes | `swyxkit` |
| **Elixir/Phoenix** | Langage non maîtrisé | Challenge | `phoenix-elixir-app` |

**Pourquoi l'option Elixir/Phoenix ?**

Travailler sur un langage/framework **que vous ne connaissez pas** simule un scénario réaliste : l'IA vous aide à monter en compétence rapidement sur une nouvelle stack.

```bash
# Cloner l'application choisie
git clone <repo-choisi> mon-app-demo
cd mon-app-demo
```

---

# Étape 5 : Premier test

**LancerOpenCode sur le projet :**

```bash
cd mon-app-demo
opencode
```

**Prompt de test :**
```
>Analyse ce projet et explique-moi sa structure
```

**Observations à noter :**
- Combien de tokens utilisés ?
- Temps de réponse ?
- Qualité de l'analyse ?

---

# Étape 6 : Changer de modèle

**Essayer un modèle premium :**

```yaml
# ~/.config/opencode/config.yaml
default_model: anthropic/claude-3.5-sonnet
```

**Refaire le même prompt et comparer :**
- Qualité de la réponse
- Temps de réponse
- Coût (tokens utilisés)

---

# Étape 7 : Grille de comparaison qualité/prix

**Comparer objectivement vos 2 modèles testés :**

| Critère | Modèle A (Gemini Flash) | Modèle B (Claude Sonnet) |
|---------|------------------------|--------------------------|
| **Vitesse** (1-5) | | |
| **Pertinence** (1-5) | | |
| **Précision** (1-5) | | |
| **Suggestions utiles** (1-5) | | |
| **Tokens consommés** | | |
| **Coût estimé** | | |
| **Score qualité** (moyenne) | | |

**Formule du score qualité :**
```
Score = (Pertinence + Précision + Suggestions) / 3
```

**Question clé :** Le modèle premium justifie-t-il son prix pour cette tâche ?

---

# Étape 7 : tmux pour les agents autonomes

**Astuce : laisser tourner les agents en parallèle**

```bash
# Créer une session tmux pourOpenCode
tmux new -s opencode

# LancerOpenCode
opencode

# Détacher : Ctrl+B puis D
# Revenir : tmux attach -t opencode
```

**Pourquoi ?**
- Agents autonomes (Ralph Loop) peuvent tourner des heures
- Vous gardez le contrôle du terminal principal
- Sessions persistantes entre les déconnexions SSH

---

# Livrable

À la fin de ce TP, vous devez avoir :

- [ ] OpenCode configuré avec OpenRouter
- [ ] Testé au moins 2 modèles différents
- [ ] Une application démo clonée
- [ ] Noté les différences de comportement entre modèles

---

# Ressources

- [Documentation OpenRouter](https://openrouter.ai/docs)
- [Modèles disponibles](https://openrouter.ai/models)
- [OpenCode GitHub](https://github.com/opencode-ai/opencode)