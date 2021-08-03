# Windows Terminal

## A word of advice

WSL documantation talks about "distros", where they should have used the word "instances".
With WSL tooling you can copy a "distro" and rename it which makes zero sense from a semantics perspective.
It would be cleaner if they were called instances fron the get go but what do you expect from MS? *shrug*

## list distros

```bash 
wsl -l
```

## Backup distro

- goto Windows user folder
- list the distros so you can copy the name for improved accuracy
- Replace Distro with the name you just copied and type some temp file path such as .\linuxbackup.tar 

```bash 
wsl --export <Distro> <FileName>
```

## Duplicate WSL distro

- backup a distro that you want to duplicate as described in this doc, don't forget where you put the file.
- Replace Distro with some new name, install location can theoretically be any path but don't be sloppy, and type the temp filepath that you typed in while backing up

```bash 
wsl --import <Distro> <InstallLocation> <FileName>
```

## Force windows terminal to start Linux distros properly

- replace following placeholders with appropriate values
- modify start command to `wsl.exe -d <linux distro> -u <linux username>`
- modify start folder to `\\wsl$\<linux distro>\home\<linux username>`
