# Star Rouge

A 2D side-view roguelike chariot-builder whose fights play out in a fully destructible, Noita-style falling-everything arena.

## Language

**Chariot**:
The player's (or enemy's) machine — a body with slots that hold props.
_Avoid_: Vehicle, car, cart, mech

**Body**:
The chassis of a chariot; it carries the slots.
_Avoid_: Frame, hull, base (when meaning the chassis)

**Slot**:
A mount point on a body that holds one prop. Position is physical, not nominal.
_Avoid_: Socket, hardpoint, attachment point

**Prop**:
A part bolted into a slot — weapon, tool, wheel, passive piece, or similar.
_Avoid_: Part, item, module, component, attachment

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
The win/lose meter for a chariot. A fight ends when one side's HP hits zero.
_Avoid_: Health pool, life, durability (when meaning the win/lose condition)
