[//]: # (title: Start TeamCity Agent)
[//]: # (auxiliary-id: Start TeamCity Agent)

## Starting the Build Agent

TeamCity build agents can be started manually or configured to start automatically.

### Manual Start

__To start the agent manually__, run the following script:
* for Windows: `<installation path>\bin\agent.bat start`
* for Linux and macOS: `<installation path>\bin\agent.sh start`

### Automatic Start

To configure the agent to be __started automatically__, see the corresponding sections:
* [Windows](#Automatic+Agent+Start+under+Windows)
* [Linux](#Automatic+Agent+Start+under+Linux): configure a daemon process with the `agent.sh start` command to start it and the `agent.sh stop` command to stop it.
* [macOS](#Automatic+Agent+Start+under+macOS)

#### Automatic Agent Start under Windows

To run an agent automatically on the machine boot under Windows, you can either set up the agent to be run as a Windows service or use another way of the automatic process start.   
Using the Windows service approach is the easiest way, but Windows applies [some constraints](known-issues.md#Agent+running+as+Windows+Service+Limitations) to the processes run this way.   
A TeamCity agent works reliably under Windows service provided all the [requirements](#Necessary+OS+and+environment+permissions) are met, but is often not the case for the build processes configured to be run on the agent.

That is why it is recommended to run a TeamCity agent as a Windows service only if all the build scripts support this. Otherwise, it is advised to use alternative OS-specific ways to start a TeamCity agent automatically.   
One of the ways is to configure an [automatic user logon](https://support.microsoft.com/en-us/help/324737/how-to-turn-on-automatic-logon-in-windows) on Windows start and then configure the TeamCity agent start (via `agent.bat start`) on the user logon (for example, via Windows [Task Scheduler](https://msdn.microsoft.com/en-us/library/windows/desktop/aa383614(v=vs.85).aspx)).

#### Build Agent as a Windows Service

In Windows, you may want to run TeamCity agent as a Windows service to allow it running without any user logged on. If you use the Windows agent installer, you have an option to install the service in the installation wizard.

<warning>

To run builds, the build agent must be started under a user with sufficient permissions for performing a build and [managing](#Windows) the service. By default, a Windows service is started under the SYSTEM account which is not recommended for production use due to extended permissions the account uses. To change it, use the standard Windows Services applet (__Control Panel | Administrative Tools | Services__) and change the user for the _TeamCity Build Agent_ service.
</warning>

>If you start an Amazon EC2 cloud agent as a Windows service, check the [related notes](setting-up-teamcity-for-amazon-ec2.md#Preparing+Image+with+Installed+TeamCity+Agent).
>
{type="tip" product="tc"}

The following instructions can be used to install the Windows service manually (for example, after `.zip` agent installation). This procedure should also be performed to create Windows services for the [second and following agents](#Installing+Several+Build+Agents+on+the+Same+Machine) on the same machine.

__To install the service:__
1. Check if the service with the required name and id (see #4 below, service name is `TeamCity Build Agent` by default) is not present; if installed, remove it.
2. Check that the `wrapper.java.command` property in the `<agent home>\launcher\conf\wrapper.conf` file contains a valid path to the Java executable in the JDK installation directory. You can use `wrapper.java.command=../jre/bin/java` for the agent installed with the Windows distribution. Make sure to specify the path of the java.exe file without any quotes.
3. If you want to run the agent under user account (recommended) and not "System", add the `wrapper.ntservice.account` and `wrapper.ntservice.password` properties to the `<agent home>\launcher\conf\wrapper.conf` file with appropriate credentials
4. (for second and following installations) Modify the `<agent>\launcher\conf\wrapper.conf` file so that the `wrapper.console.title`, `wrapper.ntservice.name`, `wrapper.ntservice.displayname`, and `wrapper.ntservice.description` properties have unique values within the computer.
5. Run the `<agent home>\bin\service.install.bat` script under a user with sufficient privileges to register the new agent service. Make sure to start the agent for the first time only after it is configured as described.

__To start the service:__
* Run `<agent home>/bin/service.start.bat` (or use standard Windows Services applet)

__To stop the service:__
* Run `<agent home>/bin/service.stop.bat` (or use standard Windows Services applet)

You can also use the standard Windows `net.exe` utility to manage the service once it is installed.   
For example (assuming the default service name):


```Shell
net start TCBuildAgent

```


The `<agent home>\launcher\conf\wrapper.conf` file can also be used to alter the agent JVM parameters.

The user account used to run the build agent service must have enough rights to start/stop the agent service, as described [above](#Windows).

#### Automatic Agent Start under Linux

To run an agent automatically on the machine boot under Linux, configure a daemon process with the `agent.sh start` command to start it and the `agent.sh stop` command to stop it.

For systemd, see the example `teamcityagent.service` configuration file:

```Shell
[Unit]
Description=TeamCity Build Agent
After=network.target

[Service]
Type=oneshot

User=teamcityagent
Group=teamcityagent
ExecStart=/home/teamcityagent/agent/bin/agent.sh start
ExecStop=-/home/teamcityagent/agent/bin/agent.sh stop

# Support agent upgrade as the main process starts a child and exits then
RemainAfterExit=yes
# Support agent upgrade as the main process gets SIGTERM during upgrade and that maps to exit code 143
SuccessExitStatus=0 143

[Install]
WantedBy=default.target

```


For `init.d`, refer to an example procedure:

1\. Navigate to the services start/stop services scripts directory:

```Shell
cd /etc/init.d/

```

2\. Open the build agent service script:


```Shell
sudo vim buildAgent

```

3\. Paste the following into the file :


```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          TeamCity Build Agent
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start build agent daemon at boot time
# Description:       Enable service provided by daemon.
### END INIT INFO
#Provide the correct user name:
USER="agentuser"
 
case "$1" in
start)
 su - $USER -c "cd BuildAgent/bin ; ./agent.sh start"
;;
stop)
 su - $USER -c "cd BuildAgent/bin ; ./agent.sh stop"
;;
*)
  echo "usage start/stop"
  exit 1
 ;;
 
esac
 
exit 0

```


4\. Set the permissions to execute the file:


```Shell
sudo chmod 755 buildAgent

```


5\. Make links to start the agent service on the machine boot and on restarts using the appropriate tool:

* For Debian/Ubuntu:

```Shell
sudo update-rc.d buildAgent defaults

```


* For Red Hat/CentOS:

```Shell
sudo chkconfig buildAgent on

```

#### Automatic Agent Start under macOS

For macOS, TeamCity provides the ability to load a build agent automatically when a build user logs in.

The recommended approach is to use [`launchd`](https://support.apple.com/guide/terminal/script-management-with-launchd-apdc6c1077b-5d5d-4d35-9c19-60f2397b2369/mac) (LaunchAgent):

To configure an automatic build agent startup via `launchd`, follow these steps:

1\. Install a build agent on a Mac via `buildAgent.zip`.

2\. Prepare the `conf/buildAgent.properties` file (set agent name there, at least).

3\. Make sure that all files under the `buildAgent` directory are owned by `your_build_user` to ensure a proper agent upgrade process.

4\. Load the build agent via the command:


```Shell
mkdir buildAgent/logs  # Directory should be created under your_build_user user
sh buildAgent/bin/mac.launchd.sh load

```

Run these commands under the `your_build_user` account.

__Wait for some time__ for the build agent to auto-upgrade from the TeamCity server. This can take up to several minutes. You can watch the process in the logs:


```Shell
tail -f buildAgent/logs/teamcity-agent.log

```

5\. When the build agent upgrades and successfully connects to TeamCity server, stop the agent:


```Shell
sh buildAgent/bin/mac.launchd.sh unload

```

6\. After the build agent upgrades from the TeamCity server, copy the `buildAgent/bin/jetbrains.teamcity.BuildAgent.plist` file to the `$HOME/Library/LaunchAgents/` directory (you might have to create it). If you don't want TeamCity to start under the root permissions, specify the __UserName__ key in the `.plist` file, for example:
```XML
<key>UserName</key>
<string>your_build_user</string>
```

7\. Configure your Mac system to __automatically login__ as `your_build_user`, as described [here](https://support.apple.com/en-us/HT201476).

8\. Reboot.

On the system startup, the build user should automatically log in, and the build agent should start.

To quickly check that the build agent is running, use the following command:

```Shell
launchctl list | grep BuildAgent 
69722	0	jetbrains.teamcity.BuildAgent

```

## Stopping the Build Agent

__To stop the agent manually__, run the `<Agent home>\agent` script with a `stop` parameter.

Use `stop` to request stopping after the current build finished.   
Use `stop force` to request an immediate stop (if a build is running on the agent, it will be stopped abruptly (canceled)).   
Under Linux, you have one more option top use `stop kill` to kill the agent process.

If the agent runs with a console attached, you may also press __Ctrl\+C__ in the console to stop the agent (if a build is running, it will be canceled).

### Stopping the Agent Service on macOS

If a build agent has been started as a `LaunchAgent` service, it can be stopped using the `launchctl` utility:

```Shell
launchctl unload $HOME/Library/LaunchAgents/jetbrains.teamcity.BuildAgent.plist
# or  
launchctl remove jetbrains.teamcity.BuildAgent

```
