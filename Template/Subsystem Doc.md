---
tags: [subsystem, <% tp.date.now("YYYY") %>, <% tp.file.title %>]
created: <% tp.date.now("YYYY-MM-DD") %>
description: "Subsystem documentation for <% tp.file.title %>"
---

# <% tp.file.title %>

> **Season:** <% tp.date.now("YYYY") %>
> **Last Updated:** <% tp.date.now("YYYY-MM-DD") %>

## Overview

<!-- What does this subsystem do? What is its role on the robot? -->
<% tp.file.cursor() %>

## Hardware

<!-- Physical components: motors, sensors, pneumatics, etc. -->

| Component | Model / Type | CAN ID / Port | Notes |
|---|---|---|---|
| | | | |

## Software Structure

<!-- How is this subsystem implemented in code? Describe the class structure, commands, and any notable logic. -->

### Key Classes / Files

- `SubsystemName.java` — 

### States / Modes

<!-- If the subsystem has distinct states or modes, describe them here -->

## Tunable Constants

<!-- PID values, speed limits, offsets, etc. that may need adjustment -->

| Constant | Value | Description |
|---|---|---|
| | | |

## Known Issues / Quirks

<!-- Anything future members should know — edge cases, fragile behavior, hardware gotchas -->

## Related

- [[<% tp.date.now("YYYY") %> Season Overview]]
- [[]]
