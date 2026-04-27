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

## Étape 2 : Écrire le skill

```markdown
# .claude/commands/a11y-review.md

Review the accessibility (RGAA 4.1) of the component passed as argument: $ARGUMENTS

## Steps

1. If `docs/rgaa.md` does not exist:
   - Run `pdftotext docs/RGAA_4.1.pdf docs/rgaa.md`
   - If pdftotext is not available, run `pip install pdfminer.six` then
     `python3 -m pdfminer.high_level docs/RGAA_4.1.pdf > docs/rgaa.md`

2. Read the component to understand its nature (image, form, navigation, interactive element, etc.)

3. Identify which RGAA criteria apply. Common mappings:
   - Images → Critères 1.x
   - Forms / inputs → Critères 11.x
   - Colors / contrast → Critères 3.x
   - Navigation / links → Critères 6.x, 12.x
   - Multimedia → Critères 4.x

4. For each applicable criterion family, extract the relevant text:
   `rg "critère [NUMBER]" docs/rgaa.md -A 20 -i`

5. For each criterion extracted, evaluate the component:
   - ✅ PASS — criterion is met
   - ❌ FAIL — criterion is not met (explain what's missing)
   - ⚠️ PARTIAL — partially met
   - N/A — criterion does not apply

6. Output a structured report:

## A11y Review — [Component name]

### Summary
[One sentence on overall state]

### Criteria evaluated

| Critère | Intitulé | Status | Note |
|---------|----------|--------|------|
| 1.1 | Chaque image décorative... | ✅ | alt="" présent |
| 11.1 | Chaque champ... | ❌ | Label manquant sur #email |

### Required fixes
[Concrete list of what needs to change, with line references]

### Code suggestions
[Modified code snippets for the failing criteria]
```

## Étape 3 : Préparer le PDF

Mettez le PDF RGAA dans `docs/RGAA_4.1.pdf` (fourni ou téléchargeable sur accessibilite.numerique.gouv.fr).

```bash
mkdir -p docs
# Placer RGAA_4.1.pdf dans docs/
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

## Si vous avez des tokens de design (Figma ou design system)

Si votre équipe a un design system avec des tokens de couleur, vérifiez le contraste :

```
@design-tokens.json @mon-vrai-composant.tsx

Check if the colors used in this component meet WCAG AA contrast ratio (4.5:1 for text).
Use the color values from the design tokens.
Flag any combination that fails.
```

Le MCP Figma (si configuré) expose les tokens de design en JSON — couleurs, espacements, typographie. L'agent peut les lire directement et vérifier la conformité sans screenshot.

---

# Bonus : même pattern pour la sécurité

Le même principe fonctionne avec n'importe quel référentiel documentaire.

```markdown
# .claude/commands/security-review.md

Review the security of the code passed as argument: $ARGUMENTS

1. If `docs/owasp.md` does not exist:
   Run `pdftotext docs/OWASP_Testing_Guide.pdf docs/owasp.md`

2. Identify the risk categories relevant to this code (auth, SQL, file upload, etc.)

3. Extract the relevant OWASP sections:
   `rg "SQL injection|input validation" docs/owasp.md -A 15 -i`

4. Review the code against each applicable guideline.

5. Output: vulnerability name, OWASP reference, line number, concrete fix.
```

```bash
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
