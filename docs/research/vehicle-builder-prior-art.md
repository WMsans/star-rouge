# Prior art: vehicle-building combat roguelikes

Research findings for [Star Rouge #4](https://github.com/WMsans/star-rouge/issues/4), a child of the
[design + technical architecture map](https://github.com/WMsans/star-rouge/issues/1).

**Question.** What has already been built in the space where vehicle/machine building meets combat
and/or roguelike runs? How is building presented, how much complexity do casual players tolerate,
how are fights framed, and where do reviews and developer statements say the design broke down?

**Scope of this document.** Facts and cited lessons only. No design decisions for *Star Rouge* are
made here — later tickets own those. Anything not directly attributable to a source is labelled
**[inference]**.

**Date of data capture.** 2026-08-05. All Steam review counts and percentages are point-in-time
snapshots read from the live store pages on that date and will drift.

**Source-trust notes.**

- Steam store pages and Steam review listings are first-party: the store copy is the developer's own
  description, and review text is the players' own words. Aggregated review percentages are Valve's.
- Steam "negative reviews, top rated" listings are used deliberately as the primary evidence for
  *where designs broke down*. They are self-selected complaints, not a representative sample — a
  recurring theme across the top-rated negatives is evidence that the complaint resonates with many
  players, not that most players hold it.
- Two intended primary sources could not be read: the PC Gamer *Besiege* review and GodisaGeek's
  *Nimbatus* review are paywalled/403 to this agent. The GDC talks cited below are referenced by
  their official session/vault entries and contemporaneous *Game Developer* coverage rather than
  from a transcript.
- No formal developer postmortem exists (or could be found) for *Scraps: Modular Vehicle Combat* or
  *Nimbatus*; the developer statements cited for those are store-page and delisting-announcement
  material plus player reviews.

---

## 1. Reception snapshot (2026-08-05)

Ordered roughly from "toybox builder" to "builder inside a competitive/roguelite loop". The point of
the table is the *shape* of the pattern, not the individual numbers.

| Game | What it is | Steam positive % | Reviews (all langs) |
|---|---|---|---|
| Brick Rigs | 3D brick sandbox, destruction physics | 95% | 53,017 |
| Besiege | 3D physics siege-engine builder, level goals | 94% (23,262 EN) | 47,898 |
| Cosmoteer | 2D grid starship builder + module-targeted combat | 94% (7,539 EN) | 11,919 |
| Trailmakers | 3D block vehicle builder, "easy to learn, hard to master" | 93% (22,740 EN) | 32,621 |
| Forts | 2D destructible fort builder, real-time PvP artillery | 93% (8,824 EN) | 25,196 |
| Scrap Mechanic | 3D creation sandbox, 400+ parts | 92% | 121,144 |
| TerraTech | 3D block vehicle builder, open-world campaign | 92% | 13,196 |
| TerraTech Legion | Block vehicle builder inside a bullet-heaven roguelite | 81% | 1,931 |
| Nimbatus | 2D part+logic drone builder, roguelite campaign | 79% | 1,752 |
| Crossout | 3D part vehicle builder, F2P PvP | 75% overall / 68% English | 69,602 |
| Scraps | 3D modular vehicle combat | 77% (SteamSpy) | delisted 2023 |

Reference points from the map, for calibration:

| Game | Steam positive % | Reviews (all langs) |
|---|---|---|
| Slay the Spire | 97% (76,513 EN) | 219,033 |
| Brotato | 96% (31,817 EN) | 117,480 |
| Noita | 95% (47,165 EN) | 80,625 |

Sources: individual Steam store pages, linked in §6.

**Pattern.** Every builder in the 92–95% band is a *sandbox or puzzle* builder where the building is
the product. Every builder that puts the same building system inside a run structure or a
competitive ladder lands 15–20 points lower. **[inference]** The drop is not evidence that
building-plus-combat is a bad genre — the two lowest are also the two smallest and least resourced —
but it is consistent with the failure themes in §3 and §4, which are all about the *interaction*
between building and a run/match, not about building itself.

---

## 2. How building is presented

### Besiege (Spiderling Studios, 1.0 on 2020-02-18)

Store copy calls it an "intuitive & flexible building system" with an "extensive 70+ block
collection", and frames the goal as "siege engines, battering rams, ballistas — anything you can
imagine". Building is free-form 3D block attachment against a physics sim that models weight and
structural failure. Fights are framed as *levels with a destruction objective*, not as duels: you
build in peace, then press go and watch.
([store page](https://store.steampowered.com/app/346010/Besiege/))

### Trailmakers (Flashbulb, 2019-09-18)

Store copy is explicit about the accessibility ladder: building is "intricate yet intuitive", players
may either use **pre-made vehicles** or build "from scratch using hundreds of blocks", and the pitch
line is "Easy to learn, hard to master!". The store page names aerodynamics, functionality, speed and
combat ability as things the build affects.
([store page](https://store.steampowered.com/app/585420/Trailmakers/))

**Transferable:** shipping the game with ready-made vehicles as a first-class option, not a tutorial
crutch, is a documented commercial choice in the highest-reviewed accessible builder in the set.

### Bad Piggies (Rovio, 2012) — the mass-casual end of the spectrum

The most-played vehicle builder ever made is a casual mobile puzzle game. Players assemble a
contraption on a **grid** from "attachable objects such as crates, wheels, bellows, balloons,
umbrellas, soft drink bottles, and coil springs, with more being unlocked as the game progresses" —
42 objects total, unlocked gradually. Each level has three star goals (complete it, plus two variable
objectives). A "Mechanic Pig" character **pre-assembles** a transport option so the player can skip
construction and just drive. Metacritic 83/100 (iOS, 24 reviews); IGN 9.2, TouchArcade 5/5, Pocket
Gamer 8/10 with a Silver Award. Reached **100 million downloads by February 2015**. Critics praised
the departure from Angry Birds while noting the difficulty was high.
([Wikipedia](https://en.wikipedia.org/wiki/Bad_Piggies))

**Transferable, and the strongest single data point on casual tolerance in this document:** a
hundred-million-download casual audience *will* build vehicles, given a grid, a small part count
(~42, drip-fed), a single readable goal per level, and an explicit escape hatch that builds the
vehicle for you. Note that the reviews cited still called it hard — casual tolerance was bought with
presentation and scope, not by making the puzzles easy.

### Nimbatus – The Space Drone Constructor (Stray Fawn, 2020-05-14)

The closest structural analogue found: 2D, side-on, part-based vehicle assembly, and a roguelite
campaign. Store copy: "more than 75 physically simulated parts", "the drone construction workshop
enables full creative freedom", and players "construct fully automated space drones by combining
sensors and logic parts". Building includes wiring logic gates and binding parts to keys.
([store page](https://store.steampowered.com/app/383840/Nimbatus__The_Space_Drone_Constructor/))
Overall 79% of 1,752 reviews.

### Cosmoteer (Walternate Realities, EA 2022-10-24)

2D grid ship builder. Players "customize your ship's shape and floor plan, choosing where to place
individual modules", and combat is "physics-driven" with "each module can be individually targeted
and destroyed". Crew members are individually simulated and physically carry munitions from storage
to weapons, so *layout adjacency* is a combat variable, not decoration. 94% of 7,539 English reviews.
([store page](https://store.steampowered.com/app/799600/Cosmoteer_Starship_Architect__Commander/))

**Transferable:** a 2D slot-placement builder where the *position* of a part changes its combat
behaviour, with per-part destruction, sits at the top of the reception band. It is the strongest
existence proof that side-on/plan-view modular building plus destructible-per-module combat can be
extremely well received.

### TerraTech Legion (Payload Studios, 2026-04-30, $9.99)

The most directly comparable recent release: block-by-block vehicle building dropped into a casual
bullet-heaven roguelite. Store copy: "200+ blocks across weapons, propulsion and utility", players
"snap them together however you like", and "where you put things matters — weight, balance, and
whether you can actually steer it all come into play". Run structure: four planets, each ending in a
"brutal planetary boss", with skill upgrades between missions and a separate endless mode. 81% of
1,931 reviews.
([store page](https://store.steampowered.com/app/3596700/TerraTech_Legion/))

Compare with the studio's own sandbox game *TerraTech* (2018), same building system, open-world
campaign: 92% of 13,196 reviews.
([store page](https://store.steampowered.com/app/285920/TerraTech/))

### Crossout (Targem/Gaijin, 2017-07-26)

Free-to-play PvP: "create a unique design for an armoured vehicle from dozens of parts", then fight
in multiplayer battles. 75% of 69,602 reviews overall, but only **68% of 22,431 English** reviews —
the English-language audience rates it markedly worse than the overall pool.
([store page](https://store.steampowered.com/app/386180/Crossout/))

### Forts (EarthWork Games, 2017-04-19)

2D, pixel-art, physics-based: players "design, construct, and customize imposing fortresses,
bristling with devastating firepower" and "blast your opponent's buildings to rubble". Critically,
building happens **during** the fight rather than before it. 93% of 8,824 English reviews.
([store page](https://store.steampowered.com/app/410900/Forts/))

### Scraps: Modular Vehicle Combat (Moment Studio, 1.0 2020-12-18)

"A vehicle combat game where you can build your vehicle from functional parts, with design as your
ally and physics as your accomplice", with arena play against AI or humans and a Gauntlet mode where
you upgrade the vehicle as you progress. Estimated 0–20,000 owners, 77% user score, 4,057 followers.
([SteamSpy](https://steamspy.com/app/350150)) The developer announced delisting in February 2023,
citing that the game "has been selling near zero copies for quite some time", that they could not
keep supporting it forever, and that it ran on a Unity version so old that fixing it would be
impossible without a huge amount of work.
([Delisted Games](https://delistedgames.com/scraps-modular-vehicle-combat-likely-leaving-steam-on-february-20th/),
[Steam news](https://store.steampowered.com/news/app/350150/view/3673285123512481054))

**Transferable:** the one game in this set whose pitch is closest to "build a functional combat
vehicle from parts, then fight" is also the one that commercially failed and was removed from sale.
**[inference]** The developer's stated reasons are sales and engine rot, not design, so this is a
cautionary market data point rather than a design verdict — but "modular vehicle combat" as the
headline proposition has one delisting and no breakout hit behind it.

---

## 3. How much building complexity casual players tolerate

This is the section the map flags as most valuable. Everything here is cited; the synthesis lines are
marked.

**Fine-grained per-part configuration is the single most-cited friction point.** In Besiege's
top-rated negative reviews, players describe the process as "tedious and needlessly repetitive",
requiring manual block-by-block placement with no shortcuts, and specifically call out "changing key
bindings and other settings manually block by block" as something that "creates discouragement before
even attempting levels". A 209-hour player notes there are "no guides besides the basics of building".
([Besiege negative reviews](https://steamcommunity.com/app/346010/negativereviews/?browsefilter=toprated))

Trailmakers negative reviews hit the same nerve from the opposite side — bad *defaults* for bindings:
"Default controls can not be set. If you place an engine, it will be mapped to W/S", requiring manual
reconfiguration on every build. The same reviews call the build menu "a PITA with inconsistent snap,
glitchy mirror mode".
([Trailmakers negative reviews](https://steamcommunity.com/app/585420/negativereviews/?browsefilter=toprated))

**Directly relevant to the locked "props bound to a trigger by the player" mechanic:** per-part
binding is exactly the interaction that two of the best-received builders in the genre get complained
about for. The complaints are not about the concept but about (a) having to do it per part, every
build, and (b) unhelpful defaults. **[inference]** Binding cost scales with part count, so the same
interaction that is a chore across 70+ blocks may be a non-issue across a handful of slots.

**Logic and wiring layers exceed tolerance without dedicated tooling.** Nimbatus' top-rated negatives
attack the logic system on legibility, not on power: "YOU CANNOT SEE THE INPUT OR OUTPUT WITHOUT
CLICKING ON EACH INDIVIDUAL LOGIC GATE", and there is "no way to get a list of these bindings that
are already in use". Logic parts also "count against your ship's mass/part total" while suffering
"propagation delay that is too long to be relevant" — i.e. the complexity carried a cost without
paying out. Structural rules compounded it: "Drone components can only be attached to one other
component", making it "near impossible to build trusses".
([Nimbatus negative reviews](https://steamcommunity.com/app/383840/negativereviews/?browsefilter=toprated))

Niche Gamer's 8/10 review of the same game is the counterweight and is worth reading as the *success*
condition for onboarding: it praises a "simple drone building system that can be as complex as you
want it to be", says drones can be assembled in **under 2 minutes** once basics are learned, and that
"concise tutorials get you up to speed within 15 minutes" — while criticising that advanced parts get
no tutorials at all. It calls the campaign "brutal".
([Niche Gamer review](https://nichegamer.com/reviews/nimbatus-the-space-drone-constructor-review/))

**Transferable:** the tolerated shape is *shallow by default, deep on request*, with a sub-two-minute
build time for a working machine and tutorial coverage that reaches every part, not just the basics.

**Physics that punishes rather than rewards drives players to minimum-effort builds.** Besiege
negatives report the engine "will just hate some of the things you build and vibrate until it
shatters", and that "anything you make" gets "smashed to pieces", explicitly forcing players "toward
bare-minimum solutions rather than creative designs". Trailmakers negatives echo it: "parts snap off
like twigs when you bump up against a collectible rock the wrong way". Nimbatus negatives report
non-deterministic physics where designs are "either exploding or performing amazingly, depending on
your luck".
(same three review listings)

**Transferable, and load-bearing for a falling-sand arena:** a simulation that non-deterministically
destroys player creations converts a creative system into a risk-avoidance system. Two separate
well-reviewed builders are on record losing creative engagement to it.

**But building that has no consequence is also rejected — as busywork.** TerraTech Legion's top-rated
negatives complain the opposite way: "You aren't engineering a truly functional machine. Weight
distribution barely matters", and that "building and strategising how to build to be effective… is
less prominent than before", reducing the core mechanic to tedium.
([TerraTech Legion negative reviews](https://steamcommunity.com/app/3596700/negativereviews/?browsefilter=toprated))

**Transferable:** there is a narrow band. Physics consequential enough that placement matters, but not
so consequential (or so random) that the safe build wins. Both walls of the corridor are documented
by player reviews in this set.

**Creative building loses to meta building the moment fights are competitive.** Crossout negatives
report "4/8 players on both teams" running identical meta builds, that "the vast majority of players
rock variations of the 1 appropriate or possibly 2 appropriate builds", that "machine guns and
radiators just do just about everything better, which gets boring fast", and that unique vehicles
perform poorly — forcing a choice between expression and effectiveness.
([Crossout negative reviews](https://steamcommunity.com/app/386180/negativereviews/?browsefilter=toprated))

Forts, with far better overall reception, shows the same convergence in a pure-2D destructible
setting: "Rush cannon or laser and whoever gets there first basically can win", "the same strat every
game", "little diversity with weapons".
([Forts negative reviews](https://steamcommunity.com/app/410900/negativereviews/?browsefilter=toprated))

**Real-time building under combat pressure reads as stress, not strategy, to newer players.** Forts
negatives: the game becomes "stressful, tedious" when constantly managing repairs while trying to
build weapons; "rushed and chaotic" against expectations of strategic play; "one slip with building
from the start its almost a guarantee loss"; a new player was "kicked mid game for what I can only
assume was building too slowly".
([Forts negative reviews](https://steamcommunity.com/app/410900/negativereviews/?browsefilter=toprated))

**Transferable:** the separation of a build phase from a fight phase — which *Star Rouge* already has
via the locked scavenge-session structure — is exactly the design decision Forts did not make, and
Forts' newcomer complaints are concentrated on the consequence.

---

## 4. How the run/roguelite loop interacts with building

**Growing the part pool across a run or across meta-progression can make a roguelite worse, not
richer.** TerraTech Legion's top-rated negatives describe this as their central complaint: "the more
you progress, the more parts options you unlock… this just makes the game more RNG-heavy and
therefore harder", and, bluntly, "You get so many unlocks it becomes pretty much impossible to
actually make a build. As you unlock more options, you become weaker." Reviewers specifically note the
absence of any filtering mechanism.
([TerraTech Legion negative reviews](https://steamcommunity.com/app/3596700/negativereviews/?browsefilter=toprated))

**Transferable, and directly aimed at the provisional "3 scavenge phases × 3 options" structure:**
in a builder-roguelite, every unlocked part dilutes every future offer. Unlock breadth is a dilution
cost, not only a content win. Reviewers of a 2026 release in almost exactly this genre named it as
the reason the game got worse the longer they played.

**Weak meta-progression rewards are noticed and resented.** Same listing: high-tier unlocks amount to
"get one extra copy of your common starting weapon" after 10+ hours, with most upgrades being minimal
stat bumps described as "pointless padding".

**Run pacing must let the finished build be enjoyed.** Same listing: "The timer on missions usually
ends when you are just getting strong so you can't really enjoy the build you made." Forced
objectives (e.g. equipping specific corrupted blocks) created walls that felt "almost impossible"
inside the time limit.

**Transferable:** in a builder-roguelite the payoff moment *is* driving the finished machine. A run
structure that cuts the fight short at the point of peak build is a documented dissatisfaction source.

**Nimbatus' campaign shows the other failure mode: a run that does not teach the builder.** Negatives
describe "boring and repetitive quests", "small scale world design" lacking "difficult longterm
goals", and — the sharpest point for a design that wants players to explore a prop pool — players are
"never presented with instructions on how to use each part, or challenges that may utilize that part".
([Nimbatus negative reviews](https://steamcommunity.com/app/383840/negativereviews/?browsefilter=toprated))

**Transferable:** the run's encounters are the tutorial for the part pool. If no fight ever motivates
a given prop, that prop is dead weight in the offer pool.

---

## 5. The reference points named in the map

### Slay the Spire (Mega Crit, 1.0 2019-01-23) — the reward-choice structure being copied

Reception: 97% of 76,513 English reviews, 219,033 reviews all languages — the highest-rated game in
this document. Content scale at 1.0: 350+ cards, 200+ items (relics), 50+ combat encounters, 50+
story events, four characters. Framed by the developers as a fusion of "card games and roguelikes".
([store page](https://store.steampowered.com/app/646570/Slay_the_Spire/))

Design statements from co-designer Anthony Giovannetti:

- "The core design of Slay the Spire is risk versus reward."
- Choices carry *both* positive and negative outcomes, deliberately, to keep decisions from becoming
  rote.
- "Playtest more than you think you need to" — using local indie events and Discord to recruit.
- On iteration: "It's good to make something and then have it kind of suck. We learned a lot about
  finishing something, even if it's just finishing something bad."
  ([Think Like A Game Designer interview](https://justingarydesign.substack.com/p/anthony-giovannetti-crafting-slay))

From the GDC 2019 talk *'Slay the Spire': Metrics Driven Design and Balance* (Giovannetti, Mega Crit;
the game had sold over 1 million copies in its first year), as reported by *Game Developer*:

- The team built a metric server at the **early prototype stage** — before Early Access — tracking
  every decision a player made. Rationale: "We have so many cards and so many interactions that even
  though we have a pretty strong card game background there's no way we can intuitively do it all
  correctly."
- The two headline metrics are **how often players pick a card when it is offered**, and **how often
  that card appears in winning runs**.
- Instrumentation grew from 3 graphs to "at least 90".
- Scale of data after launch: "In one hour we get more data than we had throughout the whole
  prototyping phase."
- Metrics were explicitly insufficient alone: "The numbers are really useful but they're not telling
  us how things feel." A tagged-feedback Discord covered the gap.
  ([Game Developer coverage](https://www.gamedeveloper.com/design/how-i-slay-the-spire-i-s-devs-use-data-to-balance-their-roguelike-deck-builder),
  [GDC Vault session](https://www.gdcvault.com/play/1025731/-Slay-the-Spire-Metrics),
  [talk video](https://www.youtube.com/watch?v=7rqfbvnO_H0))

**Transferable:** the pick-rate ÷ win-rate pair is a domain-agnostic instrument. It applies to a prop
offered in a scavenge phase exactly as it applies to a card offered after a fight. **[inference]** For
a slot-and-prop builder the same instrumentation would extend to *which slot* a prop gets placed in
and *which trigger* it gets bound to, since those are the additional player decisions this design
introduces over a deckbuilder's.

### Brotato (Blobfish, 2023) — casual roguelite loop shape

96% of 31,817 English reviews, 117,480 all languages. Store copy establishes the casual contract
explicitly: "Auto-firing weapons by default with a manual aiming option"; "Fast runs (under 30
minutes)"; waves lasting 20–90 seconds with material collection between them; dozens of characters
and hundreds of items and weapons.
([store page](https://store.steampowered.com/app/1942280/Brotato/))

**Transferable:** the top-of-genre casual roguelite ships (a) a sub-30-minute run, (b) short discrete
waves with a shop beat between them, and (c) an input floor low enough that aiming is *optional*. It
carries hundreds of items regardless — item breadth is not what it economises on; **input and run
length are**.

### Noita (Nolla Games, 1.0 2020-10-15) — falling sand

95% of 47,165 English reviews, 80,625 all languages. Store copy: "Pixel-based physics… Every pixel in
the world is simulated. Burn, explode or melt anything", in "a simulated world that is more
interactive than anything you've seen before".
([store page](https://store.steampowered.com/app/881100/Noita/))

Petri Purho's GDC 2019 talk *Exploring the Tech and Design of 'Noita'* (2019-03-19) covers scaling the
falling-sand simulation to large continuous worlds, integrating destructible rigid-body physics
(marching squares for rigid bodies), multithreading, fine-tuning the resulting physics-based emergent
gameplay, roguelite design considerations, and fixing glitches in a physics-based game.
([GDC Vault](https://www.gdcvault.com/play/1025695/Exploring-the-Tech-and-Design),
[Game Developer](https://www.gamedeveloper.com/design/video-understanding-the-remarkable-tech-and-design-of-i-noita-i-),
[talk video](https://www.youtube.com/watch?v=prXuyMCgbTc))

*Caveat: the talk's specific claims above come from the official session description and Game
Developer's summary; the video itself was not transcribed for this document. Technical tickets that
depend on these details should watch it.*

### Worms (Team17, 1995–) — 2D destructible-terrain combat

"Turn-based artillery games presented in a 2D or 3D environment". Terrain is fully deformable: "when
most weapons are used, they cause explosions that deform the terrain, creating circular cavities",
forcing strategy to adapt as the battlefield changes. Weapon breadth is large — 60 weapons in the 2D
games, 40 in the 3D ones, with some removed from 3D titles "for gameplay reasons" — and players
customise "schemes" that preselect their preferred subset for future matches. Turns are time-limited
to control pace. Movement tools (Bungee, Ninja Rope) exist specifically to reach terrain that
deformation made inaccessible. The 3D transition (Worms 3D, 2003) did not displace the 2D games'
popularity, and later entries drew criticism for "lack of meaningful additions", with reviewers
calling sequels "expansion packs".
([Wikipedia](https://en.wikipedia.org/wiki/Worms_(series)))

**Transferable:** three mechanisms Worms uses to keep a large arsenal usable in destructible 2D —
per-match weapon *schemes* (player-curated subsets), a turn timer as a pacing governor, and dedicated
traversal tools that exist because deformation strands players. The last is the one most easily
forgotten when designing a destructible arena.

### Games named in the ticket but not covered in depth

- **Nuclear Throne** and **Death Rally** were not researched to primary sources within this ticket's
  budget. *Brotato* was used as the casual-roguelite-loop reference instead, being both more recent
  and more explicitly casual-targeted. Flagged as a gap in §7.
- **Brick Rigs** and **Scrap Mechanic** appear in the reception table but are 3D sandboxes with
  little combat framing; their evidence value here is as the top of the "building is the product"
  band.

---

## 6. Cross-cutting lessons

Each numbered lesson is anchored to sources above.

1. **The best-reviewed builders sell building as the product; putting the same builder inside a run
   or ladder costs 15–20 review points.** (§1 table.) **[inference]** on causation.
2. **A mass-casual audience will build vehicles.** Bad Piggies: grid placement, ~42 parts drip-fed,
   one goal per level, an NPC that pre-builds for you — 100M downloads, MC 83. (§2)
3. **Ship pre-made builds as a first-class option.** Trailmakers (93%) leads with it on the store
   page; Bad Piggies has an in-fiction character for it. (§2)
4. **Target a build time of ~2 minutes for a working machine and ~15 minutes of tutorial to
   competence** — the terms in which a positive review of the nearest analogue praised its
   onboarding. Cover *every* part, not just the basics; both Nimbatus and Besiege are criticised for
   stopping at basics. (§3)
5. **Per-part configuration is where builders bleed casual players.** Besiege players call
   block-by-block key binding discouraging; Trailmakers players call bad binding defaults a per-build
   chore. Bindings need good defaults and a global view. (§3)
6. **Logic/wiring layers need first-class inspection tooling or they are rejected.** Nimbatus'
   complaints are about not being able to *see* wiring state, not about difficulty. (§3)
7. **Non-deterministic or punishing physics converts creativity into risk-avoidance.** Three separate
   builders documented. (§3)
8. **But building must have consequence.** TerraTech Legion is criticised for placement "barely"
   mattering, making building busywork. Both walls of the corridor are player-documented. (§3)
9. **Competitive framing collapses build diversity.** Crossout and Forts both converge on 1–2
   dominant builds by player report, despite very different quality levels. (§3) **[inference]** a
   single-opponent PvE duel does not have Crossout's ladder pressure, but a dominant build can still
   emerge against a fixed AI.
10. **Building under combat pressure reads as stress to newcomers.** Forts' newcomer complaints are
    concentrated here; a separated build phase avoids it. (§3)
11. **Every unlocked part dilutes every future offer.** The loudest complaint about the most
    comparable 2026 release: more unlocks made the game harder and builds worse, with no filtering.
    (§4)
12. **Let the finished build be played.** Runs that end at the moment of peak build are a documented
    dissatisfaction source. (§4)
13. **Encounters teach the part pool.** A prop no fight ever motivates is dead weight — Nimbatus'
    campaign is criticised for exactly this. (§4)
14. **Instrument pick-rate and win-rate from the prototype, not from launch.** Mega Crit's stated
    reason: nobody can intuit a large interaction space correctly. Pair with qualitative channels,
    because "the numbers… aren't telling us how things feel". (§5)
15. **Economise on input and run length, not on content breadth.** Brotato: sub-30-minute runs,
    optional aiming, hundreds of items. (§5)
16. **Destructible 2D arenas need traversal answers and a pacing governor.** Worms ships rope/bungee
    tools because deformation strands players, plus turn timers, plus player-curated weapon subsets.
    (§5)
17. **"Modular vehicle combat" as a headline proposition has one delisting and no breakout.** Scraps
    sold "near zero copies for quite some time" and was pulled in 2023. (§2) **[inference]** on how
    much to read into a single failure.

---

## 7. Gaps and open questions

- **Paywalled/blocked:** PC Gamer's *Besiege* review, GodisaGeek's *Nimbatus* review. Neither GDC
  talk (Slay the Spire metrics, Noita tech/design) was transcribed here; both are freely available on
  YouTube and the GDC Vault and are worth watching directly for the tickets that depend on them.
- **Not researched:** Nuclear Throne, Death Rally, Worms W.M.D specifically, Instruments of
  Destruction, Highfleet, Reassembly, Airships: Conquer the Skies (the developer's dev blog exists at
  [zarkonnen.com/airships](https://www.zarkonnen.com/airships) and is a plausible source for 2D
  side-on destructible-ship-builder UI lessons, but no sales postmortem or UI-complexity article was
  found on it).
- **No hard data found on:** how many slots/props a casual player tolerates as a *number*. The
  evidence in §3 constrains the *shape* (grid placement, good defaults, drip-fed unlocks, ~2 min
  build) but no source states a tolerated part count. Bad Piggies' 42-objects-drip-fed and Brotato's
  "hundreds of items but a trivial input floor" are the closest anchors, and they point in different
  directions — item breadth is cheap, *per-item configuration* is expensive.
- **Emerging comparables to re-check before launch:** *Carvival* (unreleased at time of writing, no
  reviews) and *Vehicle No. 4* (released 2026-05-05, only ~25 reviews) are both very close in
  concept; too little data now, useful later.
- **Not resolved:** whether the ~15–20 point reception gap between sandbox builders and
  builder-roguelites is causal or an artefact of studio size and budget. Both low scorers are small
  productions.

## Sources

Steam store pages (read 2026-08-05): [Besiege](https://store.steampowered.com/app/346010/Besiege/) ·
[Trailmakers](https://store.steampowered.com/app/585420/Trailmakers/) ·
[Scrap Mechanic](https://store.steampowered.com/app/387990/Scrap_Mechanic/) ·
[Brick Rigs](https://store.steampowered.com/app/552100/Brick_Rigs/) ·
[Forts](https://store.steampowered.com/app/410900/Forts/) ·
[Crossout](https://store.steampowered.com/app/386180/Crossout/) ·
[Cosmoteer](https://store.steampowered.com/app/799600/Cosmoteer_Starship_Architect__Commander/) ·
[Nimbatus](https://store.steampowered.com/app/383840/Nimbatus__The_Space_Drone_Constructor/) ·
[TerraTech](https://store.steampowered.com/app/285920/TerraTech/) ·
[TerraTech Legion](https://store.steampowered.com/app/3596700/TerraTech_Legion/) ·
[Vehicle No. 4](https://store.steampowered.com/app/3363890/Vehicle_No_4/) ·
[Carvival](https://store.steampowered.com/app/2626430/Carvival/) ·
[Slay the Spire](https://store.steampowered.com/app/646570/Slay_the_Spire/) ·
[Brotato](https://store.steampowered.com/app/1942280/Brotato/) ·
[Noita](https://store.steampowered.com/app/881100/Noita/)

Steam top-rated negative review listings (read 2026-08-05):
[Besiege](https://steamcommunity.com/app/346010/negativereviews/?browsefilter=toprated) ·
[Trailmakers](https://steamcommunity.com/app/585420/negativereviews/?browsefilter=toprated) ·
[Forts](https://steamcommunity.com/app/410900/negativereviews/?browsefilter=toprated) ·
[Crossout](https://steamcommunity.com/app/386180/negativereviews/?browsefilter=toprated) ·
[Nimbatus](https://steamcommunity.com/app/383840/negativereviews/?browsefilter=toprated) ·
[TerraTech Legion](https://steamcommunity.com/app/3596700/negativereviews/?browsefilter=toprated)

Developer talks and interviews: [GDC Vault — 'Slay the Spire': Metrics Driven Design and
Balance](https://www.gdcvault.com/play/1025731/-Slay-the-Spire-Metrics) ·
[video](https://www.youtube.com/watch?v=7rqfbvnO_H0) ·
[Game Developer coverage](https://www.gamedeveloper.com/design/how-i-slay-the-spire-i-s-devs-use-data-to-balance-their-roguelike-deck-builder) ·
[Anthony Giovannetti interview, Think Like A Game Designer](https://justingarydesign.substack.com/p/anthony-giovannetti-crafting-slay) ·
[GDC Vault — Exploring the Tech and Design of 'Noita'](https://www.gdcvault.com/play/1025695/Exploring-the-Tech-and-Design) ·
[video](https://www.youtube.com/watch?v=prXuyMCgbTc) ·
[Game Developer coverage](https://www.gamedeveloper.com/design/video-understanding-the-remarkable-tech-and-design-of-i-noita-i-)

Other: [Bad Piggies — Wikipedia](https://en.wikipedia.org/wiki/Bad_Piggies) ·
[Worms (series) — Wikipedia](https://en.wikipedia.org/wiki/Worms_(series)) ·
[Niche Gamer — Nimbatus review](https://nichegamer.com/reviews/nimbatus-the-space-drone-constructor-review/) ·
[Delisted Games — Scraps](https://delistedgames.com/scraps-modular-vehicle-combat-likely-leaving-steam-on-february-20th/) ·
[SteamSpy — Scraps](https://steamspy.com/app/350150) ·
[Zarkonnen — Airships dev blog](https://www.zarkonnen.com/airships)
