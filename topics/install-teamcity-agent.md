[//]: # (title: Install TeamCity Agent)
[//]: # (auxiliary-id: Install TeamCity Agent)

Before installing a TeamCity build agent, make sure to read the [prerequisites](install-and-start-teamcity-agents.md#Prerequisites).

Choose a convenient installation option:
* using an [installer](#Install+from+Windows+Executable) on Windows
* [installing manually from a ZIP file](#Install+from+ZIP+File) on any platform
* preparing a Docker container based on the official TeamCity Agent image. See the details on [Docker Hub](https://hub.docker.com/r/jetbrains/teamcity-agent/).

### Install from Windows Executable

1. Open the __Agents__ page in TeamCity.
2. Click __Install Build Agents__ and select __Windows Installer__ to download the installer.
3. On the agent machine, run `agentInstaller.exe` and follow the installation instructions.

Ensure that the user account used to run the agent service has appropriate [permissions](install-and-start-teamcity-agents.md#Necessary+OS+and+environment+permissions)

### Install from ZIP File

1. On the agent computer, make sure the `JRE_HOME` or `JAVA_HOME` environment variables are set (pointing to the installed JRE or JDK directory respectively).
2. In the TeamCity web UI, navigate to the __Agents__ tab.
3. Click __Install Build Agents__ and select one of the two options to download the archive:  
    * __Minimal ZIP file distribution__: a regular build agent with no plugins
    * __Full ZIP file distribution*__: a full build agent prepacked with all plugins currently enabled on the server
4. Extract the downloaded file into the desired directory.
5. Navigate to the `<installation path>\conf` directory, locate the file called `buildAgent.dist.properties`, and rename it to `buildAgent.properties`.
6. Edit the `buildAgent.properties` file to specify the TeamCity server URL (HTTPS is recommended, see [these notes](install-and-start-teamcity-agents.md#Agent-Server+Data+Transfers)), the name of the agent, and the [authentication token](install-and-start-teamcity-agents.m#Generating+Authentication+Token). Refer to [this article](build-agent-configuration.md) for details on the agent configuration.
   {product="tcc"}
7. Edit the `buildAgent.properties` file to specify the TeamCity server URL and the name of the agent. Refer to [this article](build-agent-configuration.md) for details on the agent configuration.
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

### Install via Agent Push
{product="tc"}

TeamCity provides the Agent Push functionality that allows installing a build agent to a remote host. Currently, supported combinations of the server host platform and targets for build agents are:
* from the Unix-based TeamCity server, build agents can be installed to Unix hosts only (via SSH)
* from the Windows-based TeamCity server, build agents can be installed to Unix (via SSH) or Windows (via psexec) hosts

<note>

__SSH note__

Make sure the "Password" or "Public key" authentication is enabled on the target host according to a preferred authentication method.
</note>

### Install via Agent Push
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

>You can use Agent Push presets in [Agent Cloud profile](agent-cloud-profile.md) settings to automatically install a build agent to a started cloud instance.
