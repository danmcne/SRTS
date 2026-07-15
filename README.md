# SRTS

A semi-real-time strategy prototype in which the scarce resource is *command*, not
mouse speed. The player acts as a commander who issues intent — where to go, how to
fight, when to break off — and the units supply the tactical execution themselves.
The design thesis is deliberately narrow:

> This is not a game about controlling units; it is a game about controlling an
> organization.

Most real-time strategy games simulate combat and rely on the player's clicks to
serve as the unit-level intelligence. This project moves that intelligence into the
simulation, so the player is free to think one level up: what should the force *try
to do*, and where is the single most valuable decision to spend limited command
capacity right now.

Each build is a single self-contained HTML file. There is no build step, no package
manager, and no external dependency: open the file in any modern browser and it runs.

## Current state

The prototype is being built in gated phases (see `ROADMAP.md`). Six are complete.

**Phase 0 — motion skeleton.** A fixed-timestep simulation with interpolated
rendering, and one steering primitive: *arrive* (ease into a destination) plus
*separation* (do not overlap neighbours). Units sent to a point form a tight,
non-overlapping cluster and come to rest cleanly.

**Phase 1 — combat and the stance kernel.** Units acquire targets, fire, take
damage, and die. On top of the movement primitive sit four order *verbs* — Move,
Attack-move, Hold, Withdraw — which are presets over a small set of stance
parameters rather than four separate behaviours. Orders may target a point (units
cluster) or an *area* (units spread into slots), and the shape of the area drawn
determines the formation: a wide, shallow rectangle yields a firing line, a squarer
one a dispersed block. A scripted hostile force and two test tools are included so
the execution layer can be watched under pressure.

**Phase 2 — doctrine as editable state.** The four stance fields the verbs preset
are lifted into persistent, editable settings on the units — *doctrine*. The verb
still sets what a force is doing now; doctrine sets how it fights, always. Five axes
are editable on the current selection: fire discipline, pursuit distance, spacing,
target priority, and a morale reflex that makes a unit auto-withdraw once its health
falls past a threshold. Defaults reproduce the Phase 1 behaviour, so doctrine adds
control without changing the baseline.

**Phase 3 — groups and the full order packet.** Units belong to named groups, which
can be formed, merged, and split freely by selecting units and pressing Group.
Doctrine now lives on the group rather than on the loose selection. An order is
issued to a whole group as one packet — verb and geometry, plus an optional *trigger*
(a monitored condition) and *follow-up* (the order to switch to when it fires).
Triggers are fraction-based and one-shot: "if this group falls below forty percent
strength, withdraw to the rally point" runs to completion on its own, with no further
input. This is the first appearance of the design's core idea — one command buying a
span of autonomous behaviour, escape clause included.

**Phase 4 — command bandwidth.** Command itself becomes the scarce resource, in three
selectable modes: *free* (no limit, for setup), *continuous* (a regenerating token
bucket — bank commands in a lull, spend a burst to set up a fight), and *phased* (the
sim runs for an interval, then pauses and grants a fixed pool of command points to
spend before resuming). Issuing an order to a group costs one command whatever the
group's size, with any attached trigger and follow-up riding along free; changing a
doctrine setting costs one; selecting, grouping, and planning a packet are free. This
build also removed the morale dial from the doctrine panel.

**Phase 5 — troop types and cohesion.** One generic token becomes four classes with
distinct shapes and a rock-paper-scissors of roles: light infantry (fast, fragile),
heavy infantry (slow, armoured, braces against cavalry), bowmen (ranged, weak in
melee), and cavalry (fastest, charge bonus on contact). Combat splits into ranged and
melee; cavalry that reaches contact from speed lands a charge bonus, unless it strikes
braced heavy infantry, which negates the charge and lets the spearmen strike back.
Armour reduces incoming damage. A group now marches at the pace of its slowest member,
so mixed forces stay cohesive rather than letting the cavalry string out ahead. A
static barricade class exists but is not yet placed on the field.

The file to open for the latest build is `command_p5.html`.

## The stance model

The heart of the design is that the four verbs share one motor behaviour and differ
only in four fields:

- **leash** — how far a unit will stray from its anchor to close on an enemy. For
  Hold this doubles as the guard radius around the post.
- **fire** — its fire discipline: `HOLD` (do not shoot), `RETURN` (shoot back only,
  never chase), or `FREE` (engage anything reachable).
- **stopToFight** — whether it halts to engage, or fires while continuing to move.
- **urgency** — a speed multiplier; Withdraw runs.

From these, the verbs read as follows. Move beelines to its destination, deviating
for nothing, and returns fire only at point-blank while passing. Attack-move
advances but stops to destroy any enemy it meets within its leash, then resumes.
Hold sits on its post, engages within the guard radius, and returns after each
fight. Withdraw runs to the rally point at maximum speed and does not stop to
fight. This is the behavioural contrast the phase exists to prove: the same code,
four presets, four legibly different intents.

The leash is what prevents overchasing. When an Attack-move unit begins a fight it
drops an *anchor* at that spot and will pursue only so far from it; if the enemy runs
beyond reach, the unit breaks contact and resumes its advance. The *pursuit* doctrine
scales how far that leash reaches.

## Doctrine

Where the stance model fixes what each verb does, doctrine lets the player retune the
secondary parameters persistently, per force. The doctrine panel edits whatever is
currently selected; each setting is remembered on those units and shapes every future
order they receive, at no command cost. Defaults reproduce the Phase 1 behaviour
exactly, so a force with untouched doctrine behaves as before.

- **fire** — override the verb's fire discipline, or leave it *by verb*. Setting a
  force to *hold fire* makes even an Attack-move advance without shooting; *fire at
  will* makes a plain Move engage.
- **pursuit** — scale the verb's leash short, normal, or long, changing how far a
  force will chase before breaking off.
- **spacing** — scale the separation radius tight, normal, or loose, changing how
  densely a force packs and how widely it settles on arrival.
- **target** — pick the *nearest* enemy, or the *weakest*, to concentrate fire on
  wounded units.

Morale is no longer a doctrine setting. A unit still breaks and withdraws to its rear
once its health falls very low, but that reflex now runs on a fixed threshold in the
engine rather than a player dial; it becomes an emergent unit attribute in a later
phase.

The intended test is to give one order and watch the same order execute differently
as doctrine changes: an Attack-move that advances silently under *hold fire*, chases
further under *long* pursuit, or spreads wider under *loose* spacing.

## Groups and the order packet

Every unit belongs to a named group (the player starts with group A; the hostile
force is OpFor). Grouping is free and is the way organization is rewarded before a
battle rather than paid for during it. Select any units and press **Group** (or
Ctrl+G) to move them into a fresh group; this one action covers forming, merging, and
splitting, since it simply reassigns the selection. Double-click a unit, or click a
group's chip in the strip along the top, to select its whole group.

An order is issued to a group as a single packet with up to four parts. The **verb
and geometry** are the active intent, issued the usual way — pick a verb, right-click
a point or right-drag an area. The **doctrine** is the group's standing settings,
edited in the doctrine panel. The **trigger** is one condition watched while the order
runs, and the **follow-up** is the order the group switches to when the trigger fires.

The order-packet panel edits the selected group's trigger and follow-up. Available
triggers are a strength threshold (below fifty, forty, or twenty-five percent of the
group's strength when the order was given), reaching the objective, and a timer.
Available follow-ups are any of the four verbs; withdraw is the common one. The rally
row sets where the follow-up sends the group — click it to arm, then click the map,
or leave it unset to default to the group's own rear. Triggers are one-shot: they
fire once and switch the order, rather than re-firing every tick at the boundary. A
group's chip shows its strength bar and whether its trigger is armed or has fired.

The intended test — and the phase's gate — is to give group A an attack-move onto the
hostile line with a "below forty percent, withdraw" packet attached, start the fight
(the **Red attacks** button makes it a real one), and watch the group pull itself back
to the rally point on its own once it has taken enough casualties.

## Troop types and cohesion

Units come in four classes, distinguished on the map by shape (a legend sits along the
bottom): a triangle is light infantry, a square heavy infantry, a diamond bowmen, and
an arrowhead cavalry. Shape marks the class; colour marks the side. Each class has its
own speed, health, armour, and attack, all kept in a single table so the roster can be
retuned for a different era.

Combat is split into ranged and melee. Bowmen strike from a distance and are feeble in
a melee; everyone else must close to contact. Three interactions produce the match-up
triangle without any pairing being special-cased. Cavalry reaching contact from speed
lands a charge bonus, which makes it deadly against loose bowmen and routing troops.
Heavy infantry that is holding and stationary is *braced*: a cavalry charge against it
is negated and blunted, the horse takes recoil, and the spearmen strike back with a
bonus — so cavalry breaks on a steady line. Armour then reduces whatever damage lands,
which is why heavy infantry endures and light infantry does not. The upshot is the
familiar triangle: cavalry beats exposed archers, archers beat unshielded infantry at
range, and a braced infantry line beats cavalry.

Cohesion addresses mixed-speed groups. Ordering a selection to move creates a marching
formation: the units take fixed positions in a block, and a single origin slides toward
the destination at the *slowest* member's speed while every unit holds its offset from
that moving origin. Because the whole block advances as one, a fast unit cannot outrun a
slow one — cavalry or bowmen ordered alongside heavy infantry keep station instead of
walking away from them, and this holds even when the order spans several groups at once.
The instant a unit engages it leaves the formation to fight at its own speed, then falls
back into its slot afterward. If you want the cavalry to move at its own pace, order it
on its own — organisation remains the lever. Rotating the block to face the line of
march is a later refinement; for now the formation translates without turning.

A static barricade class is implemented — immovable, armoured, and destructible, and it
blocks movement by making units bank up against it — but it is not placed in the
default scenario, because without pathfinding units slide along an obstacle rather than
route around it. Fortifications become useful once terrain and pathfinding arrive.

## Command bandwidth

Command is the scarce resource that gives the design its name, and the mode selector
in the test bar switches between three ways of rationing it. In *free* mode there is
no limit — useful for setting up a scenario. In *continuous* mode commands regenerate
over time into a bucket with a cap; each command spends one, and the simulation runs
without interruption, so the skill is banking commands during quiet stretches and
spending a burst when a fight is joined. In *phased* mode the simulation runs for a
fixed interval and then pauses, granting a pool of command points; the player spends
them during the pause and presses Space (or Resume) to continue, and unspent points do
not carry over — a more deliberate, turn-like rhythm.

The cost rules are the same in both limited modes. Issuing an order to a group costs
one command regardless of how many units are in the group, and any trigger and
follow-up attached to that order ride along at no extra cost — bundling foresight is
rewarded, not penalised. Changing one doctrine setting costs one command. Selecting
units, forming and reorganising groups, switching the active order type, and planning
a packet's trigger, follow-up, and rally point are all free. The principle is that
organisation is never taxed; only committing an order or a change to the field is.

The command readout in the situation panel shows the current pool and, in phased mode,
the countdown to the next window; it flashes when a command is attempted with an empty
budget. Space pauses and resumes: in continuous and free modes it is a manual pause for
looking at the field; in phased mode it closes the command window and resumes the run.

## Controls

Selection and orders follow the familiar real-time-strategy idiom, with one
addition — orders can target an area, not only a point.

- **Left-drag** — box-select friendly units. **Left-click** — pick one, or click
  empty ground to clear. Hold **Shift** to add to the selection.
- **Right-click** — issue the active order to a point; the selected units cluster
  there.
- **Right-drag** — issue the active order to an *area*; the selected units spread
  across the rectangle. Draw it wide and shallow for a line, square for a dispersed
  block.
- **1 / 2 / 3 / 4** — set the active order to Move / Attack-move / Hold / Withdraw
  (also available as buttons).
- **A** select all · **Esc** clear selection (and cancel rally placement) · **R**
  reset the scenario.
- **Ctrl+G** or the **Group** button — move the selected units into a fresh group.
  **Double-click** a unit, or click a group **chip** (top strip), to select its group.
- **Doctrine panel** (top-right) and **order-packet panel** (below it) — click any row
  to cycle that setting for the selected group. The doctrine panel shows *mixed* when
  several selected groups disagree; the rally row arms map-click placement.
- **Mode** button — cycle the command-bandwidth mode (free / continuous / phased).
  **Space** — manual pause in free and continuous modes; resume the command window in
  phased mode.

Test tool: **Red attacks** orders the whole hostile force to advance on the player
line, turning the scripted defenders into an active threat.

## Architecture

The simulation runs on a fixed timestep (20 Hz) driven by an accumulator, decoupled
from rendering. Each animation frame advances the simulation by as many fixed steps
as real time has accrued, then draws the world interpolated between the previous and
current step. This keeps motion smooth regardless of frame rate and keeps the
simulation deterministic in its stepping, which matters increasingly as later phases
add order evaluation and an opponent brain.

A simulation step runs in passes: each unit first decides on a movement target and
possibly a shot (target acquisition, leash and fire-discipline checks); then every
unit computes a desired velocity from arrive-plus-separation toward that target; then
positions integrate and residual motion is damped to rest; finally the dead are
removed and shot-flashes expire. Separating the decision pass from the movement pass
keeps neighbour queries order-independent.

A unit carries its position and velocity, health, faction, current verb, its post
(destination or assigned formation slot), its fighting anchor, and a weapon cooldown.
The command layer — verb selection and order issue — sits entirely above the
simulation and only mutates unit orders, so the simulation remains inspectable on its
own.

## Tuning

The constants worth adjusting are grouped at the top of the script, in two blocks.
Steering: `MAX_SPEED`, `ARRIVE_RAD`, `SEP_RAD`, `SEP_SPEED`, `STEER_GAIN`,
`REST_EPS`. Combat: `SENSE`, `W_RANGE`, `W_DMG`, `W_CD`. The four verbs and their
stance fields are defined together in the `VERB` table, which is the right place to
experiment with how each order feels.

## Files

- `command_p5.html` — the current build (Phase 5).
- `command_p4.html` — the Phase 4 build (command bandwidth), for reference.
- `command_p3.html` — the Phase 3 build (groups and the order packet), for reference.
- `command_p2.html` — the Phase 2 build (doctrine), for reference.
- `command_p1.html` — the Phase 1 build (combat and stance), for reference.
- `command_p0.html` — the Phase 0 motion skeleton, for reference.
- `ROADMAP.md` — the phased plan, its gates, and open design questions.
- `CHANGELOG.md` — the record of what changed in each version.

## Not yet built

Terrain that affects movement and combat (and the pathfinding it requires); a map and
scenario editor to paint terrain, deploy forces, and set objectives; and finally an
opponent that issues packets at the intent level, imperfect execution from fatigue and
officer quality (at which point morale becomes an emergent attribute), and marching
formation that holds its shape. The framework is intended to be tunable to different
eras of warfare; the first target is the ancient-to-medieval period. See the roadmap
for the full sequence.
