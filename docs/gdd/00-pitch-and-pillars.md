# Star Rouge — Pitch & Design Pillars

Source decision: [Design pillars and the one-page pitch](https://github.com/WMsans/star-rouge/issues/2).

Every later design and technical decision must justify itself against the pillars below.

---

## One-page pitch

**One-liner.** A 2D side-view roguelike where you bolt weapons onto a chariot and fight in a fully destructible, Noita-style falling-everything arena — in five buttons.

**The loop.** Fight → scavenge → rebuild → fight. Between bouts you attach props, bind their actions to two triggers, and roll out again. A fight ends when a chariot's HP hits zero — yours or theirs. The arena can kill, and that counts.

**What you do in a fight.** Drive left, drive right, jump (wheels push down), trigger 1, trigger 2. No more inputs. Depth is what you bound before the gate opened.

**What makes a build matter.** Props are physical. Slot position is real. A gun on the roof is not a gun on the flank. Combos are the game — the material simulation exists to make those combos *readable* when they go off (a drill's tunnel through dirt, a bomb dropping someone into lava, a flood filling a trench you cut), not to win the fight on its own.

**Pillars.**

1. Builds are the joke.
2. Destruction is the punchline, not the bit.
3. Five inputs, full expression.

**Who it's for.** Casual roguelike players who liked the *decision* of Slay the Spire more than the grind, and who grin when a Rube Goldberg plan actually works.

**Platform.** PC/Steam, keyboard + gamepad, 60fps on modest hardware.

---

## Design pillars

### 1. Builds are the joke

The delight is the player's machine. Fights, material simulation, and scavenge exist to set up and punch that joke.

**Forbids.** Content that is fun even with a boring build; set-pieces that ignore loadout.

### 2. Destruction is the punchline, not the bit

The falling-everything simulation *reads out* why a build worked — a drill's tunnel, a bomb's sinkhole, a flood in a cut trench. Legible cause → effect.

**Forbids.** Spectacle that wins fights without the build; destruction-as-noise.

### 3. Five inputs, full expression

The whole game fits in drive left, drive right, jump, trigger 1, trigger 2. Depth is *what* is bound, not more buttons.

**Forbids.** Modes, stances, or skill trees mid-fight; actions that need a sixth key.

---

## Standing preferences (not pillars)

These steer later tickets but are not yardsticks every decision must answer:

- **System priority.** Building is the primary source of fun; destruction is the feedback channel that makes builds legible; the run/scavenge loop is the structure that forces build decisions over time.
- **Second-run hook.** A run should leave the player with "next time, X with Y" — untried combos, not meta-unlocks alone.
- **Restart tax.** ~2 minutes to a working machine that differs from last run; ~15 minutes to competence (from vehicle-builder prior art).
- **No sold fantasy.** Identity order used while deciding (mad engineer > demolition > gladiator) is internal orientation only. The pitch and GDD do not sell a role-fantasy; fun is mechanical.
- **View.** 2D side-view — not top-down. Camera and arena shape locked in [The Fight](01-the-fight.md): Smash-style midpoint+zoom camera, fixed bounded arena with side walls and open sky.
