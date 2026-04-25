## Overview

The `Intake` subsystem manages two coordinated mechanisms: a roller assembly that draws game pieces in, and a deployment arm (referred to as the "deployment motor") that rotates the intake between a stowed position and the floor. It is built on a layered IO abstraction, meaning the same `Intake` class drives both real hardware (`IntakeIOCTRE`) and simulation (`IntakeIOSIM`) without any changes to the high-level logic.

The subsystem's three headline features are:

- **State-driven motion** — a compact `IntakeStates` enum encodes every operating mode with its own arm setpoint, roller speed, and per-state MotionMagic velocity/acceleration profile.
- **Automatic jam recovery** — debouncer-gated current and velocity checks detect roller jams and trigger a timed reversal without any driver input.
- **Deployment stuck detection** — a parallel debouncer watches for abnormally high deployment-motor current and temporarily commands the arm down to relieve the load, then returns it to its target.

---

## Architecture
*See [[Subsystem Architecture]] for more info*

```
Intake (subsystem / state machine)
  └── IntakeIO (interface)
        ├── IntakeIOCTRE  (real robot – TalonFX motors, CTRE Phoenix 6)
        └── IntakeIOSIM   (simulation – SingleJointedArmSim + SmartDashboard)
```

`Intake` never references any motor controller directly. Every hardware call goes through the `IntakeIO` interface, keeping the state-machine logic hardware-agnostic.

The subsystem also accepts two constructor dependencies injected at robot startup:

|Dependency|Type|Purpose|
|---|---|---|
|`io`|`IntakeIO`|Hardware or simulation backend|
|`consumer`|`TimesConsumer`|Callback to change the subsystem's scheduler period|
|`bumpSupplier`|`BooleanSupplier`|Override that runs the rollers in reverse at full speed|

---

## Motor Control

### Deployment Arm — `DynamicMotionMagicVoltage`

The arm is controlled with CTRE's `DynamicMotionMagicVoltage` request, which lets each state supply its own velocity and acceleration constraints at runtime rather than using a fixed profile baked into the motor configuration. This means the arm moves quickly when jostling (high velocity/acceleration) but more gently when deploying during normal intake (lower values).

```java
m_deploymentMotor.setControl(
    m_deploymentRequest
        .withPosition(angle)
        .withVelocity(velocity)
        .withAcceleration(acceleration));
```

`FOC` (Field-Oriented Control) is enabled on this request for improved torque accuracy.

**Slot 0 gains (deployment motor):**

|Gain|Value|Notes|
|---|---|---|
|kG|0.4|Arm cosine gravity compensation|
|kV|6.625|Velocity feedforward|
|kP|0.2|Proportional position error|
|kS, kA, kI, kD|0|Not tuned / not needed|
|GravityType|`Arm_Cosine`|Scales kG by cos(θ) automatically|
|StaticFeedforwardSign|`UseClosedLoopSign`|Matches kS direction to motion direction|

The sensor-to-mechanism ratio is **50:1** (gear reduction). The motor is initialized to position `0.29 rotations`, which corresponds to the fully stowed (`Up`) position. A soft stop is enforced in `setAngle`: if the measured position is below `0.0875 rotations` and the target is the `Down` setpoint, the motor is commanded to stop rather than push further into the hard stop.

### Intake Rollers — `VoltageOut`

The rollers use open-loop `VoltageOut` control. The speed value from `IntakeStates` (a normalized `[-1.0, 1.0]` value) is scaled to volts:

```java
m_intakeMotor.setControl(m_intakeRequest.withOutput(speed * 12));
```

A secondary follower motor (`m_intakeMotorFollower`) is configured in opposition to the leader via CTRE's `Follower` control with `MotorAlignmentValue.Opposed`. An open-loop voltage ramp of `0.0625 s` is applied to both roller motors to limit inrush current.

---

## State Machine
*For more information, see [[Finite State Machine]]*

The active state is stored in `m_currentState` and changed via `setState(IntakeStates)`. Each state bundles all the parameters needed for one operating mode:

|State|Arm Setpoint|Roller Speed|Velocity|Acceleration|Period|
|---|---|---|---|---|---|
|`Up`|0.29 rot|0|1|0.5|60 ms|
|`UpAndIntake`|0.29 rot|1.0 (in)|1|0.5|60 ms|
|`Half`|0.15 rot|1.0 (in)|1.25|0.75|60 ms|
|`Down`|0° (0 rot)|0|1|0.5|60 ms|
|`Jostle`|0.05 rot ↔ 0.125 rot|1.0 (in)|1.5|1.5|20 ms|
|`Jammed`|0° (0 rot)|−0.3 (out)|1|0.5|20 ms|
|`Intaking`|0° (0 rot)|1.0 (in)|1|0.5|20 ms|

States with a 20 ms period (`Jostle`, `Jammed`, `Intaking`) run the subsystem's `periodic()` faster than idle states. When `setState` is called with a state whose period differs from the current state's, it notifies the `TimesConsumer` to reschedule the subsystem accordingly.

### Bump Override

At the top of every `periodic()` call, `m_bumpSupplier` is checked first. If it returns `true`, the rollers are driven at `−1.0` (full reverse) regardless of the current state, bypassing all jam logic. This is intended for clearing fuel from the intake when going over the bump.

---

## Jam Detection & Recovery

Jam detection is only active in the `Intaking` state. It uses three `Debouncer` objects to gate entry into, persistence of, and exit from the jammed condition:

```
m_minimumJamTime   — must elapse before jam detection activates (prevents false triggers on startup)
m_jamTime          — how long the jam condition must persist before recovery begins
m_jamUndoTime      — how long the reversal runs before resuming normal intake
```

The jam condition itself (`isJammed()`) is:

```java
rollerCurrent >= kJamCurrent  &&  rollerVelocity <= kJamSpeedThreshold
```

High current combined with low velocity is a reliable indicator that something is blocking the rollers.

**Recovery sequence:**

1. `m_minimumJamTime` has elapsed (detection enabled).
2. `isJammed()` remains true for `kJamMinimumTime` → `m_wasJammed = true`.
3. The rollers are commanded to `Jammed.rollerSpeed` (−0.3) for `kJamUndoTime`.
4. After `kJamUndoTime` elapses, the debouncers are reset and normal intake speed resumes.

A `m_wasJammed` flag prevents the condition from flickering: once jammed is declared, the reversal continues until the undo timer completes even if the jam clears momentarily.

---

## Deployment Stuck Detection

When the intake is in the `Up` state (returning to stow), the deployment motor current is monitored to detect cases where the arm is mechanically obstructed. The logic mirrors the roller jam pattern using its own set of debouncers:

```
m_minimumStuckTime  — settling time before stuck detection activates
m_stuckTime         — how long high current must persist to declare "stuck"
m_stuckUndoTime     — how long the arm is driven down before retrying the stow
```

**Recovery sequence:**

1. `m_minimumStuckTime` has elapsed.
2. `isStuck()` (`deploymentCurrent >= kStuckCurrent`) remains true for `kStuckMinimumTime` → `m_wasStuck = true`.
3. The arm is commanded to `Down.setpoint` (0°) for `kStuckUndoTime` — moving away from the obstruction.
4. After `kStuckUndoTime`, debouncers reset and the arm re-attempts the `Up` setpoint.

Stuck detection is only active in the `Up` state; all other deployment states clear all three stuck debouncers on every tick.

---

## Jostle Behavior

The `Jostle` state oscillates the arm between two angles (`setpoint = 0.05 rot`, `m_alternate = 0.125 rot`) push fuel into the feeder. The transition between the two positions is triggered by reading the MotionMagic reference slope from the deployment motor:

```java
if (m_io.getSetpointReferenceVelocityIsZero()) {
    m_wasUp = !m_wasUp;
}
```

`getSetpointReferenceVelocityIsZero()` returns `true` when the closed-loop reference slope reaches zero — i.e., the MotionMagic profile has finished its current move and the arm is settled. At that point, `m_wasUp` is toggled to target the opposite angle. This produces a natural oscillation that is self-pacing: the arm only reverses once it has actually reached each waypoint.

The Jostle state uses higher velocity (1.5) and acceleration (1.5) than normal states to snap between positions quickly, and runs at the 20 ms scheduler period for responsiveness.

---

## Signal Refresh

`IntakeIOCTRE` maintains three cached `BaseStatusSignal` arrays (intake motor, follower, deployment) plus one dedicated signal for the deployment motor's closed-loop reference slope. All are configured at **50 Hz** and refreshed together at the start of each `update()` call:

```java
BaseStatusSignal.refreshAll(m_intakeSignals);
BaseStatusSignal.refreshAll(m_intakeFollowerSignals);
BaseStatusSignal.refreshAll(m_deploymentSetpointVelocitySignal);
BaseStatusSignal.refreshAll(m_deploymentSignals);
```

`optimizeBusUtilization()` is called on all three TalonFX devices to suppress signals that are not explicitly requested, reducing CAN bus load.

---

## Simulation
*See [[Subsystem Architecture#Simulation|simulation]] for more info*

`IntakeIOSIM` provides a physics-backed arm simulation using WPILib's `SingleJointedArmSim` with a `ProfiledPIDController` (kP = 5) tracking the angle setpoint. Roller behavior is visual only; no physics are simulated for the rollers.

Jam and stuck testing is enabled via SmartDashboard sliders that feed `getRollerVelocity()`, `getRollerCurrent()`, and `getDeploymentCurrent()` — allowing the full jam and stuck recovery sequences to be triggered without real hardware.

The `IntakeVisualSim` helper renders the arm angle and roller spin in a `Mechanism2d` widget on SmartDashboard.

> **Note:** `IntakeIOSIM.setAngle` does not accept the `velocity` and `acceleration` parameters present in the `IntakeIO` interface — the simulation uses a fixed `ProfiledPIDController` with constant constraints regardless of the active state.

---

## Logging

The subsystem and its IO layer are annotated with `@Logged` (WPILib Epilogue). Key signals and their logging importance:

|Field|Importance|
|---|---|
|`m_io` (IO layer)|CRITICAL|
|`m_currentState`|INFO|
|`m_wasJammed`|INFO|
|`m_wasStuck`|INFO|
|`m_wasUp`|CRITICAL|
|`isJammed()`|CRITICAL|
|`isStuck()`|CRITICAL|
|Roller/follower current & velocity|DEBUG|
|Deployment current|DEBUG|
|`m_angleSetpoint` (CTRE)|INFO|

Fields marked `@NotLogged` include the debouncers, the `BooleanSupplier`, the motor request objects, and the `TimesConsumer`.

---

## Testing Mode

Calling `setTesting(true)` locks out `setState()` so that automated command calls cannot override the state during on-robot testing. In this mode, only `setStateTesting()` can change the active state, allowing direct control from a test dashboard or SysId routine without interference from the command scheduler.