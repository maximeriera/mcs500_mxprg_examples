# Disclaimer: what these examples simplify, and why

## Who these examples are written for

Someone with **little or no background in robotics or industrial automation** — a newcomer, a
student, an engineer from another discipline, someone evaluating an MCS500 for the first time.

The goal is that you can read a file top to bottom and come away able to *use* the feature and
*explain* it to somebody else. That goal shapes every explanation in this repo, and it
sometimes conflicts with being exhaustively precise.

**Where those two conflict, this repo chooses the explanation that leaves you with the right
instinct, over the one that is technically complete.**

That is a real trade-off and it deserves to be stated openly rather than discovered. This
document is the list of places where it happens.

## If you already know robotics

You will notice things here that are not quite right. Most of them are on the list below, and
they are deliberate. Reading this file first will save you the irritation.

The precise version of everything here is in the
[Programming Manual](https://resources.mecademic.com/en/doc/MC-PM-MCS500/latest/). It is well
written and unusually direct — if a simplification below matters to your application, go and
read the real thing. Nothing in this repo is a substitute for it.

---

## The simplifications

### Motion and paths

**"The tool tip travels a curve."**
Said of `MoveJoints`, `MovePose` and the travel in the middle of a `MoveJump`. The real path is
whatever falls out of moving the joints linearly in joint space. When only joint 1 moves it
happens to be a circular arc about the base; in general it is not any simple shape, and it is
three-dimensional when the shaft moves too. "A curve" is used because the thing that matters
to a beginner is *it is not a straight line*.

**"`SetJointVel` is a percentage of each joint's top speed."**
It is a percentage of the **top rated joint velocities**, and the speed actually reached is
also bounded by `SetJointVelLimit`, by acceleration on short moves, and by the requirement that
every joint arrives at the same time. `SetJointVel(50)` is a request, not a guarantee that
any joint ever reaches half its top speed.

**"Blending at 100% gives the roundest corner."**
Blending percentage is not a corner radius. 100% means blending occupies the whole of the
acceleration and deceleration phases. Because those phases depend on your speeds and
accelerations — and on `SetJointVelLimit`, which shortens them — **the same blending percentage
produces a different-sized corner at different speeds**. Treat the number as a dial to tune by
eye, not a geometric specification.

**The motion queue as a simple FIFO.**
Commands are described as going into a queue and being executed one at a time in order. That
is the observable behaviour and the right mental model. Internally the controller looks ahead
across queued commands — that is how blending can start a corner before the previous move has
finished at all.

**`MoveJump` as "lift, travel, land".**
The three-phase picture is the right one, and the vertical parts really are straight. What it
glosses over: the travel phase is a joint-space move whose highest point can fall anywhere
along it, the speed caps of `SetMoveJumpApproachVel` are ignored when they exceed what
`SetJointVel` produces anyway, and changing `SetJointVel` changes the shape of the hop, not just
its speed. `02_intermediate/07` mentions the first; the manual covers all three.

### Frames and configuration

**"The TCP is the tool tip."**
The TCP is the origin of the Tool Reference Frame, and it is wherever you define it with
`SetTrf`. Usually you put it at the physical tip, because that is the useful place. Nothing
requires it: for some applications it belongs at the centre of a gripped part, or at a
camera's optical axis.

**"There are two ways to reach a pose."**
Two postures is the general case, and the number to remember. It is not universal: on the
singular circle the two collapse into one, and behind the base, out to each side, only one of
them is reachable because joint 1 runs out of range. On top of that, every posture comes with
many possible turns of joint 4. The examples say "usually two" where it matters.

**"γ is how far the tool is turned."**
That is the right way to read it, and on a SCARA it is honestly most of the story — one angle
instead of three. What is glossed over: γ and γ ± 360° describe the same orientation, and
the turn of joint 4 that produced it is a separate fact (`GetRtConfTurn`). If you compare poses
numerically in host code, compare angles modulo 360, and do not assume the number you read
back is the number you sent.

**The singular circle as "225 mm from the base".**
That is exact for the axis of the shaft — the singularity is purely θ2 = 0. It is *not* a
statement about the tool tip: with an off-axis TRF, the tip can be anywhere when the arm is
straight. `03_advanced/07` says so; other files say "225 mm" for short.

**"Keep the elbow bent by at least 30 degrees."**
A rule of thumb from this repo, not from the manual. The manual says to avoid singularities in
production and explains why paths near them are slow. Where exactly "near" begins depends on
your speeds and on `SetJointVelLimit`; 30 degrees is a comfortable, round margin, not a
specification.

**Accuracy and repeatability are never distinguished.**
This is the single biggest omission in the repo, and it is the most common confusion among
people new to robots. Briefly:

- **Repeatability** — how closely the robot returns to the *same* point it went to before. This
  is the headline number on a datasheet, and it is very good on an MCS500.
- **Accuracy** — how closely the robot reaches the point you *asked for* in absolute
  coordinates. This is a different, larger, and harder number.

Most applications are repeatability-limited, which is why teaching points by jogging works so
well. Applications driven by a CAD model or a camera are accuracy-limited. If you are quoting a
number to anyone, know which one you are quoting.

### State and settings

**"Deactivating does not move the arm."**
Correct in the sense that matters: there is no commanded motion, and no homing on reactivation.
Deactivating does engage the brakes and de-energise the motors, which is a physical event — and
on a SCARA the shaft is vertical, so joint 3 is holding the tool's weight against gravity when
it happens. Read the claim as *"no commanded motion"*, not *"perfectly rigid"*, and do not
deactivate with the tool pressed against something.

**Settings are described as simply "sticky".**
The full picture is the three-way split documented in the
[README](README.md#three-kinds-of-setting) — non-persistent, reboot-only and persistent.
Individual example files mostly say "this survives until something changes it" and leave the
detail there.

### Architecture

**The host/robot split is presented cleanly.**
"Logic in the host, motion in the robot" is the right architecture and genuinely why the robot
has no `IF`. On the MCS500 the host owns the tool too, which makes the split even cleaner — and
makes its **cost** more visible: every decision and every tool action between two moves is a
round-trip to the host, and a cycle that consults the host frequently will be slower than one
that does not. Designing to keep decisions out of the inner loop is real work. The examples show
the pattern; they do not show the tuning.

**"A `Delay` is a guess, a checkpoint is a fact."**
A fair summary, and the right instinct. To be exact: a checkpoint tells you the robot's motion
queue *reached that point* — and with blending on, it is reported somewhere along the blended
corner, not at a dead stop. That is what you want and it is genuinely reliable. Coordinating
with equipment outside the robot still needs the host to do the handshake — the checkpoint is
one half of it.

---

## What is *not* simplified

Some claims in this repo were deliberately kept precise even where a simpler line would have
read better. Do not treat these as approximations:

- **The work zone and collision features are not certified safety functions.** They prevent
  mistakes and protect equipment. They are not a substitute for a risk assessment, guarding, or
  the robot's actual safety signals. `03_advanced/04` says so, and it means it.
- **Torque limiting is a process guard, not a safety rating.** `03_advanced/05` says so.
- **Robot variables are a beta feature.** `03_advanced/01` says so, because someone designing a
  machine around them deserves to know.
- **`MoveLin` cannot cross the singularity.** Not "should not" — the manual calls it physically
  impossible, because crossing means changing posture. `03_advanced/07` demonstrates it.
- **Every command signature, argument range, default value and usage restriction** was checked
  against the MCS500 Programming Manual for firmware 11.3. Where a file states a default or a
  limit, that is meant to be exact. If one is wrong, it is a bug — please report it.
- **Every pose in every file** was checked against a kinematic model built from the user
  manual's dimensions — 100 mm and 125 mm links, a 102 mm stroke, the joint limits — for
  reachability in the posture the file demands, and every `MoveLin` along its whole length.
  That is a check against the documents, not against a robot: these files were written without
  one to hand. Behaviour that could not be confirmed from the documents is flagged in the file
  that relies on it.

---

## Found something wrong?

Two different things, and it helps to say which you think you have found:

- **A simplification** — something on the list above, or something that should be. If it misled
  you, that is worth knowing even though it was deliberate; the trade-off may have been called
  wrong.
- **An error** — a command that does not exist, an argument in the wrong order, a wrong default,
  a pose that cannot be reached, a claim the manual contradicts. These are bugs and there is no
  defence for them.

Open an issue either way. Corrections to the technical content are especially welcome.

Where this repo and the Programming Manual disagree, **the manual is right**.
