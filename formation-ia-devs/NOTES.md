## Notes 27/04/36

Ajouter lexique

tokenization

schéma TUI <=> provider

schéma tool calling


- rajouter vittascience + gemma scope neuronpedia

- revoir partie Reasoning => confus entre effort max et mode thinking

- feature simple : supprimer tweets, a t il pensé a la protection misclick? est il en train de fix l'env et les tests sur le même commit ?

- pour tp mcp / tools : juste évoquer les autres mais aller direct sur playwright, parler du sandboxing codex qi fait bugger le mcp playwright, parler du mode headless à désac, puis faire se créer un compte à l'ia via playwright, et enfin tester mode vision vs. mode DOM desc.


- pour tp3/4 : faire en sorte que le skill soit plutot slop reviewer que playwright (ou les 2 mais enlever redondant setup mcp playwrith tp3 et tp4)

- rework tp4 skills to cutout slop and say : https://developers.openai.com/codex/skills +  $skill-creator teste l'app dans playwright en mode "headful" et sans pb de sandbox + dis lui de faire un subagent pour voir + dis que dans skills y a aussi des scripts python

Pour jour 2 :
- prompt engineering / mieux écrire, raccourcis à prendre, contexte, blagues sur formulaires de politesse
- profils : dev ou manager ? agents spé / pour gestion de projet, sur mon poste j'ai un agents.md qui décrit mon équipe et ce que je sais faire


- d'autres outils de base à install sur sa machine, génération de docs lecture d'excel / (pandoc)

- skills générique ou spécifique ? guidelines

- feautres : multi repo, flutter + web => TP avec tmux qui parle à des agents ? communication multi agents => tmux, a2a

- c'est quoi le workflow state of the art

- métiers créent jira, revue, pull requests

- connexion avec figma => créer à partir d'une maquette

- sécu : améliorer sécu, pentest TP dans microblog, mais écosystème d'outil pour renforcer la sécu avec PR review, et autres tools

- worfklow deep research, avec couts et markdown en cache

- faire du sysadmin avec ? du frontend !! faire autrement que du bateau => design system + visuel + a11y + pdf du rgaa !!!

- google stich ? flow ? mcp figma

- claude design

- bouquins ux : Qdrant 

- design system boursorama

- ticket ?

- génération de documentation mermaid => skills avec docs/ png (+ ou sans vision)


- /statusline !!!

- dire "!" pour lancer une commande shell
---
# feedbacks jour 2

- https://github.com/microsoft/markitdown
- Ralph loop avec max retry et budget pour chaque tache
<!-- - Avec fallbacks de quel plan de conso (async ou non) -->
- Gandalf + deepseek tiananmen qui s'arrête (as a judge)
- (ou pas) Parler des guardrails : comment bien architecturer un chatbot qui va pas avoir des accès API plus que juste aider a remplir des formulaires exemple
- Montrer system prompts + openrouter observability + karaphty guidelines (https://github.com/forrestchang/andrej-karpathy-skills/blob/main/skills/karpathy-guidelines/SKILL.md) ou plutot  Skills.sh
- pour tp frontend : https://skills.sh/anthropics/skills/frontend-design + https://skills.sh/vercel-labs/agent-skills/web-design-guidelines
- https://supports.uptime-formation.fr/08-ia-pour-les-devs/avanc%C3%A9/00_cours_workflow/ => pas ouf
- parler des Jules.google.com + Ex: https://github.com/besimple-oss/broccoli
- bien s'assurer dans tp a11y que c'est vraiment lighthouse qui a été lancé / que le pdf a bien été lu

- exercise sur optimization

- remplacer "tp" par "exercice"

- comment on contribue / travaille en groupe / mutualiser

---


- WSL2 !!!
- mettre dès le début du J1 le TP AGENTS.md et "comment parler à un llm" 
- pré-installer uvx / npx, voire Docker


- rajout de la commande qui personalise sidebar de codex 

- rajout de la commande qui personalise sidebar de codex 

---
## Workflow for Github PRs

Annoying to setup but in the end you just say @codex wdy think?
ex: https://github.com/ketsapiwiq/pelagica/pull/1

## Laïus on plans: weekly+daily limit+token limit (and cache use or not)

---
## Roast
Problèmes structurels majeurs
1. Deux outils, zéro explication de la relation

La formation mélange OpenCode (configuré sur OpenRouter) et claude (CLI Anthropic). TP1–TP5 utilisent opencode --verbose, mais TP1 étape 7 introduit soudainement claude --verbose et ~/.claude/settings.json. Pour un participant, c'est deux outils différents qui font la même chose — sans jamais expliquer pourquoi il y en a deux, ni quand utiliser lequel.

2. AGENTS.md fait deux fois

TP2 étape 2 : générer un AGENTS.md complet avec l'agent. TP4 étape 4 : "utiliser OpenCode pour générer un AGENTS.md complet". Étape 3 de TP2 et étape 5 de TP4 : "README par dossier" — même instruction, mot pour mot. Si quelqu'un suit les TPs dans l'ordre, il fait deux fois la même chose.

3. Le routing: dans la config OpenCode est fictif

TP5 étape 6 :


routing:
  complex_tasks:
    - "refactor"
    model: anthropic/claude-3.5-sonnet
OpenCode n'a pas cette fonctionnalité. Le participant configure ça, rien ne se passe, il ne sait pas si c'est lui ou l'outil. C'est le genre de chose qui détruit la crédibilité d'une formation.

Problèmes pédagogiques
4. Le timing est impossible

12 modules × (cours + TP à 7-9 étapes) en 2 jours. Sans time estimate par TP, le formateur ne sait pas quoi couper en live. TP4 seul (Makefile + Docker + AGENTS.md + Feature complète + PR review) représente facilement 3h si les gens ne connaissent pas Docker.

5. Comparia est un point de défaillance unique

Si le setup Docker / HuggingFace token prend 45 minutes (ce qui est réaliste en salle avec des proxies d'entreprise, des perms système etc.), toute la journée 1 est en retard. Aucun plan B documenté pour les participants qui ne font pas tourner l'app.

6. Les grilles d'observation ne servent à rien

TP1, TP2, TP3, TP4 ont des tableaux vides type :

Critère	Oui/Non	Notes
Lit les fichiers avant de répondre		
En atelier, personne ne remplit ça. Soit c'est une activité guidée avec un animateur qui demande les réponses à voix haute, soit c'est du remplissage de slide. Le format ne correspond pas à l'usage réel.

7. Le context7 example utilise Bevy

TP3 étape 8 illustre context7 avec "Bevy 0.15 changed the Text2d API". Bevy n'apparaît que dans TP11 (Unknown Tech), module optionnel de veille. Pour quelqu'un qui suit la formation en ordre, la référence est incompréhensible.

Ce qui fonctionne bien
Le "fil rouge Comparia" est une bonne idée — un projet réel, cohérent, qui grandit au fil des TPs.
Le pattern pingre (Gemini gratuit pour planifier, modèle cheap pour implémenter) est original et honnête.
Les scénarios de bugs dans exercises/bug_scenarios/ sont excellents — concrets, basés sur des vraies erreurs d'agents.
TP8 (debugging) est le meilleur TP : des exercices pratiques qui font vraiment comprendre les patterns d'échec.
Ce que je ferais
Par ordre d'impact :

Supprimer ou corriger la fausse config routing: — c'est le seul vrai bug de crédibilité.
Choisir un outil et s'y tenir : soit OpenCode soit Claude Code, pas les deux sans explication.
Fusionner TP2 et TP4 ou supprimer les doublons AGENTS.md/README.
Ajouter une estimation de temps par TP pour que le formateur puisse ajuster en live.
Remplacer les grilles vides par des questions de débrief à poser en groupe.

## https://github.com/Yeachan-Heo/oh-my-codex


## Mention prompt injection and what approach Codex has. 

Web search

Codex ships with a first-party web search tool. For local tasks in the Codex CLI, Codex enables web search by default and serves results from a web search cache. The cache is an OpenAI-maintained index of web results, so cached mode returns pre-indexed results instead of fetching live pages. This reduces exposure to prompt injection from arbitrary live content, but you should still treat web results as untrusted. If you are using --yolo or another full access sandbox setting, web search defaults to live results. To fetch the most recent data, pass --search for a single run or set web_search = "live" in Config basics. You can also set web_search = "disabled" to turn the tool off.

You’ll see web_search items in the transcript or codex exec --json output whenever Codex looks something up.





---
# Old/integrated

Hadrien, [06/04/2026 03:44]
Tip: Ask Claude to create a todo list when working on complex tasks to track progress and remain on track

Hadrien, [06/04/2026 03:50]
Tip: Run /install-github-app to tag @claude right from your Github issues and PRs

Hadrien, [06/04/2026 03:51]
https://code.claude.com/docs/fr/best-practices

Hadrien, [06/04/2026 05:54]
Ajout implems de charbot avec guardrails dans la formation

Hadrien, [06/04/2026 07:30]
ajout des git worktree


EXAMPLE of Lib checking: Bewy with context7 MCP:                                                     
     Bevy 0.15 changed the Text2d API significantly. Let me check the Bevy 0.15 text2d example for the correct API:        
                                                     
     ⚙️ context7_resolve-library-id [libraryName=bevy, query=Text2d text rendering in bevy 0.15]                            

     ⚙️ context7_query-docs [libraryId=/websites/rs_bevy_bevy, query=Text2d spawn text label in 3D world space Bevy 0.      
     15]                                                                     
     Now I understand Bevy 0.15's new Text2d API. Let me fix the code:

Hadrien, [06/04/2026 17:38]
s'appuyer sur les todo

Hadrien, [06/04/2026 20:24]
les faire utiliser github via mcp et via gh

Hadrien, [08/04/2026 13:58]
Le biais sycophantique / flatterie / flagorneur

Hadrien, [08/04/2026 13:59]
Sécurité : overengineering sans bon cadre vs. can-do aptitude béate. Ce qu'il faut : une passe apres coup avec quelqu'un qui ne se laisse pas raconter des salades

Hadrien, [08/04/2026 14:00]
Faire debattre conclusion pure llm par llm avec research ou par llm pas de la meme famille (plus grande / mois flatteuse) / carrément pattern adversariel "roast me"

Hadrien, [09/04/2026 00:18]
y a https://news.ycombinator.com/item?id=47004712

Hadrien, [09/04/2026 00:35]
claude —verbose essentiel

Hadrien, [10/04/2026 02:04]
https://www.reddit.com/r/ExperiencedDevs/comments/1r6olcv/an_ai_ceo_finally_said_something_honest/

Hadrien, [10/04/2026 02:13]
https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/guide/cheatsheet.md#context-management-critical

Hadrien, [10/04/2026 02:14]
Context Management (CRITICAL)
Statusline

Model: Sonnet | Ctx: 89.5k | Cost: $2.11 | Ctx(u): 56.0%

Watch Ctx(u): → >70% = /compact, >85% = /clear

Enhanced statusline (ccstatusline): Add to ~/.claude/settings.json:

{ "statusLine": { "type": "command", "command": "npx -y ccstatusline@latest", "padding": 0 } }

Context Thresholds
Context %   Status   Action
0-50%   Green   Work freely
50-70%   Yellow   Be selective
70-90%   Orange   /compact now
90%+   Red   /clear required
Actions by Symptom
Sign   Action
Short responses   /compact
Frequent forgetting   /clear
>70% context   /compact
Task complete   /clear
Context Recovery Commands
Command   Usage
/compact   Summarize and free context
/clear   Fresh start
/rewind   Undo recent changes
claude -c   Resume last session (CLI flag)
claude -r <id>   Resume specific session (CLI flag)

Hadrien, [10/04/2026 02:16]
https://github.com/anthropics/skills/tree/main/skills

Hadrien, [10/04/2026 02:16]
https://github.com/anthropics/claude-code-security-review

Hadrien, [10/04/2026 02:18]
((https://github.com/rtk-ai/rtk))

Hadrien, [10/04/2026 02:22]
dire Plan then Build then Test then Plan more modestly lol

Hadrien, [10/04/2026 02:22]
https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/quiz/questions/01-quick-start.yaml
https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/quiz/questions/05-skills.yaml
https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/quiz/questions/04-agents.yaml
https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/quiz/questions/03-memory-settings.yaml
call me daddy => sanity check : https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/quiz/questions/02-core-concepts.yaml

Hadrien, [10/04/2026 02:30]
- "secrets", "API keys", "credentials" → mcp_secrets_management (secrets handling)
  - "security", "sandbox", "isolation" → sandbox_native_guide or security_hardening
  - "permission", "allow", "deny" → permission_modes
  - "memory", "persist", "session" → memory_files
  - "template", "structure", "format" → skill_template
  - "validation", "checklist", "deploy" → agent_validation_checklist
  - "plan", "pipeline" → plan_pipeline_workflow (Plan-Validate-Execute)

Hadrien, [10/04/2026 02:31]
https://www.anthropic.com/research/AI-fluency-index
les gens se shootent aux artefacts, le modèle de chat back and forth garde un peu + d'esprit critique que "et rajotue ça aussi" => fuite en avant car effet wow

Hadrien, [10/04/2026 02:37]
https://resources.anthropic.com/hubfs/2026%20Agentic%20Coding%20Trends%20Report.pdf

Hadrien, [12/04/2026 15:40]
- dire sue les closed source models enshittify without u knowing

Hadrien, [17/04/2026 02:11]
https://github.com/AgentSeal/codeburn

Hadrien, [18/04/2026 06:51]
if you fix it yourself, Claude doesn't learn the context. Either tell it what you did or let Claude correct its own mistakes so it builds a better mental model of my codebase.

Run /simplify before doing a review. Claude tends to over-engineer. That's why I let it clean up first.

Do a retro at the end of each session.=> upgrade agents.md

<!-- Hadrien, [18/04/2026 14:12]
comparia: 
- faire marcher l'app
- faire créer un compte hugging face via playwright ? voir que bof, mais réfléchir au moyen d'instrumentalisation (visuel, playwright, api), peut etre faire une recherche sur alternatives pour mettre un token gratuit et observer mode recherche
-- faire foncitonenr app simplifiée
- créer un nouve usage funky, e.g. spécilisé en conseils en médecine super nazes -->