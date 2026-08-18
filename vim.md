# Vim Text Editor Basics

## What is Vim?

**Vim** (Vi Improved) is a powerful text editor that runs in the terminal. It's available on virtually every Linux system and is designed for efficient text editing without needing a mouse.

**Why learn Vim?**
- It's installed by default on most Linux servers
- Works over SSH connections where graphical editors won't
- Very fast once you learn the basics
- Essential for system administration and server management

**First-time warning:** Vim works differently from editors like Notepad. It has different "modes" for different tasks. This can be confusing at first, but becomes powerful with practice.

## Vim's Modes

Vim has three main modes you need to know:

### 1. Normal Mode (Command Mode)
- This is the **default mode** when you open Vim
- Used for navigation and commands
- Keys do NOT type text - they execute commands
- Press `Esc` to return to Normal Mode from any other mode

### 2. Insert Mode
- This is where you **actually type text**
- Press `i` from Normal Mode to enter Insert Mode
- You'll see `-- INSERT --` at the bottom of the screen
- Press `Esc` to return to Normal Mode

### 3. Command-Line Mode
- Used for **saving, quitting, and other file operations**
- Press `:` from Normal Mode to enter Command-Line Mode
- You'll see a `:` prompt at the bottom
- Type your command and press `Enter`

**Key concept:** Most confusion happens because people try to type in Normal Mode or can't exit Insert Mode. Remember: `Esc` always returns you to Normal Mode.

## Opening and Closing Files

### Opening a file
```bash
vim filename.txt
```

### Creating a new file
```bash
vim newfile.txt
```
The file is created when you save it.

### Exiting Vim

| Command | What it does |
|---------|-------------|
| `:q` | Quit (only works if no changes were made) |
| `:q!` | Quit without saving (discard all changes) |
| `:w` | Write (save) the file |
| `:wq` | Write and quit (save and exit) |
| `:x` | Same as `:wq` (save and exit) |
| `ZZ` | Save and exit (from Normal Mode, no `:` needed) |

**Getting stuck?** If Vim won't let you quit, press `Esc` a few times, then type `:q!` and press `Enter`. This forces Vim to quit without saving.

## Basic Navigation (Normal Mode)

Don't reach for the arrow keys - Vim uses keyboard letters for speed:

| Key | Movement |
|-----|----------|
| `h` | Left |
| `j` | Down |
| `k` | Up |
| `l` | Right |

**Tip:** The arrow keys work too, but learning `hjkl` makes you faster.

### Faster navigation

| Key | Movement |
|-----|----------|
| `w` | Jump forward one word |
| `b` | Jump backward one word |
| `0` | Go to beginning of line |
| `$` | Go to end of line |
| `gg` | Go to first line of file |
| `G` | Go to last line of file |
| `:n` | Go to line number n (e.g., `:15` goes to line 15) |

**Practice tip:** Open a file and practice moving around using only these keys. It feels awkward at first but becomes natural.

## Editing Text

### Entering Insert Mode

| Key | What it does |
|-----|-------------|
| `i` | Insert before cursor |
| `a` | Insert after cursor (append) |
| `I` | Insert at beginning of line |
| `A` | Insert at end of line |
| `o` | Open new line below and insert |
| `O` | Open new line above and insert |

**Remember:** After pressing any of these, you're in Insert Mode. Type your text, then press `Esc` to return to Normal Mode.

### Deleting Text (Normal Mode)

| Key | What it does |
|-----|-------------|
| `x` | Delete character under cursor |
| `dd` | Delete entire line |
| `dw` | Delete word |
| `D` | Delete from cursor to end of line |

**Deleted text is stored:** Anything you delete with `dd` or `dw` can be pasted back with `p`.

### Undo and Redo

| Key | What it does |
|-----|-------------|
| `u` | Undo last change |
| `Ctrl+r` | Redo (undo the undo) |

**Pro tip:** You can press `u` multiple times to keep undoing changes.

## Copy and Paste (Normal Mode)

In Vim, copying is called "yanking":

| Key | What it does |
|-----|-------------|
| `yy` | Yank (copy) entire line |
| `yw` | Yank (copy) word |
| `p` | Put (paste) after cursor |
| `P` | Put (paste) before cursor |

**Example workflow:**
1. Move cursor to a line you want to copy
2. Press `yy` (yank the line)
3. Move cursor to where you want to paste
4. Press `p` (paste below cursor)

## Searching Text

### Find text in file

| Command | What it does |
|---------|-------------|
| `/searchterm` | Search forward for "searchterm" |
| `?searchterm` | Search backward for "searchterm" |
| `n` | Go to next match |
| `N` | Go to previous match |

**Example:** Type `/error` and press `Enter` to find the word "error". Press `n` to jump to the next occurrence.

## Common Workflow Example

Here's a typical editing session:

```bash
# 1. Open the file
vim myfile.txt

# 2. Navigate to where you want to edit (use hjkl or arrow keys)

# 3. Press 'i' to enter Insert Mode

# 4. Type your changes

# 5. Press Esc to return to Normal Mode

# 6. Save and exit with :wq
```

## Essential Commands Reference

### Quick reference table

| Task | Command |
|------|---------|
| Enter Insert Mode | `i` |
| Exit Insert Mode | `Esc` |
| Save file | `:w` |
| Quit | `:q` |
| Save and quit | `:wq` or `ZZ` |
| Quit without saving | `:q!` |
| Delete line | `dd` |
| Undo | `u` |
| Copy line | `yy` |
| Paste | `p` |
| Search | `/searchterm` |

## Tips for Beginners

1. **Practice on test files first** - don't learn on important documents
2. **Use `:q!` to escape** - if you get stuck, this forces Vim to close
3. **Esc is your friend** - press it whenever you're unsure what mode you're in
4. **Start simple** - learn `i` (insert), `Esc` (normal), `:wq` (save and quit) first
5. **Arrow keys work** - use them until you're comfortable with `hjkl`
6. **Read the bottom of the screen** - Vim shows helpful information there

## Getting Help

While in Vim:
- `:help` - Opens Vim's built-in help
- `:help command` - Gets help for a specific command (e.g., `:help dd`)
- `:q` - Close the help window

## Practice Exercise

Try this to build muscle memory:

```bash
# 1. Create a practice file
vim practice.txt

# 2. Press 'i' to enter Insert Mode

# 3. Type a few lines of text

# 4. Press Esc to return to Normal Mode

# 5. Press 'dd' to delete a line

# 6. Press 'u' to undo the deletion

# 7. Press 'yy' to copy a line

# 8. Press 'p' to paste it

# 9. Type '/word' (where 'word' is something in your text) and press Enter

# 10. Press 'n' to find the next occurrence

# 11. Type ':wq' and press Enter to save and exit
```

## Why Vim Seems Hard

Vim has a learning curve because:
- **Modal editing** is unfamiliar (switching between Normal and Insert modes)
- **No visual menus** - you must remember commands
- **Different from modern editors** - no mouse, different shortcuts

But once you learn the basics, Vim becomes very efficient, especially when:
- Working on remote servers via SSH
- Making quick configuration file edits
- Navigating and editing large files without a mouse

## Alternatives

If Vim feels too complex, consider:
- **nano** - simpler terminal editor, shows commands on screen
- **vi** - Vim's predecessor, simpler but less features
- **vim with easy mode** - run `vim -y` for a more familiar experience

**However:** Learning basic Vim is valuable because it's guaranteed to be available on any Linux server you'll encounter.

---

*Start with the basics: `i` to insert, `Esc` to exit Insert Mode, `:wq` to save and quit. Master these three, then gradually add more commands as you need them.*
