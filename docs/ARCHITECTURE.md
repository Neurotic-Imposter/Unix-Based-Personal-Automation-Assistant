# System Architecture

A simple, clear guide to the architecture of the **Unix-Based Personal Automation Assistant**, written for second-year CSE students.

---

## 1. High-Level Architecture

The project connects three core technologies:

```text
+-------------------------------------------------------------+
|                     User Terminal (CLI)                     |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
|                 C++ Main Driver (src/main.cpp)              |
|  - Displays interactive menu (displayMenu)                  |
|  - Holds 8 module objects in a vector                       |
|  - Logs every run to logs/activity.log (logActivity)        |
+-------------------------------------------------------------+
                              |
              +---------------+---------------+
              |                               |
              | (Choice 1-5, 7, 8)            | (Choice 6)
              v                               v
+-----------------------------+ +-----------------------------+
|    C++ Script Modules       | |      Process Manager        |
|      (src/modules.cpp)      | |      (src/modules.cpp)      |
|  - FileManager              | |  - Calls C function         |
|  - BackupManager            | |    process_manager()        |
|  - SystemMonitor            | +--------------+--------------+
|  - TextAnalyzer             |                |
|  - CleanupUtility           |                | extern "C"
|  - ActivityLogger           |                v
|  - AutomationManager        | +-----------------------------+
+--------------+--------------+ |    C POSIX Process Engine   |
               |                |   (src/process_manager.c)   |
               | runScript()    |  - fork() & execlp()        |
               v                |  - waitpid()                |
+-----------------------------+ |  - signal() & raise()       |
|         Bash Scripts        | +-----------------------------+
|        (scripts/*.sh)       |
|  - grep, sed, awk, tar,     |
|    find, wc, df, free, ps   |
+-----------------------------+
```

---

## 2. The Three Layers Explained

### Layer 1: High-Level Interface (C++)
* **Why C++?** C++ gives us Object-Oriented Programming (classes, inheritance, and polymorphism).
* **What it does:**
  * Runs the main loop and validates user input.
  * Treats all 8 modules uniformly through a shared base class `AutomationModule`.
  * Automatically writes timestamps and module launch events into `logs/activity.log`.

### Layer 2: Low-Level Systems Programming (C)
* **Why C?** C gives us direct access to UNIX operating system system calls without language overhead.
* **What it does:**
  * Demonstrates the parent-child process lifecycle: `fork()` creates a child process, `execlp()` replaces the child with a system command (`ps`, `df`, `date`), and `waitpid()` waits for the child to exit.
  * Demonstrates POSIX signal handling: registers `handle_signal()` for `SIGUSR1` and sends signals with `raise()`.
* **Connecting to C++:** Uses `extern "C"` in `include/assistant.h` so the C++ linker knows not to change (mangle) the C function name.

### Layer 3: Shell Automation (Bash)
* **Why Bash?** Shell scripts are the fastest and simplest way to automate file operations and chain UNIX filter utilities.
* **What it does:**
  * Seven C++ modules call `runScript("scripts/<name>.sh")`.
  * The scripts run standard Linux utilities like `find` (file searching), `tar` (compression), `grep` (pattern search), `sed` (stream editing), `awk` (data extraction), and `wc` (word count).

---

## 3. Object-Oriented Design in C++

### Inheritance
All modules share common traits (a name and description) defined in `AutomationModule` (`include/automation_module.hpp`). Each specific module inherits from this base class (`include/modules.hpp`).

### Polymorphism (Dynamic Dispatch)
In `src/main.cpp`:
```cpp
std::vector<std::unique_ptr<AutomationModule>> modules;
modules.push_back(std::make_unique<FileManager>());
modules.push_back(std::make_unique<BackupManager>());
// ... other modules ...

// The user inputs a number (e.g., choice = 2 for BackupManager)
AutomationModule* selectedModule = modules[choice - 1].get();
selectedModule->execute(); // Calls BackupManager::execute() at runtime
```
Because `execute()` is marked `virtual` in the base class, C++ automatically decides which module's `execute()` function to run based on the actual object stored at that index.

---

## 4. Compilation & Build Workflow

The project uses a single `Makefile` to compile both C and C++ files:
1. `gcc` compiles `src/process_manager.c` into `src/process_manager.o` (C11 standard).
2. `g++` compiles `src/main.cpp` and `src/modules.cpp` into `.o` files (C++17 standard).
3. `g++` links all object files into the final executable `assistant`.
