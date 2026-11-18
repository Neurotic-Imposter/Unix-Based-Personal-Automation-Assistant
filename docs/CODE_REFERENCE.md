# Code Reference Guide

A simple, beginner-friendly guide to the files, classes, and functions in the **Unix-Based Personal Automation Assistant**. This document is designed to help any team member easily understand how our code works and explain it during project evaluation.

---

## 1. How the Project Connects Together

Our project uses a clear three-part connection:
1. **C++ Main Menu (`src/main.cpp`):** Starts the application, shows the main menu, and holds a list of 8 modules.
2. **C++ Modules (`src/modules.cpp`):** When you choose an option, C++ uses polymorphism to call `execute()` on that module.
   - Seven modules run a Bash script using the helper function `runScript()`.
   - One module (`ProcessManager`) calls a low-level C function `process_manager()`.
3. **C & Bash Workers:**
   - **Bash scripts (`scripts/*.sh`)** handle file operations, backups, text analysis, and monitoring using standard Linux commands.
   - **C file (`src/process_manager.c`)** demonstrates POSIX system calls like `fork()`, `execlp()`, and `signal()`.

---

## 2. C++ Header Files (`include/`)

### `include/assistant.h`
* **Purpose:** Acts as a bridge between C and C++.
* **Key Code:**
  ```c
  #ifdef __cplusplus
  extern "C" {
  #endif

  void process_manager(void);

  #ifdef __cplusplus
  }
  #endif
  ```
* **Why we need `extern "C"`:** C++ changes function names internally (called *name mangling*) to support function overloading. C does not do this. `extern "C"` tells the C++ compiler to keep the function name plain and unmangled so it can link properly with `src/process_manager.o`.

---

### `include/automation_module.hpp`
* **Purpose:** Defines the base class `AutomationModule` that all 8 modules inherit from.
* **Important Variables:**
  * `std::string moduleName`: Holds the friendly name of the module (e.g., `"Smart File Manager"`).
  * `std::string description`: A short summary of what the module does.
* **Important Functions:**
  * `AutomationModule(name, description)`: Constructor that stores the module name and description.
  * `virtual void execute() = 0`: A **pure virtual function**. This means every child class must write its own version of `execute()`.
  * `void displayInfo() const`: Prints a banner with the module's name and description.
  * `const std::string& getName() const`: Returns the module name for activity logging.
  * `bool runScript(const std::string& scriptPath)`: Helper function that runs a Bash script and checks if it finished successfully.

---

### `include/modules.hpp`
* **Purpose:** Declares the 8 specific module classes that inherit from `AutomationModule`.
* **Classes:**
  1. `FileManager`
  2. `BackupManager`
  3. `SystemMonitor`
  4. `TextAnalyzer`
  5. `CleanupUtility`
  6. `ProcessManager`
  7. `ActivityLogger`
  8. `AutomationManager`
* Each class has a simple constructor and overrides `void execute() override;`.

---

## 3. C++ Source Files (`src/`)

### `src/main.cpp`
* **Purpose:** The entry point of the entire application.
* **Important Functions:**
  * `void logActivity(const std::string& activity)`:
    * **Input:** A string describing what happened (e.g., `"Assistant exited"`).
    * **What it does:** Ensures `logs/` directory exists (`mkdir -p logs`), formats the current date and time (`YYYY-MM-DD HH:MM:SS`), and appends the entry to `logs/activity.log`.
    * **Output:** None.
  * `void displayMenu()`:
    * **What it does:** Prints the interactive terminal menu (options 1 to 8, plus 0 to exit).
  * `int main()`:
    * **What it does:**
      1. Creates a `std::vector` of `std::unique_ptr<AutomationModule>` containing instances of all 8 modules.
      2. Enters a loop: displays the menu, reads the user's choice, and validates input.
      3. If the user picks `0`, it logs the exit and terminates.
      4. If the user picks `1`–`8`, it retrieves the selected module pointer, calls `displayInfo()`, logs the launch to `logs/activity.log`, and executes the module via `selectedModule->execute()`.

---

### `src/modules.cpp`
* **Purpose:** Implements the methods declared in `automation_module.hpp` and `modules.hpp`.
* **Important Functions:**
  * `bool AutomationModule::runScript(const std::string& scriptPath)`:
    * **Input:** Path to a script (e.g., `"scripts/backup.sh"`).
    * **What it does:** Builds the command `bash <scriptPath>` and calls `system(command.c_str())`. It checks POSIX macros `WIFEXITED(result)` and `WEXITSTATUS(result) == 0` to confirm success.
    * **Output:** Returns `true` if the script ran cleanly, or `false` on error.
  * `execute()` methods for each module:
    * `FileManager::execute()` → runs `scripts/file_manager.sh`
    * `BackupManager::execute()` → runs `scripts/backup.sh`
    * `SystemMonitor::execute()` → runs `scripts/system_monitor.sh`
    * `TextAnalyzer::execute()` → runs `scripts/search_analyzer.sh`
    * `CleanupUtility::execute()` → runs `scripts/cleanup.sh`
    * `ProcessManager::execute()` → calls the C function `process_manager()`
    * `ActivityLogger::execute()` → runs `scripts/activity_logger.sh`
    * `AutomationManager::execute()` → runs `scripts/automation.sh`

---

## 4. C Source File (`src/process_manager.c`)

* **Purpose:** Demonstrates core operating system concepts in native C using POSIX system calls.
* **Important Functions:**
  * `void handle_signal(int sig)`:
    * **Input:** Signal number (`sig`).
    * **What it does:** Simple signal handler that prints the caught signal ID.
  * `void execute_command(char *command, char *arg)`:
    * **Input:** Command name (e.g., `"ps"`) and argument (e.g., `"-ef"`).
    * **What it does:**
      1. Calls `fork()` to create a child process.
      2. In child (`pid == 0`): Prints child PID and runs `execlp(command, command, arg, NULL)`.
      3. In parent (`pid > 0`): Prints parent PID and calls `waitpid(pid, &status, 0)` to wait until the child finishes. Then checks `WEXITSTATUS(status)`.
  * `void process_manager(void)`:
    * **What it does:** Displays an interactive menu for process operations:
      * Option 1: Shows current PID (`getpid()`) and parent PID (`getppid()`).
      * Option 2: Displays running processes (`execute_command("ps", "-ef")`).
      * Option 3: Displays disk usage (`execute_command("df", "-h")`).
      * Option 4: Displays system date (`execute_command("date", NULL)`).
      * Option 5: Demonstrates signals by binding `handle_signal` to `SIGUSR1` with `signal()` and sending it with `raise(SIGUSR1)`.
      * Option 0: Returns back to the main C++ menu.

---

## 5. Bash Shell Scripts (`scripts/`)

Each script runs independently in Bash and handles user interaction through a simple loop:

| Script Name | Main Commands Used | What It Does in Plain English |
| :--- | :--- | :--- |
| `file_manager.sh` | `ls -lah`, `touch`, `mkdir -p`, `cp -i`, `mv -i`, `rm -i`, `find` | Allows users to create files/folders, copy, move, rename, delete, and search for files. |
| `backup.sh` | `tar -czf`, `tar -xzf`, `date`, `ls -lh` | Creates compressed `.tar.gz` backups with date-time stamps, stores logs in `backups/history.txt`, and extracts backups. |
| `system_monitor.sh` | `whoami`, `hostname`, `uptime`, `free -h`, `df -h`, `ps` | Displays current user, uptime, RAM memory usage, hard disk usage, and CPU-intensive processes. |
| `search_analyzer.sh` | `wc`, `grep -in`, `sort -u`, `awk`, `sed -i` | Counts lines and words, searches text patterns, removes duplicate lines, counts word occurrences, and replaces words. |
| `cleanup.sh` | `find -empty`, `find -name "*.tmp"`, `find -size +10M`, `rm` | Scans for empty files, temporary files, and files over 10 MB, asking for confirmation before deleting. |
| `activity_logger.sh` | `cat`, `grep -in`, `wc -l` | Lets users view `logs/activity.log`, search for specific events, or add custom notes. |
| `automation.sh` | `df`, `free`, `uptime`, `mv`, `tar` | Automates routine maintenance: generates system reports in `reports/`, sorts files by file extension, and runs backups. |

---

## 6. Build File (`Makefile`)

The `Makefile` automates compiling our mixed C and C++ codebase:

```makefile
CC = gcc
CXX = g++
CFLAGS = -Wall -Wextra -Iinclude -std=c11
CXXFLAGS = -Wall -Wextra -Iinclude -std=c++17
TARGET = assistant
```

* **Compilation:**
  * Compiles `src/process_manager.c` using `gcc` into `src/process_manager.o`.
  * Compiles `src/main.cpp` and `src/modules.cpp` using `g++` into `src/main.o` and `src/modules.o`.
* **Linking:**
  * Runs `g++` on all three `.o` files to produce the final executable `assistant`.
* **Clean:**
  * `make clean` deletes `assistant` and all `src/*.o` files so you can build fresh.
