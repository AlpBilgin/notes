# Bash

## debugger

http://bashdb.sourceforge.net/

## Print active shell interpreter in a terminal window with current process placeholder $0

echo $0

## print status code of last command with ?

echo $?

## Directory stack

The directory stack is a list of recently-visited directories. The `pushd` built-in adds directories to the stack as it changes the current directory, and the `popd` built-in removes specified directories from the stack and changes the current directory to the directory removed.

Content can be displayed issuing the `dirs` command or by checking the content of the `DIRSTACK` variable.


## tips and tricks

### create a background process and detach



First of all; once you've started a process, you can background it by first stopping it (hit Ctrl-Z) and then typing bg to let it resume in the background. It's now a "job", and its stdout/stderr/stdin are still connected to your terminal.

You can start a process as backgrounded immediately by appending a "&" to the end of it:

firefox &

To run it in the background silenced, use this:

firefox </dev/null &>/dev/null &

Some additional info:

nohup is a program you can use to run your application with such that its stdout/stderr can be sent to a file instead and such that closing the parent script won't SIGHUP the child. However, you need to have had the foresight to have used it before you started the application. Because of the way nohup works, you can't just apply it to a running process.

disown is a bash builtin that removes a shell job from the shell's job list. What this basically means is that you can't use fg, bg on it anymore, but more importantly, when you close your shell it won't hang or send a SIGHUP to that child anymore. Unlike nohup, disown is used after the process has been launched and backgrounded.

What you can't do, is change the stdout/stderr/stdin of a process after having launched it. At least not from the shell. If you launch your process and tell it that its stdout is your terminal (which is what you do by default), then that process is configured to output to your terminal. Your shell has no business with the processes' FD setup, that's purely something the process itself manages. The process itself can decide whether to close its stdout/stderr/stdin or not, but you can't use your shell to force it to do so.

To manage a background process' output, you have plenty of options from scripts, "nohup" probably being the first to come to mind. But for interactive processes you start but forgot to silence (firefox < /dev/null &>/dev/null &) you can't do much, really.

I recommend you get GNU screen. With screen you can just close your running shell when the process' output becomes a bother and open a new one (^Ac).

Oh, and by the way, don't use "$@" where you're using it.

$@ means, $1, $2, $3 ..., which would turn your command into:

gnome-terminal -e "vim $1" "$2" "$3" ...

That's probably not what you want because -e only takes one argument. Use $1 to show that your script can only handle one argument.

It's really difficult to get multiple arguments working properly in the scenario that you gave (with the gnome-terminal -e) because -e takes only one argument, which is a shell command string. You'd have to encode your arguments into one. The best and most robust, but rather cludgy, way is like so:

gnome-terminal -e "vim $(printf "%q " "$@")"

### if conditional evaluators
\[] vs. \[\[]]
 	

Contrary to \[, \[\[ prevents word splitting of variable values. So, if VAR="var with spaces", you do not need to double quote $VAR in a test - eventhough using quotes remains a good habit. Also, \[\[ prevents pathname expansion, so literal strings with wildcards do not try to expand to filenames. Using \[\[, == and != interpret strings to the right as shell glob patterns to be matched against the value to the left, for instance: \[\[ "value" == val* ]].

### file descriptor 5

Using file descriptor 5 might cause problems. When Bash creates a child process, as with exec, the child inherits fd 5 (see Chet Ramey's archived e-mail, SUBJECT: RE: File descriptor 5 is held open). Best leave this particular fd alone.
