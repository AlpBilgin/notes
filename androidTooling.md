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

loop this command like the following for multiple devices

```sh
adb devices | grep "emulator-" | while read -r emulator device; do # get list of detected devices, filter emulators, assign emu name to emulator variable and rest of output to device variable
  adb -s $emulator emu kill
done
```

#### query boot status of an emulator


`adb shell getprop init.svc.bootanim` OR `adb shell getprop sys.boot_completed` gets you the state
use -s DEVICENAME in case of multi emulators
loop like the following to wait on boot
```sh
while [ "`adb shell getprop sys.boot_completed | tr -d '\r' `" != "1" ] ; do sleep 1; done
```

### Emulator

#### start-up with a clean memory

emulator @Nexus_5X_API_28 -wipe-data (-avd Nexus_5X_API_28 also works)

#### to get a list of available images

>$~/Library/Android/sdk/tools/emulator -list-avds

#### this should take a string selected from the output of the command above

>$~/Library/Android/sdk/tools/emulator -avd <avdName>
  
  
 ### Incantations
 
 ```sh
 #!/bin/bash

#Start the emulator
$ANDROID_SDK/tools/emulator -avd Nexus_5X_API_23 -wipe-data &
EMULATOR_PID=$!

# Wait for Android to finish booting
WAIT_CMD="$ANDROID_SDK/platform-tools/adb wait-for-device shell getprop init.svc.bootanim"
until $WAIT_CMD | grep -m 1 stopped; do
  echo "Waiting..."
  sleep 1
done

# Unlock the Lock Screen
$ANDROID_SDK/platform-tools/adb shell input keyevent 82

# Clear and capture logcat
$ANDROID_SDK/platform-tools/adb logcat -c
$ANDROID_SDK/platform-tools/adb logcat > build/logcat.log &
LOGCAT_PID=$!

# Run the tests
./gradlew connectedAndroidTest -i

# Stop the background processes
kill $LOGCAT_PID
kill $EMULATOR_PID
 ```
