https://github.com/matt1398/claude-devtools:  The missing DevTools for Claude Code — inspect session logs, tool calls, token usage, subagents, and context window in a visual UI.
https://github.com/rtk-ai/rtk:  CLI proxy that reduces LLM token consumption by 60-90% on common dev commands.

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