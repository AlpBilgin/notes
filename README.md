# notes

## New install breadcrumbs

Read sibling sections for tips on the listed tools

### Terminal Emulator

NTFS: cmder vanilla || git bash
Posix: zsh + oh-my-zsh powerlevel9k theme

### Node-npm
 1
 `npm prefix -g` to see current global install prefix
 `mkdir ~/.npm-global` to create a new root for global node_modules
 `npm config set prefix '~/.npm-global'` to set it as new global install prefix
 export it to environment `export PATH=~/.npm-global/bin:$PATH`
 2
 install pnpm. Then use pnpm to install n. Use npm only when something breaks, pnpm will never be 100% there
3
node-gyp needs python 2.7 somewhere in the environment

MSB error=>  https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2017 ; npm config set msvs_version 2017 --global

### Python

Read up again on the python package (py-env?) that exposes different versions per project.

### Git
1 setup identity

`git config --global user.name "John Doe"`
`git config --global user.email johndoe@example.com`

2 make a given executable the default for commit mesages

`git config --global core.editor "nano"`

## macOS

### From SO

1

show hidden: Just adding the key to the global domain seems to work:
`defaults write -g AppleShowAllFiles -bool true`
You have to quit and reopen applications to apply changes as usual.

2

To start screensharing:

`sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.screensharing.plist`

To stop:

`sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.screensharing.plist`

The -w flag modifies the Disabled key as appropriate. It's best to let launchctl handle this, as the location where the config files are stored has changed a bit between OS versions.


## Node.JS

### peripherals

1

Avoid npm:

https://www.npmjs.com/package/pnpm

## XCODE

### pick an Xcode version
https://xcodereleases.com/

### official ref for simulator
https://help.apple.com/simulator/mac/current/#/


## QA Solutions

### Intelligent test scripter
https://github.com/CognizantQAHub/Cognizant-Intelligent-Test-Scripter

### Arjuna
https://github.com/test-mile/arjuna

### Testlink

#### tarball
https://sourceforge.net/projects/testlink/files/latest/download?source=files
#### Container
https://bitnami.com/stack/testlink
https://github.com/bitnami/bitnami-docker-testlink
#### Docu
https://www.softwaretestinghelp.com/testlink-tutorial-1/
http://testlink.sourceforge.net/docs/documents/end-users/manual.html
https://docs.bitnami.com/installer/apps/testlink/configuration/configure-smtp/
#### actionable tips
- SSH into server
- use `docker ps` to find out the proper container id
- use `docker exec -it <mycontainerid> bash` to bash into the container
- goto /bitnami/testlink edit custom_config.inc.php
- use `apt-get install nano` to download a temporary shell editor if there isn't any (vim, ed etc are also okay)
- edit values
- save and exit editor, exit shell, docker-compose down, docker-compose up -d
- check website to see if it still works
- ```docker run --rm --volumes-from bitnami-docker-testlink_mariadb_1 -v $(pwd):/backup ubuntu tar cvf /backup/mariadb_backup.tar /bitnami``` AND ```docker run --rm --volumes-from bitnami-docker-testlink_testlink_1 -v $(pwd):/backup ubuntu tar cvf /backup/testlink_backup.tar /bitnami``` => will extract backups
- ```docker run --rm --volumes-from bitnami-docker-testlink_testlink_1 -v $(pwd):/backup ubuntu bash -c "cd /bitnami && tar xvf /backup/testlink_backup.tar --strip 1"``` AND ```docker run --rm --volumes-from bitnami-docker-testlink_mariadb_1 -v $(pwd):/backup ubuntu bash -c "cd /bitnami && tar xvf /backup/mariadb_backup.tar --strip 1"``` => will reapply backups
-`docker-compose down`, `docker-compose up -d`
- check website to see if it still works

## Spring boot

Override default test properties through a side channel.

https://www.baeldung.com/spring-test-property-source
