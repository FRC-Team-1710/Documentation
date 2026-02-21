---
aliases:
  - Programming Standards
  - Standards
tags:
  - Core
created: 2026-02-21
---

# Java Programming Standards

In order to keep a codebase consistent and coherent, programmers must follow a set of coding standards. Within Java, the [Oracle Code Conventions](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html) are followed. Within our WPILib environment, most of these standards are already enforced, but there are conventions such as naming standards that aren’t enforced that you will need to read up on. Overall, if you aren’t sure how to format a block of code, look elsewhere in the codebase to find a similar structure to copy.

# Codebase Standards

## Overview

Within the 2026 codebase, there are certain standards that are in place to keep the code consistent among us and with Hemlock’s Code Template. This also applies to the file structure of our project.

## General Code Standards

Within our codebase, there aren’t many standards to be particularly aware of, but we use two spaces as a tab.

## Hungarian Notation 

Hungarian notation is a set of naming conventions used in programming that adds extra readability to a codebase. To see the full list of standards, the [Wikipedia article](https://en.wikipedia.org/wiki/Hungarian_notation) goes extremely in depth into the different naming conventions that you can use. As for Team 1710 Programming, we will use only two specific standards:


-  **`m_`: Prefixing class variables especially private variables**
	- Distinguishes between parameters of functions and local variables, clarifies [scope](https://www.w3schools.com/programming/prog_scope.php) of the variable
	- Example: `m_desiredShooterRPM`

-  **`k`: Prefixing all constants**
	- Communicates that a variable is final and sometimes static especially to ensure there are no [magic numbers](https://en.wikipedia.org/wiki/Magic_number_\(programming\)) or ambiguity within usage
	- Example: `kFieldLength` or `kFIELD_LENGTH`

## Structure

Most decisions made considering the structure are our robot code are done to ensure that all related files are centralized. Furthermore, the division of the files allows readability and modularity within our code. After all, our entire codebase could technically just be one giant file.

#### Main Code Directory

The directory (folder) that holds the code: 

- **Main Code Directory: `src\main\java\frc\robot`**

Within this main directory is our entry point (`Main.java`), our robot class (`Robot.java`), and the robot subclass that controls all the subsystems (`RobotContainer.java`). 

The Robot.java file holds all the fundamental functionality of our robot. It gives us the methods for each stage of the game to handle changes in the game from the [FMS](https://fms-manual.readthedocs.io/en/latest/fms-whitepaper/fms-whitepaper.html), and gives us the flexibility for alternative robot states such as test mode and technically simulation.

We then dive a level deeper into our RobotContainer.java which essentially deconstructs the abstraction of “Robot” into “Mechanisms, Controls, States” and more. This is where we program all of our controls, instantiate our subsystems, and handle robot state functionality such as real vs. simulated robot.

#### Autonomous Mode Directory

Under the [[Coding Standards#Main Code Directory|Main Code Directory]], the autonomous directory will hold our autobuilder.

In an attempt of modularity with clarity, we have an autonomous folder that holds the functionality of our robot within the autonomous period since this is considerably different from the rest of the time within a match such as TeleOp. We have the code for our autobuilder in here to centralize that functionality.

#### Constants Directory

Under the [[Coding Standards#Main Code Directory|Main Code Directory]], the constants directory will hold files that include the constants for different subsystems or utilities.

The Constants directory is essential for centralizing all the magic numbers of the codebase. Therefore, if we need to change the height of our shooter, there is a clear place to change it in our codebase.

#### Generated Directory

Under the [[Coding Standards#Main Code Directory|Main Code Directory]], the generated directory will hold the swerve tuning constants. These are generated and not directly coded (99% of the time).

Since our swerve is from the [CTRE generation](https://v6.docs.ctr-electronics.com/en/latest/docs/tuner/tuner-swerve/index.html), we have this directory to isolate foreign code.

#### Subsystems Directory

Under the [[Coding Standards#Main Code Directory|Main Code Directory]], the subsystems directory will hold the superstructure, swerve, vision, and related subsystems code files and directories.

Deconstructing another layer of abstraction, we turn the mechanisms from the `RobotContainer.java` file to the intricate parts such as the motors or sensors integrated into a subsystem. Each subsystem will have states and basic functionality, and then superstructure abstracts these states under sub-robot states. Therefore, we can change the state of a robot and the descending states of each subsystem changes alongside it.

#### Utils Directory

Under the [[Coding Standards#Main Code Directory|Main Code Directory]], the utils directory will hold files and directories related to small packages of related functions such as Shooter Math or Vector Methods.