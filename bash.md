# Bash

##debugger

http://bashdb.sourceforge.net/

## Print active shell interpreter in a terminal window with current process placeholder $0

echo $0

## print status code of last command with ?

echo $?

## Directory stack

The directory stack is a list of recently-visited directories. The `pushd` built-in adds directories to the stack as it changes the current directory, and the `popd` built-in removes specified directories from the stack and changes the current directory to the directory removed.

Content can be displayed issuing the `dirs` command or by checking the content of the `DIRSTACK` variable.
