# HP and Damage

Source decision: [HP model under full destructibility](https://github.com/WMsans/star-rouge/issues/8).

---

## What HP is

**HP** is a **single body pool** on each chariot — the only win/lose meter. A fight ends when either side’s HP hits zero.

- Props have **no HP** and are not ablative cover.
- **Default max HP is shared** for the player and normal enemies in the prototype (elites/boss may break this later).
- HP is **per-fight only**. Every chariot enters every fight at **full HP**. Nothing in scavenge reads or writes HP.

## Props stay bolted

For the duration of a fight, mounted props **do not**:

- shear off under force,
- soft-break or jam,
- burn away as a second life bar.

The build you rolled in with keeps its verbs until KO. Parts are gained and lost in **scavenge**, not mid-brawl. (A future prop may *author* a self-discard verb; that is content, not the default HP model.)

## What drains HP

Only these sources may touch the body pool:

| Lethal | Not lethal |
|---|---|
| Active **props** — melee while active, projectiles, explosions, prop-applied DoTs (yours or theirs) | Passive body grind (≈ none; see [The Fight](01-the-fight.md)) |
| **Hazard materials** on contact (lava, acid, and whatever [arena materials](https://github.com/WMsans/star-rouge/issues/14) names) | Burial, pin, plain sand / stone / dirt |
| **Acid rain** (anti-stall) | Fall damage, wall-slam impact |

**Burial and crush do not kill.** Soft geometry may displace or pin a chariot; it never drains HP and is never an insta-lose.  
**“Arena kills count”** means a **hazard material** (or acid rain) zeroed the pool — e.g. a bomb drops someone into lava — not that the sand pile got deep enough.

### Self-damage

**Full friendly fire.** Props and leftover hazards you create can drain **your** own HP. Greedy bomb/flame builds can backfire; tune yields rather than granting immunity. Brief muzzle/self-contact grace so a drill does not tick its own body is an implementation detail, not a design exception to self-damage.

## Survivability props (Plate)

**Plate** (and similar) does **not** add max HP and does not break.

It provides:

1. **Mass / shove** — inertia and push contests.
2. **Modest damage reduction vs prop damage only** — not vs hazard materials or acid rain.

Armor does not make sitting in lava safe. Less HP lost also means less sand-shed (below).

## Reading HP mid-fight

- **HUD bars only** — fixed screen chrome for you and them. No world-space bar over the chariot.
- **No numeric readout** during the fight.
- Same bar shape language both sides; distinguished by placement and color.
- Low HP: fill **color shift** and light **pulse**.
- Exact bar art → [pixel-art spec](https://github.com/WMsans/star-rouge/issues/15).

## Damage feedback: sand-shed + juice

When a chariot **loses HP**, the **victim**:

1. **Emits inert debris powder** (sand or a dedicated debris material) into the **material simulation**, preferably near the impact point; amount **scales with HP lost**.
2. Plays **hit juice** (flashes, shake, particles, scale pops — **no SFX in the prototype phase**).

Further rules:

- Silhouette and gameplay colliders **stay intact** while alive — shed is blood/debris, not a shrinking hitbox and not rigid-body scrap.
- **Death** = final large burst of debris + juice, then fight-end.
- Continuous drains (hazards, acid rain) → continuous light shed.
- Material identity of the debris powder is owned by arena/materials; this chapter only requires **non-hazard inert powder that piles**.

## Scavenge

Scavenge **does not interact with HP**. Auto-full after each fight removes heal-as-top-up from the model. The provisional scavenge option **heal** is **dropped** from the pool; [Scavenge session and the money economy](https://github.com/WMsans/star-rouge/issues/12) redesigns options without it.

## What this does *not* settle

- Hazard material set and DPS curves — [Arena design, hazards, and sand material taxonomy](https://github.com/WMsans/star-rouge/issues/14).
- Elite/boss HP exceptions and telegraphs — [Enemy chariots](https://github.com/WMsans/star-rouge/issues/13).
- Prop action cadence, ammo, heat — [Prop actions and trigger binding](https://github.com/WMsans/star-rouge/issues/9).
- How sand-shed is injected into the GPU material sim in engine — coupling / transfer tickets.
- Absolute HP numbers and damage tuning for the 30–60s fight target — playtest.
