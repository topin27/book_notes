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


