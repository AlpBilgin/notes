# xcrun

## Simctl

### get a list of all simulators. Actually running simulators will be marked as (booted in the list). Identify the UUID for the simulator that you want to use from the list

xcrun simctl list

#### filter names of booted simulators in seperated lines

```
#!/bin/bash

cd;
xcrun simctl list | grep Booted | while read i;
do echo "${i:0:$((${#i} - 48))}"; # remove last 48 chars from string and print
done;
```

### kill all devices

xcrun simctl shutdown all

### wipe all devices

xcrun simctl erase all

### Restart a specific device

xcrun simctl boot "iPhone 8 Plus"

### Force install an app to a simulator
>$xcrun simctl install <UUID> ~/Library/Developer/Xcode/DerivedData/<projectFolder>/Build/Products/Debug-iphonesimulator/<appName>.app

if you intentionally chaged the deriveddata path you should naturally use that path here.

(should look similar to this: xcrun simctl install F77DFD12-730C-48A8-B937-85D3AE1C1EA8 ~/Library/Developer/Xcode/DerivedData/Mein_o2-alhzujwksbfabyeyappxkmiqezvt/Build/Products/Debug-iphonesimulator/Mein o2 Beta.app)


# xcodebuild

You can't just dump a normal .ipa or .app file into the simultor. You need a specific .app file for a simulator. XCode actually creates this file automatically each the time you build an .ipa but will not hand it to you. You have to go into system folders and get it for yourself.

Because XCode makes the file for every .ipa build you can probably immediately find it in ~/Library/Developer/Xcode/DerivedData/<projectFolder>/Build/Products/Debug-iphonesimulator/<appName>.app

For Jenkins jobs you will want to build with CLI commands. Navigate to the project folder and do the following.
>$xcodebuild -workspace './Mein o2.xcworkspace/' -scheme '<schemeName>' -arch x86_64 -sdk <SDKName> (example: >$xcodebuild -workspace './Mein o2.xcworkspace/' -scheme 'Mein o2 Beta' -arch i386 -sdk iphonesimulator12.1)

Either way you can search for the the results using >$find ~/Library/Developer/Xcode/DerivedData -name "<schemeName>.app"

You can change the weird path with the compiler option -derivedDataPath /<some>/<path>

## example cli invocation to build for simulator
`xcodebuild -derivedDataPath <some path> -workspace '<filename>.xcworkspace' -scheme '<one single scheme>' \
-sdk iphonesimulator12.1 -arch x86_64 clean build;`

### list available sdks
`xcodebuild -showsdks`


# xcode-select

## For switching between XCODE root paths of multiple coexiting XCODE bundles
This will dump a list of discovered paths:
`xcode-select -p  
/Applications/Xcode-7.2.app/Contents/Developer
This will programmatically set the path to use:
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer`

# Instruments

### list available simulators
`instruments -s devices`

#Simulator

## To start the simulator form command line do:

open /Applications/Xcode.app/Contents/Developer/Applications/Simulator.app/



