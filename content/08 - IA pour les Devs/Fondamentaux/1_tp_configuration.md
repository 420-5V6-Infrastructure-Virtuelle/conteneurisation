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
- [ ] Un compte OpenRouter avec ~$10 de crédit

---

# Étape 1 : OpenRouter

**Créer votre compte et récupérer la clé API :**

1. Aller sur [openrouter.ai](https://openrouter.ai)
2. Créer un compte
3. Générer une clé API
4. Ajouter du crédit ($10 minimum)

**Pourquoi OpenRouter ?**
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

# Choisir son app de travail

Comparia est l'app démo de cette formation, mais vous pouvez appliquer les mêmes exercices à votre propre projet. Voici comment calibrer le niveau :

| Type d'app | Exemples | Niveau avec LLM |
|------------|----------|-----------------|
| **CRUD / utilitaire** | Lecteur RSS, todo app, lecteur de musique, API simple | ✅ Excellent — patterns bien documentés, l'agent excelle |
| **App "métier"** | Logique domaine complexe, règles métier spécifiques, orchestration | ⚠️ Avancé — sans guardrails, l'agent se cassera la gueule |

Pour cette formation : on reste sur Comparia (CRUD + API bien structurée). Les patterns avancés avec guardrails arrivent en jour 2.

---

# Étape 4 : Cloner et faire marcher Comparia

L'application démo est **[Comparia](https://github.com/betagouv/comparia)**, un outil de comparaison de modèles d'IA développé par beta.gouv.fr.

```bash
git clone https://github.com/betagouv/comparia
cd comparia
```

**Lire le README et lancer l'app :**

```bash
# Explorer la structure du projet avec OpenCode
opencode
```

```
> Analyse ce projet, explique sa structure et dis-moi comment le lancer en local
```

Suivez les instructions générées pour installer les dépendances et démarrer l'application.

**Vérification :** l'interface est accessible dans le navigateur.

---

# Étape 5 : Obtenir un token Hugging Face

Comparia s'appuie sur des modèles accessibles via [Hugging Face](https://huggingface.co). Vous avez besoin d'un token d'accès.

**Option A — Compte manuel (5 min) :**
1. Créer un compte sur [huggingface.co](https://huggingface.co)
2. Aller dans Settings → Access Tokens
3. Créer un token avec les droits `read`

**Option B — Explorer l'automatisation :**

Réfléchissez aux différentes façons d'instrumentaliser cette étape :

| Méthode | Outil | Cas d'usage |
|---------|-------|-------------|
| **Navigation visuelle** | Playwright / Puppeteer | Automatiser la création de compte |
| **API directe** | `curl` / SDK HF | Gérer les tokens programmatiquement |
| **Mode recherche IA** | OpenCode en mode recherche | Trouver des alternatives gratuites |

```
> Recherche des alternatives gratuites à Hugging Face Inference API
  pour tester des LLMs open-source sans créer de compte
```

Observez comment l'IA explore et présente ses résultats en **mode recherche**.

**Configurer le token :**
```bash
export HF_TOKEN="hf_votre_token"
```

---

# Étape 6 : Faire tourner l'app simplifiée

Comparia a une version allégée pour le développement. Avec l'aide de l'IA :

```
> Comment lancer Comparia en mode simplifié / développement local
  sans toute l'infrastructure de production ?
```

Objectif : avoir une interface fonctionnelle avec au moins un modèle accessible.

**Points à observer :**
- L'IA lit-elle correctement la doc du projet ?
- Propose-t-elle des raccourcis pertinents ?
- Gère-t-elle bien les erreurs de configuration ?

---

# Étape 7 : Mode verbose et statusline

## `claude --verbose`

Indispensable pour comprendre ce que fait réellement l'agent :

```bash
claude --verbose
```

Vous verrez les tool calls en temps réel : quels fichiers sont lus, quelles commandes sont exécutées, combien de tokens sont consommés. C'est la première chose à activer quand un comportement vous surprend.

## Statusline

Ajoutez la statusline dans `~/.claude/settings.json` pour suivre votre consommation de contexte en permanence :

```json
{
  "statusLine": {
    "type": "command",
    "command": "npx -y ccstatusline@latest",
    "padding": 0
  }
}
```

Exemple de ce que vous voyez :

```
Model: Sonnet | Ctx: 89.5k | Cost: $2.11 | Ctx(u): 56.0%
```

**Règle simple :** `Ctx(u)` > 70% → `/compact`. > 85% → `/clear`.

---

# Livrable

À la fin de ce TP, vous devez avoir :

- [ ] OpenCode configuré avec OpenRouter
- [ ] Comparia qui tourne en local
- [ ] Un token Hugging Face configuré
- [ ] `claude --verbose` testé, statusline configurée

---

# Ressources

- [Comparia — betagouv](https://github.com/betagouv/comparia)
- [Documentation OpenRouter](https://openrouter.ai/docs)
- [Hugging Face Inference API](https://huggingface.co/docs/api-inference)
- [OpenCode GitHub](https://github.com/opencode-ai/opencode)
