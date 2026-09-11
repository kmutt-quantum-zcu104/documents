# Development environment

This guide helps you identify what you need for a particular task. It is not a fixed toolchain for every project. Start with the project's README or the relevant board guide for tested versions and setup steps; if those details are missing, please ask before installing tools or changing the board.

Depending on the task, you will need familiarity with the Linux command line, Git, digital hardware, or the programming language used by the project. You do not need every tool below just to read or edit documentation.

## Operating systems

Choose a host operating system supported by the exact tools you need. A Linux image running on the board does not determine which operating system every contributor must use.

* **Editing documentation or source code:** Use your preferred editor and follow the repository's formatting settings.
* **Building a Yocto image:** Follow the guide for that Yocto/BSP release, including its supported host distribution, packages, memory, and disk requirements. The [Yocto build guide is being developed in PR #1](https://github.com/kmutt-quantum-zcu104/documents/pull/1); it is not yet a complete recovery procedure.
* **Building PL designs:** Check the selected Vivado release's host requirements and the project's tested build procedure. Do not assume a newer tool version is interchangeable with the recorded one.
* **Building applications:** Use the project's documented native or cross-compilation environment. With a compatible SDK, application development may not require rebuilding the whole board image.

Windows, WSL2, containers, and remote hosts have different tool and device-access constraints. Use them where supported by the relevant procedure, rather than assuming they are interchangeable.

## Git

[We have a guide on how to configure and use Git](git-workflow.md). Please read it carefully before committing any code or documentation.

## EditorConfig

This repository includes an [EditorConfig](https://editorconfig.org/) file to request consistent indentation and line endings from supporting editors. Please make sure that your IDE or text editor supports it. If you use VS Code, be sure to install the EditorConfig extension.

## Vivado and EDA tooling

For projects using AMD Vivado, follow the project's recorded release, edition/license requirements, device support, and any board-file requirements. Keep hardware exports, bitstreams, and software/BSP inputs matched to the tested configuration.

If you need to program the board, follow the selected tool release's official cable-driver instructions. Identify the board's serial interface separately; do not assume the JTAG driver also provides the UART driver.

Vivado generates caches, temporary files, and build outputs. Follow the implementation repository's `.gitignore` and build instructions, and review generated files before committing. Tcl-based project recreation can help reproducibility, but the project should document which sources and configuration must be retained; do not discard files merely because a tool generated them.

## Software toolchain (Processing System)

For Linux applications running on the Cortex-A53 cores, the compiler, target architecture, libc, and libraries must be compatible with the installed image. The language and build tools depend on the project.

### Native builds on the board

Build on the board only if its image includes or supports the required compiler, headers, libraries, and build tools. A Yocto image does not necessarily include a package manager or Debian packages such as `build-essential`. Record how development tools are provided by that image instead of applying host Ubuntu package commands to it.

### Cross-compilation

For a Yocto-based target, use an SDK built for the relevant image/configuration where available. Follow its environment-setup instructions so the build uses the intended compiler and target sysroot. Installing a generic AArch64 compiler on the host is not enough to establish compatibility with the image.

The [Yocto SDK manual](https://docs.yoctoproject.org/sdk-manual/intro.html) explains the SDK model. Select the documentation version matching the Yocto release in use. Kernel modules also need the matching kernel build configuration and development artifacts; an application SDK alone may not provide them.

### Language-specific requirements

* **C / C++:** Follow the project's compiler, language-standard, library, and build-system requirements. Use its documented SDK integration for cross-builds.
* **Rust:** If the project uses Rust, follow the [official installation instructions](https://www.rust-lang.org/tools/install) and any pinned toolchain. For cross-builds, use the project's tested target, linker, and sysroot configuration. Adding a target with `rustup` does not configure all of these. Native installation on the board also depends on the image supporting the toolchain.

## Code formatting

Follow the formatter configuration and version recorded in the repository you are editing. Common tools include `clang-format` for C/C++ and `rustfmt` through `cargo fmt` for Rust. These are not required for projects that do not use those languages.

Avoid mixing unrelated formatting changes into a functional change.

## Terminal and serial communication

Serial-terminal options include Tera Term or PuTTY on Windows, and `picocom` or `minicom` on Linux. Use the device/COM port, baud rate, and other console settings confirmed for the board setup. Device names can change when cables or hosts change.

Consult the [ZCU104 Evaluation Board User Guide (UG1267)](https://docs.amd.com/r/en-US/ug1267-zcu104-eval-bd) for the board's interfaces and connections. The setup procedure should identify the actual console interface and any required host driver.

## Network and remote access

The shared-board access procedure is still to be documented. Confirm permission, availability, and the current connection method with the other board users before connecting or changing anything. Do not assume that campus Wi-Fi, a VPN, or Tailscale provides access, or that the image has an SSH server enabled.

Keep deployment-specific access details in internal documentation and credentials outside Git. This guide does not define scheduling or hardware safety policy.

## Other requirements

Specific sub-projects may require additional libraries or Python packages. Check the `README.md` inside each project repository for details before getting started.
