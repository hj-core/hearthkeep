# HearthKeep: WSL2 Android Hybrid Environment Diagnostic Checklist

## Instruction to the Troubleshooting LLM

You will act as an expert systems engineer and troubleshooter. When the user reports a build failure, emulator connectivity issue, or system lag, run their error logs and environment parameters against this checklist. Identify the root cause and output a structured resolution ticket detailing:

- **Diagnostic ID**: The alphanumeric tag violated or triggered.
- **Triage Severity**: [CRITICAL / WARNING / ADVISORY].
- **Affected Subsystem**: [ADB / Filesystem / Memory / Gradle / Network].
- **Root Cause Analysis**: Why the WSL2-to-Windows bridge failed.
- **Remediation Commands**: Direct copy-pasteable terminal commands or configuration blocks to resolve the lock.

---

## 1. ADB Connection & Device Bridging [CONN]

### [CONN-01] Version Alignment

- **Target Scope**: Windows host and WSL2 Ubuntu terminal
- **Severity**: CRITICAL
- **DO**: Ensure the Android Debug Bridge (`adb`) version compiled inside WSL2 matches the exact version running on the Windows host. Check version mismatch via:
  - Windows: `adb.exe version`
  - WSL2: `adb version`
- **DO NOT**: Allow the Windows host and WSL2 to run mismatched minor or major ADB versions, as they will continuously kill and restart each other's ADB background servers, losing emulator connections.

### [CONN-02] Port Forwarding Bridge

- **Target Scope**: WSL2 network interface
- **Severity**: CRITICAL
- **DO**: Bridge WSL2 to the Windows ADB server host via TCP port 5037. Connect WSL2's adb client to Windows by setting the environment variable in your `~/.bashrc` or `~/.zshrc`:
  - `export ADB_SERVER_SOCKET=tcp:$(cat /etc/resolv.conf | grep nameserver | cut -d' ' -f2):5037`
- **DO NOT**: Attempt to run a separate native ADB server inside WSL2 alongside the Windows ADB server.

---

## 2. Filesystem Boundaries & Performance [FILE]

### [FILE-01] Mount Point Isolation

- **Target Scope**: Git Workspace Directories
- **Severity**: CRITICAL
- **DO**: Maintain your entire `HearthKeep` repository root folder strictly inside the native Linux filesystem (e.g., `/home/username/code/HearthKeep`).
- **DO NOT**: Store your project files inside mapped Windows drives (such as `/mnt/c/Users/...`) and access them via WSL2. This cross-system filesystem bridging degrades disk performance by up to 10x, leading to extremely slow Gradle syncs and compiler timeouts.

### [FILE-02] Linux Inotify Capacity

- **Target Scope**: Linux Kernel Configurations (`/etc/sysctl.conf`)
- **Severity**: WARNING
- **DO**: Increase the Linux file-watcher limit (`fs.inotify.max_user_watches`) to support high-volume file indexing when Android Studio on Windows opens WSL2 directories. Apply this via:
  - `echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p`
- **DO NOT**: Leave default, low kernel limits active, which causes IDE indexing to silently freeze or drop file updates.

---

## 3. JVM & Gradle Execution Stability [GRAD]

### [GRAD-01] JDK Consistency

- **Target Scope**: Gradle build environments (Windows IDE vs WSL CLI)
- **Severity**: CRITICAL
- **DO**: Force the exact same JDK version (e.g., OpenJDK 17) inside your WSL2 environment (via SDKMAN!) and your Windows Android Studio settings.
- **DO NOT**: Allow compilation to mismatch (such as WSL terminal compiling with JDK 21 while Windows IDE relies on JDK 17), which corrupts compiled Gradle cache states and leads to strange runtime exceptions.

### [GRAD-02] Frozen Daemon Recovery

- **Target Scope**: Memory-locked compilation states
- **Severity**: WARNING
- **DO**: Completely kill hung compiler daemons, release local system locks, and wipe persistent Gradle temporary caches whenever builds lock up indefinitely:
  - Clean commands sequence: `./gradlew --stop && rm -rf .gradle/`
- **DO NOT**: Chain repetitive retry compiles without clearing out zombie Gradle worker processes first.

---

## 4. Resource Allocation & System Triage [RES]

### [RES-01] VMMem Bloat Limit

- **Target Scope**: Windows `.wslconfig` file (`%USERPROFILE%\.wslconfig`)
- **Severity**: WARNING
- **DO**: Limit the maximum physical RAM and CPU cores that WSL2 can consume to prevent the Windows host system and your visual Android Emulator from starving for resources. Configure it by adding these lines to your Windows `%USERPROFILE%\.wslconfig` file:

  [wsl2]
  memory=8GB
  processors=4
  guiApplications=false

- **DO NOT**: Allow WSL2 to run without a configuration cap, which lets compile processes consume 100% of Windows memory and crash both WSL2 and your emulator.
