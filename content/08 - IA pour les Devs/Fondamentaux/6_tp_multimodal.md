---
title: "6 - TP Multimodal"
weight: 1065
draft: false
---

## _Comparer et optimiser les coûts_

---

# Objectif

Comparer différents modèles, mesurer les coûts, et expérimenter la multimodalité.

## Quand utiliser le multimodal

| Cas d'usage | Description |
|-------------|-------------|
| **Screenshot d'erreur** | Montrer l'erreur en UI plutôt que de la décrire |
| **Mockup → code** | Transformer un design en HTML/CSS |
| **Diagramme d'archi** | Analyser un schéma et suggérer une implémentation |
| **Debug visuel** | "Pourquoi la page s'affiche comme ça ?" |

## Limitations à connaître

| Limitation | Impact |
|------------|--------|
| Coût élevé | Une image = 500–2000 tokens selon la résolution |
| Hallucination visuelle | Le modèle peut "lire" du texte qui n'existe pas |
| Résolution limitée | Les détails fins sont souvent manqués |

**L'astuce Playwright :** au lieu d'envoyer une image (lourd), le MCP Playwright retourne une représentation textuelle du DOM. Aucun token d'image, même précision pour naviguer la structure. Voir TP3 Étape 6.

---


## Étape 1 : Screenshot d'erreur

**Prendre un screenshot d'une erreur dans votre app :**

```bash
# Si l'app tourne localement
# Capturer un screenshot de l'erreur avec un outil de screenshot
```

**Prompt multimodal :**

```
Voici un screenshot d'erreur dans mon application.
Analyse le problème et propose une solution.
[Image attachée]
```

---

## Étape 2 : Playwright Snapshot

**Utiliser le snapshot textuel :**

```bash
# Avec OpenCode configuré avec Playwright MCP
opencode
```

```
>Lance l'app localement avec playwright et prends un snapshot
>de la page d'accueil. Analyse la structure.
```

**Observer :**
- Le snapshot est-il textuel ou image ?
- Combien de tokens consommés ?
- Est-ce que l'agent comprend la structure ?

---

## Étape 3 : Transformer un mockup

**Trouver un mockup simple (ou en créer un) :**

```
Voici le design de la page de profil.
Génère le HTML/CSS correspondant.
[Image du mockup]
```

**Comparer :**
- La fidélité du résultat
- Le temps de génération
- Les tokens consommés

---
