---
title: "10 - TP Bot de Review de PR"
weight: 2035
draft: true
---

## _Un agent qui commente vos Pull Requests_

> ⏱ **1h30**

---

# Objectif

Mettre en place un bot GitHub Actions qui analyse chaque PR avec un agent IA, identifie les problèmes, et poste un commentaire structuré — sans intervention humaine.

## Pourquoi

- Repérer les oublis évidents (gestion d'erreurs manquante, TODO laissés)
- Uniformiser le niveau d'attention sur toutes les PRs
- Donner du feedback instantané avant qu'un humain regarde

---

# Prérequis

- Un repo GitHub avec les Actions activées
- Une clé API OpenRouter
- `gh` CLI installé localement pour tester

---

# Partie 1 : Configurer les secrets — 10 min

## Étape 1 : Ajouter la clé API au repo

Dans **Settings → Secrets and variables → Actions**, ajouter :

| Secret | Valeur |
|--------|--------|
| `OPENAI_API_KEY` | Votre clé  |

## Étape 2 : Vérifier les permissions du workflow

Dans **Settings → Actions → General → Workflow permissions**, activer :

- `Read and write permissions`

Cela permet au workflow de poster des commentaires via `GITHUB_TOKEN`.

---

# Partie 2 : Le workflow GitHub Actions — 25 min

## Étape 1 : Créer le fichier

```bash
mkdir -p .github/workflows
```

## Étape 2 : Écrire le workflow

```yaml
# .github/workflows/ai-review.yml
name: AI PR Review

on:
  pull_request:
    types: [opened]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install opencode
        run: npm install -g opencode-ai

      - name: Setup opencode
      #  TODO: Opencode config

      - name: Run AI review
        run: |
          PROMPT=$(cat .github/review-prompt.md)
          opencode -p "$PROMPT"
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}

      - name: Post review comment
        run: |
          gh pr comment ${{ github.event.number }} \
            --body "$(cat review.md)"
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

# Partie 3 : Le prompt de review — 20 min

## Étape 1 : Créer le fichier de prompt

```bash
touch .github/review-prompt.md
```

```markdown
You are a senior software engineer reviewing a pull request. Write your review in `review.md`.

The git diff follows this message.

Return a markdown review with these sections:

## Summary
One paragraph describing what this PR does.

## Issues
Bullet list of bugs, security risks, or logic errors. Be specific: include file names and line numbers. Write "None found" if the diff looks clean.

## Suggestions
At most 5 bullet points for improvements (naming, missing tests, edge cases not handled).

## Verdict
Exactly one of: ✅ Ready to merge | ⚠️ Minor issues | ❌ Needs rework

Be concise. If the diff is small and clean, say so and move on.
```

## Étape 2 : Adapter le prompt à votre stack


```markdown
# Exemple pour un projet Django
Also check for:
- ORM queries inside loops (N+1)
- Model changes without migration
- Missing `select_related` or `prefetch_related`
```
<!-- 
---
## Étape 1 : Demander à l'agent de générer des suggestions

Ajoutez cette section à votre prompt :

```markdown
When you spot a fixable issue, include a GitHub suggestion block after the bullet point:

Example:
- `utils.py:42` — valeur de retour non vérifiée

```suggestion
return value if value is not None else default
```

Only add suggestions when the fix is straightforward and confined to a single line.
```

## Étape 2 : Choisir entre `comment` et `review`

```bash
# Commenter sans verdict formel
gh pr comment $PR_NUMBER --body "$(cat review.md)"

# Poster un review avec verdict (approve / request-changes / comment)
gh pr review $PR_NUMBER --request-changes --body "$(cat review.md)"
gh pr review $PR_NUMBER --approve --body "$(cat review.md)"
```

**Limite actuelle :** les suggestions avec coordonnées précises (fichier + numéro de ligne exact) nécessitent l'API REST GitHub, pas le `gh` CLI. Pour des suggestions inline au niveau de la ligne, utiliser `actions/github-script`. -->

---

# Partie 5 : Tester — 15 min

## Étape 1 : Ouvrir une PR de test

```bash
git checkout -b test/ai-review-bot
echo "# test" >> README.md
git add README.md
git commit -m "test: déclencher le bot de review"
git push origin test/ai-review-bot
gh pr create --title "Test bot de review IA" --body "Vérification du workflow"
```

## Étape 2 : Suivre l'exécution

```bash
gh run watch
```

## Étape 3 : Vérifier le commentaire

```bash
gh pr view --comments
```

---

# Améliorations
- Bot commente à chaque push: Supprimer le commentaire précédent avant d'en poster un nouveau |
- Filtrer les PRs draft : ajouter `if: github.event.pull_request.draft == false` |

<!-- **Supprimer le commentaire précédent du bot :** -->
<!-- 
```yaml
- name: Supprimer l'ancien commentaire bot
  run: |
    COMMENT_ID=$(gh pr view ${{ github.event.number }} \
      --json comments \
      --jq '.comments[] | select(.author.login == "github-actions[bot]") | .databaseId' \
      | tail -1)
    if [ -n "$COMMENT_ID" ]; then
      gh api repos/${{ github.repository }}/issues/comments/$COMMENT_ID -X DELETE
    fi
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
``` -->

<!-- ---

# Livrable

À la fin de ce TP :

- [ ] Secret `OPENAI_API_KEY` configuré dans le repo
- [ ] `.github/workflows/ai-review.yml` commité et fonctionnel
- [ ] `.github/review-prompt.md` commité avec un prompt adapté au projet
- [ ] Un commentaire bot visible sur une vraie PR (`gh pr view --comments`) -->
