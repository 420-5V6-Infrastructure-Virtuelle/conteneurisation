---
title: "11 - TP Frontend & Skills documentaires"
weight: 2036
---

## _Créer des skills qui savent lire tes docs_

> ⏱ **1h30**

---

# Principe

Un skill Codex est un fichier markdown dans `.codex/commands/`. Quand vous tapez `/nom-du-skill`, l'agent exécute les instructions du fichier.

Ce qui change ici : on va créer des skills qui **donnent à l'agent les instructions pour aller chercher le contexte lui-même** dans des documents locaux (PDF de référence, guides internes). L'agent fait le travail de conversion et d'extraction — pas vous.

---

# Partie 1 : Skill générique search-pdf — 20 min

Avant de créer des skills spécialisés, il faut un skill de base qui sait interroger un PDF.

## Étape 1 : Générer le skill avec $skill-creator

```
/skill-creator
```

Décrivez ce que vous voulez :

```
Crée un skill search-pdf qui :
- prend deux arguments : le chemin vers un PDF et une requête de recherche
- si le fichier .md correspondant n'existe pas, convertit le PDF avec pdftotext
  (fallback : pdfminer.six si pdftotext absent)
- cherche la requête dans le texte extrait avec ripgrep (-A 20 -i)
- retourne les passages pertinents avec leur contexte
```

## Étape 2 : Tester sur le RGAA

```bash
mkdir -p docs
curl -L https://accessibilite.numerique.gouv.fr/doc/RGAA-v4.1.pdf -o docs/RGAA_4.1.pdf
```

```
/search-pdf docs/RGAA_4.1.pdf "critère 1.1"
```

Vérifiez que l'agent extrait bien les bons passages. Relisez le skill généré, ajustez si besoin.

## Étape 3 : Tester sur un bouquin de sécurité

Placez un PDF de sécurité dans `docs/` (OWASP Testing Guide, The Web Application Hacker's Handbook, ou tout autre livre en PDF).

```
/search-pdf docs/mon-bouquin-secu.pdf "SQL injection"
```

Le skill doit fonctionner sur n'importe quel PDF sans modification.

---

# Partie 2 : Skill a11y-review — 40 min

## Contexte

Le RGAA (Référentiel Général d'Amélioration de l'Accessibilité) est le standard d'accessibilité numérique en France. C'est un PDF de ~250 pages. Les équipes front en ont besoin mais personne ne le lit en entier.

**L'idée :** un skill qui, quand vous lui passez un composant, utilise `search-pdf` pour extraire les critères RGAA pertinents et faire la review.

## Étape 1 : Générer le skill

```
/skill-creator
```

```
Crée un skill a11y-review qui :
- prend un composant en argument ($ARGUMENTS)
- lit le composant pour identifier sa nature (image, formulaire, navigation…)
- utilise search-pdf pour extraire les critères RGAA pertinents :
  images → critères 1.x, formulaires → 11.x, couleurs → 3.x, liens → 6.x/12.x, médias → 4.x
- pour chaque critère extrait, évalue le composant : ✅ ❌ ⚠️ N/A
- écrit le rapport dans docs/a11y-report.md :
  tableau critère / statut + liste des fixes concrets avec références de lignes
```

## Étape 2 : Tester sur le composant exemple

```html
<!-- composant-exemple.html -->
<div class="card">
  <img src="banner.jpg">
  <h2>Notre offre</h2>
  <form>
    <input type="email" placeholder="votre@email.com">
    <button onclick="submit()">Envoyer</button>
  </form>
</div>
```

```
/a11y-review composant-exemple.html
```

**Ce composant a au moins 3 violations RGAA.** Le skill doit les trouver.

## Étape 3 : Appliquer sur votre vrai code

Prenez un composant réel de votre projet : formulaire, navigation, carte, page entière.

```
/a11y-review mon-vrai-composant.tsx
```

Le skill écrit le rapport dans `docs/a11y-report.md`. C'est intentionnel : **un fichier markdown est le moyen le plus simple pour qu'un agent communique avec un autre**.

Ouvrez un second terminal et demandez les fixes dans une nouvelle session :

```
@mon-vrai-composant.tsx @docs/a11y-report.md

Apply the required fixes listed in the report.
Minimum changes to pass the failing criteria.
Keep the existing structure and styling.
```

Cet agent n'a pas fait la review — il lit le fichier produit par le premier. Chaque terminal est un agent indépendant, ils ne partagent pas de mémoire, seulement des fichiers. (tmux est pratique pour gérer plusieurs terminaux côte à côte, mais trois onglets font exactement la même chose.)

Re-lancez le skill pour vérifier que les violations sont corrigées.

## Optionnel : MCP Figma

Si le MCP Figma est configuré, l'agent peut lire directement les tokens de design (couleurs, typographie) et vérifier les contrastes WCAG sans screenshot ni copier-coller.

---

# Partie 3 (bonus) : Skill security-review — 20 min

Même pattern. Générez le skill :

```
/skill-creator
```

```
Crée un skill security-review qui :
- prend un fichier de code en argument ($ARGUMENTS)
- utilise search-pdf pour interroger docs/owasp.md
- identifie les catégories de risque pertinentes (auth, SQL, upload, etc.)
- évalue le code contre chaque section extraite
- écrit le rapport dans docs/security-report.md :
  nom de la vulnérabilité, référence dans le doc, ligne, fix concret
```

Téléchargez le PDF de référence :

```bash
curl -L https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Testing_Guide_v4.pdf \
     -o docs/owasp.pdf
```

Lancez-le **dans un terminal séparé, en parallèle d'un `/a11y-review`** sur un autre composant — les deux agents écrivent chacun leur rapport sans se bloquer :

```
/security-review src/api/routes/users.py
```

---

# Pourquoi ce pattern est puissant

- **Pas de RAG, pas de serveur** : pdftotext + ripgrep + l'agent. Ça marche en offline.
- **Les skills sont versionnés** avec le repo : toute l'équipe a les mêmes outils
- **Le contexte est précis** : l'agent reçoit exactement les critères qui s'appliquent, pas 250 pages
- **Extensible** : n'importe quel PDF de référence (OWASP, PCI-DSS, guide interne, doc d'architecture) devient interrogeable via `search-pdf`
- **Composable** : chaque agent écrit un fichier markdown, le suivant le lit — pas d'API, pas de mémoire partagée, juste des fichiers

Pour des corpus vraiment larges (> 500 pages, plusieurs livres), regarder **Docling** (IBM, open source) ou **Qdrant** pour du vrai RAG vectoriel. Mais pour la majorité des cas, ripgrep suffit.

---

# Livrable

À la fin de ce TP :

- [ ] `.codex/commands/search-pdf.md` testé sur le RGAA et sur l'OWASP Testing Guide
- [ ] `.codex/commands/a11y-review.md` qui écrit dans `docs/a11y-report.md`
- [ ] Rapport a11y sur le composant exemple avec au moins 3 violations identifiées
- [ ] Fixes appliqués depuis `docs/a11y-report.md` dans un second terminal et vérifiés avec un 2ème run
- [ ] (Bonus) `.codex/commands/security-review.md` lancé en parallèle dans un terminal séparé
