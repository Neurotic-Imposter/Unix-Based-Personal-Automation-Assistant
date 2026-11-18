# Team Learning Guide: UNIX Systems & C/C++ Architecture

This guide is designed for the project members—**Maantrik Rana**, **Eshan Sbharwal**, **Luv Malhotra**, and **Atul Upmanyu**—as a foundational and practical reference for the core computer science and UNIX systems concepts implemented in the **Unix-Based Personal Automation Assistant**.

---

## 1. Multi-Language Compilation & The GNU Toolchain

### 1.1 What Happens When You Type `make`?
In this project, we combine **C** (`src/process_manager.c`) and **C++** (`src/main.cpp`, `src/modules.cpp`). These languages have different compilation rules:

1. **Preprocessing (`cpp`):** Handles `#include`, `#define`, and `#ifdef` macros. For example, `#include "modules.hpp"` replaces the line with the complete contents of `modules.hpp`.
2. **Compilation (`gcc` / `g++`):** Translates preprocessed source code into assembly code.
   - `gcc` compiles C using `-std=c11`.
   - `g++` compiles C++ using `-std=c++17`.
3. **Assembly (`as`):** Converts assembly into relocatable machine code stored in object files (`.o` files).
4. **Linking (`ld`):** Resolves symbols between object files. Here, `g++` is invoked as the linker because C++ requires runtime standard libraries (`libstdc++`):
   ```bash
   g++ -Wall -Wextra -Iinclude -std=c++17 src/process_manager.o src/main.o src/modules.o -o assistant
   ```

### 1.2 Demystifying `extern "C"`
Why do we need this block in `include/assistant.h`?
```c
#ifdef __cplusplus
extern "C" {
#endif

void process_manager(void);

#ifdef __cplusplus
}
#endif
```
- **Name Mangling in C++:** In C++, function overloading is supported (two functions can have the same name if their parameters differ). To support this, the C++ compiler alters function names in the symbol table (e.g., `_Z15process_managerv`).
- **C Linkage:** The C compiler does not mangle names; `process_manager` remains `process_manager`.
- Without `extern "C"`, `src/modules.o` would search for the mangled name, while `src/process_manager.o` only exports the unmangled name, causing an `undefined reference to process_manager` linker error.

---

## 2. Object-Oriented Design in C++

### 2.1 Abstract Base Classes & Pure Virtual Functions
In `include/automation_module.hpp`:
```cpp
class AutomationModule {
public:
    virtual void execute() = 0; // Pure virtual function
};
```
- A class with at least one pure virtual function (`= 0`) is an **abstract base class**. You cannot create a direct instance of it (`AutomationModule mod;` will fail to compile).
- It acts as an interface or contract: every derived class **must** provide its own definition of `execute()`.

### 2.2 Dynamic Polymorphism with Smart Pointers
Look at `src/main.cpp`:
```cpp
std::vector<std::unique_ptr<AutomationModule>> modules;
modules.push_back(std::make_unique<FileManager>());
...
AutomationModule* selectedModule = modules[choice - 1].get();
selectedModule->execute(); // Dynamic dispatch!
```
- `std::unique_ptr` is an **exclusive-ownership smart pointer**. When `modules` goes out of scope, all dynamically allocated module objects are deleted automatically with zero memory leaks.
- When `selectedModule->execute()` is called, the program consults the **virtual method table (vtable)** at runtime to determine whether to call `FileManager::execute()`, `BackupManager::execute()`, or another subclass implementation.

---

## 3. UNIX Process Control & System Calls

UNIX processes follow a strict hierarchy managed by kernel system calls.

### 3.1 Process IDs: PID & PPID
- Every running program in Linux has a unique **Process ID (PID)**.
- Every process (except `init` / `systemd` with PID 1) was spawned by a parent process, identified by the **Parent Process ID (PPID)**.
- In `src/process_manager.c`:
  ```c
  printf("Current Process ID: %d\n", getpid());
  printf("Parent Process ID: %d\n", getppid());
  ```

### 3.2 Spawning Processes: `fork()`
`fork()` creates an exact clone of the calling process:
```c
pid_t pid = fork();
```
- `fork()` is called **once**, but returns **twice**:
  - In the **parent process**, it returns the PID of the newly created child (> 0).
  - In the **child process**, it returns `0`.
  - If creation fails (e.g., system out of PID resources), it returns `-1`.

```
         Parent Process (PID: 1000)
                     |
                 fork()
                     +-----------------------+
                     |                       |
        Child Process (PID: 1001)   Parent Continues (PID: 1000)
        Returns: 0                  Returns: 1001
                     |                       |
                  execlp()                waitpid()
```

### 3.3 Image Overlay: `execlp()`
In `src/process_manager.c`:
```c
execlp(command, command, arg, (char *)NULL);
```
- `execlp()` replaces the current process memory (code, stack, heap, registers) with a new executable loaded from `PATH`.
- If `execlp()` succeeds, it **never returns** to the code that called it! The process finishes when the new executable terminates.
- If it returns at all, it means an error occurred (e.g., command not found).

### 3.4 Process Synchronization: `waitpid()`
```c
int status;
waitpid(pid, &status, 0);
```
- Without `waitpid()`, if the child finishes before the parent reads its exit code, the child becomes a **zombie process** (`<defunct>`).
- `waitpid()` blocks the parent until the child with ID `pid` changes state.
- We check the child's exit status using POSIX macros:
  - `WIFEXITED(status)`: True if child terminated normally (via `exit()` or `return`).
  - `WEXITSTATUS(status)`: Extracts the actual 8-bit exit code (0 for success).

---

## 4. UNIX Signals & Asynchronous Events

### 4.1 What is a Signal?
A signal is an asynchronous software interrupt sent to a process to notify it of an event (e.g., `SIGINT` on Ctrl+C, `SIGKILL` for immediate termination).

### 4.2 Handling `SIGUSR1` in C
In `src/process_manager.c`:
```c
void handle_signal(int sig) {
    printf("\nSignal received: %d\n", sig);
}

// In menu:
signal(SIGUSR1, handle_signal);
printf("\nRaising SIGUSR1...\n");
raise(SIGUSR1);
```
- `signal(SIGUSR1, handle_signal)` informs the kernel: "When `SIGUSR1` arrives, interrupt normal execution and run `handle_signal`."
- `raise(SIGUSR1)` sends the signal to the calling process itself.
- Once the handler completes, execution resumes at the instruction immediately following `raise()`.

---

## 5. Shell Scripting, Pipes & Redirection

### 5.1 File Descriptors
Every UNIX process starts with three default open streams:
- `0`: Standard Input (`stdin`)
- `1`: Standard Output (`stdout`)
- `2`: Standard Error (`stderr`)

### 5.2 Redirection Operators Used in This Project
- **`>` (Overwrite stdout):**
  `{ ... } > "$report"` in `scripts/automation.sh` writes all output to a report file.
- **`>>` (Append stdout):**
  `echo "$(date) | $activity" >> "$LOG_FILE"` appends a line without deleting prior logs.
- **`<` (Redirect stdin):**
  `wc -l < "$file"` feeds file contents into `wc` without displaying the filename in the output.
- **`|` (Pipe):**
  Connects `stdout` of the left command directly into `stdin` of the right command.
  Example in `scripts/system_monitor.sh`:
  ```bash
  ps -eo pid,comm,%cpu,%mem --sort=-%cpu | head -n 11
  ```
  `ps` emits all running processes sorted by CPU; `head -n 11` captures only the header plus the top 10 consumers.

---

## 6. UNIX Filter Utilities in Action

### 6.1 `grep` (Global Regular Expression Print)
Used in `scripts/search_analyzer.sh` and `scripts/activity_logger.sh`:
```bash
grep -in -- "$pattern" "$file"
```
- `-i`: Case-insensitive search (matches "hello", "Hello", "HELLO").
- `-n`: Prefixes each matching line with its 1-based line number.
- `--`: Indicates end of command flags (prevents patterns starting with `-` from breaking the command).

### 6.2 `sed` (Stream Editor)
Used in `scripts/search_analyzer.sh`:
```bash
sed -i "s|$old|$new|g" "$file"
```
- `-i`: In-place editing (modifies the file directly rather than streaming to stdout).
- `s|...|...|g`: Substitute pattern. The delimiter `|` is used instead of `/` so that file paths containing slashes do not cause syntax errors.
- `g`: Global flag (replaces all occurrences on each line, not just the first).

### 6.3 `awk` (Pattern Scanning and Processing Language)
Used in `scripts/search_analyzer.sh`:
```bash
awk -v word="$word" '
{
    for (i = 1; i <= NF; i++) {
        if (tolower($i) == tolower(word))
            count++
    }
}
END {
    print "Occurrences:", count+0
}' "$file"
```
- `-v word="$word"`: Passes a shell variable into the AWK runtime.
- `NF`: Built-in AWK variable storing the **Number of Fields** on the current line.
- `$i`: The value of the $i$-th space-separated word on the line.
- `tolower()`: Converts strings to lowercase for case-insensitive matching.
- `END { ... }`: Block executed after reading the entire file.

### 6.4 `find` (Recursive File Search)
Used in `scripts/cleanup.sh`:
```bash
find "$path" -type f \( -name "*.tmp" -o -name "*.temp" -o -name "*~" \) -print0
```
- `-type f`: Restricts results to regular files.
- `\( ... \)`: Logical grouping.
- `-o`: Logical OR.
- `-print0`: Separates each filename with an ASCII NUL character (`\0`) instead of a newline, preventing filenames containing spaces or special characters from breaking downstream processing.

---

## 7. Practical Exercise for Team Members

To practice and test these concepts:
1. **Trace a System Call:** Run `./assistant` under `strace`:
   ```bash
   strace -f -e trace=process,signal ./assistant
   ```
   Select Menu Option 6 (`Process Manager`) and Option 2 (`Running processes`). Observe `clone`/`fork`, `execve`, and `wait4` system calls in real time.
2. **Examine Symbols:** Inspect the compiled object files with `nm`:
   ```bash
   nm src/process_manager.o | grep process_manager
   nm src/modules.o | grep process_manager
   ```
   Verify that the symbol names match due to `extern "C"`.
