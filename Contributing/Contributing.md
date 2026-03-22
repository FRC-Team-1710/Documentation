---
aliases:
  - Contributing
  - Meta
tags:
  - General
  - meta
  - guide
editors: Collin Arthur
title: Contributing
---
# How to Contribute

Welcome! If you are here to contribute to the docs, this is a simple guide to get your workflow setup. 

## Context

For some context, this guide is built using a mix of [Quartz](https://quartz.jzhao.xyz/), [Obsidian](https://obsidian.md/), and [GitHub Actions](https://github.com/features/actions). The documentation is written within Obsidian, managed by version control, and then automatically updates the Quartz documentation website on push.

## Setup

1. Install [Obsidian](https://obsidian.md/) 
2. Install [Git](https://git-scm.com/install/)
3. Open the folder that will hold your obsidian vaults in file explorer
4. In the path bar, type "cmd" and hit enter to open up a terminal
 ![[Pasted image 20260322124806.png]]
 ![[Pasted image 20260322124838.png]]
 5. Paste in this command: `git clone https://github.com/FRC-Team-1710/Documentation.git`
 6. Open Obsidian and open the folder created by git as a vault
 7. Install Obsidian Plugins (**Make sure to enable after install**)
	1. Git
	2. Linter
	3. Templater
8. Template setup
	1. Go `Settings > Templates` and set the template folder to "Template"
	2. Go `Settings > Templater` and set the template folder to "Template"
	3. Set `Trigger Templater on new file creation` to true
	4. Enter all fields below ![[Pasted image 20260322125803.png]]
9. Go `Settings > Linter` and set `Lint on save` to true
Setup done!

## How to contribute?

To contribute, make sure that you have a clear goal in mind with the CTO's permission and feedback. After that, just like when you program, always fetch before you start working using the Git plugin (`Ctrl + P` to `Git: fetch`). No branches need to be made since it is documentation and not code that will impact our team's performance (hopefully), so feel free to commit and push directly to main.