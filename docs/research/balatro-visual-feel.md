# Balatro's visual feel — what actually produces it

Research findings for [Star Rouge issue #5](https://github.com/WMsans/star-rouge/issues/5) (child of the
[architecture map, #1](https://github.com/WMsans/star-rouge/issues/1)).

**Status:** facts only. This document does **not** choose an art spec for Star Rouge — the pixel-art spec
ticket does that. Every claim is sourced. Statements that are my reading rather than a source's claim are
marked **[inference]**.

## Source trust levels used here

| Tier | What | Used for |
| --- | --- | --- |
| A | Balatro's own shipped GLSL and Lua (LÖVE `.love` archives are plain files; extracted copies are mirrored publicly) | all rendering/animation claims |
| A | The game's own shipped assets (PNG atlases, fonts), inspected byte-level | resolution and filtering claims |
| A | LocalThunk's own words (his blog, direct interview answers) | intent, tooling, process |
| B | Godot 4 official docs | what transfers to our engine |
| C | Community ports (Godot Shaders, Mix and Jam) | existence proof that a technique is portable, not as evidence about Balatro |

Primary shader/Lua source paths below are as they appear inside the shipped game
(`resources/shaders/*.fs`, `engine/*.lua`, `game.lua`, `globals.lua`, `functions/*.lua`). The public mirror
read for this research is [`skornuta/balatro-web-modded`, `extracted/`](https://github.com/skornuta/balatro-web-modded/tree/main/extracted)
(a modded web build that ships the game's unpacked resource tree); `engine/moveable.lua` cross-checks
against a second independent mirror, [`GladdonT/balatro-source-code`](https://github.com/GladdonT/balatro-source-code).

---

## 1. The headline question: is Balatro pixel art?

**The assets are pixel art. The rendering is not.** Both halves are load-bearing.

### The assets are genuinely pixel art, authored at 1x

LocalThunk, asked directly about his art process:

> "The art for the game, in general, had some restrictions to help everything appear more visually cohesive:
> rules about colour palette, art resolution, UI standardization, strict card sets, etc. One example of a
> restriction like this is that, apart from legendary Jokers, every Joker card has the word 'Joker' written in
> the art somewhere. More specifically, **I drew it all with my mouse and Aseprite (pixel art software)**."
> — [PlayDay.One interview, 2024-03-09](https://playday.one/2024/03/09/there-is-a-lot-more-design-to-explore-within-balatro/)

And on his own blog, on Dec 2020 – Jan 2021:

> "I remember spending a long time making custom pixel art for the red deck back and all the playing cards.
> It was the first time I had tried making proper pixel art."
> — [The Balatro Timeline, localthunk.com](https://localthunk.com/blog/balatro-timeline-3aarh)

Verified against the shipped atlases (`game.lua`, `Game:set_render_settings`):

- Card atlas cell size is declared `px=71, py=95` — `{name = "cards_1", path = "resources/textures/"..texture_scaling.."x/8BitDeck.png", px=71, py=95}`.
- `resources/textures/1x/8BitDeck.png` measures **923×380 px** = exactly 13×4 cells of 71×95. `Jokers.png` at 1x is 710×1520.
- Animated atlases are tiny: `blind_chips` is `px=34, py=34` over 21 frames; `shop_sign` is `px=113, py=57` over 4 frames.

So: real hand-pixelled sprites, one 71×95 card, all drawn with a mouse in Aseprite.

### The ship also carries a pre-doubled "2x" texture set — and it is a nearest-neighbour upscale

`resources/textures/` contains parallel `1x/` and `2x/` trees. I byte-compared them: **every 2×2 pixel block in
`2x/8BitDeck.png` and `2x/Jokers.png` is perfectly uniform** (350,740 and 1,079,200 blocks tested, 0 non-uniform).
That is a mathematically exact integer nearest-neighbour doubling of the 1x art, not separately drawn hi-res art.

Which set loads is a graphics setting, and the default is 2x:

```lua
-- game.lua, Game:set_render_settings()
self.SETTINGS.GRAPHICS.texture_scaling = self.SETTINGS.GRAPHICS.texture_scaling or 2

--Set fiter to linear interpolation and nearest, best for pixel art
love.graphics.setDefaultFilter(
    self.SETTINGS.GRAPHICS.texture_scaling == 1 and 'nearest' or 'linear',
    self.SETTINGS.GRAPHICS.texture_scaling == 1 and 'nearest' or 'linear', 1)
```

`globals.lua` confirms the shipped default: `GRAPHICS = { texture_scaling = 2, shadows = 'On', crt = 70, bloom = 1 }`.

**So by default Balatro renders pixel art through a bilinear filter, from a 2x-pre-doubled atlas.** The named
technique: *pre-scale pixel art by an integer factor, then let bilinear absorb the remaining non-integer scale.*
It keeps edges reasonably crisp while allowing free rotation, sub-pixel translation and continuous scaling.
`texture_scaling = 1` is the "true pixel art" path (1x atlas, nearest filter) and it is not the default.

### Nothing else in the pipeline preserves a pixel grid either

- **No fixed internal resolution, no integer scaling.** `globals.lua` sets `TILESIZE = 20`, `TILESCALE = 3.65`;
  `prep_draw()` in `functions/misc_functions.lua` does `love.graphics.scale(G.TILESCALE*G.TILESIZE)` — 73 units
  per tile, a non-integer scale applied to every drawable. The canvas is created at window size
  (`love.graphics.newCanvas(w*G.CANV_SCALE, h*G.CANV_SCALE, ...)` in `main.lua`) and set to
  `G.CANVAS:setFilter('linear','linear')`. `G.CANV_SCALE` is 1.5 on high-DPI Windows.
- **Cards are continuously rotated and sub-pixel-positioned** by the spring system (§3), which by construction
  cannot land on pixel boundaries.
- **The background and the score flame are procedural shaders that invent their own screen-space pixel grid**,
  unrelated to any asset. `background.fs` and `splash.fs` both do
  `pixel_size = length(love_ScreenSize.xy)/PIXEL_SIZE_FAC` with `#define PIXEL_SIZE_FAC 700.` and then
  `floor(screen_coords.xy*(1./pixel_size))*pixel_size`. `flame.fs` uses `PIXEL_SIZE_FAC 60.` in UV space.
  The "chunky pixels" of the background are a *quantisation step inside a fragment shader*, not sprite art.
- **A full-screen CRT pass resamples the entire frame** (§2), with barrel distortion and chromatic aberration —
  both of which destroy pixel-grid alignment by definition.
- **The UI font is a TTF, not a bitmap font.** `resources/fonts/m6x11plus.ttf` (a pixel-styled TrueType face),
  loaded at `render_scale = self.TILESIZE*10` (=200) and drawn at `FONTSCALE = 0.1` — i.e. rasterised large and
  scaled down, with CJK fallbacks in Noto Sans / Go Noto.

**[inference]** The honest description of Balatro's look is: *low-resolution hand-pixelled sprites treated as
ordinary textured quads, animated with continuous spring motion, then run through a heavy procedural
post-processing stack.* Calling it "pixel art" describes the asset authoring, not the renderer. A brief that
says "pixel art like Balatro" and then specifies nearest-neighbour filtering, integer camera scaling and a
fixed low internal resolution would be chasing something Balatro does not do — Balatro's default settings
break all three.

---

## 2. The full-frame post-processing stack

There are 19 shaders in `resources/shaders/`, loaded generically at boot:

```lua
-- game.lua
self.SHADERS = {}
local shader_files = love.filesystem.getDirectoryItems("resources/shaders")
for k, filename in ipairs(shader_files) do
    if string.sub(filename, -3) == '.fs' then
        self.SHADERS[string.sub(filename, 1, -4)] = love.graphics.newShader("resources/shaders/"..filename)
    end
end
```

`CRT.fs`, `background.fs`, `booster.fs`, `debuff.fs`, `dissolve.fs`, `flame.fs`, `flash.fs`, `foil.fs`,
`gold_seal.fs`, `holo.fs`, `hologram.fs`, `negative.fs`, `negative_shine.fs`, `played.fs`, `polychrome.fs`,
`skew.fs`, `splash.fs`, `vortex.fs`, `voucher.fs`.

### `CRT.fs` — one pass, seven effects, on by default

Applied to the whole canvas every frame (`game.lua` draw, ~line 2955). Default `crt = 70`, and the draw code
scales it by 0.3 before sending, so the effective strength is ~21/100. In one pass:

1. **Barrel distortion.** `tc += (tc.yx*tc.yx) * tc * (distortion_fac - 1.0)`, with
   `distortion_fac = {1.0 + 0.07*crt/100, 1.0 + 0.1*crt/100}` — asymmetric, more vertical bulge than horizontal.
2. **Edge vignette / feather to black.** `mask = (1 - smoothstep(1-feather_fac, 1, abs(tc.x)-BUFF)) * (…y…)`, `feather_fac = 0.01`.
3. **Slight inward scale** so the bulge doesn't sample outside: `scale_fac = 1.0 - 0.008*crt/100`.
4. **Horizontal chromatic aberration** — R and G sampled at opposite ±0.0005 offsets, resolution-compensated
   by `*1600./love_ScreenSize.x` so the aberration is a constant *apparent* width at any window size.
5. **Scanlines / phosphor mask**, three separate phase-offset sine masks for R, G, B, plus a 4×-frequency
   horizontal mask — an RGB triad simulation, not a plain darkening. Sent as
   `scanlines = G.CANVAS:getPixelHeight()*0.75/G.CANV_SCALE`, i.e. **scanline count follows the resolution**.
   The comment in the shader is explicit that this is an overlay, not the image: *"a repeated 'pixel' mask that
   doesn't actually render the real image. If these pixels were used to render the image it would be too harsh."*
6. **Film grain** from a cheap hash of quantised `tc` × `time`; `noise_fac = 0.001*crt/100`.
7. **Bloom**, a `(2*BLOOM_AMT+1)²` = 49-tap box gather (`#define BLOOM_AMT 3`) with a hard
   `cutoff = 0.6` highlight threshold and a triangular weight, mixed in at `0.03*(crt_intensity/(0.16*0.3))`.
   Followed by explicit contrast/brightness recentring (`-= 0.55 … *= 1.14 … += 0.5`).

There is also a **`glitch_intensity`** path — four summed sines per side producing horizontal line
displacement, plus R/G channel-swap on scattered scanlines — but it is currently sent as `0` (the live value is
commented out in `game.lua`). **[inference]** dormant/cut feature.

### `background.fs` — the animated menu/table background

Procedural, no texture input. Named steps, in order: **screen-space pixel quantisation** (`PIXEL_SIZE_FAC 700`) →
**polar swirl** about a point offset `-vec2(0.12, 0.)` from centre, angle `+= speed - SPIN_EASE*20.*(spin_amount*uv_len + (1-spin_amount))`
with `#define SPIN_EASE 0.5` → **5-iteration domain-warp "paint" loop** (`uv += 0.5*vec2(cos(...uv2.y...), sin(...uv2.x...))` etc.)
→ **three-colour ramp** blended by `paint_res` into `colour_1/colour_2/colour_3` with a `contrast` knob.

Crucially the colours are *uniforms driven from game state*, not baked. `functions/common_events.lua` has
`ease_background_colour{new_colour = …, special_colour = …, tertiary_colour = …, contrast = …}` and
`ease_background_colour_blind(state, blind_override)`, called on every state change — shop, booster pack,
each blind, boss blind (`lighten(mix_colours(boss_col, G.C.BLACK, 0.3), 0.1)` with `contrast = 2`), game over
(`G.C.BLACK, contrast = 3`). `game.lua` sends `contrast` and `spin_amount` as live-eased uniforms
(`{name = 'contrast', ref_table = G.C.BACKGROUND, ref_value = 'contrast'}`, `{name = 'spin_amount', ref_table = G.ARGS.spin, ref_value = 'amount'}`).

**Named technique: the background is one procedural shader whose palette and swirl rate are tweened by
gameplay events.** That is where a lot of "the game reacts to me" comes from, with zero art cost.

`splash.fs` is the same swirl+domain-warp family with `vort_speed`, `vort_offset` and `mid_flash` — the title
splash. `flame.fs` is the same 5-iteration warp loop restricted to a card-local UV with an upward scroll vector
`vec2(0., mod(4.*time,10000.) - 5000. + mod(1.781*id,1000.))` and a per-object `id` seed so two flames never sync.

### The score flame was suggested, not planned

From LocalThunk's timeline blog, March 2023: a friend suggested a *"flaming effect on your score"* for strong
hands; he initially resisted, then agreed — *"That flaming score effect really fit perfectly with the game I
was trying to create."* ([localthunk.com](https://localthunk.com/blog/balatro-timeline-3aarh))

---

## 3. The per-card shader vocabulary

Every card shader (`dissolve`, `foil`, `holo`, `polychrome`, `negative`, `negative_shine`, `booster`, `voucher`,
`debuff`, `played`, `hologram`) shares the same skeleton and the same uniform contract, sent from
`engine/sprite.lua` via `G.SHADERS[_shader]:send(name, val)`: `texture_details` (atlas cell x, y, w, h),
`image_details` (atlas px size), `dissolve`, `time` (derived from object id, so cards de-sync),
`burn_colour_1` / `burn_colour_2`, `shadow`, `hovering`, `mouse_screen_pos`, `screen_scale`.

### The fake-3D hover tilt is a `w`-component hack — this is the single most distinctive trick

Identical vertex function in `dissolve.fs`, `foil.fs`, `holo.fs`, `polychrome.fs`, `negative*.fs`, `skew.fs`, `CRT.fs`:

```glsl
vec4 position( mat4 transform_projection, vec4 vertex_position ) {
    if (hovering <= 0.) return transform_projection * vertex_position;
    float mid_dist    = length(vertex_position.xy - 0.5*love_ScreenSize.xy)/length(love_ScreenSize.xy);
    vec2  mouse_offset = (vertex_position.xy - mouse_screen_pos.xy)/screen_scale;
    float scale = 0.2*(-0.03 - 0.3*max(0., 0.3-mid_dist))
                * hovering*(length(mouse_offset)*length(mouse_offset))/(2. - mid_dist);
    return transform_projection * vertex_position + vec4(0,0,0,scale);
}
```

It perturbs only **`w`** — the homogeneous coordinate. The GPU's perspective divide then displaces each of the
quad's four corners by a different amount, producing a genuine projective warp (a *homography*), so the card
appears to lean in 3D. The lean magnitude is `length(mouse_offset)²`, so corners further from the cursor swing
harder, and it is damped by `mid_dist` (distance from screen centre) so cards near the middle of the screen
tilt more. `skew.fs` is *only* this function — the reusable "tilt anything" shader. `CRT.fs` runs the same
function on the whole canvas with `hovering` hard-sent as `1`, at 1/100th the strength (`0.002` vs `0.2`), so
**the entire screen leans toward the mouse, all the time**.

### Edition effects

| Shader | Named technique |
| --- | --- |
| `foil.fs` | Four superimposed sine/cosine interference fields (`fac`…`fac4`) over centred UV, one of them angular (`dot(rotater, adjusted_uv)/(…)`); combined `maxfac` pushes B up ×1.9 while pulling R and G down — a cold blue metallic sheen. Also modulates alpha. Animated by a `foil` **vec2** uniform (two independent phases), not raw `time`. |
| `holo.fs` | Full **RGB↔HSL round trip** (hand-written `HSL()`/`RGB()`/`hue()`), then `hsl.x += res + fac` (hue *rotation*), `hsl.y *= 1.3` (saturation boost), `hsl.z = hsl.z*0.6 + 0.4` (lightness compression). `res` comes from a three-term moving cosine **interference field**; `fac` is a **diagonal lattice mask** (`gridsize = 0.79`, three `7*cos(...)-6` ridges at 20/45 frequencies) — the rainbow-grid look. Result tinted `vec4(0.9,0.8,1.2,·)`. |
| `polychrome.fs` | Same HSL round trip, but `hsl.x += res + polychrome.y*0.04` and `hsl.y = min(0.6, hsl.y+0.5)` — a hard saturation *floor*, so every pixel becomes vivid regardless of source. Slow-scrolling hue band. |
| `negative.fs` / `negative_shine.fs` | Channel inversion + a separate additive shine pass. |
| `dissolve.fs` | The universal appear/disappear. A three-term moving cosine field is thresholded against `adjusted_dissolve = (d*d*(3-2*d))*1.02 - 0.01` (smoothstep, deliberately over-ranged to −0.01…1.01 "so the mask does not pause at extreme values"), with edge bias terms that make borders dissolve first. The threshold band is painted with `burn_colour_1` then `burn_colour_2` — a **two-colour burning edge**. Note `floored_uv = floor(uv*texture_details.ba)/max(...)` — the mask is quantised to the *sprite's* pixel grid, so the burn reads as chunky even under linear filtering. |
| `vortex.fs` | Vertex-only polar twist of the whole screen: `angle = atan(uv.y,uv.x) + effectAngle*smoothstep(effectRadius, 0., len)`, driven by `vortex_amt`. The scene-transition suck. |
| `gold_seal.fs`, `flash.fs` | Small additive/flash passes (780 and 879 bytes). |

Every card also draws a **shadow pass**: `engine/sprite.lua` re-draws with `shadow = true`, and each shader
returns `vec4(vec3(0.), tex.a*0.3)` in that case — i.e. the *shader itself* emits the silhouette at 30% alpha,
offset by `shadow_parrallax` (`engine/moveable.lua`), which is derived from the object's horizontal position in
the room. So shadows fan outward from the screen centre like a single overhead light. `shadows = 'On'` by default.

---

## 4. The animation and juice vocabulary (this is code, not art)

### Every moveable has a target transform and a visible transform, joined by a damped spring

`engine/moveable.lua`. `T` is the desired `{x,y,w,h,r,scale}`; `VT` is the drawn one; nothing snaps.

```lua
velocity = G.exp_times.xy * velocity + (1 - G.exp_times.xy) * (T - VT) * 35 * dt   -- position
des_r    = self.T.r + 0.015*vel.x/dt + (self.juice and self.juice.r*2 or 0)        -- rotation
```

- Velocity is clamped to `G.exp_times.max_vel`; motion stops when both `abs(VT.x - T.x) < 0.01` and `abs(velocity.x) < 0.01`.
- **`0.015*vel.x/dt` is the lean.** Rotation is *derived from horizontal velocity*, so cards bank into their
  own motion automatically — no authored animation, every card that moves anywhere tilts correctly.
- Separate exponential coefficients per channel: `G.exp_times.xy`, `.scale`, `.r`.
- `pinch.x/.y` animates width/height toward 0 linearly at `8*dt` — the "card closes up" wipe.
- Hover adds `+0.05` scale, drag adds `+0.1`, multiplicatively in `move_scale()`.

### `juice_up` — one function, ~68 call sites

```lua
-- engine/moveable.lua, Moveable:juice_up(amount, rot_amt)
juice.scale = scale_amt * math.sin(50.8*(REAL_TIME - start)) * max(0, decay)^3
juice.r     = r_amt     * math.sin(40.8*(REAL_TIME - start)) * max(0, decay)^2
```

A **damped sinusoid** on scale (≈8.1 Hz, cubic decay envelope) and on rotation (≈6.5 Hz, quadratic decay),
running about 0.4 s. Two different frequencies and two different decay powers, so scale and rotation fall out of
phase — that is what stops it reading as a mechanical bounce. `Card:juice_up` adds randomisation:

```lua
-- card.lua
local rot_amt = rot_amount and 0.4*(math.random()>0.5 and 1 or -1)*rot_amount
                or (math.random()>0.5 and 1 or -1)*0.16
scale = scale and scale*0.4 or 0.11
```

**The rotation direction is coin-flipped every call**, so repeated pops never look identical.

Call-site counts in the shipped source: `card.lua` 49, `common_events.lua` 16, `misc_functions.lua` 2,
`game.lua` 1. Representative amplitudes: `self:juice_up(0.05, 0.03)` on hover (tiny), `(0.3, 0.3)` on a card
flip, `(0.3, 0.5)` on consumable use, `(0.8, 0.8)` on a Joker triggering, `(1, 0.5)` at the loudest.
Wrappers: `juice_card(card)` (fires `juice_up(0.7)` as a queued event) and `juice_card_until(card, eval_func, first, delay)`,
which **re-arms itself every 0.8 s** with `juice_up(0.1, 0.1)` while a predicate holds — that is how an active
Joker keeps breathing.

### The screen is never still: idle sway + cursor parallax + impulse shake

`functions/common_events.lua`, `update_canvas_juice(dt)`. Three superimposed things, all on `G.ROOM`'s transform:

```lua
shake_amt = (reduced_motion and 0 or 1)*G.SETTINGS.screenshake/100*3
G.ROOM.jiggle = (G.ROOM.jiggle or 0)*(1-5*dt)*(shake_amt > 0.05 and 1 or 0)
G.ROOM.T.r = (0.001*sin(0.3*REAL) + 0.002*jiggle*sin(39.913*REAL)) * shake_amt
G.ROOM.T.x = ORIG.x + shake_amt*(0.015*sin(0.913*REAL) + 0.01*jiggle*shake_amt*sin(19.913*REAL)
                                 + (eased_cursor_pos.x - 0.5*(ROOM.T.w + ORIG.x))*0.01)
G.ROOM.T.y = ORIG.y + shake_amt*(0.015*sin(0.952*REAL) + 0.01*jiggle*shake_amt*sin(21.913*REAL)
                                 + (eased_cursor_pos.y - 0.5*(ROOM.T.h + ORIG.y))*0.01)
```

1. **Permanent idle drift** — slow sines at ≈0.91/0.95 rad/s on x/y and 0.3 rad/s on rotation. Always on.
   Deliberately *incommensurate* frequencies on the two axes so the drift never loops visibly.
2. **Cursor parallax** — the whole room translates toward the mouse by `0.01 *` the offset from centre, using a
   pre-eased cursor position (`eased_cursor_pos.x = x*(1-3*dt) + 3*dt*target`, a first-order lag). Combined with
   the `CRT.fs` `w`-tilt at `hovering=1`, the entire scene both slides and leans toward the pointer.
3. **Impulse shake** — `jiggle` is set on events and decays at `*(1-5*dt)`, riding fast sines (39.9, 19.9, 21.9 rad/s).

Default `screenshake = true`, migrated to a **0–100 slider** (`G.SETTINGS.screenshake = reduced_motion and 0 or 50`).
Note `shake_amt` for the cursor-parallax term is `max(0, screenshake-30)/100` — the parallax only engages above 30.

### Sound is pitched per event, and pitch encodes progression

`play_sound(sound_code, per, vol)` in `functions/misc_functions.lua` — `per` is a **pitch multiplier**, passed
straight to `s.sound:setPitch(s.original_pitch*args.pitch_mod)`. The recurring pattern is a for-loop that walks
pitch across a sequence:

```lua
for i=1, #G.hand.highlighted do
    local percent = 1.15 - (i-0.999)/(#G.hand.highlighted-0.998)*0.3
    G.E_MANAGER:add_event(Event({trigger='after', delay=0.15, func=function()
        G.hand.highlighted[i]:flip(); play_sound('card1', percent)
        G.hand.highlighted[i]:juice_up(0.3, 0.3); return true end}))
end
```

Pitch descends 1.15 → 0.85 across the cards, and each card's flip, its `juice_up` and its sound fire in the
**same event**, 0.15 s apart. That is the sound-synced motion: not audio reacting to animation, but one queued
event emitting both. `Card:hover()` randomises: `play_sound('paper1', math.random()*0.2 + 0.9, 0.35)`.
There is also a global `G.PITCH_MOD`, lerped `(1-dt)*old + dt*target`, dropped to `0.5` on game over —
**the whole soundtrack and all SFX slow-pitch down when you lose.**

### `card_eval_status_text` — the floating number popups

`functions/common_events.lua`. A single table-driven function mapping each event type to `{text, sound, colour, scale, type, delay}`:

| eval_type | sound | colour | notes |
| --- | --- | --- | --- |
| `chips` | `chips1` | `G.C.CHIPS` | delay 0.6 |
| `mult` / `h_mult` | `multhit1` | `G.C.MULT` | `type='fade'`, scale 0.7 |
| `x_mult` / `h_x_mult` | `multhit2` | `G.C.XMULT` | volume 0.7 |
| `dollars` | `coin3` | `G.C.MONEY` / `G.C.RED` if negative | |
| `debuff` | `cancel` | `G.C.RED` | scale 0.6 |
| `swap` | `generic1` | `G.C.PURPLE` | |
| `extra`/`jokers` | `foil2` if edition, else `multhit1`/`multhit2`/`generic1` | `G.C.DARK_EDITION` if edition | delay 0.75 |

Default pitch is `percent = percent or (0.9 + 0.2*math.random())` — **randomised ±10% on every popup**.
Anchoring is context-dependent: `bm` (bottom-middle) for Jokers, `tm` (top-middle) with negative y-offset for
cards in hand or in play, so numbers never cover the card that produced them.

### `DynaText` — text as an animated object

`engine/text.lua`. Text is not drawn as a string; each letter is a transform. Named modes, all config flags:

- **`pop_in`** — letters appear sequentially, `letter.pop_in = clamp((REAL - pop_in - created)*#string*pop_in_rate - k + 1)`
  then **squared** (`letter.pop_in = letter.pop_in*letter.pop_in`) for an ease-in curve. `pop_in_rate = 3` default.
  Each letter's arrival fires `play_sound('paper1', 0.45 + 0.05*random() + (0.3/#string)*k + (pitch_shift or 0))` —
  **a rising typewriter arpeggio across the word.**
- **`bump`** — per-letter y offset, `bump_amount*sqrt(scale)*7*max(0, (5+bump_rate)*sin(bump_rate*REAL + 200*k) - 3 - bump_rate)`,
  `bump_rate = 2.666`. The `200*k` phase offset per letter index is what makes it a travelling wave, and the
  `max(0, …)` clamp makes each letter *hop* rather than oscillate.
- **`float`** — continuous sine bob, `sin(2.666*REAL + 200*k)`.
- **`quiver`** — four superimposed high-frequency sines/cosines (41.1, 63.2, 36.1, 95.1, 30.1 rad/s) on rotation
  plus a flat `+0.1*amount` scale. `set_quiver()` defaults `{speed = 0.5, amount = 0.7}`.
- **`pulse`** — a *travelling scale wave* down the string: scale peaks at letter index `k` as
  `(REAL - start)*speed` sweeps past. `pulse()` defaults `{speed = 40, width = 2.5, amount = 0.2}`. Rotation is
  coupled to scale and mirrored about the string centre (`0.02*(-#letters/2 - 0.5 + k)`) so the word fans as it pulses.
- **`rotate`** — arcs the whole string, `0.2*(-#letters/2 - 0.5 + k)/#letters`, plus a slow `0.02*sin(2*REAL + k)` wobble.
- **`shadow`** — drop shadow per letter.
- **`scale`** — `score_number_scale(scale, amt)` in `misc_functions.lua` shrinks big numbers so they still fit:
  `14*0.75/(floor(log(amt))+4)*scale` above 1,000,000.

Every one of these is gated on `(G.SETTINGS.reduced_motion and 0 or 1)`.

### Numbers are never assigned, they are eased

`functions/common_events.lua` exposes `ease_chips`, `ease_dollars`, `ease_discard`, `ease_hands_played`,
`ease_ante`, `ease_round`, and the generic `ease_value(ref_table, ref_value, mod, floored, timer_type, not_blockable, delay, ease_type)`.
`ease_chips(mod)` queues an `Event{trigger='ease', ref_table=G.GAME, ref_value='chips', ease_to=mod, delay=0.3, func=math.floor}`,
**then** juices the HUD element and plays `chips2`. So the counter rolls over 0.3 s, integer-floored, while the
widget pops. Same for `ease_colour(old_colour, new_colour, delay)` — colours are tweened in place too.

### `Particles`

`engine/particles.lua` — a `Moveable` subclass with `timer` (spawn interval, default 0.5), `lifespan`, `speed`,
`vel_variation`, `max`, `pulse_max` (clamped to 20 — the burst count), `scale`, `colours` (a *list*, sampled per
particle), `fill` (spawn anywhere in the rect vs at a point), and `attach` (bonds to a parent Moveable with
`bond = 'Strong'`, so emitters follow cards). `initialize = true` runs 60 warm-up steps at 15/60 s so the effect
is already mid-flight on its first frame — no visible ramp-up.

### Everything runs through one event queue

`G.E_MANAGER:add_event(Event({trigger = 'after'|'immediate'|'ease', delay = …, blocking = …, blockable = …, timer = 'REAL'|'TOTAL', func = …}))`
is the single scheduling primitive; `delay(time, queue)` inserts gaps. Every juice, sound, ease and state change
is a queued event with an explicit delay, which is why the scoring sequence reads as *choreography* rather than
simultaneous chaos. **[inference]** This is the structural reason Balatro's feel is reproducible: the timing is
data in one queue, not scattered timers.

---

## 5. Palette and saturation

Balatro does **not** use a small restricted pixel-art palette. `globals.lua`'s `G.C` table is a *semantic* palette.
Measured HSL of the shipped hex values:

| Role | Hex | H | S | L |
| --- | --- | --- | --- | --- |
| `MULT` / `RED` / `XMULT` | `FE5F55` | 4° | **99%** | 66% |
| `CHIPS` / `BLUE` | `009DFF` | 203° | **100%** | 50% |
| `FILTER` / `IMPORTANT` | `FF9A00` | 36° | **100%** | 50% |
| `ORANGE` | `FDA200` | 38° | **100%** | 50% |
| `MONEY` | `F3B958` | 38° | 87% | 65% |
| `GOLD` | `EAC058` | 43° | 78% | 63% |
| `GREEN` / `CHANCE` | `4BC292` | 156° | 49% | 53% |
| `PURPLE` | `8867A5` | 272° | 26% | 53% |
| `BLACK` (the ground) | `374244` | 189° | **11%** | 24% |
| `L_BLACK` | `4F6367` | 190° | 13% | 36% |
| `GREY` | `5F7377` | 190° | 11% | 42% |
| UI `BACKGROUND_LIGHT` | `B8D8D8` | 180° | 29% | 78% |
| UI `BACKGROUND_DARK` | `7A9E9F` | 182° | 16% | 55% |
| Suit Hearts | `F03464` | 345° | 86% | 57% |
| Suit Diamonds | `F06B3F` | 15° | 86% | 59% |
| Suit Spades | `403995` | 245° | 45% | 40% |
| Suit Clubs | `235955` | 176° | 44% | 24% |

The structure: **a near-neutral desaturated blue-grey ground (S 11–16%) with a handful of 87–100% saturated
accents reserved for numbers that matter.** Every "black" and every UI chrome colour is a *cool* hue (176–190°);
nothing is a true grey. Saturation is a semantic channel — chips are 100%-saturated blue, mult is 99%-saturated
red, and the game's chrome deliberately stays out of their way.

Colours are manipulated programmatically, not hand-picked per case: `lighten(colour, percent)`,
`darken(colour, percent)`, `mix_colours(C1, C2, proportionC1)`, `adjust_alpha(colour, new_alpha)` in
`misc_functions.lua`. E.g. the boss-blind background is `lighten(mix_colours(boss_col, G.C.BLACK, 0.3), 0.1)`.

**Where the "highly saturated" impression actually comes from** — three separate amplifiers, none of them in the art:

1. `polychrome.fs` clamps saturation *upward*: `hsl.y = min(0.6, hsl.y + 0.5)`.
2. `holo.fs` multiplies it: `hsl.y = hsl.y*1.3`, and compresses lightness to 0.4–1.0.
3. `CRT.fs` applies contrast expansion and threshold bloom over the whole frame.

**[inference]** The reference's "highly saturated and fresh" quality is a *post-process and palette-discipline*
result, not a consequence of the sprites being vivid. The sprite palette is mostly muted; the saturation is
added at the top of the pipeline and reserved for meaning.

---

## 6. Art versus code

Countable facts:

| | Count / size |
| --- | --- |
| Shader files | 19 (`resources/shaders/*.fs`) |
| `CRT.fs` alone | 7,117 bytes, 7 distinct effects, always on |
| `juice_up` call sites | ~68 |
| Named `ease_*` functions | 11 |
| `DynaText` motion modes | 7 (`pop_in`, `bump`, `float`, `quiver`, `pulse`, `rotate`, `shadow`) |
| Card atlas cell | 71×95 px, drawn with a mouse in Aseprite |
| Animated sprite atlases in the whole game | 2 (`blind_chips`, 21 frames; `shop_sign`, 4 frames) |

**Two hand-animated sprite sequences in the entire game.** Everything else that moves — the tilt, the lean, the
pops, the flames, the background, the burns, the text, the shake — is code and shaders operating on static
sprites. **[inference]** The "very animated" quality of the reference costs almost no animation labour; it is
bought with a spring system, one damped-sinusoid function, a procedural background, and a post stack.

Note also that the whole motion layer is *switchable*: `G.SETTINGS.reduced_motion` zeroes juice, quiver, bump,
float, pulse, shake and tilt; `screenshake` is a 0–100 slider; `crt` 0–100; `bloom`; `shadows`; `texture_scaling`;
plus `colourblind_option`. **[inference]** The feel was built as a layer over a readable base, not fused into it —
which is why it can be turned off without breaking the game.

---

## 7. What transfers to Godot 4.7, and what does not

### Transfers directly (Lua → GDScript, same maths)

| Technique | Godot equivalent |
| --- | --- |
| Target/visible transform with exponential velocity | Plain `_process` code on `Node2D.position/rotation/scale`. No engine feature needed; the formulas above are engine-agnostic. |
| Rotation derived from velocity (`0.015*vel.x/dt`) | Same, one line. |
| `juice_up` damped sinusoid | Same, or `Tween` with `TRANS_ELASTIC`/`TRANS_BACK`/`TRANS_SPRING` + `EASE_OUT` ([Tween docs](https://docs.godotengine.org/en/stable/classes/class_tween.html)). The hand-rolled version is more controllable (independent scale/rot frequency and decay power); Godot's `Tween` cannot easily express two out-of-phase channels in one call. |
| Idle sway + cursor parallax + decaying impulse shake | `Camera2D.offset`/`rotation`, same three superimposed terms. |
| Number easing (`ease_chips` etc.) | `create_tween().tween_method(...)` with a rounding setter, or the same custom queue. |
| Event queue with `delay`/`blocking` | No built-in equivalent to `G.E_MANAGER`; **build it**. `Tween.chain()`/`tween_interval()` covers simple chains; the blocking/blockable distinction and re-arming (`juice_card_until`) needs custom code. |
| Per-letter text animation | `RichTextEffect` (custom BBCode effects) or a custom `Node2D` that draws glyphs individually — Godot's `Label` will not animate per-character. `RichTextEffect` gives per-character `offset`, `color`, `visible`; it does **not** give per-character rotation or scale, so `quiver`/`pulse`/`rotate` need a custom draw. |
| Pitch-per-event sound | `AudioStreamPlayer.pitch_scale`, and `AudioStreamRandomizer` for the `0.9 + 0.2*random()` pattern. |
| Palette-as-uniform procedural background | `ColorRect` + `canvas_item` shader; `background.fs` translates almost line-for-line (`love_ScreenSize` → `1.0/SCREEN_PIXEL_SIZE`, `screen_coords` → `FRAGCOORD.xy`). |
| Dissolve with two-colour burn edge | Direct port; `texture_details`/`image_details` become `TEXTURE_PIXEL_SIZE` and region uniforms, or are unnecessary with individual textures instead of an atlas. |
| Particle config (`pulse_max`, `vel_variation`, warm-up) | `GPUParticles2D` covers most of it; the `initialize`-style warm-up is `preprocess`. |
| Pre-doubled pixel art + linear filter | Project setting `rendering/textures/canvas_textures/default_texture_filter`, or per-`CanvasItem` `texture_filter`. Ship 2x art, set Linear. |

### Does **not** transfer as written

1. **The `w`-perturbation tilt.** This is the crown jewel and it is engine-specific. In Godot's `canvas_item`
   vertex shader, `VERTEX` is a **`vec2`** in local space and there is no writable clip-space `w`
   ([CanvasItem shader reference](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)).
   You cannot add `vec4(0,0,0,scale)`. Workarounds, in increasing fidelity:
   - **Fragment-space perspective divide.** Build a 3×3 rotation, lift UV into 3D, divide by z in the fragment
     stage: `p = inv_rot_mat * vec3(UV - 0.5, 0.5/t); uv = p.xy/p.z - o`. This is what the public
     [Balatro card tilt](https://godotshaders.com/shader/balatro-card-tilt/) Godot shader does. Correct-looking,
     but the sprite's *silhouette* stays a rectangle — only the contents warp. Fine for cards (the art fills the
     quad); wrong for anything with a transparent margin.
   - **Four-point perspective / homography on UV** ([4-point perspective transformation](https://godotshaders.com/shader/4-point-perspective-transformation/)).
   - **Actual 3D:** a quad in a `SubViewport` with a `Camera3D`, composited into 2D. Real silhouette warp,
     real cost, and it fights a pure-2D project.
   **[inference]** For Star Rouge — a physics arena, not a card table — the tilt is probably the least relevant
   Balatro technique anyway. Flagging the cost so nobody budgets it as "just port the shader".

2. **`CRT.fs` as one pass.** Godot's `Environment.glow` is not a fragment shader you can fold in: it is
   **only supported in Forward+ and Mobile**, not Compatibility, and for 2D it needs **HDR 2D** enabled plus
   `glow_hdr_threshold` **decreased below 1.0**, because "2D rendering is performed in SDR"
   ([Environment docs](https://docs.godotengine.org/en/stable/classes/class_environment.html)). So the Balatro
   stack splits into: a full-screen `ColorRect` shader with `hint_screen_texture` for barrel distortion,
   chromatic aberration, scanlines and grain; **plus** a `WorldEnvironment` for glow. That is two mechanisms
   where LÖVE had one, and it constrains the renderer choice (rules out Compatibility if you want engine glow).
   The 49-tap in-shader bloom *can* be ported verbatim into the `ColorRect` shader if you'd rather not depend on
   `Environment` — at 49 texture fetches per pixel per frame.
   **[inference]** Also worth flagging against our "60fps on modest hardware" constraint: a 49-tap gather plus a
   falling-sand simulation are competing for the same budget, in a way they never were in Balatro.

3. **`resources/shaders` auto-discovery.** Balatro globs a directory at boot. Godot wants
   `ShaderMaterial`/`.gdshader` resources; the equivalent is a preloaded dictionary or a resource folder scan.
   Cosmetic difference, mentioned only because Balatro's shader *naming convention* (one `.fs` per visual state,
   discovered by filename) is a nice pattern that Godot can keep.

4. **The uniform-send-per-draw pattern.** `engine/sprite.lua` sends `texture_details`, `time`, `hovering`,
   `mouse_screen_pos` per sprite per frame. In Godot that means either a `ShaderMaterial` per instance (breaks
   batching) or `INSTANCE_CUSTOM` / per-instance shader uniforms. Godot 4 supports instance uniforms
   (`instance uniform`), which the card-tilt shader above uses for `x_rot`/`y_rot` — that is the batching-safe route.

5. **`love_ScreenSize`, `Texel`, `extern`, the `number` type.** Trivial syntactic renames
   (`1.0/SCREEN_PIXEL_SIZE`, `texture()`, `uniform`, `float`) but every ported shader needs them.

### Godot-side notes with no Balatro counterpart

- `CanvasItem.texture_filter` is per-node, so Star Rouge can mix nearest-filtered sand with linear-filtered
  props if that's ever wanted. Balatro had one global setting.
- `CanvasLayer` + `follow_viewport` gives layered parallax for free; Balatro hand-rolls `layered_parallax` and
  `shadow_parrallax` in `moveable.lua`.

---

## 8. Named-technique index

For the pixel-art spec ticket to choose among. Ordered roughly by how much feel-per-effort each buys.

**Motion**
1. Target/visible transform pair with per-channel exponential velocity + threshold stop (`moveable.lua`).
2. Rotation derived from horizontal velocity — automatic banking.
3. `juice_up`: damped sinusoid, independent frequency and decay power on scale (50.8, ³) vs rotation (40.8, ²), coin-flipped direction.
4. Self-re-arming juice (`juice_card_until`) for persistent "this thing is active" breathing.
5. Hover/drag scale bumps (+0.05 / +0.1) as state feedback.
6. `pinch` — animate w/h to zero for a close-up wipe.
7. Permanent low-frequency idle sway on incommensurate axis frequencies.
8. Cursor parallax on the whole scene, through a first-order lag.
9. Decaying-impulse screen shake layered on top of (7).
10. One central event queue with explicit per-step delays — choreography as data.

**Text**
11. `pop_in` with squared ease and a per-letter rising-pitch sound.
12. `bump` — clamped travelling hop, phase-offset by letter index.
13. `pulse` — travelling scale wave with coupled fan rotation.
14. `quiver` — four superimposed high-frequency sines.
15. `float`, `rotate`, per-letter `shadow`.
16. Auto-shrink for large numbers (`score_number_scale`).
17. Table-driven floating popups: one map from event type → text + sound + colour + scale + delay + anchor.

**Shaders**
18. `w`-perturbation projective tilt (LÖVE-only; see §7.1 for Godot routes).
19. Whole-screen version of the same at 1/100 strength, always on.
20. Dissolve mask from a moving three-term cosine field, over-ranged smoothstep, edge-biased, quantised to the sprite pixel grid, with a two-colour burn band.
21. Shader-emitted shadow pass (`shadow` bool → black at 30% alpha) with position-derived offset.
22. Procedural background: screen-space pixel quantisation → polar swirl → 5-iteration domain warp → 3-colour ramp, with palette/contrast/spin as gameplay-eased uniforms.
23. Per-object `id`/`time` seeds so identical effects never sync.
24. HSL round-trip edition effects: hue rotation + saturation floor (`polychrome`) or multiplier (`holo`) + lightness compression.
25. Interference-field foil: several superimposed sines, one angular, driving a cold channel-biased sheen.
26. Vertex-only polar twist for scene transitions (`vortex`).
27. CRT stack: barrel distortion, edge feather, resolution-compensated chromatic aberration, RGB-triad scanlines scaled to resolution, hash grain, threshold box bloom, contrast recentring.

**Colour**
28. Semantic palette: 11–16%-saturation cool neutral ground, 87–100%-saturation accents reserved for meaningful numbers.
29. Programmatic colour derivation (`lighten`/`darken`/`mix_colours`/`adjust_alpha`) instead of per-case hex picking.
30. Saturation added at the top of the pipeline (shader), not baked into sprites.

**Assets**
31. Pixel art authored at 1x, shipped pre-doubled by exact nearest-neighbour, displayed with a linear filter.
32. Only two hand-animated sprite sequences in the entire game.

**Audio-motion coupling**
33. Sound, sprite juice and state change emitted from the *same* queued event.
34. Pitch as a sequence index (1.15 → 0.85 across a run of cards).
35. Randomised ±10% pitch on every repeated cue.
36. Global pitch modifier lerped by game state (0.5 on game over).

**Accessibility as a first-class layer**
37. `reduced_motion` gates every motion term; `screenshake`, `crt`, `bloom`, `shadows`, `texture_scaling` are all user sliders.

---

## 9. Gaps and open questions

- **No developer commentary on the technical juice work exists that I could find.** LocalThunk has published on
  design, tooling and process, but there is no shader breakdown, postmortem or GDC talk from him. Balatro took
  Game of the Year, Best Debut, Best Design and Innovation at [GDCA 2025](https://gdconf.com/article/-balatro-plays-winning-hand-at-gdca-2025-receiving-game-of-the-year/)
  without an accompanying technical session. Everything in §2–§4 is read from the shipped source, not from
  developer statements. Treat the *intent* attributions as **[inference]**; the mechanisms are verified fact.
- The `glitch_intensity` path in `CRT.fs` is fully implemented but sent as `0`. Unclear whether cut or reserved.
- I did not verify `hologram.fs`, `booster.fs`, `voucher.fs`, `debuff.fs`, `played.fs`, `negative*.fs`,
  `flash.fs`, `gold_seal.fs` line-by-line — only their skeletons and sizes. If the spec ticket wants one of
  those specific looks, read the file.
- `attention_text` (the big centred announcement popups) is referenced in the modding community but was not in
  the file subset I read; its config is unverified here.
- The `G.exp_times` values are initialised to `{xy = 0, scale = 0, r = 0}` in `globals.lua` and set elsewhere at
  runtime; I did not trace the live values, so the *absolute* spring stiffness is unverified. The *structure* is verified.

## Sources

Primary — shipped game source and assets (read via public mirror):
- [`skornuta/balatro-web-modded`, `extracted/`](https://github.com/skornuta/balatro-web-modded/tree/main/extracted) — `resources/shaders/*.fs`, `resources/textures/{1x,2x}/`, `resources/fonts/`, `engine/*.lua`, `functions/*.lua`, `game.lua`, `globals.lua`, `main.lua`
- [`GladdonT/balatro-source-code`](https://github.com/GladdonT/balatro-source-code) — independent mirror, used to cross-check `engine/moveable.lua` and `engine/sprite.lua`

Primary — developer's own words:
- [LocalThunk, "The Balatro Timeline"](https://localthunk.com/blog/balatro-timeline-3aarh)
- [LocalThunk, "I'm Slow"](https://localthunk.com/blog/im-slow)
- [LocalThunk interview, PlayDay.One, 2024-03-09](https://playday.one/2024/03/09/there-is-a-lot-more-design-to-explore-within-balatro/)
- [LocalThunk interview, Rogueliker](https://rogueliker.com/balatro-interview/)

Godot 4 official documentation:
- [CanvasItem shader reference](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)
- [Environment class reference (glow)](https://docs.godotengine.org/en/stable/classes/class_environment.html)
- [Tween class reference](https://docs.godotengine.org/en/stable/classes/class_tween.html)

Community ports, cited only as portability evidence:
- [Balatro card tilt — Godot Shaders](https://godotshaders.com/shader/balatro-card-tilt/)
- [4-point perspective transformation — Godot Shaders](https://godotshaders.com/shader/4-point-perspective-transformation/)
- [Balatro background shader — Godot Shaders](https://godotshaders.com/shader/balatro-background-shader/)
- [Mix and Jam, "Balatro's Card Movements & Shaders Recreated in Unity"](https://80.lv/articles/balatro-s-card-movements-shaders-recreated-in-unity) / [`mixandjam/balatro-feel`](https://github.com/mixandjam/balatro-feel)
- [Steamodded `SMODS.Edition` wiki](https://github.com/Steamodded/smods/wiki/SMODS.Edition) — confirms `dissolve`/`booster`/`voucher` as the base shader set

Awards context:
- [GDCA 2025 results, gdconf.com](https://gdconf.com/article/-balatro-plays-winning-hand-at-gdca-2025-receiving-game-of-the-year/)
