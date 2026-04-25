Plan: TP 6 — Tests avec l'IA
Context
content/08 - IA pour les Devs/Fondamentaux/6_tp_tests.md is a stub (frontmatter only). The TP teaches writing 1 functional test + 3-4 unit tests for the Comparia app using AI assistance. The clever hook: students let AI generate tests → tests look green → we introduce a deliberate bug → AI-generated tests miss it. Anchored in real HN discourse ("AI Generated Tests as Ceremony", "AI Generated Tests Might Be Lying to You").

File to write
content/08 - IA pour les Devs/Fondamentaux/6_tp_tests.md

TP structure (90 min)
Frontmatter
---
title: "6 - TP Tests"
weight: 6
draft: false
---
Opening
Subtitle: "Des tests qui mordent vraiment" Duration: > ⏱ **90 min** Objectif: Écrire des tests unitaires et fonctionnel pour Comparia avec Claude — et apprendre à distinguer les tests qui valident de ceux qui vérifient.

Test targets (4 functions)
Unit test 1 — convert_range_to_value (warm-up)
File: comparia/backend/llms/utils.py
What it does: Averages a {"min": x, "max": y} dict or returns a scalar unchanged
Test cases to derive:
int input → same int
{"min": 0, "max": 100} → 50.0
{"min": 10, "max": 10} → 10.0
Pedagogical point: AI generates these easily. These are the "ceremony" tests. They prove nothing beyond "the code runs."
Unit test 2 — is_spam (external dependency)
File: comparia/backend/arena/spam_detection.py
What it does: Loads regex patterns from a JSON file, returns True if prompt matches any
Key teaching moment: AI often generates tests that hit the real file on disk (brittle) OR over-mocks and tests nothing. Students must choose a strategy.
Test cases:
Known spam string (injection attempt like <script>)
Clean prompt → False
Empty string → False
Approach: Use unittest.mock.patch to inject a known pattern list, isolating from the JSON file
Unit test 3+4 — fit_bradley_terry (algorithmic properties)
File: comparia/utils/ranking/bradley_terry.py
What it does: MM algorithm — takes list[tuple[str, str, str]] (winner, loser, "_") → returns {model: elo_score}
Test cases (property-based thinking):
A beats B always → Elo(A) > Elo(B)
Transitivity: A>B, B>C → Elo(A) > Elo(B) > Elo(C)
Symmetry: balanced wins/losses → scores converge
Single model → score = 1000.0 (baseline)
Pedagogical point: These are tests that could fail if the algorithm is wrong. This is what tests are for.
Functional test — FastAPI TestClient
File: comparia/backend/main.py (app factory)
Target endpoint: GET /llms (model listing, no auth, no DB/Redis needed)
Setup: from fastapi.testclient import TestClient + mock config to avoid Redis/PG connections
Test:
Response 200
Response is a list
Each item has id and name keys
Pedagogical point: Tests the integration of routing + data loading. Different failure modes than unit tests.
The clever moment: "le test qui mord" (étape 6, ~10 min)
Introduce a deliberate bug in convert_range_to_value:

# bug: min instead of average
return value_or_range["min"]  # was: (min + max) / 2
Students run AI-generated tests → they still pass (because the AI tested the happy path without asserting the math).

Then students write an intentional test:

assert convert_range_to_value({"min": 0, "max": 100}) == 50.0  # now fails
Key takeaway: a test that can't fail isn't a test — it's ceremony.

Resources to include
https://news.ycombinator.com/item?id=46778433 — "AI Generated Tests as Ceremony" (2026)
https://news.ycombinator.com/item?id=46393008 — "AI Generated Tests Might Be Lying to You" (2025)
https://news.ycombinator.com/item?id=39486717 — Meta's TestGen-LLM: 57% build+pass, only 25% increase coverage
TP steps breakdown
Étape	Contenu	Durée
1	Tour du proprio avec l'IA — identifier quoi tester	10 min
2	Unit test convert_range_to_value — warm-up + première génération AI	15 min
3	Unit test is_spam — dépendance externe, stratégie de mock	20 min
4	Unit tests fit_bradley_terry — tester des propriétés, pas juste l'exécution	15 min
5	Test fonctionnel FastAPI TestClient	15 min
6	Le test qui mord — bug délibéré, regard critique	10 min
Format conventions (from existing TPs)
French prose, English code comments
**Bold** for subheadings within steps
> ⏱ blockquote for duration
Observer grids as markdown tables
Code blocks with python / bash language tags
No prerequisites hidden — reference comparia/ relative paths
End with "Ressources" section and link to next module
Verification
After writing:

All referenced file paths exist in comparia/
Python code blocks are syntactically valid
The bug exercise is self-contained (no external setup needed)
HN links match the correct thread IDs