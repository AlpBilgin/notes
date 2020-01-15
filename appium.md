# Selector Strategies

Strategy |	Description
------------ | -------------
Accessibility ID |	Read a unique identifier for a UI element. For XCUITest it is the element's accessibility-id attribute. For Android it is the element's content-desc attribute.
Class name |	For IOS it is the full name of the XCUI element and begins with XCUIElementType. For Android it is the full name of the UIAutomator2 class (e.g.: android.widget.TextView)
ID |	Native element identifier. **resource-id** for android; **name** for iOS.
Name |	Name of element
XPath |	Search the app XML source using xpath (not recommended, has performance issues)
Image |	Locate an element by matching it with a base 64 encoded image file

# Example MacOS agent to keep appium server alive
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>StandardOutPath</key>
        <string>/Users/jenkins/automation/appium/appium.log</string>
        <key>EnvironmentVariables</key>
        <dict>
                <key>ANDROID_HOME</key>
                <string>/Users/jenkins/Library/Android/sdk</string>
                <key>PATH</key>
                <string>/Users/jenkins/.npm-global/bin:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/bin:/Users/jenkins/Library/Android/sdk/platform-tools:/Users/jenkins/Library/Android/sdk/tools:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Applications/jenkins_slave</string>
        </dict>
        <key>ProgramArguments</key>
        <array>
                <string>/usr/local/bin/node</string>
                <string>/Users/jenkins/.npm-global/lib/node_modules/appium/build/lib/main.js</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
        <key>KeepAlive</key>
        <dict>
                <key>NetworkState</key>
                <true/>
        </dict>
        <key>Label</key>
        <string>com.o2.appium</string>
</dict>
</plist>
```
