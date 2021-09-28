[//]: # (title: Install and Start TeamCity Agents)
[//]: # (auxiliary-id: Install and Start TeamCity Agents;Setting up and Running Additional Build Agents)

>This section is about [self-hosted build agents](teamcity-cloud-subscription-and-licensing.md#cloud-self-hosted-agents). [JetBrains-hosted build agents](teamcity-cloud-subscription-and-licensing.md#cloud-jb-hosted-agents) are maintained by the TeamCity Cloud team and require no actions from users.
>
{type="note" product="tcc"}

Before you can start customizing projects and creating build configurations, you need to install and configure [build agents](build-agent.md). Review the [agent-server communication](#Agent-Server+Data+Transfers) and [Prerequisites](#Prerequisites) sections before proceeding with agent installation. Make sure to also read our [security notes](security-notes.md#Build+Agents) on maintaining build agents.
{product="tc"}

Before you can start customizing projects and creating build configurations, you need to add and configure [build agents](build-agent.md). Review the [agent-server communication](#Agent-Server+Data+Transfers) and [Prerequisites](#Prerequisites) sections before proceeding with agent installation. Make sure to also read our [security notes](security-notes.md#Build+Agents) on maintaining build agents and [licensing policy](teamcity-cloud-subscription-and-licensing.md) on adding new agents.
{product="tcc"}


If you install TeamCity bundled with a Tomcat servlet container, or use the TeamCity installer for Windows, both the server and one build agent are installed on the same machine. This is not a recommended setup for [production purposes](configure-server-installation.md#Configure+Server+for+Production+Use) because of [security concerns](security-notes.md). Moreover, the build procedure can slow down the responsiveness of the web UI and overall TeamCity server functioning.

## Prerequisites

### Necessary OS and environment permissions

Before the installation, review the [Conflicting Software](known-issues.md#Conflicting+Software) section. In case of any issues, make sure no conflicting software is used.

Note that to run a TeamCity build agent, the environment and user account used to run the Agent need to comply with the following requirements:

<anchor name="Network"/>

#### Common

The agent process (Java) must:
* be able to open outbound HTTP connections to the server URL configured via the `serverUrl` property in the [`buildAgent.properties`](build-agent-configuration.md) file (typically the same address you use in the browser to view the TeamCity UI). Sending requests to the paths under the configured URL should not be limited. See also the recommended [reverse proxy settings](how-to.md#Set+Up+TeamCity+behind+a+Proxy+Server). Ensure that any firewalls installed on the agent or server machines, network configuration and proxies (if any) comply with these requirements.
  {product="tc"}
* be able to open outbound HTTP connections to the server URL configured via the `serverUrl` property in the [`buildAgent.properties`](build-agent-configuration.md) file (typically the same address you use in the browser to view the TeamCity UI). Sending requests to the paths under the configured URL should not be limited. Ensure that any firewalls installed on the agent, network configuration, and proxies (if any) comply with these requirements.
  {product="tcc"}
* have full permissions (read/write/delete) to the following directories recursively: [`<agent home>`](agent-home-directory.md) (necessary for automatic agent upgrade and agent tools support), [`<agent work>`](agent-work-directory.md), [`<agent temp>`](agent-home-directory.md#Agent+Directories), and agent system directory (set by `workDir`, `tempDir`, and `systemDir` parameters in the `buildAgent.properties` file).
* be able to launch processes (to run builds).
* be able to launch nested processes with the following parent process exit (this is used during agent upgrade).

<anchor name="Windows"/>

#### Windows
* Log on as a service (to run as Windows service).
* Start/Stop service (to run as Windows service, necessary for the agent upgrade to work, see also [Microsoft KB article](https://support.microsoft.com/en-us/help/325349/how-to-grant-users-rights-to-manage-services-in-windows-server-2003)).
* Debug programs (required for take process dump functionality).
* Reboot the machine (required for agent reboot functionality) .
* To be able to [monitor performance](performance-monitor.md) of a build agent run as a Windows [service](#Build+Agent+as+a+Windows+Service), the user starting the agent must be a member of the Performance Monitor Users group.

<note>


For granting necessary permissions for unprivileged users, see [Microsoft documentation](https://support.microsoft.com/en-us/kb/325349).   
You can assign rights to manage services with Microsoft [SubInACL](http://www.microsoft.com/downloads/details.aspx?FamilyID=e8ba3e56-d8fe-4a91-93cf-ed6985e3927b&displaylang=en) utility, a command-line tool enabling administrators to directly edit security information. The tool uses the following syntax:

```Shell
SUBINACL /SERVICE \\MachineName\ServiceName /GRANT=[DomainName]UserName[=Access]

```

For example, to grant Start/Stop rights, you might need to execute the command:

```Shell
subinacl.exe /service TCBuildAgent /grant=<user login name>=PTO

```

</note>

#### Linux

* The user must be able to run the `shutdown` command (for the agent machine reboot functionality and the machine shutdown functionality when running in a cloud environment).
* If you are using `systemd`, it should not kill the processes on the main process exit (use [`RemainAfterExit=yes`](https://serverfault.com/questions/660063/teamcity-build-agent-gets-killed-by-systemd-when-upgrading)). See also [how to set up automatic agent start under Linux](#Automatic+Agent+Start+under+Linux).

#### Build-related Permissions

The build process is launched by a TeamCity agent and thus shares the environment and is executed under the OS user used by the TeamCity agent. Ensure that the TeamCity agent is configured accordingly. See [Known Issues](known-issues.md) for related Windows Service Limitations.

<anchor name="SettingupandRunningAdditionalBuildAgents-ServerDataTransfers"/>
<anchor name="SettingupandRunningAdditionalBuildAgents-Agent-ServerDataTransfers"/>

### Agent-Server Data Transfers

[//]: # (AltHead: Server-Agent Data Transfers)

A TeamCity agent connects to the TeamCity server via the URL configured as the `serverUrl` agent property. This is called [unidirectional](#Unidirectional+Agent-to-Server+Communication) agent-to-server connection.

If specifically configured, TeamCity agent can use legacy [bidirectional communication](#Bidirectional+Communication) which also requires establishing a connection from the server to the agents. To view whether the agent-server communication is unidirectional or bidirectional for a particular agent, navigate to __Agents | &lt;Agent Name&gt; | Agent Summary__ tab, the __Details__ section, __Communication Protocol__.
{product="tc"}

#### Unidirectional Agent-to-Server Communication

Agents use unidirectional agent-to-server connection via the polling protocol: the agent establishes an HTTP(S) connection to the TeamCity Server, and polls the server periodically for server commands.

It is recommended to use __HTTPS__ for agent-to-server communications (check related [server configuration notes](how-to.md#Configure+HTTPS+for+TeamCity+Web+UI)). If the agents and the server are deployed into a secure environment, agents can be configured to use plain HTTP URL for connections to the server as this reduces transfer overhead. Note that the data travelling through the connection established from an agent to the server includes build settings, repository access credentials and keys, repository sources, build artifacts, build progress messages and build log. In case of using the HTTP protocol that data can be compromised via the "man in the middle" attack.
{product="tc"}

#### Bidirectional Communication
{product="tc"}

The bidirectional communication is a legacy connection between the agent and the server and it needs to be specifically enabled (see the example below). When enabled, it requires the agent to connect to the server via HTTP (or HTTPS) and the server to connect to the agent via HTTP.

The data that is transferred via the connections established by the server to agents is passed via an unsecured HTTP channel and thus is potentially exposed to any third party that may listen to the traffic between the server and the agents. Moreover, since the agent and server can send "commands" to each other, an attacker that can send HTTP requests and capture responses may in theory trick the agent into executing an arbitrary command and perform other actions with a security impact.

The communication protocol used by TeamCity agents is determined by the value of the `teamcity.agent.communicationProtocols` server [internal property](server-startup-properties.md#TeamCity+internal+properties). The property accepts a comma-separated ordered list of protocols (`polling`  and `xml-rpc` are supported protocol names) and is set to `teamcity.agent.communicationProtocols=polling` by default. If several protocols are specified, the agent tries to connect using the first protocol from this list and if it fails, it will try to connect via the second protocol in the list. `polling` means unidirectional protocol, `xml-rpc` - older, bidirectional communication.

#### Changing Communication Protocol
{product="tc"}

* To change the communication protocol __for all agents__, set the TeamCity server `teamcity.agent.communicationProtocols` server [internal property](server-startup-properties.md#TeamCity+internal+properties). The new setting will be used by all agents which will connect to the server after the change. To change the protocol for the existing connections, restart the TeamCity server.
* By default, the agent's property is not configured; when the agent first connects to the server, it receives it from the TeamCity server. To change the protocol __for an individual agent__ after the initial agent configuration, change the value of the `teamcity.agent.communicationProtocols` property in the [agent's properties](build-agent-configuration.md). The agent's property overrides the server property. After the change the agent will restart automatically upon finishing a running build, if any.

[//]: # (Internal note. Do not delete. "Setting up and Running Additional Build Agentsd283e376.txt")

## Generating Authentication Token
{product="tcc"}

The recommended approach to connecting a self-hosted agent to a TeamCity Cloud instance is to generate a unique authentication token for this agent. To do this, go to __Agents__, open the __Install Build Agents__ menu in the upper right corner of the screen, and click _Use authentication token_. There are two options:

* _Generate plain-text token_: you need to copy the generated token and enter it in the [build agent configuration](build-agent-configuration.md) file. On Windows, you will be prompted to enter it right in the _Configure Build Agent Properties_ installation dialog.
* _Download config_: enter an agent name (`name` attribute in the [build agent config](build-agent-configuration.md)) and download the entire config file. Place it as the `buildAgent.properties` file in the build agent directory.

Please generate own token or configuration file per each self-hosted agent.

<anchor name="SettingupandRunningAdditionalBuildAgents-InstallingAdditionalBuildAgents"/>