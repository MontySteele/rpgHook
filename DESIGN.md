Executive Overview
Build a voice- and chat-driven automation layer (“VoiceBot”) for Pathfinder 2e on Foundry VTT.
VoiceBot converts natural-language intents (Rhasspy or /rollbot … chat commands) into Foundry macro calls that:

import Pathbuilder characters,

run actions/feats/spells with correct math,

apply damage, saves, conditions, initiative changes, and auras,

whisper audit logs to the GM.

Most PF2e rules logic already lives in community modules; VoiceBot is a thin orchestrator glued together with Socket messages and a few Rule Elements.
foundryvtt.com
foundryvtt.com
foundryvtt.com

1 Goals
Functional
One-click Pathbuilder → Foundry sheet per player.
foundryvtt.com

Spoken or typed commands (“Eclipse Burst on all targets”, “Double Slice goblin A, goblin B”) trigger the correct rolls and downstream effects.
rhasspy.readthedocs.io
reddit.com

Edge-case support up to level 20 (evasion, MAP, persistent damage, auras, regeneration, polymorph).
foundryvtt.com
foundryvtt.com
github.com

Non-Functional
Runs fully offline; only outbound traffic is optional Discord webhooks.

Module-safe upgrades: pin versions, validate on PF2e system releases.
foundryvtt.com

2 Tech Stack
Layer	Tool / Module	Purpose
STT / Intent	Rhasspy	Offline voice → JSON intent.
rhasspy.readthedocs.io
Orchestrator	Node 18 + TypeScript	Bridges Rhasspy ↔ Foundry via WebSocket/REST.
foundryvtt.com
reddit.com
Foundry Core	PF2e system 5.x	All rolls, saves, conditions.
foundryvtt.com
Automation	PF2e Automations	Auto-apply damage, crit special, save outcomes.
foundryvtt.com
Macro Suite	xdy-pf2e-Workbench	Ready-made macros (Double Slice, Twin Takedown, etc.).
foundryvtt.com
Import	Pathmuncher	Pathbuilder JSON import.
foundryvtt.com
Auras	Active Auras	Radius-based buffs/debuffs.
foundryvtt.com

3 Component Design
3.1 Rhasspy & Chat Front-end
Intent grammar lives in /voicebot/grammar.ini; emits {user, actor, action, targets[]}.

CLI fallback: players type /rollbot {json} into Foundry chat.

3.2 Node Orchestrator
Listens on localhost:1880/intent (HTTP) and mqtt://rhasspy.

Translates intent → Socket payload:

ts
Copy
Edit
game.socket.emit("module.voicebot", { actorId, itemSlug, targets });
Optionally calls Foundry REST API for out-of-game tasks (e.g., nightly recap export).
foundryvtt.com

3.3 Foundry VoiceBot Module
Registers Hooks.on("socketlib.ready") listener.

Looks up actor.items.get(itemSlug) and executes:

item.toMessage() for spells/feats

game.pf2e.actions.doubleSlice({...}) for compound strikes
reddit.com

If no targets, opens crosshair dialog (canvas.targetCrosshairs).

After roll resolves, whispers audit line + triggers optional Discord webhook.

4 Edge-Case Handling Matrix
Scenario	Native?	Module	Extra Work
Eclipse Burst + Evasion	✔ PF2e core (template, evasion rule element)
github.com
✔ Automations	none
Double Slice agile + non-agile MAP	✔ Workbench macro / rollStrike rules
reddit.com
—	confirm order dialog
Persistent 2d6 fire tick	✔ PF2e ≥ 4.6 built-in
foundryvtt.com
—	none
Regen 15 (fire/acid off-switch)	✔ Rule Element JSON (Regen)
github.com
—	author RE once
Bless aura 20 ft	—	Active Auras
foundryvtt.com
voice alias to resize
Polymorph (Animal Shape)	✔ Form items override stats ⟶ toggle	—	only for custom forms

5 Socket & REST Contracts
5.1 Socket Payload (module.voicebot)
ts
Copy
Edit
interface IntentPayload {
  actorId: string;          // Foundry Actor UUID
  itemSlug: string;         // "fireball", "double-slice"
  targets: string[];        // Token IDs
  meta?: Record<string, any>;
}
5.2 REST Endpoints (optional)
Path	Method	Purpose
/api/actors/:id/import/pathbuilder	POST JSON	Import PB JSON via Pathmuncher CLI.
/api/session/log	GET	Return last n chat messages for recap export.

6 Deployment Steps
Install Foundry 13.x and PF2e system 5.x.

Add modules in this order: Pathmuncher → PF2e Automations → xdy-pf2e-Workbench → Active Auras → VoiceBot (custom).

Pin versions in world.json: "requires": { "pf2e": "5.+" }.

Run pnpm i && pnpm dev in /voicebot to start Node bridge.

Import heroes via Pathmuncher; verify AoE template + auto-saves.

Train Rhasspy sentences; speak “Fireball at skeletons” to test end-to-end flow.


Risks & Mitigations
Module churn – lock versions; run CI against PF2e nightly builds.

Latency on voice commands – cache speech model; keep Node and Foundry on same LAN.

Rule exceptions (homebrew feats) – expose /voicebot/registerAction so Codex can generate new macros on demand.
