# VCMI Custom Creature Modding — Research, Live-Test Conclusions, Failures and Fixes

**Purpose:** preserve the full practical knowledge gained while building and repeatedly repairing the Moonfang Werewolves, Nereid/Undine Sirens and Sepulchral Lamassu mods. This is written so a new ChatGPT conversation, another AI, or a human modder can continue without repeating the same failures.

**Scope:** VCMI creature mods using direct PNG animation manifests, custom icons, sounds, dwellings and optional VCMI 1.8 Lua combat-event scripts.

**Last verified:** 30 August 2026. When continuing in a new chat, provide this document together with the latest mod ZIP and its `QA_REPORT.txt`.

---

## 1. Engine choice and version reality

Native HotA is excellent as a game expansion but is not distributed as a general-purpose public SDK for arbitrary extra creature/faction IDs. VCMI is the practical engine for data-driven new creatures, factions, spells, artifacts, skills, map objects and Lua-based mechanics.

As verified on 30 August 2026, the latest stable VCMI release is **1.7.5**. VCMI development/daily builds are available separately. The custom combat-event mechanics used for same-tier Lycanthropy and once-per-round Funerary Reprisal require the newer Lua combat-event API and therefore were developed against **VCMI 1.8 development builds**.

Official sources:

- VCMI releases: https://github.com/vcmi/vcmi/releases
- Development builds: https://download.vcmi.eu/branch/develop/
- Creature format: https://vcmi.eu/modders/Entities_Format/Creature_Format/
- Animation format: https://vcmi.eu/modders/Animation_Format/
- Creature art guide: https://vcmi.eu/modders/Guides/Creature_Help/
- Combat event scripts: https://vcmi.eu/modders/Lua/Combat_Event_Scripts/
- Lua API: https://vcmi.eu/modders/Lua/API_Reference/
- Dwelling format: https://vcmi.eu/modders/Map_Objects/Dwelling/

---

## 2. The most important lesson: concept art is not production art

The first Lamassu package looked impressive only in `generated_preproduction_concept.png`. The actual battle/map PNGs were simplistic procedural shapes. Static validation said files existed and had transparency, but that did not prove visual quality.

**Permanent rule:**

- A concept poster is not evidence that the mod contains good sprites.
- A QA report is not evidence that the art is readable.
- Every release must include a contact sheet generated from the **actual packaged PNG files**.
- Review native-size frames and nearest-neighbor enlarged frames before calling the art finished.
- Do not describe assets as “high quality” unless the actual package was visually inspected.

---

## 3. Resource paths: explicit beats clever

The early packages produced VCMI’s missing-resource symbol: a pink/yellow/blue prohibition-like icon. Creatures were invisible in combat, editor icons were missing, and a broken symbol moved at an offset from the invisible stack.

The recurring cause was ambiguous or wrong resource paths, especially animation `basepath` concatenation and mismatches between `Content/data`, `Content/sprites` and `Content/sounds`.

**Fix pattern:**

- Use explicit complete resource paths in every animation manifest.
- Creature JSON paths are relative to the corresponding VCMI resource root:
  - battle/map animation JSON → `Content/sprites/...`
  - `iconSmall`, `iconLarge`, map-attack PNGs → `Content/data/...`
  - sounds → `Content/sounds/...`
- Do not add a second folder level by accident when installing.
- Validate every referenced file before packaging.

Example:

```json
"animation": "mymod/myCreature/battle/animation.json"
```

Inside that animation JSON:

```json
"frames": [
  "mymod/myCreature/battle/g02_00.png",
  "mymod/myCreature/battle/g02_01.png"
]
```

---

## 4. UI image roles are different

VCMI’s creature format distinguishes:

- `iconSmall`: exchange/army/casualty-style UI;
- `iconLarge`: town/hero-style UI.

Official documentation describes these roles, but the dimensions below are empirical values that worked during this project:

| Asset | Working size | Background rule |
|---|---:|---|
| Small creature icon | 32×32 | Transparent; head/chest crop |
| Large creature icon | 58×64 | Include faction/town-style background |
| Ability icon | 48×48 | Art centered with a small internal margin |

What failed:

- whole-body miniatures were unreadable and looked like colored blobs;
- black or blue generic backgrounds looked wrong;
- presentation text and `I`/`II` markers were accidentally baked into icons;
- transparent large icons caused missing town/faction backgrounds in hero and town overlays;
- 32×32 ability art looked half-size in its UI slot; earlier 64×64 art overflowed.

**Final rule:** small icon transparent; large icon has the appropriate background; both focus on the head and upper torso; ability artwork is prepared at 48×48.

---

## 5. Battle frame canvas, anchor and scale

Direct PNG battle frames were standardized on **450×400 RGBA** canvases in these projects. A consistent full canvas made every frame share one origin, avoiding per-frame jumps.

Empirical starting anchors after live testing:

- single-wide creature: feet/ground near **x≈171, y≈267**;
- double-wide creature: visual center near **x≈215, y≈267**.

These are project calibration values, not official VCMI constants. Final art must still be live-tested.

Symptoms of bad anchoring:

- visible model one hex north-east of its physical stack;
- creature apparently connected to a broken icon by an invisible diagonal line;
- model outside recruitment/stat preview;
- only a foot or wing visible in the preview;
- double-wide creature covering many visual tiles while collision occupied two.

**Fix pattern:**

1. Place all frames on the same 450×400 canvas.
2. Keep the feet on the same ground line.
3. Move the artwork in pixels; do not change the engine footprint to fix art.
4. Start with the calibration guide, then test in battle.
5. Adjust in small increments; record the working anchor per body type.

---

## 6. Recruitment/stat preview reuses creature animation logic

A creature may look correct on the map and still be invisible or clipped in the recruitment/stat screen. That screen uses the battle animation, so battle canvas scale and vertical placement also control the preview.

Observed failures:

- model entirely outside the frame;
- feet in the middle and head cut off;
- Lamassu zoomed so far in that only a leg was visible.

**Fix:** lower the whole battle sequence and/or reduce art scale. Never repair only the idle frame; every group must use the same anchor logic.

---

## 7. Facing and turning

Creature art should be authored facing **right**. VCMI mirrors it for the opposing side.

Nereid/Siren failures included always facing left and a movement sequence that turned left-right-left before walking.

Correct turn structure:

- group 7: first half of turning toward the viewer;
- group 8: second half of turning away;
- move once;
- turn back once.

Do not embed mirrored direction changes inside the walk group. Keep turning and walking separate.

A petrified creature freezing while facing the attacker was treated as normal engine state: Stone Gaze freezes the current animation/orientation rather than forcing the stack to rotate back.

---

## 8. Animation groups and actual movement

VCMI’s documented group map was used:

- 0 walk;
- 1 mouse-over/fidget;
- 2 idle;
- 3 hit;
- 4 defend;
- 5/6 death;
- 7/8 turn;
- 11/12/13 melee attacks;
- 14/15/16 ranged attacks;
- 17/18/19 special;
- 20/21 movement start/end;
- 22/23 corpse;
- 24 resurrection;
- 30/31/32 cast;
- 40/41/42 group attack;
- 50/51 teleport.

What failed:

- “walk” was a static picture gliding over the ground;
- “attack” was a tiny twitch or an abstract purple V;
- Sirens had no visible walk/attack/cast motion;
- idle animation inflated and deflated the whole werewolf like a balloon;
- death frames changed size or exceeded the canvas.

**Fix rules:**

- animate joints, limbs, cloth, tails, wings and body translation;
- do not animate idle by scaling the entire sprite;
- attacks need anticipation, lunge/contact and recovery;
- defend, hit and attack must be visually distinguishable;
- keep corpse frames inside the same canvas and at a believable size;
- use stable silhouette dimensions in idle frames.

---

## 9. Timing

The official creature format exposes `timeBetweenFidgets` and `animationTime` for walk, idle and ranged/cast attack timing.

Working conservative defaults from this project:

```json
"animationTime": {
  "walk": 1.2,
  "attack": 1.0,
  "idle": 14.0
},
"timeBetweenFidgets": 4.0
```

Map animations do not use the same creature timing field, so repeated frames were used to slow map idle loops.

Symptoms of bad timing:

- jogging in place;
- spasmic hover animation;
- breathing cycle repeating too rapidly;
- map idle obviously faster than classic objects.

---

## 10. Transparency and white halos

The werewolves showed white gaps around snout, claws and fur. This came from poor background removal and resampling RGB against white before transparency.

**Fix pipeline:**

- render/draw on RGBA with transparent pixels storing RGB 0,0,0;
- never flatten against white before cutting alpha;
- use premultiplied-alpha-aware resizing or ensure transparent source pixels are black;
- remove low-alpha matte fragments;
- inspect on grass, sky, dark and checkerboard backgrounds;
- ensure no important art touches the canvas border.

---

## 11. Map sprites and dwellings

Map creature sprites that worked used small dedicated canvases rather than scaled battle screenshots. Dwellings also need their own map art.

Dwelling issues:

- about 10% too large compared with classic dwellings;
- caves, doors and windows were transparent holes showing the terrain behind.

**Fix:** reduce the object art while keeping the canvas/mask stable, and paint all interior voids with opaque dark pixels. Alpha should describe the building’s outer silhouette, not its interior darkness.

Map dwelling config uses `core:creatureGeneratorCommon`. A safe custom-map mask used in these packages was:

```json
"mask": [
  "VVV",
  "VVV",
  "BAV"
]
```

For random-dwelling replacement, follow the stricter official dwelling rules. Custom 3×3 dwellings should normally set `bannedForRandomDwelling` to true.

---

## 12. Sound format

The generated sounds that loaded correctly were:

- mono;
- PCM 16-bit;
- 44.1 kHz WAV.

Provide separate attack, defend, killed, move, startMoving, endMoving and wince files. Keep volume conservative to avoid clipping against game audio.

---

## 13. Custom mechanics and Lua

### Same-tier Lycanthropy

Live test results:

- converted Swordsmen and Ogres of the same tier;
- did not convert elementals, undead or creatures of another tier;
- exact functional mechanic succeeded.

Two engine/bookkeeping quirks were observed:

1. converted units could be counted as removed original creatures and later-killed werewolves on the victory screen;
2. after the player lost, the neutral map object retained its original Swordsman appearance even though the next encounter contained werewolves.

These were accepted as non-mandatory engine presentation quirks because combat state was correct.

### Funerary Reprisal

The Lamassu successfully reacted once per round to melee attacks and cast Death Ripple instead of retaliating. Elementals and mechanical creatures were affected because normal Death Ripple filtering is not identical to “living biological creature only.” If that restriction is desired, the Lua script must filter targets and apply custom damage rather than calling the standard spell unchanged.

### Sacrifice from a creature stack

Sacrifice is a two-target spell. It requires a completely dead friendly stack and a separate living donor stack. Partially depleted stacks are not valid resurrection targets. Creature-cast two-target interaction may be awkward in development builds and must be tested directly.

### Hero-granted Summon Boat

Propagation must be scoped carefully. An army-level propagated bonus can appear in other creatures’ ability lists because they receive the propagated modifier. The intended effect is on the hero/spellbook, not a permanent modifier displayed on every allied stack. Prefer a correctly scoped HERO propagation or an adventure-map script that grants/removes the spell based on army contents.

---

## 14. Validation that matters

Static validation should include:

- strict JSON parsing;
- mod.json reference checks;
- every animation frame reference exists;
- all required animation groups exist;
- battle frame dimensions and alpha;
- map frame dimensions;
- icon dimensions;
- WAV format;
- duplicate IDs and namespace collisions;
- no art touching canvas edges;
- idle bounding-box stability;
- ZIP top-level folder correctness;
- SHA-256 manifest;
- contact sheet generated from actual assets.

Static validation cannot prove:

- live visual anchor;
- correct recruitment preview framing;
- correct defender mirroring;
- UI-specific portrait background behavior;
- multi-target spell usability;
- custom Lua synchronization;
- visual quality.

A live disposable map remains mandatory.

---

## 15. Minimum live test matrix

For every new creature and upgrade:

1. Editor creature-list icon.
2. Drag onto map.
3. Adventure-map idle.
4. Hero bottom-right status.
5. Hero army overlay.
6. Town/garrison overlay if applicable.
7. Right-click/stat panel.
8. Recruitment panel.
9. Attacker-side battle orientation.
10. Defender-side mirrored orientation.
11. Idle and hover timing.
12. Walk.
13. Up/front/down attack.
14. Defend and hit.
15. Death and corpse.
16. Resurrection.
17. Spell/cast groups if used.
18. Ability icon size.
19. Sound events.
20. Dwelling list icon, map object, visit and recruitment.

---

## 16. Version-history summary

### Sepulchral Lamassu

- 1.0: mechanics/config existed, but actual art was unacceptable procedural geometry.
- 2.1: detailed art rebuilt.
- 2.2: missing-resource paths, icons, map sprites and VFX repaired.
- 2.3: scaling, timing, anchor and dwellings adjusted.
- 2.4: battle size/anchor, backgrounds, alpha and dwellings refined.
- 2.5: further southwest placement, size reduction, lower preview, clearer walk/defend/attack and sharper art.

### Moonfang Werewolves

- 1.0: Lycanthropy worked, visuals had missing-resource and placement failures.
- 1.1: explicit paths, icons, map and VFX repaired.
- 1.2: scale, anchor, alpha, timing and dwellings adjusted.
- 1.3: models rebuilt as bipedal werewolves; stable idle; clean alpha.
- 1.4: southwest battle placement, real walk/attack, lower preview and smaller corpses.

### Nereid / Undine and Nereid Siren

- 1.0/1.1: concept-derived assets and mechanics.
- 1.2: broken paths, icons, previews and dwellings repaired.
- 1.3: base renamed Undine, facing/anchor/background/timing adjusted.
- 1.4: turn sequence, walk, attacks, casting, defend, death and preview position rebuilt.

---

## 17. Release discipline

- Never merge a new version over an old mod folder; delete the old folder first.
- Keep internal IDs stable when updating maps/saves.
- Change IDs only when making a genuinely separate mod.
- Increase semantic version for each public package.
- Preserve a SHA-256 checksum.
- Include a short migration note.
- Do not redistribute proprietary HoMM3/HotA/WoG art without permission.
- Keep source/concept rights clearly documented.

---

## 18. Final baseline used by the blank template

The accompanying blank template encodes the most conservative known baseline:

- VCMI 1.7.5-compatible core mod;
- optional VCMI 1.8 Lua examples kept inactive;
- explicit paths;
- 450×400 battle canvas;
- 64×64 map frames;
- 32×32 transparent small icon;
- 58×64 large icon with neutral background;
- 48×48 ability icon template;
- right-facing source art;
- stable-size idle;
- complete group coverage;
- slow idle/fidget settings;
- two correctly sized dwellings with opaque interiors;
- clone, validator and packaging tools;
- actual-assets preview generated from packaged files.
