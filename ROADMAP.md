# Roadmap

The prototype is built in phases. Each phase is gated on one concrete question, and
nothing downstream is begun until that question is answered in the affirmative. Two
gates can invalidate the design outright and are marked accordingly. The ordering is
deliberate: the autonomous execution layer, which is the project's real risk, comes
first; the command-bandwidth economy — the headline mechanic but a trivial one to
implement — comes late, because it only becomes interesting once there is enough
happening on the field that the player would want more actions than they are allowed.

## Phase 0 — Motion skeleton · **complete**

Fixed-timestep loop with interpolated rendering; one steering primitive, arrive plus
separation.

*Gate (met):* a handful of units move to a clicked point, keep their spacing, and
come to rest cleanly.

## Phase 1 — Combat and the stance kernel · **complete** · design-critical

Targeting, weapon range, cooldown, damage, and death. The single movement primitive
parameterized by a stance, with the four verbs — Move, Attack-move, Hold, Withdraw —
defined as presets over it. Area-target geometry for line and dispersed formations
was folded in here, since an area is only a different destination shape over the same
primitive. Direct click-to-issue control for testing, against a scripted hostile
force.

*Gate (met):* Attack-move against the scripted enemy engages sensibly and does not
overchase, then resumes; Move, Attack-move, and Hold are visibly and correctly
different. This is the phase on which everything rests — if execution had felt like a
slower real-time-strategy game the player wished they could micromanage, the design
would have failed here.

## Phase 2 — Doctrine as editable state · **complete**

The stance parameters the verbs preset are now persistent, editable overrides on the
units: fire discipline, pursuit distance (leash), spacing, target priority, and a
morale auto-rout reflex. Doctrine is set on the current selection and then shapes
every future order at no command cost; defaults reproduce the Phase 1 baseline. This
build also fixed the arrival-spread regression, so point-moves settle into an evenly
spaced cluster while area formations keep their shape.

*Gate (met):* changing a force's leash or fire discipline visibly changes how the
*same* order executes — an Attack-move advances silently under hold-fire, chases
further under long pursuit, or spreads wider under loose spacing.

## Phase 3 — Groups and the full order packet · **complete**

Units belong to named groups that can be formed, merged, and split freely by selecting
and pressing Group. Doctrine now lives on the group. An order is issued to a group as a
single packet — verb and geometry, plus an optional trigger and follow-up. Triggers
are fraction-based (a share of the group's strength) and one-shot, so a threshold fires
its transition once rather than chattering at the boundary.

*Gate (met):* "Attack-move onto this line; if the group drops below forty percent
strength, withdraw to the rally point" runs end to end on group A with no further
input — the group pulls itself back once it has taken enough casualties.

## Phase 4 — Command bandwidth · **complete** · thesis-validation

Command is now the scarce resource, in three selectable modes. *Free* imposes no limit.
*Continuous* is a token bucket: commands regenerate to a cap and the simulation runs
without interruption. *Phased* runs for an interval, then pauses and grants a fixed
pool of command points to spend before resuming, with no carry-over. Issuing an order
to a group costs one command whatever its size, with any attached trigger and follow-up
free; a doctrine change costs one; selecting, grouping, and planning are free. A fully
random arrival model was rejected — unpredictability belongs in execution, not in the
player's control of their own forces.

*Gate (met):* with the budget binding, the player banks commands during lulls, values
a pre-set escape clause because it spares a scarce command mid-crisis, and rations
toward the most valuable decision. Both restriction modes are available so either
temperament — continuous pressure or deliberate phased planning — is served.

The remaining phases add the content and depth that turn the testbed into a game. The
framework is intended to be tunable to different eras of warfare; the first target is
the ancient-to-medieval period, which shapes the troop roster below.

## Phase 5 — Troop types and fortifications · **complete**

Four classes with distinct shapes and a rock-paper-scissors of roles — light infantry,
heavy infantry, bowmen, cavalry — with combat split into ranged and melee, a cavalry
charge bonus, braced heavy infantry that breaks a charge, and armour. A group marches
at the pace of its slowest member so mixed forces stay cohesive, accelerating to their
own speed on contact. A static barricade class is implemented but not yet placed, since
meaningful fortifications need pathfinding.

*Gate (met):* a mixed force reads clearly by shape and the match-ups resolve as
expected — cavalry rides down loose bowmen but breaks on a braced line, bowmen punish
foot at range but fold in contact — and cavalry grouped with foot keeps pace instead of
outrunning them.

## Phase 6 — Terrain · next

A terrain field beneath the units that affects both movement and combat: passability
(impassable water or cliffs), movement cost (forest, mud, slope), and combat modifiers
(cover, high ground). Straight-line steering gives way to movement that respects
impassable terrain, with local avoidance and, if needed, pathfinding. Terrain is what
makes positioning — chokepoints, high ground, flank protection — a real concern.

*Gate:* a force ordered across the map routes around impassable terrain and is visibly
slowed by difficult ground, and holding high ground or a chokepoint confers a
measurable advantage.

## Phase 7 — Map and scenario editor

A tool to author scenarios: paint terrain, place fortifications, deploy both forces
with their groups and doctrine, and set objectives, then save and load the result
(as JSON). This is what makes it possible to build situations worth playing rather
than the single hard-coded test scenario.

*Gate:* a scenario built in the editor — terrain, two deployed forces, an objective —
saves, reloads, and plays.

## Phase 8 — Opponent, imperfect execution, and polish

An opponent that issues order packets at the intent level — cheap precisely because the
Phase 1 autonomy already supplies the tactical execution for both sides, and now able
to use troop types and terrain. Morale and fatigue modulating obedience and accuracy,
so an order becomes intent rather than certainty; at this point morale stops being a
fixed placeholder and becomes an emergent attribute driven by training, fatigue, and
officer quality, which the player influences only indirectly. Proper formation-keeping
in motion. This is where the prototype becomes a complete game.

## Deferred

- **Patrol** and **Screen** verbs. Patrol is the only order needing multi-point route
  geometry rather than a single destination; both are held back to keep the earlier
  phases' geometry simple, and fit naturally alongside the troop-types work.

## Open design questions

- **Marching formation.** An ordered cohort now moves as a formation — fixed slot
  offsets and a single origin that advances at the slowest member's speed — so fast
  troops no longer outrun slow ones and the block holds its shape as it translates. What
  remains is *rotation*: the block does not yet turn to face its line of march, and slots
  are not re-oriented when the formation changes heading. That refinement is slated for
  the final phase unless the terrain fights demand it sooner.
- **Pathfinding depth.** Terrain forces a choice between cheap local obstacle
  avoidance and full grid or navmesh pathfinding. Local avoidance may suffice for
  sparse obstacles; dense terrain and chokepoints will likely require real
  pathfinding. To be settled when Phase 6 begins.
- **Epoch tuning.** The class roster, ranges, and speeds are being set for the
  ancient-to-medieval period first, but the combat and doctrine framework is meant to
  generalise to other eras. Keeping class stats in one table, separate from the
  mechanics, is what will make retuning to a different period tractable.

## Settled

- **What counts as one command.** One order to a group costs one command regardless of
  size; a doctrine change costs one; a compound packet (order plus trigger plus
  follow-up) still costs one, since bundling the escape clause is the reward for
  planning; selecting, grouping, and reorganisation are free. Implemented in Phase 4.
- **Bandwidth model.** Both a continuous token bucket and a phased pause-window mode
  ship, with a free mode for setup; a fully random arrival model is rejected, because
  randomness in the resource used to control one's own forces reads as unfair rather
  than realistic.
- **Rally geometry.** A follow-up rally is always a single point, never an area.
