<div align="center">

<img src="docs/images/logo.png" alt="Diamond Rush" width="400">

# Diamond Rush Web

A browser port of *Diamond Rush* (J2ME, Gameloft, 2006) that runs the original game logic.

[![Test and publish](https://github.com/formae-group/diamond-rush-web/actions/workflows/pages.yml/badge.svg)](https://github.com/formae-group/diamond-rush-web/actions/workflows/pages.yml)
[![play in the browser](https://img.shields.io/badge/play-in%20the%20browser-2ea44f.svg)](https://formae-group.github.io/diamond-rush-web/)
[![objectives verified: 47/47](https://img.shields.io/badge/objectives%20verified-47%2F47-brightgreen.svg)](docs/PLAYABILITY.md)
[![platform: Web](https://img.shields.io/badge/platform-Web-0078D6.svg)](#building)

**[Play it on GitHub Pages](https://formae-group.github.io/diamond-rush-web/)**

</div>

Diamond Rush Web plays the Sony Ericsson S700 1.2.0 (non-padlock) release in the browser. Inside a
stage it is not a remake: the pinned S700 decompilation is translated mechanically into TypeScript at
build time and runs at the handset's 20 frames per second on a minimal MIDP layer over Canvas 2D. The
original code draws the stage, the HUD, the dialogues, the result screens and the bosses; the port
adds the menus, world maps, shop, translations, controls and saving around it, fed by the record the
game writes. Every route that proves a stage can be finished is replayed tick by tick against the
same decompilation recompiled on the JVM.

The repository contains no game data and no decompiled code. The build reads both from a pinned clone
of the decompilation, and the extracted resources and the generated core are never committed.

## Features

- **Original game logic.** The 11 classes of the decompilation run as generated TypeScript with
  Java semantics preserved: 32-bit `int` overflow, integer division, `long` as `BigInt`, `switch`
  fallthrough, labelled breaks and Java's 48-bit `java.util.Random`. See
  [docs/NUCLEO_S700.md](docs/NUCLEO_S700.md).
- **Verified against the JVM.** The generated core and the recompiled Java must agree on every tick:
  game state, hero position and animation, health, lives, diamonds, keys, and hashes of the tile and
  object planes. At the current revision 67 of 67 sequences are identical (47 routes and 20 random
  sequences).
- **Every stage finishable.** All 47 completion objectives across the 41 maps (the tutorial and
  40 stages) have an approved route made only of controls, each also finished on the JVM, including
  the three bosses and the ten secret stages ([docs/PLAYABILITY.md](docs/PLAYABILITY.md)).
- **The original campaign.** New Game plays the tutorial of Angkor with its scenes, then Angkor 1
  starts from the record the tutorial wrote; after each stage come the original results screen and
  the world map. The seal, the maps of the three worlds, the armour shop (maximum health 5 to 8) and
  the secret stages read and write the same RMS record the game uses.
- **Beyond the original screen.** In any window shape the stage can continue past the 240×320 view:
  the world backdrop, the scenery, or the scenery with its objects, painted by the game's own drawing
  routine so creatures off screen move and animate as they do on it. The HUD bars can span the
  window, and screen shake, cutscene bars and fades cover all of it.
- **Presentation.** Integer, fit or stretched scaling; sharp, smooth or CRT filters; fullscreen; an
  FPS and tick readout.
- **Sound.** The 21 original MIDI tracks play through General MIDI samples (GeneralUser GS) or a
  simple oscillator synth, with the original rule of one sound at a time by priority
  ([docs/AUDIO.md](docs/AUDIO.md)).
- **Input.** Keyboard, gamepad and touch. The touch controls sit over the picture and can be moved,
  resized and switched between a d-pad and an analog stick.
- **Languages.** The original English, plus the port's Brazilian Portuguese and Spanish
  translations, drawn with the original font.
- **Port extras.** Game speed, fast-forward, rewind, save states and cheats. All are off by default,
  and none is used by the verified routes or the fidelity tests.
- **Preservation lab.** Inspect the 41 maps, play a single stage with chosen equipment, browse sprites
  and palettes, listen to the 21 MIDI tracks, step one tick at a time, and export or import replays,
  sessions and the RMS record.

## Reference version

The port follows one release. Its logic and resources come from a pinned commit of the community
decompilation, and other releases are used only for comparison; no resource from another release is
loaded at runtime.

| Field | Value |
|---|---|
| Release | Diamond Rush 1.2.0, non-padlock, Sony Ericsson S700 |
| Decompilation | [palaceswitcher/Diamond-Rush-Decomp](https://github.com/palaceswitcher/Diamond-Rush-Decomp) at [`5e05c42`](https://github.com/palaceswitcher/Diamond-Rush-Decomp/commit/5e05c42aa1aae3377790600eb6d27497101b79e7) |
| Logic | The 11 classes of `src/*.java`, translated by `tools/j2ts` |
| Resources | `res/`, unpacked into `public/assets/` by `npm run extract-assets` |
| Researched JAR | Manifest 1.2.0, SHA-256 `305e5f778014500833377953e60c2074ada5c1f15abf7a99c83d830ceda50d49`; 40 of 40 resources identical to the decompilation |

The equivalence this project demonstrates is with the **recompiled decompilation**, not with the
JAR's bytecode: a decompilation error would be reproduced faithfully, and identical resources do not
by themselves prove identical code. See [docs/VERSION_MATRIX.md](docs/VERSION_MATRIX.md) and
[docs/ORIGINAL_JAR.md](docs/ORIGINAL_JAR.md).

## Building

Requirements: Node.js 22.18+ with npm, Git, and JDK 21 (`generate-core` uses `javac` internals that
JDK 17 lacks). The tools expect the decompilation two levels above the repository:

```text
workspace/
├─ outputs/diamond-rush-web/    ← this repository
└─ work/reference-s700/         ← the pinned decompilation
```

```powershell
git clone https://github.com/formae-group/diamond-rush-web.git outputs/diamond-rush-web
git clone https://github.com/palaceswitcher/Diamond-Rush-Decomp.git work/reference-s700
git -C work/reference-s700 checkout 5e05c42aa1aae3377790600eb6d27497101b79e7
cd outputs/diamond-rush-web
npm ci
npm run extract-assets
npm run generate-core
npm run dev -- --port 5173
```

Open `http://127.0.0.1:5173/`. `npm run build` type-checks and writes the static site to `dist/`.

| Command | Effect |
|---|---|
| `npm run extract-assets [res-dir]` | Unpacks the original resources into `public/assets/` (ignored by git) |
| `npm run generate-core [clone]` | Patches the decompilation, translates it and writes `src/generated/s700.ts` (ignored by git) |
| `npm run dev` | Development server on `127.0.0.1` |
| `npm run build` | TypeScript check and production build in `dist/` |
| `npm run preview` | Serves the production build |

| Variable | Effect |
|---|---|
| `DIAMOND_JDK_BIN` | The JDK's `bin` folder, when `java` and `javac` are not on `PATH` |
| `DIAMOND_REFERENCE` | Another path to the decompilation clone |
| `DIAMOND_ORACLE_WORK` | Where the compiled JVM oracle is cached (default `work/oracle-s700`) |

The generated core records the SHA-256 of the patched sources; routes recorded on another core are
refused. Every push to `main` runs the same steps on GitHub Actions, then the tests and both JVM
comparisons, and publishes to GitHub Pages only when all of them pass.

## Playing

The page shows only the game, filling the window. The ⚙ button (or **O**) opens the port's options;
it hides during play and returns when the pointer moves or the screen is touched, and the game is
suspended while the panel is open.

| Action | Keyboard | Gamepad | Touch |
|---|---|---|---|
| Move | Arrow keys, WASD | D-pad, left stick | D-pad or analog stick |
| Act / confirm | Space, Enter | A | ◆, OK |
| Pause / back | Escape | | Ⅱ, ◂ |
| Back to checkpoint | R | | ↺ |
| Options panel | O | | ⚙ |
| Fullscreen | F | | |
| Fast-forward (hold) | Tab | | » |
| Rewind (hold) | Backspace | | « |
| Save / load state | F5 / F7 | | |

**R** is the handset's `*` key: it returns to the last checkpoint, and costs a life outside the
checkpoint's circle. On the game's own screens (results, the seal, the shop after a crystal),
**Escape** acts as the handset's right soft key.

### Options

Settings are kept in the browser (`localStorage`), and the defaults reproduce the original game. The
full reference is [docs/OPCOES_DO_PORT.md](docs/OPCOES_DO_PORT.md).

| Tab | Setting | Values |
|---|---|---|
| Game | Language | English (original), Brazilian Portuguese, Spanish |
| | Game speed | 0.5× to 2× of the handset's 20 frames per second |
| | Fast-forward | 3× while Tab or » is held |
| | Rewind | Back in time while Backspace or « is held, up to 30 s |
| | Save states | F5 saves and F7 loads, per stage |
| | Cheats | Infinite lives, infinite health, all equipment |
| Video | Scaling | Integer pixels, fit, stretch |
| | Filter | Sharp, smooth, CRT |
| | Around the original screen | Black, endless backdrop, scenery, scenery and objects |
| | Game interface | Follows the window (default), original 240×320 |
| | Show FPS and tick | Off (default), on |
| Audio | Sound, vibration | The same switches as the game's Options menu |
| | Synthesizer | GM samples, simple oscillators (no download) |
| | Volumes | Master, music, effects |
| Controls | Touch controls | Automatic, always, never |
| | Layout | Drag each group; right-handed, left-handed and compact presets; size, opacity, dead zone, haptics |
| Lab | Preservation tools | Maps, sprites, palettes, MIDI tracks, single stages, tick stepping, replays, RMS record |

Rewinding cuts the session at the point it returns to, and a state loaded after a page reload is
rebuilt by replaying the session, so both stay exact. Cheats change the original game, and their
result goes into the campaign save.

## How it works

1. **Translation.** `npm run generate-core` reads `src/*.java` from the pinned clone and applies the
   harness patches of [ReferencePatches.ts](tools/routes/ReferencePatches.ts); each patch must match
   its original text exactly once, so a changed reference stops the build. The
   [J2TS](tools/j2ts/J2TS.java) translator walks `javac`'s attributed trees and emits TypeScript with
   Java semantics, and the 11 classes are written in static-initialisation order.
2. **Patches.** The MIDlet thread is not started (the host runs the frames), a debug overlay the
   decompilation added is removed, `Random` gets a fixed seed so replays are reproducible, and the
   clock and `Thread.sleep` run on a virtual handset clock. None changes gameplay beyond the fixed
   seed.
3. **Runtime.** [runtime.ts](src/core/s700/runtime.ts) provides the Java library the game uses and
   [j2me.ts](src/core/s700/j2me.ts) the MIDP layer: `Graphics` and `Image` on Canvas 2D with the eight
   `Sprite` transforms, `GameCanvas`, and an in-memory `RecordStore`. Sound and vibration are inert in
   the core; the port listens to what the game asks for.
4. **Host.** [Engine.ts](src/core/s700/Engine.ts) starts the game the original way (menu, then
   New Game), enters each stage through its world map as the game does, delivers keys as the handset
   did, and runs one pass of `cGame.run()` per 50 ms frame, logic and drawing both, including the lag
   frame the original takes after a checkpoint.
5. **Port layer.** [Simulation.ts](src/core/Simulation.ts) reads the session state, the outcome and the
   RMS record the game writes; `src/platform`, `src/render` and `src/ui` add the front end, the
   campaign save, input, sound, the surroundings and the options panel.

## Verification

```powershell
npm test                          # formats, saves, controls, routes, the chained campaign
npm run verify:routes -- --jvm    # replay every route on the core and on the JVM
npm run verify:core -- --fuzz=8   # core against the JVM, tick by tick, plus 8 random sequences
```

The JVM oracle in [tools/oracle](tools/oracle) compiles the **same** patched decompilation against
headless J2ME stubs and drives it frame by frame with key events, the virtual clock and the fixed
seed.

| Check | Compares |
|---|---|
| `verify:core` | Every route and N random sequences on both engines: each tick, every state transition (including 200 idle frames after the last input) and the six full map planes at the start, middle and end |
| `verify:routes -- --jvm` | Each route must leave play on the JVM in the same state, on the same tick, with the same position, health, lives, diamonds and keys |
| `verify:routes -- --jvm --write-report` | Regenerates [PLAYABILITY.md](docs/PLAYABILITY.md) and [PLAYABILITY.json](docs/PLAYABILITY.json) |

An approved route starts at the stage's original load and holds only per-tick controls (direction,
action, return to checkpoint). Its starting record (lives, equipment, maximum health) is declared and
justified in the file, and it is refused if it was recorded on another core.

| Test | Checks |
|---|---|
| `tests/core.test.ts` | Every stage loads with its original entrance; snapshots, replays and the New Game record |
| `tests/campaign-chain.test.ts` | New Game through the tutorial, Angkor 1–8 and the boss's crystal, carrying the record; secret exits, game over, the Bavaria notice, the shop on the seal |
| `tests/campaign.test.ts` | Map links, world access and rewards |
| `tests/store-localization.test.ts` | Seal thresholds, crystals, armour prices and tiers, coverage of the translations |
| `tests/save.test.ts` | The RMS record layout, equipment and chest bytes, refusal of malformed records |
| `tests/formats.test.ts` | Packs, text decoding and all 95 original sprites |
| `tests/original-animation.test.ts` | Animation timings and frames against `cGame` |
| `tests/audio.test.ts` | Sound priorities and the one-player rules of `cSoundEngine` |
| `tests/hud.test.ts`, `tests/surround.test.ts` | The wide HUD and the surroundings paint without changing the game |
| `tests/settings.test.ts`, `tests/mobile-controls.test.ts` | Defaults reproduce the original; stored settings and touch layouts are validated |

What is not covered yet is listed in
[docs/VERIFICACAO_DAS_FASES.md](docs/VERIFICACAO_DAS_FASES.md#o-que-ainda-falta): a campaign chained
through all three worlds (Bavaria and Tibet are verified with standalone routes), a pixel comparison
with a J2ME emulator, and the handset's MIDI timbres. The Tibet boss route is the only one that draws
random numbers; it finishes with all 50 verification seeds and with 978 of 1000 in a larger sample.

## Documentation

Most documents under `docs/` are written in Portuguese.

| Document | Contents |
|---|---|
| [NUCLEO_S700.md](docs/NUCLEO_S700.md) | The generated core, its patches, the host frame and the JVM comparison |
| [VERIFICACAO_DAS_FASES.md](docs/VERIFICACAO_DAS_FASES.md) | How stages are verified, writing routes, and what remains |
| [PLAYABILITY.md](docs/PLAYABILITY.md) | Routes and objectives of every map (generated) |
| [OPCOES_DO_PORT.md](docs/OPCOES_DO_PORT.md) | Every port option: game, video, audio, controls, lab |
| [AUDIO.md](docs/AUDIO.md) | The original sound engine, the 21 tracks and the SoundFont |
| [FRONT_END_S700.md](docs/FRONT_END_S700.md) | Menus, seal and shop against the Java reference |
| [SAVE_FORMAT.md](docs/SAVE_FORMAT.md) | The RMS record and replays |
| [SOURCE_MAP.md](docs/SOURCE_MAP.md), [LEVEL_FORMAT.md](docs/LEVEL_FORMAT.md), [SPRITE_FORMAT.md](docs/SPRITE_FORMAT.md) | Extracted formats and their Java sources |
| [VERSION_MATRIX.md](docs/VERSION_MATRIX.md), [ORIGINAL_JAR.md](docs/ORIGINAL_JAR.md), [FORENSIC_AUDIT.md](docs/FORENSIC_AUDIT.md) | Release, researched JAR and resource audit |

## Credits and license

- **Game, maps, sprites, text, music and original logic:** Gameloft. The data and logic reference is
  [Diamond-Rush-Decomp by palaceswitcher](https://github.com/palaceswitcher/Diamond-Rush-Decomp),
  pinned at commit [`5e05c42`](https://github.com/palaceswitcher/Diamond-Rush-Decomp/commit/5e05c42aa1aae3377790600eb6d27497101b79e7).
  The port translates its `src/*.java` and uses its `res/` at build time, without versioning either
  here.
- **Comparison sources:** [DiamondRushSource](https://github.com/kubikaugustyn/DiamondRushSource) and
  [diamondRush](https://github.com/kubikaugustyn/diamondRush) by kubikaugustyn, used for comparison and
  research only.
- **Web port and the Brazilian Portuguese and Spanish translations:** independent work in this
  repository. The translations start from the English texts of `lang.f` and `demo.f` and are not
  official.
- **Sound:** the [GeneralUser GS](public/soundfont/LICENSE-GeneralUser-GS.txt) SoundFont by
  S. Christian Collins, played with [spessasynth](https://github.com/spessasus/spessasynth_core)
  (Apache-2.0), each under its own license.

*Diamond Rush* is a Gameloft game. This is a fan-made preservation project, not affiliated with
Gameloft or with the authors of the decompilations, and it grants no new license to the original
resources or to the reference code.
