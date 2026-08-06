# The Fight

Source decision: [The fight: arena shape, camera, and how two chariots engage](https://github.com/WMsans/star-rouge/issues/6).

Playable throwaway that locked the feel: branch `prototype/fight-feel` (`./scripts/run_fight_feel.sh`).

---

## What a fight is

One combat encounter between two chariots in a single arena. Ends when either side's HP hits zero. Arena kills count (via **hazard materials** / acid rain, not burial — see [HP and Damage](03-hp-and-damage.md)). Normal fights last **30–60 seconds**. Both sides start at full HP every fight.

## Arena shape

- **One fixed, bounded simulated area** — no streaming, no chunk load as you move.
- **Side walls, open sky** (no ceiling).
- **Sand over bedrock.** The material simulation sits on a permanent floor; chariots never fall out of the world.
- **Proportions (baseline):** ~**8 chariot-widths** wide; sand ~**1.25 chariot-heights** deep. Starting positions: opposite ends, on the sand surface, facing inward.
- Wide enough that a ranged chariot can kite for a few seconds and a melee chariot has a real chase; not so wide that closing the gap eats half the fight.

## Camera

**Smash-style:** tracks the midpoint of both chariots and zooms to keep both in frame. The arena can be larger than the viewport. Continuous zoom is a hard constraint for the pixel-art / resolution ticket — crisp nearest-only integer scaling is in tension with this camera.

## Engagement

Both chariots **want the same ground.** Position relative to the enemy *and* the terrain is half the damage. The fight reads as a shove contest over footing.

- **Props are mostly close-in**, but **ranged weapons are allowed and good** — a gun chariot shapes terrain to put the enemy in a worse position.
- **Balance rule:** a melee chariot must be able to reach a ranged chariot before dying. Ranged must never be "delete them at range while they can't touch me."
- **Drive left / drive right** is screen-absolute. When both want the same ground, they collide and jockey.

## Contact

- Chariots are **hard bodies** — they collide and shove; mass/momentum win the push (Build decides who bullies whom).
- **Body-on-body contact alone does almost nothing.** Passive grinding is negligible.
- **Active melee props do the clash.** A drill (or similar) pointed at the other chariot is the primary reason contact hurts — the clash is visible and obvious.
- Soft sand erodes under props and play; bedrock does not.

## Anti-stall: acid rain

Walls stop driving off-map. **Acid rain** starts at **~75 seconds** if the fight is still going — a backstop, not a normal phase. Most fights end before it falls. It forces resolution when both sides camp or stalemate.

Hazard identity and material behaviour of acid rain belong to arena / material design; this ticket only locks *that it exists as the anti-stall timer*.

## What this does *not* settle

A bare arena with only drive + one drill is intentionally decision-poor mid-fight (hold trigger, drive to mid). **Active mid-fight decisions come from prop design and environment/hazard design** — those tickets own the depth. The feel locked here is the stage those systems play on: space, camera, contact, duration, anti-stall.
