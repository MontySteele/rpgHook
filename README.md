Project Summary
VoiceBot is a two-part automation layer for Foundry VTT:

a Node + TypeScript bridge that turns Rhasspy voice/JSON intents into socket messages, and

a lightweight Foundry module that receives those messages and triggers PF2e rolls, macros, and imports.

By leaning on community modules—Pathmuncher for character import, PF2e Automations for save/damage handling, xdy-pf2e-Workbench for macro shortcuts, Active Auras for radius effects, and SocketLib for inter-client RPC—VoiceBot delivers near hands-free high-level Pathfinder 2e play. 
foundryvtt.com
github.com
github.com
github.com
github.com

Features
Category	What You Get
Character Import	One-line or voice command to pull a Pathbuilder JSON/code into a ready-to-play Foundry sheet via Pathmuncher. 
foundryvtt.com
Action & Spell Rolls	Voice//rollbot chat triggers items; PF2e Automations applies damage, conditions, crit specialization, etc. 
github.com
Edge-Case Support	MAP math (e.g., Double Slice), AoE templates with Evasion, persistent dmg tick-down, regeneration shut-off, aura propagation. Handled by Workbench, core PF2e, and Active Auras. 
github.com
github.com
Audit Logs	GM-only whispers summarise rolls, post-mitigation damage, and new conditions after every action.
Offline Voice	Rhasspy keeps the entire pipeline local; no cloud STT needed. 
rhasspy.readthedocs.io
Optional REST Hooks	Foundry-REST-API provides endpoints for nightly chat exports or out-of-game tooling. 
github.com

Prerequisites
Foundry VTT ≥ v13 with the Pathfinder 2e system ≥ v5 installed. 
github.com

Community modules (minimum versions are pinned in package-lock.json):

Pathmuncher, PF2e Automations, xdy-pf2e-Workbench, Active Auras, SocketLib, Foundry-REST-API. 
foundryvtt.com
github.com
github.com
github.com
github.com
github.com

Node 18+ (tested with 18.19) and pnpm for the bridge.

A Rhasspy 2.5 instance (optional if you only want chat commands). 
rhasspy.readthedocs.io

Quick-Start (Local LAN)
Clone & Install

bash
Copy
Edit
git clone https://github.com/yourname/voicebot-fvtt.git
cd voicebot-fvtt
pnpm install
Foundry Setup

Launch Foundry, create a VoiceBot Dev world, and install the prerequisite modules from the Add-on Browser.

In Settings ▸ Game Settings ▸ Module Settings ▸ VoiceBot, tick Enable external socket (requires SocketLib).

Bridge .env
Copy .env.sample ➜ .env and set:

ini
Copy
Edit
FOUNDRY_URL=http://localhost:30000
FOUNDRY_API_KEY=<from REST-API module>
RHASSPY_URL=http://localhost:12101     # optional
Run Bridge

bash
Copy
Edit
pnpm dev    # hot-reload w/ ts-node-dev
Try It
In Foundry chat:

bash
Copy
Edit
/rollbot {"actor":"Arthur","action":"eclipse-burst"}
All targeted tokens should auto-roll Reflex saves; allies with Evasion will get upgraded results via core PF2e logic. 
github.com
github.com

Repository Layout
bash
Copy
Edit
voicebot-fvtt/
├── bridge/              # Node/TS source
│   ├── intents/         # Rhasspy grammar & parsers
│   ├── routes/          # REST endpoints (import, recap)
│   └── tests/
├── foundry/voicebot/    # Foundry module
│   ├── module.json
│   ├── main.js
│   └── utils.js
├── DESIGN.md            # High-level architecture
├── TASKS.md             # Work-breakdown for Codex
└── README.md            # (this file)
Common Voice / Chat Phrases
Example Utterance	Parsed Intent
“Fireball those three”	{actor: "Wizard", itemSlug: "fireball", targets:[...]}
“Double Slice goblin A and B”	Prompts weapon order, rolls strikes with correct MAP.
“Import character code 3E8DF7”	Pulls Pathbuilder data via Pathmuncher and refreshes sheet.

Development Scripts
Command	Purpose
pnpm dev	Hot-reload bridge on code change.
pnpm test	Jest unit tests for parsers and utilities.
pnpm lint	ESLint + Prettier.
docker compose up	Spin up headless Foundry + Rhasspy + bridge for CI.

Contributing
Fork ➜ feature branch ➜ PR.

New PF2e edge cases? Add a failing Jest spec under bridge/tests/edgeCases/—CI will protect regressions.

Keep module versions in world.json aligned with Foundry+PF2e release notes to avoid breaking API changes. 
github.com
github.com

License
MIT for original VoiceBot code. Third-party modules retain their original licenses—see /THIRD_PARTY.md for details. 
github.com
github.com
