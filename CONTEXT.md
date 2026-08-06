# Star Rouge

A 2D side-view roguelike chariot-builder whose fights play out in a fully destructible, Noita-style falling-everything arena.

## Language

**Chariot**:
The player's (or enemy's) machine — a body with slots that hold props.
_Avoid_: Vehicle, car, cart, mech

**Body**:
A shaped chassis silhouette that carries a fixed set of named slots at concrete local transforms. Slot count and layout are per body type; a body does not grow slots mid-run.
_Avoid_: Frame, hull, base (when meaning the chassis), grid, brick build

**Silver Chariot**:
The starter body — five slots (two ground, two front, one top). The only body in the prototype.
_Avoid_: Default chassis, starter frame

**Slot**:
A named mount point on a body that holds at most one prop. Slots are universal (any prop, any slot). Position is physical, not nominal.
_Avoid_: Socket, hardpoint, attachment point, mount (as a noun competing with Slot)

**Prop**:
A part bolted into a slot, defined by tags and behavior rather than by a formal category enum.
_Avoid_: Part, item, module, component, attachment, weapon (as the umbrella term)

**Prop tag**:
A labeled capability or property on a prop (can-drive, can-jump, uses-trigger, material interaction, and similar). Mobility and role come from tags, not from slot types or prop categories.
_Avoid_: Prop category, prop type, item class

**Build**:
The player's current chariot configuration — body plus mounted props and their trigger bindings. The primary source of fun.
_Avoid_: Loadout, kit, setup, deck

**Trigger**:
One of the two action buttons (trigger 1, trigger 2). Props with actions are bound to a trigger during the building phase.
_Avoid_: Ability button, skill slot, action key

**Fight**:
One combat encounter between chariots in an arena. Ends when a chariot's HP hits zero; arena kills count.
_Avoid_: Match, battle, round (when meaning the combat encounter)

**Arena**:
The side-view combat space — one fixed bounded area with side walls and open sky — whose terrain is a falling-everything material simulation over bedrock.
_Avoid_: Level, map, stage, battlefield

**Bedrock**:
The permanent floor under an arena's material simulation. Chariots never fall through it.
_Avoid_: World floor, kill plane, bottom boundary (when meaning the solid floor)

**Material simulation**:
The Noita-style falling-everything cellular simulation that makes up arena terrain — powders, liquids, fires, solids, and other materials, not sand alone.
_Avoid_: Sand simulation, sand, terrain system (when meaning the multi-material sim)

**Acid rain**:
The arena anti-stall hazard that begins when a fight drags past its target length, forcing a resolution.
_Avoid_: Sudden death, timer damage, hazard rain (when meaning this specific backstop)

**Scavenge**:
The between-fight session where the player attaches props, rebinds triggers, and takes other run options before the next fight.
_Avoid_: Shop, draft, reward screen, intermission

**HP**:
A single per-fight body pool on a chariot — the only win/lose meter. Always full at fight start; props have no HP of their own.
_Avoid_: Health pool, life, durability (when meaning the win/lose condition), prop HP, ablative armor

**Sand-shed**:
Inert debris powder emitted into the material simulation from a chariot that just lost HP — scaled to the loss, plus hit juice. Not rigid scrap and not a shrinking collider.
_Avoid_: Blood particles (when meaning sim cells), gibs, prop debris, chip damage (when meaning gameplay prop loss)
