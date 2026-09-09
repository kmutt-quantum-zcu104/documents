# Development environment

This document assumes basic familiarity with Unix-like operating systems, Git, C++, Rust, and digital hardware concepts. If anything is unclear or seems broken, please just ask someone in the lab.

## Operating systems

FPGA tooling and embedded systems compilers run best on Linux.

* **Ubuntu 22.04 LTS (x86_64)** is our standard baseline for host-side development. If you are configuring a development machine or a build server, please stick to Ubuntu 22.04 LTS to avoid toolchain version mismatches.
* **Building Yocto from source** is not required for day-to-day work. The base OS image for the ZCU104 is built once and flashed to the board. You only need a full Yocto build environment on Ubuntu if you are modifying the kernel, device drivers, or recreating the system image from scratch.
* **Windows users** can write RTL (Verilog) and application code on Windows, but be sure to read our guidelines below regarding line endings, and consider using WSL2 or remote SSH into the board for running Linux-side tests.

## Git

[We have a guide on how to configure and use Git](git-workflow.md). Please read it carefully before committing any code or documentation.

## EditorConfig

We use [EditorConfig](https://editorconfig.org/) to enforce consistent indentation and line endings across different editors. Please make sure that your IDE or text editor supports it. If you use VS Code, be sure to install the EditorConfig extension.

## Vivado and EDA tooling

For Programmable Logic (PL) development, we standardize on AMD/Xilinx Vivado:

* Please install **Vivado ML Standard or Enterprise (version 2023.2 or later)**.
* Make sure you install the **Zynq UltraScale+ device support** and download the board definition files for the **ZCU104 Evaluation Kit**.
* Install the cable drivers using the official `install_drivers` script provided with Vivado so that your machine can communicate with the onboard JTAG/UART interface.

Vivado generates large amounts of transient build artifacts, temporary caches, and log files. **Never commit generated Vivado project files or build logs into Git.** Always use our repository `.gitignore` and generate projects using Tcl scripts.

## Software toolchain (Processing System)

The Processing System (PS) on the ZCU104 runs a 64-bit ARM Cortex-A53 quad-core processor. We support development in both C++ and Rust.

### C++

* We use modern C++ (C++17 or C++20).
* For building software directly on the board, ensure `build-essential`, `g++`, and `cmake` are installed on the board's Linux environment.
* If you cross-compile from an x86_64 host machine, install the aarch64 cross-toolchain:
  * On Ubuntu: `sudo apt install g++-aarch64-linux-gnu gcc-aarch64-linux-gnu cmake`

### Rust

* Install Rust using [`rustup`](https://rustup.rs/):
  ```sh
  curl --proto '=https' --tlsv1.2 -sSf [https://sh.rustup.rs](https://sh.rustup.rs) | sh
  ```
* If you cross-compile for the ZCU104 from your host machine, add the 64-bit ARM Linux target:
  ```sh
  rustup target add aarch64-unknown-linux-gnu
  ```
  (Note: You will also need `aarch64-linux-gnu-gcc` installed on your host as the linker).
* Alternatively, you can install `rustup` and `cargo` directly on the board's Linux to build natively.

## Code formatting

To keep formatting consistent across all repositories:

* **C / C++**: Format with `clang-format` (`clang-format -i <file>`).
* **Rust**: Format with `rustfmt` (`cargo fmt`).

## Terminal and serial communication

We communicate with the ZCU104 board via USB-UART and network shells:

* **Serial terminal**:
  * On Windows, we recommend **Tera Term** (or PuTTY). Connect to the Silicon Labs / FTDI Serial COM port at **115200 baud (8-N-1)**.
  * On Linux, use `picocom` (`picocom -b 115200 /dev/ttyUSB0`) or `minicom`.
* **Remote shell**: For everyday remote development, deploying bitstreams, and streaming data, we connect via **SSH** and **SCP** over the local lab network or Tailscale.

## Network and remote access

The ZCU104 board is connected to the university internal network.

* **On-campus**: If your workstation or laptop is connected directly to the university network (such as eduroam or lab Ethernet), you can access the board directly via SSH using its assigned local IP address or hostname.
* **Off-campus (KMUTT VPN)**: If you are working outside the university campus (e.g., from home), you must connect to the university VPN before attempting to SSH into the board:
  * For Windows 11 setup instructions, refer to the official [KMUTT VPN Configuration Guide (Windows 11, PDF in Thai)](https://cc.kmutt.ac.th/Files/VPN/VPN_Manual_update202510/Windows%2011/%E0%B8%84%E0%B8%B9%E0%B9%88%E0%B8%A1%E0%B8%B7%E0%B8%AD%E0%B8%81%E0%B8%B2%E0%B8%A3%E0%B8%95%E0%B8%B1%E0%B9%89%E0%B8%87%E0%B8%84%E0%B9%88%E0%B8%B2%20VPN%20Windows%2011.pdf).
  * General VPN service details and credentials are managed through the [KMUTT Computer Center](https://cc.kmutt.ac.th/).

## Other requirements

Specific sub-projects may require additional libraries or Python packages. Check the `README.md` inside each project repository for details before getting started.
