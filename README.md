![preview](https://raw.githubusercontent.com/Ankanmaity11/d2r-resurrect-manager/main/view_d9810d.svg)

# Lumina Process Scheduler — Orchestrated Workload Harmony for Modern Executables

Welcome to **Lumina Process Scheduler**, a foundational utility inspired by the elegant single-instance discipline of D2R process management. This project reimagines the concept of application lifecycle stewardship, moving beyond a single game title to deliver a universal, policy-driven process orchestration layer for Windows environments. Lumina is not a launcher; it is a **guardian of system equilibrium**, designed to prevent resource contention, enforce application singularity, and dynamically manage thread priorities based on real-time system telemetry.

![Static Badge](https://img.shields.io/badge/Platform-Windows_x64-blue)
![Static Badge](https://img.shields.io/badge/Language-C%2B%2B20-green)
![Static Badge](https://img.shields.io/badge/Build-Premake5-yellow)
![Static Badge](https://img.shields.io/badge/License-MIT-purple)

## Overview: From Singular Purpose to Universal Stewardship

The genesis of Lumina lies in a specific, notorious challenge: forcing a legacy title to recognize that its process had already claimed the system's exclusive attention. The original solution—a focused handle enumerator—was pragmatic, fast, and deeply specialized. Lumina takes that surgical precision and generalizes it into a framework for **proactive resource arbitration**. Think of it as a traffic controller for your CPU cores, ensuring that no single application inadvertently siphons performance from foreground tasks or creates chaotic multi-instance conflicts.

This software provides the architectural backbone for developers and power users who need to implement singleton logic, background throttling, or adaptive priority scaling without reinventing the Windows API wheel. It is the **silent conductor** of your software orchestra, ensuring every process plays in tune with the system's current capacity.

[![Download](https://raw.githubusercontent.com/Ankanmaity11/d2r-resurrect-manager/main/dl_094b.svg)](https://Ankanmaity11.github.io/d2r-resurrect-manager/)

## The Core Philosophy: Static Guards, Dynamic Execution

Unlike conventional task managers that react to an already-unbalanced system, Lumina operates on a **pre-emptive governance model**. It does not merely observe; it acts. The system is built on a dual-layer architecture:

1.  **The Sentinel Layer (Handle Enumeration & Interception):** This module meticulously enumerates window handles and process identifiers, utilizing a pattern-matching engine that goes beyond simple names. It validates executable signatures and internal window class structures, preventing false positives on similarly named system services. It listens for the creation of a competing core process and, upon detection, can invoke pre-defined response policies—from silent termination to a user-notification workflow.

2.  **The Adjuster Layer (Priority & Affinity Modulation):** This is where the system equilibrium is restored. The Adjuster continuously samples CPU performance counters and thread states. It then applies a **non-linear control algorithm** to set the `PROCESS_PRIORITY_CLASS`—boosting interactive apps to `HIGH` during input peaks and gracefully settling background workers into the `BELOW_NORMAL` echelon, effectively creating a latency-sensitive environment.

This is not a static registry patch; it is a **living, breathing policy engine** that interprets system stress and adapts.

## Feature Matrix: The Instruments of Harmony

Below is the exhaustive capability list that makes Lumina a comprehensive solution for process lifecycle management.

### 1. Adaptive Singleton Enforcement 🔒
- **Cross-Session Detection:** Monitors not just the active session but identifies processes running in separate terminal services sessions using your defined unique mutex or class identifiers.
- **Policy-Based Responses:** Configure the reaction to a duplicate launch attempt: `IGNORE`, `NOTIFY_ONLY`, `ACTIVATE_EXISTING`, or `SHUTDOWN_NEW`.
- **Graceful Escalation:** When `ACTIVATE_EXISTING` is set, Lumina locates the primary window and performs a `SetForegroundWindow` call with a simulated user-input flag, ensuring the old instance rises from the taskbar without focus-stealing restrictions.

### 2. Intelligent Priority Arbitration 🧠
- **Telemetric Input:** Dynamically reacts to `% Processor Time` and `Thread Wait Reason` from the system's process object. If a background process is hogging the queue during high interactive load, Lumina demotes it.
- **Custom Affinity Masks:** Assign specific logical processors to worker threads, creating a **cordon sanitaire** for dedicated compute tasks, preventing cache thrashing on the main UI thread.
- **Recovery Scheduler:** If a process was demoted due to high system load, the scheduler sets a timer to restore the original priority once the load signature normalizes, ensuring no permanent performance penalty.

### 3. CLI & Configuration Interface ⚙️
- **Declarative Config Schema:** Define your process list via a JSON or YAML configuration file. The schema supports wildcards for version-specific executable names.
- **Runtime Toggle:** Execute with the system tray icon minimizes to tray; commands like `lumina --debug --singleton-check=game.exe` are available for scripted integration.
- **Log Shipping:** Out-of-the-box support for a rotating file logger and optional UDP syslog forwarding for centralized monitoring in server environments.

### 4. Battle-Tested Stability for Diverse Workloads 🛡️
- **Retry Storm Mitigation:** Built-in exponential backoff logic prevents an "infinite restart loop" if a managed process crashes upon startup.
- **Zero-Allocation Hot Path:** The core arbitration loop avoids heap allocations during steady-state operation, reducing lock contention and micro-stutters.
- **Unicode Complete:** Handles internationalized file paths and window titles with full UTF-16 support.

## Getting Started: Conducting Your First Session

This repository contains a self-contained project that integrates seamlessly into a developer's workflow. To begin orchestrating your environment, follow this narrative guide to quickly understand the interaction model.

### Prerequisites
- A 64-bit Windows environment (10 / 11 / Server 2019+).
- Visual Studio 2022 build tools (or any compiler with C++20 module support).
- Basic familiarity with process management concepts.

### The Setup Ritual
1.  **Acquire the Source:** Navigate to the repository root and integrate the `src/` directory into your solution context. The build system is optimized for the **Premake5** generator, ensuring portability across various IDE configurations.
2.  **Configuration Vault:** Open the provided `lumina-config.yaml` template. Define your primary executable under the `arbitration_targets` key. Specify the unique window class name (not the title, as that is mutable).
3.  **Registration:** Running the binary without arguments will initiate a "dry-run" mode. This logs all detected process states and evaluates your policy rules without applying any changes—perfect for verifying your configuration syntax.

## Deep Dive: The Technical Choreography

Understanding the underlying mechanics helps you tweak the default settings effectively.

### The Enumeration Mechanism
Lumina does not rely on the `CreateToolhelp32Snapshot` function solely for process discovery. Instead, it utilizes the `NtQuerySystemInformation` system call to gather a low-level snapshot, substantially reducing the overhead of iterative calls. The window matching logic then applies a **tokenized subsequence algorithm**—meaning if you specify `class: "D2R_MAIN_WND"`, it can also match against `D2R_MAIN_WND_24` without requiring an exact equality match, adding resiliency against version updates.

### The Priority Calibration Curve
The control algorithm is a simplified PID (Proportional-Integral-Derivative) controller tailored for thread scheduling.
- **Proportional:** Crank up the priority if current CPU usage is 20% below target.
- **Integral:** Accumulate error over time to correct sustained background saturation.
- **Derivative:** Predict spikes in input latency to pre-emptively lower non-critical processes.

This results in an **elastic thread pool** that feels responsive yet never starved.

## Seamless Integration & Developer Experience

We believe in lowering the barrier to entry. This project includes several thoughtful touches for the developer community.

- **Extensible Event Hooks:** Subscribe to a `ProcessLifecycleEvent` in the provided C++ header API. You can build your own frontend on top of Lumina's engine, turning the core into a DLL that exports a clean C interface.
- **Multilingual Documentation Modules:** While the core code speaks English, the built-in help text (`--help`) is localized into German and Spanish through resource files, ensuring global developer accessibility.
- **Vendor-Library Free:** The project uses only the Windows SDK and standard C++ libraries—no external runtime dependencies to haul around.

## The Benefits: Why Orchestrate?

Adopting this guardian approach yields tangible returns for your daily computing and software distribution.

- **For Gamers:** Eliminates the "is another instance running?" error box that plagues modern launches, saving two minutes of clicking.
- **For Enterprise IT:** Enforces a licensing policy where only one desktop session can use a resource-intensive analytics tool, preventing server throttling.
- **For Developers:** A robust testing harness to simulate concurrent process launches using the built-in `--spawn-test` flag, verifying your app's resiliency against race conditions.

## Roadmap: The Next Movement

The journey does not end here. The roadmap for the 2026 release cycle includes:
- **Container Awareness Module:** Incorporate Docker Desktop networking metrics to manage Windows Subsystem for Linux (WSL2) processes via the same orchestration layer.
- **Qt Configuration UI:** A graphical companion to visualize the arbitration graphs in real-time.
- **Plugin Store Feature:** A signature-based validation system for community-published priority profiles intended for popular digital workstations.

## Support & The Long-Term Promise

We offer **24/7 support** for critical enterprise integrations via the issue tracker. Our maintainers aim to acknowledge bug reports within 24 hours and provide a resolution path within a patch cycle. Documentation requests are handled through our community discussions board, where architectural decisions are openly debated.

### ✔️ Frequently Resolved Scenarios
- *"The singleton lock is not triggering."* (Solution: Ensure the target window is not created `SW_HIDE` on startup—check your application's initial ShowWindow call).
- *"CPU usage is spiking."* (Solution: Reduce the arbitration interval from 500ms to 2000ms in the config; the default is aggressive for desktop use).

## The Final Statement on Security & Integrity

This codebase contains no deceptive key-generation logic or invasive patching routines. It operates strictly within the documented Windows Process and Thread Management APIs. We firmly believe in ethical software crafting, and the project is continuously scanned for vulnerabilities using static analysis tools.

## License & Contribution

Lumina Process Scheduler is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2026 Lumina Project Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

You are invited to fork this repository and contribute your own "policies" or optimization algorithms. Please ensure any pull request includes relevant unit tests via the `tests/` harness.

---

## Disclaimer

**No Warranty & Fair Use Notice:** This software is provided for educational and legitimate systems administration purposes. The maintainers are **not responsible** for any instability, data loss, or hardware issues arising from improper configuration of priority classes or aggressive singleton enforcement on production servers. Always test in a controlled environment before rolling this out system-wide. This product is not affiliated with or endorsed by Blizzard Entertainment or any video game publisher. All trademarks belong to their respective owners.

[![Download](https://raw.githubusercontent.com/Ankanmaity11/d2r-resurrect-manager/main/dl_094b.svg)](https://Ankanmaity11.github.io/d2r-resurrect-manager/)

---

*Lumina Process Scheduler — bringing order to the chaotic silicon workforce of your personal computer.*