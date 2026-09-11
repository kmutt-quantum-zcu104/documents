# KMUTT Quantum Hardware documents

Shared documentation for research and engineering work using the quantum lab's ZCU104 at King Mongkut's University of Technology Thonburi (KMUTT).

The board is shared across projects. We want current and future students to be able to understand the setup, reproduce useful work, and recover a documented working configuration without relying on the original authors being around.

## What's here

This repository is for shareable board setup, build, recovery, and engineering guides. Documentation tied to a particular implementation should stay with its source code and be linked here when useful.

A private `testbed-meta` repository is planned for lab-specific access information, current configuration records, and maintenance notes. See the [proposal in issue #3](https://github.com/kmutt-quantum-zcu104/documents/issues/3). Reusable procedures should live here rather than be duplicated in public and private guides.

This repository is public. Keep confidential lab information out of documents, logs, and screenshots. Passwords, tokens, and private keys do not belong in any Git repository, including a private one.

## Getting started

The documentation is being built incrementally. The first guides are under review:

* [Development environment and Git workflow — PR #2](https://github.com/kmutt-quantum-zcu104/documents/pull/2): Proposed development setup and collaboration practices.
* [Yocto build guide — PR #1](https://github.com/kmutt-quantum-zcu104/documents/pull/1): Building a Linux image for the ZCU104.

These are works in progress, not a complete onboarding or recovery procedure. Proposed standards are still being discussed. Check a guide's prerequisites, tested configuration, and limitations before following it; if something is missing or unclear, please ask.

The technology stack may change. Versions recorded for a tested procedure describe that setup, not a permanent requirement for every project.

Before using or changing the shared board, coordinate with the other users and confirm the applicable lab safety requirements. Access, scheduling, and handover procedures are not documented here yet.

## Prerequisites and references

These guides focus on the testbed rather than teaching FPGA, Linux, or Git basics. Depending on the task, you will need familiarity with digital hardware and the Zynq processing system (PS) and programmable logic (PL), Linux command-line tools, or Git.

Useful references:

* [ZCU104 Evaluation Board User Guide (UG1267)](https://docs.amd.com/r/en-US/ug1267-zcu104-eval-bd): Board components, interfaces, and configuration reference.
* [The Linux command line for beginners](https://ubuntu.com/tutorials/command-line-for-beginners): A starting point if you are unfamiliar with the shell.
* [Pro Git](https://git-scm.com/book/en/v2): Git fundamentals; the first three chapters cover getting started, basic usage, and branching.

Use the requirements and version-matched references for the particular guide or project you are working on. You do not need to install every tool just to read or contribute to the documentation.

## Questions and contributions

If something is unclear or broken, please [open an issue](https://github.com/kmutt-quantum-zcu104/documents/issues). Link the relevant guide and describe what you expected and what happened, including relevant versions or errors when useful. Remove sensitive information before posting.

Small corrections and focused pull requests are welcome. For larger changes, discuss the idea in an issue first so we can avoid duplicating work. Keep descriptions brief; for technical procedures, explain what was tested and what remains unverified. If a decision is made outside GitHub, leave a short summary on the relevant issue or pull request.
