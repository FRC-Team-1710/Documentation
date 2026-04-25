# Software Stack

| Tool / Library | Version   | Purpose                                                                                                        |
| -------------- | --------- | -------------------------------------------------------------------------------------------------------------- |
| WPILib         | v2026.2.1 | Core robot framework                                                                                           |
| BLine          | v0.8.3    | Odometry-based path following library                                                                          |
| PhotonVision   | v2026.3.4 | Vision processing software running on the coprocessors                                                         |
| PhotonLib      | v2026.3.2 | Library that allows the RoboRio to interface with the cameras and perform calculations such as pose estimation |
# Overview
We have our robot code split up into [[#subsystems]], [[#vision]], and [[#autos]]. 
# Subsystems

## Overview
All subsystems use a finite state machine (FSM). The state of the robot and each subsystem is stored in an `enum`. The `RobotContainer` file takes control inputs and directs the `Superstructure` to handle state transitions. Each subsystem then derives its own state from the robot's current state. Each loop iteration, the subsystem sets its power and/or angle based on the arguments defined in that state.

*See [[Finite State Machine]] and [[Subsystem Architecture]] for more info*
## [[Intake]]
The intake subsystem uses a `DynamicMotionMagicVoltage` request to control arm movement and a `VoltageOut` request for the intake rollers. It uses automatic jam detection to reverse jams without driver input. 
## [[Indexer]]
The indexer subsystem has changed roles over the season. It originally controlled the spindexer, but now controls the rollers on the floor of the hopper. The indexer is controlled directly via a power output.
## [[Feeder]]
The feeder feeds the fuel fast into the flywheel (try saying that 5 times fast). It directly controls the feeder using raw power.
## [[Shooter]]
The robot has a drum shooter. We use a `InterpolatingDoubleTreeMap` to get data from in between preexisting points. We lookup the `targetVelocity` for the flywheel and the `hoodTarget` angle.
## [[LEDs]]
The LED code runs on an Arduino. Currently, it communicates to the RoboRio using SPI but it can also be connected via USB and other methods. It sends over a byte of data containing details about the state of the robot using the LED subsystem. Then, the Arduino will interpret the signals and use the respective pattern.
## [[Vision]]
We use 4 cameras connected to 3 Orange Pi 5s running [PhotonVision](https://docs.photonvision.org/en/latest/index.html) for vision processing. This year we chose not to use object detection on our final design.
## [[Autos]]
This year our team moved away from PathPlanner in favor of BLine. Timed-based autos proved unreliable when driving over the bump, so we needed a position-based pathfinding solution.
## [[Logging]]
This year we switched from AdvantageKit to Epilogue. We made this change because we did not need to log controller inputs and wanted the flexibility to log custom classes.
# New

## [[DynamicTimedRobot]]
Instead of using the default `TimedRobot`, we now use a `DynamicTimedRobot` to schedule all of our subsystems. This allows each subsystem to run at a different frequency based on its importance.
## [[Documentation]]
This year we made two major improvements to our documentation. We expanded our us of Javadocs throughout the codebase, which lets our members view parameter descriptions and method explanations directly on hover. We also expanded this documentation site, written in Markdown using Obsidian, stored in GitHub, and published via Quartz.