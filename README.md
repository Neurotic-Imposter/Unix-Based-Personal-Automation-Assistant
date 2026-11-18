# Unix-Based Personal Automation Assistant

A menu-driven terminal assistant built using C++, C, and Bash to simplify everyday UNIX/Linux system administration, file management, text analysis, and process monitoring.

Developed as a Project-Based Learning (PBL) project for the **UNIX Lab** course at **Jaypee Institute of Information Technology, Noida**.

---

## Problem It Solves

Using a UNIX/Linux terminal often requires remembering many different commands, flags, and syntax rules for basic tasks like searching files (`find`), creating archives (`tar`), checking system stats (`df`, `free`, `ps`), and filtering text (`grep`, `sed`, `awk`, `wc`). 

This project solves that by bringing these everyday tasks into a single interactive menu. It also demonstrates how high-level object-oriented code in C++, low-level system calls in C, and shell scripts in Bash work together in a single application.

---

## Main Features

The application provides eight distinct functional modules:

1. **Smart File Manager (`scripts/file_manager.sh`)**
   - List files and directories with permissions and sizes.
   - Create, copy, rename, move, and safely delete files or folders.
   - Search for files by name, find empty files, and find large files.

2. **Backup Manager (`scripts/backup.sh`)**
   - Create timestamped `.tar.gz` compressed backup archives.
   - View backup history stored in `backups/history.txt`.
   - Safely extract existing backup archives to a target directory.

3. **System Monitor (`scripts/system_monitor.sh`)**
   - Display username, hostname, system date, and system uptime.
   - Show available memory and RAM usage (`free -h`).
   - Show mounted disk partitions and disk usage (`df -h`).
   - List top active processes sorted by CPU usage (`ps`).

4. **File and Text Analyzer (`scripts/search_analyzer.sh`)**
   - Count lines, words, and characters in a file using `wc`.
   - Search for words or patterns using `grep`.
   - Display unique sorted lines using `sort -u`.
   - Count word occurrences using `awk`.
   - Find and replace text inside files using `sed`.

5. **Cleanup Utility (`scripts/cleanup.sh`)**
   - Scan for and clean empty files.
   - Find and delete temporary files (`*.tmp`, `*.temp`, `*~`).
   - Scan for files larger than 10 MB with interactive confirmation before deletion.

6. **Process Manager (`src/process_manager.c`)**
   - Display the current process ID (`getpid`) and parent process ID (`getppid`).
   - Spawn child processes using `fork()`, execute system commands using `execlp()`, and wait for child completion using `waitpid()`.
   - Demonstrate POSIX signal handling using `signal()` to catch `SIGUSR1` and `raise()` to trigger it.

7. **Activity Logger (`scripts/activity_logger.sh` & `src/main.cpp`)**
   - Automatically logs module launches and exit events with timestamps to `logs/activity.log`.
   - Interactive shell menu to view log contents, search log entries using `grep`, count total events, or add custom notes.

8. **Automation Scripts (`scripts/automation.sh`)**
   - Generate and save timestamped system health reports in `reports/`.
   - Automatically sort and organize loose files in a folder into subdirectories based on file extension.
   - Quick one-step directory backup.

---

## Technologies Used

* **C++ (C++17):**
  * Object-Oriented design using an abstract base class (`AutomationModule`).
  * Runtime polymorphism to run the selected module based on user input.
  * Smart pointers (`std::unique_ptr`) and STL vector for safe object management.
  * File streams (`std::ofstream`) for timestamped activity logging.
* **C (C11):**
  * Direct POSIX system calls: `fork()`, `execlp()`, `waitpid()`.
  * POSIX signal handling: `signal()`, `raise()`, `SIGUSR1`.
  * Standard C I/O: `printf()`, `scanf()`, `perror()`.
  * Linked to C++ using `extern "C"` in `include/assistant.h`.
* **Bash Shell Scripting:**
  * Interactive CLI scripts with user input validation.
  * Standard UNIX utilities and pipelines: `find`, `tar`, `grep`, `sed`, `awk`, `sort`, `wc`, `df`, `free`, `ps`.
* **Make:**
  * Automated compilation of C and C++ source files into a single binary (`assistant`).

---

## Project Structure

```text
Unix-Based-Personal-Automation-Assistant/
├── include/
│   ├── assistant.h               # C header declaring process_manager() with extern "C"
│   ├── automation_module.hpp     # C++ abstract base class AutomationModule
│   └── modules.hpp               # C++ derived class declarations for the 8 modules
├── src/
│   ├── main.cpp                  # Main menu loop, module selection, and logging
│   ├── modules.cpp               # C++ module implementations and script launcher
│   └── process_manager.c         # C process management, fork, exec, and signals
├── scripts/
│   ├── file_manager.sh           # File management script
│   ├── backup.sh                 # Backup and archiving script
│   ├── system_monitor.sh         # System diagnostics script
│   ├── search_analyzer.sh        # Text and pattern processing script
│   ├── cleanup.sh                # Temporary and large file cleanup script
│   ├── activity_logger.sh        # Log viewing and searching script
│   └── automation.sh             # Automated maintenance script
├── docs/
│   ├── ARCHITECTURE.md           # System architecture overview
│   ├── CODE_REFERENCE.md         # Simple code and file reference
│   ├── FINAL_REPORT.md           # College project report draft
│   └── LEARNING_GUIDE.md         # Concepts guide for viva and evaluation
├── Makefile                      # Build rules to compile C and C++ files
├── AGENTS.md                     # Antigravity project memory and rules
└── README.md                     # Main project synopsis and documentation
```

---

## How to Compile and Run

### Prerequisites
* Linux environment (Ubuntu / Debian, or WSL2 on Windows)
* GCC and G++ compilers (`gcc`, `g++`)
* GNU Make (`make`)
* Bash shell

### Step 1: Make Scripts Executable
Ensure the Bash scripts have execute permissions:
```bash
chmod +x scripts/*.sh
```

### Step 2: Compile the Project
Run `make` to compile the C and C++ source files and link them:
```bash
make
```

### Step 3: Run the Application
Start the assistant:
```bash
./assistant
```

### Step 4: Clean Build Files
To remove compiled `.o` files and the executable:
```bash
make clean
```

*(Note: Folders like `logs/`, `backups/`, and `reports/` are created automatically when the assistant runs.)*

---

## Team Details

| Name | Enrolment Number | Role |
| :--- | :--- | :--- |
| **Maantrik Rana** | 992501030454 | Team Member |
| **Eshan Sbharwal** | 992501030467 | Team Member |
| **Luv Malhotra** | 992501030468 | Team Member |
| **Atul Upmanyu** | 992501030473 | Team Member |

* **Institution:** Jaypee Institute of Information Technology, Noida  
* **Course:** UNIX Lab (Second-Year B.Tech CSE)
