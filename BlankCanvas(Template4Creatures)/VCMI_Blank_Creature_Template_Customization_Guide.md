# VCMI Blank Creature Template — Complete Cloning and Customization Guide

This guide assumes the template folder is named:

```text
vcmi-blank-creature-template
```

The fastest and safest method is the included clone tool. A full manual method is also documented.

Before using the visual validator or contact-sheet generator, run this once on Windows:

```text
SETUP_TEMPLATE_TOOLS.cmd
```

It installs the only Python dependency used by the visual validator, contact-sheet generator and safe ZIP packer: Pillow. Creating a folder-only clone without `--zip` uses only Python’s standard library.

---

## 1. Install and test the untouched template first

1. Extract `vcmi-blank-creature-template` into:

   ```text
   %USERPROFILE%\Documents\My Games\vcmi\Mods\
   ```

2. Confirm:

   ```text
   ...\Mods\vcmi-blank-creature-template\mod.json
   ```

3. Enable **VCMI Blank Creature Template** in VCMI Launcher.
4. Restart Map Editor.
5. Place **Canvas Guardian**, **Canvas Guardian Prime**, **Canvas Workshop** and **Masterwork Atelier**.
6. Run the checklist in `LIVE_TEST_CHECKLIST.md`.

Do not start custom art until the untouched template loads correctly on your VCMI build.

---

## 2. Recommended: clone it with one command

Open Command Prompt in the extracted template folder and run:

```bat
CLONE_TEMPLATE.cmd ^
  --mod-name "Stone Lynx Creatures" ^
  --base-name "Stone Lynx" ^
  --base-plural "Stone Lynxes" ^
  --upgrade-name "Obsidian Lynx" ^
  --upgrade-plural "Obsidian Lynxes" ^
  --slug stone-lynx-creatures ^
  --author "Your Name" ^
  --zip
```

The tool creates a collision-free folder and optionally a ZIP under:

```text
generated-mods\
```

It changes:

- mod folder/namespace;
- resource root;
- base and upgrade IDs;
- display names;
- dwelling IDs and names;
- every internal JSON, animation and sound path;
- author metadata.

It does **not** invent your final stats, abilities or artwork. It deliberately preserves the working visual dimensions and anchors.

### Naming rules

Use unique ASCII slugs:

```text
stone-lynx-creatures
frost-wyrm-pack
mythic-neutral-set
```

Use camelCase internal IDs:

```text
stoneLynx
obsidianLynx
```

Never enable two mods with the same folder namespace or internal resource root.

---

## 3. Manual clone method

Only use this when you understand every replacement.

1. Copy the entire folder.
2. Rename the copied folder to a unique slug.
3. In all text files, replace:

   ```text
   vcmi-blank-creature-template
   blankcreature
   canvasGuardian
   canvasGuardianPrime
   canvas_guardian
   canvas_guardian_prime
   Canvas Guardian
   Canvas Guardian Prime
   Canvas Guardians
   Canvas Guardians Prime
   canvasWorkshop
   masterworkAtelier
   Canvas Workshop
   Masterwork Atelier
   ```

4. Rename matching resource directories.
5. Run `VALIDATE_TEMPLATE.cmd`.
6. Search the entire folder for leftover `blankcreature`, `canvasGuardian` and `vcmi-blank-creature-template` strings.

A missed namespace replacement causes cross-mod resource collisions or missing-resource icons.

---

## 4. Edit the mod metadata

Open:

```text
mod.json
```

Change:

```json
"name": "Your Mod Name",
"description": "What the mod adds.",
"version": "1.0.0",
"author": "Your Name"
```

Keep this for a normal no-Lua creature mod:

```json
"compatibility": {
  "min": "1.7.5"
}
```

Raise the minimum to a VCMI 1.8 development version when activating combat-event Lua.

---

## 5. Edit creature names, tier and numbers

Open:

```text
Content\config\<resource-root>\creatures.json
```

Important fields:

```json
"name": {
  "singular": "Stone Lynx",
  "plural": "Stone Lynxes"
},
"faction": "neutral",
"level": 4,
"cost": { "gold": 300 },
"speed": 6,
"hitPoints": 30,
"attack": 9,
"defense": 9,
"damage": { "min": 6, "max": 9 },
"fightValue": 450,
"aiValue": 500,
"growth": 5,
"advMapAmount": { "min": 10, "max": 20 },
"doubleWide": false
```

### Upgrade link

The base creature contains:

```json
"upgrades": [
  "your-mod-slug:yourUpgradeId"
]
```

To make two independent creatures instead of an upgrade pair, remove the `upgrades` array.

### Faction

Common built-in identifiers include:

```text
neutral
castle
rampart
tower
inferno
necropolis
dungeon
stronghold
fortress
conflux
```

A custom faction uses its fully qualified identifier.

### Random generation

During development, keep:

```json
"excludeFromRandomization": true
```

Change it only after balance and map assets are final.

---

## 6. Replace the UI images correctly

### Small icon

File:

```text
Content\data\<resource-root>\<creature-folder>\icon_small.png
```

Requirements:

- 32×32 RGBA;
- transparent background;
- head and chest, not full body;
- no text, Roman numerals or presentation labels;
- do not touch the edge.

Used in army/exchange/casualty-style UI.

### Large icon

File:

```text
...\icon_large.png
```

Requirements:

- 58×64 RGBA;
- head and upper torso;
- include the correct town/faction-style background;
- neutral creatures can use a neutral landscape;
- no text.

Used in town/hero-style UI. A transparent large icon produces a missing-looking background in those overlays.

### Ability icons

Use 48×48 RGBA. Keep 3–5 pixels of internal margin. Do not create 32×32 art inside a 48×48 canvas; it will look half-size.

The template includes:

```text
Content\data\<resource-root>\shared\template_ability_48.png
```

---

## 7. Replace battle art without breaking placement

Each battle PNG is 450×400 RGBA.

Source guides:

```text
source\calibration\battle_anchor_singlewide_450x400.png
source\calibration\battle_anchor_doublewide_450x400.png
source\calibration\BLANK_battle_frame_450x400.png
```

### Single-wide baseline

Start with the feet near:

```text
x = 171
y = 267
```

### Double-wide baseline

Start with the visual center near:

```text
x = 215
y = 267
```

Double-wide art always requires live testing.

### Facing

Author the creature facing **right**. VCMI mirrors the defender.

### Stable idle rule

Do not enlarge/shrink the entire sprite between idle frames. Animate:

- chest/shoulder translation by 1–2 pixels;
- hair, cloth, tail, ears or wings;
- head tilt;
- hand movement;
- subtle joint rotation.

The validator warns when idle bounding boxes change too much.

### Required groups

The template supplies all groups. Replace files without changing filenames to preserve manifests.

| Group | Purpose |
|---:|---|
| 0 | Walk |
| 1 | Hover/fidget |
| 2 | Idle |
| 3 | Hit |
| 4 | Defend |
| 5, 6 | Death |
| 7, 8 | Turn |
| 11–13 | Melee attacks |
| 14–16 | Ranged attacks |
| 17–19 | Special |
| 20, 21 | Movement start/end |
| 22, 23 | Corpse |
| 24 | Resurrection |
| 30–32 | Cast |
| 40–42 | Group attacks |
| 50, 51 | Teleport |

A non-shooter may reuse cast or attack art in groups 14–16. A non-caster may reuse special art in groups 30–32. Keeping the groups prevents invisible fallback behavior.

### Attack readability

An attack needs:

1. anticipation;
2. forward lunge or weapon travel;
3. contact frame;
4. recovery.

A tiny two-pixel twitch is not an attack animation.

### Corpse

Keep the corpse entirely inside the 450×400 canvas. Do not suddenly enlarge it. Do not make it so large that it covers several unrelated hexes.

---

## 8. Replace adventure-map sprites

Files:

```text
Content\sprites\<resource-root>\<creature-folder>\map\map_00.png
...
map_07.png
```

Requirements:

- 64×64 RGBA;
- feet near x=32, y=61;
- dedicated map view, not a tiny screenshot of a battle frame;
- calm animation;
- opaque creature interior, transparent outer background.

The map manifest repeats frames to slow playback. Keep that structure unless live testing proves a different rate is needed.

Replace map attack images:

```text
Content\data\...\map_attack_left.png
Content\data\...\map_attack_right.png
```

They are separate from battle attacks.

---

## 9. Replace sounds

Files are under:

```text
Content\sounds\<resource-root>\<creature-folder>\
```

Use:

- mono;
- PCM 16-bit;
- 44,100 Hz WAV.

Expected names:

```text
attack.wav
defend.wav
killed.wav
move.wav
startMoving.wav
endMoving.wav
wince.wav
```

The included validator checks the format.

If an editor or resizer stores color data under pixels whose alpha is exactly zero, run:

```text
CLEAN_TRANSPARENT_RGB.cmd
```

This prevents hidden RGB from becoming a white or colored matte in engines or later image-processing steps. It intentionally does not change semi-transparent antialiasing pixels, which still require visual review.

---

## 10. Customize dwellings

Config:

```text
Content\config\<resource-root>\dwellings.json
```

Change display names and creature links.

Map images:

```text
Content\sprites\<resource-root>\dwellings\<dwelling-id>\
```

Rules learned from live testing:

- keep the outer object roughly comparable to vanilla dwellings;
- do not leave transparent holes in doors, caves or windows;
- paint interiors opaque dark;
- keep the bottom visitable tile aligned with the mask;
- use `bannedForRandomDwelling: true` for nonstandard custom-map dwellings.

The template includes separate direct-recruit dwellings for base and upgraded forms.

Future town-building art must be remade for the town-screen perspective; map dwelling art is only a reference.

---

## 11. Add standard abilities

The active template has an empty `abilities` object:

```json
"abilities": {}
```

Reference examples are in:

```text
examples\ABILITY_SNIPPETS.jsonc
```

Copy a snippet into the creature and replace the icon path and description.

Official bonus documentation:

- https://vcmi.eu/modders/Bonus_Format/
- https://vcmi.eu/modders/Bonus/Bonus_Types/
- https://vcmi.eu/modders/Bonus/Bonus_Propagators/
- https://vcmi.eu/modders/Bonus/Bonus_Limiters/

Test aura propagation carefully. An ARMY-propagated bonus may appear in the modifiers of every affected stack. Use HERO propagation or Lua when the effect belongs only to the hero/spellbook.

---

## 12. Add creature spellcasting

A simple one-target creature spell can use a `SPELLCASTER` bonus. Set creature `spellPoints` when required and provide cast groups 30–32.

Two-target spells such as Sacrifice need special live testing. The target stack normally must be fully dead, and a separate living donor must be selected.

Use VCMI 1.8 development builds for mechanics that need custom targeting or event logic.

---

## 13. Add advanced VCMI 1.8 Lua mechanics

Read:

```text
examples\ADVANCED_LUA_1_8_README.md
```

Typical pattern:

1. add a scripts config to `mod.json`;
2. register a script with `"implements": "combatEvent"`;
3. attach it through a `COMBAT_EVENT_TRIGGER` ability;
4. inspect the event payload;
5. reject retaliation/ranged/summoned/clone/invalid cases explicitly;
6. mutate battle state only through server/netpack APIs;
7. store per-round state in a synchronized bonus or supported script state;
8. test multiplayer/save/load behavior.

Never assume a script can safely change battle state by modifying a local Lua object.

---

## 14. Make a shooter

1. Add:

   ```json
   "shots": 12
   ```

2. Add a `SHOOTER` ability.
3. Create projectile animation assets.
4. Configure `graphics.missile` with projectile/ray, climax frame, angles and offsets.
5. Replace groups 14–16 with real shooting animations.
6. Add `shoot.wav`.

Use an existing open-source VCMI creature as a reference and remeasure every projectile offset.

---

## 15. Make a double-wide creature

1. Set:

   ```json
   "doubleWide": true
   ```

2. Start from the double-wide calibration image.
3. Increase visual width without placing feet above the occupied hexes.
4. Test attacker and defender sides.
5. Test turns, movement, siege and narrow paths.

Do not fix a visually double-wide monster by merely making a single-wide PNG enormous.

---

## 16. Validate

Run:

```bat
VALIDATE_TEMPLATE.cmd
```

It checks:

- JSON parsing;
- mod references;
- animation references;
- required groups;
- battle/map/icon dimensions;
- empty frames;
- canvas-edge contact;
- idle size stability;
- WAV format;
- SHA-256 manifest.

Warnings require review even when the command exits successfully.

Then regenerate and inspect the actual-asset contact sheet:

```text
MAKE_ACTUAL_ASSETS_PREVIEW.cmd
```

Review these files:

```text
QA_REPORT.txt
MANIFEST_SHA256.txt
preview\ACTUAL_PACKAGED_ASSETS.png
```

The contact sheet is generated from the PNG files currently referenced by the cloned mod. Regenerate it whenever final art changes. Never use a concept poster as release evidence.

---

## 17. Live-test before packaging

Use `LIVE_TEST_CHECKLIST.md` in a disposable map.

Critical failures:

- missing-resource prohibition symbol;
- invisible model;
- model one hex north-east;
- wrong facing;
- recruitment preview empty or clipped;
- white alpha halo;
- idle scaling/pulsing;
- static walk/attack/cast;
- incorrect icon backgrounds;
- ability icon too small/large;
- dwelling transparent holes.

Do not work around a failed asset by changing gameplay collision or creature size.

---

## 18. Package

After validation and contact-sheet generation, run:

```bat
PACK_MOD.cmd
```

`PACK_MOD.cmd` regenerates the contact sheet, reruns validation, verifies the archive, and writes a SHA-256 file beside the ZIP.

The ZIP must contain exactly one top-level folder:

```text
your-mod-slug\mod.json
```

Install updates by deleting the old folder and extracting the new one. Do not merge versions.

---

## 19. Repeat safely for many creatures

For each new independent mod:

1. run the clone tool with a new slug;
2. test the untouched clone;
3. edit names/stats;
4. replace small/large icons;
5. replace one complete animation group at a time;
6. test after each major group;
7. replace map assets;
8. replace sounds;
9. customize dwellings;
10. add abilities last;
11. validate;
12. live-test;
13. package and checksum.

Keeping each creature pair in a separate mod is easiest for development. Once stable, several creature configs can be combined into one larger pack as long as all IDs and resource paths remain unique.

---

## 20. Recommended order of work

1. Clone and namespace.
2. Stats and names.
3. Small/large icons.
4. One correct idle frame.
5. Recruitment preview and battle anchor.
6. Full idle/fidget.
7. Walk and turns.
8. Attacks.
9. Defend/hit/death/corpse/resurrection.
10. Cast/special/teleport.
11. Map sprite and attacks.
12. Sounds.
13. Dwelling.
14. Standard abilities.
15. Lua mechanics.
16. Final contact sheet and package.

This sequence catches structural and placement failures before expensive animation work.
