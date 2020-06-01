# Lecture 1: Shell

Way of interacting with computer.
Shell for more than visual interfaces; limited by UI.
Shell: textual tools, composable with one another and way to combine them or automate them.
Bash most common. Covered here.
Write commands at prompt.
Often executing programs with/without arguments.
Arguments can modify output of programs.

echo: takes multiple arguments. Spaces separate arguments. What if you want a word with space in? Can quote "hello world", which makes it one argument.

Single/double quotes are different.
Can escape single character: `Hello\ World`

Shell is really a programming language.
Environment variables often set already when starting the shell.
$PATH list that is colon separated

Refers to echo and finding which executable it is (not mentioned: slightly complicated in practice because of shell built-ins).

Paths are a way to name location of a file on a computer.
For writing scripts that work anyway, use either absolute path or name of program (if a program).
Generally, program work on the current working directory, by default. Don't have to give full paths for things.

... in help for commands; optional number zero or more.

Anything that doesn't take a value is usually considered a flag; anything that does is usually considered an option.
If you have write permission on file, but not on directory, you can empty the file, but not delete it as requires writing to directory.

rwx on file: read contents, modify file, run file
rwx on directory: list directory, modify directory, enter directory; for cd into directory, need execute on that directory and parent directory

Ctrl-L for clear.

Redirection of standard streams more a function of the shell; rewiring a program that needs know nothing about where e.g. its output comes from.

`>` redirects output
`>>` appends to file

Pipe connects output to input: `ls -l | tail -n1` — note that `ls` and `tail` don't know anything about each other, haven't been made to be compatible, all they know is how to read input and write to output. Pipe wires them together.

Pipes not just for text.

Root user: user ID 0.

/sys filesystem that aren't files, but kernel parameters, core of computer. What looks like a filesystem lets you access them.
Can use tools seen so far to manipulate them.
e.g. /sys/class/backlight/brightness

Can read brightness with `cat`, also reduce it with `echo`.

sudo echo 500 > brightness doesn't work; because programs don't know about the redirection. So, echo 500 runs as sudo and then the redirection happens as normal user and fails. The shell opens the brightness file, the shell is running as normal user, and therefore get permission denied error.

Dollar indicates normal user, hash indicates root.

However,

`echo 500 | sudo tee brightness` will work.

This is because we run `echo 500` as normal user, and then run `tee` as sudo. `tee` running as root then writes to standard output and to a file, which it is able to do.

# Lecture 2: Shell tools and scripting

## Assignment

`foo=bar`

No space as `foo = bar` would be: execute `foo` with arguments `=` and `bar`.

## Strings

Single and double quotes define strings, but double quotes: substitute in variable values; single quotes: literal string, do not substitute in variable values, e.g.

`echo "$foo"` outputs `bar`.
`echo '$foo'` outputs `$foo`.

## Functions

```
mcd () {
    mkdir -p "$1"
    cd "$1"
 }
```

## Special arguments

`$1` first argument to script or function. Bash has lots of special variables. There are others, including but not limited to (quoting from notes):

```
"   $0 - Name of the script
    $1 to $9 - Arguments to the script. $1 is the first argument and so on.
    $@ - All the arguments
    $# - Number of arguments
    $? - Return code of the previous command
    $$ - Process Identification number for the current script
    !! - Entire last command, including arguments. A common pattern is to execute a command only for it to fail due to missing permissions, then you can quickly execute it with sudo by doing sudo !!
    $_ - Last argument from the last command. If you are in an interactive shell, you can also quickly get this value by typing Esc followed by ."
```

## Return codes

Commands return output via STDOUT, error via STDERR and a return code/exit status reports errors.

`true` command always a 0 return code, success.
`false` command always a 1 return code, failure (any non-zero return code is a failure).

## `&&` (and) operator; `||` (or operator)

Can combine commands with these operators and conditionally execute.

e.g. `false || echo "Oops" prints `Oops`

`&&` execute the second command if the first one has 1 return code.

## Semicolon

Separates commands;

e.g. `false ; echo "This will always run"`

## Command and process substitution

### Command substitution

Placing: `$ ( CMD )` will execute `CMD` get the output and substitute it in place, e.g. `for file in $(ls)` then `ls` is run first and then the values are substituted in.

### Process substitution

`<( CMD )` executes `CMD` and puts the output in a temporary file, substituting `<(CMD)` with the file's name. Useful when commands expect values to be passed by a file instead of STDIN, e.g. `diff <(ls foo) <(ls bar)` will `diff` the two directories' contents as output by `ls`.

## Basic scripting

Example:

```
#!/bin/bash

echo "Starting program at $(date)" # Date will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in $@; do
    grep foobar $file > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

Double brackets should be used in Bash for comparisons; not portable to all other shells, but fewer gotchas

## Shell globbing

Often want to provide arguments to scripts or commands that are similar. Bash can expand these via "globbing".

### Wildcards

`?` used to match any single character.
`*` used to match any number of characters.

### Curly braces

`{ }` can expand a common substring e.g.

`convert image.{png,jpg}`
becomes
`convert image.png image.jpg`

### Combining wildcards and curly braces

`mv *{.py,.sh} folder`
becomes
`mv *.py *.sh folder`

### Other example

```
mkdir foo bar
touch {foo,bar}/{a..h} # create foo/a, foo/b… foo/h; bar/a, bar/b… bar/h
```

## Shellcheck

Useful tool for checking errors in scripts.

## Scripts called from shell don't have to be in shell script

e.g.
```
#!/usr/local/bin/python
…
```

Include the `#!` line: shebang line. Often useful to use the `env` command to resolve the location of the command, i.e. here would be `#!/usr/bin/env python`.

## Differences between shell functions and scripts

Functions in same language as shell. Scripts can be in any scripting language.
Functions loaded once when definition is read. Scripts loaded every time executed. Functions faster to load, but need to reload their definition if changed.
Functions are executed in current shell environment. Scripts execute in their own process. Functions can modify environment variables, but scripts can't, though scripts can be passed modified environment variables by use of `export`.
Functions are useful for modularity, code reuse and clarity of code. Shell language scripts often use their own function definitions.

## Finding out how to use commands

Can try:
* Call command with `-h` or `--help` flgs.
* `man` with command, e.g. `man rm`
* `tldr` pages as an alternative.

For interactive tools e.g. based on ncurses, often get help with `:help` or `?`.

## Finding files

### `find`

Examples:

```
# Find all directories named src
find . -name src -type d
# Find all python files that have a folder named test in their path
find . -path '**/test/**/*.py' -type f
# Find all files modified in the last day
find . -mtime -1
# Find all zip files with size in range 500k to 10M
find . -size +500k -size -10M -name '*.tar.gz'
```

### `locate`

Used a database that's regularly updated via `updatedb`, fast but may not be up-to-date. Also only uses name, but `find` can search based on different attributes.

## Finding code

`grep` match files based on some pattern.

Useful flags:

* `-C` for context around the line.
* `-v` for inverting the match.
* `-R` for recursively descending into directories.

Alternatives: `ack`, `ag`, `rg`.

## Finding shell commands

Up arrow: go back through history.
`history`: output history to standard output, can `grep` it.
Ctrl+R: most shells, search through history by typing a substring and repeatedly pressing Ctrl+R.

Note that starting a command with a leading space means it doesn't get added to command history. Can always edit history if you do accidentally type something sensitive without a leading space.

## Navigating directories

Tools like `fasd`, `tree`.

# Lecture 3: Vim

## Philosophy

* Vim is a modal editor: different modes for inserting text and manipulating text.
* Vim is programmable.
* Vim's interface is (like? — they say it is) a programming language: keystrokes are commands and commands are composable.
* Vim avoids use of mouse, and the use of the arrow keys for movement.

## Modal editing

Design based on the idea that programming involves lots of reading and
navigating, and making small edits, not writing long streams of text.

Multiple operating modes:

* **Normal**: for moving around a file and making edits (also found
  elsewhere: "manipulating text").
* **Insert**: for inserting text.
* **Replace**: for replacing text.
* **Visual** (plain, line, or block): for selecting blocks of text
* **Command-line**: for running a command.

Keystrokes have different meanings in different operating modes, e.g.
`x` key.

Vim shows current mode in bottom left; default is **normal**. Usually
spend most of time in normal and insert modes.

Change modes by pressing `<ESC>`. From normal mode, get to insert mode
with `i`, replace mode with `R`, visual mode with `v`, visual line mode
with `V`, visual block mode with `<C-v>` (also can write as `^v` or
`<Ctrl-v>`) and command-line mode with `:`.

## Basics

### Inserting text

From normal mode, enter insert mode via `i`. Can return to normal mode, to stop entering text.

### Buffers, tabs and windows

Vim has a set of open files: "buffers" (elsewhere: a file loaded into
memory). A Vim session has a number of tabs, each of which has a number
of windows (split panes). Each window shows a single buffer.

Not a 1-to-1 correspondence between buffers and windows: a window is a
view. A buffer can be open in multiple windows, even in the same tab,
e.g. view different parts of file at once.

By default, Vim opens with a single tab containing a single window.

### Command-line

Entering `:` in normal mode. Cursor jumps to command line at bottom of
screen.

Some commands:

* `:q` quit (close window)
* `:w` save ("write")
* `:wq` save and quit
* `:e {name of file}` open file for editing
* `ls` show open buffers
* `:help {topic} open help
  * `:help :w` opens help for `:w` command
  * `:help w` opens help for `w` movement

## Vim's interface is a programming language

(NB: not sure if this is their idea or a real concept.)

Keystrokes are commands and these commands *compose*. This enables
efficient movement and edits.

### Movement

Most of time usually spent in normal mode, using movement commands to
navigate the buffer.

Movements in Vim also called "nouns" because they refer to chunks of
text.

* Basic movement: `hjkl`
* Words: `w` (next word), `b` (beginning of word), `e` (end of word)
* Lines: `0` (beginning of line), `^` (first non-blank character), `$`
  (end of line)
* Screen: `H` (top of screen), `M` (middle of screen), `L` (bottom of
  screen)
* Scroll: `<Ctrl-u>` (up), `<Ctrl-d>` (down)
* File: `gg` (beginning of file), `G` (end of file)
* Line numbers: `:{number}<CR>` or `{number}G`
* Misc: `%` {corresponding item}
* Find: `f{character}`, `t{character}`, `F{character}`, `T{character}`,
  find/till forward/backward `{character}` on the current line; use `,`
  and `;` 
* Search: `/{regex}`, `n` and `N` for navigating matches.

### Selection

Visual modes:

* Visual
* Visual line
* Visual block

Can also use movement commands to select.

### Edits

Can do everything you would do with mouse via editing commands that
compose with movement commands.

Vim's editing commands are also called "verbs" because verbs act on
nouns.

* `i` enter insert mode
* `o`, `O` insert line below or above.
* `d{motion}` delete `{motion}`, e.g. `dw` delete word; `d$` delete to
  end of line, `d0` delete to beginning of line.
* `c{motion}` change `{motion}`, like `d{motion}` followed by `i`, e.g.
  `cw` change word
* `x` delete character (equal to `dl`)
* `s` substitute character (equal to `xi`)
* Visual mode and manipulation: select text, `d` to delete or `c` to
  change.
* `u` to undo, `<C-r>` to redo.
* `y` to copy/"yank" (some commands like `d` also copy)
* `p` to paste.
* `~` flips case of characters.

### Counts

Combine nouns and verbs with a count that perform a given action a
number of times:

* `3w` move 3 words forward.
* `5j` move 5 lines down.
* `7dw` delete 7 words.

### Modifiers

Can use modifiers to change "meaning". Some modifiers are
`i` meaning "inner" or "inside" and `a` meaning "around".

* `ci(` change contents inside current pair of parentheses
* `ci[` change contents inside current pair of square brackets
* `da'` delete a single-quoted string, including the surrounding single
  quotes.

## Vim mode in other programs

### Shell

For Bash, `set -o vi`

For all shells, `export EDITOR=vim` to use Vim as default editor.

### Readline

For programs using GNU Readline for their CLI,

`set editing-mode vi`

uses basic Vim emulation.

e.g. Python's REPL will support Vim bindings.

### Others

Lots of programs have extensions/plugins for this.

## Advanced Vim

### Search and replace

* `%s/foo/bar/g` replace foo with bar globally in file
* `%s/\[.*\](\(.*\))/\1/g` replace named Markdown links with plain URLs

### Multiple windows

* `:sp`, `:vsp` to split windows

### Macros

* `q{character}` to start recording a macro in register `{character}`
* `q` to stop recording
* `@{character}` replays the macro
* Macro execution stops on error
* `{number}@{character}` executes a macro {number} times
* Macros can be recursive. First clear the macro with `q{character}q`.
  Record the macro, with `@{character}` to invoke the macro recursively
  (will be a no-op until recording is complete). Example I looked at;
  doing an edit on a line for a series of similar lines. Can make macro
  for one line, then move down and re-run macro for next line, and
  repeat for all relevant lines. Alternatively, can include the move
  down and "call macro recursively", which will repeat for each line.
  Macro stops when it errors.

# Lecture 4: Data wrangling

Changing data from one format into another that you want, could be text or binary.

Often when using `|` in shell, you are data wrangling: e.g. `journalctl | grep -i intel` — going from entire system log to the Intel log entries.

Most data wrangling about knowing the tools available and how to combine them.

## Analyzing logs from server

With `ssh` to retrieve and then `grep` to search, e.g.

`ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' | less`

if we're interested in which usernames are trying to connect to the server.

May be preferable to do all the filtering on the remote server, means you send less data, and view locally.

Can save the logs instead to just have a single log to work with while developing.

Still lots of noise in logs, how can we reduce this?

### `sed`: a stream editor

Builds on `ed` editor. Give it short commands for modifying the file.

```
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

`s`: substitution command, common use of `sed`.

`.*Disconnected from ` is a "regular expression" or "regex" (from Wikipedia: "a sequence of characters that define a search pattern") — lets you match text against patterns.

Format of `s` command: `s/REGEX/SUBSTITUTION` — the text you want to substitute matching text with.

### Regular expressions

Regular expressions often surrounded by `/`. Most ASCII characters have normal meaning, but some characters have "special" matching behaviour. Commonly seen are:

* `.` means “any single character” except newline
* `*` zero or more of the preceding match
* `+` one or more of the preceding match
* `[abc]` any one character of a, b, and c
* `(RX1|RX2)` either something that matches RX1 or RX2
* `^` the start of the line
* `$` the end of the line

Note that `sed` often requires a `\` before these characters to give them their special meaning, or you can use `-E` flag.

`.*Disconnected from ` means the following:

* `.*` matches any text starting with any number of characters
* followed by `Disconnected from ` (with space at the end)

Note that, by default, `*` and `+` are "greedy", matching as much text as possible, e.g. for the text:

```
Jan 17 03:13:00 example.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
```

where someone tried to login with the username "Disconnected from", the result of this match is:

```
46.97.239.16 port 55920 [preauth]
```

Could fix this by using `?` after `.*` which makes them non-greedy (`?` matches zero or once), but `sed` doesn't support this. (However, e.g. Perl does at the command-line, but stick with `sed` here as it's commonly used.)

Difficult to match the text following the username as the username could have e.g. spaces. Instead, match the whole line:

```
sed -E 's/^.*Disconnected from (invalid |authenticating )?user .* [0-9.]+ port [0-9]+( \[preauth\])?$//'
```

* Matches the start roughly as before, but also using `^` to mark the start of line, so the regex matching starts from beginning of line, not arbitrarily part way through the line.
* Matches either `invalid user ` or `authenticating user ` zero or once.
* Matches any string for the username.
* Matches a series of numbers and dots (there's an IP address here).
* Match the `port ` and a number.
* Match the suffix `[preauth]` zero or once and the line end.

Here, the `Disconnected from` username is no longer a problem as we're also matching on the text "user", which appears before the user input of username.

The output of this is an empty log, because we've replaced all matching lines with an empty string. We can keep the username by using a "capture group" — text matched by a regex in parentheses is stored in a number capture group that are available in the substitution (and in some regex engines, in the pattern!) as `\1`, `\2`…:

So our command becomes:

```
sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [0-9.]+ port [0-9]+( \[preauth\])?$/\2/'
```

Regular expressions can be very complicated to understand, but are very useful.

### Processing outputs with `sort` and `uniq`

We can pipe the commands above into `sort` and `uniq`:

```
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [0-9.]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
```

`sort` sorts input; `uniq -c` collapses consecutive identical lines into a single line with a count of the occurrences. Can sort that output too:

```
…
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
```

```
129 abcdef
356 123
489 123123
494 asdfasdf
500 foo
511 bar
522 baz
799 foobar
893 foobarbaz
1000 root
```

`sort -n` will sort in numeric order, `-k1,1` means sort by the first whitespace separated column. (Otherwise `sort` sorts by whole line, which wouldn't actually make a difference here.)

`tail` gives the bottom lines, these are the ones with the highest counts. Could use `head` instead to get the lowest counts. Also `sort -r` sorts in reverse order.

What if we want usernames in one line, instead of one per line:

```
 | awk '{print $2}' | paste -sd,
```

`paste -sd,` combines lines `-s` with a single-character delimiter `-d`.

### `awk` — another editor

`awk` is a programming language that is good at processing text streams (By text stream, I'm assuming it's meant that it processes one line at a time: "awk is a line-oriented language", from GNU Awk User's Guide.)

`awk` programs take the form of an optional pattern plus a block saying what to do if the pattern matches 

Default pattern matches all lines. Inside the `{ }`, `$0` is the entire line, `$1` to `$n` represent fields of that line when separated by the `awk` field separator (whitespace by default, configure with `-F`).

`{print $2}` prints the second field, which is the username.

#### More complex usage

Compute the number of single-use username that start with `c` and end with `e`:

```
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l
```

Now there is a pattern, before `{…}`.

Pattern says the first field has to match 1 and the second has to start with `c` and end with `e`. The block prints the username if the pattern matches. Then count the number of lines with `wc -l`.

However, `awk` can do the counting itself:

```
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

`BEGIN` is a pattern that matches the start of the input and `END` a pattern that matches the end of the input. The count gets incremented by the value of the first field in rows that match (although this is always 1).

### Miscellaneous tools

#### Maths, statistics and plotting

`bc` calculates arithmetic expressions.

e.g. `printf "1\n2\n3\n" | paste -sd+` so if you have a series of numbers on lines, can combine them into one line separated by addition sign with `paste` and then add them with `bc`.

Can also use `R` at command line, `gnuplot`.

#### Data wrangling to make arguments

If you wrangle data into the format of arguments, can then run a command using `xargs`:

```
rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

#### Binary data

Can use pipes for working with binary data:

```
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.png' | feh -
```

This pipeline:
* captures a photo from a connected webcam;
* converts it to greyscale;
* `gzip`s it;
* copies the gzip to a remote machine;
* decompresses the file, leaving a copy on the machine, while piping the file as standard output;
* opens the file in an image viewer on local machine.

# Lecture 5: Command-line environment

## Job control

Unix has a communication mechanism called signals. When a process receives a signal, it stops its execution, deals with the signal and potentially changes the flow of execution based on the information delivered by the signal. Signals are software interrupts.

Example: pressing `Ctrl-C` in long running process in shell, delivers `SIGINT` signal to the process.

`SIGINT`, `SIGTERM` often associated with terminal related requests. More generic signal to ask a process to exit gracefully is `SIGTERM`. Can send this signal, with `kill -TERM <PID>`.

### Pausing and background processes

Signals can do things other than killing a process. `SIGSTOP` pauses a process. In terminal, `Ctrl-Z` prompts shell to send a `SIGTSTP` signal (Terminal Stop, the terminal's version of `SIGSTOP`). Can continue paused job in the foreground with `fg` or background with `bg`.

`jobs` lists unfinished jobs associated with the current terminal session. Can refer to those jobs with their pid (using `pgrep` to find that out). Also can refer to a process with `%` followed by job number (as shown by `jobs`). To refer to last backgrounded job, can use the shell `$!` special parameter.

`&` suffix will run the command in the background, giving you the shell prompt back, although it will still use the shell's standard output which can be annoying (can use shell redirection).

To background a running program, can use `Ctrl-Z` followed by `bg`. Backgrounded processes are child processes of the terminal and will die if you close the terminal (this sends `SIGHUP`). To prevent this, can run a program with `nohup` which is a wrapper to ignore `SIGHUP` or use `disown` if the process had been started. Or you can use a terminal multiplexer to avoid this problem.

# Lecture 6: Version Control and Git

Version control systems (VCS):
* Are tools to track changes to source code, or other files and folders.
* Maintain a history of changes.
* Facilitate collaboration.

Track changes to a folder and contents in series of snapshots, where each snapshot encapsulates the state of files/folders within a top-level directory. Also maintain metadata like who created each snapshot.

Useful even by yourself; look at old snapshots, keep log of changes, work on parallel branches of development.

## Git

Git is effectively a current standard for version control. Its interface is a leaky abstraction, however, therefore learning from the interface can be confusing. Commands can be "magic".

Git's underlying design and ideas are, however, beautiful.

### Git's data model

#### Snapshots

Git models the history of a collection of files and folders within some top-level directory as a series of snapshots. In Git terminology, a file is a "blob" — a bunch of bytes. A directory is a "tree" and maps names to blobs or trees (directories can contain other directories). A snapshot is the top-level tree that is being tracked:

```
<root> (tree)
|
+- foo (tree)
|   |
|   + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

##### Relating snapshots

How should a VCS relate snapshots? One simple model could be linear history. But Git doesn't use this. Instead, a history is a directed acyclic graph (DAG) of snapshots. Each snapshot refers to a set of "parents", snapshots that preceded it. This is a set of parents, not just a single parent, because a snapshot might descend from multiple branches, e.g. when you combine (merge) two parallel development branches.

Git calls these snapshots "commits".

Commit history might look like:

```
O <-- O <-- O <-- O
            ^
             \
              --- O <-- O
```

`O` represents a commit.
`<--` point to the parent of each commit.

After the third commit, the history branches into two separate branches. Could be two separate features developed in parallel. These branches could be merged to create a snapshot with both features:

```
O <-- O <-- O <-- O <---- X
            ^            /
             \          v
              --- O <-- O
```
With `X` being the merge commit.

Commits in Git are immutable. This doesn't mean you can't correct mistakes, just means that edits to the commit history are creating entirely new commits and references are updated to point to the new ones.

#### Data model as pseudocode

```
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | file>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parent: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

This is a clean, simple model of history.

##### Objects and content-addressing

An "object"! is a blob, tree, or commit:

```
type object = blob | tree | commit
```

In Git data store, all objects are content-addressed by their SHA-1 hash.

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blobs, trees, and commits are unified in this way: they are all objects. When they reference other objects, they don’t actually contain them in their on-disk representation, but have a reference to them by their hash.

For example, the tree for the example directory structure above (visualized using `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`), looks like this:

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

The tree itself contains pointers to its contents, `baz.txt` (a blob) and `foo` (a tree). If we look at the contents addressed by the hash corresponding to `baz.txt` with `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`, we get the following:

```
git is wonderful
```

##### References

Now, all snapshots can be identified by their SHA-1 hash. That’s inconvenient, because humans aren’t good at remembering strings of 40 hexadecimal characters.

Git’s solution to this problem is human-readable names for SHA-1 hashes, called "references". References are pointers to commits. Unlike objects, which are immutable, references are mutable (can be updated to point to a new commit). For example, the `master` reference usually points to the latest commit in the main branch of development.

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

With this, Git can use human-readable names like "master" to refer to a particular snapshot in the history, instead of a long hexadecimal string.

One detail is that we often want a notion of "where we currently are" in the history, so that when we take a new snapshot, we know what it is relative to (how we set the parents field of the commit). In Git, that "where we currently are" is a special reference called "HEAD".

##### Repositories

Finally, we can define what (roughly) is a Git repository: it is the data objects and references.

On disk, all Git stores are objects and references: that’s all there is to Git’s data model. All `git` commands map to some manipulation of the commit DAG by adding objects and adding/updating references.

Whenever you’re typing in any command, think about what manipulation the command is making to the underlying graph data structure. Conversely, if you’re trying to make a particular kind of change to the commit DAG, e.g. "discard uncommitted changes and make the master ref point to commit 5d83f9e", there’s probably a command to do it (e.g. in this case, `git checkout master`; `git reset --hard 5d83f9e`).

### Staging area

This is another concept that’s orthogonal to the data model, but it’s a part of the interface to create commits.

One way you might imagine implementing snapshotting as described above is to have a "create snapshot" command that creates a new snapshot based on the current state of the working directory. Some version control tools work like this, but not Git. We want clean snapshots, and it might not always be ideal to make a snapshot from the current state. For example, imagine a scenario where you’ve implemented two separate features, and you want to create two separate commits, where the first introduces the first feature, and the next introduces the second feature. Or imagine a scenario where you have debugging print statements added all over your code, along with a bugfix; you want to commit the bugfix while discarding all the print statements.

Git accommodates such scenarios by allowing you to specify which modifications should be included in the next snapshot through a mechanism called the "staging area".

# Lecture 7: Debugging and profiling

## Debugging

### Print statements versus logging

Print statements are useful for debugging; adding them around code you want to inspect.

Using logging is another approach, instead of print statements, to record events. Also has benefits:

* Can direct logs to different places, files, sockets or server instead of standard output/error.
* Often support severity levels that allow filtering of output.
* Can be useful if you encounter new issues.

### Third party logs

* Most programs write logs to somewhere in system, e.g. `/var/log/journal`.
* Can use `logger` shell program to send messages to logs.
* Logs often need processing and filtering to get the information you want. Can use flags of e.g. `journalctl` to help.

### Debuggers

Programs that let you interact with execution of a program, allowing you to:

* Halt execution of the program at a certain line.
* Step through the program one instruction at a time.
* Inspect values of variables after the program crashed.
* Conditionally halt the execution when a given condition is met.
* And more.

Many programming languages come with some form of debugger. For Python, this is `pdb`.

There is also `ipdb` that uses the IPython REPL.

### Specialised tools

#### System calls

When programs need to perform actions that only the kernel can, they use system calls ("syscalls").

Can trace the syscalls your program makes: `strace` for Linux; `dtrace` for OSX and BSD. `dtrace` has its own language, can use `dtruss` as a wrapper which makes `dtrace` more like `strace`.

For example:

```
strace -e lstat ls -l > /dev/null
```

#### Network tools

May need to look at the network packets to figure out an issue. Can use `tcpdump` and Wireshark to read network packets and filter them.

#### Web tools

Firefox/Chrome developer tools are useful.

#### Static analysis

For some issues, don't need to run code. Can analyse code to find issues without running it; static analysis tools take source code as input and analyse them using rules to reason about it.

Code linting for flagging problems, of code correctness, of style — can have autoformatting, of security. And for many languages, including shell scripts, and even e.g. Markdown.

## Profiling

Even if you code behaves as you expect, still might not be good enough if it uses too much CPU or memory. Finding hot spots in programs is useful; use profiling and monitoring tools.

### Timing

In a simple case, can print time around lines. Wall clock time can be misleading due to computer running other processes or waiting for events. Tools often distinguish different measures of time, e.g. output from `time`:

* Real — wall clock elapsed time from start to finish, including time taken by other processes and time taken while blocked (e.g. waiting for input/output or network).
* User — amount of time spent in the CPU running user code.
* Sys — amount of time spent in the CPU running kernel code.

### Profilers

#### CPU

Often by profilers people mean CPU profilers, which are the most common. Two main types:

* Tracing profilers: record every function call your program makes.
* Sampling profilers probe your program periodically and record the program's stack, and aggregates statistics of what your program spends time doing.

Most languages have a command-line profiler.

In Python `cProfile`: can run via `python -m cProfile -s tottime grep.py 1000 '^(import|\s*def)[^,]*$' *.py`

Many profilers, including `cProfile`, display time per function call. Can be unintuitive since internal function calls are accounted for. Often more intuitive to display the time taken per line of code; line profilers do this.

#### Memory

In non-garbage collected languages, e.g. C and C++, can have memory leaks from not releasing memory that's no longer needed, e.g. Valgrind.

In garbage collected languages, e.g. Python, still useful to use a memory profiler: as long as there are references to an object in memory, the object won't be garbage collected.

#### Event profiling

Can ignore specifics of code, a bit like using `strace`, and use `perf` to profile code instead, treating program as a black box, to report system events related to programs.

#### Visualisation

Profilers output lots of information. Useful to represent visually, e.g. flame graphs, call graphs.

#### Resource monitoring

Programs often run slowly when resource constrained, e.g. not enough memory or slow network. Examples of tools:

* General monitoring: `htop`
* I/O: `iotop`
* Disk usage: `df`, `du`
* Memory usage: `free`
* Open files: `lsof`
* Network connections and config: `ss`, `ip`
* Network usage: `nethogs`

Can stress a machine by `stress` and carry out black box benchmarking with `hyperfine`.

Web developer tools useful for profiling web page loading.

## Extra notes I made on cgroups for one of the exercises

Exercise was to restrict CPU and memory usage for `stress`.

cgroups, control groups
`/sys/fs/cgroup`


* Control resources
* Can use `cgcreate`: `sudo cgcreate -a <user> -t <user> -g memory,cpuset:stresslimit`
* Can use `cgexec`: `cgexec -g memory,cpuset:stresslimit stress -c 3`

Can set values inside `/sys/fs/cgroup/cpuset/stresslimit`

```sh
$ cat cpuset.cpus
0,2
$ cat cpuset.mems # this also has to be set
0
```

`/sys/fs/cgroup/memory/stresslimit`

From [kernel.org docs](https://www.kernel.org/doc/Documentation/cgroup-v1/memory.txt):

```
 memory.limit_in_bytes           # set/show limit of memory usage
 memory.memsw.limit_in_bytes     # set/show limit of memory+Swap usage
```

However, `memory.memsw.limit_in_bytes` doesn't seem to exist in Ubuntu.

# Lecture 8: Metaprogramming

By "metaprogramming" mean things that are more about process than writing code (actually also a term for programs that operate on programs).

## Build systems

For example, writing a LaTeX paper, what commands are need to produce the paper? What about running benchmarks, plotting them and inserting the plot into the paper? Or compiling code and running tests?

Most projects, whether they contain code or not, have a "build process": some sequence of operations to go from your inputs to outputs. May have many steps, e.g. run this to generate a plot, run that to generate results and something else to produce a paper.

"Build systems" are tools that can help. Many of them: which you use might depend on the task, language you are using and size of project. All similar in operation. Define a number of *dependencies*, *targets* and *rules* for going from dependencies to targets. Tell the build system that you want a particular target and its job is to find all the transitive dependencies of the target, and apply the rules to produce intermediate targets until the final target has been produced. Ideally the build system does this without executing rules for targets whose dependencies haven't changed and where the result is available from a previous buld.

### `make`

`make` is a very common build system, installed on almost any *nix OS. Works well for simple to moderate projects but does have its warts.

When you run `make`, it consults a file called `Makefile` in the current directory. All the targets, dependencies and rules are defined in that file.

Example:

```make
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@
```

Each section of this file is a rule. Each target in this file, on the left of a colon, specifies its dependencies on the right. The indented block, a *recipe*, is a series of commands to produce the target from the dependencies.

For `make`, the first target is the default: run `make` without arguments and it will build this target. Or you can run e.g. `make plot-data.png` and it will build that instead.

The `%` in a rule is a "pattern" and matches on the left and on the right. For example, if `plot-foo.png` is requested, `make` will look for dependencies `foo.dat` and `plot.py`.

Running `make` with an empty directory:

```
make: *** No rule to make target 'paper.tex', needed by 'paper.pdf'.  Stop.
```

So, to make `paper.pdf` we need `paper.tex`. There is no rule to specify how to make `paper.tex`. So, we need to provide it:

```sh
$ touch paper.tex
$ make
make: *** No rule to make target 'plot-data.png', needed by 'paper.pdf'.  Stop
```

There is a rule to make `plot-data.png` but is a pattern rule. Since the source files including `data.dat` don't exist, `make` states it cannot make the file.

So, create the files:

```
$ cat paper.tex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
\includegraphics[scale=0.65]{plot-data.png}
\end{document}
$ cat plot.py
#!/usr/bin/env python
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('-i', type=argparse.FileType('r'))
parser.add_argument('-o')
args = parser.parse_args()

data = np.loadtxt(args.i)
plt.plot(data[:, 0], data[:, 1])
plt.savefig(args.o)
$ cat data.dat
1 1
2 2
3 3
4 4
5 8
```

Now `make` works, producing `paper.pdf`:

```sh
$ make
./plot.py -i data.dat -o plot-data.png
pdflatex paper.tex
... lots of output ...
```

If we run `make` again, it doesn't do anything because it checks all of the previously built targets and they are all up-to-date with respect to their listed dependencies.

```sh
$ make
make: 'paper.pdf' is up to date.
```

If e.g. `paper.tex` changes, then running `make` will rebuild `paper.pdf`.

## Dependency management

Software projects often have dependencies on other software projects. You might depend on:

* Installed programs (e.g. `python`)
* System packages (e.g. `openssl`)
* Libraries in your programming language (e.g. `matplotlib`)

Most dependencies are available through a repository that hosts many dependencies in one place and often there is some convenient method to install them, e.g.

* `apt` for Ubuntu packages
* RubyGems for Ruby libraries
* PyPI for Python libraries
* Arch User Repository for Arch Linux user-contributed packages

Mechanisms for installing vary, but often share common ideas.

### Versioning

Most projects that other projects depend on issue a version number with each release. They are often, but not always, numerical. Serve many purposes, especially to ensure that software keeps working.

If someone releases a new version of their library where they rename a function, someone else's software will break if they already depend on that library and use that function; it will no longer exist. Versioning tries to solve this problem by letting a project say it depends on a particular version or range of versions of another project. So, if the underlying library changes, dependent software just continues to use an older version of the library.

This isn't ideal because the underlying library may have a security update that doesn't change the underlying public interface (its API) and which any project using the library should immediately be using. This is where different parts of a version number are important in semantic versioning. In semantic versioning, each version number is of the form major.minor.patch.

The rules are:

* If a new release does not change the API, increase the patch version.
* If you add to your API in a backwards-compatible way, increase the minor version.
* If you change the API in a non-backwards-compatible way, increase the major version.

This gives a means to specify what version of a library to use. Should be able to use the same major version, with an equal or higher minor version, without changing your code.

#### Lock files

Lock files are files specifying the versions your code currently depends on. Often need to run a program to upgrade to newer dependency versions. Many reasons for this including avoiding recompiles, having reproducible builds and not automatically upgrading to the latest version, which could be broken.

#### Vendoring

Vendoring goes even further in locking dependencies by copying all the code of your dependencies into the project. This gives you total control over changes to it, and also allows you to make your own changes easily, but also means you have to pull in updates from the upstream maintainers over time.

## Continuous integration systems

On larger projects, may find additional tasks to do when you make a change to it. For example:

* Upload a new documentation version
* Upload a compiled version somewhere
* Release the code to PyPI
* Run your test suite

Maybe you want pull requests to be style checked and benchmarks to run. Continuous integration (CI) systems can help.

Many companies provide various types of CI, e.g. Azure Pipelines, Travis CI and GitHub Actions. All work roughly in the same way: add a file to your repository to describe what should happen when events happen in that repository. A common one is "when someone pushes code, run the tests". The CI provider then spins up a virtual machine (or more), runs your "recipe" and notes the results somewhere.

Can use CI to build a project entirely, e.g. publishing a web site whose content is written in Markdown.

## Testing

Most large software projects come with a "test suite". Terms you might encounter:

* *Test suite*: a collective term for all the tests
* *Unit test*: a "micro-test" that tests a specific feature in isolation
* *Integration test*: a "macro-test" that runs a larger part of the system to check that different feature or components work together.
* *Regression test*: a test that implements a particular pattern that previously caused a bug to ensure that the bug does not resurface.
* *Mocking*: replacing a function, module, or type with a fake implementation to avoid testing unrelated functionality. For example, you might "mock the network" or "mock the disk".

# Lecture 9: Security and Cryptography

## Entropy

Entropy is a measure of randomness. Useful when determining password strength. How is it quantified?

It is measured in *bits* and when selecting uniformly at random from a set of possible outcomes, it is equal to the base 2 logarithm of the number of possibilities. A fair coin flip has 1 bit of entropy; a 6-sided die roll has ~2.58 bits of entropy.

You should consider that the attacker knows the *model* of the password, but not the randomness.

How many bits of entropy is enough for a password? Depends on your threat model: for online guessing (e.g. that might be rate limited) you may be happy with fewer than if you think that it is likely that offline guessing is possible.

## Hash functions

Cryptographic hash functions map data of arbitrary size to a fixed size, e.g. SHA-1 as used in Git (though SHA-1 is no longer recommended). Maps arbitrary-sized inputs to 160-bit outputs (that can be represented by 40 hexadecimal digits).

Cryptographic hash functions can be thought of as hard-to-invert random-looking (but deterministic) functions. (The ideal model of a hash function is a "random oracle".)

Cryptographic hash functions have the following properties:
* Deterministic: same input always generates the same output.
* Non-invertible: hard to find an input `m` such that `hash(m) = h` for some desired output `h`.
* Target collision resistant: given an input `m_1`, it's hard to find a different input `m_2` such that `hash(m_1) = hash(m_2)`.
* Collision resistant: hard to find two inputs such that `hash(m_1) = hash(m_2)`; a stronger property than target collision resistance.

Implicit in the notes, but not stated is the property that two similar inputs should give very different outputs. Their example is SHA-1 of `hello` versus `Hello`.

### Applications

* Git, for content-addressed storage. The idea of a hash function is more general than in cryptography.
* Short summary of a file's contents. Allowing you to e.g. download a file from a mirror, and verifying the hash from the original source.
* Commitment schemes. E.g. allows you to make a coin toss under this scenario: one person flips a coin in their head by picking a random number, where even is defined as heads and odd as tails, finding the hash, sharing the hash, and the other person guesses heads or tails. After the guess, the coin flipper can share the original random value with the guesser and they can verify the result and confirm the flipper hasn't cheated.

## Key derivation functions (KDFs)

Related to hash functions, KDFs produced fixed-length output for uses as keys in other cryptographic applications. Often KDFs are deliberately slow to limit offline brute force attacks. Since the algorithm is otherwised used relatively rarely, being somewhat slow is not a severe problem. (This is notable because usually the ideal is to have algorithms that are fast.)

Can use a passphrase as input to the KDF, then use the output from the KDF.

### Applications

* Producing keys from passphrases for use in other cryptographic algorithms (e.g. symmetric cryptography).
* Storing login credentials. Storing plaintext passwords is bad. Better to generate and store a random salt for each user, store the output of the KDF on the password + salt and verify login attempts by computing the output of the KDF on the input + salt.

## Symmetric cryptography


e.g. AES.

Scheme is composed of the following functions:

```
keygen() -> key  (this function is randomized)

encrypt(plaintext: array<byte>, key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, key) -> array<byte>  (the plaintext)
```

`encrypt()` has the property that given the output, it is hard to determine the input without the key. `decrypt()` has the correctness property that `decrypt(encrypt(m, k), k) = m`.

Note that securely distributing the key is important.

### Applications

* Encrypting files for storage in an untrusted cloud service. Can be combined with KDFs. Store the output of `encrypt(file, key)` and store the key elsewhere.

## Asymmetric cryptography

Asymmetric: two keys, two roles:

* Private key, keep private.
* Public key, can distribute and doesn't affect security (unlike in a symmetric cryptosystem).

Scheme:

```
keygen() -> (public key, private key)  (this function is randomized)

encrypt(plaintext: array<byte>, public key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, private key) -> array<byte>  (the plaintext)

sign(message: array<byte>, private key) -> array<byte>  (the signature)
verify(message: array<byte>, signature: array<byte>, public key) -> bool  (whether or not the signature is valid)
```

Can encrypt/decrypt or sign/verify.

`encrypt()` and `decrypt()` have same properties as those in symmetric cryptosystems.

`sign()` and `verify()` have a similar use as written signatures: they provide proof something was created by a particular author.

## Lock analogies for symmetric and asymmetric cryptosystems

Can compare the two approaches to physical locks:

* Symmetric cryptosystems are like having a door lock, where anyone with the key can unlock it or lock it.
* Asymmetric cryptosystems are like a padlock: someone can provide an open padlock (the public key) but, when it is locked, only the private key holder can open the lock again.

### Applications

* Email encryption; post public keys and receive encrypted email.
* Private messaging.
* Signing software.

### Key distribution

Asymmetric cryptography avoids the problem of sharing keys that symmetric cryptography has. Instead, there is a challenge of mapping public keys to real-world identities. Different solutions, e.g. Signal uses trust on first use by supporting out-of-band public key exchange. PGP usees web of trust. Keybase did this by using social media accounts.

## Hybrid encryption

Hybrid encryption combines symmetric and asymmetric cryptography.

Asymmetric cryptography is slow, symmetric cryptography is faster. Therefore use asymmetric cryptography to easily distribute a symmetric key initially and then use only the symmetric key to transmit information more quickly.

## Case studies

### Password managers

Use unique, randomly generated high-entropy passwords and save all passwords in one place, symmetrically encrypted with a key produced from a passphrase using a KDF. Only need to remember a single high-entropy password, avoiding password reuse.

### Full disk encryption

Protect your data. Encrypt with a symmetric cipher with a key protected by a passphrase.

### SSH

`ssh-keygen` generates an asymmetric keypair, randomly using entropy provided by the operating system. Public key is stored as is; private key should be encrypted with a passphrase: `ssh-keygen` prompts for passphrase, used in KDF to produce a key which encrypts the private key.

Once the server knows the client's public key (stored in `.ssh/authorized_keys` on the server), a connecting client can prove its identify using asymmetric signatures. This is done via challenge-response. At a high level, the server picks a random number and sends it to the client. The client signs this message and sends the signature back to the server, which checks the signature against the public key on record. This effectively proves that the client has the private key corresponding to the public key that the server has stored.
