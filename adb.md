# ADB

## Access cellphone over network

Debug considerations
When debugging applications that use USB accessory or host features, you most likely will have USB hardware connected to your Android-powered device. This will prevent you from having an adb connection to the Android-powered device via USB. You can still access adb over a network connection. To enable adb over a network connection:

Connect the Android-powered device via USB to your computer.
From your SDK platform-tools/ directory, enter adb tcpip 5555 at the command prompt.
Enter adb connect <device-ip-address>:5555 You should now be connected to the Android-powered device and can issue the usual adb commands like adb logcat.
To set your device to listen on USB, enter adb usb.
