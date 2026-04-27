---
title: "11 - TP Frontend & Skills documentaires"
weight: 2036
---

## _Créer des skills qui savent lire tes docs_

> ⏱ **1h30**

---

# Principe

Un skill Claude Code est un fichier markdown dans `.claude/commands/`. Quand vous tapez `/nom-du-skill`, l'agent exécute les instructions du fichier.

Ce qui change ici : on va créer des skills qui **donnent à l'agent les instructions pour aller chercher le contexte lui-même** dans des documents locaux (PDF de référence, guides internes). L'agent fait le travail de conversion et d'extraction — pas vous.

---

# Partie 1 : Skill de review a11y RGAA — 45 min

## Contexte

Le RGAA (Référentiel Général d'Amélioration de l'Accessibilité) est le standard d'accessibilité numérique en France. C'est un PDF de ~250 pages. Les équipes front en ont besoin mais personne ne le lit en entier.

**L'idée :** un skill qui, quand vous lui passez un composant, sait aller extraire les critères RGAA pertinents et faire la review.

## Étape 1 : Structure du skill

```bash
mkdir -p .claude/commands
touch .claude/commands/a11y-review.md
```

## Étape 2 : Générer le skill avec $skill-creator

Le skill qui sait lire un PDF et en extraire des critères — c'est exactement le genre de chose que l'IA génère mieux que vous ne l'écrivez à la main.

```bash
/skill-creator
```

Décrivez ce que vous voulez en langage naturel :

```
Crée un skill a11y-review qui :
- prend un composant en argument ($ARGUMENTS)
- si docs/rgaa.md n'existe pas, convertit docs/RGAA_4.1.pdf avec pdftotext
  (fallback : pdfminer.six si pdftotext absent)
- identifie la nature du composant (image, formulaire, navigation…)
- extrait les critères RGAA pertinents avec ripgrep (images → 1.x, forms → 11.x,
  couleurs → 3.x, liens → 6.x/12.x, médias → 4.x)
- produit un rapport avec tableau critère / statut (✅ ❌ ⚠️ N/A) et les fixes concrets
```

`$skill-creator` génère le fichier `.claude/commands/a11y-review.md` directement. Relisez-le, ajustez si besoin.

## Étape 3 : Télécharger le PDF RGAA

```bash
mkdir -p docs
curl -L https://accessibilite.numerique.gouv.fr/doc/RGAA-v4.1.pdf -o docs/RGAA_4.1.pdf
```

## Étape 4 : Tester le skill

Avec un de vos propres composants, ou ce composant exemple :

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

```bash
# Dans Claude Code :
/a11y-review composant-exemple.html
```

L'agent va :
1. Convertir le PDF RGAA si nécessaire
2. Lire votre composant
3. Extraire les critères pertinents (images → 1.x, form → 11.x)
4. Générer le rapport avec les fixes concrets

**Ce composant a au moins 3 violations RGAA.** Le skill doit les trouver.

---

# Partie 2 : Appliquer sur votre vrai code — 45 min

## Amener votre composant du boulot

Prenez un composant réel de votre projet : formulaire, navigation, carte, page entière. Copiez-le dans un fichier local.

```bash
/a11y-review mon-vrai-composant.tsx
```

## Itérer avec l'agent

Une fois le rapport généré, demandez les fixes :

```
@mon-vrai-composant.tsx

Apply the required fixes from the a11y review. 
Make the minimum changes to pass the failing criteria.
Keep the existing structure and styling.
```

Vérifier les changements :
```bash
git diff mon-vrai-composant.tsx
```

Re-lancer le skill pour vérifier que les violations sont corrigées.

## Optionnel : MCP Figma

Si le MCP Figma est configuré dans votre projet, l'agent peut lire directement les tokens de design (couleurs, typographie) et vérifier les contrastes WCAG sans screenshot ni copier-coller.

---

# Bonus : même pattern pour la sécurité

Le même principe fonctionne avec n'importe quel référentiel documentaire. Générez le skill de la même façon :

```bash
/skill-creator
```

```
Crée un skill security-review qui :
- prend un fichier de code en argument ($ARGUMENTS)
- si docs/security.md n'existe pas, convertit le PDF de référence avec pdftotext
- identifie les catégories de risque pertinentes (auth, SQL, upload, etc.)
- extrait les sections applicables avec ripgrep
- produit un rapport : nom de la vulnérabilité, référence dans le doc, ligne, fix concret
```

Puis testez-le sur un vrai bouquin de sécurité en PDF (OWASP Testing Guide, The Web Application Hacker's Handbook, etc.) :

```bash
# Télécharger un PDF de référence, ex. l'OWASP Testing Guide
curl -L https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Testing_Guide_v4.pdf \
     -o docs/owasp.pdf

/security-review src/api/routes/users.py
```

---

# Pourquoi ce pattern est puissant

- **Pas de RAG, pas de serveur** : c'est du shell. pdftotext + ripgrep + l'agent. Ça marche en offline.
- **Le skill est versionné** avec le repo : toute l'équipe a le même outil
- **Le contexte est précis** : l'agent reçoit exactement les critères qui s'appliquent, pas 250 pages
- **Extensible** : n'importe quel PDF de référence (OWASP, PCI-DSS, guide interne, doc d'architecture) devient interrogeable

Pour des corpus vraiment larges (> 500 pages, plusieurs livres), regarder **Docling** (IBM, open source) ou **Qdrant** pour du vrai RAG vectoriel. Mais pour la majorité des cas, ripgrep suffit.

---

# Livrable

À la fin de ce TP :

- [ ] `.claude/commands/a11y-review.md` créé et fonctionnel
- [ ] `docs/rgaa.md` généré par le skill (pas manuellement)
- [ ] Rapport a11y sur le composant exemple avec au moins 3 violations identifiées
- [ ] Fixes appliqués et vérifiés avec un 2ème run du skill
- [ ] (Bonus) `.claude/commands/security-review.md` créé sur le même pattern
