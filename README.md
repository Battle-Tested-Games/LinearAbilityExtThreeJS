# Elemental Sandbox

A skillshot VFX sandbox built with **Three.js**, **Vite** and hand-written **GLSL**.

Five abilities and two ways to aim them. Four are **line casts**: press the key to arm, a
League-of-Legends style arrow appears on the ground and swings with the mouse, click to fire. The
fifth is a **far cast**: the arrow is replaced by a circle with a deliberately thick boundary that
follows the cursor and answers the only question a ground-targeted AoE has to answer before you
commit — how much space is this going to take.

**Q — Frost Lance.** A fracture front races out along the line while a field of ice crystals
tears up out of the floor behind it — small and dense at your feet, opening into a wall of blades
at the far end, with a cluster thrown up around the impact point.

**E — Storm Lance.** A bolt leaves the caster's hand and a bundle of lightning filaments is drawn
out behind the strike front, holds while it gutters and re-strikes, then blows out. Sparks come
off it the whole way, the floor underneath takes a branching electric burn and a dark scorch, and
the far end gets a shell of ionised air.

**R — Cinder Fall.** A burning rock is lobbed downrange on an arc, trailing a raymarched wake of
burning gas and heating up the whole way: the lava seams splitting its surface prise wider and
brighter as it comes in. It detonates on arrival, throws its own shattered chunks across the floor, and tears the
ground open into a network of molten cracks that keep glowing while the crater burns out.

**F — Nova Beam.** The caster winds a ball of light up in both hands, pulling motes in out of the
air, then lets a column of it out along the line — white-hot core, cyan sheath, gold ribbons
spiralling around it and shock discs racing down it. It *holds* there, burning into the floor and
throwing spray back up the beam, before collapsing to a thread and blinking out. The only cast in
the sandbox that is still happening a second after it landed.

**V — Voltaic Snare.** The far cast. A leash of current is whipped out across the floor, and where
it lands the ring snaps open past its own radius and pulls back onto it: a violet column tears up
out of the middle, tendrils crawl outward to the boundary, arcs run around the rim and the whole
disc burns. It holds there re-striking and hauling the air up into the pillar, then collapses to a
thread. The circle you measured out before the click is exactly the circle you get.

Everything you can see is generated. There are no textures, no sprite sheets and no meshes on
disk except the character: the crystals are procedural geometry, the bolt is a strip of ribbon
placed entirely by a vertex shader, the meteor is an icosphere cratered and sliced by fracture
planes on the CPU, the beam is a parametric tube drawn three times at three radii, the snare's
whole cage is that same ribbon strip threaded along four different parametric paths, the arrow, the
targeting circle, the rime, the burns and the molten cracks are signed-distance and noise shaders,
and the mist, sparks, chips and glitter are GPU particles.

**Every parameter is a live slider** — 938 of them — and they stay live while the simulation is
paused. That is the point of the project: freeze a frame mid-eruption, mid-strike or mid-burn with
**P**, then reshape the silhouette, the palette and the timing against a still image.

References for the look: `icecast.jpg`, `thundercast.jpg`, `superbeam.jpg` and
`electricalboost.jpg`.

---

## Quick start

```bash
npm install
```

```bash
npm run dev
```

Then open the URL Vite prints (default <http://127.0.0.1:5173>).

```bash
npm run build
```

```bash
npm run preview
```

### Assets

Six binary assets are served from `public/` and loaded automatically at boot:

| File | Purpose |
| --- | --- |
| `public/models/Idle.fbx` | Rigged character **and** its idle animation clip |
| `public/models/diffuse.png` | The character's colour map |
| `public/models/cast1.fbx` | Cast animation |
| `public/models/cast2.fbx` | Cast animation |
| `public/models/cast3.fbx` | Cast animation — the default for Frost Lance, Root Snare and Glacier Crown |
| `public/hdri/spruit_sunrise.hdr` | HDR probe used for image-based lighting and crystal reflections |

All four FBX files are Mixamo exports of the same rig, each carrying a skinned mesh plus one
animation stack. The character comes from the idle file; the cast files are loaded for their clip
alone, and the duplicate rig that arrives with each one is released the moment its `AnimationClip`
has been taken. Clips bind to the skeleton by bone name, which is the whole reason an animation
authored in another file plays here without retargeting.

The rig ships no material, so `diffuse.png` is loaded beside it and assigned as the colour map when
the imported materials are converted to PBR — an FBX that *does* carry an embedded texture keeps its
own, since that map is authored against its own UVs.

Every ability picks the clip it throws — `castAnim` in its settings block, a dropdown under **The
cast** in its editor folder. Out of the box slots 1, 5 and 6 — Frost Lance, Root Snare and Glacier
Crown — throw `cast3`, and the other three throw `cast1`. The clip is a one-shot laid over
the looping idle, with `character.castBlendIn` / `castBlendOut` as the two edges of that overlap.

The HDR is loaded as image-based lighting and as the reflection source for the ice — it is never
shown as a visible sky. The stage keeps its flat dark backdrop.

---

## Controls

| Input | Action |
| --- | --- |
| **Q** (or **1**) | Arm Pyre Crown — a far cast, aimed with a circle |
| **E** (or **2**) | Arm Kraken Crown — a far cast, aimed with a circle |
| **R** (or **3**) | Arm Electrical Sphere — a far cast, aimed with a circle |
| **F** (or **4**) | Arm Earthen Spire — a line cast, aimed with the arrow |
| **V** (or **5**) | Arm Verdant Gate — a gate cast, aimed with a threshold and a standing arch |
| **X** (or **6**) | Arm Tidewrought Ring — a ring cast, aimed with a sigil the ring tips up out of |
| **Z** (or **7**) | Arm Fire Portal — a scribe cast, aimed with a circle a spark runs round |
| **B** | Electric Boost — a self buff; press again to let go |
| **M** | Magic Boost — a self buff; press again to let go |
| **K** | Fire Boost — a self buff; press again to put it out |
| **Move the mouse** | Swing the aim arrow, or move the far-cast circle |
| **Left click** | Cast along the arrow, or drop the circle where it is |
| **Esc** / **right click** | Cancel an armed cast |
| **Right mouse + drag** | Orbit the camera |
| **Scroll** | Zoom |
| **G** | Show/hide the VFX editor |
| **P** | Pause / resume — *the editor keeps applying* |
| **C** | Clear all active effects |
| **H** | Hide the controls panel |

`range` and `minRange` are per ability, so the indicator's reach changes with the slot you have
selected. Aiming closer than the selected ability's `minRange` tints it red and refuses the cast;
set `minRange` to 0 if you would rather cast at your own feet, which is what the Snare ships with —
a trap you cannot drop on yourself is missing half its uses. Cooldowns are per ability too, so
spending one slot never locks the other out.

The three boosts are not slots: nothing selects them and nothing aims them, so they sit outside the
arm-and-cast loop entirely and are simply switched on. Any of them, all of them or none can be
running at once — where two want to light the character's own materials, the stronger claim wins
(see `src/materials/FresnelAura.js`). Fire Boost differs from the other two in where it hangs its
effect: its flames are rooted on the rig's **actual bones**, so the fire on a forearm swings with
the arm, and its orbiting embers trail wakes that are the orbit *sampled backward in time* rather
than a recorded path — which is why dragging their tilt re-sweeps a second of fire instantly, even
paused.

---

## Project layout

```
src/
  abilities/      Ability base class (the travelling front), IceAbility, ThunderAbility,
                  MeteorAbility, BeamAbility, SnareAbility, pooling manager
  animation/      FBX character loading, AnimationMixer, the per-ability cast clips,
                  the procedural cast lunge
  assets/         Procedural crystal and asteroid geometry, the bolt ribbon strip,
                  the beam tube and its shock discs
  config/         settings.js — the single source of truth for every parameter
  core/           App, Renderer, CameraRig, Time, Layers, shared frame uniforms
  effects/        Aim arrow, far-cast circle, ground decals, fissures, bursts,
                  light pool, shake, flash
  input/          InputManager (events) and AimController (both targeting shapes)
  loaders/        AssetLoader with a shared LoadingManager
  materials/      IceMaterial, LightningMaterial, MeteorMaterial,
                  VolumetricFireMaterial, BeamMaterial, SnareMaterial
  particles/      GPU particle system + engine and rate emitters
  postprocessing/ Composer pipeline, grade shader, distortion shader
  shaders/lib/    Shared GLSL: noise library, common helpers
  ui/             HUD, lil-gui editor, preset manager, styles
  utils/          Maths, colour cache, pooling, disposal, shader patching
  world/          Environment (stage lighting), floor, dust, contact shadows
  archive/        The retired four-element sandbox — see archive/README.md
```

---

## How it fits together

### Settings are the API

`src/config/settings.js` holds every tweakable value. Nothing else owns that state: shaders,
particle systems, lights and post passes *read* those objects every frame. That is what makes the
editor work with no rebuild — moving a slider changes the ice field that is already standing, the
next cast, the environment and the post stack at once. Preset loading deep-merges *into* the same
objects so every live binding stays valid.

```js
import { settings } from './config/settings.js';
settings.ice.height = 7;          // visible on the next frame, even mid-cast
settings.thunder.jitter = 1.2;    // re-kinks a bolt that is already in the air
settings.global.timeScale = 0.1;  // slow the whole cast to a crawl
```

Ability blocks are keyed by their id in `ELEMENTS`, and the shared systems that need to know
"which ability is the player holding" — the aim controller, the cooldowns, the HUD — look it up as
`settings[element]`. The four fields they rely on being present are `range`, `minRange`, `speed`
and `cooldown`; a far cast adds a fifth, `zoneRadius`. Everything else in a block is that ability's
own business.

### The rule that makes "edit while paused" work

A spike record in `IceAbility` stores **only what the dice decided**: a position *fraction* along
the line, a signed lateral *fraction*, and a handful of unitless jitters. Not one metre, radian or
second is captured when the cast starts. Every dimension is resolved against `settings.ice` inside
the update loop, which runs on a zero-length frame too.

So dragging `height` re-grows a field that is already standing; dragging `lean` re-tilts it;
dragging `clumping` re-packs it toward the centre line. The only values a record *does* capture
are timestamps — the moment its own eruption was triggered. Those are events, not dimensions.

The four *shape* controls (`facets`, `taper`, `roughness`, `bend`) cannot be expressed as a
per-instance transform, so they are baked into the geometry instead — and a six-sided crystal is
just 60 triangles, cheap enough to regenerate outright rather than approximate in a vertex shader.
`IceAbility#_syncGeometry` hashes those four values and rebuilds the three crystal meshes when the
hash changes, which is what keeps them live sliders rather than restart-required constants.

### Aiming

`AimController` raycasts the pointer onto the ground plane **every frame**, not only on mouse
move, so orbiting the camera with a cast armed swings the indicator under a stationary cursor. It
clamps the distance into `[minRange, range]`, tracks a 0..1 reveal envelope, and emits a single
`cast` event carrying an origin, a unit direction and a distance — which is exactly the signature
`Ability#spawn` takes. It decides nothing about what the cast does.

It runs on **real** time rather than the scaled simulation delta, so the indicator keeps animating
while the sandbox is paused.

There are three indicators and one controller. Which one is drawn comes from
`ELEMENT_META[element].cast` — `CastShape.LINE`, `CastShape.ZONE` or `CastShape.GATE` — and that is
the *only* thing the three shapes disagree about. Arming, clamping, validating, revealing and
firing are shared, and all three end in the same three-argument `cast` event, because from the
targeting side a far cast is a line cast you only care about the far end of, and a gate cast is one
you also care about the heading of. That is why neither zone nor gate targeting needed a change in
`Ability` or `App`: both read their site as `pointAt(1)` and work outward from there.

### The far-cast circle

`ZoneIndicator` is the arrow's opposite number, and it is built out of the same two ideas: metres,
and no textures.

The **footprint** is one quad whose fragment shader remaps UV into metres from the target, so the
boundary stays 0.34 m thick whether the circle is 2 m or 8 m across. The band is deliberately the
heaviest mark on screen — it is the whole message — and it is split about the nominal radius by
`boundaryBias` rather than centred on it, so its *outer* lip stays honest about where the effect
ends. Inside there is a rim-weighted wash, contour rings travelling outward, warped filaments and a
reticle whose downrange arm is longer, because the quad carries the caster's yaw and that arm is
therefore the heading.

The **reach ring** at the caster is the bolt's ribbon strip bent into a circle: `(t, side)` in,
world position out. A quad big enough to hold a 20 m range would be 40 m across and shade a
screenful of discarded fragments for one thin line.

The circle **snaps out past its radius and settles back** when the cast is armed, and the trap does
the same thing when it lands. A circle that grows linearly reads as a UI element; one that
overshoots reads as something the caster did.

### The arrow is one SDF

`AimIndicator` is a single ground quad. Its fragment shader remaps UV into **metres measured from
the caster**, so every control in `settings.aim` is a real measurement — the shaft stays 0.42 m
wide whether the cast is 3 m or 15 m long.

The silhouette is a rounded union of a box (the shaft) and iq's exact triangle SDF (the head);
the cheap half-plane intersection leaves visible corner artefacts on a wedge this shallow. From
that one distance field the shader derives the outline, the rim-weighted interior wash, the
chevrons (a phase skewed by `|x|`, which turns flat bands into arrowheads pointing the way the
cast does), the frost noise and voronoi plates, the ring at the caster's feet, the range cap arc,
a six-fold frost rosette pinned to the impact point, and the sweep-out when the ability is armed.

### The gate template

The third targeting shape, and the first one that leaves the floor. An arrow answers *which way*
and a circle answers *how much ground*; neither answers the question a **structure** raises, which
is what will be standing there and which way it will face.

`GateIndicator` draws three things, all in metres and none of them a texture: a **threshold** on the
floor — a slot the width of the opening with a heavy pad under each jamb, so the footprint the
stones will take is honest — the shared **reach ring**, and the **ghost**: the arch itself, standing
upright in the gate's own plane, drawn from the same `archDistance` SDF the portal surface uses. It
draws itself floor-upward as the cast is armed, which is the order the gate is built in, and it is
the only way a targeting indicator can show a *facing*: stand the silhouette up and let the camera
see it edge-on when you are about to build it edge-on.

The ghost reads `gateWidth` and `gateHeight` off the ability rather than off its own settings block,
so the preview and the gate that gets built can never disagree.

### The ring template

The fourth targeting shape, and the first one that previews a **sequence** rather than a shape.

The gate template can stand its silhouette up and leave it standing because a doorway is built
where it stands. A ring is not: it is forged flat on the floor and then raised, so a template that
only drew the standing pose would be promising the wrong half of the cast. `RingIndicator` draws
the **sigil** — the disc of floor the segments will be laid on, carrying the ring's own lobed
contour, one tick per segment and one spoke per course — and the **ghost**, which is drawn lying on
that sigil and **tips upright as the cast arms**, on the same overshooting settle the cast itself
uses. Arming is a rehearsal of what the click will do.

The contour is not a circle. `ringDistance` modulates the radius with a few shallow lobes, because
a true circle is the one shape that reads as *drawn* rather than as *made*; the same SDF is shared
with the rift surface, the way `archDistance` is shared with the portal's.

Like the gate's ghost, it reads `ringRadius` and `ringHover` off the ability rather than off its own
settings block, so the preview and the ring that gets built can never disagree.

### The gate

`PortalAbility` is the first cast that **builds** something and leaves it there. Everything else in
the sandbox is an event — it happens, it fades, the pool takes it back. A gate is a place.

Three beats. A seam of green races along the aimed line to the site (the base class's travelling
front, doing its usual job). The arch is then **constructed**: quarried blocks break out of the
floor outside the footprint and swing up into their slots, both jambs climbing together, the outer
courses lagging the inner ones, the keystone seating last with the only shake worth feeling. Then
the portal floods the opening and **stays lit** — until another gate is raised, at which point the
standing one is asked to come apart, keystone first.

Two pieces of that are worth calling out:

**The stones hold no metres.** Each one stores where it sits along the contour as a signed 0..1 —
which jamb, and how far up toward the keystone — plus its course and its dice. Every position, angle
and size is resolved against `settings.portal` each frame, so dragging the span of a gate that has
been standing for a minute re-lays the whole arch around the new opening, keystone included, while
the clock is paused. It is the same rule the rest of the project runs on, and a standing structure
is where it pays the most: this is the one cast you can walk around and study.

**The opening is never geometry.** The surface is one quad carrying the arch's SDF in its fragment
shader, which is why the aperture can flood open and the span can be dragged without anything being
rebuilt. The vortex inside it is a logarithmic spiral — winding the radius through a `log` is what
turns concentric rings into a funnel you can read depth in — over a colour floor that keeps the
whole opening lit, because a surface with dark patches in it reads as a hole in the wall rather
than as a gate. The halo that spills onto the stones is the same quad drawn a second time
additively, and the seams in the blocks themselves are lit by the portal's own colour, so the arch
goes dark the moment the gate shuts.

The blocks are `createBlockGeometry`, not the boulder the spire heaves up: a chamfered box with
flat faces and chipped edges, knocked off independently at all eight corners. An arch is something
somebody stacked, and a round rock reads as the opposite of built. Two seeds are instanced side by
side, because a single silhouette repeated forty times is what gives a procedural wall away.

### The ring

`AetherRingAbility` is the gate's argument answered by an engineer instead of a mason, which makes
the pair the project's clearest statement about **animation** the way the two crowns are its
statement about material. Both build something and leave it standing; everything that separates
them is when and where the pieces move.

Four beats. A tide of light runs out along the aimed line. The ring is then **forged lying down** —
segments swing in out of a wide orbit *in the ground plane*, spiralling inward against the ring's
own rotation and locking from the foot upward, both arcs closing on the crown, with a band of runes
lighting behind them one mark at a time. The finished hoop **stands up**, hinging off the floor
about its own lateral axis and settling a few degrees past vertical. Then the horizon **irises
open** from the middle out, slams into the rim, and stays lit.

Three pieces of that are worth calling out:

**The tip-up is four lines and no state.** `_updateFrame` hinges the ring's up-axis toward the
heading, lifts the centre by the same progress, and turns the in-plane axes by whatever the ring is
spinning at — all of it read out of `settings.aether` and three ages. The spin is analytic rather
than integrated (a fixed number of turns eased off over the forging, plus a slow idle), so nothing
about the pose is accumulated. Pause halfway up and drag `stand up over` and the ring re-poses on
the spot; drag `clear radius` on one that has been standing for a minute and it re-forges itself
around the new circle, crown included.

**The segments arrive in polar.** A piece that lerps between two points slides across the middle of
the ring; a piece that lerps between two *(angle, radius)* pairs orbits into place. The angle and
the radius are eased on different curves on purpose — the segment swings most of the way round
while it is still far out, and only then falls inward.

**The rift is the inverse of the gate.** The portal surface is brightest where it meets the stone
and hazy in the middle, because a doorway is lit by its frame. A rift is a hole: `rim` puts the
light against the segments and `eye` takes the middle away, and the pool thins over the eye so the
scene shows through it. Turn the eye down to zero and what is left is a glowing plate — that one
slider is most of the difference between a portal and a hole. The same quad drawn additively
carries the halo *and* the rune band, which is why the forging is legible while the ring is still
lying on the floor with nothing in it.

It ends by coming off the spindle rather than falling down: the crown lets go first and the pieces
are flung outward and tangentially while the horizon implodes ahead of them.

### The fire portal

The third standing cast, and deliberately the smallest ability in the project. The gate stacks
stones and the ring swings segments into a hoop; this one has no pieces at all. It is two things:

- **the way through** — a black disc that irises open from the middle out. It is drawn with normal
  blending and writes depth, so it genuinely removes the room rather than glowing over it, and
  sparks drifting on the far side do not show through the hole. Every other portal in the sandbox
  puts light in its opening; this is the only one that takes the opening away.
- **the ring** — which is not really a shape, it is an *emitter*. A circle standing in the air that
  throws stretched sparks off itself on a tangent, all the way round, every frame.

Nothing in `src/abilities/FirePortalAbility.js` draws a curve. The sparks leave on a straight
tangent and the particle system's **drag** is what bends them into the long lines — low drag gives a
starburst, high drag scrolls them tight around the rim. `sparkLife` is then how far the fan reaches,
because the colour is the system's own lifetime gradient and nothing else: white where a spark is
born, orange through the middle of its life, red as it goes out. There is no noise, no shear and no
second surface anywhere in the ability; the whole look is the ring's line, the disc behind it, and
how those four colours are spread across one lifetime.

It is not switched on, it is **struck**. A spark is lit at the foot of the circle and runs all the
way round it in `scribeTime`, and three things hang off that one clock: the fragment shader refuses
to draw contour the spark has not reached yet, the stroke immediately behind it burns `scribeTrailHeat`
times hotter than the settled ring and cools over `scribeTrail` metres, and the emitter stops
dressing the whole circle — during the draw every spark is born within `scribeTail` of the running
head and carries a share of its travel, so the shower is a moving source rather than a ring
dissolving into view. Only once the spark is nearly home (`apertureDelay`, a fraction of the draw)
is the way through allowed to start irising open inside what it drew.

The mask is the fiddly part, and both its ends are feathered on purpose. A plain angular cut steps
at the head *and* falls off a cliff at the seam where the angle wraps, which slices the ring's bloom
down a radial line and hangs a straight edge in the air below the circle; feathering both ends — by
an amount that widens with distance off the contour, because the mask is angular and the bloom is
not — lets the glow bleed a little back round the start, which is what the beginning of a stroke
looks like.

The one setting that can ruin it is `ringInner` — how far the ring's bloom licks back over the hole.
Past a couple of centimetres the middle lights, and the hole is the ability.

### Persistent casts

`Ability#isPersistent` is the whole of "the gate stays open". A persistent cast is never the one
retired to make room when `MAX_CONCURRENT` is reached — a gate four fireballs can delete is not a
gate that stays open — and only one of its element may stand at a time, so casting it again calls
`dismiss()` on the standing one and lets it play its collapse. The rule is per element, not global,
which is why a gate and a ring can stand at the same time and a second ring only ever dismisses the
first ring. Both rules live in `AbilityManager`
rather than in the ability, because they are questions about the *set* of live casts.

It also gives the camera back: `wantsCamera` is true while the gate is being built and false once
it is standing, or a gate raised a minute ago would pin the camera forever and make every later
cast unwatchable.

### The ice

`materials/IceMaterial.js` patches a `MeshStandardMaterial` rather than replacing it, so the
crystals cast and receive the stage's real shadows and pick up the HDR probe. The stylisation is
injected on top:

- **Thickness tint** — a facet seen head-on has the longest path through the crystal, so it
  darkens toward `colorDeep`; grazing edges stay pale. This is the term that makes the field read
  as a solid you can see *into* rather than as blue plastic.
- **Internal fracture** — ridged noise sampled in **world** space, so the crack planes stay a fixed
  physical size whether a spike is ankle-high or three metres tall, and neighbouring crystals look
  quarried from the same block.
- **Feather frost and rime** — fbm sampled in **local** space (0..1 up the crystal), so the milky
  veining and the frost creeping up from the base follow each spike's own axis however it is
  scaled or leaned.
- **Glint** — a hard-thresholded high-frequency field scrolling in world space, biased toward
  grazing angles, which is where real ice catches.
- **Birth flash** — a per-instance attribute the ability drives from 1 to 0 over `birthFade`, so a
  crystal is lit from within for the moment it erupts.

Three `InstancedMesh`es share one material. Three rather than one because the *facets* differ, not
just the proportions — per-instance scaling alone cannot buy that silhouette variety, and three
draw calls is a cheap price.

### The lightning

`ThunderAbility` takes the "no dimensions on the CPU" rule further than the ice does: there is no
path object at all. The bolt is one `InstancedBufferGeometry` — a flat ladder of quads in
*parameter* space, where each vertex carries only `(t, side)`: how far along the bolt it is, and
which edge of the ribbon it is on. One instance is one filament. `materials/LightningMaterial.js`
turns that pair into a world position every frame, so a single strip serves a bolt of any length,
any shape and any width.

Three things stack to make the shape:

- **the axis** — a straight line from the hand to the impact point, bowed by `sag`. The only part
  that knows where the cast is pointing.
- **the fan** — a constant per-filament offset in the plane perpendicular to the axis, opening
  from `spreadNear` at the hand to `spread` at the target and rolling around the axis with
  `twist`. This is what separates one filament from the next.
- **the kinks** — octaves of *linearly* interpolated value noise. Linear on purpose: smoothstep
  would round the corners off, and the corners are the entire reason it reads as lightning rather
  than as a wobbly tube.

The ribbon is turned to face the camera by crossing the local tangent with the view vector, which
is why the bolt keeps its apparent thickness from any angle without ever being a screen-space
line. It is drawn twice — a wide soft halo underneath and the hot core on top — because drawing
the glow as real ribbon rather than leaving it to bloom is what keeps it *attached* to every kink.

Two clocks run the flicker. `restrike` snaps every filament onto a new shape N times a second,
and `crawl` slides the kinks continuously in between; together they stop a held bolt from looking
like a static ribbon. A cast captures exactly one number — a seed, so two casts do not draw the
identical bolt — and resolves every metre, radian and second against `settings.thunder` each
frame. That is why dragging `jitter` re-kinks a bolt that is already in the air.

The ground burns are worth a note as a thing *not* to do. The first version sampled the filament
field on `atan(y, x)`, which hands every radius along a given bearing the same value and draws
dead-straight spokes out of the centre — a firework, not a burn. Sampling the same noise in the
plane and warping the lookup is what lets the filaments meander and fork.

### The beam

The Nova Beam shares the bolt's rule — no dimensions on the CPU — and reaches the opposite look
with it. Where the bolt's whole charm is that its noise is *piecewise-linear* and keeps its
corners, every noise term in the beam is smooth, stretched hard along the flow and crawling
downrange. A beam that kinks is a bolt.

It is a real tube rather than a camera-facing ribbon, because a column this thick has to *have* a
cross-section: the silhouette must bow correctly when you orbit it, the far wall must add through
the near one, and the shock discs have to hug it. `createBeamTubeGeometry` is the ribbon strip one
dimension richer — every vertex carries `(t, a)`, how far along the barrel it is and how far around
— and `materials/BeamMaterial.js` turns that pair into a world position each frame.

That one tube is drawn three times, and the trick is in how the three are weighted:

- **halo** — widest, nothing but a rim term. The atmosphere the beam is shoving out of the way.
- **sheath** — rim-weighted, so it reads as *hollow* and its silhouette edges are its brightest part.
- **core** — narrow, and weighted the **opposite** way: brightest where the view ray runs down the
  barrel and its path through the tube is longest.

Rim-weighted outside, axis-weighted inside, both faces adding: that is a volume integral, cheaply,
and the inversion is the entire reason the middle reads as a solid rod of light instead of as a lit
pipe. Widen `coreWidth` or push `coreFill` up and the three layers collapse into one white tube —
the cyan sheath and the gold coils are only legible because the core leaves them room.

Two more instanced passes put structure on it. The **coils** are the bolt's ribbon strip bent into a
helix, camera-facing and warm on purpose — the colour split is what stops them dissolving into the
sheath. The **shock discs** are an instanced annulus whose phase is `fract(index / count + time ×
speed)`, so the train is a pure function of the clock and there is no queue on the CPU. Both place
themselves against the same `beamRadius()` the tube uses, which is why all five stay welded together
when the profile is dragged.

The beam is also the one ability with a **fourth beat**. The other three run travel → impact →
fade; this one puts a wind-up in front of that, and it needed nothing from the base class:
`advance()` simply refuses to let the front leave the hand until the orb is up to power, so `IMPACT`
becomes the burn and the phase machine is untouched. The far end therefore has an impact that keeps
happening — spray thrown back up the line, pressure shells shed off the burning point, dust and
shockwave rings pushed across the floor, all rate-throttled through the same fractional-rate emitter
the particles use so every rate is a live slider.

### The snare

The Voltaic Snare is the first ability built around a *point* instead of a line, and the thing that
holds it together is that `zoneRadius` is read in exactly one place per consumer and nowhere is it
copied: the indicator measures it out, the tendrils end on it, the rim arcs run along it, the field
burns it and the column's throat and flare are fractions of it. Drag it and all five move together,
mid-cast, with the clock stopped.

The whole cage — the whip that plants it, the pillar, the tendrils and the rim arcs — is **one
instanced ribbon strip**, the same one the bolt and the beam's coils are drawn on. A filament's
*role* is decided in the vertex shader by testing its instance index against four live counts, and
the role picks which parametric path it is threaded along:

- **leash** — a sagging line from the hand to the travelling tip, dropped onto the floor.
- **column** — a twisting climb whose radius opens from `throat` to `columnSpread`.
- **tendril** — a meander running outward, its veer a per-filament constant rather than noise, so
  it curves the way a discharge that has committed to a direction does.
- **rim** — an arc travelling around the boundary, hopping over it at mid-span.

Every offset then lives in a frame taken by finite difference off that path, which is what lets one
kink function serve a vertical pillar and a filament crawling flat across the floor. The two
ground-hugging roles damp the vertical component of that offset and clamp above the floor — a kink
with a free `y` buries half of every tendril and the effect reads as a broken dotted line. Setting
a count to zero retires the role outright, which is how the leash disappears on the frame the ring
takes over. Two draw calls cover all four roles, however many filaments are in the air.

The **field** is a quad rather than a pooled decal for one reason: a decal captures its radius when
it spawns, and this circle has to re-scale under `zoneRadius` while it is standing. Its veins are
sampled in the plane and domain warped — the same lesson the bolt's ground burns taught, and for
the same reason.

The one thing worth stealing for the next far cast is the **snap**: the ring opens on
`Easing.outCubic` multiplied by a bump that peaks late and dies at exactly 1, so it overshoots its
radius and pulls back onto it, and the pillar climbs on the same clock 1.7× slower. The ground goes
first, then the air breaks down over it.

### The two crowns

The Glacial Crown and the Pyre Crown are the same ability twice, and they are in the project on
purpose: they are the clearest statement it makes about where an ability's identity actually lives.

Both fill a footprint the same way — a ring of shards seated at `zoneRadius`, a skirt banked against
its foot, and a **middle left empty**, because the read is a wall you are looking *into* and filling
the disc stops it being a ring. Both erupt as a sweep that starts on the bearing the front arrived
on and runs around both arms to close at the far side. Their settings blocks share slider names line
for line, which is what makes them comparable: you can tune one against the other.

Everything that tells them apart is material and timing.

- **The shard.** `GlacierMaterial` is near-empty glass carried by its edges — a chromatically split
  fresnel, light piped up the body, one real reflection of the stage off every facet.
  `PyreMaterial` is its inverse: opaque, lit by nothing but its own combustion, and almost entirely
  emissive. One domain-warped flame field, squashed along the blade's axis and scrolled so it
  climbs, run through one four-stop heat ramp — and then pushed through a contrast curve (`sharp`),
  which is the single control that turns a soft gradient into tongues with black voids between them.
- **The arrival.** Ice punches through the floor: it overshoots its height and springs back, and
  that damped bounce is most of what sells it as something *hard*. Fire does not do that, and a
  flame that bounces onto its height reads as rubber — so the Pyre Crown's eruption is strictly
  monotonic. There is no overshoot control in its settings block at all. `riseSnap` blends two
  curves that both land exactly on 1 and neither of which crosses it, and `creep` then approaches a
  little past full height from *below*, forever, which is what a flame does when it settles.
- **The air inside.** The Glacial Crown's signature system is snow *falling* through the ring — the
  only thing in the project that falls. The Pyre Crown's is the same idea reversed: an updraft, with
  embers released at the floor and carried up the column over the crater, orbiting its middle as
  they climb (`ParticleSystem`'s swirl mode, anchored on the centre).
- **The departure.** The ice shatters — a per-fragment chunk id, half voronoi and half a hash of the
  facet normal, taken away one plate at a time. The fire is **consumed**: the same combustion front
  that lit the blade from the ground up runs backwards from the point down, with an ember rim riding
  it and the body draining to ash behind it, sweeping back around the ring the way the bloom came.
- **The room.** The Pyre Crown is the only live ability that writes to `LAYER.DISTORTION`. A fire
  that does not bend the floor behind it reads as a decal on the lens, and no amount of extra flame
  geometry fixes that.

The crater is worth one more note. Its dark half is a `SCORCH` decal and its bright half is the
ability-owned quad — split precisely because burnt ground has to *subtract* from the floor and the
quad is additive. Trying to do both in one pass is how you end up with grey.

### The third crown, which moves

The Glacial and Pyre Crowns argue that an ability's identity lives in its material. The **Kraken
Crown** (`abilities/KrakenAbility.js`) is the counter-argument: it lives in the *motion*.

Both of those bloom once and then stand — nothing about either silhouette changes after the first
half second, which is why they are tuned on a paused frame. This one is never the same on two
consecutive frames. A slick of black water runs out to the aimed point, the flagstones give way, and
a ring of cephalopod arms hauls itself out of the rift, rears back, and then **hammers the middle of
the footprint** over and over — each landing throwing stone, spray and ink — until every arm rears
together for one synchronised slam and they drag themselves back into the hole.

- **The arm is bent in the vertex shader.** `createTentacleGeometry` bakes a tapered tube standing
  straight up +Y and it is never drawn in that shape. `KrakenMaterial` integrates a curvature
  profile — a lean, a curl and a travelling wave — up the arm, per vertex, per frame, and rebuilds
  the local frame at every ring from the same integral. Coiling out of a hole, rearing, whipping
  down and peeling back off the floor are that profile with four numbers moved. Nothing about the
  animation touches the CPU, and one `InstancedMesh` per silhouette draws the whole ring.
- **The arms actually hit the middle, and it is arithmetic rather than tuning.** A constant-curvature
  arc of length `L` turning through `Θ` puts its tip `L(1−cosΘ)/Θ` along the bend and `L·sinΘ/Θ`
  above the floor. At `Θ = π` that is `(2L/π, 0)` — on the ground. So an arm of length `πR/2` seated
  on a footprint of radius `R` strikes its exact centre, and the ability derives every arm's length
  from `zoneRadius` through that identity: drag the footprint slider *while they are hammering* and
  they keep hitting the middle. The same closed form hands the CPU the impact point with no readback,
  which is what every shockwave, crack, dust ring, chip of stone and sheet of water is placed with.
  The strike is also the only pose that puts its turn in the linear term, because it is the only one
  whose tip position has to be known.
- **The beat is the content.** Each arm runs its own rear → whip → press → peel cycle, scattered
  around the ring by `cycleScatter` so the slams arrive as rolling thunder, and the strike itself is
  eased with a quartic — the arm barely moves for the first half of it and covers most of the arc in
  the last few frames. Then `finaleLead` seconds before the end every arm abandons its own clock and
  lands on the same frame. Scattering them all cast long is what buys that moment its weight.
- **The skin is the only shaded material here that is not energy.** Everything else in the project is
  emissive and forgiven by the bloom pass. This is lit by the room's own sun, has a wet specular coat
  and one sample of the HDR probe, and carries three things nothing else does: chromatophore bands
  travelling the length of the arm (real cephalopods do exactly this, and it is what says *alive*),
  two staggered rows of suckers laid down the ventral face — which is why the baked `uv.x = 0` is
  anchored on the side the arm curls toward — and bioluminescence spent almost entirely on the sucker
  rims, because the inside of a curl is the side that faces you across the ring.
- **The air inside hangs.** The Glacial Crown's snow falls, the Pyre Crown's embers race up. This
  one's marine snow does neither: high drag and near-zero gravity, so the motes lose their launch
  speed at once and simply sit there, turning slowly about the throat. It is what says the space
  inside this ring is full of water.
- **The rift turns.** Fire spreads and ice creeps; water rotates. `AbyssFieldMaterial` shears its
  angular coordinate by the radius, which is the whole trick behind a spiral, and its curtain of
  spray is the only veil in the project that is **not additive** — spray is matter, and half of what
  makes the crown look deep is that the far arms are seen through a haze of it and the near ones are
  not.

Its rift splits the same way the Pyre Crown's crater does, and for the same reason: the dark half is
a `SCORCH` decal in deep navy under an additive quad.

### Adding another ability

1. Add a settings block in `config/settings.js` and an entry in `ELEMENTS` / `ELEMENT_META`.
2. Subclass `Ability` and implement `createShaders`, `createParticles`, `onTravel`, `onImpact`,
   `onFade`.
3. Register the class in `abilities/AbilityManager.js`.
4. Add an editor folder in `ui/Editor.js`, and a sigil in `ui/glyphs.js`.
5. Bind a key in `input/InputManager.js` — it emits `ability` with the 0-based slot index, which
   `App` maps through `ELEMENTS`.

To make it a **far cast** instead of a line cast, add two things and nothing else: `cast:
CastShape.ZONE` in its `ELEMENT_META` entry, and a `zoneRadius` in its settings block. The circle
indicator, the reach ring, the snap-out and the whole targeting loop come for free, and the ability
reads its centre as `pointAt(1)`.

To make it a **gate cast**, the same trade: `cast: CastShape.GATE`, plus `gateWidth` and
`gateHeight` in its settings block. The threshold, the standing arch ghost and the reach ring come
for free, and the ability reads its site as `pointAt(1)` and its facing as `direction`.

To make it a **ring cast** or a **scribe cast**, `cast: CastShape.RING` or `CastShape.SCRIBE`, plus
`ringRadius` and `ringHover`. Both hang a circle in the air and both measure it out of the same two
fields; what differs is what the template promises. The ring template draws the sigil the hoop is
forged on and tips the hoop up out of it, because a machine is not assembled in the pose it ends up
in. The scribe template never touches the floor at all — it just stands the circle in the air where
the thing will hang, and lets the reach ring carry the distance read.

To make it **stay** once it has been cast, override `isPersistent`, give `impactDuration` an
`Infinity` — which is exactly the statement "this cast does not end on its own" — and implement
`dismiss()` to start whatever it does to come apart. The manager handles the rest.

Everything else — pooling, the travelling front, the local frame, lights, phases, per-ability
cooldowns, the aim reach and camera framing — is inherited or driven off `ELEMENTS`. The HUD
builds its slots from that array, so a new ability appears in the bar on its own.

### Particles

`particles/ParticleSystem.js` is a GPU-simulated, instanced-quad system. Motion (velocity, gravity,
analytic drag, curl turbulence, vortex swirl), size-over-lifetime, the colour gradient and alpha
fade are all evaluated in the shader from per-instance attributes; the CPU only ever writes spawn
data, and only the slots that changed are uploaded. Particles live in a ring buffer, so spamming
the ability recycles slots instead of allocating. Silhouettes (soft, smoke, streak, leaf, chip,
ring) are procedural — there are no sprite textures anywhere in the project.

Frost Lance uses three systems: **mist** (non-additive, so the fog genuinely occludes and gives the
field depth), **shards** (lit chips under gravity) and **glitter** (additive, negative gravity — the
rising plume that is the signature of the reference frame).

Storm Lance uses four: **sparks** (velocity-stretched streaks under gravity), **motes** (the slow
ionised drift around the bolt), **smoke** (non-additive haze off the scorched floor) and **debris**
(lit chips). Its sparks are emitted from several points along the bolt each frame rather than one:
a beam sheds along its whole length, and a single origin makes every batch read as a starburst.

Nova Beam uses four as well, and works one of them twice: its **motes** are the intake spiralling
*into* the orb while it charges and the drift shed off the column once it is firing — the same glow,
thrown the other way. Its **sparks** are thrown radially off the barrel and then dragged downrange
by `sparkForward`, which is the read that says "pressure"; the bolt's fall instead, and that one
difference does a lot of the work of keeping the two abilities apart.

### Render pipeline

Per frame:

1. **Depth prepass** — the opaque world into a half-res packed-depth buffer. Every VFX shader
   samples it for soft intersections, so nothing cuts a hard line into the ground. The crystals sit
   on `LAYER.WORLD`, so mist and glitter fade softly against them.
2. **Distortion pass** — meshes on the distortion layer write screen-space UV offsets into a second
   half-res buffer. Nothing writes to it in the current build; the pass is kept because it is the
   hook a refraction effect would use.
3. **Composer** — scene → refraction warp → bloom → tone map (ACES) → grade.

The grade pass folds chromatic aberration, lift/gain/contrast/saturation/temperature, vignette,
film grain and the impact flash into one resample.

Shadows come from a single directional light whose orthographic shadow camera is re-centred on the
character each frame and fitted to a 52 m box at 4096² (~1.3 cm/texel). The `three/addons` CSM
module was tried first and removed: it replaces three's `lights_fragment_begin` chunk *globally*,
so any material not explicitly registered with it silently loses all directional lighting.

Contact shadows are a real render: the character's depth is captured from below into a 256²
target, blurred twice and projected onto the ground.

---

## Editor and presets

Press **G** for the panel. Folders: Presets, Global, Aim indicator, Far-cast circle, Frost Lance,
Storm Lance, Cinder Fall, Nova Beam, Voltaic Snare, Environment, Post processing, Camera,
Character. Every folder starts collapsed — there are enough controls here that one open section
pushes the rest off the screen.

- **Global** multipliers scale everything at once (speed, glow, noise, particles, lights, impact
  intensity, camera shake, time scale…).
- **Aim indicator** — the arrow's silhouette in metres, its outline and fill, the chevrons and
  frost, and the rings and rosette.
- **Far-cast circle** (40 controls) — the boundary band, the interior, the ticks, sweep and
  reticle, the reach ring, and the snap-out. Shared by every far cast, so it is filed with the
  targeting rather than with any one ability.
- **Frost Lance** (113 controls, 25 of them colours) — the cast, the footprint, the silhouette,
  the crystal itself, the eruption timing, the ice material, the frost on the ground,
  mist/chips/glitter, the impact and the dynamic light.
- **Storm Lance** (123 controls, 34 of them colours) — the cast, where the bolt leaves the hand,
  the bundle, one filament, the ribbon, flicker and restrike, the bolt's colour, the burns on the
  ground, sparks/motes/smoke/debris, the muzzle and impact, and the dynamic light.
- **Nova Beam** (176 controls) — the cast, where it leaves the hands, the column, the core/sheath/
  halo stack, the surface and its flow, the beam's colour, the coils, the shock discs, the charge
  and its intake, what the floor does, sparks/motes/steam/debris, release/impact/burn, and the two
  dynamic lights.
- **Voltaic Snare** (174 controls, 33 of them colours) — the cast and its footprint, the leash, the
  column, the tendrils, the rim arcs, the shared filament shape and flicker, the ribbon and its
  colour, the field on the floor, the burns, sparks/updraft/smoke/debris, throw/snap/hold, and the
  dynamic light.
- **Presets** save to `localStorage`, and can be duplicated, deleted, exported to JSON, imported
  from JSON, or reset to the shipped defaults.

Every ability exposes **every** colour it draws with, and none is derived from another: the crystal
palette, the bolt palette, the beam's four layers and its coils and discs, the ground marks, the
impact shells, the shockwave rings, the screen flashes, and a four-stop lifetime gradient
(`birth → early → late → death`) for each particle system. Tinting the fog without touching the ice,
or cooling the sparks to orange while the filaments stay blue, is a picker away.

Presets are plain snapshots of the settings tree, so an exported file is readable and editable by
hand.

Knobs worth knowing about, because they reshape their ability the most:

- `ice.heightCurve` — how late the ramp climbs; raise it and the field stays low until it explodes
  at the target. `ice.frontBias` below 1 crowds the crystals toward the impact point.
- `thunder.jitter` and `thunder.jitterScale` — how violently the bolt kinks, and how often.
  `thunder.strands` and `thunder.spread` set how wide the bundle reads, and `thunder.restrike`
  how hard it strobes. Those five carry the character of the effect.
- `beam.radius` and `beam.flare` — how heavy the column reads and how hard it opens out where it
  lands. `beam.charge` and `beam.lifetime` are the wind-up and the hold, which are what make this
  ability feel unlike the other three, and `beam.coreWidth` / `beam.coreFill` decide whether the
  layers stay separable or blow out to white.
- `snare.zoneRadius` — the one number the whole far cast is built on. It resizes the targeting
  circle, the tendrils, the rim arcs, the burnt field and the pillar's throat together, live.
  After that, `snare.snapTime` and `snare.height` carry the moment it opens, and `snare.tendrils` /
  `snare.rimArcs` / `snare.strands` decide how much of that footprint is actually lit.
- `zone.boundary` and `zone.snap` — how thick the far-cast circle's edge reads, and how hard it
  overshoots on the way out. Between them they decide whether the indicator feels like a UI overlay
  or like something the caster is doing.

---

## Performance notes

- Abilities, decals, bursts and particles are pooled, per type. Twelve casts in a row build at most
  **four** instances of an ability and then stop allocating.
- The whole crystal field is three draw calls regardless of crystal count; the cap is 288.
- A whole bolt is **two** draw calls regardless of filament count; the cap is 24 filaments at 72
  samples each. Nothing about the path touches the CPU, so `strands` is nearly free.
- A whole snare — leash, pillar, tendrils and rim arcs — is **two** draw calls plus one for the
  field, regardless of how many filaments are in it; the cap is 56 across the four roles. As with
  the bolt, none of the shape touches the CPU, so raising `tendrils` or `rimArcs` is nearly free.
  Its targeting circle is two more: one quad and one ring strip.
- A whole beam is **six** draw calls regardless of how many coils and discs are on it — three tube
  passes over one shared geometry, plus one instanced draw each for the coils, the discs and the
  charge orb. As with the bolt, none of the shape touches the CPU, so `coils` and `rings` are
  nearly free. It takes two of the six dynamic lights (the column and the caster's hands), so four
  concurrent beams would exhaust the pool; `LightPool.acquire()` returns null and every use of the
  handle is guarded.
- The six dynamic point lights are created at boot and parked at zero intensity rather than added
  and removed — changing the light count forces three to recompile every material.
- Shadow maps update exactly once per frame even though the scene is rendered several times.
- `renderer.compileAsync()` runs during boot so the first cast never stutters on shader compile.
- Pixel ratio is capped at 1.75; the depth and distortion buffers are half resolution.

Measured on a default cast: 32 draw calls idle, ~69 with a full ice field standing and ~49 with a
bolt in the air, ~1150 live particles. A snare standing with its cage, field and rim burns is ~45
draw calls and ~480 live particles, and arming its circle costs two. Four concurrent casts —
the pool's ceiling, whichever slots they came from — peaks at ~186 draw calls and five of the six
dynamic lights.

Live counters (FPS, live particles, instances, draw calls) are in the top-right of the HUD.

---

## The archive

`src/archive/` holds the previous incarnation of this project: a four-element bending sandbox
(fire, water, earth, air) cast along a freehand-drawn spline, plus a walk mode that let the avatar
ride the same stroke. None of it is imported by the live app, so Vite never bundles it.

It was retired because this build replaced path drawing with a linear skillshot, which removed the
input every one of those systems was built on. The raymarched flame and water surfaces in
particular are worth mining. See `src/archive/README.md` for what is in there and how to restore a
piece of it.

---

## Known rough edges

- Crystals are drawn with `transparent: true` and `depthWrite: true`. That is the right trade for
  near-opaque ice and it keeps the field from sorting through itself, but at low `ice.opacity` the
  sorting artefacts between overlapping spikes become visible.
- The eruption front is a straight line on a flat floor. Both assumptions are baked in — the ground
  is a single plane at y = 0, and the aim raycast targets that plane.
- The distortion pass runs with nothing writing to it. It costs a half-res clear per frame.
- The impact cluster is placed radially around the end point, so at very short cast distances it
  can overlap the band behind it more than it should.
- The far cast inherits the flat-floor assumption twice over: the circle is drawn on a single quad
  at `y = 0`, and the snare's tendrils and rim arcs are placed against that same plane. Neither
  would drape over a step.
- Both the targeting circle and the snare's field are additive, so the footprint brightens the
  floor rather than shading it. On a pale floor the boundary would need a non-additive pass under
  it to stay readable.

---

## Licence

Code is provided as-is for the purposes of this project. The bundled HDR probe and the character
FBX retain their original licences.
