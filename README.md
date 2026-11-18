# Unix-Based Personal Automation Assistant

A hybrid C++, C, and Bash automation system designed to streamline routine UNIX/Linux system administration, file operations, process monitoring, and data analysis tasks through an integrated, menu-driven interface.

---

## Abstract

Routine UNIX/Linux administration involves repetitive command-line invocations across multiple standalone utilities (`find`, `tar`, `ps`, `df`, `grep`, `awk`, `sed`). The **Unix-Based Personal Automation Assistant** integrates these capabilities into a single, cohesive terminal application. Built with a modular C++ Object-Oriented core, low-level POSIX process management in C, and automated Bash scripts, the application demonstrates core operating system concepts—process creation, execution control, signal handling, stream redirection, and file-system traversal—within a unified framework.

---

## Introduction

UNIX environments provide robust command-line utilities, but coordinating them across complex multi-step workflows often requires manual context-switching and complex syntax memorization. This project establishes an extensible, unified assistant combining:
- High-level object-oriented management in **C++** (polymorphism, encapsulation, modular driver).
- System-level execution and signal demonstration in **C** (`fork`, `exec`, `wait`, POSIX signals).
- Rapid file, text, and maintenance automation via **Bash** and UNIX pipeline utilities.

Developed as a Project-Based Learning (PBL) initiative for the **UNIX Lab** course at **Jaypee Institute of Information Technology, Noida**.

---

## Problem Statement

Users and junior developers working in UNIX/Linux systems frequently encounter friction when performing recurring system maintenance, file analysis, log monitoring, and process audits. Remembering varied flags across divergent utilities (`tar`, `ps`, `wc`, `awk`, `find`, `sed`) introduces operational overhead and error susceptibility. Existing full-scale enterprise automation frameworks introduce heavy dependencies and steep learning curves for localized desktop or single-server administrative workflows. A lightweight, dependency-free, modular assistant is required to bundle essential UNIX operations without obscuring underlying operating system fundamentals.

---

## Objectives

1. **Streamline Routine Administration:** Provide an interactive, menu-driven command interface for common file management, cleanup, monitoring, and backup tasks.
2. **Demonstrate Core UNIX Systems Programming:** Showcase practical implementations of UNIX system calls (`fork()`, `execlp()`, `waitpid()`), signal handling (`signal()`, `raise()`), and standard C I/O.
3. **Leverage UNIX Filter Pipelines:** Integrate standard UNIX tools (`grep`, `sed`, `awk`, `sort`, `wc`, `cat`, `find`) through pipes and I/O redirection.
4. **Ensure Extensible Software Architecture:** Utilize C++ OOP principles (abstract base classes, dynamic dispatch, runtime polymorphism) to decouple the menu controller from module execution logic.
5. **Maintain Operational Traceability:** Automatically log all assistant actions and module invocations with timestamps in local audit logs.

---

## Key Features

- **Centralized Menu Dispatcher:** Unified C++ interface utilizing runtime polymorphism to invoke specialized modules.
- **Low-Level POSIX Process Control:** Direct demonstration of parent-child process lifecycle, execution overlay via `execlp()`, process status inspection, and user signal handling (`SIGUSR1`).
- **Comprehensive File Operations:** Interactive file and directory creation, copying, moving, deleting with safety confirmations, and deep hierarchical searching.
- **Archive & Backup Management:** Automated timestamped `.tar.gz` archive creation, archive extraction, and chronological backup history tracking.
- **Real-Time System Diagnostics:** Instant reporting on CPU utilization, memory thresholds (`free -h`), mounted disk capacity (`df -h`), uptime, and user sessions.
- **Pattern Matching & Text Processing:** In-file line/word/character metrics (`wc`), unique record sorting (`sort -u`), case-insensitive token counting (`awk`), and in-place substitutions (`sed`).
- **Disk Sanitization Utility:** Detection of empty files, zero-byte artifacts, temporary editor files (`*.tmp`, `*.temp`, `*~`), and files exceeding size thresholds (>10 MB) with interactive deletion safeguards.
- **Automated Directory Organization:** Automated sorting of directory contents into extension-based subdirectories.

---

## Architecture

The system utilizes a 3-tier hybrid architecture combining C++, C, and Shell environments:

```text
+-------------------------------------------------------------+
|                     User Terminal (CLI)                     |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
|                  C++ Main Driver (main.cpp)                 |
|  - Menu controller loop                                     |
|  - Polymorphic vector: std::vector<unique_ptr<AutomationModule>> |
|  - Activity logger hook (logs/activity.log)                |
+-------------------------------------------------------------+
                              |
      +-----------------------+-----------------------+
      | (Dynamic Dispatch via virtual execute())       |
      v                                               v
+-----------------------------+         +-----------------------------+
|   Shell-Bridged Modules     |         |     Native C Subsystem      |
|   (modules.cpp)             |         |     (process_manager.c)     |
|   - FileManager             |         |   - Direct POSIX syscalls   |
|   - BackupManager           |         |   - fork(), execlp()        |
|   - SystemMonitor           |         |   - waitpid(), exit status  |
|   - TextAnalyzer            |         |   - signal(), raise(SIGUSR1)|
|   - CleanupUtility          |         +-----------------------------+
|   - ActivityLogger          |
|   - AutomationManager       |
+-----------------------------+
      | (system("bash scripts/..."))
      v
+-------------------------------------------------------------+
|                   Bash Scripts (scripts/*.sh)               |
|  - Pipes (|), Redirection (>, >>, <)                        |
|  - UNIX Utilities: find, tar, grep, awk, sed, sort, wc, ps   |
+-------------------------------------------------------------+
```

### Object-Oriented Design (C++)
- **Base Class (`AutomationModule`):** Defines abstract `virtual void execute() = 0;` interface alongside module metadata (`moduleName`, `description`) and script invocation helpers (`runScript()`).
- **Derived Modules:** Eight distinct classes (`FileManager`, `BackupManager`, `SystemMonitor`, `TextAnalyzer`, `CleanupUtility`, `ProcessManager`, `ActivityLogger`, `AutomationManager`) override `execute()` to invoke their respective scripts or native routines.
- **Runtime Polymorphism:** Stored inside `std::vector<std::unique_ptr<AutomationModule>>` and dispatched dynamically based on numeric user selection.

---

## Technologies and UNIX Concepts

| Concept / Technology | Implementation Details & Usage |
|---|---|
| **C++ (C++17)** | Abstract base classes, inheritance, smart pointers (`std::unique_ptr`), STL vectors, streams (`std::ifstream`, `std::ofstream`). |
| **C (C11)** | Low-level POSIX systems programming, memory safety, POSIX signal definitions. |
| **Standard C I/O** | `printf()`, `scanf()`, `perror()`, `getchar()`, formatting strings. |
| **UNIX File-System Operations** | `touch`, `mkdir -p`, `cp -i`, `mv -i`, `rm -i`, `find`, directory testing (`[[ -d ]]`), file existence testing (`[[ -f ]]`). |
| **Shell Scripting (Bash)** | Conditional logic, `case` statements, string parameter expansions (`${file##*.}`), subshells, exit status checks (`$?`). |
| **Text Processing & Filters** | `grep` (regex pattern search), `sed` (stream editor substitutions), `awk` (token/field iteration and frequency counting), `sort -u` (unique line sorting), `wc` (line, word, byte counting), `cat`. |
| **Process Control Syscalls** | `fork()` (process duplication), `execlp()` (binary image overlay), `waitpid()` / `wait()` (parent-child synchronization and exit status capture `WIFEXITED`, `WEXITSTATUS`). |
| **Signal Handling** | `signal(SIGUSR1, handle_signal)` for registering signal traps; `raise(SIGUSR1)` for programmatically sending POSIX signals. |
| **Pipes and Redirection** | Pipes (`|`) for chaining filters; stdout redirection (`>`), append redirection (`>>`), stdin redirection (`<`), null-byte termination (`-print0` with `read -d ''`). |
| **Compression & Packaging** | `tar -czf` (gzip compression archive), `tar -xzf` (extraction). |
| **System Diagnostics** | `uptime` (system load and elapsed time), `df -h` (filesystem capacity), `free -h` (RAM buffers/cache), `ps -eo` (process inspection). |
| **Build Automation** | `Makefile` driving `gcc` and `g++` compilation, object dependency management, clean targets. |

---

## Module Descriptions

### Implemented Modules

1. **Smart File Manager (`scripts/file_manager.sh`)**
   - Interactive CRUD operations for files and directories.
   - Guarded copying (`cp -i`), renaming/moving (`mv -i`), and interactive deletion (`rm -i`).
   - Deep search utilizing `find` with filename and directory filters.
   - Scan for empty files (`find -empty`) and large files.

2. **Backup Manager (`scripts/backup.sh`)**
   - Generates compressed archives (`tar -czf`) tagged with timestamps (`YYYYMMDD_HHMMSS`).
   - Maintains an append-only archive log in `backups/history.txt`.
   - Lists available archives (`ls -lh`) and safely extracts backups (`tar -xzf`) into specified restoration targets.

3. **System Monitor (`scripts/system_monitor.sh`)**
   - Aggregates system metrics into an instant terminal dashboard.
   - Inspects active user (`whoami`), hostname (`hostname`), current date (`date`), uptime, memory allocation (`free -h`), mounted disk partitions (`df -h`), and top processes sorted by CPU usage (`ps -eo ... --sort=-%cpu`).

4. **File and Text Analyzer (`scripts/search_analyzer.sh`)**
   - Computes line, word, and character counts via `wc`.
   - Searches case-insensitive patterns via `grep -in`.
   - Filters duplicate lines via `sort -u`.
   - Executes case-insensitive token frequency analysis via `awk`.
   - Conducts global string replacement directly in files via `sed -i`.

5. **Cleanup Utility (`scripts/cleanup.sh`)**
   - Locates zero-byte empty files (`find -type f -empty`).
   - Identifies temporary and lock files matching `*.tmp`, `*.temp`, and editor backups (`*~`).
   - Flags files exceeding 10 MB (`find -size +10M`).
   - Implements safe, confirmed batch cleanup using temporary tracking buffers and null-delimited file streams.

6. **Process Manager (`src/process_manager.c`)**
   - Interactively queries active PID (`getpid()`) and parent PID (`getppid()`).
   - Uses `fork()` to spawn child processes and overlays them with `execlp()` for `ps -ef`, `df -h`, and `date`.
   - Parent synchronizes using `waitpid()` and checks child termination exit codes (`WEXITSTATUS`).
   - Demonstrates POSIX user signal handling by binding `SIGUSR1` to a custom C handler and triggering it via `raise()`.

7. **Activity Logger (`scripts/activity_logger.sh` & `src/main.cpp`)**
   - Hybrid logging engine: C++ records high-level module invocations into `logs/activity.log`.
   - Shell interface allows custom manual activity recording, keyword searches (`grep -in`), whole history inspections (`cat`), and event count auditing (`wc -l`).

8. **Automation Scripts (`scripts/automation.sh`)**
   - Generates timestamped system diagnostic dump files saved in `reports/system_report_*.txt`.
   - Automatically organizes unstructured files in a target directory into extension-specific folders.
   - Performs one-touch automated backups of specified directories into `backups/`.

### Planned Functionality (Future Roadmap)

- **Inter-Process Pipes in Core C++:** Implementation of native POSIX pipes (`pipe()`) and FIFOs (`mkfifo`) directly between C++ module dispatchers and C worker processes.
- **Background Cron Automation:** Native cron scheduling interface (`crontab`) for recurring automated backups and cleanup cycles.
- **Enhanced Log Rotation:** Automated rotation and gzip compression of `logs/activity.log` upon reaching size thresholds.
- **Permission & Security Auditing:** Automated scanning for insecure file permissions (e.g., world-writable files, SUID/SGID binaries).

---

## Repository Structure

```text
Unix-Based-Personal-Automation-Assistant/
├── include/
│   ├── assistant.h               # C function declaration for process_manager()
│   ├── automation_module.hpp      # C++ abstract base class AutomationModule
│   └── modules.hpp               # C++ derived class declarations (8 modules)
├── src/
│   ├── main.cpp                  # Application entry, menu loop, OOP dispatcher, audit logging
│   ├── modules.cpp               # C++ module implementations and script invocation bridge
│   └── process_manager.c         # C POSIX process management, fork, exec, signal handling
├── scripts/
│   ├── file_manager.sh           # Smart File Manager implementation
│   ├── backup.sh                 # Backup Manager implementation
│   ├── system_monitor.sh         # System Monitor implementation
│   ├── search_analyzer.sh        # File and Text Analyzer implementation
│   ├── cleanup.sh                # Cleanup Utility implementation
│   ├── activity_logger.sh        # Activity Logger implementation
│   └── automation.sh             # Automation Scripts implementation
├── Makefile                      # GNU Make build configuration
├── .gitignore                    # Git untracked patterns (binaries, logs, backups, reports)
└── README.md                     # Project documentation & official synopsis
```

---

## Installation, Compilation and Execution

### System Requirements
- Linux / Ubuntu environment (or WSL2 on Windows)
- GNU Compiler Collection (`gcc` supporting C11, `g++` supporting C++17)
- GNU Make
- Bash shell (version 4.0+)
- Standard POSIX utilities (`coreutils`, `tar`, `grep`, `sed`, `gawk`, `procps`)

### 1. Clone the Repository
```bash
git clone https://github.com/Neurotic-Imposter/Unix-Based-Personal-Automation-Assistant.git
cd Unix-Based-Personal-Automation-Assistant
```

### 2. Configure Script Permissions
Ensure all automation shell scripts have executable permissions:
```bash
chmod +x scripts/*.sh
```

### 3. Compile the Application
The included `Makefile` handles multi-language compilation (`gcc` for C, `g++` for C++) and links object files into the final executable:
```bash
make
```

### 4. Run the Application
Launch the menu-driven assistant:
```bash
./assistant
```

### 5. Clean Build Artifacts
To remove compiled object files and binaries:
```bash
make clean
```

*Note: Runtime directories (`logs/`, `backups/`, `reports/`) are provisioned automatically as modules execute.*

---

## Team Members

| Name | Enrolment Number | Institutional Email / Role |
|---|---|---|
| **Maantrik Rana** | 992501030454 | Team Member |
| **Eshan Sbharwal** | 992501030467 | Team Member |
| **Luv Malhotra** | 992501030468 | Team Member |
| **Atul Upmanyu** | 992501030473 | Team Member |

---

## Faculty and Academic Information

- **Institution:** Jaypee Institute of Information Technology, Noida
- **Course:** UNIX Lab
- **Program:** Bachelor of Technology (B.Tech) — Computer Science and Engineering
- **Semester:** 3rd Semester – 2nd Year
- **Batch:** F9
- **Project Type:** Project-Based Learning (PBL)
- **Faculty Guide:** Vishal Kumar

---

## Expected Outcomes

1. **Practical Mastery of UNIX Systems Programming:** Demonstrable comprehension of process lifecycles (`fork`, `exec`, `wait`), inter-process signaling, and file descriptor manipulation.
2. **Proficiency in UNIX Pipeline Tools:** Competence in utilizing standard filters (`grep`, `sed`, `awk`, `sort`, `wc`, `find`) to automate non-trivial data extraction and text transformation tasks.
3. **Multi-Language Systems Integration:** Hands-on experience integrating procedural C system calls, object-oriented C++ software architectures, and declarative Bash scripts into a cohesive software package.
4. **Enhanced Productivity & Reduced Operator Error:** Elimination of manual syntactical errors through validated, automated script templates for backups, maintenance, and diagnostics.
5. **Robust Software Engineering Fundamentals:** Application of modular design, source version control (Git), build automation (Make), and defensive input validation.

---

## Attribution & Acknowledgments

This repository builds upon the foundational architecture developed by Anuj Kumar Singh, Chirayu Mudgal, Bhanu Pratap Singh Dev, and Pratyaksh Kaushik ([Anuj27049/Unix-Based-Personal-Automation-Assistant](https://github.com/Anuj27049/Unix-Based-Personal-Automation-Assistant)), adapted and expanded under the academic curriculum of JIIT Noida.
