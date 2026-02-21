---
tags:
  - Core
created: 2026-02-21
---
# Git Workflow

### Commit Standards

Each commit message must represent the general summary of the code changes. A good way to approach this is to think, can you put “This commit will” in front of your commit message? I also recommend signing up for the [GitHub Student Developer Pack](http://education.github.com/pack) to gain access to Copilot which can create commit messages for you with the click of a button.

### Branch Standards

Branches will be divided up into three different categories:

**Competition Branches** (`*`)
- These will be used during competitions as the pseudo-main branch

**Robot Feature Branches** (`feature/*`)
- These branches will implement one feature for the robot
- Generally, they majorly affect only one file or a group of related files

**Bugfix Branches** (`fix/*`)
- These branches fix a single bug within the code
- Generally, these branches only affect one file at most
- They can also bypass merge restrictions during competition

### Pull Request Standards

To protect the main branch from faulty code or unpolished code, there are pull request standards set in place:

- Commits cannot be made directly to main
- Two collaborators must approve a pull request
- The robot code must build  according to our [CI GitHub Action](https://github.com/FRC-Team-1710/2026-Robot/actions/workflows/main.yml)
- The `main` branch cannot be deleted

Keep in mind that this includes our Spotless integration in the CI GitHub Action, so the code must also be formatted properly for the code to build.


[Back to index](https://programming.team1710.com/)