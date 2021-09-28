[//]: # (title: Start TeamCity Server)
[//]: # (auxiliary-id: Start TeamCity Server)

## Start TeamCity

Under Windows, if a TeamCity server is installed as a service, follow the usual procedure of starting and stopping services.

If TeamCity is installed using the `.exe` or `.tar.gz` distributions, it can be started and stopped by the `teamcity-server` scripts provided in the `<[TeamCity Home](teamcity-home-directory.md)>/bin` directory. The script accepts `run` (run in the same console), `start` (start new detached process and exit from the script), and `stop` commands.

* __(evaluation only) To start/stop the TeamCity server and one default agent at the same time__, use the `runAll` script, for example:
  * Use `runAll.bat start` to start the server and the default agent.
  * Use `runAll.bat stop` to stop the server and the default agent.
* __To start/stop the TeamCity server only__, use the `teamcity-server` scripts and pass the required parameters. Start the script without parameters to see the usage instructions. The `teamcity-server` scripts support the following options for the `stop` command:
  * `stop n` — sends the stop command to the TeamCity server and waits up to `n` seconds for the process to end.
  * `stop n -force` — sends the stop command to the TeamCity server, waits up to `n` seconds for the process to end, and terminates the server process if it did not stop.

>The TeamCity server will restart automatically if the server process exits (crashes or is killed) without invoking the `teamcity-server stop` script.

If you need to pass special properties to the server, refer to [this article](server-startup-properties.md).

## Open TeamCity Web UI

The TeamCity UI can be accessed via a web browser. The default addresses are [`http://localhost/`](http://localhost/) for the Windows distribution and [`http://localhost:8111/`](http://localhost:8111/) for the `tar.gz` distribution. See how to [change the server port](configure-server-installation.md#Changing+Server+Port), if necessary.

If you cannot access the TeamCity web UI after a successful installation, please refer to the [troubleshooting section](configure-server-installation.md#Troubleshoot+TeamCity+Installation).

## Autostart TeamCity Server on macOS

Starting up TeamCity server on macOS is quite similar to starting Tomcat on macOS.
1. Install TeamCity and make sure it works if started from the command line with `bin/teamcity-server.sh start`. This instruction assumes that TeamCity is installed to `/Library/TeamCity`.
2. Create the `/Library/LaunchDaemons/jetbrains.teamcity.server.plist` file with the following content:
    ```XML
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>WorkingDirectory</key>
        <string>/Library/TeamCity</string>
        <key>Debug</key>
        <false/>
        <key>Label</key>
        <string>jetbrains.teamcity.server</string>
        <key>OnDemand</key>
        <false/>
        <key>KeepAlive</key>
        <true/>
        <key>ProgramArguments</key>
        <array>
            <string>/bin/bash</string>
            <string>--login</string>
            <string>-c</string>
            <string>bin/teamcity-server.sh run</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
        <key>StandardErrorPath</key>
        <string>logs/launchd.err.log</string>
        <key>StandardOutPath</key>
        <string>logs/launchd.out.log</string>
    </dict>
    </plist>
    
    ```
3. Test your file by running this command:
    ```Shell
    launchctl load /Library/LaunchDaemons/jetbrains.teamcity.server.plist
    
    ```
   This command should start the TeamCity server (you can see this from `logs/teamcity-server.log` and in your browser).
4. If you don't want TeamCity to start under the root permissions, specify the `UserName` key in the `.plist` file, for example:
    ```XML
    <key>UserName</key>
    <string>teamcity_user</string>
    
    ```
The TeamCity server will now start automatically when the machine starts. To configure automatic start of a TeamCity build agent, see the [dedicated section](start-teamcity-agent.md#Automatic+Start).