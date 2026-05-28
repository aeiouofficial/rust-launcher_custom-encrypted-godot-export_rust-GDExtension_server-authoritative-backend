# Universal Godot Security Architecture Pre-Planning Blueprint

Version: 1.0
Target use: Any Godot 4.x desktop game that wants stronger protection against casual unpacking, basic file tampering, copy-paste cracking, basic asset ripping, simple client-side cheating, and direct executable bypass.
Primary architecture: Rust launcher + custom encrypted Godot export + Rust GDExtension + server-authoritative backend.
Important security stance: This is not impossible-to-crack DRM. This is a practical production architecture that makes the client untrusted, moves valuable authority server-side, and adds enough packaging, validation, and tamper evidence to stop casual Godot decompiler users, noob crackers, basic PCK extractors, and simple asset rippers.

This document is intentionally written as a pre-planning blueprint before implementation. It is meant to be reusable for any Godot game, regardless of genre, and should be copied into the project documentation before coding begins.

---

## 1. Executive Summary

The goal is to protect a Godot game without pretending that local client security can ever be perfect.

A player-controlled machine is hostile by default. Anything shipped to the player can eventually be inspected, dumped, patched, or emulated by a determined attacker. Therefore, the correct architecture is not to hide everything inside a Rust shell. The correct architecture is to make the Godot client untrusted, move important authority to the backend, encrypt and strip the shipped build enough to block casual ripping, and use Rust for the launcher, sensitive client-side bridge code, patching, build verification, and server logic.

The recommended architecture is:

Rust launcher starts first.

The launcher verifies the install.

The launcher checks a signed file manifest.

The launcher repairs or rejects modified files.

The launcher authenticates the player.

The backend issues a short-lived single-use launch token.

The launcher starts the Godot export with controlled arguments.

The Godot game loads a Rust GDExtension.

The Rust GDExtension validates the launch/session state.

The backend remains the source of truth for important gameplay state.

The Godot client only displays, predicts, animates, and requests actions.

The backend decides whether valuable state changes actually happen.

This structure can be applied to singleplayer-with-online-features, co-op, multiplayer, MMO-style games, competitive games, RPGs, survival games, extraction games, browser demos, and commercial Godot desktop games.

---

## 2. Source-Aware Technical Reality Check

Godot supports encrypted PCK export using a 256-bit AES key. This protects scenes, scripts, and resources from being stored in plain text and from being easily ripped by basic tools, but the encryption key must still exist in the binary. That means PCK encryption is useful resistance against casual ripping, not absolute protection.

Godot GDExtension is the official native extension mechanism for Godot 4. It lets a Godot project load native shared libraries and call into native code at runtime.

The godot-rust/gdext project is the practical Rust binding layer for Godot 4. It allows Godot and Rust code to coexist, with custom Rust APIs callable from GDScript.

Godot Web export can publish games to browsers through WebAssembly/WebGL, but web clients must be considered fully inspectable. Browser builds cannot receive the same protection assumptions as desktop builds. Any serious gameplay authority must still be server-side.

Authoritative planning rule:

Encrypted PCK protects files from casual ripping.

Custom export template makes the encrypted export usable.

Rust GDExtension moves selected sensitive client helpers into native code.

Rust launcher controls normal entry, patching, manifest checks, and launch tokens.

Rust backend owns important gameplay truth.

Browser build is convenience and reach, not strong anti-cheat.

---

## 3. Security Mission Statement

The mission is not:

Make the Godot client impossible to reverse engineer.

Build malware-like anti-cheat.

Hide everything in Rust.

Prevent all piracy forever.

Trust local files because the launcher checked them once.

Trust command-line arguments because the launcher passed them.

Trust the browser client.

The mission is:

Make the client untrusted.

Move important validation server-side.

Encrypt and strip shipped Godot data enough to stop casual ripping.

Use Rust where Rust gives meaningful security, performance, or tooling value.

Use signed build manifests and short-lived tokens.

Detect and reject basic tampering.

Make direct Game.exe launching useless for online/protected mode.

Reject impossible gameplay requests server-side.

Keep security behavior trustworthy, explainable, and non-invasive.

---

## 4. Target Attacker Model

This architecture is designed to stop or frustrate:

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

People who can inspect a running process, patch instructions, or dump decrypted runtime memory.

Correct response to advanced attackers:

Do not escalate into shady invasive anti-cheat.

Do not make fake claims.

Harden server authority.

Improve telemetry.

Improve protocol validation.

Improve rate limits.

Improve signed manifests and build rollout.

Improve content watermarking and abuse response.

---

## 5. Universal Architecture Overview

Protected Desktop Client:

Rust Launcher / Security Wrapper

Responsibilities:

Login.

Patch/update.

Signed manifest verification.

File hash checks.

Local install repair.

Basic local anti-tamper checks.

Crash/log upload.

Short-lived launch token request.

Controlled Godot process start.

Optional child process heartbeat.

Optional launcher self-update.

Custom Godot Export:

Responsibilities:

Encrypted PCK/resources.

Custom export template built with matching encryption key.

Stripped release binary.

No debug export.

No remote debug.

No editor/dev assets.

Rust GDExtension loaded.

Backend communication.

Rust GDExtension:

Responsibilities:

Launcher/game handshake helper.

Build/session validation helper.

Packet signing helper.

Local sanity prechecks.

Sensitive client helper logic.

Save/session verification for offline-compatible modes.

Asset manifest verification.

Client telemetry packing.

Protocol schema consistency.

Rust Backend:

Responsibilities:

Authentication.

Session validation.

Build ID validation.

Inventory authority.

Economy authority.

Quest validation.

Movement sanity checks.

Combat validation.

Loot validation.

Premium entitlement validation.

Telemetry storage.

Cheat flag pipeline.

Moderation/audit trail.

Browser/Web Build:

Responsibilities:

Browser playability.

Account login.

Server-authoritative gameplay.

Reduced asset set where useful.

No trusted local secrets.

No client-authoritative progression.

No local-only economy authority.

No fake protection claims.

---

## 6. Core Security Rule

The launcher protects entry.

The encrypted PCK protects against casual ripping.

The Rust extension protects selected local glue and helper logic.

The backend protects actual game state.

The client is never trusted.

The server decides important outcomes.

If a state change matters to progression, economy, combat, entitlement, ranking, matchmaking, achievements, or trading, it must be validated or owned server-side.

---

## 7. Game-Type Adaptation Matrix

Singleplayer offline game:

Use encrypted PCK.

Use custom export template.

Use stripped release binary.

Use optional Rust GDExtension for save validation and local anti-tamper.

Use no mandatory always-online backend unless the product design requires it.

Do not pretend offline saves cannot be modified.

Treat local saves as convenience, not competitive truth.

Singleplayer with online account/progression:

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

Use higher tick/reconciliation requirements.

Do not trust client hit results.

Do not trust client movement beyond tolerance.

Add telemetry and replay sampling.

MMO-style RPG:

Use full stack.

Backend owns characters, items, gold, XP, quests, loot, vendors, trading, mail, auction, cooldowns, and movement sanity.

Client becomes renderer, input collector, UI layer, and prediction layer.

Browser demo:

Use Godot Web export.

Use reduced content where possible.

No permanent secrets.

No trusted progression unless backend-validated.

Optional account-gated demo permissions.

---

## 8. Repository Template

Recommended monorepo layout:

```text
godot-security-stack/
│
├─ launcher/
│  ├─ Cargo.toml
│  ├─ src/
│  │  ├─ main.rs
│  │  ├─ app.rs
│  │  ├─ config.rs
│  │  ├─ manifest.rs
│  │  ├─ patcher.rs
│  │  ├─ downloader.rs
│  │  ├─ auth.rs
│  │  ├─ process.rs
│  │  ├─ integrity.rs
│  │  ├─ self_update.rs
│  │  ├─ crash_report.rs
│  │  └─ telemetry.rs
│  └─ assets/
│
├─ gdextension/
│  ├─ Cargo.toml
│  ├─ src/
│  │  ├─ lib.rs
│  │  ├─ bridge.rs
│  │  ├─ handshake.rs
│  │  ├─ crypto.rs
│  │  ├─ protocol.rs
│  │  ├─ gameplay_validation.rs
│  │  ├─ movement.rs
│  │  ├─ combat.rs
│  │  ├─ inventory.rs
│  │  ├─ save_validation.rs
│  │  └─ telemetry.rs
│  └─ godot/
│     └─ rust_security_bridge.gdextension
│
├─ backend/
│  ├─ Cargo.toml
│  ├─ src/
│  │  ├─ main.rs
│  │  ├─ config.rs
│  │  ├─ routes.rs
│  │  ├─ auth.rs
│  │  ├─ sessions.rs
│  │  ├─ builds.rs
│  │  ├─ entitlements.rs
│  │  ├─ characters.rs
│  │  ├─ combat.rs
│  │  ├─ movement.rs
│  │  ├─ inventory.rs
│  │  ├─ economy.rs
│  │  ├─ quests.rs
│  │  ├─ loot.rs
│  │  ├─ telemetry.rs
│  │  └─ cheat_flags.rs
│  └─ migrations/
│
├─ shared/
│  ├─ Cargo.toml
│  ├─ src/
│  │  ├─ protocol.rs
│  │  ├─ build_manifest.rs
│  │  ├─ crypto_types.rs
│  │  ├─ token_types.rs
│  │  ├─ validation_types.rs
│  │  └─ error_codes.rs
│
├─ godot_project/
│  ├─ project.godot
│  ├─ export_presets.cfg
│  ├─ bin/
│  │  └─ rust_security_bridge.dll
│  ├─ scenes/
│  ├─ scripts/
│  ├─ addons/
│  └─ resources/
│
├─ build_tools/
│  ├─ Cargo.toml
│  ├─ src/
│  │  ├─ generate_manifest.rs
│  │  ├─ sign_manifest.rs
│  │  ├─ verify_export.rs
│  │  ├─ package_release.rs
│  │  ├─ check_for_dev_files.rs
│  │  └─ validate_release_folder.rs
│
├─ deploy/
│  ├─ docker-compose.dev.yml
│  ├─ backend.Dockerfile
│  ├─ migrations/
│  └─ cdn_upload/
│
├─ docs/
│  ├─ 00_README_SECURITY_OVERVIEW.md
│  ├─ 01_THREAT_MODEL.md
│  ├─ 02_ARCHITECTURE_DECISIONS.md
│  ├─ 03_RELEASE_BUILD_PIPELINE.md
│  ├─ 04_LAUNCHER_REQUIREMENTS.md
│  ├─ 05_GODOT_EXPORT_REQUIREMENTS.md
│  ├─ 06_GDEXTENSION_REQUIREMENTS.md
│  ├─ 07_BACKEND_AUTHORITY_REQUIREMENTS.md
│  ├─ 08_BROWSER_BUILD_POLICY.md
│  ├─ 09_AGENT_TASKS.md
│  ├─ 10_VALIDATION_TESTS.md
│  └─ 11_FINAL_SECURITY_AUDIT.md
│
└─ ci/
   ├─ build_launcher.yml
   ├─ build_gdextension.yml
   ├─ build_backend.yml
   ├─ export_godot.yml
   ├─ package_release.yml
   └─ security_smoke_tests.yml
```

---

## 9. Recommended Technology Choices

Launcher:

Rust.

tokio for async.

reqwest for HTTPS.

serde for JSON/TOML.

blake3 or sha2 for hashing.

ed25519-dalek or ring for signature verification.

tracing for logging.

egui, Slint, Tauri, or a minimal native UI depending on design needs.

Do not use a giant launcher stack before the security pipeline is proven.

GDExtension:

godot-rust/gdext.

Small exported API surface.

Release-mode native library.

No huge gameplay rewrite on day one.

Use Rust for sensitive helpers, protocol creation, validation helpers, and performance-critical systems.

Backend:

Rust.

axum or actix-web.

PostgreSQL for durable game/account state.

Redis optional for sessions, rate limiting, and short-lived token state.

SQLx or SeaORM.

Structured logs.

OpenTelemetry or tracing-compatible pipeline.

Godot:

Godot 4.x.

Custom export template for encrypted PCK.

Encrypted PCK/resources.

Release export only.

No remote debug.

No debug symbols in shipped build.

GDScript/C# only for non-authoritative presentation/client glue.

Build/CI:

GitHub Actions, GitLab CI, or local scripted release pipeline.

Separate dev/staging/production release channels.

Signed manifests.

Automated release folder validation.

Automated smoke tests for tampering.

---

## 10. Pre-Implementation Decision Gates

Before coding, answer these questions.

Game mode:

Is the game fully offline?

Is there account-based progression?

Is there multiplayer?

Is PvP competitive?

Is trading/economy important?

Is browser support required for full game or just demo?

What state must be server-authoritative?

Combat?

Movement?

Inventory?

XP?

Gold/currency?

Quest completion?

Drops/loot?

Premium unlocks?

Achievements?

Leaderboard/ranking?

How much latency is acceptable?

Action RPG low latency?

MMO-style moderate latency?

Turn-based?

Survival/crafting?

Shooter-like strict latency?

What platforms are required?

Windows only first?

Linux support?

macOS support?

Browser support?

Steam/Epic/itch.io standalone?

What security level is commercially reasonable?

Casual protection only?

Basic multiplayer cheat resistance?

Serious PvP integrity?

MMO economy integrity?

What budget exists for backend hosting?

Local-only?

Small VPS?

Managed Postgres?

Redis?

CDN for patch files?

Do you need offline play?

If yes, which parts may be offline?

Can offline progress sync online?

If offline progress syncs online, what prevents local save abuse?

---

## 11. Build Variants

Development build:

Unencrypted PCK allowed.

Debug enabled.

Remote debug allowed.

No production backend.

Fake local auth allowed.

Verbose logs allowed.

Never distributed to players.

Internal QA build:

Can be encrypted or unencrypted depending on QA needs.

Debug symbols may exist in private symbol server only.

Uses staging backend.

Launcher optional depending on test.

Test accounts only.

Preview build:

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

Backend validates manifest hash/session.

Production browser build:

Godot Web export.

No trusted client secrets.

No local authority.

Backend validates gameplay.

May have reduced content or demo restrictions.

---

## 12. Custom Godot Export Plan

Purpose:

Make scenes, scripts, and resources non-plain-text.

Prevent basic PCK extraction from immediately exposing project internals.

Raise reverse-engineering effort.

Do not rely on this as the only protection.

Required steps:

Generate per-channel encryption key.

Build Godot export template with key.

Configure export preset to encrypt PCK/resources.

Export release build.

Strip symbols.

Check output folder for forbidden dev files.

Generate signed file manifest after export.

Smoke test clean launch through launcher.

Security notes:

The PCK encryption key exists in the binary.

A determined reverse engineer can extract it.

This is still worth doing because the target attacker is casual/basic.

Do not ship the key in scripts, config files, docs, or CI logs.

Do not reuse production keys in public test builds.

Use separate dev/staging/production keys.

Key handling:

Production key should live in CI secret storage or a restricted local release machine.

Never commit key to git.

Never print key in build logs.

Rotate key per major release channel if practical.

Keep old keys only as long as old builds need patch support.

Export validation checklist:

Game executable exists.

Encrypted PCK exists.

GDExtension binary exists.

No raw .gd files outside encrypted PCK.

No .godot folder.

No editor-only plugins.

No raw source asset dumps.

No debug symbol files in public package.

No private keys.

No production API secrets.

Manifest generated after final packaging state.

---

## 13. Rust Launcher Plan

Purpose:

Make the launcher the normal and required entry point for protected desktop mode.

Responsibilities:

Self-version check.

Download latest signed manifest.

Verify manifest signature.

Hash local game files.

Repair or reject modified files.

Download missing files.

Authenticate user.

Request short-lived launch token.

Start Godot with controlled arguments.

Monitor child process exit code.

Collect crash logs.

Upload integrity telemetry.

Show useful errors to players.

Launcher should not:

Scan unrelated processes.

Read private browser/password/app memory.

Install kernel drivers.

Hide processes.

Persist itself secretly.

Block debuggers in sketchy ways that break normal machines.

Pretend to be unbreakable.

Launcher launch flow:

Player starts launcher.

Launcher loads config.

Launcher checks launcher version.

Launcher downloads channel manifest.

Launcher verifies signature using embedded public key.

Launcher checks local install folder.

Launcher hashes required files.

Launcher downloads changed/missing files.

Launcher verifies patched files.

Launcher authenticates player.

Backend creates launcher session.

Launcher requests single-use launch token.

Launcher starts Game.exe with build ID, manifest hash, launch token, and launcher PID.

Game loads.

Rust GDExtension validates token with backend.

Backend activates game session.

Game opens main menu.

Direct Game.exe run fails protected online mode.

Controlled argument example:

```text
Game.exe
  --build-id 2026.05.29.001
  --channel production
  --launch-token <single_use_short_lived_token>
  --launcher-pid <pid>
  --manifest-hash <sha256_or_blake3>
```

Do not rely on command-line secrecy.

Command-line arguments can be inspected.

The token must be short-lived.

The token must be single-use.

The token must be backend-validated.

The token must be bound to account, build ID, manifest hash, issue time, and channel.

---

## 14. Signed Manifest Plan

Purpose:

Prevent casual file replacement and ensure the launcher knows the exact approved release layout.

Manifest fields:

Build ID.

Channel.

Created timestamp.

Minimum launcher version.

Minimum backend protocol version.

Game executable path and hash.

Encrypted PCK path and hash.

GDExtension library path and hash.

Optional patch files.

Optional asset bundles.

File sizes.

Compression info.

Download URLs.

Signature.

Example manifest:

```json
{
  "schema_version": 1,
  "build_id": "2026.05.29.001",
  "channel": "production",
  "created_at": "2026-05-29T00:00:00Z",
  "min_launcher_version": "0.1.0",
  "min_protocol_version": 1,
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
  "signature_algorithm": "ed25519",
  "signature": "base64_signature_here"
}
```

Rules:

Manifest is signed by release private key.

Launcher embeds only public verification key.

Backend stores accepted build IDs and manifest hashes.

Backend rejects unknown build IDs.

Backend can force update.

Backend can reject stale builds.

Do not invent cryptography.

Use audited signature libraries.

Private signing key never ships.

Public verification key may ship.

---

## 15. Rust GDExtension Plan

Purpose:

Create a native bridge between Godot and Rust for protected client helpers, session validation, protocol construction, and selected sensitive logic.

Good Rust GDExtension candidates:

Launcher/game handshake.

Build/session validation helper.

Packet signing helper.

Movement sanity prechecks.

Combat request construction.

Inventory transaction request construction.

Local telemetry packing.

Asset manifest validation.

Save validation for offline sections.

Sensitive constants split/obfuscated.

Protocol versioning.

Anti-tamper heartbeat helper.

Bad Rust GDExtension candidates:

Full UI.

Basic animation code.

Normal quest dialogue.

Every scene script.

All gameplay code before there is a server authority model.

Anything that slows iteration without real security value.

Godot-facing API should be small:

validate_launch_args(args) -> bool/result.

create_handshake_payload() -> Dictionary/bytes.

make_combat_request(...) -> PackedByteArray/Dictionary.

make_inventory_request(...) -> PackedByteArray/Dictionary.

submit_local_telemetry(event) -> void.

validate_local_asset_manifest(...) -> bool/result.

get_protocol_version() -> int.

get_build_fingerprint() -> string.

Godot example:

```gdscript
var bridge := RustSecurityBridge.new()

var result := bridge.validate_launch_args(OS.get_cmdline_args())

if not result.ok:
    push_error(result.message)
    get_tree().quit()
```

Combat example:

```gdscript
var request := bridge.make_combat_request(
    player_id,
    target_id,
    spell_id,
    player_position,
    client_timestamp
)

Network.send(request)
```

Important:

Rust can structure/sign/prepare the request.

The server still validates everything.

Never assume Rust code in the client makes the result trusted.

---

## 16. Backend Authority Plan

Purpose:

Stop cheating by refusing invalid state changes.

Backend services:

AuthService.

SessionService.

BuildValidationService.

CharacterService.

InventoryService.

EconomyService.

QuestService.

CombatService.

MovementService.

LootService.

EntitlementService.

TelemetryService.

CheatFlagService.

AuditLogService.

The client may submit intent.

The server validates intent.

The server applies valid state changes.

The server rejects invalid state changes.

The server writes audit logs for important changes.

Never trust client for:

Gold.

Premium currency.

Items.

XP.

Level.

Loot.

Quest completion.

Premium unlocks.

Cooldowns.

Combat result.

Movement result.

NPC kill credit.

Trading.

Auction/economy operations.

Crafting results.

Dungeon/raid lockouts.

Achievement unlocks.

Ranked match results.

Entitlements.

Client may send:

I want to cast spell X on target Y.

I want to move from A to B.

I want to use item X.

I want to loot corpse X.

I want to accept quest X.

I want to complete quest objective X.

I want to buy vendor item X.

I want to trade item X.

I want to enter zone X.

Server decides whether it happened.

---

## 17. Combat Authority Plan

Client attack intent:

Player 10 casts Fireball rank 2 on Mob 883.

Server checks:

Player session valid.

Build ID allowed.

Player owns character.

Character is alive.

Character is not stunned/silenced if relevant.

Spell is learned.

Spell rank is valid.

Spell is not on cooldown.

Global cooldown is valid.

Enough mana/rage/energy/resource.

Target exists.

Target is alive.

Target is hostile or valid.

Target is in range.

Line-of-sight valid if required.

Cast time/channel time valid.

Action timing not impossible.

Player position is plausible.

Target position is plausible.

No impossible action spam.

Server calculates:

Hit/miss.

Crit.

Block/dodge/parry/resist if relevant.

Damage.

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

UI cooldown.

Server result.

Never accept from client:

Damage amount.

Critical hit result.

Loot roll result.

Kill confirmation.

XP amount.

Cooldown reset.

Resource gain without validation.

---

## 18. Movement Authority Plan

Purpose:

Prevent basic speed, fly, teleport, and interaction-distance cheats without requiring perfect physics on day one.

Server tracks:

Last known position.

Last movement timestamp.

Movement mode.

Max allowed speed.

Current zone/terrain bounds.

Mount state.

Falling state.

Swimming state.

Root/stun/snare state.

Teleport permissions.

Recent correction history.

Reject or flag:

Impossible distance per second.

Repeated micro-teleports.

Moving while stunned/rooted.

Attacking while too far away.

Looting from impossible distance.

Interacting through walls beyond tolerance.

Entering locked zones.

Invalid vertical movement.

Movement mode mismatch.

Movement severity ladder:

Level 1: tolerate minor network weirdness.

Level 2: soft correction.

Level 3: reject action.

Level 4: flag session/account.

Level 5: temporary disconnect.

Level 6: manual review/ban queue.

Do not instantly ban from one weird packet.

Use accumulated evidence.

Keep false positives low.

---

## 19. Inventory And Economy Authority Plan

Purpose:

Make item, gold, currency, vendor, trade, crafting, and loot hacks worthless.

Never accept:

Set gold to X.

Add item Y.

Quest complete true.

Loot table says I got item.

Trade accepted with modified values.

Crafting succeeded locally.

Vendor buy without server validation.

Premium unlock true.

Use transaction requests:

Client request:

Use item 1004 on target 889.

Server checks:

Item exists in player inventory.

Item is usable.

Player owns the item.

Item is not expired/locked.

Target is valid.

Cooldown is valid.

Zone/quest state allows it.

Consumes charges if valid.

Applies result transactionally.

Database transaction rule:

```text
BEGIN
  validate current state
  apply change
  write audit log
COMMIT
```

Important systems requiring transaction logs:

Item creation.

Item deletion.

Gold changes.

Premium currency changes.

Trade.

Auction.

Mail attachments.

Crafting.

Vendor buy/sell.

Loot.

Quest rewards.

Admin grants.

Compensation grants.

Rollback events.

---

## 20. Quest And Progression Authority Plan

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

Player level/faction/class allowed.

Objective event is plausible.

Required item exists.

Required kill was credited server-side.

Required location reached.

Reward already not claimed.

Reward choice valid.

---

## 21. Entitlement And Premium Unlock Plan

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

Build/channel supports entitlement.

No duplicate claim.

No refunded purchase.

No chargeback block if relevant.

---

## 22. Save System Plan

For fully offline games:

Use signed save files to detect casual edits.

Accept that determined users can still edit offline saves.

Do not claim offline saves are cheat-proof.

For online progression:

Local save is cache only.

Server owns real progression.

Client syncs from server.

Client submits intent, not final state.

For hybrid games:

Separate offline profile from online profile.

Never let offline-only progress directly become authoritative online economy/progression without validation.

Signed local save structure:

Save payload.

Schema version.

Build version.

Timestamp.

Hash.

Signature/MAC.

Optional device-local key.

Remember:

If the key is local, it can be extracted.

Signed saves stop casual edits, not professional tampering.

---

## 23. Token And Session Plan

Token types:

Launcher login token:

Received after user auth.

Used only by launcher.

Short lived.

Not permanent.

Launch token:

Short-lived.

Single-use.

Passed to Godot.

Bound to build ID, manifest hash, account, issue timestamp, and channel.

Validated by backend.

Game session token:

Issued after Godot validates launch token.

Used for game server communication.

Rotated periodically.

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

## 24. Anti-Tamper Without Malware Behavior

Good checks:

File hashes.

Signed manifest.

Build ID validation.

Process started through launcher for protected mode.

Expected command args present.

GDExtension loaded.

Backend launch token confirmed.

Runtime version heartbeat.

Manifest mismatch telemetry.

Known bad build rejection.

Release folder validation.

Bad checks:

Kernel drivers.

Scanning unrelated processes.

Reading browser/password/app memory.

Hiding files/processes.

Persistence tricks.

Aggressive debugger blocking that breaks normal users.

Rootkit behavior.

Anything that looks like malware.

Principle:

The security system should be trustworthy, explainable, and commercially defensible.

---

## 25. Browser Build Strategy

Browser clients are inspectable.

Browser devtools exist.

WASM can be inspected.

Network calls can be inspected.

Local storage can be modified.

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

Social/playable preview.

Low-stakes gameplay.

Browser may use:

Server authority.

Reduced asset set.

Separate browser balance.

Demo-only content.

Optional restrictions.

Browser must not use:

Permanent secret keys.

Local economy authority.

Client-side reward authority.

Client-only anti-cheat claims.

Recommended split:

Desktop:

Full protected client.

Launcher required.

Encrypted PCK.

Rust GDExtension.

Stronger telemetry.

Production progression.

Browser:

No launcher.

No strong client protection.

Server-authoritative only.

Reduced asset set optional.

Demo or limited progression optional.

---

## 26. Release Build Pipeline

Production release flow:

Build Rust backend.

Run backend tests.

Build Rust shared crate.

Build Rust GDExtension release library.

Copy GDExtension into Godot project.

Generate or load PCK encryption key from secure secret storage.

Build custom Godot export template with key.

Export encrypted Godot project.

Strip symbols.

Build Rust launcher.

Assemble release folder.

Check for forbidden files.

Generate file manifest.

Sign manifest.

Verify manifest signature.

Package release.

Upload release to patch CDN/storage.

Register build ID and manifest hash with backend.

Smoke test clean install.

Smoke test modified file rejection.

Smoke test stale build rejection.

Smoke test direct Game.exe launch rejection.

Smoke test launch token reuse rejection.

Smoke test missing GDExtension rejection.

Smoke test backend combat/inventory authority.

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

No secrets in config.

Crash reporting works.

Logs redact tokens.

Patch download works.

Clean install works.

Tampered install rejected/repaired.

---

## 27. CI/CD Pipeline

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

Forbidden file scan should fail build if release includes:

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

---

## 28. Telemetry And Cheat Flag Plan

Telemetry goals:

Detect impossible behavior.

Find broken builds.

Catch repeated tampering.

Understand false positives.

Support manual review.

Do not overcollect private user data.

Good telemetry events:

Launch success/failure.

Manifest mismatch.

Token validation failure.

Build mismatch.

Session reuse attempt.

Invalid combat request.

Impossible movement event.

Invalid inventory request.

Repeated rejected actions.

Crash report.

Protocol mismatch.

Telemetry should include:

Account/session ID.

Build ID.

Manifest hash.

Event type.

Timestamp.

Zone/map if game-relevant.

Action summary.

Severity.

Redacted metadata.

Telemetry should not include:

Passwords.

Private browser data.

Unrelated process list.

Personal files.

Sensitive unrelated machine data.

Full memory dumps unless user-approved and safe.

Cheat flag severity:

Info.

Suspicious.

Likely cheat.

Confirmed invalid.

Manual review.

Auto action.

Avoid instant bans unless evidence is overwhelming.

Prefer:

Reject action.

Correct state.

Log.

Escalate with repeated evidence.

---

## 29. Validation Test Plan

Minimum protected boot tests:

Start via launcher succeeds.

Start Game.exe manually fails protected mode.

Modify game.pck and launcher repairs/rejects.

Modify rust_security_bridge.dll and launcher repairs/rejects.

Use old build ID and backend rejects.

Reuse launch token and backend rejects.

Remove GDExtension and game fails protected boot.

Use invalid manifest signature and launcher rejects.

Use stale launcher version and backend rejects if configured.

Encrypted export tests:

Basic PCK extractor does not reveal plain scenes/scripts.

Exported client runs normally.

Dev scenes not present.

Raw .gd files not present outside encrypted PCK.

Debug symbols absent from public package.

GDExtension tests:

Godot loads Rust library.

Rust class visible in Godot.

Handshake method works.

Packet construction works.

Telemetry event method works.

Missing extension blocks protected mode.

Backend authority tests:

Client cannot set damage.

Client cannot bypass cooldown.

Client cannot attack out of range.

Client cannot kill mob directly.

Client cannot add gold.

Client cannot add items.

Client cannot claim invalid loot.

Client cannot claim invalid quest reward.

Movement tests:

Speed hack gets corrected/flagged.

Teleport attempt rejected/flagged.

Loot from impossible distance rejected.

Attack from impossible distance rejected.

Repeated movement violations escalate.

Browser tests:

No permanent secrets visible.

Browser client cannot grant gold/items/XP.

Browser build can be blocked or limited by backend policy.

Backend treats browser as untrusted.

---

## 30. Agent Execution Order

Agents must not jump ahead. Build the stack in this order:

01_SECURITY_MODEL.md

Define threat model.

Define protected state.

Define non-goals.

Define safety boundaries.

02_RUST_WORKSPACE_SCAFFOLD.md

Create launcher, backend, gdextension, shared, build_tools crates.

Add formatting, linting, and minimal tests.

03_BACKEND_AUTH_AND_BUILD_ID.md

Create backend skeleton.

Add build ID registry.

Add session model.

Add health endpoints.

04_SIGNED_MANIFEST_FORMAT.md

Define manifest schema.

Add manifest signing tool.

Add manifest verification tool.

Add tests for bad signature.

05_LAUNCHER_HASH_CHECKER.md

Launcher reads manifest.

Verifies signature.

Hashes local files.

Reports mismatch.

06_LAUNCHER_PATCHER.md

Launcher downloads missing/changed files.

Verifies after patch.

Handles errors.

07_LAUNCH_TOKEN_FLOW.md

Launcher authenticates.

Backend issues launch token.

Token is single-use and short-lived.

08_GODOT_GDEXTENSION_BOOTSTRAP.md

Create Rust GDExtension.

Expose simple Godot-callable class.

Godot loads extension.

09_GODOT_LAUNCH_VALIDATION.md

Godot passes args to GDExtension.

GDExtension validates token with backend.

Protected mode blocks invalid launch.

10_CUSTOM_GODOT_EXPORT_TEMPLATE.md

Build export template with PCK encryption key.

Document key handling.

11_ENCRYPTED_PCK_EXPORT.md

Export encrypted Godot build.

Validate no raw dev files.

12_RELEASE_PACKAGING.md

Package launcher + Godot export + PCK + GDExtension.

Generate signed manifest.

13_SERVER_COMBAT_AUTHORITY.md

Move combat validation server-side.

Client sends intent.

Server returns result.

14_SERVER_INVENTORY_AUTHORITY.md

Move inventory/gold/loot transactions server-side.

Add audit logs.

15_SERVER_MOVEMENT_SANITY.md

Add server movement sanity checks.

Add corrections and flags.

16_TELEMETRY_AND_CHEAT_FLAGS.md

Add telemetry event pipeline.

Add severity logic.

Add false-positive cautious escalation.

17_BROWSER_BUILD_POLICY.md

Create browser export policy.

Define limitations.

Ensure server authority.

18_FINAL_SECURITY_AUDIT.md

Run full validation test plan.

Check docs.

Check release package.

Check secrets.

Sign off.

---

## 31. Agent Prompt Template

Use this prompt when handing work to another coding agent:

```text
You are implementing one step of a Godot security architecture. Do not improvise outside the assigned step.

Project goal:
Rust launcher + custom encrypted Godot export + Rust GDExtension + server-authoritative backend.

Security goal:
Stop casual Godot unpacking, noob crackers, basic asset ripping, simple file tampering, direct Game.exe bypass, and client-side state cheating. Do not claim impossible security.

Hard rules:
- No malware-like behavior.
- No kernel anti-cheat.
- No invasive process scanning.
- No fake security claims.
- No permanent secrets in client.
- No private keys in repo.
- Client is untrusted.
- Server owns important progression/economy/combat state.
- Build must be reproducible.
- Add logs and tests.
- Update docs when done.
- Stop after assigned step.

Assigned step:
<insert step name>

Required output:
- Files changed.
- Commands run.
- Tests run.
- Result.
- Known issues.
- Next recommended step.
```

---

## 32. Implementation Risk Register

Risk: PCK encryption gives false confidence.

Mitigation: State clearly that backend authority is the real protection.

Risk: Launcher can be bypassed.

Mitigation: Backend requires valid launch/session token and accepted build ID.

Risk: Command-line token can be copied.

Mitigation: Token is short-lived, single-use, backend-bound, and rotated.

Risk: Rust GDExtension can still be reverse engineered.

Mitigation: Do not trust client-side Rust for authority. Use server validation.

Risk: Server authority increases hosting complexity.

Mitigation: Start with one vertical slice and expand by system.

Risk: Browser build weakens security.

Mitigation: Treat browser as untrusted and optional/limited if needed.

Risk: Anti-tamper becomes invasive.

Mitigation: Explicitly ban malware-like behavior.

Risk: False positive bans.

Mitigation: Use severity ladder, soft corrections, manual review.

Risk: CI leaks secrets.

Mitigation: Secret scanning, no key output, no secrets in artifacts.

Risk: Agents overbuild everything.

Mitigation: Use strict agent execution order and vertical slice first.

---

## 33. Best First Vertical Slice

Build this first:

Rust launcher downloads signed manifest.

Launcher verifies manifest signature.

Launcher verifies Game.exe, game.pck, and rust_security_bridge.dll hashes.

Launcher logs in to backend.

Backend issues single-use launch token.

Launcher starts encrypted Godot export.

Godot calls Rust GDExtension.

GDExtension validates launch token with backend.

Main menu opens only if valid.

Success criteria:

Launcher path works.

Direct Game.exe path fails protected online mode.

Modified game.pck is rejected or repaired.

Modified GDExtension is rejected or repaired.

Reused token fails.

Old build ID fails.

Missing manifest fails.

Invalid manifest signature fails.

This one slice proves the whole architecture.

Do not start with full combat, full inventory, browser support, or complex anti-cheat.

---

## 34. Generic Configuration Blueprint

Project security config:

```toml
[project]
game_name = "ExampleGame"
channel = "production"
build_id = "2026.05.29.001"
protocol_version = 1

[launcher]
min_launcher_version = "0.1.0"
allow_direct_game_launch = false
repair_modified_files = true
upload_crash_reports = true

[security]
require_signed_manifest = true
require_launch_token = true
require_gdextension_handshake = true
allow_offline_mode = false
token_ttl_seconds = 60

[backend]
base_url = "https://api.examplegame.com"
build_validation_required = true
session_rotation_minutes = 30

[godot]
game_executable = "Game.exe"
pck_file = "game.pck"
gdextension_library = "bin/rust_security_bridge.dll"
```

Do not ship production secrets in this config.

---

## 35. Documentation Files To Create

Every implementation should produce these docs:

SECURITY_MODEL.md

Explains threat model, goals, non-goals, and trust boundaries.

BUILD_PIPELINE.md

Explains how to build launcher, backend, GDExtension, Godot export, manifest, and package.

RELEASE_CHECKLIST.md

Step-by-step release signoff.

LAUNCHER_PROTOCOL.md

Explains manifest verification, patching, auth, token request, process start.

GDEXTENSION_API.md

Documents Godot-callable Rust methods.

BACKEND_AUTHORITY.md

Defines which game systems are server-authoritative.

BROWSER_POLICY.md

Defines browser limitations and allowed use cases.

AGENT_TASKS.md

Defines execution order and rules for coding agents.

SECURITY_AUDIT_LOG.md

Tracks security decisions, risks, and signoffs.

---

## 36. Final Do And Do-Not List

Do:

Use Rust launcher.

Use signed manifest.

Use encrypted PCK.

Use custom export template.

Use stripped release builds.

Use Rust GDExtension for selected sensitive helpers.

Use backend authority for important state.

Use short-lived tokens.

Use build ID validation.

Use telemetry cautiously.

Use release smoke tests.

Use clear documentation.

Do not:

Literally embed Godot inside a Rust window as the main strategy.

Claim the client is impossible to crack.

Trust local files after one check.

Trust command-line arguments.

Trust local saves for online progression.

Ship private keys.

Ship production secrets.

Ship raw dev files.

Use kernel anti-cheat.

Scan unrelated processes.

Use malware-like tricks.

Instant-ban from one suspicious movement packet.

Let agents skip the vertical slice.

---

## 37. Final Recommended Stack

Launcher:

Rust.

tokio.

reqwest.

serde.

blake3 or sha2.

ed25519-dalek or ring.

tracing.

egui, Slint, Tauri, or minimal native UI after core launcher works.

GDExtension:

godot-rust/gdext.

Rust release build.

Small public Godot API surface.

Backend:

Rust.

axum or actix-web.

PostgreSQL.

Redis optional.

SQLx or SeaORM.

Structured telemetry logs.

Opaque server-side sessions or short-lived signed tokens.

Godot:

Godot 4.x.

Custom export template.

Encrypted PCK.

Release-only export.

GDScript/C# for presentation.

Rust GDExtension for protected helpers.

Browser:

Godot Web export.

Server-authoritative.

No trusted secrets.

Reduced content optional.

---

## 38. Final Architecture Statement

For any Godot game that needs stronger protection, the production-grade strategy is:

Do not try to make the client impossible to crack.

Make the client untrusted.

Use a Rust launcher to control normal entry, patching, verification, and launch tokens.

Use a custom encrypted Godot export to stop casual ripping.

Use Rust GDExtension for selected sensitive helper logic and handshake/protocol code.

Use a Rust backend as the source of truth for valuable gameplay state.

Use signed manifests and short-lived sessions.

Use telemetry and validation tests.

Keep anti-tamper safe, non-invasive, and commercially defensible.

Browser builds are possible, but must be treated as fully inspectable untrusted clients.

This plan is suitable as the reusable foundation for any Godot game before implementation begins.

---

## 39. Reference Notes

Godot PCK encryption:
Godot documentation describes encrypted PCK export with a 256-bit AES key and notes that the key must exist in the binary, making it resistance against casual ripping rather than absolute protection.

Godot GDExtension:
Godot documentation defines GDExtension as the Godot 4 native extension system for loading native shared libraries and interacting with engine functionality.

godot-rust/gdext:
The godot-rust/gdext project provides Rust bindings for Godot 4 and allows Rust and GDScript to be mixed in one project.

Godot Web export:
Godot documentation describes Web export for browser deployment and lists web platform limitations. Browser builds should be treated as inspectable and untrusted.
