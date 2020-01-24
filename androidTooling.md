# Android SDK terminal operation guide

## How to set it up

### Get latest Ubuntu Apt repos
```
$ sudo apt update
$ sudo apt upgrade
```

### installing android sdk on server terminal
```
$ mkdir ~/android
$ cd ~/android
$ wget https://dl.google.com/android/repository/tools_r25.2.5-linux.zip
$ unzip tools_r25.2.5-linux.zip

$ nano ~/.profile
```

### and add following in opened .profile file:
```export ANDROID_HOME="/home/<user>/android"
export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin
```

### use new .profile:

`$ . ~/.profile`

### we want to be able to compile the code through gradle commands. And we need the build libraries

### initialize sdkmanager repo cfg by hand:
`$ nano /home/<user>/.android/repositories.cfg`

### and add following:
```### User Sources for Android SDK Manager
#Fri Nov 03 10:11:27 CET 2017 count=0
```

### both commands should work, accept licences when they come up:
```
$ sdkmanager --update
$ sdkmanager --licenses
```

### If sdkmanager can't be found ask someone who knows how to use shell for help

### use following to get individual SDK names available for download:
`$ sdkmanager --list`

### use following to update base platform tools AND download SDK tools specific to API 28 AND download the most basic emulator image:
`$ sdkmanager "platform-tools" "platforms;android-28" "system-images;android-28;default;x86" "system-images;android-28;default;x86_64"`

### Now we want to create an emulator with the data we just downloaded to run the compiled app on:
```
$ sdkmamager "emulator"
$ avdmanager create avd -n Nexus_5X_API_28 -k "system-images;android-28;default;x86_64"
```
### Now we want to see if the emulator can find the virtual device:
`$ emulator -list-avds`

### If you just saw the name in output you should be able to start the new virtual device:

`$ emulator -avd Nexus_5X_API_28`

### Linux may complain about audio and video drivers, installing the following base drivers MIGHT solve your problems, (there is no guarantee as badly written 3rd party driver interfaces is a known plague of Linux ecosystem)
```
$ sudo apt-get install libgl1
$ sudo apt-get install libpulse0
```
Goto the following pages to learn how all this works:

https://developer.android.com/studio/command-line/sdkmanager
https://developer.android.com/studio/command-line/avdmanager
https://developer.android.com/studio/run/emulator-commandline

## How to operate

### ADB

#### List running emulator instances

`adb devices | grep "emulator-"`

#### How to kill a single emulator

adb emu kill

#### How to kill a specific emulator instance when multiple are open

adb -s emulator-5554 emu kill

#### loop this command

```sh
adb devices | grep "emulator-" | while read -r emulator device; do # get list of detected devices, filter emulators, assign emu name to emulator variable and rest of output to device variable
  adb -s $emulator emu kill
done
```

### Emulator

#### start-up with a clean memory

emulator @Nexus_5X_API_28 -wipe-data 

#### to get a list of available images

>$~/Library/Android/sdk/tools/emulator -list-avds

#### this should take a string selected from the output of the command above

>$~/Library/Android/sdk/tools/emulator -avd <avdName>
