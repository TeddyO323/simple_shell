<p><img src="https://s3.amazonaws.com/intranet-projects-files/holbertonschool-low_level_programming/235/shell.jpeg" alt="" style="" /></p>

## Resources

<p><strong>Read or watch</strong>:</p>

<ul>
<li><a href="https://en.wikipedia.org/wiki/Unix_shell" title="Unix shell" target="_blank">Unix shell</a> </li>
<li><a href="https://en.wikipedia.org/wiki/Thompson_shell" title="Thompson shell" target="_blank">Thompson shell</a> </li>
<li><a href="https://en.wikipedia.org/wiki/Ken_Thompson" title="Ken Thompson" target="_blank">Ken Thompson</a> </li>
<li><strong>Everything you need to know to start coding your own shell</strong> concept page</li>
</ul>

<p><strong>man or help</strong>: </p>

<ul>
<li><code>sh</code> (<em>Run <code>sh</code> as well</em>)</li>
</ul>

## Description :speech_balloon:

**Shellby** is a simple UNIX command language interpreter that reads commands from either a file or standard input and executes them.

### Invocation :running:

Usage: **shellby** [filename]

To invoke **shellby**, compile all `.c` files in the repository and run the resulting executable:

```
gcc *.c -o shellby
./shellby
```

**Shellby** can be invoked both interactively and non-interactively. If **shellby** is invoked with standard input not connected to a terminal, it reads and executes received commands in order.

Example:
```
$ echo "echo 'hello'" | ./shellby
'hello'
$
```

If **shellby** is invoked with standard input connected to a terminal (determined by [isatty](https://linux.die.net/man/3/isatty)(3)), an *interactive* shell is opened. When executing interactively, **shellby** displays the prompt `$ ` when it is ready to read a command.

Example:
```
$./shellby
$
```

Alternatively, if command line arguments are supplied upon invocation, **shellby** treats the first argument as a file from which to read commands. The supplied file should contain one command per line. **Shellby** runs each of the commands contained in the file in order before exiting.

Example:
```
$ cat test
echo 'hello'
$ ./shellby test
'hello'
$
```

### Environment :deciduous_tree:

Upon invocation, **shellby** receives and copies the environment of the parent process in which it was executed. This environment is an array of *name-value* strings describing variables in the format *NAME=VALUE*. A few key environmental variables are:

#### HOME
The home directory of the current user and the default directory argument for the **cd** builtin command.

```
$ echo "echo $HOME" | ./shellby
/home/vagrant
```

#### PWD
The current working directory as set by the **cd** command.

```
$ echo "echo $PWD" | ./shellby
/home/vagrant/holberton/simple_shell
```

#### OLDPWD
The previous working directory as set by the **cd** command.

```
$ echo "echo $OLDPWD" | ./shellby
/home/vagrant/holberton/printf
```

#### PATH
A colon-separated list of directories in which the shell looks for commands. A null directory name in the path (represented by any of two adjacent colons, an initial colon, or a trailing colon) indicates the current directory.

```
$ echo "echo $PATH" | ./shellby
/home/vagrant/.cargo/bin:/home/vagrant/.local/bin:/home/vagrant/.rbenv/plugins/ruby-build/bin:/home/vagrant/.rbenv/shims:/home/vagrant/.rbenv/bin:/home/vagrant/.nvm/versions/node/v10.15.3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/home/vagrant/.cargo/bin:/home/vagrant/workflow:/home/vagrant/.local/bin
```

### Command Execution :hocho:

After receiving a command, **shellby** tokenizes it into words using `" "` as a delimiter. The first word is considered the command and all remaining words are considered arguments to that command. **Shellby** then proceeds with the following actions:
1. If the first character of the command is neither a slash (`\`) nor dot (`.`), the shell searches for it in the list of shell builtins. If there exists a builtin by that name, the builtin is invoked.
2. If the first character of the command is none of a slash (`\`), dot (`.`), nor builtin, **shellby** searches each element of the **PATH** environmental variable for a directory containing an executable file by that name.
3. If the first character of the command is a slash (`\`) or dot (`.`) or either of the above searches was successful, the shell executes the named program with any remaining given arguments in a separate execution environment.

### Exit Status :wave:

**Shellby** returns the exit status of the last command executed, with zero indicating success and non-zero indicating failure.

If a command is not found, the return status is `127`; if a command is found but is not executable, the return status is 126.

All builtins return zero on success and one or two on incorrect usage (indicated by a corresponding error message).

### Signals :exclamation:

While running in interactive mode, **shellby** ignores the keyboard input `Ctrl+c`. Alternatively, an input of end-of-file (`Ctrl+d`) will exit the program.

User hits `Ctrl+d` in the third line.
```
$ ./shellby
$ ^C
$ ^C
$
```

### Variable Replacement :heavy_dollar_sign:

**Shellby** interprets the `$` character for variable replacement.

#### $ENV_VARIABLE
`ENV_VARIABLE` is substituted with its value.

Example:
```
$ echo "echo $PWD" | ./shellby
/home/vagrant/holberton/simple_shell
```

#### $?
`?` is substitued with the return value of the last program executed.

Example:
```
$ echo "echo $?" | ./shellby
0
```

#### $$
The second `$` is substitued with the current process ID.

Example:
```
$ echo "echo $$" | ./shellby
6494
```

### Comments :hash:

**Shellby** ignores all words and characters preceeded by a `#` character on a line.

Example:
```
$ echo "echo 'hello' #this will be ignored!" | ./shellby
'hello'
```

### Operators :guitar:

**Shellby** specially interprets the following operator characters:

#### ; - Command separator
Commands separated by a `;` are executed sequentially.

Example:
```
$ echo "echo 'hello' ; echo 'world'" | ./shellby
'hello'
'world'
```

#### && - AND logical operator
`command1 && command2`: `command2` is executed if, and only if, `command1` returns an exit status of zero.

Example:
```
$ echo "error! && echo 'hello'" | ./shellby
./shellby: 1: error!: not found
$ echo "echo 'all good' && echo 'hello'" | ./shellby
'all good'
'hello'
```

#### || - OR logical operator
`command1 || command2`: `command2` is executed if, and only if, `command1` returns a non-zero exit status.

Example:
```
$ echo "error! || echo 'but still runs'" | ./shellby
./shellby: 1: error!: not found
'but still runs'
```

The operators `&&` and `||` have equal precedence, followed by `;`.

### Shellby Builtin Commands :nut_and_bolt:

#### cd
  * Usage: `cd [DIRECTORY]`
  * Changes the current directory of the process to `DIRECTORY`.
  * If no argument is given, the command is interpreted as `cd $HOME`.
  * If the argument `-` is given, the command is interpreted as `cd $OLDPWD` and the pathname of the new working directory is printed to standad output.
  * If the argument, `--` is given, the command is interpreted as `cd $OLDPWD` but the pathname of the new working directory is not printed.
  * The environment variables `PWD` and `OLDPWD` are updated after a change of directory.

Example:
```
$ ./shellby
$ pwd
/home/vagrant/holberton/simple_shell
$ cd ../
$ pwd
/home/vagrant/holberton
$ cd -
$ pwd
/home/vagrant/holberton/simple_shell
```

#### alias
  * Usage: `alias [NAME[='VALUE'] ...]`
  * Handles aliases.
  * `alias`: Prints a list of all aliases, one per line, in the form `NAME='VALUE'`.
  * `alias NAME [NAME2 ...]`: Prints the aliases `NAME`, `NAME2`, etc. one per line, in the form `NAME='VALUE'`.
  * `alias NAME='VALUE' [...]`: Defines an alias for each `NAME` whose `VALUE` is given. If `name` is already an alias, its value is replaced with `VALUE`.

Example:
```
$ ./shellby
$ alias show=ls
$ show
AUTHORS            builtins_help_2.c  errors.c         linkedlist.c        shell.h       test
README.md          env_builtins.c     getline.c        locate.c            shellby
alias_builtins.c   environ.c          helper.c         main.c              split.c
builtin.c          err_msgs1.c        helpers_2.c      man_1_simple_shell  str_funcs1.c
builtins_help_1.c  err_msgs2.c        input_helpers.c  proc_file_comm.c    str_funcs2.c
```

#### exit
  * Usage: `exit [STATUS]`
  * Exits the shell.
  * The `STATUS` argument is the integer used to exit the shell.
  * If no argument is given, the command is interpreted as `exit 0`.

Example:
```
$ ./shellby
$ exit
```

#### env
  * Usage: `env`
  * Prints the current environment.

Example:
```
$ ./shellby
$ env
NVM_DIR=/home/vagrant/.nvm
...
```

#### setenv
  * Usage: `setenv [VARIABLE] [VALUE]`
  * Initializes a new environment variable, or modifies an existing one.
  * Upon failure, prints a message to `stderr`.

Example:
```
$ ./shellby
$ setenv NAME Poppy
$ echo $NAME
Poppy
```

#### unsetenv
  * Usage: `unsetenv [VARIABLE]`
  * Removes an environmental variable.
  * Upon failure, prints a message to `stderr`.

Example:
```
$ ./shellby
$ setenv NAME Poppy
$ unsetenv NAME
$ echo $NAME

$
```

## Authors :
* Nobert Ayesiga
* Divine Chisom Ukonu:

