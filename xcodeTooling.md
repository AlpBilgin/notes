# xcrun

## Simctl

### kill all devices

xcrun simctl shutdown all

### wipe all devices

xcrun simctl erase all

### Restart a specific device

xcrun simctl boot "iPhone 8 Plus"

# xcodebuild

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

