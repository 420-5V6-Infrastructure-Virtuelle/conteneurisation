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

# Étape 7 : Créer un usage funky

Maintenant que l'app tourne, ajoutez une feature de votre choix.

**Exemple d'idée :** un comparateur de modèles spécialisé dans les **conseils médicaux catastrophiques** — l'IA joue le rôle d'un médecin incompétent.

D'autres pistes :
- Générateur de commits git poétiques
- Assistant juridique qui cite des lois inexistantes
- Un coach sportif bidon

**Démarche :**

```
> Je veux adapter Comparia pour [votre idée]. Par où commencer ?
  Quels fichiers modifier pour changer le prompt système ?
```

Travaillez en itérations courtes avec l'IA pour modifier le comportement des modèles comparés.

---

# Étape 8 : Grille de comparaison qualité/prix

**Comparer objectivement 2 modèles sur votre usage funky :**

| Critère | Modèle A (Gemini Flash) | Modèle B (Claude Sonnet) |
|---------|------------------------|--------------------------|
| **Vitesse** (1-5) | | |
| **Créativité** (1-5) | | |
| **Cohérence** (1-5) | | |
| **Tokens consommés** | | |
| **Coût estimé** | | |

**Question clé :** Le modèle premium justifie-t-il son prix pour cette tâche ?

---

# Étape 9 : tmux pour les agents autonomes

**Astuce : laisser tourner les agents en parallèle**

```bash
# Créer une session tmux pour OpenCode
tmux new -s opencode

# Lancer OpenCode
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
- [ ] Comparia qui tourne en local
- [ ] Un token Hugging Face configuré
- [ ] Une version modifiée de Comparia avec un usage original

---

# Ressources

- [Comparia — betagouv](https://github.com/betagouv/comparia)
- [Documentation OpenRouter](https://openrouter.ai/docs)
- [Hugging Face Inference API](https://huggingface.co/docs/api-inference)
- [OpenCode GitHub](https://github.com/opencode-ai/opencode)
