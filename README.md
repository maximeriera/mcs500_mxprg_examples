# MCS500 `.mxprog` Examples

Annotated, ready-to-run example programs for the **Mecademic MCS500** SCARA robot — and the MCS500 only.

Every example is written to be **read as much as run**. The comments are the point: they explain *what* a command does, *why* you would choose it over the alternative, and *what you should see happen*. You should be able to open any file, read it top to bottom, and understand that feature without a robot in front of you.

Two audiences, one set of files:

- **Learning the robot?** Start at `01_basic/01` and read in order. By the end of `01_basic/` you can write your own programs; by the end of `03_advanced/` you can design a cell around one.
- **Demonstrating the robot?** Each file is self-contained and ends with concrete things to try, along with the question that feature answers.

> [!IMPORTANT]
> This is a collection of examples, not Mecademic product documentation.
> The authoritative reference is always the [Programming Manual](https://resources.mecademic.com/en/doc/MC-PM-MCS500/latest/).
> See [Status and licence](#status-and-licence).

## Who these examples are written for

Someone with **little or no background in robotics or industrial automation** — a newcomer, a
student, an engineer from another discipline, someone evaluating an MCS500 for the first time.

The goal is that you can read a file top to bottom and come away able to *use* the feature and
*explain* it to somebody else. That goal shapes every explanation in this repo, and it
sometimes conflicts with being exhaustively precise. [DISCLAIMER.md](DISCLAIMER.md) lists
where.

## Contents

- [MCS500 `.mxprog` Examples](#mcs500-mxprog-examples)
  - [Who these examples are written for](#who-these-examples-are-written-for)
  - [Contents](#contents)
  - [Safety first](#safety-first)
  - [What you need](#what-you-need)
  - [What is an `.mxprog` file?](#what-is-an-mxprog-file)
    - [Comments](#comments)
    - [Where else these files come from](#where-else-these-files-come-from)
  - [What the command interface does *not* have](#what-the-command-interface-does-not-have)
  - [The MCS500 in one page](#the-mcs500-in-one-page)
  - [Conventions used in these examples](#conventions-used-in-these-examples)
  - [Coming from the Meca500?](#coming-from-the-meca500)
  - [Folder structure](#folder-structure)
  - [The three setup programs](#the-three-setup-programs)
  - [Three kinds of setting](#three-kinds-of-setting)
  - [File index](#file-index)
  - [Running an example](#running-an-example)
    - [Run it in simulation first](#run-it-in-simulation-first)
    - [If something goes wrong](#if-something-goes-wrong)
    - [Editor support](#editor-support)
  - [Target hardware and firmware](#target-hardware-and-firmware)
  - [Status and licence](#status-and-licence)
  - [Official Mecademic resources](#official-mecademic-resources)

---

## Safety first

**These programs move a real robot.** Read this section before running anything.

- **Clear the workspace.** Every example opens by moving to the **zero position**: the arm fully
  stretched straight out in front of the base, the shaft all the way up. From wherever the arm
  was left, that can be a large sweep.
- **Know your power supply.** The MCS500's safety power supply has a key switch. In
  *automatic* the robot runs once its motors are powered with the RESET button; in *manual* it
  only moves while someone holds the enabling device halfway; in *locked* it will not power its
  motors at all. If you are using the prototyping dongle instead of wired safety I/O, the user
  manual is explicit: manual mode is unavailable, and the robot must be kept in a safety
  enclosure.
- **Run in simulation first** on any setup you have not tested. `ActivateSim(1)` executes
  everything with the motors doing nothing — see [Run it in simulation first](#run-it-in-simulation-first).
- **Check the poses against your cell.** The coordinates here assume a **bare shaft and an
  empty workspace**, or the generic tool described below. A real tool, a fixture, or a part
  changes what is reachable and what collides. Nothing in this repo knows about your fixture.
- **Start slow.** Examples run at reduced speed on purpose. An MCS500 at full speed is quick;
  raise the speed only once you have watched the path.
- **`00_common/00_cold_start.mxprog` can reset your safety envelope.** Its factory-reset section is
  commented out for exactly this reason. On a robot in a working cell, the work zone, joint
  limits, tool sphere and P-Stop 2 behaviour may be deliberate protection — read the audit
  output before changing anything.

These examples are a learning aid. They are **not** a substitute for the risk assessment,
guarding and safety design your installation requires. You are responsible for the safety of
your own cell.

---

## What you need

- An **MCS500** (R1) on firmware **11.x**, ideally 11.3.
- Its power supply, with the E-Stop released and the motors powered (RESET pressed).
- A network connection to it. The robot's default address is `192.168.0.100`.
- A web browser, for the **MecaPortal** — the robot's built-in web interface. Nothing to
  install.
- That is all. There is no SDK, toolchain or licence needed to run these files.

Every example runs on a bare shaft. A tool is optional — and if you fit one, it is driven by
your PLC or PC, not by these programs. See [the MCS500 in one page](#the-mcs500-in-one-page).

---

## What is an `.mxprog` file?

An `.mxprog` file is a **plain-text program** saved from the MecaPortal code editor. It is a sequence of the robot's own text commands — the same commands documented in the [Programming Manual](https://resources.mecademic.com/en/doc/MC-PM-MCS500/latest/) — one per line.

There is no separate programming language and no compilation. What you type in the code editor is exactly what gets sent to the robot over TCP/IP.

```
// Move to the pick approach position
SetJointVel(25)
MoveJoints(-45, 90, -40, 0)
```

Command names are **case-insensitive**, so `SetWrf`, `SetWRF` and `setwrf` all reach the same command. These examples use the manual's spelling throughout.

### Comments

Comments are **C/C++ style**, and they are most of what this repo is:

```
// A single-line comment

/* A block comment,
   used for the header of each example. */
```

`Ctrl` + `/` toggles comments on the selected lines in the MecaPortal editor.

### Where else these files come from

Not every `.mxprog` is hand-written. **RoboDK** exports to this format, and its output is recognisable: a generated header comment, whole-line comments carrying the original target names, and integer program names like `StartProgram(1)` rather than strings. Worth knowing if you meet a generated file and wonder why it looks nothing like these examples.

---

## What the command interface does *not* have

This is the single most important thing to understand before writing a program, and the most common surprise:

> The robot's command interface does **not** support conditionals, loops, or other flow control statements.

There is no `IF`, no `WHILE`, no `FOR`, no jumps or labels. An `.mxprog` program is a **linear sequence of commands**, executed in order.

That is a deliberate design choice. An MCS500 is a precision motion component intended to be driven by a PC, IPC or PLC that already owns the application logic:

- **Program logic lives in the host** — Python, C#, C/C++, Java, or Structured Text on a PLC. The host holds the state machine, the vision result, the recipe, the error handling.
- **The host also drives the tool.** The MCS500 has no tool I/O of its own; the user manual's answer is to control the end-effector directly from your PLC. Gripper valves, vacuum switches, dispenser triggers are all wired to the host.
- **The robot holds the motion** — the trajectories, frames, speeds and blending.
- `.mxprog` programs are the reusable motion building blocks the host calls into.

In practice this works well: the logic stays in a language you already have an IDE, a debugger, tests and version control for, and the motion stays in small named programs you can test one at a time. The trade-off is real, though — if you want the robot itself to branch on a sensor, it cannot, and the host has to.

The three tools that give you structure without flow control:

| Need | Mechanism |
|---|---|
| Reuse / modularity | `StartProgram("folder/program_name")` calls another saved program |
| Parameterised motion | Robot variables (beta): `MovePose(*vars.myGroup.pickPose)` |
| Application logic | The host program over TCP/IP, EtherCAT, EtherNet/IP or PROFINET |

`03_advanced/02_program_calls.mxprog` works through the architecture this implies, and `03_advanced/03_pick_and_place.mxprog` shows the host-side handshake at the grip.

---

## The MCS500 in one page

The facts every file leans on, collected once. All from the user and programming manuals.

**It is a SCARA.** An arm that swings in a horizontal plane, with a vertical shaft at the end that slides up and down and turns. Four joints:

| Joint | What it does | Unit | Range | Top speed |
|---|---|---|---|---|
| θ1 | swings the whole arm about the base | deg | −140 … 140 | 300 °/s |
| θ2 | swings the outer link (the elbow) | deg | −145 … 145 | 500 °/s |
| d3 | moves the shaft up and down | **mm** | −102 … 0 | 700 mm/s |
| θ4 | turns the shaft about its own axis | deg | ±10 turns | 5,000 °/s |

Joint 3 is a **slide**, so its number is a distance: `0` is the shaft all the way up, `-102` all the way down.

**Geometry.** The inner link is 100 mm and the outer link 125 mm, so the end of the shaft reaches at most **225 mm** from the axis of joint 1, and at least about 72 mm. With the shaft fully up its end is 109 mm above the bottom of the base; fully down, 7 mm.

**Poses are four numbers.** `MovePose(x, y, z, γ)`: a position, and **one** angle — how far the tool is turned about the vertical. Every reference frame on the MCS500 has its z axis pointing up, and the robot accepts nothing else, so `SetWrf` and `SetTrf` take four numbers too.

**The zero position** — all joints at zero — is the arm fully stretched straight forward with the shaft up: `MovePose(225, 0, 109, 0)`. It is where the MecaPortal's *Zero all joints* button goes, and it is a **singularity** (see `03_advanced/07`).

**Two postures, not eight.** A point can usually be reached with the elbow bent one way (*righty*, `SetConf(1)`) or the other (*lefty*, `SetConf(-1)`). The singularity — arm straight — is the border between them.

**No homing.** The MCS500 has absolute encoders: it knows where its joints are from power-up. An activated MCS500 is ready for motion.

**Its own move.** `MoveJump` does the lift-travel-land of a pick and place in one command. See `02_intermediate/07`.

---

## Conventions used in these examples

**Units** — distances in millimetres, angles in degrees, time in seconds. Always. No inches, no radians, no configuration to get wrong.

**Every example resets the robot first.** All of them open with the same four-line block:

```
DeactivateRobot()
ActivateRobot()
ResetError()
ResumeMotion()
```

Deactivating loses every non-persistent setting, so each example starts from firmware defaults regardless of what ran before it. That is the point: the file behaves the same on your bench as on a robot someone else used last. It does **not** move the arm — the MCS500 never needs homing, which is also why there is no `Home()` in the block.

`01_basic/01_first_program.mxprog` explains all four commands line by line and is the one to run first.

Two things this block does not do:

- It does not reset **persistent** settings, or the ones only a **reboot** clears — see [Three kinds of setting](#three-kinds-of-setting). Use `00_common/00_cold_start.mxprog` for those.
- It does not touch anything outside the robot. A tool driven by your PLC keeps whatever state the PLC left it in.

`00_common/01_init.mxprog` deliberately has **no** reset block — it is called from inside another program's flow, and a deactivate there would tear down the state its caller just built.

<details>
<summary><b>To remove the reset block from every example</b></summary>

The block is byte-identical in all of them and always ends with `ResumeMotion()`. Delete from the `/* --- Standard reset` line through `ResumeMotion()` — one find-and-replace across the repo, or:

```bash
perl -0pi -e 's{/\* --- Standard reset.*?\nResumeMotion\(\)\n\n}{}s' */*.mxprog
```

That leaves `01_basic/01` untouched, which is correct — its copy is the teaching one, explained command by command, and is not the shared block.

</details>

**Header block** — every example opens with a block comment stating what it is for, what it needs, and what to watch:

```
/* ============================================================
   02 — MoveJoints: the joint-space move
   ------------------------------------------------------------
   ROBOT     MCS500 only (R1)
   TEACHES   What a joint move is, what the four numbers mean
   NEEDS     Robot activated, no tool mounted
   WATCH     All four joints start together and stop together
   ============================================================ */
```

**Speeds** — examples run slow on purpose (typically 10–30%) so the motion is legible and safe
to stand next to. Every example says where to change the speed.

**Return to a known position** — examples end where they started, at the zero position, so you
can run one after another, or the same one twice, without re-jogging the robot.

**Reachability** — all working poses sit in front of the base, with the shaft between about 120
and 200 mm from the base axis — well inside the arm's 225 mm and away from the singularity —
except where a file visits the singularity on purpose. Every pose was checked against the
geometry and joint limits above with a kinematic model; see [DISCLAIMER.md](DISCLAIMER.md#what-is-not-simplified).

**The house tool** — from `02_intermediate/02` on, examples that use a tool define it as
`SetTrf(0, 0, -30, 0)`: a tool on the shaft axis, its tip 30 mm below the end of the shaft.
The **z is negative** because every frame points up and tools hang down. With no tool fitted
these examples are still safe — the bare shaft simply runs 30 mm higher than the poses say.

---

## Coming from the Meca500?

If you know the [Meca500 examples](https://github.com/maximeriera/meca500_mxprg_exemple), this repo has the same structure, the same conventions and largely the same lessons. What changes:

| | Meca500 | MCS500 |
|---|---|---|
| Joints | six rotary | three rotary, **one linear (mm)** |
| `MoveJoints`, `SetTorqueLimits` | six values | four values |
| Poses, `SetTrf`, `SetWrf` | `x, y, z, α, β, γ` | `x, y, z, γ` — z always up |
| Tool frame z | usually out of the tool | **up**, so a hanging tool has negative z |
| Homing | `Home()` after activation | none — absolute encoders (`Home()` is accepted and ignored) |
| Posture | `SetConf(cs, ce, cw)`, up to eight | `SetConf(ce)`, lefty or righty |
| Last joint | θ6, `SetConfTurn` −100 … 100 | θ4, `SetConfTurn` −10 … 10 |
| Singularities | wrist, elbow, shoulder | elbow only — and `MoveLin` can never cross it |
| Pick and place | `MovePose` + `MoveLin` approach and retract | `MoveJump` |
| Tool | MEGP 25* gripper commands | driven by the host |
| Persistent settings | thirteen | fourteen — adds `SetPStop2Cfg`, `SwitchToEtherCat`; no `SetOfflineProgramLoop` |

---

## Folder structure

Examples are organised by **level**, and numbered so the reading order is the teaching order.

```
00_common/         Reusable helpers, not a level. init and cold_start.
01_basic/          What the robot does. One concept per file, nothing assumed.
02_intermediate/   How to make it useful. Frames, configurations, MoveJump.
03_advanced/       How it goes into a real cell. Composition, variables, safety.
```
Start with `00_common/` for configuration knowledge.
Read `01_basic/` in order and you have the vocabulary.
Read `02_intermediate/` and you can build something useful.
Read `03_advanced/` and you can design a cell and answer an integrator's questions about it.

---

## The three setup programs

**`01_basic/01_first_program.mxprog`** takes the robot from powered-on to ready for motion: `DeactivateRobot()`, `ActivateRobot()`, `ResetError()`, `ResumeMotion()`. The last three are idempotent — none fails if the robot is already in the state it asks for.

**`00_common/01_init.mxprog`** puts the *settings* in a known state: frames, posture selection, speeds, MoveJump shape, blending, move mode, payload, torque limits. Save it on the robot as `00_common/01_init`, then open your own programs with:

```
StartProgram("00_common/01_init")
```

**`00_common/00_cold_start.mxprog`** is for a robot you do not know — a loaner, a shared bench, anything someone else used last. It **audits** the persistent settings, clears the reboot-only ones, then deactivates and reactivates to wipe every non-persistent one back to firmware defaults.

| | Scope | When |
|---|---|---|
| `01_basic/01_first_program` | Robot **state** — activate, clear fault | Once per power-up |
| `00_common/01_init` | **Non-persistent settings** — frames, speeds, blending | Start of every program |
| `00_common/00_cold_start` | All of the above, **plus** reboot-only settings and an audit of persistent ones | On an unfamiliar robot |

Settings are sticky, and they only reset **on activation**. A robot that stays activated all day never gets that reset, so between two runs the only thing that restores them is you — either explicitly (`init`) or by cycling activation (`cold_start`).

---

## Three kinds of setting

This is the distinction that decides what a reset can actually fix, and it is worth knowing before you debug a robot that is "behaving strangely".

**Non-persistent** settings live only as long as the robot is activated: TRF, WRF, speeds, blending, payload, torque limits, posture and turn selection, move mode, MoveJump heights. Deactivating loses all of them — which is what makes the standard reset block work.

**Reboot-only** settings survive a deactivation and are only reset by a power cycle. Two matter for motion: **time scaling** — the slider at the bottom of the MecaPortal's program panel, which slows every move *and* every `Delay` — and **recovery mode**, which relaxes limits and caps speed. A robot left at 30% time scaling runs your demo at a third of its speed, and no reset block will tell you. `cold_start` sets both back.

**Persistent** settings are written to the robot's SD card and survive deactivation, reboots and power cycles. Fourteen commands set them — the manual calls them *configuration commands* — and **deactivating does nothing to them**. The ones that change how motion behaves:

| Command | Factory default | Why it bites |
|---|---|---|
| `SetJointLimits` / `SetJointLimitsCfg` | user limits **disabled** | a restricted joint changes every program, and a joint parked outside its limits stops the robot activating |
| `SetWorkZoneLimits` / `SetWorkZoneCfg` | ±10,000 mm; `(4, 1)` | moves get refused for no visible reason |
| `SetToolSphere` | `(0,0,0,0)`, disabled | affects work-zone and collision checks |
| `SetCollisionCfg` | `4` | changes what a near-collision does |
| `SetPStop2Cfg` | `3`, clear motion | decides whether a P-Stop 2 keeps or clears the motion queue |
| `SetCalibrationCfg` | `1`, enabled | disabled means a calibrated robot's Cartesian accuracy quietly drops |
| `SetSimModeCfg` | `1`, normal | decides which mode `ActivateSim()` picks |

The table above covers nine of the fourteen. The remaining five are `SetRobotName`, `SetNetworkOptions`, `EnableEtherNetIp`, `EnableProfinet` and `SwitchToEtherCat` — the robot's identity and how it is wired into your network. `cold_start` deliberately leaves all five alone: resetting them would take the robot off the network.

**Every one of the nine in the table can only be set while the robot is deactivated** — a running program must not be able to widen its own safety envelope. That restriction is also the opportunity: the deactivate step `cold_start` needs anyway is exactly the window in which they can be corrected.

> [!WARNING]
> `cold_start` leaves the factory-reset lines **commented out on purpose**.
> On a bare demo robot, uncomment them. On a robot in a working cell, several of them may be a safety envelope somebody
> configured deliberately — run the audit, look at the output, and decide before you change anything.

---

## File index

**`00_common/` — helpers**

| File | Purpose |
|---|---|
| `00_cold_start.mxprog` | Save as `00_common/00_cold_start`; settings audit, reboot-only reset, then a deactivate/reactivate reset |
| `01_init.mxprog` | Save as `00_common/01_init`; call with `StartProgram("00_common/01_init")` at the top of every program |

**`01_basic/` — the vocabulary**

| File | Teaches |
|---|---|
| `01_first_program.mxprog` | Readiness: `DeactivateRobot`, `ActivateRobot`, `ResetError`, `ResumeMotion` — why there is no `Home`, program anatomy, and the motion queue |
| `02_move_joints.mxprog` | Joint-space motion — `MoveJoints`, and a joint measured in millimetres |
| `03_move_pose.mxprog` | Cartesian target, joint-space path — `MovePose`, and why up-and-down is one joint |
| `04_move_lin.mxprog` | Straight-line motion — `MoveLin`, and why it is *not* the default |
| `05_joint_motion_settings.mxprog` | `SetJointVel`, `SetJointAcc`, `SetJointVelLimit` — tuning in percentages |
| `06_linear_motion_settings.mxprog` | `SetCartLinVel`, `SetCartAngVel`, `SetCartAcc` — tuning in mm/s |
| `07_delay_and_sequencing.mxprog` | `Delay`, `SetCheckpoint`, and what "finished" actually means |

**`02_intermediate/` — making it useful**

| File | Teaches |
|---|---|
| `01_world_reference_frame.mxprog` | `SetWrf` — moving the origin, not the program |
| `02_tool_reference_frame.mxprog` | `SetTrf` — telling the robot where the tool tip is, and the negative z |
| `03_relative_moves.mxprog` | `MoveLinRelTrf` / `MoveLinRelWrf` / `MoveJointsRel` — approach, slide, retract |
| `04_posture_configuration.mxprog` | `SetConf`, `SetAutoConf` — lefty and righty |
| `05_turn_configuration.mxprog` | `SetConfTurn` — joint 4, and why cables fail |
| `06_blending.mxprog` | `SetBlending` — cycle time you get for free, and the moves that never blend |
| `07_move_jump.mxprog` | `MoveJump`, `SetMoveJumpHeight`, `SetMoveJumpApproachVel` — the SCARA's own move |

**`03_advanced/` — into a real cell**

| File | Teaches |
|---|---|
| `01_variables.mxprog` | Robot variables (beta): `CreateVariable`, `vars.` and `*vars.` |
| `02_program_calls.mxprog` | `StartProgram` — composition, and how to architect a cell |
| `03_pick_and_place.mxprog` | A full cycle, assembled from everything above — and the host's half of it |
| `04_work_zone_and_collision.mxprog` | `SetWorkZoneLimits`, `SetToolSphere`, `SetCollisionCfg` |
| `05_payload_and_torque_limits.mxprog` | `SetPayload`, `SetTorqueLimits` — accuracy and process guarding |
| `06_time_based_motion.mxprog` | `SetMoveMode`, `SetMoveDuration` — moving on a fixed beat |
| `07_singularities.mxprog` | The elbow singularity — at it, near it, and why a line cannot cross it |

Two files are mostly prose, because the commands they cover cannot go in a running program: `03_advanced/04_work_zone_and_collision` (work zone and collision settings require the robot to be **deactivated**) and parts of `03_advanced/02_program_calls` (which needs sub-programs saved on the robot first). Both say so at the top and both still have a runnable section.

Every file ends with a **THINGS TO TRY** block — two or three concrete exercises with the robot, and the question that feature answers. They are the fastest way to make a concept stick, whether you are learning the robot or showing it to someone else.

---

## Running an example

1. Open the MecaPortal and connect to the robot **in control mode** — you cannot save or run programs from monitoring mode.
2. Load the `.mxprog` file into the code editor with the *load from computer* icon.
3. **Read the header block.** Check the preconditions match your setup, and that the poses are clear of anything you have mounted.
4. Press **run** to execute the whole program, or select a section and press `Ctrl` + `Enter` to run only those lines. Running a file a few lines at a time the first time is the best way to read it.

You do not need to activate first: every example does that itself.

To keep a program on the robot, save it. Program and folder names are case-sensitive, max 63 characters; stick to letters, digits and underscores. A `/` in the name creates a folder, which is how `StartProgram("00_common/01_init")` finds its target.

### Run it in simulation first

Every example is written to be safe on a bare robot, but poses that are fine on one setup can collide with a fixture on another. Before running an unfamiliar program on real hardware, deactivate the robot and enable **simulation mode**:

```
ActivateSim(1)
```

The robot then executes everything — the 3D view moves, the log fills, errors and work-zone breaches are reported — with the motors doing nothing. `ActivateSim(2)` runs the same thing as fast as possible, which is the quickest way to check a long program for errors. `DeactivateSim()` returns to real motion. Both can only be issued while the robot is deactivated.

### If something goes wrong

The robot validates syntax **on save**, not as you type — a red dot on the tab and a red marker on the offending line means the robot rejected it. A motion error turns the run button red; click it (or *Reset error*) to send `ResetError`, then press resume to send `ResumeMotion`.

Common responses you will meet:

| Code | Meaning |
|---|---|
| `[1005]` | The robot is not activated |
| `[1011]` | The robot is already in error |
| `[1013]` | Activation failed — usually a stop signal, or RESET not pressed since power-up |
| `[1016]` | Destination out of reach — for any posture, for the posture you demanded, or somewhere along a `MoveLin` |
| `[1033]` | The move starts in a different posture from the one demanded with `SetConf` |
| `[3017]` | No offline program saved — `StartProgram` could not find its target |
| `[3049]` | Work zone limit — see `03_advanced/04` |

If a joint ends up outside its user-defined limits, the robot will not activate at all. `SetRecoveryMode(1)` relaxes the limits — at a heavily reduced speed — so you can jog it back in; `SetRecoveryMode(0)` puts things back.

### Editor support

The [mecademic-mxprog VS Code extension](https://github.com/maximeriera/mecademic-vscode-extension) (v0.1.0) models the **Meca500** instruction set. On these files it will report every four-value `MoveJoints`, `MovePose`, `SetTrf` and `SetWrf` as an argument-count error, and every `MoveJump` as unknown. Those are false positives: the syntax here follows the MCS500 Programming Manual. The MecaPortal, which validates on the robot itself, is the authority.

---

## Target hardware and firmware

**These examples are for the MCS500 only** (R1, including the hollow-shaft MCS500-HS, whose only differences are mechanical).

Do not run them on any other Mecademic model. They are written throughout against the MCS500's four-axis SCARA geometry: four-value joint sets with a linear third joint in millimetres, four-value poses, the joint limits listed above, single-parameter posture configuration, and the one elbow singularity that goes with that arm. None of that transfers to a six-axis arm, and a program that assumes it will either be refused or move somewhere you did not intend.

The examples assume the robot is mounted **tabletop**. The MCS500 can also be mounted upside down; if yours is, heights, approach directions and the signs of the `MoveJump` heights all change — check every file against the manual before running it.

The firmware target is **11.3**. The examples are accurate across the 11.x family; the MecaPortal UI details and the variable commands are the parts most likely to have moved if you are on something older.

---

## Status and licence

**Licence: [MIT](LICENSE.md).** Copy these programs into your own projects, modify them, ship them. Attribution is appreciated but the licence only asks that you keep the copyright notice with substantial copies.

**Disclaimer.** This is a collection of examples. It is not Mecademic product documentation, it carries no warranty, and it is not a substitute for the official manuals or for Mecademic support. Where this repo and the Programming Manual disagree, the manual is right and this repo has a bug — please report it.

Note the warranty disclaimer in the licence is not boilerplate here: these files command a real robot, and what is safe on a bare arm on a bench may not be safe in your cell. Read [Safety first](#safety-first).

**Written without a robot.** Every command, argument and default was checked against the manuals, and every pose against a kinematic model of the arm — but these files have not yet been run on an MCS500. Where behaviour could not be confirmed from the documents, the file says so. If something behaves differently from its comment, trust the robot and open an issue.

**Firmware drift.** These files were written against firmware 11.3. Commands, defaults and the MecaPortal UI change between firmware versions. If something here does not match your robot, check the manual for your firmware first.

**Found a mistake?** Open an issue. Corrections to the technical content are especially welcome — every claim here is meant to be traceable to the manual.

---

## Official Mecademic resources

Everything in this repo is derived from the documents below. They are the authority; this repo is a study aid. If the two disagree, believe the manual.

**Documentation**

| Resource | What it covers |
|---|---|
| [Programming Manual — MC-PM-MCS500](https://resources.mecademic.com/en/doc/MC-PM-MCS500/latest/) | Every command: syntax, arguments, defaults, usage restrictions. The reference these examples were checked against |
| [MecaPortal Operator Manual — MC-OM-MCS500](https://resources.mecademic.com/en/doc/MC-OM-MCS500/latest/) | The web interface: code editor, program manager, jogging, 3D view |
| [User Manual — MC-UM-MCS500](https://resources.mecademic.com/en/doc/MC-UM-MCS500/latest/) | Installation, safety wiring, specifications, dimensions, the workspace drawing, and end-effector mounting |
| [All technical resources](https://resources.mecademic.com/en/index.html) | Documentation hub, all models and firmware versions |

**Software**

| Resource | What it is |
|---|---|
| [Firmware downloads](https://resources.mecademic.com/en/firmware/index.html) | Firmware images and release notes |
| [mecademicpy](https://github.com/Mecademic/mecademicpy) | The official Python API — the natural next step once you need flow control |
| [Mecademic on GitHub](https://github.com/Mecademic) | Official drivers, APIs and integration examples |

**Support**

| Resource | What it is |
|---|---|
| [Knowledge base](https://support.mecademic.com/) | Articles, FAQs and troubleshooting |
| [mecademic.com](https://www.mecademic.com/) | Products, specifications, and how to contact Mecademic |

Documentation URLs above point at `latest`. To read the manual for a specific firmware version, replace `latest` with the version — for example `.../MC-PM-MCS500/11.1/`.
