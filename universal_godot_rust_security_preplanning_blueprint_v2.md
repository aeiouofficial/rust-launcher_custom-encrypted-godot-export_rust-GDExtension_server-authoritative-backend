Universal Godot + Rust Security Architecture Pre-Planning Blueprint v2.0

Version: 2.0
Target: Any Godot game — genre-agnostic — reusable — agent-ready
Updated: May 2026
Godot stable: 4.6.3
gdext: March 2026 update
Primary architecture: Rust launcher + custom encrypted Godot 4.6 export + Rust GDExtension (gdext) + server-authoritative Rust backend

Important security stance: This is not impossible-to-crack DRM. This is a practical production architecture that makes the client untrusted, moves valuable authority server-side, and adds enough packaging, validation, and tamper evidence to stop casual Godot decompiler users, noob crackers, basic PCK extractors, and simple asset rippers. Godot 4.6 adds LibGodot (embed Godot in Rust binary), native Delta PCK patching, Jolt physics as default, and GDExtension nullability — all integrated in this v2.0 update.

---

PART 1: FOUNDATION AND ARCHITECTURE

---

1. Executive Summary

The goal is to protect any Godot game without pretending that local client security can ever be perfect. A player-controlled machine is hostile by default. Anything shipped to the player can eventually be inspected, dumped, patched, or emulated by a determined attacker. Therefore, the correct architecture is not to hide everything inside a Rust shell. The correct architecture is to make the Godot client untrusted, move important authority to the backend, encrypt and strip the shipped build enough to block casual ripping, and use Rust for the launcher, sensitive client-side bridge code, patching, build verification, and server logic.

The recommended architecture:

Rust launcher starts first.
Launcher verifies the install.
Launcher checks a signed file manifest.
Launcher repairs or rejects modified files.
Launcher authenticates the player.
Backend issues a short-lived single-use launch token.
Launcher starts the Godot export with controlled arguments.
Godot game loads a Rust GDExtension (gdext).
Rust GDExtension validates the launch/session state.
Backend remains the source of truth for important gameplay state.
Godot client only displays, predicts, animates, and requests actions.
Backend decides whether valuable state changes actually happen.

Godot 4.6 adds LibGodot, which allows Godot to run as an embedded library inside a Rust binary, removing the separate Game.exe entirely for even tighter launcher coupling.
Godot 4.6 adds native Delta PCK patching, allowing patch files to include only changed portions of resources — dramatically reducing update bandwidth.

This structure applies to: singleplayer with online features, co-op PvE, multiplayer, MMO-style, competitive PvP, RPG, survival, extraction, browser demo, and commercial Godot desktop games.

---

2. Source-Aware Technical Reality Check

Godot 4.6 supports encrypted PCK export using a 256-bit AES key. This protects scenes, scripts, and resources from being stored in plain text and from being easily ripped by basic tools, but the encryption key must still exist in the binary. PCK encryption is useful resistance against casual ripping, not absolute protection.

Godot GDExtension is the official native extension mechanism for Godot 4. It lets a Godot project load native shared libraries and call into native code at runtime.

The godot-rust/gdext project is the practical Rust binding layer for Godot 4. As of March 2026, gdext supports three safeguard tiers (Strict for debug, Performance for release) and AnyArray/AnyDictionary types for better engine interop. Godot 4.6 adds nullability support to the GDExtension API.

Godot 4.6 specific additions affecting this architecture:
LibGodot: Godot can now run as a library embedded inside a Rust application. Instead of Rust launcher spawning a separate Game.exe process, the Rust binary can contain Godot itself.
Delta PCK patching: The export system can generate binary delta patches so only changed resource portions are downloaded during updates.
Jolt physics as default for new 3D projects: Affects server-side physics validation planning.
D3D12 as default renderer on Windows for new projects.
GDExtension nullability: Rust GDExtension can express nullable return types cleanly.

Godot Web export can publish games to browsers through WebAssembly and WebGL, but web clients must be considered fully inspectable. Browser builds cannot receive the same protection assumptions as desktop builds. Any serious gameplay authority must still be server-side.

Authoritative planning rule:
Encrypted PCK protects files from casual ripping.
Custom export template makes the encrypted export usable.
Rust GDExtension moves selected sensitive client helpers into native code.
Rust launcher controls normal entry, patching, manifest checks, and launch tokens.
LibGodot (4.6 optional) can eliminate the separate game executable entirely.
Delta PCK (4.6 native) reduces patch bandwidth dramatically.
Rust backend owns important gameplay truth.
Browser build is convenience and reach, not strong anti-cheat.

---

3. Security Mission Statement

The mission is NOT:
Make the Godot client impossible to reverse engineer.
Build malware-like anti-cheat.
Hide everything in Rust.
Prevent all piracy forever.
Trust local files because the launcher checked them once.
Trust command-line arguments because the launcher passed them.
Trust the browser client.
Claim fake guarantees to players or investors.

The mission IS:
Make the client untrusted.
Move important validation server-side.
Encrypt and strip shipped Godot data enough to stop casual ripping.
Use Rust where Rust gives meaningful security, performance, or tooling value.
Use signed build manifests and short-lived single-use tokens.
Detect and reject basic tampering.
Make direct Game.exe launching useless for online/protected mode.
Reject impossible gameplay requests server-side.
Keep security behavior trustworthy, explainable, and non-invasive.
Evaluate LibGodot for tighter launcher/game coupling when warranted.
Use Delta PCK for bandwidth-efficient updates.

---

4. Target Attacker Model

This architecture stops or frustrates:
Godot decompiler users.
Basic PCK extractors.
Users trying to open exported scenes/scripts in plain text.
Copy-paste crackers.
Casual file replacers.
Simple save editors.
Basic packet replay users.
Basic asset rippers.
Players trying to launch Game.exe directly without launcher flow.
Players trying to edit local gold, XP, item amounts, cooldowns, quest state, or premium unlocks.
Players trying to call client functions to force rewards.
Players trying simple speed, teleport, or cooldown hacks.
Players using outdated or modified builds.

This architecture does not claim to fully stop:
Professional reverse engineers.
Memory dumpers.
Runtime debuggers.
Custom emulator writers.
Skilled protocol reversers.
People willing to spend weeks attacking the client.
Kernel-level attackers.
Attackers with admin-level control over their machine.

Correct response to advanced attackers:
Harden server authority.
Improve telemetry.
Improve protocol validation.
Improve rate limits.
Improve signed manifests and build rollout.
Improve content watermarking and abuse response.
Do NOT escalate into shady invasive anti-cheat.

---

5. Universal Architecture Overview

Component: Rust Launcher
Layer: Entry Gate
Responsibilities: Login, patch, manifest verify, hash check, file repair, token request, process spawn, crash upload, telemetry upload
Trust Level: Controls entry — not trusted after handoff

Component: Custom Godot 4.6 Export
Layer: Client Application
Responsibilities: Encrypted PCK, stripped binary, no debug, no dev assets, GDExtension loads on startup
Trust Level: Untrusted — inspectable by determined attacker

Component: Rust GDExtension (gdext)
Layer: Native Bridge
Responsibilities: Handshake, session validation, packet signing, movement prechecks, save validation, telemetry packing
Trust Level: Untrusted — native code is still reversible

Component: Rust Backend
Layer: Authority
Responsibilities: Auth, session, build validation, inventory, economy, quest, combat, loot, telemetry, cheat flags, audit log
Trust Level: TRUSTED — the real protection

Component: LibGodot (Godot 4.6 optional)
Layer: Launcher+Client Combined
Responsibilities: Embed Godot inside Rust binary — removes separate Game.exe — tighter coupling
Trust Level: Controls entry — harder to bypass than process-spawn model

Component: Browser/Web Build
Layer: Demo Layer
Responsibilities: Account login, server-authoritative gameplay, no trusted secrets
Trust Level: Fully Inspectable — always treat as hostile

---

6. Core Security Rule

The launcher protects entry.
The encrypted PCK protects against casual ripping.
The Rust extension protects selected local glue and helper logic.
The backend protects actual game state.
The client is never trusted.
The server decides important outcomes.

The iron rule: If a state change matters to progression, economy, combat, entitlement, ranking, matchmaking, achievements, or trading — it MUST be validated or owned server-side.

---

7. Game-Type Adaptation Matrix

Singleplayer offline game:
Use encrypted PCK.
Use custom export template.
Use stripped release binary.
Use optional Rust GDExtension for save validation and local anti-tamper.
No mandatory always-online backend unless the product design requires it.
Do not pretend offline saves cannot be modified.
Treat local saves as convenience, not competitive truth.

Singleplayer with online account or progression:
Use launcher.
Use encrypted PCK.
Use Rust GDExtension.
Use backend-owned cloud save checkpoints for valuable state.
Use signed local save files only as cache.
Never trust local premium unlocks.

Co-op PvE:
Use server authority for inventory, XP, drops, quest state, cooldowns, and trade.
Allow client prediction for movement and animations.
Validate combat and loot on server.

Multiplayer PvP:
Use stricter server validation.
Use higher tick and reconciliation requirements.
Do not trust client hit results.
Do not trust client movement beyond tolerance.
Add telemetry and replay sampling.

MMO-style RPG or Extraction game:
Use full stack.
Backend owns characters, items, gold, XP, quests, loot, vendors, trading, mail, auction, cooldowns, and movement sanity.
Client becomes renderer, input collector, UI layer, and prediction layer.

Idle or Casual with Economy:
Backend owns premium currency and purchase entitlements.
Economy server handles all currency changes.
Offline-capable portions use server sync checkpoints.

Browser demo:
Use Godot Web export.
Use reduced content where possible.
No permanent secrets.
No trusted progression unless backend-validated.

---

8. Repository Template

Recommended monorepo layout:

godot-security-stack/
├─ launcher/
│  ├─ Cargo.toml
│  └─ src/
│     ├─ main.rs
│     ├─ app.rs
│     ├─ config.rs
│     ├─ manifest.rs
│     ├─ patcher.rs
│     ├─ downloader.rs
│     ├─ auth.rs
│     ├─ process.rs
│     ├─ integrity.rs
│     ├─ self_update.rs
│     ├─ crash_report.rs
│     └─ telemetry.rs
├─ gdextension/
│  ├─ Cargo.toml
│  └─ src/
│     ├─ lib.rs
│     ├─ bridge.rs
│     ├─ handshake.rs
│     ├─ crypto.rs
│     ├─ protocol.rs
│     ├─ gameplay_validation.rs
│     ├─ movement.rs
│     ├─ combat.rs
│     ├─ inventory.rs
│     ├─ save_validation.rs
│     └─ telemetry.rs
├─ backend/
│  ├─ Cargo.toml
│  └─ src/
│     ├─ main.rs
│     ├─ config.rs
│     ├─ routes.rs
│     ├─ auth.rs
│     ├─ sessions.rs
│     ├─ builds.rs
│     ├─ entitlements.rs
│     ├─ characters.rs
│     ├─ combat.rs
│     ├─ movement.rs
│     ├─ inventory.rs
│     ├─ economy.rs
│     ├─ quests.rs
│     ├─ loot.rs
│     ├─ telemetry.rs
│     └─ cheat_flags.rs
├─ shared/
│  ├─ Cargo.toml
│  └─ src/
│     ├─ protocol.rs
│     ├─ build_manifest.rs
│     ├─ crypto_types.rs
│     ├─ token_types.rs
│     ├─ validation_types.rs
│     └─ error_codes.rs
├─ godot_project/
│  ├─ project.godot
│  ├─ export_presets.cfg
│  ├─ bin/
│  │  └─ rust_security_bridge.dll
│  ├─ scenes/
│  ├─ scripts/
│  ├─ addons/
│  └─ resources/
├─ build_tools/
│  ├─ Cargo.toml
│  └─ src/
│     ├─ generate_manifest.rs
│     ├─ sign_manifest.rs
│     ├─ verify_export.rs
│     ├─ package_release.rs
│     ├─ check_for_dev_files.rs
│     └─ validate_release_folder.rs
├─ deploy/
│  ├─ docker-compose.dev.yml
│  ├─ backend.Dockerfile
│  ├─ migrations/
│  └─ cdn_upload/
├─ docs/
│  ├─ SECURITY_MODEL.md
│  ├─ BUILD_PIPELINE.md
│  ├─ RELEASE_CHECKLIST.md
│  ├─ LAUNCHER_PROTOCOL.md
│  ├─ GDEXTENSION_API.md
│  ├─ BACKEND_AUTHORITY.md
│  ├─ BROWSER_POLICY.md
│  ├─ AGENT_TASKS.md
│  ├─ DELTA_PATCH_GUIDE.md
│  └─ SECURITY_AUDIT_LOG.md
└─ ci/
   ├─ build_launcher.yml
   ├─ build_gdextension.yml
   ├─ build_backend.yml
   ├─ export_godot.yml
   ├─ package_release.yml
   └─ security_smoke_tests.yml

---

9. Recommended Technology Choices

Launcher:
Rust.
tokio for async.
reqwest for HTTPS.
serde for JSON/TOML.
blake3 or sha2 for hashing.
ed25519-dalek or ring for signature verification.
tracing for structured logging.
egui, Slint, Tauri, or minimal native UI after core security pipeline is proven.
Do not build launcher UI before the security pipeline works end-to-end.

GDExtension:
godot-rust/gdext (March 2026 or later).
Built in release mode (Performance safeguard tier) for all shipped builds.
Small exported Godot-facing API surface.
Use AnyArray/AnyDictionary types for engine method interop where appropriate.
Leverage GDExtension nullability (Godot 4.6) for cleaner optional return types.

Backend:
Rust.
axum or actix-web.
PostgreSQL for durable game and account state.
Redis optional for sessions, rate limiting, and short-lived token state.
SQLx or SeaORM with compile-time checked queries.
Structured logs with tracing.
OpenTelemetry-compatible observability pipeline.

Godot:
Godot 4.6.x (4.6.3 stable as of May 2026).
Custom export template compiled with PCK encryption key.
Encrypted PCK and resources.
Release export only.
No remote debug.
No debug symbols in shipped build.
D3D12 set as default Windows renderer for new projects (Godot 4.6 default).
Jolt physics as default for new 3D projects (Godot 4.6 default).
Delta PCK patching enabled for update builds.
GDScript or C# only for non-authoritative client presentation and glue code.

Build and CI:
GitHub Actions, GitLab CI, or local scripted release pipeline.
Separate dev, staging, and production release channels.
Signed manifests.
Automated release folder forbidden-file scan.
Automated smoke tests for tampering, token reuse, and stale build rejection.

---

10. Pre-Implementation Decision Gates

Before coding, answer all of these questions. Document answers in SECURITY_MODEL.md.

Game Mode:
Is the game fully offline?
Is there account-based progression?
Is there multiplayer?
Is PvP competitive?
Is trading or economy important?
Is browser support required for full game, demo, or not at all?

Server Authority Scope:
Combat: damage, hit detection, kill credit?
Movement: speed limits, teleport prevention?
Inventory: items, equipment, storage?
XP, level, reputation, faction?
Gold and premium currency?
Loot and drop results?
Quest completion and objectives?
Premium unlocks and DLC?
Achievements and leaderboard positions?
Ranked match results?
Crafting results?

Latency and Network:
What latency is acceptable? (action RPG vs turn-based vs MMO vs shooter)
What reconciliation approach? (rollback, forward prediction, server-authoritative interpolation)
What tick rate does the game logic require?

Platform:
Windows-only first?
Linux support?
macOS support?
Browser support?
Steam, Epic, itch.io, or self-hosted?

Security Level:
Casual protection only?
Basic multiplayer cheat resistance?
Serious PvP integrity?
MMO economy integrity with fraud operations team?

Budget:
Local-only?
Small VPS?
Managed Postgres?
Redis?
CDN for patch files?
Global edge infrastructure?

Offline Play:
If offline play required, which parts?
Can offline progress sync online?
If yes, what prevents local save abuse?

---

PART 2: BUILD VARIANTS AND CORE SECURITY SYSTEMS

---

11. Build Variants

Development build:
Unencrypted PCK allowed.
Debug enabled.
Remote debug allowed.
No production backend.
Fake local auth allowed.
Verbose logs.
Never distributed to players.

Internal QA build:
Encrypted or unencrypted depending on QA needs.
Debug symbols in private symbol server only.
Staging backend.
Launcher optional depending on test.
Test accounts only.

Preview or beta build:
Encrypted PCK.
Release export.
Launcher required.
Staging or preview backend.
Extra telemetry.
No production economy.

Production desktop build:
Encrypted PCK.
Custom export template.
Release binary.
Stripped symbols.
No debug.
No remote debug.
Launcher required.
Signed manifest required.
Backend validates build ID.
Backend validates session.

Production browser build:
Godot Web export.
No trusted client secrets.
No local authority.
Backend validates gameplay.
May have reduced content or demo restrictions.

---

12. Custom Godot 4.6 Export Plan

Purpose: Make scenes, scripts, and resources non-plain-text. Prevent basic PCK extraction from immediately exposing project internals. Raise reverse-engineering effort.

Required steps:
Generate per-channel AES-256 encryption key.
Build Godot 4.6 custom export template with key.
Configure export preset to encrypt PCK and resources.
Export release build.
Strip symbols.
Check output folder for forbidden dev files.
Generate signed file manifest after export.
Smoke test clean launch through launcher.

Security notes:
The PCK encryption key exists in the binary.
A determined reverse engineer can extract it.
This is still worth doing because the target attacker is casual.
Do not ship the key in scripts, config files, docs, or CI logs.
Do not reuse production keys in public test builds.
Use separate dev, staging, and production keys.

Godot 4.6 specific notes:
D3D12 is now the default renderer for new Windows projects.
Jolt physics is now the default for new 3D projects. Check migration guide if migrating from 4.5.
Delta PCK patching can be generated during export for incremental updates.

Key handling rules:
Production key lives in CI secret storage or restricted local release machine only.
Never commit key to git.
Never print key in build logs.
Rotate key per major release channel if practical.
Keep old keys only as long as old builds need delta patch support.

Export validation checklist:
Game executable exists.
Encrypted PCK exists.
GDExtension binary exists for target platform.
No raw .gd files outside encrypted PCK.
No .godot folder in release package.
No editor-only plugins.
No raw source asset dumps.
No debug symbol files in public package.
No private keys.
No production API secrets.
Manifest generated after final packaging state.

---

13. Rust Launcher Plan

Purpose: Make the launcher the normal and required entry point for protected desktop mode. Optionally use LibGodot (Godot 4.6) to embed Godot inside the Rust binary — see Section 40.

Responsibilities:
Self-version check against backend minimum launcher version.
Download latest signed manifest from CDN.
Verify manifest signature using embedded public key (ed25519).
Hash local game files (blake3).
Repair or reject modified files by re-downloading from CDN.
Authenticate user — receive launcher session token from backend.
Request short-lived single-use launch token from backend.
Start Godot process with controlled arguments (or run LibGodot inline).
Monitor child process exit code.
Collect crash logs and upload to backend.
Upload integrity telemetry.
Show useful errors to players.
Optional: launcher self-update from signed CDN artifact.

Launcher must not:
Scan unrelated processes.
Read browser, password manager, or app memory.
Install kernel drivers.
Hide files or processes.
Persist itself secretly.
Block debuggers in ways that break normal machines.
Pretend to be unbreakable.

Launcher launch flow:
Player starts launcher.
Launcher loads config.
Launcher checks own version.
Launcher downloads channel manifest from CDN.
Launcher verifies manifest signature using embedded public key.
Launcher checks local install folder.
Launcher hashes required files with blake3.
Launcher downloads changed or missing files.
Launcher verifies patched files.
Launcher authenticates player.
Backend creates launcher session.
Launcher requests single-use launch token (60s TTL).
Launcher starts Game.exe with build ID, manifest hash, launch token, and launcher PID.
Game loads.
Rust GDExtension validates token with backend.
Backend activates game session.
Game opens main menu.
Direct Game.exe run fails protected online mode.

Controlled argument example:
Game.exe
  --build-id 2026.05.29.001
  --channel production
  --launch-token SINGLE_USE_SHORT_LIVED_TOKEN
  --launcher-pid PID
  --manifest-hash BLAKE3_HASH

Important:
Command-line arguments can be inspected on any OS.
The token must be short-lived (60 seconds TTL).
The token must be single-use.
The token must be backend-validated.
The token must be bound to account, build ID, manifest hash, issue time, and channel.

---

14. Signed Manifest Plan (v2 with Delta PCK)

Purpose: Prevent casual file replacement. Give launcher the exact approved release layout. Track delta patches for bandwidth-efficient updates.

Manifest v2 example:
{
  "schema_version": 2,
  "build_id": "2026.05.29.001",
  "channel": "production",
  "created_at": "2026-05-29T00:00:00Z",
  "godot_version": "4.6.3",
  "min_launcher_version": "0.2.0",
  "min_protocol_version": 2,
  "files": [
    {
      "path": "Game.exe",
      "hash_algorithm": "blake3",
      "hash": "abc...",
      "size": 128000000,
      "required": true
    },
    {
      "path": "game.pck",
      "hash_algorithm": "blake3",
      "hash": "def...",
      "size": 2400000000,
      "required": true
    },
    {
      "path": "bin/rust_security_bridge.dll",
      "hash_algorithm": "blake3",
      "hash": "ghi...",
      "size": 9000000,
      "required": true
    }
  ],
  "delta_patches": [
    {
      "from_build": "2026.05.20.001",
      "to_build": "2026.05.29.001",
      "patch_file": "patches/game_20_to_29.patch",
      "hash": "jkl...",
      "size": 45000000
    }
  ],
  "signature_algorithm": "ed25519",
  "signature": "base64_signature_here"
}

Rules:
Manifest is signed by release private key.
Launcher embeds only public verification key.
Backend stores accepted build IDs and manifest hashes.
Backend rejects unknown build IDs.
Backend can force update or reject stale builds.
Do not invent cryptography — use audited signature libraries.
Private signing key never ships in any artifact.
Public verification key may ship in launcher binary.

---

15. Rust GDExtension Plan

Purpose: Native bridge between Godot and Rust for protected client helpers, session validation, and protocol construction.

As of March 2026, gdext supports:
Three safeguard tiers: Strict (debug), Standard, Performance (release).
AnyArray and AnyDictionary for engine method interop.
GDExtension nullability in Godot 4.6 for clean optional return types.

Good GDExtension candidates:
Launcher and game handshake validation.
Build and session validation helper.
Packet signing helper.
Movement sanity prechecks.
Combat request construction.
Inventory transaction request construction.
Local telemetry packing.
Asset manifest validation.
Save validation for offline sections.
Sensitive constants.
Protocol versioning.
Anti-tamper heartbeat helper.

Bad GDExtension candidates:
Full UI.
Basic animation code.
Normal quest dialogue.
Every scene script.
All gameplay code before there is a server authority model.
Anything that slows iteration without real security value.

Godot-facing API should be minimal:
validate_launch_args(args) -> Result
create_handshake_payload() -> Dictionary
make_combat_request(player_id, target_id, spell_id, position, timestamp) -> PackedByteArray
make_inventory_request(player_id, item_id, action) -> PackedByteArray
submit_local_telemetry(event) -> void
validate_local_asset_manifest(path) -> Result
get_protocol_version() -> int
get_build_fingerprint() -> String

GDScript usage example:
var bridge := RustSecurityBridge.new()
var result := bridge.validate_launch_args(OS.get_cmdline_args())
if not result.ok:
    push_error(result.message)
    get_tree().quit()

var request := bridge.make_combat_request(
    player_id, target_id, spell_id, player_position, client_timestamp)
Network.send(request)

Important: Rust can structure and sign the request. The server still validates everything. Never assume Rust code in the client makes the result trusted.

---

16. Backend Authority Plan

Purpose: Stop cheating by refusing invalid state changes.

Backend services:
AuthService
SessionService
BuildValidationService
CharacterService
InventoryService
EconomyService
QuestService
CombatService
MovementService
LootService
EntitlementService
TelemetryService
CheatFlagService
AuditLogService

The client may submit intent. The server validates intent. The server applies valid state changes. The server rejects invalid state changes. The server writes audit logs for all important changes.

Never trust client for:
Gold, premium currency, items, XP, level.
Loot, quest completion, premium unlocks.
Cooldowns, combat result, movement result.
NPC kill credit, trading, auction, crafting results.
Dungeon and raid lockouts, achievement unlocks.
Ranked match results and entitlements.

Client may send intent:
I want to cast spell X on target Y.
I want to move from A toward B.
I want to use item X.
I want to loot corpse X.
I want to accept quest X.
I want to complete quest objective X.
I want to buy vendor item X.
I want to trade item X.

Transaction pattern for all important state changes:
BEGIN TRANSACTION
  Validate current state
  Validate request
  Apply state change atomically
  Write audit log entry
COMMIT
ON FAILURE: ROLLBACK — do not partially apply

---

PART 3: ADVANCED AUTHORITY AND SECURITY SYSTEMS

---

17. Combat Authority Plan

Client sends: Player 10 casts Fireball Rank 2 on Mob 883.

Server checks:
Player session valid and build ID allowed.
Player owns character.
Character is alive.
Character is not stunned or silenced if relevant.
Spell is learned.
Spell rank is valid.
Spell is not on cooldown.
Global cooldown is valid.
Enough mana or resource.
Target exists.
Target is alive.
Target is hostile or valid.
Target is in range.
Line-of-sight valid if required.
Cast time and channel time valid.
Action timing not physically impossible.
Player position plausible.
Target position plausible.
No impossible action spam.

Server calculates:
Hit or miss.
Critical hit.
Block, dodge, parry, or resist if relevant.
Damage amount.
Threat.
Durability changes.
Resource cost.
Cooldown application.
XP eligibility.
Loot eligibility.
Quest credit eligibility.

Client displays:
Animation.
VFX.
Floating text.
UI cooldown display.
Server result notification.

Server never accepts from client:
Damage amount.
Critical hit result.
Loot roll result.
Kill confirmation.
XP amount.
Cooldown reset.
Resource gain without validation.

---

18. Movement Authority Plan

Purpose: Prevent basic speed, fly, teleport, and interaction-distance cheats without requiring perfect physics on day one.

Server tracks:
Last known position.
Last movement timestamp.
Movement mode.
Max allowed speed.
Current zone and terrain bounds.
Mount state.
Falling state.
Swimming state.
Root, stun, and snare state.
Teleport permissions.
Recent correction history.

Reject or flag:
Impossible distance per second.
Repeated micro-teleports.
Moving while stunned or rooted.
Attacking while too far away.
Looting from impossible distance.
Interacting through walls beyond tolerance.
Entering locked zones.
Invalid vertical movement.
Movement mode mismatch.

Movement severity ladder:
Level 1: tolerate minor network weirdness.
Level 2: soft correction — snap client to server position.
Level 3: reject the action but allow session to continue.
Level 4: flag session and account for review.
Level 5: temporary disconnect.
Level 6: manual review and ban queue.

Do not instantly ban from one suspicious movement packet. Use accumulated evidence. Keep false positives low.

---

19. Inventory and Economy Authority Plan

Never accept from client:
Set gold to X.
Add item Y.
Quest complete true.
Loot table says I got item.
Trade accepted with modified values.
Crafting succeeded locally.
Vendor buy without server validation.
Premium unlock true.

Use transaction requests:
Client request: Use item 1004 on target 889.
Server checks: Item exists in player inventory, item is usable, player owns item, item not expired or locked, target is valid, cooldown is valid, zone and quest state allows it.
Server applies: Consumes charges, applies result transactionally.

Required transaction log systems:
Item creation.
Item deletion.
Gold changes.
Premium currency changes.
Trade.
Auction.
Mail attachments.
Crafting.
Vendor buy and sell.
Loot.
Quest rewards.
Admin grants.
Compensation grants.
Rollback events.

---

20. Quest and Progression Authority Plan

Server owns:
Quest accepted state.
Quest objective progress.
Quest completion state.
Quest rewards.
XP grant.
Reputation.
Achievement progress.
Unlocks.

Client may display:
Quest text.
Objective UI.
Markers.
Dialogue.
Progress feedback.

Client may request:
Accept quest.
Abandon quest.
Turn in quest.
Report objective event.

Server validates:
Quest is available.
Prerequisites met.
Player level, faction, or class allowed.
Objective event is plausible.
Required item exists.
Required kill was credited server-side.
Required location reached.
Reward not already claimed.
Reward choice valid.

---

21. Entitlement and Premium Unlock Plan

Never ship permanent premium secrets in the client.

Server owns:
Purchases.
DLC ownership.
Premium currency.
Account entitlements.
Season passes.
Cosmetic ownership.
Founder rewards.

Client may cache:
Public entitlement display.
Temporary access token.
Non-secret product metadata.

Backend validates:
Account owns entitlement.
Token is current.
Build and channel support entitlement.
No duplicate claim.
No refunded purchase.
No chargeback block if relevant.

---

22. Save System Plan

For fully offline games:
Use signed save files to detect casual edits.
Accept that determined users can still edit offline saves.
Do not claim offline saves are cheat-proof.

For online progression:
Local save is cache only.
Server owns real progression.
Client syncs from server on login.
Client submits intent, not final state.

For hybrid games:
Separate offline profile from online profile.
Never let offline-only progress directly become authoritative online economy or progression without server validation.

Signed local save structure:
Save payload.
Schema version.
Build version.
Timestamp.
Hash.
Signature or MAC.
Optional device-local key.

Remember: If the key is local, it can be extracted. Signed saves stop casual edits, not professional tampering.

---

23. Token and Session Plan

Token types:

Launcher login token:
Received after user auth.
Used only by launcher.
Short-lived.
Not permanent.

Launch token:
Short-lived (60 seconds TTL).
Single-use.
Passed to Godot.
Bound to build ID, manifest hash, account, issue timestamp, and channel.
Validated by backend.

Game session token:
Issued after Godot validates launch token.
Used for game server communication.
Rotated every 30 minutes.
Can be revoked.

Backend rejects if:
Token expired.
Token already used.
Build ID not allowed.
Manifest hash wrong.
Account banned.
Launcher version blocked.
Suspicious repeated launch failures.
Protocol version mismatch.
Session was revoked.

Token storage rules:
No permanent secrets in client.
No long-lived production API keys in client.
No admin secrets in client.
No private signing keys in client.
Use opaque server-side sessions when possible.
Use short-lived signed tokens only where practical.

---

24. Anti-Tamper Without Malware Behavior

Good checks:
File hashes via signed manifest.
Build ID validation at backend.
Process started through launcher for protected mode.
Expected command args present.
GDExtension loaded and handshake completed.
Backend launch token confirmed.
Runtime version heartbeat.
Manifest mismatch telemetry.
Known bad build rejection.
Release folder validation.

Bad checks — do not use:
Kernel drivers.
Scanning unrelated processes.
Reading browser, password, or app memory.
Hiding files or processes.
Persistence tricks.
Aggressive debugger blocking that breaks normal machines.
Rootkit behavior.
Anything that looks like malware.

Principle: The security system must be trustworthy, explainable, and commercially defensible. Players should be able to read what it does.

---

25. Browser Build Strategy

Browser clients are inspectable. Browser devtools exist. WASM can be inspected. Network calls can be inspected. Local storage can be modified.

Therefore:
No important secrets.
No client-side authority.
No trusted local progression.
No trusted local currency.
No trusted premium unlocks.
No trusted combat results.
No trusted inventory decisions.

Browser is good for:
Demo.
Lightweight access.
Early-zone testing.
Account login.
Social and playable preview.
Low-stakes gameplay.

Browser must not use:
Permanent secret keys.
Local economy authority.
Client-side reward authority.
Client-only anti-cheat claims.

Recommended split:
Desktop: full protected client, launcher required, encrypted PCK, Rust GDExtension, production progression.
Browser: no launcher, no strong client protection, server-authoritative only, reduced asset set optional, demo or limited progression optional.

---

26. Release Build Pipeline

Production release flow in order:
Build Rust backend.
Run backend tests.
Build Rust shared crate.
Build Rust GDExtension release library.
Copy GDExtension into Godot project.
Generate or load PCK encryption key from CI secret storage.
Build custom Godot 4.6 export template with key.
Export encrypted Godot project.
Strip symbols.
Build Rust launcher.
Assemble release folder.
Check for forbidden files (private keys, .env, .godot folder, dev assets, debug symbols).
Generate file manifest.
Sign manifest with release private key.
Verify manifest signature locally.
Package release.
Upload release to patch CDN or storage.
Register build ID and manifest hash with backend.
Smoke test clean install.
Smoke test modified file rejection.
Smoke test stale build rejection.
Smoke test direct Game.exe launch rejection.
Smoke test launch token reuse rejection.
Smoke test missing GDExtension rejection.
Smoke test backend combat and inventory authority.

Release checklist:
Build ID unique.
Channel correct.
Manifest signed.
Backend accepts build ID.
Launcher version allowed.
PCK encrypted.
Debug disabled.
Remote debug disabled.
Symbols stripped.
No dev files present.
No private keys present.
No secrets in config files.
Crash reporting works.
Logs redact tokens.
Patch download works.
Clean install works.
Tampered install rejected or repaired.
Delta PCK generated for supported previous versions.

---

27. CI/CD Pipeline

Minimum CI jobs:
Rust format check.
Rust clippy.
Rust tests.
Backend integration tests.
GDExtension build.
Launcher build.
Godot export check.
Release folder forbidden-file scan.
Manifest generation.
Manifest signature verification.
Package integrity test.
Security smoke tests.

Suggested CI stages:
Validate source.
Build Rust crates.
Build Godot extension.
Export Godot.
Package.
Sign.
Deploy to staging.
Run smoke tests.
Promote to production.

Forbidden file scan must fail build if release includes:
Private keys.
.env files.
.godot directory.
Raw source asset dumps.
Debug symbols.
Editor-only plugins.
Dev scenes.
Test credentials.
Unencrypted script files outside PCK.

CI secrets rule:
Do not print secrets.
Do not upload keys as artifacts.
Do not include signing keys in generated packages.
Use masked secret variables only.

---

28. Telemetry and Cheat Flag Plan

Telemetry goals:
Detect impossible behavior.
Find broken builds.
Catch repeated tampering.
Monitor build distribution.
Find performance regressions.
Monitor auth failure rates.
Monitor movement violation rates.
Track economy anomalies.

Cheat flag pipeline:
Client behavior generates telemetry events (packed by GDExtension).
Backend receives and stores events.
Anomaly detection rules evaluate events.
Flags accumulate on session or account.
Soft responses fire first (correction, rejection).
Hard responses after sustained evidence (disconnect, flag queue).
Manual review before permanent bans.

Key metrics to alert on:
Auth failure rate above threshold.
Cheat flag rate spike.
Movement violation rate spike.
Token reuse attempts.
Unknown build ID requests.
Manifest signature failures.
PCK hash mismatches.

---

29. Validation Tests

Required security smoke tests:
Clean install launches successfully through launcher.
Modified game.pck is rejected or repaired.
Modified GDExtension binary is rejected or repaired.
Direct Game.exe launch fails online protected mode.
Reused launch token fails on second attempt.
Expired launch token fails.
Unknown build ID fails backend session creation.
Invalid manifest signature fails launcher with clear error.
Missing manifest file fails launcher with clear error.
Missing GDExtension fails game startup with clear error.
Backend correctly rejects invalid combat request.
Backend correctly rejects invalid inventory claim.
Backend correctly rejects movement outside plausible bounds.

---

30. Logging Standards

A high-quality log entry answers:
What happened? (event type, action, result)
Where did it happen? (service, function, module)
Which entity was affected? (player_id, character_id, session_id)
What was the result? (success, failure, fallback, rejected)
What was the impact? (business or gameplay effect)
Why did it happen? (when determinable from context)
What did the system do after? (retry, fallback, abort, flag)
How can I trace this? (trace_id, correlation_id, span_id)
Is this expected or abnormal behavior?

Rust + tracing structured log example:
tracing::warn!(
    trace_id = %ctx.trace_id,
    player_id = %player.id,
    session_id = %session.id,
    build_id = %session.build_id,
    event = "movement_violation",
    severity = "level_3",
    expected_max_distance = max_dist,
    actual_distance = actual_dist,
    action = "request_rejected",
    "Impossible movement rejected: distance {actual_dist:.2}m in {dt:.3}s (max {max_dist:.2}m)"
);

---

PART 4: AGENT EXECUTION FRAMEWORK

---

31. Agent Task Definitions

Rules for coding agents executing this blueprint:

Do:
Read AGENT_TASKS.md before starting any task.
Follow strict step order. Each step has defined inputs, outputs, and tests.
Run all listed tests after each step. Do not proceed if tests fail.
Update docs during work, not after.
Document all files changed, commands run, tests run, and known issues.

Do not:
Skip the vertical slice. It is non-negotiable.
Build full features before the security handshake is proven end-to-end.
Output placeholder code. All code must be functional.
Commit secrets, keys, or production credentials to any file.
Overbuild. Implement only the assigned step and stop.

Agent step template:
## Agent Step: [STEP_NAME]

Context:
  Architecture: Rust launcher → encrypted Godot 4.6 → Rust GDExtension → Rust backend
  Server owns all progression, economy, and combat state.
  Build must be reproducible and deterministic.

Required output:
  Files changed with descriptions.
  Commands run exactly as written.
  Tests run with results.
  Result: pass or fail with details.
  Known issues if any.
  Next recommended step.

Rules:
  No placeholders or TODO comments.
  All code must compile and be logically tested.
  Stop after assigned step only.

---

32. Implementation Risk Register

Risk: PCK encryption gives false confidence.
Mitigation: State clearly that backend authority is the real protection.

Risk: Launcher can be bypassed.
Mitigation: Backend requires valid launch token and accepted build ID regardless of how the game is started.

Risk: Command-line token can be copied.
Mitigation: Token is short-lived (60s), single-use, backend-bound, and rotated.

Risk: Rust GDExtension can still be reverse engineered.
Mitigation: Do not trust client-side Rust for authority. Server validates all results independently.

Risk: Server authority increases hosting complexity.
Mitigation: Start with one vertical slice and expand system by system.

Risk: Browser build weakens overall security.
Mitigation: Treat browser as untrusted and optional or demo-only.

Risk: Anti-tamper becomes invasive.
Mitigation: Explicitly ban malware-like behavior in code review and documentation.

Risk: False positive bans harm real players.
Mitigation: Use severity ladder, soft corrections, and require manual review before permanent bans.

Risk: CI leaks signing secrets.
Mitigation: Secret scanning, masked variables, no key output in logs, no keys in artifacts.

Risk: Agents overbuild everything.
Mitigation: Strict agent execution order and vertical slice first rule enforced.

Risk: LibGodot API instability.
Mitigation: Monitor godot-rust/gdext changelog. Pin versions explicitly.

Risk: Delta PCK breaks on partial or interrupted downloads.
Mitigation: Launcher verifies delta patch hash before applying. Falls back to full download on mismatch.

---

33. Best First Vertical Slice

Build this first, before any feature work:

Step 1: Rust launcher downloads signed manifest from CDN.
Step 2: Launcher verifies manifest signature using embedded public key.
Step 3: Launcher verifies Game.exe, game.pck, and rust_security_bridge.dll hashes.
Step 4: Launcher authenticates player. Backend issues launcher session.
Step 5: Launcher requests single-use launch token from backend.
Step 6: Launcher starts encrypted Godot 4.6 export with controlled arguments.
Step 7: Godot loads Rust GDExtension at startup.
Step 8: GDExtension validates launch token with backend via HTTPS.
Step 9: Backend marks token used and activates game session.
Step 10: Main menu opens only if all checks passed.

Success criteria — all must pass:
Launcher path works end-to-end.
Direct Game.exe path fails protected online mode.
Modified game.pck is rejected or repaired.
Modified GDExtension is rejected or repaired.
Reused launch token fails.
Old or unknown build ID fails.
Missing manifest fails with clear error.
Invalid manifest signature fails with clear error.

This one slice proves the whole architecture. Do not start full combat, full inventory, browser support, or complex anti-cheat before this slice passes all tests.

---

34. Generic Configuration Blueprint

[project]
game_name         = "YourGame"
channel           = "production"
build_id          = "2026.05.29.001"
protocol_version  = 2
godot_version     = "4.6.3"

[launcher]
min_launcher_version     = "0.2.0"
allow_direct_game_launch = false
repair_modified_files    = true
upload_crash_reports     = true
use_libgodot             = false

[security]
require_signed_manifest          = true
require_launch_token             = true
require_gdextension_handshake    = true
allow_offline_mode               = false
token_ttl_seconds                = 60
session_rotation_minutes         = 30
use_delta_pck_patches            = true

[backend]
base_url                  = "https://api.yourgame.com"
build_validation_required = true

[godot]
game_executable      = "Game.exe"
pck_file             = "game.pck"
gdextension_library  = "bin/rust_security_bridge.dll"
renderer_windows     = "d3d12"
physics_3d           = "jolt"

Do not ship production secrets in this config.

---

35. Documentation Files To Create

SECURITY_MODEL.md: Threat model, goals, non-goals, trust boundaries.
BUILD_PIPELINE.md: How to build launcher, backend, GDExtension, Godot export, manifest, and package.
RELEASE_CHECKLIST.md: Step-by-step release signoff.
LAUNCHER_PROTOCOL.md: Manifest verification, patching, auth, token request, process start.
GDEXTENSION_API.md: All Godot-callable Rust methods with signatures and semantics.
BACKEND_AUTHORITY.md: Which game systems are server-authoritative and why.
BROWSER_POLICY.md: Browser limitations and allowed use cases.
AGENT_TASKS.md: Agent execution order and rules.
DELTA_PATCH_GUIDE.md: Godot 4.6 delta PCK strategy, patch generation, verification, CDN upload.
SECURITY_AUDIT_LOG.md: Security decisions, risks accepted, signoffs, incident history.

---

36. Final Do and Do-Not List

Always do:
Use Rust launcher.
Use signed manifest with delta patch support (schema v2).
Use encrypted PCK.
Use custom Godot 4.6 export template.
Use stripped release builds.
Use Rust GDExtension for selected sensitive helpers.
Use backend authority for important state.
Use short-lived tokens.
Use build ID validation.
Use telemetry cautiously.
Use release smoke tests.
Use clear documentation written during work.
Evaluate LibGodot for tighter launcher coupling.
Use Delta PCK for bandwidth-efficient updates.

Never do:
Claim the client is impossible to crack.
Trust local files after one check.
Trust command-line arguments for security decisions.
Trust local saves for online progression.
Ship private keys or production secrets in any artifact.
Use kernel anti-cheat.
Scan unrelated processes.
Use malware-like tricks.
Instant-ban from one suspicious movement packet.
Let agents skip the vertical slice.
Overbuild before vertical slice passes all tests.

---

37. Final Recommended Stack

Launcher:
Rust.
tokio, reqwest, serde.
blake3 or sha2.
ed25519-dalek or ring.
tracing.
egui, Slint, Tauri, or minimal native UI after security pipeline proven.

GDExtension:
godot-rust/gdext (March 2026 or later).
Rust release build with Performance safeguard tier.
Small public Godot API surface.
AnyArray/AnyDictionary for engine interop.

Backend:
Rust.
axum or actix-web.
PostgreSQL with SQLx or SeaORM.
Redis optional.
Structured telemetry with tracing.
OpenTelemetry-compatible pipeline.
Opaque server-side sessions or short-lived signed tokens.

Godot:
Godot 4.6.x.
Custom export template.
Encrypted PCK.
Release-only export.
D3D12 default on Windows.
Jolt physics default for 3D.
Delta PCK patching for updates.
GDScript or C# for presentation only.
Rust GDExtension for protected helpers.

Browser:
Godot Web export.
Server-authoritative.
No trusted secrets.
Reduced content optional.

---

38. Final Architecture Statement

For any Godot game that needs stronger protection, the production-grade strategy is:

Do not try to make the client impossible to crack.
Make the client untrusted.
Use a Rust launcher to control normal entry, patching, verification, and launch tokens.
Evaluate LibGodot (Godot 4.6) for embedding Godot inside the Rust binary.
Use a custom encrypted Godot 4.6 export to stop casual ripping.
Use Delta PCK patching (Godot 4.6 native) for bandwidth-efficient updates.
Use Rust GDExtension (gdext) for selected sensitive helper logic and handshake code.
Use a Rust backend as the source of truth for valuable gameplay state.
Use signed manifests (v2 with delta patch support) and short-lived sessions.
Use telemetry and validation tests.
Keep anti-tamper safe, non-invasive, and commercially defensible.
Browser builds are possible but must be treated as fully inspectable untrusted clients.

This plan is suitable as the reusable foundation for any Godot game before implementation begins.

---

PART 5: GODOT 4.6 FEATURE INTEGRATION

---

40. LibGodot: Godot Embedded in Rust

Godot 4.6 introduces LibGodot, a supported mechanism to embed the Godot engine as a library inside another application. This changes the fundamental launcher architecture.

Traditional approach (pre-4.6):
Rust Launcher.exe spawns Game.exe as a separate process.
Token passed via command-line arguments (inspectable).
Two separate binaries distributed.
Direct launch of Game.exe bypasses launcher.

LibGodot approach (Godot 4.6):
Single Rust binary contains Godot.
No separate Game.exe.
Token passed via internal function call (not on command line).
Much harder to bypass because there is no game binary separate from the launcher.
Godot runs inside the Rust process.

Security impact:
Removes the separate game executable as a bypass surface.
Tighter coupling between launcher security checks and game startup.
Token exchange happens inside process memory, not on visible command line.
Distribution is one binary plus PCK.

Note: LibGodot is marked as early-stage in Godot 4.6. Monitor stability before shipping in production. Use traditional launcher for early projects. Evaluate LibGodot for later revisions.

Conceptual Rust integration:
fn main() -> Result<()> {
    // Phase 1: Rust-side security checks before Godot starts
    let manifest = download_and_verify_manifest()?;
    let token = authenticate_and_get_launch_token(&manifest)?;

    // Phase 2: Initialize Godot as embedded library
    let godot = GodotInstance::new();
    godot.set_main_scene("res://scenes/Main.tscn");

    // Phase 3: Pass token via internal API (not CLI)
    godot.set_internal_launch_token(token);

    // Phase 4: Run Godot event loop inside Rust
    godot.run();
    Ok(())
}

---

41. Delta PCK Patching Strategy

Godot 4.6 introduces native delta encoding for Patch PCKs. Patch files include only changed portions of resources, not entire files.

How it works:
Godot 4.6 export generates a patch PCK with only changed resource portions.
Especially effective for localization updates, balance patches, and small content updates.
Launcher downloads delta patch file instead of full PCK.
Launcher verifies delta patch file hash before applying.
Applied patch transforms old PCK to new PCK deterministically.
Manifest v2 tracks delta patch files alongside full files.

Launcher patching decision logic:
fn determine_patch_strategy(current_build: &str, manifest: &Manifest) -> PatchStrategy {
    if let Some(delta) = manifest.find_delta_patch(current_build, &manifest.build_id) {
        let current_pck_hash = hash_file(&config.pck_path)?;
        if current_pck_hash == delta.from_hash {
            return PatchStrategy::Delta(delta.clone());
        }
    }
    PatchStrategy::Full(manifest.files.find("game.pck").clone())
}

CDN and build pipeline requirements:
Generate delta patches from each recent N versions to current (e.g., last 3 versions).
Store delta patches on CDN with content-hashed filenames.
Include delta patch metadata in signed manifest.
Set maximum supported delta chain length.
Keep full PCK always available as fallback.

---

42. gdext Safeguard Tiers (March 2026)

godot-rust/gdext supports three safeguard levels that trade safety against performance.

Strict (default in debug builds):
Extra guardrails to detect bugs as early as possible.
Use for all development and QA builds.
Never ship with Strict tier.

Standard:
Balanced safety and performance.
Configurable.
Use for staging or preview builds.

Performance (default in release builds):
Minimal overhead.
Production performance.
Always use cargo build --release for shipped GDExtension.

Cargo.toml for production GDExtension:
[profile.release]
opt-level = 3
lto = true
codegen-units = 1
strip = true

Never ship a debug-profile GDExtension to players.
Always cargo build --release for any binary that leaves development machines.

AnyArray and AnyDictionary (March 2026 gdext):
New types that store any array or dictionary regardless of element type.
AnyArray exposes only operations that work for all element types.
Use when passing arrays to engine functions that previously required VarArray.
Deref coercion allows existing Array<T> code to keep working.

---

43. Jolt Physics and Server Authority (Godot 4.6)

Godot 4.6 makes Jolt the default 3D physics engine for new projects.

What changed:
Jolt is a third-party rigid body solver focused on determinism and stability.
Existing projects are not automatically migrated. Results will differ from old physics.
No bit-exact cross-platform determinism is claimed, but behavior is more consistent under load.

Server-side physics validation implications:
For simple MMO-style movement validation: server-side positional bounds checking is sufficient. No need to run a full physics engine on the server.
For physics-authoritative server movement (physics-driven games): consider running Jolt on the server too for consistency.
Always check the migration guide before upgrading from Godot 4.5 to 4.6 for 3D physics behavior.

---

44. GDExtension Nullability in Godot 4.6

Godot 4.6 introduces nullability support in GDExtension. This allows Rust GDExtension code to properly express nullable types in the Godot-facing API.

Before (ambiguous):
#[func]
fn get_player_data(player_id: GString) -> Dictionary {
    find_player(&player_id).map(|p| p.to_dict()).unwrap_or_default()
    // Callers cannot distinguish 'not found' from 'found but empty'
}

After (Godot 4.6 nullability):
#[func]
fn get_player_data(player_id: GString) -> Option<Dictionary> {
    find_player(&player_id).map(|p| p.to_dict())
    // GDScript receives null or a valid Dictionary — unambiguous
}

Use nullability for all GDExtension functions where absence of a value is meaningful.

---

PART 6: NETWORKING AND PROTOCOL DESIGN

---

45. Networking Protocol Selection Matrix

HTTPS/REST:
Latency: High.
Best for: Turn-based actions, account management, economy transactions.
Security notes: TLS required. Rate limit all endpoints. Idempotency keys for transactions.

WebSocket (WSS):
Latency: Medium.
Best for: Chat, notifications, MMO event streams.
Security notes: Auth token on connect. Heartbeat required. Rate limit message types.

Custom UDP (ENet or quiche):
Latency: Very Low.
Best for: Action RPG, PvP, fast-paced games.
Security notes: Packet signing required. Sequence numbers required. Replay prevention required.

gRPC (HTTP/2):
Latency: Low to Medium.
Best for: Backend microservices.
Security notes: TLS and auth interceptor on every call.

---

46. Protocol Security Hardening

Every request must carry:
Session token.
Build ID.
Protocol version.
Sequence number (for replay prevention).

Rate limiting:
Per player, per endpoint, per action type.
Reject bursts at load balancer before reaching game logic.
Different limits for different action types (movement vs economy vs combat).

Replay prevention:
Track received sequence numbers per session.
Reject any packet with a sequence number already seen.
Reject requests older than tolerance window (e.g., 30 seconds).

Payload validation:
Schema validation before any business logic.
Reject malformed payloads early with opaque error codes.
Never reveal internal state in error messages.

---

47. Client Prediction and Reconciliation

Client prediction is the technique of running game logic locally to hide network latency. It does not mean the client is trusted.

Client predicts:
Movement.
Animation.
Local sound effects.
UI cooldown display.

Client sends to server:
Intent with input sequence number.
Signed by GDExtension if required.

Server sends back:
Authoritative result with input sequence number.
True position if client prediction diverged.

Client reconciles:
If server result differs from prediction, snap or lerp to server state.
Reapply unacknowledged inputs on top of server state.

Never predict:
Damage, loot, XP, currency, quest state, achievements.
Any valuable state that belongs to the server.

---

48. Protocol Schema Design Principles

Use versioned protocol messages:
Include protocol_version in every message.
Backend rejects messages with unsupported protocol versions.
Increment protocol_version when making breaking changes.

Use opaque IDs:
Do not expose database row IDs in protocol.
Use UUIDs or opaque identifiers for all entities.

Minimize client knowledge:
Client should know only what it needs to display.
Do not send server state that the client does not render.
Do not send other players' hidden stats, items, or data.

---

PART 7: GENRE QUICK-START TEMPLATES

---

49. Singleplayer Action / Adventure

Backend need: Low — account, cloud save, entitlements.

Required:
Encrypted PCK.
Custom export template.
Stripped release binary.
Optional launcher for update distribution.

Optional:
Signed saves to detect casual edits.
Cloud save sync checkpoint for account-owned progression.
Entitlement backend for paid DLC.

Skip or defer:
Always-online requirement unless product design requires it.
Full combat server authority (no competitive surface).
Heavy anti-cheat beyond basic tamper evidence.

Note: Offline saves cannot be made cheat-proof. Do not claim otherwise. Focus encryption on asset protection not on preventing all save editing.

---

50. Co-op Survival / PvE

Backend need: Medium — inventory, loot, XP, crafting authority.

Required:
Launcher.
Encrypted PCK.
GDExtension.
Backend authority for drops, XP, and crafting results.

Optional:
Client-side movement prediction.
Soft movement correction (PvE tolerance is higher than PvP).

Skip or defer:
Frame-perfect movement validation (PvE can be lenient).
Strict speed limits unless flying or teleporting is a real issue.

Note: Focus server authority on loot and economy. That is the main cheat surface in co-op PvE. Movement can use soft correction.

---

51. Competitive PvP

Backend need: High — all combat, movement, and match results.

Required:
Full launcher.
Encrypted PCK.
GDExtension required.
Strict server authority: movement, combat, hit detection.
Replay sampling for post-match review.
Telemetry for pattern detection.
Ranked match result authority.

Optional:
Delta rollback netcode for fighting or shooter feel.
Separate ranked queue with stricter checks than casual.

Skip or defer:
Client-reported hit results.
Client-side damage calculation of any kind.
Trusting any gameplay state from client.

Note: PvP is where cheating causes the most player-visible harm. Server authority is non-negotiable. Expect dedicated cheat tool development targeting your game once it is popular.

---

52. MMO / Extraction Game

Backend need: Maximum — everything.

Required:
Full stack — all components required.
Backend owns: characters, items, gold, XP, quests, loot, vendors, trading, mail, auction, cooldowns, and movement sanity.
Client is: renderer, input collector, UI layer, and prediction layer only.
Transaction logs for all economy events.
Rate limiting on all actions.
Anti-farm detection for economy health.
Audit trail for all item creation, deletion, and transfer.

Optional:
Auction house fraud detection.
Bot detection heuristics.
Economy anomaly alerting.

Note: Economy integrity is the hardest problem in MMOs. Duplication bugs are existential threats to the game economy. Design all economy transactions as database ACID operations from day one. Rollback procedures must be tested before launch.

---

53. Idle / Casual with Economy

Backend need: Medium — currency, purchases, soft economy.

Required:
Encrypted PCK.
Backend for premium currency and purchase validation.
Entitlement server for in-app purchases.
Stripe or payment processor webhook → entitlement grant → no client trust in purchase flow.

Optional:
Launcher (depends on distribution channel).
Offline-capable with server sync checkpoints.

Note: Main threat is currency generation hacks and IAP bypass. Payment processor owns purchase validation. Backend receives webhook and issues entitlement. Client trusts nothing about its own economy state.

---

54. Tower Defense / Strategy

Backend need: Low to medium — leaderboard, save, optional multiplayer.

Required:
Encrypted PCK.
Custom export template.
Entitlement backend if paid game.

Optional:
Launcher for update delivery.
Cloud save for progress.
Leaderboard authority if competitive.

Note: Primary threat is piracy and save editing. Encrypted PCK plus optional signed saves handle casual attackers. Competitive leaderboard modes need backend score validation.

---

55. Horror / Narrative

Backend need: Low — account, save, entitlements.

Required:
Encrypted PCK.
Custom export template.
Entitlement backend if paid game.

Optional:
Launcher for updates.
Cloud save for progress.

Note: Asset protection is the primary concern. Encrypted PCK protects scene structure and narrative content from casual ripping. No real-time authority needed.

---

PART 8: IMPLEMENTATION ROADMAPS AND BUDGET TIERS

---

56. Solo Developer Roadmap

Day 0 to 30 — Foundation:
Set up monorepo structure.
Build vertical slice: launcher + token + GDExtension handshake.
Encrypted PCK export configured for production channel.
Backend: auth, session, and build validation endpoints only.

Day 30 to 60 — Core Authority:
Implement the most-cheated system first (economy, loot, or movement depending on genre).
Add telemetry packing in GDExtension.
Add CI/CD pipeline: build, forbidden-file scan, smoke test.
Write SECURITY_MODEL.md and BUILD_PIPELINE.md.

Day 60 to 90 — Polish and Release:
Delta PCK patching configured.
Release checklist completed.
Severity ladder for movement violations live.
RELEASE_CHECKLIST.md and BACKEND_AUTHORITY.md complete.

---

57. Small Team Roadmap (2 to 5 developers)

Developer 1 (Rust Launcher and Build Tools):
Launcher implementation.
Manifest system.
Patcher and downloader.
CI pipeline.
Delta PCK integration.

Developer 2 (Godot and GDExtension):
Game client.
GDExtension integration.
Export setup.
PCK encryption.

Developer 3 (Backend Auth and Sessions):
Auth service.
Session management.
Build validation.
Token system.

Developer 4 (Game Systems Backend):
Combat, inventory, quest, and economy authority services.
Transaction logs.
Cheat flag pipeline.

All developers:
Write docs during work.
Review security decisions.
Never ship without smoke tests passing.

---

58. Studio Roadmap (5 or more developers)

Add:
Dedicated security architect who reviews threat model and architecture decisions.
Separate backend team for game authority services.
Dedicated DevOps for CI/CD, secret management, and CDN.
External security review before first public beta.
Dedicated moderation and trust-and-safety team for cheat flag review.
Automated replay sampling and anomaly detection.
Formal incident response playbook.
Penetration test before launch.

---

59. Budget Architecture Tiers

Zero budget:
Backend: Fly.io free tier or Railway.
CDN: Bunny.net or GitHub Releases.
Auth: Custom JWT implementation.
Monitoring: tracing plus file logs.
Best for: Game jam or prototype.

Indie (20 to 100 USD per month):
Backend: Hetzner VPS plus managed PostgreSQL.
CDN: Cloudflare R2.
Auth: Custom Rust auth.
Monitoring: Grafana plus Loki.
Best for: Indie solo developer or small game.

Small studio (200 to 1000 USD per month):
Backend: AWS, GCP, or Azure VMs plus managed database.
CDN: CloudFront or Cloudflare.
Auth: Custom plus optional OAuth.
Monitoring: DataDog or Honeycomb.
Best for: Commercial indie title.

Studio (1000 USD per month and above):
Backend: Kubernetes plus managed services.
CDN: Global CDN with edge caching.
Auth: Full identity platform.
Monitoring: Full observability stack with tracing and alerting.
Best for: AA-scope or live service game.

---

60. Open Source Base Forks Guide

Rust HTTP backend: Fork or use axum examples. Add game-specific routes and auth middleware.
Rust launcher: Evaluate Tauri if UI needed. Strip to minimal security core then add manifest and patch logic.
Auth: JWT is simple enough to implement correctly. Use oxide-auth for OAuth if needed.
Game client networking: Use Godot's built-in ENet. Add packet signing and sequence numbers via GDExtension.
Observability: Use opentelemetry-rust. Add game-specific spans and cheat flag events.
CI templates: Copy GitHub Actions starter workflows. Add Godot export and forbidden-file scan steps.

---

PART 9: OBSERVABILITY AND OPERATIONS

---

61. Observability Stack

Structured logs:
Use tracing and tracing-subscriber in all Rust components.
Log all security events: auth, session creation, token validation, build ID checks.
Log all important state changes with trace_id and player_id.
Cloud options: Axiom, Papertrail, Grafana Loki.

Distributed tracing:
Use opentelemetry-rust with Jaeger or Tempo.
Trace requests across launcher, backend, and database.
Include trace_id in all log entries.
Cloud options: Honeycomb, Grafana Tempo.

Metrics:
Export to Prometheus.
Track: request rates, error rates, latency percentiles, cheat flag counts, auth failure rates.
Cloud options: DataDog, CloudWatch, Grafana Cloud.

Alerting:
Set thresholds for: auth failure rate, cheat flag rate, movement violation spikes.
Alert on: manifest signature failures, unknown build IDs, token reuse attempts.
Alert on: backend P99 latency above threshold, service down events.

---

62. Key Metrics and Alerting Thresholds

Auth failure rate above 5 percent: alert. Potential credential stuffing or bot attack.
Cheat flag rate above 0.1 percent of sessions: alert. Review if new build or real exploit.
Movement violation rate spike: alert. May indicate new speed hack tool or server bug.
Backend P99 latency above 200ms: alert. Impacts player experience and cheat detection accuracy.
PCK manifest signature failures above 0: alert immediately. Possible supply chain issue.
Token reuse attempts above 0: log and flag account. Someone is probing the system.
Unknown build ID requests above 0: alert. Old clients in production or spoofed requests.

---

63. Incident Response Playbook

Detect: Automated alert fires or human reports suspicious pattern.
Triage within 15 minutes: Is this a false positive? Active exploit? Infrastructure issue?
Contain within 1 hour: Revoke affected sessions. Invalidate build ID if compromised. Rate limit attack vector.
Investigate: Pull logs with trace_id. Replay attack sequence from telemetry. Identify root cause.
Remediate: Patch vulnerability. Push new build with new build ID. Regenerate signing keys if compromised.
Communicate: Inform affected players if necessary. Internal post-mortem.
Document: Add to SECURITY_AUDIT_LOG.md. Update threat model. Update risk register.

---

64. Platform Distribution Notes

Steam:
Steamworks handles launcher. You can still add backend token validation.
Steam can provide additional DRM layer if desired.
Godot exports work well with Steam distribution.

Epic Games Store:
EGS launcher handles distribution. Still add backend token validation.
EGS handles auth. Still add your own session and build validation.

itch.io:
Full Rust launcher recommended.
itch.io butler or your CDN for updates.
Full control — implement all security yourself.

Direct or self-hosted:
Rust launcher mandatory.
Your CDN plus manifest system for updates.
Full control and full responsibility.

Browser (itch.io web or self-hosted):
No launcher possible.
Page reload equals new version.
Server-auth only. No client protection possible.

---

PART 10: COMMUNITY, LEGAL, AND MASTER CHECKLIST

---

65. Modding Policy Framework

Define before release what the boundary is between legitimate modding and exploitation.

Cosmetic mods:
Allowed: Custom textures and UI skins for offline play.
Prohibited: Replacing game assets to gain competitive advantage.

Content mods:
Allowed: New levels, quests, and items for offline or designated servers.
Prohibited: Injecting cheat items into online economy.

Code mods:
Allowed: Scripted mod API if one is provided.
Prohibited: GDExtension injection into production builds.

Network mods:
Allowed: Private server mods if policy explicitly allows.
Prohibited: Modified clients connecting to official servers.

Save mods:
Allowed: Offline save editing. Document this openly and honestly.
Prohibited: Syncing edited offline saves to online progression.

---

66. Player Trust and Transparency

Anti-cheat that players can understand and trust is commercially better than invisible invasive systems.

Publish your anti-cheat philosophy publicly.
Explain what the launcher does in game documentation.
Publish what data the launcher and telemetry collect.
Provide an appeal process for false-positive bans.
Never claim the system is impossible to bypass. This is false and damages trust when proven wrong.
Communicate bans with a reason category, not just "cheating detected."
Do not use kernel-level anti-cheat unless you have a legal team and full support infrastructure for it.

---

67. Anti-Cheat Severity Scale by Genre

Singleplayer narrative:
Minimum viable: Encrypted PCK.
Recommended: Signed saves.
Maximum: Not needed.

Co-op PvE casual:
Minimum viable: Encrypted PCK plus backend economy authority.
Recommended: Add movement soft correction.
Maximum: Not needed.

PvP casual:
Minimum viable: Full backend auth plus movement validation.
Recommended: Add replay sampling.
Maximum: Add pattern detection.

PvP competitive or ranked:
Minimum viable: Full stack mandatory.
Recommended: Stricter movement bounds.
Maximum: Dedicated review team.

MMO or live service:
Minimum viable: Full stack mandatory.
Recommended: Economy anomaly detection.
Maximum: Full fraud operations team.

Extraction game:
Minimum viable: Full stack mandatory.
Recommended: Per-match session recording.
Maximum: Replay system mandatory.

---

68. The 10/10 Pre-Planning Master Checklist

Complete this checklist before writing any game code. Every item must have a documented answer or explicit rationale for skipping.

Architecture Decisions:
What game type? Offline, online, multiplayer, or competitive. Documented.
Which systems must be server-authoritative? Documented in BACKEND_AUTHORITY.md.
Is browser support required? Decision documented with impact analysis.
Will you use LibGodot or traditional launcher-plus-process? Decision documented.
Is offline play required? If yes, sync strategy documented.

Godot 4.6.x Setup:
Godot version pinned to 4.6.x.
PCK encryption key generated and stored in CI secrets only.
Custom export template built with encryption key.
Release export preset configured with no debug and no remote debug.
D3D12 set as Windows renderer for new projects.
Jolt physics configured. Migration guide consulted if migrating from 4.5.
Delta PCK patching enabled for patch builds.

Rust Components:
godot-rust/gdext pinned to specific version.
GDExtension built in release mode for shipped builds.
GDExtension API surface minimal and documented.
Launcher responsibilities documented including what it does not do.
Backend services identified and responsibilities documented.
Signed manifest schema v2 defined with delta patch support.

Security Architecture:
Launch token properties defined: TTL, single-use, binding fields.
Session rotation interval defined.
File hashing algorithm chosen.
Signing key algorithm chosen.
Anti-tamper behavior documented as non-malware.
Severity ladder defined for movement violations.
Transaction log requirements defined for economy events.

Build and Deploy:
CI pipeline planned with minimum required jobs.
Forbidden-file scan included in CI.
Smoke tests planned for vertical slice success criteria.
Secret management decided.
CDN for manifest and patch files identified.
Build channels defined.

Documentation:
SECURITY_MODEL.md skeleton created.
BUILD_PIPELINE.md skeleton created.
BACKEND_AUTHORITY.md skeleton created.
AGENT_TASKS.md skeleton created.
RELEASE_CHECKLIST.md skeleton created.
DELTA_PATCH_GUIDE.md skeleton created.

First Vertical Slice:
Vertical slice steps defined per Section 33.
Vertical slice success criteria documented.
Commitment that no feature work begins until slice passes all tests.
Team agrees: client is never trusted, server decides important outcomes.

---

End of Universal Godot + Rust Security Architecture Pre-Planning Blueprint v2.0
