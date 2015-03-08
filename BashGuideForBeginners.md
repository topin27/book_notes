--------------------------------------------------------------------------------

## 1. Bash and Bash scripts

### 1.2. Advantages of the Bourne Again Shell

#### 1.2.2. Features only found in bash

##### 1.2.2.2. Bash startup files

A *interactive login shell* means that you got the shell after authenticating 
to the system, files read:
	* `/etc/profile`
	* `~/.bash_profile`, `~/.bash_login` or `~/.profile`
	* `~/.bash_logout` upon logout.

A *interactive non-login shell* means that you did not have to authenticate to 
the system, for instance, when you open a terminal, that is a non-login shell. 
Files read: 
	* `~/.bashrc`, this file is usually referred to in `~/.bash_profile`


##### 1.2.2.5. Shell arithmetic

The shell allows arithmetic expressions to be evaluated, as one of the shell 
expansions or by the `let` built-in.


##### 1.2.2.7. Arrays

Bash provides one-dimensional array variables. Any variable may be used as an 
array; the `declare` built-in will explicitly declare an array.


##### 1.2.2.8. Directory stack

The directory stack is a list of recently-visited directories. The `pushd` 
built-in adds directories to the stack as it changes the current directory, and 
the `popd` built-in removes specified directories from the stack and change the 
current directory to the directory removed. Content can be displayed issuing 
the `dirs` command.


##### 1.2.2.10. The restricted shell

When invoked as `rbash` or with the `-r` option.


### 1.3. Executing commands

#### 1.3.2. Shell built-in commands

Built-in commands are contained within the shell itself. When the name of a 
built-in command is used as the first word of a simple command, the shell exec-
utes the command directly, without creating a new process.


### 1.4. Building blocks

#### 1.4.1. Shell building blocks

##### 1.4.1.7. Executing commands

An important part of the tasks of the shell is to search for commands. Bash 
does this as follows:
	* Check whether the command contains slashes. If not, first chech with 
	  the function list to see if it contains a command by the name we are 
	  looking for.

	* If command is not a function, check for it in the built-in list.

	* If command is neither a function nor a built-in, look for it 
	  analyzing the directories listed in `PATH`.

	* If the search is unsuccessful, bash prints an error message and 
	  returns an exit status of 127.


### 1.5. Developing good scripts

#### 1.5.2. Structure

When starting on a new script, ask yourself the following questions:
	* Will I be needing any information from the user or from the users 
	  environment?

	* How will I store that information?

	* Are there any files that need to be created? Where and with which 
	  permissions and ownerships?

	* What commands will I use? When using the script on different systems,
	  do all these systems have these commands in the required versions?

	* Does the user need any notifications? When and why?

#### 1.5.6. Example init script

An init script starts system services on UNIX and Linux machines. The system 
log daemon, the power management daemon, the name and mail daemons are common 
examples. These scripts, also known as startup scripts, are stored in a speci-
fic location on your system, such as "/etc/rc.d/init.d" or "/etc/init.d". Init, 
the initial process, reads its configuration files and decides which services 
to start or stop in each run level. A run level is a configuration of proces-
ses; each system has a single user run level, for instance, for performing 
administrative tasks, for which the system has to be in an unused state as much 
as possible, such as recovering a critical file system from a backup. Reboot 
and shutdown run levels are usually also configured.


--------------------------------------------------------------------------------

## 2. Writing and debugging scripts

### 2.1. Creating and running a script

#### 2.1.3. Executing the script

If you dont want to start a new shell but execute the script in the current 
shell, you `source` it:
	source script_name.sh
The script does not need execute permission in this case. Commands are executed 
in the current shell context, so any changes made to your environment will be 
visible when the script finishes execution.


### 2.3. Debugging Bash scripts

#### 2.3.1. Debugging on the entire script

The most common is to start up the subshell with the `-x` option, which will 
run the entire script in debug mode. Taces of each command plus its arguments 
are printed to standard output after the commands have been expanded but before 
they are executed.


#### 2.3.2. Debugging on part(s) of the script

Using the `set` Bash built-in you can run in normal mode those portions of the 
script of which you are sure they are without fault, and display debugging inf-
ormation only for troublesome zones. Say we are not sure what the `w` command 
will do, then we could enclose it in the script like this:
	set -x 	# activeate debugging from here
	w
	set +x	# stop debugging from here


--------------------------------------------------------------------------------

## 3. The Bash environment

### 3.1.1. System-wide configuration files

#### 3.1.1.1. /etc/profile

When invoked interactively with the `--login` option or when invoked as `sh`, 
Bash reads the `/etc/profile` instructions. These usually set the shell 
variable PATH, USER, MAIL, HOSTNAME and HISTSIZE.


#### 3.1.1.2. /etc/bashrc


### 3.1.2. Individual user configuration files

#### 3.1.2.1. ~/.bash_profile

#### 3.1.2.2. ~/.bash_login

#### 3.1.2.3. ~/.profile

#### 3.1.2.4. ~/.bashrc

Today, it is more common to use a non-login shell, for instance when logged in 
graphically using X terminal windows. Upon opening such a window, the user does 
not have to provide a user name or password; no authentication is done. Bash 
searches for `~/.bashrc` when this happens, so it is referred to in the files 
read upon login as well, which means you do not have to enter the same settings 
in multiple files.


#### 3.1.2.5. ~/.bash_logout


### 3.2. Variables

#### 3.2.1. Types of variables

Bash keeps a list of two types of variables: Global variables and Local 
variables.


##### 3.2.1.1. Global variables

Global variables are available in all shells. The `env` or `printenv` commands 
can be used to display environment variables.


##### 3.2.1.2. Local variables

Local variables are only available in the current shell. Using the `set` built-
in command without any options will display a list of all variables(including 
environment variables) and functions.


##### 3.2.1.3. Variables by content

According to the sort of content the variables contains, variables come in 4 
types:
	* String variables

	* Integer variables

	* Constant variables

	* Array variables


#### 3.2.2. Creating variables

To set a variable in the shell, use:
	VARNAME="value"
Putting spaces around the equal sign will cause errors.


#### 3.2.3. Exporting variables

A variable created like the ones in the example above is only available to the 
current shell. Child processes of the current shell will not be aware of this 
variable. In order to pass variables to a subshell, we need to *export* them 
using the export built-in command.

A subshell can change variables it  inherited from the parent, but the changes 
made by the child dont affect the parent.


#### 3.2.5. Special parameters

* $* -

* $@ - Expands to the positional parameters, starting from one. When the 
       expansion occurs within double quotes, each parameter expands to a 
       separate word.

* $# - Expands to the number of positional parameters in decimal.

* $? - Expands to the exit status of the most recent executed foreground 
       pipeline.

* $- -

* $$ - Expands to the process ID of the shell.

* $! - Expands to the process ID of the most recently executed backgroud (
       asynchronous) command.

* $0 - Expands to the name of the shell or shell script.

* $_ - "> grep dictionary /usr/share/dict/words
       > echo $_
       /usr/share/dict/words"

The positional parameters are the words following the name of a shell script. 
They are put into the variables "$1, $2, $3" and so on. "$#" holds the total 
number of parameters.


### 3.3. Quoting characters

#### 3.3.3. Single quotes

Single quotes('') are used to preserve the literal value of each character 
enclosed within quotes. A single quote may not occur between single quotes, 
even when preceded by a backslash.


#### 3.3.4. Double quotes

Using double quotes the literal value of all characters enclosed is preserved, 
except for the dollar sign, the backticks(backward single quotes, ``) and the 
backslash. The dollar sign and the backticks retain their special meaning 
within the double quotes. The backslash retains its meaning only when followed 
by dollar, double quote, backslash or newline.


### 3.4. Shell expansion

#### 3.4.2. Brace expansion

Brace expansions may be nested. The results of each expanded string are not 
sorted; left to right order is preserved:
	> echo sp{el,il,al} l
	spell spill spall
Brace expansion is performed before any other expansions, and any characters 
special to other expansions are preserved in the result.


#### 3.4.3. Tilde expansion

#### 3.4.4. Shell parameter and variable expansion

The basic form of parameter expansion is "${PARAMETER}". The value of "PARAME-
TER" is substituted. The braces are required when "PARAMETER" is a positional 
parameter with more than one digit, or when "PARAMETER" is followed by a 
character that is not to be interpreted as part of its name.

If the first character of "PARAMETER" is an exclamtion point, Bash uses the 
value of the variable formed from the rest of "PARAMETER" as the name of the 
variable; this variable is then expanded and that value is used in the rest of 
the substitution.
	> echo ${!N*}
	NNTPPORT NNTPSERVER NPX_PLUGIN_PATH


#### 3.4.5. Command substitution

Command substitution allows the output of a command to replace the command its-
elf. Command substitution occurs when a command is enclosed like this:
	$(command) or `command`


#### 3.4.6. Arithmetic expansion

Arithmetic expansion allows the evaluation of an arithmetic expression and the 
substitution of the result. The format for arithmetic expansion is:
	$((EXPRESSION))
The expression is treated as if it were within double quotes, but a double 
quote inside the parentheses is not treated specially.


#### 3.4.7. Process substitution

Process substitution is supported on systems that support named pipes(FIFOs) or 
the "/dev/fd" method of naming open files. It takes the form of
	<(LIST) or >(LIST)
The process "LIST" is run with its input or output connected to a FIFO or some 
file in "/dev/fd". The name of this file is passed as an argument to the 
current command as the result of the expansion.


#### 3.4.8. Word splitting

The shell treats each character of "$IFS" as a delimiter, and splits the 
results of the other expansions into words on these characters. If "IFS" is 
unset, or its value is exactly "<space><tab><new line>".


#### 3.4.9. File name expansion

After word splitting, unless the "-f" option has been set, Bash scans each word 
for the characters "*", "?", and "[". If one of these characters appears, then 
the word is regarded as a *PATTERN*, and replaced with an alphabetically sorted 
lsit of file names matching the pattern.


### 3.5. Aliases

#### 3.5.1. What are aliases?

An alias allows a string to be substituted for a word when it is used as the 
first word of a simple command.

Aliases are not expanded when the shell is not interactive, unless the 
"expand_aliases" option is set using the `shopt` shell built-in.

To be safe, always put alias definitions on a separate line, and do not use 
"alias" in compound commands.

While aliases are easier to understand, shell functions are preferred over 
aliases for almost every purpose.


### 3.6. More Bash options

#### 3.6.1. Displaying options

Use the "-o" option to `set` to display all shell options.


#### 3.6.2. Changing options

For changing the current environment temporarily, or for use in a script, we 
would rather use `set`, use "-" for enabling an option, "+" for disabling:
	set -o noclobber
	set +o noclobber
