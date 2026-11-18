# Academic Project Report

> **Course:** UNIX Lab (24B15CS214)  
> **Program:** Bachelor of Technology (B.Tech) — Computer Science and Engineering  
> **Institution:** Jaypee Institute of Information Technology, Noida  
> **Semester:** 3rd Semester – 2nd Year  
> **Batch:** F9  
> **Project Type:** Project-Based Learning (PBL)  
> **Faculty Guide:** Vishal Kumar  

---

## Team Members & Responsibilities

| Name | Enrolment Number | Assigned Modules |
| :--- | :--- | :--- |
| **Maantrik Rana** | 992501030454 | Smart File Manager, File & Text Analyzer (`file_manager.sh`, `search_analyzer.sh`) |
| **Eshan Sbharwal** | 992501030467 | System Monitor, Cleanup Utility (`system_monitor.sh`, `cleanup.sh`) |
| **Luv Malhotra** | 992501030468 | C++ OOP Core Architecture, Module Dispatcher, Activity Logger (`main.cpp`, `modules.cpp`, `automation_module.hpp`) |
| **Atul Upmanyu** | 992501030473 | Native C Process Management, POSIX Signals, Build Automation (`process_manager.c`, `assistant.h`, `Makefile`) |

---

## 1. Abstract

Routine UNIX/Linux system administration involves running repetitive commands across various tools such as `find`, `tar`, `ps`, `df`, `grep`, `awk`, and `sed`. For students and beginners, remembering disparate syntax rules and command flags can be challenging.

The **Unix-Based Personal Automation Assistant** solves this by combining these administrative utilities into an easy-to-use, menu-driven terminal tool. It demonstrates:
- High-level object-oriented management in **C++** using inheritance and runtime polymorphism.
- Low-level operating system mechanics in **C** using `fork()`, `execlp()`, `waitpid()`, and signal handling (`SIGUSR1`).
- Shell automation in **Bash** using pipelines, filters, and standard utilities.

---

## 2. Problem Statement

Using standalone UNIX command-line tools requires memorizing many flags and commands. Mistakes while running complex commands can cause syntax errors or unintended file deletion. Our project provides an interactive menu that bundles essential tasks—file operations, system monitoring, backups, cleanup, and text processing—into a simple, educational terminal application.

---

## 3. Project Objectives

1. **Interactive Menu Interface:** Create an intuitive CLI menu for everyday system administration tasks.
2. **Demonstrate OS Systems Programming:** Implement practical examples of process creation (`fork()`), execution overlay (`execlp()`), parent-child synchronization (`waitpid()`), and POSIX signal handling in C.
3. **Use UNIX Filter Pipelines:** Combine tools like `grep`, `sed`, `awk`, `sort`, `wc`, and `find` using pipes and redirection.
4. **Object-Oriented Design:** Use an abstract base class (`AutomationModule`) in C++ to decouple the main menu from individual module implementations.
5. **Activity Logging:** Record all module invocations with timestamps into `logs/activity.log`.

---

## 4. System Architecture

The application is structured into three clean parts:

```text
                      +-----------------------------+
                      |         User Terminal       |
                      +-----------------------------+
                                     |
                                     v
                      +-----------------------------+
                      |       src/main.cpp (C++)    |
                      |   Menu loop & event logging |
                      +-----------------------------+
                                     |
                                     v (Virtual execute())
        +---------------------------------------------------------+
        |                                                         |
        v                                                         v
+-----------------------------+             +-----------------------------+
|    src/modules.cpp (C++)    |             |    src/process_manager.c    |
|   runScript("scripts/*.sh") |             |   Direct POSIX C Subsystem  |
+-----------------------------+             +-----------------------------+
        |                                                 |
        v                                                 v
+-----------------------------+             +-----------------------------+
|    7 Bash Shell Scripts     |             |  fork(), execlp(), waitpid()|
|   (file, backup, text, etc) |             |  signal(SIGUSR1), raise()   |
+-----------------------------+             +-----------------------------+
```

---

## 5. Module Summary

| Module | Implementation | Primary Commands / Functions | Purpose |
| :--- | :--- | :--- | :--- |
| **1. File Manager** | `scripts/file_manager.sh` | `ls -lah`, `touch`, `mkdir -p`, `cp -i`, `mv -i`, `rm -i`, `find` | File and directory management. |
| **2. Backup Manager** | `scripts/backup.sh` | `tar -czf`, `tar -xzf`, `ls -lh` | Create and restore `.tar.gz` backups. |
| **3. System Monitor** | `scripts/system_monitor.sh` | `whoami`, `hostname`, `date`, `uptime`, `df -h`, `free -h`, `ps` | Display memory, disk, and CPU usage. |
| **4. Text Analyzer** | `scripts/search_analyzer.sh` | `wc`, `grep -in`, `sort -u`, `awk`, `sed -i` | Text metrics, searching, sorting, and replacements. |
| **5. Cleanup Utility** | `scripts/cleanup.sh` | `find -empty`, `find -name "*.tmp"`, `find -size +10M` | Locate and remove unnecessary files. |
| **6. Process Manager** | `src/process_manager.c` | `fork()`, `execlp()`, `waitpid()`, `signal()`, `raise()` | Low-level process lifecycle and signals. |
| **7. Activity Logger** | `src/main.cpp`, `scripts/activity_logger.sh` | `std::ofstream`, `cat`, `grep -in`, `wc -l` | Record and inspect assistant activity logs. |
| **8. Automation Scripts** | `scripts/automation.sh` | `df`, `free`, `uptime`, `mv`, `tar` | Run system reports and auto-sort folders. |

---

## 6. Key Algorithms

### 6.1 Process Creation and Execution (`src/process_manager.c`)
1. Call `fork()` to create a child process.
2. In the child process (`pid == 0`):
   - Print child process ID (`getpid()`).
   - Call `execlp()` to execute the given system command.
3. In the parent process (`pid > 0`):
   - Print parent process ID.
   - Call `waitpid()` to wait for the child process to finish.
   - Check exit status using `WIFEXITED()` and `WEXITSTATUS()`.

### 6.2 Word Counting with AWK (`scripts/search_analyzer.sh`)
1. Initialize count to 0.
2. For each line in the file, loop through all words (fields $1 to $NF).
3. Convert both the word and target query to lowercase for case-insensitive matching.
4. Increment counter when a match occurs.
5. Print the total count in the `END` block.

---

## 7. Verification and Testing

| Test Case | Component | Input / Action | Expected Result | Status |
| :--- | :--- | :--- | :--- | :--- |
| **T1: Build** | `Makefile` | Run `make` | Produces `assistant` executable cleanly | **Passed** |
| **T2: Input Validation** | `src/main.cpp` | Enter non-number (e.g. `"abc"`) | Error message, clears `cin`, redisplays menu | **Passed** |
| **T3: Clean Exit** | `src/main.cpp` | Enter `0` | Logs exit to `logs/activity.log` and stops | **Passed** |
| **T4: Process IDs** | `src/process_manager.c` | Option 1 (Show PID) | Prints current PID and parent PID | **Passed** |
| **T5: Signal Handling** | `src/process_manager.c` | Option 5 (Signals) | Catches `SIGUSR1` and runs signal handler | **Passed** |
| **T6: Backup Creation** | `scripts/backup.sh` | Backup folder | Creates `.tar.gz` and records entry in `history.txt` | **Passed** |
| **T7: Text Search** | `scripts/search_analyzer.sh` | Count word frequency | Accurately returns word count | **Passed** |

---

## 8. Conclusion

The **Unix-Based Personal Automation Assistant** successfully connects C++ object-oriented control, C low-level systems programming, and Bash script automation into a cohesive tool. It provides a simple menu for everyday Linux administration while serving as a clear learning demonstration of core operating system concepts.
