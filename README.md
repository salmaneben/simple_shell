# Simple Shell (hsh)

A UNIX command line interpreter built in C, replicating the functionality of `/bin/sh`.

## Project Overview

This project is a comprehensive implementation of a simple shell that provides a command-line interface for interacting with the operating system. The shell supports basic command execution, built-in commands, command chaining, variable substitution, history management, and alias functionality.

## Author

- **Salmane Ben Yakhlaf** (salmanebenyakhlaf95@gmail.com)

## Features

### Core Functionality
- **Interactive and Non-interactive Mode**: Works both as an interactive shell and for script execution
- **Command Execution**: Execute programs from PATH or with absolute/relative paths
- **Built-in Commands**: Implements 8 essential built-in commands
- **Error Handling**: Proper error messages matching `/bin/sh` behavior
- **Memory Management**: No memory leaks, proper cleanup with custom memory functions
- **Signal Handling**: Handles Ctrl+C (SIGINT) gracefully without terminating

### Built-in Commands

1. **`exit [status]`** - Exit the shell with optional status code
   - Supports numeric exit codes with error checking
   - Returns to system with proper exit status

2. **`cd [directory]`** - Change current directory
   - `cd` - Change to HOME directory
   - `cd ~` - Change to HOME directory
   - `cd -` - Change to previous directory (OLDPWD)
   - `cd <path>` - Change to specified directory
   - Updates PWD and OLDPWD environment variables

3. **`env`** - Display all environment variables
   - Shows current environment in `KEY=VALUE` format

4. **`setenv VARIABLE VALUE`** - Set environment variable
   - Creates new or modifies existing environment variables
   - Requires exactly 2 arguments (variable name and value)

5. **`unsetenv VARIABLE`** - Remove environment variable
   - Removes specified environment variables
   - Accepts multiple variables as arguments

6. **`history`** - Display command history
   - Shows numbered list of previously executed commands
   - History stored in `.simple_shell_history` file

7. **`alias [name[='value']]`** - Manage command aliases
   - `alias` - Display all aliases
   - `alias name` - Display specific alias
   - `alias name='value'` - Create or update alias

8. **`help`** - Display help information
   - Shows basic shell usage information

### Advanced Features

#### Command Chaining
- **`;`** - Sequential execution (always executes next command)
- **`&&`** - Logical AND (executes next command only if previous succeeded)
- **`||`** - Logical OR (executes next command only if previous failed)

#### Variable Substitution
- **`$?`** - Exit status of last executed command
- **`$$`** - Process ID of the current shell
- **`$VARIABLE`** - Environment variable expansion
- **`${VARIABLE}`** - Environment variable expansion (alternative syntax)

#### Comment Handling
- Lines or parts of lines starting with `#` are treated as comments
- Comments are stripped before command processing

#### History Management
- Automatic command history logging
- Persistent history stored in `~/.simple_shell_history`
- Maximum of 4096 history entries (configurable via HIST_MAX)
- History numbering and management

#### Alias Support
- Create command shortcuts with aliases
- Recursive alias expansion (up to 10 levels deep)
- Persistent alias storage during session

#### PATH Resolution
- Automatic command lookup in PATH directories
- Support for absolute and relative paths
- Efficient path searching with caching

## Compilation

Compile the shell using:

```bash
gcc -Wall -Werror -Wextra -pedantic -std=gnu89 *.c -o hsh
```

## Usage

### Interactive Mode
```bash
$ ./hsh
$ ls -la
$ pwd
$ cd /home
$ echo "Hello World"
$ exit
```

### Non-interactive Mode
```bash
$ echo "ls -la" | ./hsh
$ ./hsh script_file
```

### Advanced Usage Examples

```bash
$ ./hsh
$ ls -l /tmp && echo "Success" || echo "Failed"
$ setenv MY_VAR "test value"
$ echo $MY_VAR
$ echo "Exit status: $?"
$ echo "Shell PID: $$"
$ alias ll='ls -la'
$ alias grep='grep --color=auto'
$ ll
$ history
$ cd ~ && pwd
$ cd - && pwd
$ ls ; echo "Done" ; pwd
$ unsetenv MY_VAR
$ # This is a comment
$ exit 98
```

## Project Structure

### Core Files

#### **`main.c`**
- Entry point of the shell
- Handles command-line arguments and file input
- Initializes shell environment and starts main loop
- Manages file descriptor operations for script execution

#### **`shell.h`**
- Master header file containing all function prototypes
- Defines data structures (`info_t`, `list_t`, `builtin_table`)
- Contains important macros and constants
- Includes all necessary system headers

#### **`shell_loop.c`**
- Implements the main shell loop (`hsh` function)
- Contains built-in command lookup and execution logic
- Manages command finding and PATH resolution
- Handles process forking and execution

### Input/Output Processing

#### **`getLine.c`**
- Custom implementation of input reading
- Handles command chaining buffer management
- Implements signal handling for Ctrl+C
- Manages input buffering and processing
- Contains `get_input()`, `_getline()`, and `sigintHandler()`

#### **`parser.c`**
- Command parsing and PATH resolution
- Determines if files are executable commands
- Implements PATH directory searching
- Functions: `is_cmd()`, `find_path()`, `dup_chars()`

#### **`tokenizer.c`**
- String tokenization for command parsing
- Splits input into command and arguments
- Handles multiple delimiters and edge cases
- Functions: `strtow()`, `strtow2()`

### Built-in Commands Implementation

#### **`builtin.c`**
- Core built-in commands: `exit`, `cd`, `help`
- `_myexit()` - Handles shell termination with status codes
- `_mycd()` - Directory change operations with PWD/OLDPWD management
- `_myhelp()` - Basic help functionality

#### **`builtin1.c`**
- Advanced built-in commands: `history`, `alias`
- `_myhistory()` - Display command history
- `_myalias()` - Alias management and display
- Helper functions: `set_alias()`, `unset_alias()`, `print_alias()`

#### **`environ.c`**
- Environment variable management: `env`, `setenv`, `unsetenv`
- `_myenv()` - Display environment variables
- `_mysetenv()` - Set environment variables
- `_myunsetenv()` - Remove environment variables
- `populate_env_list()` - Initialize environment from system

### String Manipulation

#### **`string.c`**
- Basic string operations: `_strlen()`, `_strcmp()`, `_strcat()`
- `starts_with()` - String prefix matching for environment variables

#### **`string1.c`**
- Advanced string operations: `_strcpy()`, `_strdup()`, `_puts()`, `_putchar()`
- Custom output functions with buffering

#### **`exits.c`**
- Additional string functions: `_strncpy()`, `_strncat()`, `_strchr()`
- Safe string operations with length limits

### Memory Management

#### **`memory.c`**
- Basic memory management: `bfree()` for safe pointer freeing

#### **`realloc.c`**
- Memory operations: `_memset()`, `ffree()`, `_realloc()`
- Custom memory allocation and cleanup functions

### Advanced Features

#### **`vars.c`**
- Variable substitution and command chaining logic
- `replace_vars()` - Expands variables ($?, $$, $VAR)
- `replace_alias()` - Performs alias expansion
- `is_chain()`, `check_chain()` - Command chaining logic

#### **`history.c`**
- Complete history management system
- `read_history()`, `write_history()` - File I/O for history
- `build_history_list()` - Add commands to history
- `renumber_history()` - Maintain history numbering

#### **`lists.c` & `lists1.c`**
- Comprehensive linked list implementation
- Environment variables, history, and aliases stored as linked lists
- Functions for adding, deleting, searching, and converting lists
- Memory-safe list operations

### Error Handling

#### **`errors.c`**
- Error output functions: `_eputs()`, `_eputchar()`, `_putfd()`, `_putsfd()`
- Custom error output with file descriptor support

#### **`errors1.c`**
- Error conversion and printing utilities
- `print_error()` - Formatted error messages
- `convert_number()` - Number to string conversion
- `remove_comments()` - Comment processing

### Utility Functions

#### **`_atoi.c`**
- Utility functions: `_atoi()`, `_isalpha()`, `is_delim()`, `interactive()`
- String to integer conversion and character classification

#### **`getinfo.c`**
- Information structure management
- `clear_info()`, `set_info()`, `free_info()` - Manage shell state

#### **`getenv.c`**
- Environment variable utilities
- `get_environ()`, `_setenv()`, `_unsetenv()` - Low-level env operations

## Data Structures

### Main Structure (`info_t`)
The central data structure containing all shell state:
```c
typedef struct passinfo {
    char *arg;              // Current command line
    char **argv;            // Command arguments array
    char *path;             // Path to current command
    int argc;               // Argument count
    unsigned int line_count; // Line number for errors
    int err_num;            // Error number for exit
    int linecount_flag;     // Flag for line counting
    char *fname;            // Program filename
    list_t *env;            // Environment variables (linked list)
    list_t *history;        // Command history (linked list)
    list_t *alias;          // Command aliases (linked list)
    char **environ;         // Environment array
    int env_changed;        // Environment modification flag
    int status;             // Last command exit status
    char **cmd_buf;         // Command chain buffer
    int cmd_buf_type;       // Chain type (;, &&, ||)
    int readfd;             // Input file descriptor
    int histcount;          // History count
} info_t;
```

### Linked List Structure (`list_t`)
Used for dynamic storage of environment, history, and aliases:
```c
typedef struct liststr {
    int num;                // Node number/index
    char *str;              // String data
    struct liststr *next;   // Next node pointer
} list_t;
```

## System Calls and Functions Used

### Allowed System Calls
The shell uses only these approved system calls:
- **Process Management**: `fork()`, `execve()`, `wait()`, `waitpid()`, `wait3()`, `wait4()`, `getpid()`, `exit()`, `_exit()`
- **File Operations**: `open()`, `close()`, `read()`, `write()`, `stat()`, `lstat()`, `fstat()`, `access()`
- **Directory Operations**: `opendir()`, `readdir()`, `closedir()`, `chdir()`, `getcwd()`
- **Memory Management**: `malloc()`, `free()`
- **I/O Operations**: `isatty()`, `getline()`, `fflush()`, `perror()`
- **Signal Handling**: `signal()`, `kill()`
- **String Operations**: `strtok()` (though custom implementation preferred)

## Requirements Compliance

### Technical Requirements
- **Editors**: vi, vim, emacs
- **Compilation**: Ubuntu 20.04 LTS with gcc
- **Compile flags**: `-Wall -Werror -Wextra -pedantic -std=gnu89`
- **Style**: Betty coding style compliant
- **Memory**: Zero memory leaks (tested with valgrind)
- **Functions**: Maximum 5 functions per file
- **Headers**: Include guarded with `#ifndef _SHELL_H_`

### Output Compatibility
The shell produces output identical to `/bin/sh`, including:
- Command execution results
- Error messages with program name from `argv[0]`
- Exit status codes
- Signal handling behavior

Example error output:
```bash
$ echo "qwerty" | ./hsh
./hsh: 1: qwerty: not found
```

## Learning Objectives

This project demonstrates understanding of:

### UNIX System Programming
- Process creation and management (fork, execve, wait)
- Environment variable manipulation
- File I/O and system calls
- Signal handling and process control

### C Programming Concepts
- Memory management and dynamic allocation
- Linked list data structures
- Function pointers and callback mechanisms
- String manipulation and parsing

### Shell Design Principles
- Command parsing and tokenization
- PATH resolution and command lookup
- Built-in command implementation
- Error handling and user feedback

## Architecture Flow

```
Start (main.c)
      ↓
Initialize Shell Environment
      ↓
Populate Environment List
      ↓
Read Command History
      ↓
Main Shell Loop (shell_loop.c)
      ↓
Display Prompt (if interactive)
      ↓
Read User Input (getLine.c)
      ↓
Parse & Tokenize Input (tokenizer.c)
      ↓
Process Variables & Aliases (vars.c)
      ↓
Check Built-in Commands (builtin.c/builtin1.c)
      ↓                    ↓
Built-in Found        Built-in Not Found
      ↓                    ↓
Execute Built-in      Find in PATH (parser.c)
      ↓                    ↓
      ←──────────────→ Fork & Execute (shell_loop.c)
      ↓
Handle Command Chaining (vars.c)
      ↓
Update History (history.c)
      ↓
Continue Loop or Exit
      ↓
Cleanup and Write History
```

## Testing

The shell has been extensively tested for:
- **Compatibility**: Identical behavior to `/bin/sh`
- **Memory Management**: No memory leaks (valgrind clean)
- **Error Handling**: Proper error codes and messages
- **Built-in Commands**: All functionality working correctly
- **Advanced Features**: Variable substitution, chaining, aliases
- **Edge Cases**: Empty input, invalid commands, signal handling

## File Summary

| File | Purpose | Key Functions |
|------|---------|---------------|
| `main.c` | Entry point and initialization | `main()` |
| `shell_loop.c` | Main shell loop and command execution | `hsh()`, `find_builtin()`, `find_cmd()`, `fork_cmd()` |
| `builtin.c` | Core built-in commands | `_myexit()`, `_mycd()`, `_myhelp()` |
| `builtin1.c` | Advanced built-in commands | `_myhistory()`, `_myalias()` |
| `environ.c` | Environment management | `_myenv()`, `_mysetenv()`, `_myunsetenv()` |
| `getLine.c` | Input processing | `get_input()`, `_getline()`, `sigintHandler()` |
| `parser.c` | Command parsing and PATH | `find_path()`, `is_cmd()` |
| `vars.c` | Variable and chain processing | `replace_vars()`, `is_chain()` |
| `history.c` | History management | `read_history()`, `write_history()` |
| `lists.c/lists1.c` | Linked list operations | List manipulation functions |
| `string.c/string1.c/exits.c` | String operations | Custom string functions |
| `memory.c/realloc.c` | Memory management | Custom memory functions |
| `errors.c/errors1.c` | Error handling | Error output and formatting |

## Manual Page

For detailed usage information, see the included manual page:
```bash
man ./man_1_simple_shell
```

This implementation represents a fully functional UNIX shell with modern features while maintaining compatibility with traditional shell behavior.
