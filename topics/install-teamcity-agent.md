[//]: # (title: Install TeamCity Agent)
[//]: # (auxiliary-id: Install TeamCity Agent)

## Installing Additional Build Agents

1. Install a build agent using any of the following options:
* [using Windows installer](#Installing+via+Windows+installer) (Windows only)
* [by downloading a ZIP file and installing manually](#Installing+via+ZIP+File) (any platform): minimal and full-packed ZIP archives ara available
* via the [Docker Agent Image](#Installing+via+Docker+Agent+Image) option to prepare a Docker container based on the official [TeamCity Agent image](https://hub.docker.com/r/jetbrains/teamcity-agent/)
2. After installation, configure the agent specifying its name and the address of the TeamCity server in the [`conf/buildAgent.properties`](build-agent-configuration.md) file.
   {product="tc"}
2. After installation, configure the agent specifying its name, the address of the TeamCity server, and the [authentication token](#Generating+Authentication+Token) in the [`conf/buildAgent.properties`](build-agent-configuration.md) file.
   {product="tcc"}
3. [Start](#Starting+the+Build+Agent) the agent. If the agent does not seem to run correctly, check the [agent logs](viewing-build-agent-logs.md).

When the newly installed agent connects to the server for the first time, it appears on the `Agents` page, `Unauthorized agents` tab visible to administrators/users with the permissions to authorize it. Agents will not run builds until they are authorized in the TeamCity web UI. The agent running on the same computer as the server is authorized by default.
{product="tc"}

The number of authorized agents is limited by the number of agents licenses on the server. See more under [Licensing Policy](licensing-policy.md).
{product="tc"}

### Installing via Windows installer
1. In the TeamCity web UI, navigate to the __Agents__ tab.
2. Click the __Install Build Agents__ link and select __Windows Installer__ to download the installer.
3. On the agent, run the `agentInstaller.exe` Windows Installer and follow the installation instructions.

<note>

Ensure that the user account used to run the agent service has appropriate [permissions](#Necessary+OS+and+environment+permissions)
</note>

### Installing via Docker Agent Image
1. In the TeamCity UI, navigate to the __Agents__ tab.
2. Click the __Install Build Agents__ link and select _Docker Agent Image_.
3. Follow the instructions on the opened [page](https://hub.docker.com/r/jetbrains/teamcity-agent/).

### Installing via ZIP File
1. Make sure a JDK (JRE) 1.8.0_161 or later (Java 8-11 are supported, but 1.8.0_161+ is recommended) is properly installed on the agent computer.
2. On the agent computer, make sure the `JRE_HOME` or `JAVA_HOME` environment variables are set (pointing to the installed JRE or JDK directory respectively).
3. In the TeamCity web UI, navigate to the __Agents__ tab.
4. Click the __Install Build Agents__ link and select one of the two options to download the archive:
    * __Minimal ZIP file distribution__: a regular build agent with no plugins
    * (since TeamCity 2020.1) __Full ZIP file distribution*__: a full build agent prepacked with all plugins currently enabled on the server
5. Extract the downloaded file into the desired directory.
6. Navigate to the `<installation path>\conf` directory, locate the file called `buildAgent.dist.properties` and rename it to `buildAgent.properties`.
7. Edit the `buildAgent.properties` file to specify the TeamCity server URL (HTTPS is recommended, see the [notes](#Agent-Server+Data+Transfers)), the name of the agent, and the [authentication token](#Generating+Authentication+Token). Refer to the [Build Agent Configuration](build-agent-configuration.md) page for details on agent configuration.
   {product="tcc"}
7. Edit the `buildAgent.properties` file to specify the TeamCity server URL and the name of the agent. Refer to the [Build Agent Configuration](build-agent-configuration.md) page for details on agent configuration.
   {product="tc"}
8. Under Linux, you may need to give execution permissions to the `bin/agent.sh` shell script.

On Windows, you may also want to install the [build agent Windows service](#Build+Agent+as+a+Windows+Service) instead of using the manual agent startup.

<tip>

__\*__ A __minimal TeamCity agent__ distribution does not contain plugins: the agent downloads them on the first start. The __full agent__ contains all enabled plugins and automatically stays relevant with the current TeamCity server state. This makes its distribution archive larger but significantly reduces the time spent on the first agent run.

The full agent is the most convenient if you use scripts for creating agent images (for example, [in cloud](agent-cloud-profile.md)). All instances will be synchronized with the server from the start and can instantly run a build.
{product="tc"}

The full agent is the most convenient if you use scripts for creating agent images. All instances will be synchronized with the server from the start and can instantly run a build.
{product="tcc"}

Note that after starting, the full agent behaves like a regular agent. If you modify the state of plugins on the TeamCity server, all active agents will need to restart to sync with the server.

</tip>

### Installing via Agent Push
{product="tc"}

TeamCity provides the Agent Push functionality that allows installing a build agent to a remote host. Currently, supported combinations of the server host platform and targets for build agents are:
* from the Unix-based TeamCity server, build agents can be installed to Unix hosts only (via SSH)
* from the Windows-based TeamCity server, build agents can be installed to Unix (via SSH) or Windows (via psexec) hosts

<note>

__SSH note__

Make sure the "Password" or "Public key" authentication is enabled on the target host according to a preferred authentication method.
</note>

### Installing via Agent Push
{product="tcc"}

TeamCity provides the Agent Push functionality that allows installing a build agent to a remote Unix host.

<note>

__SSH note__

Make sure the "Password" or "Public key" authentication is enabled on the target host according to a preferred authentication method.
</note>

#### Remote Host Requirements

There are several requirements for the remote host:

<table><tr>

<td width="150">

Platform


</td>

<td>

Prerequisites


</td></tr><tr>

<td>

Linux


</td>

<td>

* Installed JDK (JRE) 8-11 (__1.8.0_161 or later is recommended__). The JVM should be reachable via the `JAVA_HOME` (`JRE_HOME`) global environment variable or be in the global path (instead of being specified in, for example, user's `.bashrc` file)

* The `unzip` utility.

* Either `wget` or `curl`.


</td></tr><tr>

<td>

Windows


</td>

<td>

* Installed JDK/JRE 8-11 (__1.8.0_161 or later is recommended__).

[//]: # (Internal note. Do not delete. "Setting up and Running Additional Build Agentsd283e644.txt")

* `Sysinternals psexec.exe` has to be installed on the TeamCity server and accessible in paths. You can install it on the __Administration__ | __Tools__ page.   
  Note that [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) applies additional requirements to a remote Windows host. Make sure the following preconditions are satisfied:
    * [Administrative share](https://en.wikipedia.org/wiki/Administrative_share) on a remote host is enabled and accessible.
    * Remote services work (MMC snap-in can connect to the machine).
    * Remote registry works (`regedit` can connect to the machine via `services.msc`).
    * Server and workstation services are running (check via `services.msc`).
    * [Classic network authentication](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-access-sharing-and-security-model-for-local-accounts) is enabled.

You can test the connection with the following commands:
  ```Console
    net use \\target\Admin$ /user:Administrator 
    dir \\target\Admin$ 
   ```


</td></tr></table>

#### Installation
1. In the TeamCity Server web UI navigate to the __Agents__ | __Agent Push__ tab, and click __Install Agent__.   
   If you are going to use same settings for several target hosts, you can __create a preset__ with these settings and use it each time when installing an agent to another remote host.
2. In the _Install agent_ dialog, either select a saved preset or choose "_Use custom settings_", specify the target host platform, and configure corresponding settings. Agent Push to a Linux system via SSH supports custom ports (the default is 22) specified as the __SSH port__ parameter. The port specified in a preset can be overridden in the host name (for example, `hostname.domain:2222`), during the actual agent installation.
3. You may need to download `Sysinternals psexec.exe`, in which case you will see the corresponding warning and a link to the __Administration__ | __Tools__ page where you can download it.

<tip product="tc">

You can use Agent Push presets in [Agent Cloud profile](agent-cloud-profile.md) settings to automatically install a build agent to a started cloud instance.

</tip>