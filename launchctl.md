https://www.launchd.info/

example plist file:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>StandardOutPath</key>
	<string>/Users/jenkins/automation/appium/appium.log</string>
	<key>EnvironmentVariables</key>
	<dict>
		<key>JAVA_HOME</key>
		<string>/Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/home</string>
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
	<string>com.jenkins.appium</string>
</dict>
</plist>
```
