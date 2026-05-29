Universal Godot + Rust Security Architecture Pre-Planning Blueprint v4.1 Re-Audited Master Edition

Version: 4.1 Re-Audited Master Edition
Target: Any Godot game — genre-agnostic — reusable — agency-grade — agent-ready
Updated: May 29, 2026 — re-audited and corrected
Base: v2.0 user-upgraded blueprint, fully preserved in breadth and corrected where necessary
Expansion goal: at least same-size-plus compared to v2, with practical code snippets, acceptance tests, CI/release gates, implementation handoff templates, and operational runbooks

Important note: This v4 edition intentionally does not compress v2. It keeps the planning breadth, corrects risky claims, and adds a large implementation layer so the document becomes a serious production pre-planning artifact instead of a shorter summary.

---

V4.1 RE-AUDIT PATCH NOTES

This v4.1 file is a corrected version of the v4.0 master edition. The re-audit focused on preventing silent technical drift, especially around Godot 4.6, PCK encryption, LibGodot, web export limitations, and godot-rust safeguard levels.

Corrections applied:
- Fixed stale wording that still said “v2.0 update” inside the v4 document.
- Fixed godot-rust safeguard tier guidance. Balanced is the default Release-build tier and should be the normal production baseline. Disengaged is not the default production recommendation; it is an exceptional performance escape hatch that can cause undefined behavior if the extension code is wrong.
- Removed language that made LibGodot sound like the default first implementation path. LibGodot is now framed as an advanced evaluation track after the launcher-plus-export vertical slice works.
- Added the Godot 4 Web/C# limitation warning so teams do not accidentally plan a C# browser client that cannot ship through the normal Godot 4 web export path.
- Added the Android export caveat for PCK encryption so teams do not assume desktop PCK encryption behavior automatically applies to Android packaging.
- Strengthened the wording around Delta PCK patching: it is a bandwidth/update-system feature and must still be wrapped in signed manifests, hash verification, atomic apply, and full-download fallback.
- Kept the file size and breadth intentionally above the v2 and v4 baselines; this is a correction and expansion pass, not a compression pass.

Non-negotiable result:
The architecture remains: Rust launcher or LibGodot host as entry/control layer, encrypted custom Godot export for casual ripping resistance, Rust GDExtension for selected helpers, and backend authority for all valuable online state. No client-side system in this document is treated as trusted authority.


---

Universal Godot + Rust Security Architecture Pre-Planning Blueprint v4.1 Re-Audited Master Edition

Version: 4.1 Re-Audited Master Edition
Target: Any Godot game — genre-agnostic — reusable — agent-ready
Updated: May 2026
Godot stable: 4.6.3
gdext: March 2026 update
Primary architecture: Rust launcher + custom encrypted Godot 4.6 export + Rust GDExtension (gdext) + server-authoritative Rust backend

Important security stance: This is not impossible-to-crack DRM. This is a practical production architecture that makes the client untrusted, moves valuable authority server-side, and adds enough packaging, validation, and tamper evidence to stop casual Godot decompiler users, noob crackers, basic PCK extractors, and simple asset rippers. Godot 4.6 adds LibGodot (embed Godot in Rust binary), native Delta PCK patching, Jolt physics as default, and GDExtension nullability — all integrated and re-audited in this v4.1 update.

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

Godot 4.6 adds LibGodot, which allows Godot to run as an embedded library inside another application. In a later advanced architecture, this can remove the separate Game.exe distribution pattern, but it should be evaluated after the normal launcher-plus-export pipeline is proven.
Godot 4.6 adds native Delta PCK patching, allowing patch files to include only changed portions of resources — dramatically reducing update bandwidth.

This structure applies to: singleplayer with online features, co-op PvE, multiplayer, MMO-style, competitive PvP, RPG, survival, extraction, browser demo, and commercial Godot desktop games.

---

2. Source-Aware Technical Reality Check

Godot 4.6 supports encrypted PCK export using a 256-bit AES key. This protects scenes, scripts, and resources from being stored in plain text and from being easily ripped by basic tools, but the encryption key must still exist in the binary. PCK encryption is useful resistance against casual ripping, not absolute protection.

Godot GDExtension is the official native extension mechanism for Godot 4. It lets a Godot project load native shared libraries and call into native code at runtime.

The godot-rust/gdext project is the practical Rust binding layer for Godot 4. As of March 2026, gdext supports three safeguard tiers (Strict for debug, Balanced for validation, Disengaged only for release after tests pass) and AnyArray/AnyDict/typed Dictionary types for better engine interop. Godot 4.6 adds nullability support to the GDExtension API.

Godot 4.6 specific additions affecting this architecture:
LibGodot: Godot can now run as a library embedded inside another application. Instead of a Rust launcher spawning a separate Game.exe process, an advanced Rust host can initialize Godot directly. Treat this as an evaluation track, not the default first milestone.
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
LibGodot (4.6 optional) can reduce or eliminate the separate game executable distribution pattern if the team accepts the extra integration risk.
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
Layer: Launcher+Client Combined Evaluation Track
Responsibilities: Embed/control Godot from a host application — can remove the separate Game.exe distribution pattern — tighter coupling but higher integration risk
Trust Level: Controls entry, but the resulting client remains untrusted because all client-side code is still inspectable by a determined attacker

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
Built in release mode with the default Balanced safeguard tier for shipped builds. Use Disengaged only for measured hotspots after tests prove the API usage is sound.
Small exported Godot-facing API surface.
Use AnyArray/AnyDict/typed Dictionary types for engine method interop where appropriate.
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
Android export caveat:
By default, Android exports store assets directly in the APK and are not affected by PCK encryption in the same way as desktop PCK distribution. If Android PCK encryption is required, plan and test APK expansion behavior explicitly. Do not assume the desktop encrypted-PCK pipeline automatically applies to Android.


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
Three safeguard tiers: Strict (default in Debug builds), Balanced (default in Release builds), and Disengaged (unsafe high-performance escape hatch).
AnyArray, AnyDict, and typed Dictionary for engine method interop.
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
Godot 4 Web/C# caveat:
Projects written in C# using Godot 4 currently cannot be exported to the web through the standard Godot web export path. If browser delivery is required, prefer GDScript/GDExtension-compatible client code for the web build, keep C# out of the browser target, or maintain a separate desktop-only C# client branch.


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
Rust release build with Balanced safeguard tier by default; Disengaged only for exceptional measured hotspots after review.
Small public Godot API surface.
AnyArray/AnyDict/typed Dictionary for engine interop.

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
Single host application initializes Godot.
No separate Game.exe is required in the ideal embedded distribution, but additional platform-specific packaging and runtime validation are required.
Token passed via internal function call (not on command line).
Harder for casual users to bypass because there is no obvious separate Game.exe entry point.
Godot runs inside the Rust process.

Security impact:
Removes the separate game executable as a bypass surface.
Tighter coupling between launcher security checks and game startup.
Token exchange happens inside process memory, not on visible command line.
Distribution is one binary plus PCK.

Note: LibGodot is new in Godot 4.6. Treat it as an advanced integration path. Use the traditional launcher-plus-export flow for the first protected vertical slice, then evaluate LibGodot only after the normal build, patch, token, and GDExtension handshake pipeline works reliably.

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

Strict (default in Debug builds):
Extra guardrails to detect as many bugs as possible during development.
Use for local development, early QA, and failure reproduction.
Do not ship normal production builds with Strict because it is optimized for guardrails, not final performance.

Balanced (default in Release builds):
Fast while still preserving safety behavior intended to keep runtime errors deterministic.
Use this as the default production setting for shipped GDExtension builds.
This is the recommended baseline for serious projects.

Disengaged:
Sacrifices safeguard behavior for raw speed.
Use only for the rare 1 percent of extensions with measured, proven, critical performance requirements.
Never enable Disengaged just to hide bugs or panics. It can turn mistakes into undefined behavior.

Cargo.toml for production GDExtension:
[profile.release]
opt-level = 3
lto = true
codegen-units = 1
strip = true

Never ship a debug-profile GDExtension to players.
Always cargo build --release for any binary that leaves development machines.
Release mode does not automatically mean Disengaged; keep Balanced unless a specific audited hotspot justifies changing it.

AnyArray, AnyDict, and typed Dictionary (March 2026 gdext):
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

End of preserved and corrected v2 base section. V4 expansion continues below.


---

PART 11: V4 AGENCY-GRADE EXPANSION AND AUDIT NOTES

---

69. V4 Audit Result and Upgrade Contract

This v4 master edition intentionally preserves the broad v2 planning surface and expands it instead of compressing it. The prior v3 draft was too small compared to v2 and therefore failed the “upgrade must be additive” standard for agency handoff. The v4 rule is: no original planning intelligence should be silently removed unless it is wrong, duplicated, unsafe, or replaced by a stronger version in the same document.

V4 upgrade goals:
- Preserve the full architecture breadth from v2.
- Correct risky or unstable claims with explicit caveats.
- Add implementation-ready Rust, Godot, backend, CI, and release snippets.
- Add agency-grade acceptance criteria.
- Add operational runbooks for production support.
- Add decision gates that stop agents from overbuilding or improvising.
- Add security test cases that can be automated.
- Add browser-specific guardrails.
- Add LibGodot evaluation criteria instead of treating it as the default.
- Add delta patch fallback behavior.
- Add concrete data models for manifests, sessions, tokens, logs, movement, inventory, and combat.

V4 quality bar:
A senior engineer should be able to use this as the planning backbone for a professional Godot project before implementation begins. A junior or external agent should be able to execute a single assigned step without guessing the architecture. A technical lead should be able to audit the design and identify where each security responsibility lives.

---

70. Source-Locked Technical Corrections

Correction 1: PCK encryption is useful but not absolute protection.
Godot's PCK encryption flow uses a 256-bit AES key and prevents plain-text storage of scenes, scripts, and resources in the exported PCK. The key still exists in the binary. This means encryption raises cost for casual rippers but does not defeat a determined reverse engineer. The plan must never describe PCK encryption as DRM or as impossible to extract.

Correction 2: Custom export templates are required for this protection path.
The export template must be built from source with the same key that is used for the encrypted export. A professional build pipeline must treat this as a release engineering task, not an editor checkbox.

Correction 3: godot-rust safeguard tiers must be treated as correctness gates.
Use Strict for development and failure reproduction. Use Balanced as the normal production release default. Use Disengaged only after profiling proves the need and a senior engineer reviews the unsafe tradeoff. Do not use lower safety settings as a way to hide warnings, panics, or unsafe API assumptions.

Correction 4: LibGodot is an advanced evaluation path.
Embedding Godot into a Rust binary can reduce the separate Game.exe bypass surface, but it also increases integration risk and deployment complexity. The default first implementation should remain traditional launcher plus exported game until the security vertical slice is proven. LibGodot should be evaluated only after the normal pipeline works.

Correction 5: Browser support is never equivalent to desktop protection.
Browser builds run in an environment where developer tools, network inspection, local storage inspection, and WebAssembly inspection are expected. Browser builds can be commercially useful, but security authority must live on the backend.

Correction 6: Delta patching reduces bandwidth, not security risk by itself.
Delta PCK patching must be wrapped in the same manifest, hash, signature, and fallback rules as full downloads. A corrupted delta path must fall back to a full verified PCK download.

Correction 7: Server authority is not optional for valuable online state.
Any plan that lets the client decide gold, premium currency, inventory, XP, combat result, quest reward, ranked result, or trade result is not production-grade.

---

71. Architecture Decision Record Template

Create one Architecture Decision Record for every major security decision. ADRs stop future agents from undoing important constraints because they do not understand the original reasoning.

File path:
docs/adr/ADR-0001-client-authority-model.md

Template:

# ADR-0001: Client Authority Model

Status: Accepted / Proposed / Rejected / Superseded
Date: YYYY-MM-DD
Owner: Technical Lead
Related systems: Launcher, Godot client, GDExtension, Backend

Context:
Describe the problem and threat model. Example: The client runs on an attacker-controlled machine, so local files and memory cannot be trusted.

Decision:
The client submits intent only. The backend validates and owns all valuable state transitions.

Alternatives considered:
1. Client-authoritative gameplay with telemetry only.
2. Hybrid authority with client damage calculation.
3. Server-authoritative model.

Decision outcome:
Accepted server authority because it is the only model that protects economy, combat, and progression against modified clients.

Consequences:
- Higher backend complexity.
- More latency management work.
- Stronger cheat resistance.
- More reliable audit logs.

Validation:
- Backend rejects client-submitted damage.
- Backend rejects client-submitted gold changes.
- Backend rejects reused launch token.

Review date:
YYYY-MM-DD or before first beta.

---

72. Trust Boundary Map

Trust Level 0 — Fully trusted only inside controlled infrastructure:
- Backend production services.
- Production database.
- Release signing machine or CI secret storage.
- Private signing keys.

Trust Level 1 — Trusted with constraints:
- CDN files only after manifest signature and file hash validation.
- Launcher binary only after code signing and version validation.
- Staging backend only for staging builds.

Trust Level 2 — Untrusted but useful:
- Godot client.
- Rust GDExtension inside player machine.
- Local manifest file after download.
- Local save files.
- Local config files.
- Command-line arguments.
- Browser local storage.

Trust Level 3 — Hostile:
- Modified game files.
- Modified launcher.
- Replayed packets.
- Manually edited saves.
- Memory-dumped values.
- Browser devtools manipulation.
- Any client-provided final result.

Trust boundary rule:
A value becomes trusted only after it is validated by the backend against authoritative state. The launcher and GDExtension can provide evidence, but only the backend can accept authority.

---

73. Full Production Data Flow

Flow A: Clean desktop launch
1. User opens launcher.
2. Launcher loads local config.
3. Launcher requests latest signed manifest metadata.
4. Launcher verifies manifest signature.
5. Launcher verifies launcher minimum version.
6. Launcher compares local files against manifest hashes.
7. Launcher repairs missing or changed files.
8. Launcher authenticates player.
9. Backend returns launcher session.
10. Launcher requests launch token for build ID and manifest hash.
11. Backend creates single-use token with 60 second TTL.
12. Launcher starts exported Godot game or embedded LibGodot runtime.
13. Godot loads startup scene.
14. Rust GDExtension validates arguments and calls backend.
15. Backend marks launch token used and creates game session.
16. Backend returns session token and connection endpoints.
17. Client enters main menu.
18. Client requests character list.
19. Backend returns only display-safe character state.
20. Gameplay begins with server-authoritative validation.

Flow B: Tampered file
1. User modifies game.pck.
2. Launcher hashes game.pck.
3. Hash mismatch occurs.
4. Launcher checks repair policy.
5. If repair enabled, launcher downloads verified full PCK or delta patch.
6. Launcher verifies downloaded artifact hash.
7. Launcher replaces bad file atomically.
8. Telemetry event records mismatch and repair.
9. If repair fails, launcher blocks start with clear error.

Flow C: Direct game launch
1. User starts Game.exe manually.
2. No valid launch token exists.
3. Godot startup scene loads Rust GDExtension.
4. GDExtension fails launch validation.
5. Protected online mode exits or displays launcher-required screen.
6. Offline mode may be allowed only if the product intentionally supports it.

Flow D: Token replay
1. Attacker copies launch token from process args.
2. Attacker tries to reuse token.
3. Backend checks token used flag.
4. Backend rejects request.
5. Telemetry records token replay attempt.
6. Account/session risk score increments.

---

74. Manifest Schema v4 With Delta, Platforms, and CDN Mirrors

The manifest must be deterministic, signed, and platform-aware. It must describe exactly what the launcher is allowed to install and run.

Example:

```json
{
  "schema_version": 4,
  "game_id": "your_game",
  "build_id": "2026.05.29.001",
  "channel": "production",
  "created_at": "2026-05-29T00:00:00Z",
  "godot_version": "4.6.3",
  "protocol_version": 2,
  "min_launcher_version": "0.2.0",
  "platforms": {
    "windows-x86_64": {
      "entrypoint": "Game.exe",
      "files": [
        {
          "path": "Game.exe",
          "role": "godot_executable",
          "hash_algorithm": "blake3",
          "hash": "BLAKE3_HEX",
          "size": 128000000,
          "required": true,
          "executable": true
        },
        {
          "path": "game.pck",
          "role": "encrypted_pck",
          "hash_algorithm": "blake3",
          "hash": "BLAKE3_HEX",
          "size": 2400000000,
          "required": true,
          "executable": false
        },
        {
          "path": "bin/rust_security_bridge.dll",
          "role": "gdextension",
          "hash_algorithm": "blake3",
          "hash": "BLAKE3_HEX",
          "size": 9000000,
          "required": true,
          "executable": false
        }
      ],
      "delta_patches": [
        {
          "from_build": "2026.05.20.001",
          "to_build": "2026.05.29.001",
          "target_file": "game.pck",
          "patch_file": "patches/windows/game_2026_05_20_to_2026_05_29.pckdelta",
          "from_hash": "OLD_BLAKE3_HEX",
          "to_hash": "NEW_BLAKE3_HEX",
          "patch_hash": "PATCH_BLAKE3_HEX",
          "patch_size": 45000000
        }
      ]
    }
  },
  "mirrors": [
    "https://cdn1.yourgame.com/builds/2026.05.29.001/",
    "https://cdn2.yourgame.com/builds/2026.05.29.001/"
  ],
  "signature_algorithm": "ed25519",
  "signature": "BASE64_SIGNATURE_OVER_CANONICAL_MANIFEST_WITHOUT_SIGNATURE"
}
```

Manifest signing rule:
Sign a canonical JSON representation of the manifest without the signature field. The launcher must verify this exact representation before trusting any file list, CDN URL, delta patch, or build metadata.

Manifest rejection rules:
- Reject unknown schema_version.
- Reject unsupported protocol_version.
- Reject channel mismatch.
- Reject min_launcher_version greater than current launcher version.
- Reject signature mismatch.
- Reject missing platform block.
- Reject executable paths outside install root.
- Reject absolute paths.
- Reject `..` path traversal.
- Reject files whose normalized path differs from manifest path.

---

75. Rust Manifest Types

```rust
use serde::{Deserialize, Serialize};
use std::collections::BTreeMap;

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ManifestV4 {
    pub schema_version: u32,
    pub game_id: String,
    pub build_id: String,
    pub channel: String,
    pub created_at: String,
    pub godot_version: String,
    pub protocol_version: u32,
    pub min_launcher_version: String,
    pub platforms: BTreeMap<String, PlatformManifest>,
    pub mirrors: Vec<String>,
    pub signature_algorithm: String,
    pub signature: String,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PlatformManifest {
    pub entrypoint: String,
    pub files: Vec<FileEntry>,
    #[serde(default)]
    pub delta_patches: Vec<DeltaPatchEntry>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct FileEntry {
    pub path: String,
    pub role: String,
    pub hash_algorithm: String,
    pub hash: String,
    pub size: u64,
    pub required: bool,
    pub executable: bool,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct DeltaPatchEntry {
    pub from_build: String,
    pub to_build: String,
    pub target_file: String,
    pub patch_file: String,
    pub from_hash: String,
    pub to_hash: String,
    pub patch_hash: String,
    pub patch_size: u64,
}
```

Why BTreeMap:
BTreeMap gives deterministic key ordering when serializing canonical JSON. Determinism matters for reproducible signing and verification.

---

76. Rust Manifest Signature Verification Example

```rust
use anyhow::{anyhow, Context, Result};
use base64::{engine::general_purpose, Engine as _};
use ed25519_dalek::{Signature, Verifier, VerifyingKey};
use serde_json::Value;

/// Verifies a signed manifest using an embedded Ed25519 public key.
///
/// Security notes:
/// - Do not trust any field until signature verification succeeds.
/// - Sign canonical JSON with the `signature` field removed.
/// - Do not allow fallback to unsigned manifests in production.
pub fn verify_manifest_signature(
    manifest_json: &str,
    public_key_bytes: &[u8; 32],
) -> Result<()> {
    let mut value: Value = serde_json::from_str(manifest_json)
        .context("manifest is not valid JSON")?;

    let signature_b64 = value
        .get("signature")
        .and_then(|v| v.as_str())
        .ok_or_else(|| anyhow!("manifest missing signature"))?
        .to_owned();

    value
        .as_object_mut()
        .ok_or_else(|| anyhow!("manifest root must be object"))?
        .remove("signature");

    let canonical = serde_json::to_vec(&value)
        .context("failed to serialize canonical manifest JSON")?;

    let signature_bytes = general_purpose::STANDARD
        .decode(signature_b64)
        .context("manifest signature is not base64")?;

    let signature = Signature::from_slice(&signature_bytes)
        .context("manifest signature has invalid length")?;

    let verifying_key = VerifyingKey::from_bytes(public_key_bytes)
        .context("invalid embedded public key")?;

    verifying_key
        .verify(&canonical, &signature)
        .context("manifest signature verification failed")?;

    Ok(())
}
```

Agent instruction:
Do not ship this code without a unit test that proves a modified manifest fails verification.

Required tests:
- Valid manifest verifies.
- Changing file hash breaks verification.
- Changing CDN mirror breaks verification.
- Removing signature fails.
- Invalid base64 signature fails.

---

77. Secure Path Normalization for Launcher File Writes

The launcher must never write files outside the install directory, even if a malicious or corrupted manifest contains path traversal.

```rust
use anyhow::{anyhow, Context, Result};
use std::path::{Component, Path, PathBuf};

/// Converts a manifest path into a safe absolute path inside install_root.
///
/// Rejects:
/// - absolute paths
/// - parent directory traversal
/// - Windows prefix paths
/// - empty paths
pub fn resolve_manifest_path(install_root: &Path, manifest_path: &str) -> Result<PathBuf> {
    let rel = Path::new(manifest_path);

    if rel.as_os_str().is_empty() {
        return Err(anyhow!("manifest path is empty"));
    }
    if rel.is_absolute() {
        return Err(anyhow!("absolute paths are forbidden: {manifest_path}"));
    }

    for component in rel.components() {
        match component {
            Component::Normal(_) => {}
            Component::CurDir => {}
            Component::ParentDir => return Err(anyhow!("path traversal forbidden: {manifest_path}")),
            Component::RootDir | Component::Prefix(_) => {
                return Err(anyhow!("root or prefix paths forbidden: {manifest_path}"));
            }
        }
    }

    Ok(install_root.join(rel))
}
```

Professional requirement:
This function must be used everywhere the launcher reads, writes, hashes, patches, deletes, or executes a manifest file path.

---

78. Hash Verification and Atomic Replace

```rust
use anyhow::{anyhow, Context, Result};
use blake3::Hasher;
use std::fs::{self, File};
use std::io::{Read, Write};
use std::path::Path;

pub fn blake3_file_hex(path: &Path) -> Result<String> {
    let mut file = File::open(path)
        .with_context(|| format!("failed to open file for hashing: {}", path.display()))?;

    let mut hasher = Hasher::new();
    let mut buf = [0u8; 1024 * 64];

    loop {
        let n = file.read(&mut buf)?;
        if n == 0 { break; }
        hasher.update(&buf[..n]);
    }

    Ok(hasher.finalize().to_hex().to_string())
}

pub fn verify_hash(path: &Path, expected_hex: &str) -> Result<()> {
    let actual = blake3_file_hex(path)?;
    if actual != expected_hex {
        return Err(anyhow!(
            "hash mismatch for {}: expected {}, got {}",
            path.display(), expected_hex, actual
        ));
    }
    Ok(())
}

/// Writes a downloaded file atomically: temp file -> hash verify -> rename.
/// This prevents partially downloaded files from being treated as installed.
pub fn atomic_install(downloaded_temp: &Path, final_path: &Path, expected_hash: &str) -> Result<()> {
    verify_hash(downloaded_temp, expected_hash)?;

    if let Some(parent) = final_path.parent() {
        fs::create_dir_all(parent)?;
    }

    fs::rename(downloaded_temp, final_path)
        .or_else(|_| {
            // Windows can fail rename if target exists.
            let backup = final_path.with_extension("old");
            if final_path.exists() {
                let _ = fs::remove_file(&backup);
                fs::rename(final_path, &backup)?;
            }
            fs::rename(downloaded_temp, final_path)?;
            let _ = fs::remove_file(&backup);
            Ok::<(), std::io::Error>(())
        })
        .context("atomic install failed")?;

    Ok(())
}
```

Acceptance tests:
- Partial download never replaces valid file.
- Wrong hash file is deleted or quarantined.
- Target parent folder is created.
- Existing file replacement works on Windows.

---

79. Launcher Patch Strategy With Delta Fallback

```rust
#[derive(Debug, Clone)]
pub enum PatchStrategy {
    AlreadyCurrent,
    Delta { patch_url: String, patch_hash: String, target_hash: String },
    Full { file_url: String, file_hash: String },
}

/// Chooses delta patch only when the current local file exactly matches the delta source hash.
/// If not, fall back to full download.
pub fn choose_pck_patch_strategy(
    current_build_id: Option<&str>,
    current_pck_hash: Option<&str>,
    desired_pck_hash: &str,
    manifest: &ManifestV4,
    platform: &str,
) -> PatchStrategy {
    if current_pck_hash == Some(desired_pck_hash) {
        return PatchStrategy::AlreadyCurrent;
    }

    if let (Some(from_build), Some(local_hash)) = (current_build_id, current_pck_hash) {
        if let Some(platform_manifest) = manifest.platforms.get(platform) {
            for patch in &platform_manifest.delta_patches {
                if patch.from_build == from_build
                    && patch.to_build == manifest.build_id
                    && patch.from_hash == local_hash
                    && patch.to_hash == desired_pck_hash
                {
                    return PatchStrategy::Delta {
                        patch_url: patch.patch_file.clone(),
                        patch_hash: patch.patch_hash.clone(),
                        target_hash: patch.to_hash.clone(),
                    };
                }
            }
        }
    }

    PatchStrategy::Full {
        file_url: "game.pck".to_string(),
        file_hash: desired_pck_hash.to_string(),
    }
}
```

Delta patch rule:
A delta patch is an optimization, not a trust shortcut. The launcher must verify the delta patch hash before applying it and verify the resulting target PCK hash after applying it. If either check fails, the launcher must fall back to full download.

---

80. Backend Token Table Design

PostgreSQL schema:

```sql
CREATE TABLE launch_tokens (
    token_hash TEXT PRIMARY KEY,
    account_id UUID NOT NULL,
    build_id TEXT NOT NULL,
    channel TEXT NOT NULL,
    manifest_hash TEXT NOT NULL,
    issued_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    expires_at TIMESTAMPTZ NOT NULL,
    used_at TIMESTAMPTZ,
    launcher_version TEXT NOT NULL,
    launcher_session_id UUID NOT NULL,
    ip_hash TEXT,
    user_agent_hash TEXT
);

CREATE INDEX idx_launch_tokens_account_id ON launch_tokens(account_id);
CREATE INDEX idx_launch_tokens_expires_at ON launch_tokens(expires_at);

CREATE TABLE game_sessions (
    session_id UUID PRIMARY KEY,
    account_id UUID NOT NULL,
    build_id TEXT NOT NULL,
    channel TEXT NOT NULL,
    manifest_hash TEXT NOT NULL,
    protocol_version INTEGER NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    rotated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    revoked_at TIMESTAMPTZ,
    revoke_reason TEXT
);
```

Security notes:
- Store token hashes, not raw tokens.
- Use constant-time comparison where applicable.
- Mark token as used inside the same database transaction that creates the game session.
- Never allow a token to create more than one game session.

---

81. Backend Launch Token Consume Flow

```rust
use anyhow::{anyhow, Result};
use sqlx::{PgPool, Postgres, Transaction};
use uuid::Uuid;

pub struct LaunchContext {
    pub raw_token: String,
    pub build_id: String,
    pub manifest_hash: String,
    pub protocol_version: i32,
}

pub struct GameSession {
    pub session_id: Uuid,
    pub account_id: Uuid,
}

pub async fn consume_launch_token(
    pool: &PgPool,
    ctx: LaunchContext,
) -> Result<GameSession> {
    let token_hash = hash_token(&ctx.raw_token);
    let mut tx: Transaction<'_, Postgres> = pool.begin().await?;

    let row = sqlx::query!(
        r#"
        SELECT account_id, build_id, manifest_hash, expires_at, used_at
        FROM launch_tokens
        WHERE token_hash = $1
        FOR UPDATE
        "#,
        token_hash
    )
    .fetch_optional(&mut *tx)
    .await?
    .ok_or_else(|| anyhow!("invalid launch token"))?;

    if row.used_at.is_some() {
        return Err(anyhow!("launch token already used"));
    }
    if row.expires_at < chrono::Utc::now() {
        return Err(anyhow!("launch token expired"));
    }
    if row.build_id != ctx.build_id {
        return Err(anyhow!("build ID mismatch"));
    }
    if row.manifest_hash != ctx.manifest_hash {
        return Err(anyhow!("manifest hash mismatch"));
    }

    sqlx::query!(
        "UPDATE launch_tokens SET used_at = now() WHERE token_hash = $1",
        token_hash
    )
    .execute(&mut *tx)
    .await?;

    let session_id = Uuid::new_v4();
    sqlx::query!(
        r#"
        INSERT INTO game_sessions (
            session_id, account_id, build_id, channel, manifest_hash, protocol_version
        ) VALUES ($1, $2, $3, 'production', $4, $5)
        "#,
        session_id,
        row.account_id,
        ctx.build_id,
        ctx.manifest_hash,
        ctx.protocol_version
    )
    .execute(&mut *tx)
    .await?;

    tx.commit().await?;

    Ok(GameSession {
        session_id,
        account_id: row.account_id,
    })
}

fn hash_token(token: &str) -> String {
    blake3::hash(token.as_bytes()).to_hex().to_string()
}
```

Why `FOR UPDATE`:
It prevents two simultaneous validation requests from using the same token. Without it, a race condition could create two sessions from one token.

---

82. GDExtension Bootstrap API Contract

Godot-callable Rust API should be small, stable, and documented.

GDExtension class:
RustSecurityBridge

Methods:
- validate_launch_args(args: PackedStringArray) -> Dictionary
- create_handshake_payload() -> Dictionary
- consume_launch_token_with_backend(endpoint: String) -> Dictionary
- get_build_fingerprint() -> String
- get_protocol_version() -> int
- make_combat_request(data: Dictionary) -> PackedByteArray
- make_inventory_request(data: Dictionary) -> PackedByteArray
- submit_telemetry_event(data: Dictionary) -> bool

Result Dictionary standard:

```gdscript
{
    "ok": true,
    "code": "OK",
    "message": "validated",
    "session_id": "..."
}
```

Error example:

```gdscript
{
    "ok": false,
    "code": "LAUNCH_TOKEN_EXPIRED",
    "message": "Launch session expired. Please restart from the launcher."
}
```

User-facing rule:
The client can display clean, non-technical errors. The backend logs contain the exact reason. Do not expose internal token, hash, database, or infrastructure details to users.

---

83. GDScript Startup Guard

```gdscript
extends Node

@onready var bridge := RustSecurityBridge.new()

func _ready() -> void:
    var result: Dictionary = bridge.validate_launch_args(OS.get_cmdline_args())

    if not result.get("ok", false):
        _fail_startup(result)
        return

    var backend_result: Dictionary = bridge.consume_launch_token_with_backend(
        ProjectSettings.get_setting("application/security/backend_url")
    )

    if not backend_result.get("ok", false):
        _fail_startup(backend_result)
        return

    GlobalSession.session_id = backend_result.get("session_id", "")
    get_tree().change_scene_to_file("res://scenes/MainMenu.tscn")

func _fail_startup(result: Dictionary) -> void:
    var code := str(result.get("code", "UNKNOWN_ERROR"))
    var message := str(result.get("message", "Protected startup failed."))

    push_error("Startup failed [%s]: %s" % [code, message])

    # Use a small scene that tells the player to restart from launcher.
    GlobalStartupError.code = code
    GlobalStartupError.message = message
    get_tree().change_scene_to_file("res://scenes/errors/StartupError.tscn")
```

Acceptance test:
- No args: startup error scene.
- Expired token: startup error scene.
- Valid token: main menu.
- Backend offline: clear retry message.

---

84. Rust GDExtension Skeleton Example

```rust
use godot::prelude::*;
use godot::classes::Object;

#[derive(GodotClass)]
#[class(base=Object)]
pub struct RustSecurityBridge {
    base: Base<Object>,
}

#[godot_api]
impl IObject for RustSecurityBridge {
    fn init(base: Base<Object>) -> Self {
        Self { base }
    }
}

#[godot_api]
impl RustSecurityBridge {
    #[func]
    pub fn get_protocol_version(&self) -> i64 {
        2
    }

    #[func]
    pub fn get_build_fingerprint(&self) -> GString {
        // Build-time generated value injected by CI or build.rs.
        GString::from(env!("BUILD_FINGERPRINT"))
    }

    #[func]
    pub fn validate_launch_args(&self, args: PackedStringArray) -> Dictionary {
        let parsed = parse_launch_args(args);
        match parsed {
            Ok(ctx) => ok_dict_with_context(ctx),
            Err(err) => error_dict("INVALID_LAUNCH_ARGS", &err.to_string()),
        }
    }
}

fn error_dict(code: &str, message: &str) -> Dictionary {
    let mut dict = Dictionary::new();
    dict.set("ok", false);
    dict.set("code", code);
    dict.set("message", message);
    dict
}

fn ok_dict_with_context(_ctx: LaunchArgs) -> Dictionary {
    let mut dict = Dictionary::new();
    dict.set("ok", true);
    dict.set("code", "OK");
    dict.set("message", "launch args parsed");
    dict
}

struct LaunchArgs {
    build_id: String,
    launch_token: String,
    manifest_hash: String,
}

fn parse_launch_args(args: PackedStringArray) -> anyhow::Result<LaunchArgs> {
    let mut build_id = None;
    let mut launch_token = None;
    let mut manifest_hash = None;

    let vec: Vec<String> = args.iter_shared().map(|s| s.to_string()).collect();
    let mut iter = vec.iter();

    while let Some(arg) = iter.next() {
        match arg.as_str() {
            "--build-id" => build_id = iter.next().cloned(),
            "--launch-token" => launch_token = iter.next().cloned(),
            "--manifest-hash" => manifest_hash = iter.next().cloned(),
            _ => {}
        }
    }

    Ok(LaunchArgs {
        build_id: build_id.ok_or_else(|| anyhow::anyhow!("missing --build-id"))?,
        launch_token: launch_token.ok_or_else(|| anyhow::anyhow!("missing --launch-token"))?,
        manifest_hash: manifest_hash.ok_or_else(|| anyhow::anyhow!("missing --manifest-hash"))?,
    })
}
```

Professional notes:
- This parses arguments but does not trust them.
- Real validation happens against backend.
- Do not log the launch token.
- Use redaction middleware in logs.

---

85. Cargo Workspace Layout

```toml
[workspace]
members = [
    "launcher",
    "gdextension",
    "backend",
    "shared",
    "build_tools"
]
resolver = "2"

[workspace.package]
edition = "2024"
license = "Proprietary"
rust-version = "1.85"

[workspace.dependencies]
anyhow = "1"
thiserror = "1"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
tokio = { version = "1", features = ["full"] }
reqwest = { version = "0.12", features = ["json", "rustls-tls"] }
tracing = "0.1"
tracing-subscriber = "0.3"
blake3 = "1"
ed25519-dalek = "2"
base64 = "0.22"
uuid = { version = "1", features = ["v4", "serde"] }
chrono = { version = "0.4", features = ["serde"] }
```

Release profiles:

```toml
[profile.release]
opt-level = 3
lto = true
codegen-units = 1
strip = true
panic = "abort"
```

Rule:
Debug builds never ship to players. Release builds always pass smoke tests before packaging.

---

86. Backend Route Sketch With Axum

```rust
use axum::{extract::State, routing::post, Json, Router};
use serde::{Deserialize, Serialize};
use std::sync::Arc;

#[derive(Clone)]
pub struct AppState {
    pub db: sqlx::PgPool,
    pub accepted_builds: Arc<AcceptedBuilds>,
}

pub fn routes(state: AppState) -> Router {
    Router::new()
        .route("/v1/auth/login", post(login))
        .route("/v1/launcher/launch-token", post(issue_launch_token))
        .route("/v1/session/consume-launch-token", post(consume_launch_token_route))
        .route("/v1/game/combat/intent", post(combat_intent))
        .route("/v1/game/inventory/action", post(inventory_action))
        .with_state(state)
}

#[derive(Debug, Deserialize)]
pub struct ConsumeLaunchTokenRequest {
    pub build_id: String,
    pub manifest_hash: String,
    pub protocol_version: i32,
    pub launch_token: String,
}

#[derive(Debug, Serialize)]
pub struct ConsumeLaunchTokenResponse {
    pub ok: bool,
    pub session_id: Option<String>,
    pub code: String,
    pub message: String,
}

async fn consume_launch_token_route(
    State(state): State<AppState>,
    Json(req): Json<ConsumeLaunchTokenRequest>,
) -> Json<ConsumeLaunchTokenResponse> {
    let ctx = LaunchContext {
        raw_token: req.launch_token,
        build_id: req.build_id,
        manifest_hash: req.manifest_hash,
        protocol_version: req.protocol_version,
    };

    match consume_launch_token(&state.db, ctx).await {
        Ok(session) => Json(ConsumeLaunchTokenResponse {
            ok: true,
            session_id: Some(session.session_id.to_string()),
            code: "OK".to_string(),
            message: "session created".to_string(),
        }),
        Err(_) => Json(ConsumeLaunchTokenResponse {
            ok: false,
            session_id: None,
            code: "SESSION_REJECTED".to_string(),
            message: "Could not validate protected game session.".to_string(),
        }),
    }
}
```

Error-message rule:
Return stable public error codes. Log internal details privately. Do not reveal whether token, manifest, account, or build ID was the exact failure reason to attackers.

---

87. Combat Intent Request Model

```rust
use serde::{Deserialize, Serialize};
use uuid::Uuid;

#[derive(Debug, Clone, Deserialize)]
pub struct CombatIntentRequest {
    pub session_id: Uuid,
    pub character_id: Uuid,
    pub target_id: Uuid,
    pub spell_id: u32,
    pub input_sequence: u64,
    pub client_timestamp_ms: i64,
    pub client_position: Vec3Wire,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Vec3Wire {
    pub x: f32,
    pub y: f32,
    pub z: f32,
}

#[derive(Debug, Clone, Serialize)]
pub struct CombatIntentResponse {
    pub accepted: bool,
    pub input_sequence: u64,
    pub result_code: String,
    pub damage: Option<i32>,
    pub target_health_after: Option<i32>,
    pub server_timestamp_ms: i64,
}
```

Server must ignore:
- Client damage.
- Client crit result.
- Client hit result.
- Client XP grant.
- Client loot outcome.

Server may use:
- Client input sequence for reconciliation.
- Client position as a claim to validate against server position.
- Client timestamp only for tolerance checks, not authority.

---

88. Movement Validation Function

```rust
#[derive(Debug, Clone, Copy)]
pub struct MovementSample {
    pub x: f32,
    pub y: f32,
    pub z: f32,
    pub timestamp_ms: i64,
}

#[derive(Debug, Clone, Copy)]
pub struct MovementRules {
    pub max_speed_mps: f32,
    pub grace_multiplier: f32,
    pub max_vertical_delta_per_sec: f32,
}

#[derive(Debug, Clone)]
pub enum MovementDecision {
    Accept,
    Correct { server_x: f32, server_y: f32, server_z: f32 },
    Reject { reason: String },
    Flag { severity: u8, reason: String },
}

pub fn validate_movement(
    previous: MovementSample,
    claimed: MovementSample,
    rules: MovementRules,
) -> MovementDecision {
    let dt = (claimed.timestamp_ms - previous.timestamp_ms) as f32 / 1000.0;
    if dt <= 0.0 || dt > 10.0 {
        return MovementDecision::Reject { reason: "invalid movement timestamp delta".to_string() };
    }

    let dx = claimed.x - previous.x;
    let dy = claimed.y - previous.y;
    let dz = claimed.z - previous.z;
    let horizontal_distance = (dx * dx + dz * dz).sqrt();
    let allowed = rules.max_speed_mps * dt * rules.grace_multiplier;

    if horizontal_distance > allowed * 2.0 {
        return MovementDecision::Flag {
            severity: 4,
            reason: format!("large movement violation: {horizontal_distance:.2}m allowed {allowed:.2}m"),
        };
    }

    if horizontal_distance > allowed {
        return MovementDecision::Correct {
            server_x: previous.x,
            server_y: previous.y,
            server_z: previous.z,
        };
    }

    let vertical_speed = dy.abs() / dt;
    if vertical_speed > rules.max_vertical_delta_per_sec {
        return MovementDecision::Flag {
            severity: 3,
            reason: format!("vertical movement violation: {vertical_speed:.2}m/s"),
        };
    }

    MovementDecision::Accept
}
```

Professional note:
Movement validation must be tuned per genre. A platformer, MMO, shooter, and physics game require different tolerances. Use telemetry to tune thresholds before punishing players.

---

89. Inventory Transaction SQL Pattern

```sql
BEGIN;

-- Lock the player inventory row so two simultaneous requests cannot duplicate or consume the same item twice.
SELECT quantity
FROM inventory_items
WHERE character_id = $1 AND item_id = $2
FOR UPDATE;

-- Validate quantity and item rules in application logic.
-- Then apply state change atomically.

UPDATE inventory_items
SET quantity = quantity - 1
WHERE character_id = $1 AND item_id = $2 AND quantity >= 1;

INSERT INTO inventory_audit_log (
    event_id,
    character_id,
    item_id,
    delta_quantity,
    reason,
    source_session_id,
    created_at
) VALUES (
    gen_random_uuid(),
    $1,
    $2,
    -1,
    'consume_item',
    $3,
    now()
);

COMMIT;
```

Duplication prevention rule:
Every item creation, deletion, transfer, trade, mail, crafting, vendor, auction, and loot operation must lock the relevant rows and write an audit event in the same transaction.

---

90. Economy Ledger Model

For MMO, extraction, co-op loot, or premium currency games, use a ledger instead of only storing balances.

```sql
CREATE TABLE currency_ledger (
    ledger_id UUID PRIMARY KEY,
    account_id UUID NOT NULL,
    character_id UUID,
    currency_type TEXT NOT NULL,
    delta_amount BIGINT NOT NULL,
    balance_after BIGINT NOT NULL,
    reason TEXT NOT NULL,
    source_event_id UUID,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_currency_ledger_account ON currency_ledger(account_id, created_at DESC);
CREATE INDEX idx_currency_ledger_reason ON currency_ledger(reason);
```

Why ledger:
- Supports fraud investigation.
- Supports rollback.
- Supports compensation.
- Supports economy anomaly detection.
- Makes duplication bugs easier to trace.

Rule:
Never update premium currency without a ledger entry.

---

91. Forbidden File Scanner

The release pipeline must fail if public packages contain dev artifacts.

```rust
use anyhow::{anyhow, Result};
use walkdir::WalkDir;
use std::path::Path;

const FORBIDDEN_NAMES: &[&str] = &[
    ".env",
    ".godot",
    ".git",
    "id_rsa",
    "private_key",
    "signing_key",
    "debug.pdb",
];

const FORBIDDEN_EXTENSIONS: &[&str] = &[
    "pdb", "dSYM", "pem", "key", "env", "ps1"
];

pub fn scan_release_folder(root: &Path) -> Result<()> {
    let mut violations = Vec::new();

    for entry in WalkDir::new(root).into_iter().filter_map(Result::ok) {
        let path = entry.path();
        let name = path.file_name().and_then(|n| n.to_str()).unwrap_or("");
        let ext = path.extension().and_then(|e| e.to_str()).unwrap_or("");

        if FORBIDDEN_NAMES.iter().any(|forbidden| name.eq_ignore_ascii_case(forbidden)) {
            violations.push(path.display().to_string());
        }
        if FORBIDDEN_EXTENSIONS.iter().any(|forbidden| ext.eq_ignore_ascii_case(forbidden)) {
            violations.push(path.display().to_string());
        }
    }

    if !violations.is_empty() {
        return Err(anyhow!("release folder contains forbidden files: {violations:#?}"));
    }

    Ok(())
}
```

Must fail build if found:
- .env
- private keys
- signing keys
- debug symbols
- raw dev scenes not intended for release
- .godot folder
- raw imported source dumps
- database dumps
- test credentials

---

92. GitHub Actions CI Example

```yaml
name: release-candidate

on:
  workflow_dispatch:
  push:
    tags:
      - "v*"

jobs:
  rust-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - run: cargo fmt --check
      - run: cargo clippy --workspace --all-targets -- -D warnings
      - run: cargo test --workspace

  build-gdextension:
    needs: rust-tests
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - run: cargo build -p rust_security_bridge --release
      - run: mkdir -p godot_project/bin
      - run: copy target\release\rust_security_bridge.dll godot_project\bin\rust_security_bridge.dll
      - uses: actions/upload-artifact@v4
        with:
          name: rust_security_bridge_windows
          path: godot_project/bin/rust_security_bridge.dll

  package-release:
    needs: build-gdextension
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
      - name: Export Godot project
        run: |
          godot --headless --path godot_project --export-release "Windows Desktop" dist/Game.exe
      - name: Scan release folder
        run: cargo run -p build_tools --bin validate_release_folder -- dist
      - name: Generate manifest
        run: cargo run -p build_tools --bin generate_manifest -- dist manifest.json
      - name: Sign manifest
        env:
          RELEASE_SIGNING_KEY: ${{ secrets.RELEASE_SIGNING_KEY }}
        run: cargo run -p build_tools --bin sign_manifest -- manifest.json
      - name: Verify manifest
        run: cargo run -p build_tools --bin verify_manifest -- manifest.json
      - uses: actions/upload-artifact@v4
        with:
          name: release_candidate
          path: dist
```

CI hard rule:
Do not use production secrets on pull requests from forks. Do not print signing keys. Do not upload secret-bearing intermediate files.

---

93. Release Gate Checklist With Sign-Off Roles

Release cannot proceed until all rows are signed.

| Gate | Owner | Required Evidence | Pass/Fail |
|---|---|---|---|
| Godot export release mode | Godot lead | export log + preset | |
| PCK encryption enabled | Release engineer | export preset + smoke test | |
| Custom template used | Release engineer | template hash + build log | |
| GDExtension release build | Rust lead | cargo build --release log | |
| Backend accepts build ID | Backend lead | DB row + endpoint test | |
| Manifest signed | Release engineer | manifest + verify command output | |
| Forbidden-file scan passed | CI owner | CI artifact | |
| Direct Game.exe rejected | QA | smoke test video/log | |
| Token reuse rejected | Backend QA | automated test output | |
| Modified PCK repaired/rejected | Launcher QA | automated test output | |
| Browser policy reviewed | Product owner | BROWSER_POLICY.md | |
| Legal/privacy telemetry review | Producer/legal | privacy checklist | |

No one person should sign off every gate for a serious production build.

---

94. Security Smoke Test Matrix

| Test ID | Scenario | Expected Result | Automated? |
|---|---|---|---|
| SEC-001 | Clean install launch via launcher | Main menu opens | Yes |
| SEC-002 | Delete game.pck | Launcher redownloads or blocks | Yes |
| SEC-003 | Modify game.pck byte | Launcher repairs or blocks | Yes |
| SEC-004 | Modify GDExtension DLL | Launcher repairs or blocks | Yes |
| SEC-005 | Launch Game.exe directly | Protected online mode blocked | Yes |
| SEC-006 | Reuse launch token | Backend rejects | Yes |
| SEC-007 | Expired launch token | Backend rejects | Yes |
| SEC-008 | Unknown build ID | Backend rejects | Yes |
| SEC-009 | Invalid manifest signature | Launcher rejects | Yes |
| SEC-010 | Path traversal in manifest | Launcher rejects | Yes |
| SEC-011 | Missing GDExtension | Startup error | Yes |
| SEC-012 | Client sends fake damage | Backend rejects/ignores | Yes |
| SEC-013 | Client sends fake gold | Backend rejects | Yes |
| SEC-014 | Client moves too far | Correction or flag | Yes |
| SEC-015 | Browser client grants item locally | Backend state unchanged | Yes |

---

95. Game System Authority Matrix Template

Use this table before implementation.

| System | Client Can Display | Client Can Predict | Client Can Request | Server Must Own | Notes |
|---|---|---|---|---|---|
| Movement | Yes | Yes | Yes | Final sanity/correction | Tune by genre |
| Combat animation | Yes | Yes | Yes | Final result | Client can show anticipation |
| Damage | Yes | No | No | Yes | Server calculates |
| Loot | Yes | No | Open/loot intent | Yes | Server rolls/generates |
| Inventory | Yes | No | Use/move/equip intent | Yes | Transactional DB |
| Gold | Yes | No | Spend intent | Yes | Ledger required |
| Premium currency | Yes | No | Purchase claim via backend only | Yes | Payment webhooks |
| Quest text | Yes | N/A | Accept/turn-in intent | State/reward | Client displays narrative |
| Achievements | Yes | Maybe cosmetic local progress | Unlock intent only | Yes | Server unlocks |
| Leaderboards | Yes | No | Submit score only with validation | Yes | Validate score path |
| Chat | Yes | No | Send message | Moderation/rate limits | Store/audit as needed |

---

96. Browser Build Security Contract

The browser build is allowed only if all of these are true:
- No permanent secrets in JavaScript, WASM, HTML, local storage, IndexedDB, or PCK.
- All valuable state is backend-owned.
- Browser build cannot access production-only desktop features unless explicitly allowed.
- Browser build uses a separate build ID and channel.
- Backend can identify browser sessions separately.
- Browser build is rate-limited more aggressively.
- Browser build has feature flags for restricted economy/trading if needed.

Browser feature policy template:

| Feature | Browser Allowed? | Reason | Backend Required? |
|---|---:|---|---:|
| Account login | Yes | User access | Yes |
| Character preview | Yes | Low risk | Yes |
| Full inventory trading | Maybe/No | Economy risk | Yes |
| Premium purchases | Yes, via secure backend checkout | Commerce | Yes |
| Ranked PvP | Usually No at first | High cheat risk | Yes |
| Demo zone | Yes | Marketing | Yes for progression |
| Local graphics settings | Yes | Client-only | No |

---

97. LibGodot Evaluation Checklist

Use LibGodot only after the traditional protected launch slice works.

Evaluate:
- Does the Rust embedding API support all target platforms?
- Can the team debug crashes inside a combined process?
- Can crash reports distinguish Rust launcher, Godot runtime, and GDExtension errors?
- Can CI build the embedded artifact reproducibly?
- Does the distribution channel accept the package shape?
- Does embedding break Godot plugin assumptions?
- Does embedding complicate patches or mod support?
- Is the tighter coupling worth the engineering risk?

LibGodot decision table:

| Condition | Recommendation |
|---|---|
| Early prototype | Use traditional launcher + Game.exe |
| First vertical slice | Use traditional launcher + Game.exe |
| Serious bypass issues from direct launch | Evaluate LibGodot |
| Small team with limited engine integration experience | Defer LibGodot |
| Agency/studio with engine integration support | Prototype LibGodot branch |
| Browser target | LibGodot not relevant |

LibGodot is an architecture option, not a shortcut. It should not replace server authority.

---

98. Privacy and Player Trust Policy

Professional anti-cheat must be clear about what it does and does not collect.

Launcher may collect:
- Game file hashes from the install directory.
- Launcher version.
- Game build ID.
- Crash logs related to the game.
- Integrity mismatch telemetry.
- Basic hardware/OS compatibility data if disclosed.

Launcher must not collect:
- Browser history.
- Password manager data.
- Unrelated process memory.
- Personal files outside the install directory.
- Screenshots without explicit user consent.
- Keystrokes outside the game.
- Background surveillance data.

Public wording example:
"The launcher verifies game files, downloads updates, authenticates your session, and sends crash/integrity reports related to this game. It does not scan unrelated applications, browser history, passwords, or personal files."

This wording must match reality. Do not write privacy claims that the code violates.

---

99. Agency Handoff Package Requirements

A serious agency handoff should include:

1. SECURITY_MODEL.md
2. ADR folder
3. BUILD_PIPELINE.md
4. RELEASE_CHECKLIST.md
5. LAUNCHER_PROTOCOL.md
6. MANIFEST_SCHEMA.md
7. GDEXTENSION_API.md
8. BACKEND_AUTHORITY.md
9. BROWSER_POLICY.md
10. DELTA_PATCH_GUIDE.md
11. INCIDENT_RESPONSE.md
12. PRIVACY_AND_TELEMETRY.md
13. AGENT_TASKS.md
14. SECURITY_AUDIT_LOG.md
15. Smoke test scripts
16. CI workflow files
17. Example manifest
18. Example config
19. Example backend endpoints
20. Example GDScript startup guard

Definition of done:
The next developer can clone the repository, read docs, run the smoke suite, and understand exactly which layer owns which responsibility without asking the original architect.

---

100. Agent Prompt for Step Execution

Use this exact style for external coding agents:

You are implementing one step of a Godot + Rust security architecture.

Architecture:
Rust launcher → signed manifest → encrypted Godot export → Rust GDExtension → Rust backend authority.

Non-negotiable rules:
- No placeholder code.
- No fake security claims.
- No production secrets in files.
- No client-authoritative economy, combat, inventory, XP, or premium unlocks.
- No kernel anti-cheat or invasive process scanning.
- Do not skip tests.
- Do not continue into the next step without approval.

Your assigned step:
[INSERT STEP]

Required output:
- Files changed.
- Commands run.
- Test results.
- Risk notes.
- Whether the step passed.
- Recommended next step.

Stop after this step.

---

101. Implementation Roadmap Expanded to 12 Milestones

Milestone 1: Documentation and ADR setup.
Output: SECURITY_MODEL.md, ADR-0001, BACKEND_AUTHORITY.md skeleton.
Exit: Trust boundaries documented.

Milestone 2: Rust workspace scaffold.
Output: launcher, gdextension, backend, shared, build_tools crates.
Exit: cargo test --workspace passes.

Milestone 3: Manifest schema and signing tool.
Output: Manifest v4 structs, signer, verifier, invalid signature tests.
Exit: Modified manifest fails verification.

Milestone 4: Launcher hash verifier.
Output: safe path resolver, hash checker, atomic replacement helper.
Exit: path traversal and hash mismatch tests pass.

Milestone 5: Minimal backend auth/session.
Output: login placeholder for dev, launch token issue, launch token consume.
Exit: token reuse test passes.

Milestone 6: Godot export baseline.
Output: release export preset, custom template plan, encrypted PCK procedure.
Exit: release export launches locally.

Milestone 7: Rust GDExtension bootstrap.
Output: RustSecurityBridge, launch arg parser, GDScript startup guard.
Exit: Godot calls Rust successfully.

Milestone 8: End-to-end protected boot.
Output: launcher starts game, GDExtension validates backend token.
Exit: main menu only opens from valid launcher flow.

Milestone 9: First server-authoritative game system.
Output: choose combat, inventory, or economy depending on genre.
Exit: fake client result is rejected.

Milestone 10: Patch and CDN pipeline.
Output: full file download, delta fallback, manifest update.
Exit: modified PCK repaired.

Milestone 11: CI and smoke tests.
Output: automated build, scan, package, security tests.
Exit: CI blocks invalid release.

Milestone 12: Production readiness review.
Output: release checklist, incident response, privacy policy, audit log.
Exit: build approved for beta.

---

102. Common Failure Modes and Fixes

Failure: The launcher verifies files, but Game.exe can still run online directly.
Fix: Backend must require a valid launch token consumed by GDExtension before creating game session.

Failure: Launch token is passed on command line and copied.
Fix: Token must be single-use, 60-second TTL, and backend-bound. LibGodot can later reduce CLI exposure.

Failure: Client sends damage and server stores it.
Fix: Server must calculate damage. Client sends only spell intent.

Failure: Inventory duplication during concurrent requests.
Fix: Use database transactions and row locks.

Failure: Delta patch corrupts PCK.
Fix: Verify patch hash before apply and target hash after apply. Fall back to full download.

Failure: CI logs print encryption key.
Fix: Mask secrets and review build scripts. Never echo keys.

Failure: PCK encryption key committed to repo.
Fix: Rotate key, invalidate affected builds, remove from history, audit all artifacts.

Failure: Browser build leaks production endpoint with admin-like capability.
Fix: Browser endpoints must be normal public API only. No admin endpoints. No secrets.

Failure: Anti-tamper looks like malware.
Fix: Restrict launcher to install folder verification and documented telemetry.

---

103. Threat-to-Control Mapping

| Threat | Control | Owner | Test |
|---|---|---|---|
| Basic PCK extraction | Encrypted PCK + custom template | Build engineer | PCK extractor smoke test |
| File replacement | Signed manifest + hash verify | Launcher | Modified file test |
| Direct launch | Single-use launch token | Backend/GDExtension | Direct Game.exe test |
| Token replay | Used flag + transaction lock | Backend | Reuse test |
| Fake damage | Server combat authority | Backend | Fake damage test |
| Speed hack | Movement sanity | Backend | Impossible movement test |
| Inventory duping | DB transaction + locks | Backend | Concurrent use test |
| Premium unlock spoofing | Entitlement backend | Backend | Fake entitlement test |
| Debug export leak | Release scan | CI | Forbidden-file scan |
| Secret leak | Secret scanning + CI rules | DevOps | CI audit |
| Browser tampering | Server authority | Backend | Browser fake request test |
| False positive ban | Severity ladder + review | Trust team | Manual review workflow |

---

104. Professional Code Commenting Standard

Every security-sensitive function should answer these questions in comments:
- What trust boundary does this function enforce?
- What inputs are untrusted?
- What does this function intentionally not protect against?
- What happens on failure?
- Which tests prove this behavior?

Example:

```rust
/// Consumes a launch token and creates a game session.
///
/// Trust boundary:
/// Converts an untrusted client-provided launch token into a trusted backend session.
///
/// Untrusted inputs:
/// - raw_token
/// - build_id
/// - manifest_hash
/// - protocol_version
///
/// Security guarantees:
/// - Token must exist.
/// - Token must not be expired.
/// - Token must not have been used before.
/// - Build and manifest must match the token.
/// - Token is marked used in the same transaction that creates the session.
///
/// Non-goals:
/// - Does not prove the client is unmodified by itself.
/// - Does not prevent memory inspection on the player machine.
///
/// Required tests:
/// - token_reuse_fails
/// - expired_token_fails
/// - wrong_build_id_fails
/// - valid_token_creates_one_session
pub async fn consume_launch_token(...) -> Result<GameSession> {
    // implementation
}
```

---

105. Minimum Documentation Quality Standard

Every major document must include:
- Purpose.
- Scope.
- Non-goals.
- Owner.
- Last reviewed date.
- Inputs.
- Outputs.
- Failure modes.
- Test cases.
- Open decisions.

Bad documentation:
"The launcher checks files."

Professional documentation:
"The launcher verifies every required release file listed in the signed manifest. It normalizes paths against the install root, rejects path traversal, computes BLAKE3 hashes, repairs mismatched files from CDN, verifies repaired files, and blocks launch if repair fails. It does not scan files outside the install directory."

---

106. Final V4 Master Principle

The professional version of this architecture is not bigger because it adds more buzzwords. It is bigger because it removes ambiguity.

The purpose of this blueprint is to make every future implementation decision answerable:
- Which layer owns this?
- Is this trusted?
- What happens if it is modified?
- What is the fallback?
- How do we test it?
- What should the player see?
- What should the backend log?
- What should CI block?
- What can an agent safely implement without improvising?

If a section does not answer those questions, it is not finished.

---

APPENDIX A: Quick Command Reference

Build Rust workspace:

```bash
cargo fmt --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo build --workspace --release
```

Build GDExtension only:

```bash
cargo build -p rust_security_bridge --release
```

Export Godot release from CLI:

```bash
godot --headless --path godot_project --export-release "Windows Desktop" dist/Game.exe
```

Generate manifest:

```bash
cargo run -p build_tools --bin generate_manifest -- dist manifest.json
```

Sign manifest:

```bash
cargo run -p build_tools --bin sign_manifest -- manifest.json
```

Verify manifest:

```bash
cargo run -p build_tools --bin verify_manifest -- manifest.json
```

Scan release folder:

```bash
cargo run -p build_tools --bin validate_release_folder -- dist
```

Run security smoke tests:

```bash
cargo test -p security_smoke_tests
```

---

APPENDIX B: Example Error Code Registry

| Code | Layer | User Message | Internal Meaning |
|---|---|---|---|
| OK | All | Success | Success |
| MANIFEST_SIGNATURE_INVALID | Launcher | Update verification failed. | Manifest signature mismatch |
| FILE_HASH_MISMATCH | Launcher | Game files need repair. | Local file hash differs |
| PATCH_APPLY_FAILED | Launcher | Update repair failed. | Delta patch failed |
| LAUNCH_TOKEN_MISSING | Client | Please start from the launcher. | No token found |
| LAUNCH_TOKEN_EXPIRED | Backend | Session expired. Restart launcher. | Token TTL exceeded |
| LAUNCH_TOKEN_USED | Backend | Session already used. Restart launcher. | Replay attempt |
| BUILD_ID_REJECTED | Backend | Please update the game. | Build not accepted |
| PROTOCOL_UNSUPPORTED | Backend | Please update the game. | Protocol mismatch |
| SESSION_REVOKED | Backend | Session ended. | Backend revoked session |
| MOVEMENT_REJECTED | Backend | Position corrected. | Movement violation |
| COMBAT_REJECTED | Backend | Action failed. | Invalid combat intent |
| INVENTORY_REJECTED | Backend | Item action failed. | Invalid inventory transaction |

---

APPENDIX C: Release Readiness Scorecard

Score each item 0 to 2.
0 = missing.
1 = partially implemented.
2 = implemented, tested, documented.

| Area | Score |
|---|---:|
| Threat model documented | |
| Server authority matrix complete | |
| Manifest schema implemented | |
| Manifest signature verified | |
| File hash repair works | |
| Launch token single-use works | |
| Direct launch blocked | |
| GDExtension handshake works | |
| Encrypted PCK export works | |
| Forbidden-file scan works | |
| Backend rejects fake combat | |
| Backend rejects fake inventory | |
| Movement sanity implemented | |
| CI runs smoke tests | |
| Incident response documented | |
| Privacy policy matches launcher behavior | |
| Browser restrictions documented | |
| Release checklist signed | |

Minimum beta score: 28/36.
Minimum production score: 34/36.
No production release if any of these are 0: manifest signature, launch token single-use, backend authority for valuable state, forbidden-file scan, release checklist.

---

End of Universal Godot + Rust Security Architecture Pre-Planning Blueprint v4.1 Re-Audited Master Edition


---

APPENDIX D: Source References for Technical Claims

Godot PCK encryption documentation:
https://docs.godotengine.org/en/stable/engine_details/development/compiling/compiling_with_script_encryption_key.html

Godot Web export documentation:
https://docs.godotengine.org/en/latest/tutorials/export/exporting_for_web.html

Godot 4.6 release notes:
https://godotengine.org/releases/4.6/

godot-rust March 2026 development update:
https://godot-rust.github.io/dev/march-2026-update/

godot-rust gdext changelog:
https://github.com/godot-rust/gdext/blob/master/Changelog.md


---

APPENDIX E: V4.1 Source Verification Notes

These notes are included so future agents do not reintroduce stale or unsafe claims.

1. PCK encryption verification:
Godot's documentation states that export PCK encryption uses a 256-bit AES key, that the key must be in the binary, and that custom export templates must be built from source with the same key. It also warns that official precompiled templates will not work for this encrypted-PCK path. It further notes Android exports need APK expansion for PCK encryption to apply.

2. Godot 4.6 feature verification:
Godot 4.6 makes Jolt the default physics engine for new 3D projects while existing projects retain current physics settings. Godot 4.6 also introduces LibGodot, improves Patch PCKs with delta encoding, makes Direct3D 12 the default for new Windows projects, and adds API nullability information that helps languages with optional/nullable type systems.

3. GDExtension verification:
Godot documentation describes GDExtension as the official mechanism for letting the engine interact with native shared libraries at runtime without compiling native code into the engine.

4. godot-rust safeguard verification:
The March 2026 godot-rust update lists three safeguard levels: Strict, Balanced, and Disengaged. Strict is the default in Debug builds. Balanced is the default in Release builds. Disengaged sacrifices safety for raw speed and is typically not needed. This document therefore uses Balanced as the normal production baseline.

5. Web export verification:
Godot Web export requires WebAssembly and WebGL 2.0 support. Godot documentation warns that Godot 4 C# projects currently cannot be exported to the web through the standard path. Browser builds must also account for developer tools, local storage inspection, and network inspection.

End of Universal Godot + Rust Security Architecture Pre-Planning Blueprint v4.1 Re-Audited Master Edition
