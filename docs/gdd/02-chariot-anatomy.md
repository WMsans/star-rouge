# Chariot Anatomy

Source decision: [Chariot anatomy: body, slots, and prop taxonomy](https://github.com/WMsans/star-rouge/issues/7).

---

## What a chariot is

A **chariot** is a **body** with **slots** that hold **props**. The player's current configuration — body, mounted props, and trigger bindings — is the **build**. Builds are the joke (pillar 1); the arena only reads them out (pillar 2).

## Body

A body is a **shaped chassis silhouette** carrying a fixed set of **named slots** at concrete local positions and orientations. It is not a freeform brick grid and not a stats shell with cosmetic placement.

- Slot count and layout are **per body type**.
- A given body does not grow new slots mid-run.
- **Prototype content:** one body only — the **Silver Chariot**. Other body types are allowed by the rule above and deferred (meta / later content).

### Silver Chariot

Starter and prototype body. **Five slots:**

| Slot | Facing / role in the silhouette |
|---|---|
| Ground left | Contacts terrain; natural home for mobility props |
| Ground right | Same |
| Front low | Forward business |
| Front high | Forward business |
| Top | Roof arc / overhead |

No rear slot. Facing labels describe the silhouette only — **slots are not typed**; any prop may occupy any slot.

## Slots

- **One prop per slot.** Empty slots are allowed.
- **Fully universal.** No wheel-only / weapon-only legality.
- **Position is physical.** A lob gun on the top slot is not the same gun on a front slot — aim, arc, and contact differ because the mount transform is real.
- **Mobility is not a slot type.** Whether a build can drive or jump is determined by **prop tags** on whatever is mounted (see below). Rolling out with no mobility prop is legal: soft warning at fight start, then let them cook. A brick is a build.

## Props

Props have **no formal categories**. They are defined only by **tags** and behavior (e.g. can-drive, can-jump, uses-trigger, material interactions, always-on vs trigger-held). Design talk, scavenge UI grouping, and enemy authoring all speak in tags — not in a type enum like “Weapon” or “Mobility.”

### Combo engine

Two props are interesting together when they form a **pipeline the arena can show**:

1. **Material pipeline** — prop A leaves or changes a material; prop B exploits it (oil drip → flamethrower).
2. **Position pipeline** — prop A moves you or them into prop B’s sweet spot (hook yanks enemy into meteor-hammer arc).

**Not** the combo engine:

- Authored pair-bonus lists (drill + oil = +X%).
- Trigger double-binding as the *prize* — sharing a trigger is scavenge **friction** (two actions, two buttons), not the joke itself.

### Prototype roster (vertical slice)

Enough props to fill five slots differently across runs and to prove both pipelines. Expand only when a new verb is missing.

| Prop | What it is for |
|---|---|
| **Stock wheels** | Baseline drive + jump. Stable. |
| **Hook** | Fast, unstable mobility. Terrain anchor pulls *you*; chariot anchor pulls *them*. |
| **Drill** | Active melee clash and terrain cutter (material setup). |
| **Meteor hammer** | Always-on chain spin while mounted; trigger brakes/releases — commitment weapon with reach. |
| **Lob gun** | Arcing ranged; mount facing changes the shot. |
| **Oil drip** | Leaves slick material. |
| **Flamethrower** | Spray / ignites flammable material. |
| **Bomb drop** | Places a charge; crater and knockabout. |
| **Plate** | Passive mass/armor; changes shove and survivability. |

### Starter fill (Silver Chariot)

**Stock wheels ×2 + drill.** Top and one front slot empty so the first scavenge is “fill,” not only “swap.” Instantly playable melee chariot; two hungry mounts.

## What this does *not* settle

- **HP and destruction** of body vs props — [HP model under full destructibility](https://github.com/WMsans/star-rouge/issues/8).
- **Trigger binding rules** (how actions map to the two triggers, including meteor-hammer brake and always-on props) — [Prop actions and trigger binding](https://github.com/WMsans/star-rouge/issues/9).
- **Building-phase UX** (soft-warn presentation, empty-slot affordances) — [Building-phase UX](https://github.com/WMsans/star-rouge/issues/10).
- **Material set** oil/flame/etc. speak to — [Arena design, hazards, and sand material taxonomy](https://github.com/WMsans/star-rouge/issues/14).
- **Physics representation** (single rigid body vs jointed props, how slot transforms become colliders) — technical tickets after HP.
- **Bodies beyond Silver Chariot** — deferred; rule only.
