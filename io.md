# Linux System I/O Basics

## What is I/O?

**I/O** stands for **Input/Output** - the way programs receive data (input) and send data (output). In Linux, every program uses three standard data streams:

- **Standard Input (stdin)** - where a program reads input from (usually your keyboard)
- **Standard Output (stdout)** - where a program sends normal output (usually your screen)
- **Standard Error (stderr)** - where a program sends error messages (also usually your screen)

**Why this matters:** Linux keeps output and errors separate so you can handle them differently. For example, you might want to save errors to one file and normal output to another.

**Try it yourself:**
```bash
cat myfile.txt
```
If the file exists, you see output (stdout). If it doesn't exist, you see an error (stderr). Both appear on your screen, but they use different streams.

## Everything is a File

Linux treats everything as a file, including:
- Regular files (documents, text files)
- Directories (folders)
- Devices (keyboard, screen, printers)
- Network connections

This makes I/O operations consistent and simple to work with.

**File Descriptors:** Each stream has a number:
- `0` = Standard Input (stdin)
- `1` = Standard Output (stdout)  
- `2` = Standard Error (stderr)

That's why `2>` redirects errors - it specifically targets file descriptor 2. When you just use `>`, it defaults to `1>` (stdout).

## Output Redirection

By default, command output appears on your screen. **Redirection** lets you send that output somewhere else, like to a file.

### Basic Redirection Operators

| Operator | Purpose | Example |
|----------|---------|---------|
| `>` | Send output to a file (overwrites) | `echo "Hello" > file.txt` |
| `>>` | Append output to a file | `date >> log.txt` |
| `2>` | Send error messages to a file | `ls /fake 2> errors.txt` |
| `&>` or `2>&1` | Send both output and errors to a file | `command &> all.txt` |
| `> /dev/null` | Discard output (send to "black hole") | `command > /dev/null` |

### Examples

**Overwrite a file:**
```bash
echo "New content" > myfile.txt
```

**Append to a file:**
```bash
echo "Additional line" >> myfile.txt
```

**Save only errors:**
```bash
ls /nonexistent-directory 2> error-log.txt
```

**Important tips:**
- Use `>` when you want to start fresh (overwrites existing content)
- Use `>>` when you want to keep existing content and add more
- Always check your work with `cat filename` after redirecting
- If a command produces no output, the redirected file will be empty (or created as empty)

## Pipes

The **pipe** operator `|` connects commands together. It takes the output of one command and feeds it as input to the next command.

### Syntax
```bash
command1 | command2
```

### Examples

**List files and search for a pattern:**
```bash
ls -l | grep ".txt"
```

**View file contents and filter for specific text:**
```bash
cat /etc/passwd | grep "root"
```

**Chain multiple commands:**
```bash
cat logfile.txt | grep "error" | wc -l
```
This counts how many lines contain the word "error".

**When to use pipes:**
- Use **pipes** (`|`) when you want to process data through multiple commands in sequence
- Use **redirection** (`>`) when you want to save the final result to a file
- You can do both: `command1 | command2 | command3 > result.txt`

**Pro tip:** Pipes work left-to-right. Build your pipeline one step at a time and test each stage before adding the next command.

## Common Commands for I/O

- `echo` - Display text or variables
  - Example: `echo "Hello World"`
- `cat` - Display file contents
  - Example: `cat filename.txt`
- `grep` - Search for text patterns
  - Example: `grep "search-term" filename.txt`
  - Case-insensitive: `grep -i "search-term" filename.txt`
- `wc` - Count lines, words, or characters
  - Lines only: `wc -l filename.txt`
- `head` - Show first lines of a file
  - First 10 lines (default): `head filename.txt`
  - First 20 lines: `head -n 20 filename.txt`
- `tail` - Show last lines of a file
  - Last 10 lines (default): `tail filename.txt`
  - Last 50 lines: `tail -n 50 filename.txt`
- `sort` - Sort lines of text
  - Example: `sort filename.txt`
- `uniq` - Remove duplicate lines
  - Example: `sort filename.txt | uniq` (sort first, then remove duplicates)

**Learning tip:** Try each command on its own first, then practice combining them with pipes.

## Combining Techniques

You can combine redirection and pipes:

```bash
cat /var/log/syslog | grep "error" > error-report.txt
```

This reads a log file, filters for lines containing "error", and saves the results to a new file.

**More examples:**

```bash
# Count how many times "root" appears in a file
cat /etc/passwd | grep "root" | wc -l

# Get last 100 lines, search for "error", save to file
tail -n 100 /var/log/syslog | grep -i "error" > recent-errors.txt

# List files, find only .txt files, save the list
ls -l | grep ".txt" > text-files-list.txt
```

**Building complex commands:**
1. Start simple: `cat filename.txt`
2. Add one pipe: `cat filename.txt | grep "pattern"`
3. Test it works, then add more: `cat filename.txt | grep "pattern" | wc -l`
4. Finally redirect if needed: `cat filename.txt | grep "pattern" | wc -l > count.txt`

## Key Concepts to Remember

1. **`>`** creates or overwrites a file
2. **`>>`** adds to the end of a file
3. **`|`** connects commands in a pipeline
4. **`/dev/null`** is a special file that discards everything sent to it
5. You can chain multiple pipes together
6. Redirection and pipes can be combined in the same command

## Practice Exercise

To test your understanding, try this:
```bash
# Create a test file
echo "apple" > fruits.txt
echo "banana" >> fruits.txt
echo "apple" >> fruits.txt
echo "cherry" >> fruits.txt

# Now experiment:
cat fruits.txt                    # View the file
cat fruits.txt | sort             # Sort alphabetically
cat fruits.txt | sort | uniq      # Remove duplicates
cat fruits.txt | grep "a"        # Find lines with 'a'
cat fruits.txt | wc -l            # Count lines
```

**Troubleshooting tips:**
- If redirection doesn't work, check file permissions with `ls -l`
- If a command isn't found, it may not be installed
- Use `cat filename` to verify file contents after redirection
- Remember: `>` overwrites, `>>` appends - choose carefully!