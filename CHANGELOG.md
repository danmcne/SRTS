# Changelog

All notable changes to this project are recorded here. The format follows the
conventions of [Keep a Changelog](https://keepachangelog.com/), and the project
uses semantic-style version numbers, one minor version per completed phase.

## [0.6.1] — 2026-07-02

Fix: cohesion via a proper marching formation.

### Fixed

- Fast troops overtaking slow ones inside a moving cohort. The previous pace cap was
  computed per group, so ordering two groups together (for example bowmen behind foot)
  let each keep its own speed and the faster overtook; even within one group, capping
  only the top speed did not hold a formation. Ordering a selection now builds a
  marching formation — fixed slot offsets and a single origin that slides to the
  destination at the cohort's slowest speed, with each unit holding its offset from the
  moving origin. The block advances as one, so no unit outruns the formation, across
  one group or several ordered at once. A unit leaves the formation to fight on contact
  and returns to its slot afterward.

### Changed

- Point orders and area orders both produce a marching formation cohort rather than
  independent per-unit destinations. Slot assignment preserves each unit's starting
  place to avoid crossing paths.

## [0.6.0] — 2026-07-02

Phase 5: troop types and cohesion.

### Added

- Four troop classes with distinct shapes and stats, all defined in one table: light
  infantry (triangle), heavy infantry (square), bowmen (diamond), and cavalry
  (arrowhead). Each has its own radius, speed, health, armour, and attack.
- Split of combat into ranged and melee, with per-class attack profiles; bowmen fire at
  a distance and fight weakly in contact, other classes must close.
- The match-up triangle from three rules: a cavalry charge bonus on reaching contact
  from speed; braced heavy infantry (holding and stationary) that negates and blunts a
  charge, recoils the horse, and strikes back with a spear bonus; and armour that
  reduces incoming damage.
- Group cohesion: a group marches at the pace of its slowest present member, with units
  accelerating to their own speed once engaged. Point orders now form units into a
  block at the destination rather than a pile.
- A static barricade class (immovable, armoured, destructible, blocks via separation),
  defined and functional but not placed in the default scenario.
- A shape legend, and a mixed-arms scenario: player foot / bows / horse groups against
  a hostile force of heavy infantry, bowmen, and cavalry.

### Changed

- The unit record carries a class, per-class radius and stats, a facing used to orient
  its shape, and melee-contact bookkeeping for the charge rule. Separation distance now
  scales with unit size. The single global speed constant was replaced by per-class
  speed.

## [0.5.0] — 2026-07-02

Phase 4: command bandwidth.

### Added

- Command bandwidth in three selectable modes. *Free*: no limit. *Continuous*: a
  regenerating token bucket with a cap, spent one per command, with the simulation
  running without interruption. *Phased*: the simulation runs for a fixed interval,
  then pauses and grants a fixed pool of command points to spend before resuming, with
  no carry-over.
- Cost rules: issuing an order to a group costs one command regardless of group size,
  with any attached trigger and follow-up free; changing a doctrine setting costs one.
  Selecting, grouping, switching the active order type, and planning a packet are free.
- Command readout in the situation panel (pool, regeneration or countdown, and an
  empty-budget flash), a command-window banner with a Resume control for phased mode,
  a mode-cycle button, and a paused veil over the field.
- Space toggles a manual pause in free and continuous modes, and resumes the command
  window in phased mode.

### Changed

- Removed the morale dial from the doctrine panel, leaving four doctrine axes. The
  auto-rout reflex now runs on a fixed low threshold in the engine as a placeholder;
  emergent morale is deferred to a later phase.

## [0.4.0] — 2026-07-02

Phase 3: named groups and the full order packet.

### Added

- Named groups. Every unit belongs to a group (player group A, hostile OpFor).
  Grouping is free: selecting units and pressing Group (or Ctrl+G) moves them into a
  fresh group, which covers forming, merging, and splitting in one action.
- Group selection: double-click a unit, or click its chip in the top strip. A group
  strip shows each group's label, headcount, strength bar, current order, and whether
  its trigger is armed or has fired.
- Doctrine moved from the loose selection onto the group, so it has a persistent home;
  the doctrine panel now edits the selected group.
- The order packet: an order issued to a group carries an optional trigger and
  follow-up. Triggers are a strength threshold (below 50 / 40 / 25 percent of the
  group's strength at order time), reaching the objective, or a timer. Follow-ups are
  any of the four verbs, sent to a rally point (set on the map, or defaulting to the
  group's rear). Triggers are one-shot — they fire once and switch the order.
- Order-packet panel for editing a group's trigger, follow-up, and rally point, and a
  rally diamond drawn for the selected group.

### Changed

- The unit record replaced its per-unit doctrine fields with a group reference;
  effective stance, target priority, and the morale reflex now read the group's
  doctrine. Empty groups are removed automatically as units die.
- Removed the Bait raider test button; the pursuit doctrine and the packet mechanics
  cover its purpose.

## [0.3.0] — 2026-07-02

Phase 2: doctrine as editable state, plus the arrival-spread fix.

### Added

- Doctrine: persistent, editable per-unit settings that override the verb presets'
  secondary parameters, applied to the current selection through a doctrine panel.
  Five axes — fire discipline (by verb / hold / return / free), pursuit distance
  (short / normal / long, scaling the leash), spacing (tight / normal / loose,
  scaling the separation radius), target priority (nearest / weakest), and a morale
  reflex (stand / rout early / rout late).
- Morale auto-rout: a unit whose health falls past its morale threshold breaks off
  and withdraws to its own rear, reads muted with a dashed ring, and does not fight;
  a fresh order rallies it.
- Weakest-target priority, concentrating fire on wounded enemies.
- Doctrine panel that reflects the selection, shows *mixed* when selected units
  disagree on a setting, and dims when nothing is selected.

### Fixed

- Arrival spread. Units sent to a single point were pulled together indefinitely and
  settled almost overlapping, because the destination pull never released. A unit now
  releases its arrive pull once it reaches its post (with hysteresis), so a point-move
  settles into an evenly spaced cluster. Area formations keep their shape, because
  their slots are already spaced wider than the separation radius.

### Changed

- The unit record gained doctrine fields and an at-post / routing state. Effective
  stance is now computed from the verb base with doctrine modifiers applied, rather
  than read straight from the verb preset.

## [0.2.0] — 2026-07-02

Phase 1: combat and the stance kernel, plus area-target geometry.

### Added

- Combat model: target acquisition within a sense range, weapon range, per-shot
  damage with a small variance, a firing cooldown, and unit death.
- Four order verbs — Move, Attack-move, Hold, Withdraw — implemented as presets over
  four stance parameters (`leash`, `fire`, `stopToFight`, `urgency`) rather than as
  separate behaviours.
- Leash-and-anchor mechanic: a unit that begins fighting drops an anchor and pursues
  only within its leash of it, then breaks contact and resumes its order. This is
  what stops Attack-move units from overchasing.
- Fire discipline: `HOLD`, `RETURN`, and `FREE`, distinguishing units that never
  shoot, shoot back only, and engage anything reachable.
- Area orders. In addition to point targets, an order may target a rectangle; units
  distribute across it in aspect-matched slots. A wide, shallow rectangle produces a
  firing line; a squarer one a dispersed block. Slot assignment is greedy-nearest to
  keep formations from tangling.
- Active-order selection by number keys `1`–`4` and by on-screen buttons, with the
  current order shown in the readout.
- Factions (player and hostile), with distinct rendering, and a scripted hostile
  force that digs in on a defensive line at scenario start.
- Test tools: **Bait raider** (sends one hostile fleeing, to verify leash break-off)
  and **Red attacks** (orders the hostile force to advance).
- Visual feedback: shot tracers coloured by shooter, and health bars on damaged
  units.

### Changed

- Right-drag is now an *order-to-area* gesture; right-click remains order-to-point.
- Selection is restricted to friendly units.
- The unit record gained faction, verb, anchor, and weapon-cooldown fields; the
  Phase 0 `hasTarget` destination flag was generalized into a post plus a per-verb
  stance.

## [0.1.0] — 2026-07-02

Phase 0: the motion skeleton.

### Added

- Fixed-timestep simulation at 20 Hz driven by an accumulator, decoupled from
  rendering, with a clamp against the spiral-of-death after a stall.
- Interpolated canvas rendering between the previous and current simulation step, so
  motion stays smooth independent of frame rate.
- One steering primitive: arrive (ease into a destination within a slowing radius)
  plus separation (push apart within a fixed radius), with a rest test that damps
  residual motion so groups stop cleanly instead of jittering.
- Unit selection by box-drag or click, and move orders by right-click.
- A situation-map presentation: desaturated slate ground, faint plotting grid, a
  monospace heads-up readout, and a single amber command accent.
- A heads-up readout of simulation rate, unit and selection counts, and a
  moving/settled state indicator.

[0.6.1]: #061--2026-07-02
[0.6.0]: #060--2026-07-02
[0.5.0]: #050--2026-07-02
[0.4.0]: #040--2026-07-02
[0.3.0]: #030--2026-07-02
[0.2.0]: #020--2026-07-02
[0.1.0]: #010--2026-07-02
