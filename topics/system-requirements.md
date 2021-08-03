[//]: # (title: System Requirements)
[//]: # (auxiliary-id: System Requirements)

This article contains general recommendations on choosing and configuring the environment for TeamCity Server and Agents. If you have specific questions that are not covered here, please contact our support via any convenient [feedback channel](feedback.md).

## TeamCity Server Requirements

### Choosing Server OS/Platform

Requirements to the server's operating system are listed [here](supported-platforms-and-environments.md#TeamCity+Server). We also recommend that you review the [requirements](supported-platforms-and-environments.md) to the integrations you plan to use. For example, the following functionalities may require or work better when TeamCity Server is installed under Windows:
* VCS integration with Azure DevOps Server (TFS)
* VCS integration with VSS
* Windows domain logins (can work under Linux, but may be less stable), especially NTLM HTTP authentication
* NuGet feed on the server (can work under Linux, but may be less stable)
* Agent push to Windows machines

If you have no specific preferences, Linux platforms may be more preferable in general. They are more effective in terms of file system operations and required maintenance. The final decision on OS may depend on your company's available resources and established practices.

### Estimating Hardware Requirements for Server

The server's hardware requirements depend on the server load, which in its turn significantly depends on the type of your builds and server usage. This section contains notes on various hardware-related aspects.

__CPU__

TeamCity can utilize multiple CPU cores, so increasing their number might make sense. For a production TeamCity usage we recommend using at least 4 CPU cores.

__Memory__

TeamCity utilizes memory resources for Maven integration, version control integration, Kotlin DSL execution, and so on. See the [notes](installing-and-configuring-the-teamcity-server.md#Setting+Up+Memory+settings+for+TeamCity+Server) on the main process memory usage. Most likely, 4 GB of memory will be enough to run up to 100 concurrent builds (agents), support up to 200 online users, and work with medium-sized repositories. If your server is considerably bigger, we suggest that you scale the memory amount respectively.

__Disk__

TeamCity loads the disk for maintaining temp directories (`<[TeamCity Home](teamcity-home-directory.md)>/temp` and OS's default `temp` directory) and `<[TeamCity Data Directory](teamcity-data-directory.md)>/system`.

Note that __performance of a TeamCity server highly depends on the disk system performance__. As TeamCity stores large amounts of data under `<[TeamCity Data Directory](teamcity-data-directory.md)>/system` (most notably, VCS caches and build results), it is important to ensure that the access to the disk is fast (in particular reading/writing files in multiple threads, listing files with attributes). Ensuring that the disk has good performance is especially important if you plan to store the Data Directory on a network drive. However, it is recommended using local storage for the `TeamCity Data Directory/system/caches` directory. See also [these notes](teamcity-data-directory.md#Recommendations+as+to+choosing+Data+Directory+Location).

The free disk space requirements are mainly determined by the number of builds stored on the server and the artifacts / build log size in each of them. The disk space is also used to store VCS-related caches: it can take about twice as much space as the checkout size of all the VCS roots configured on the server.

If builds generate large amount of data (artifacts/build logs/test data), we suggest that you use a fast disk for storing the `.BuildServer/system` directory and fast network between agents and server.

__Network__

Network resources are used for the traffic from VCS servers to clients (browsers, IDEs, and so on) and to/from build agents (to send sources or receive build results, logs, and artifacts).

#### Server Load Formula

The load on the server depends on:
* number of build configurations
* number of builds in the history
* number of the builds running daily
* amount of data consumed and produced by the builds (size of the used sources and artifacts, size of the build log, number and output size of unit tests, number of inspections and duplicates hits, size and number of produced artifacts, and so on)
* clean-up rules configured
* number of agents and their utilization percentage
* number of users having TeamCity web pages open
* number of users logged in from IDE plugin
* number and type of VCS roots as well as the configured checking for changes interval for the VCS roots. VCS checkout mode is relevant too: the server checkout mode generates greater server load. Specific types of VCS also affect server load, but they can be roughly estimated based on the native VCS client performance.
* number of changes detected by TeamCity per day in all the VCS roots
* the size of the repositories TeamCity works with
* total size of the sources checked out by TeamCity daily

#### Example Server Configuration

This section gives an example of a hardware configuration which is capable of handling up to 100 concurrently running builds. It is implied that this machine hosts only TeamCity server — not agents or database.

* Server-suitable modern multicore CPU (4 or more)
* 8 GB of memory
* Fast network connection
* Fast and reliable HDD
* Fast external database access

Based on our experience, a modest hardware like _Intel 3.2 GHz dual-core CPU, 3.2 GB memory under Windows, 1 GB network adapter, and single HDD_ can provide acceptable performance for the following setup:
* 60 projects and 300 build configurations (with one forth being active and running regularly)
* more than 300 builds a day
* about 2 MB log per build
* 50 build agents
* 50 web users and 30 IDE users
* 100 VCS roots (mainly Perforce and Subversion using server checkout), average checking for changes interval is 120 seconds
* more than 150 changes per day
* Kotlin DSL is not used
* the database (MySQL) is running on the same machine
* TeamCity server process has `-Xmx1100m` JVM setting

The following configuration can provide acceptable performance for a more loaded TeamCity server: _Intel Xeon E5520 2.2 GHz CPU (4 cores, 8 threads), 12 GB memory under Windows Server 2008 R2 x64, 1 GB network adapter, 3 HDD RAID1 disks (one general, one for artifacts, logs, and caches, and one for the database storage)_. Tested for the following setup:
* 150 projects and 1500 build configurations (with one third being active and running regularly)
* more than 1500 builds a day
* about 4 MB log per build
* 100 build agents
* 150 web users and 40 IDE users
* 250 VCS roots (mainly Git, Hg, Perforce and Subversion using agent-side checkout), average checking for changes interval is 180 seconds
* more than 1000 changes per day
* the database (MySQL) is running on the same machine
* TeamCity server process has `-Xmx3700m` x64 JVM setting

However, to ensure the peak load can be handled well, more powerful hardware is recommended.

>The general recommendation for deploying a large-scale TeamCity installation is to start with a reasonable hardware while considering hardware upgrade. You can increase the load on the server gradually (for example, add more projects), while monitoring the performance characteristics, and then decide on necessary hardware or software improvements. There is also a [benchmark plugin](https://plugins.jetbrains.com/plugin/9127-benchmark) which can be used to estimate the number of simultaneous builds the current server installation can handle.  
>We recommend that you stick to the best practices of server administration, like keeping disk defragmentation on a reasonable level.  
>If you need to increase the number of concurrently running builds (agents) by some factor, be prepared to increase the CPU, memory, database, and HDD access speeds by the same factor to achieve the same performance. If you increase the number of builds per day, be prepared to increase the disk size respectively.

#### Scaling Server Depending on Agents Number

TeamCity can stably work with 500\+ build agents (500 concurrently running builds actively logging build runtime data). In synthetic tests, the server was functioning fine with as many as 1000 concurrent builds (the server with 8 cores, 32 GB of total memory running under Linux, and MySQL server running on a separate comparable machine).

The server load produced by each build depends on the amount of this build's data (logs, tests and failure details, inspections/duplicates issues number, and so on). Keeping the amount of data reasonably constrained (publishing large outputs as build artifacts, not printing those into standard output, tweaking inspection profiles to report a limited set of the most important inspection hits, and so on) will help scale the server to handle more concurrent builds.

If you need much more agents/parallel builds, it is recommended to use a [multinode setup](multinode-setup.md). If a substantially bigger number of agents is required, consider using several separate TeamCity instances and distributing the projects between them.

>We are constantly improving the TeamCity performance and are willing to work closely with organizations running large TeamCity installations. This way, we can study performance issues and improve TeamCity to handle larger loads.
>
{type="note"}

See also related posts: [maximum number of agents which TeamCity can handle](http://blog.jetbrains.com/teamcity/2015/08/benchmarking-teamcity/) and [description of a substantial TeamCity setup](http://blogs.jetbrains.com/teamcity/2011/09/05/improving-performance-and-scalability-of-your-teamcity-server/).

### Configuring TeamCity Server for Performance

This section gives recommendations on tweaking the TeamCity server setup for better performance. It assumes that the server is already [configured for production use](installing-and-configuring-the-teamcity-server.md#Configuring+Server+for+Production+Use).

* Regularly review reported [Server Health](server-health.md) reports (including hidden ones).
* Use a separate [reverse proxy server](how-to.md#Set+Up+TeamCity+behind+a+Proxy+Server) (for example, NGINX) to handle HTTPS.
* Use a separate server for the external database. Monitor the database performance.
* Monitor the server's CPU and I/O performance. Increase hardware resources as necessary.
* Make sure [clean-up](clean-up.md) is configured for all the projects with a due retention policy. Check __Administration | Clean-Up__ to make sure that the clean-up if performed regularly.
* Consider ensuring good I/O performance for the `<[TeamCity Data Directory](teamcity-data-directory.md)>/system/caches` directory: for example, move it to a separate local drive.
* Regularly [archive](archiving-projects.md) obsolete projects.
* Regularly review the installed not bundled plugins and remove those not essential for the server operation.
* Consider using [agent-side checkout](vcs-checkout-mode.md) whenever possible.
* Make sure the build logs occupy a reasonable amount of space (tens of megabytes at most, but better less than 10 MB).
* If lots of VCS roots are configured on the server, consider configuring [repository commit hooks](configuring-vcs-post-commit-hooks-for-teamcity.md) instead of using polling for changes; or at least increase [VCS polling interval](configuring-vcs-roots.md#Common+VCS+Root+Properties) to 300 seconds or more.
* If the server is often used by a large number of users (more than 1000), consider reducing the frequency of background UI requests by increasing the [UI refresh intervals](teamcity-tweaks.md#Web+Page+Refresh+Interval).
* When regularly exceeding 500 concurrently running builds which log a lot of data, consider switching to a [multinode setup](multinode-setup.md).

### Estimating External Database Capacity

If a TeamCity server is extensively used, the database performance starts to play a greater role. To ensure production performance and reliability, an [external database](setting-up-external-database.md) must be used. And its size and performance are crucial aspects to consider.

If you decide to run an external database on the same machine as the server (not recommended), choose the server's hardware with the database engine requirements in mind.

It is quite hard to provide the exact numbers when setting up or migrating to an external database, as the required capacity varies greatly depending on how TeamCity is used. The database size requirements naturally vary based on the amount of stored data (number of builds, number of tests, and so on). The active server database usage can be estimated at several gigabytes of data per year.

__Database Size__

The size of the database depends on:
* how many builds are started daily
* how many tests are reported from builds
* [clean-up](clean-up.md) rules (retention policy)
* clean-up schedule

The recommended initial size is 4 GB. When migrating from the internal database, we suggest at least doubling the size of the current internal database. For example, the size of the external database (without the redo log files) of the internal TeamCity server in JetBrains is about 50 GB. Setting your database to grow automatically helps increase file sizes to a predetermined limit when necessary, which minimizes the effort to monitor the disk space.

Allocating 1 GB for the redo log (see the table below) and undo files is sufficient in most cases.

__Database Performance__

The following factors can affect the database performance:
* type of database (RDBMS)
* number of agents (which actually means the number of builds running in parallel)
* number of web pages opened by all users
* [clean-up](clean-up.md) rules (retention policy)

It is advised to place the [`TeamCity Data Directory`](teamcity-data-directory.md) and database data files on physically different hard disks (even when both the TeamCity server and RDBMS share the same host).

Placing redo logs on a separate physical disk is also recommended — especially if there are 50 or more agents.

#### Database-Specific Notes

The _redo_ log (or a similar entity) naming for different RDBMS:

<table><tr>

<td>

RDBMS

</td>

<td>

Log name

</td></tr><tr>

<td>

Oracle

</td>

<td>

Redo Log

</td></tr><tr>

<td>

MS SQL Server

</td>

<td>

Transaction Log

</td></tr><tr>

<td>

PostgreSQL

</td>

<td>

WAL (write ahead log)

</td></tr><tr>

<td>

MySQL \+ InnoDB and Percona

</td>

<td>

Redo Log

</td></tr></table>

__PostgreSQL__: it is recommended using version 9.2\+, which has a lot of query optimization features. Also see the information on the write-ahead-log (WAL) in the [PostgreSQL documentation](http://www.postgresql.org/docs/9.2/static/wal-internals.html).

__Oracle__: it is recommended keeping statistics on — all automatically gathered statistics should be enabled (since Oracle 10.0, this is the default setup). Also see the information on redo log files in the [Oracle documentation](https://docs.oracle.com/cd/B14117_01/server.101/b10752/iodesign.htm#26022).

__MS SQL Server__: it is NOT recommended using the jTDS driver — it does not work with `nchar/nvarchar`. To preserve unicode streams, it may cause queries to take a long time and consume many I/O operations. Also see the information on redo log in the [Microsoft Knowledge base](https://support.microsoft.com/kb/2033523). If you use jTDS, [consider migrating](setting-up-external-database.md#jTDS+driver).

__MySQL__: the query optimizer might be inefficient: some queries may get a wrong execution plan causing them to take a long time and consume many I/O operations.

## TeamCity Agent Prerequisites

This section lists requirements to the environment and OS user suitable for running a TeamCity [build agent](build-agent.md) process.

### Common Requirements

The agent Java process has to:
* be able to open outbound HTTP connections to the server URL configured via the `serverUrl` property in the [`buildAgent.properties`](build-agent-configuration.md) file (typically the same address you use in the browser to view the TeamCity UI).  
  Sending requests to the paths under the configured URL should not be limited. See also the recommended [reverse proxy settings](how-to.md#Set+Up+TeamCity+behind+a+Proxy+Server). Ensure that any firewalls installed on the agent or server machines, network configuration and proxies (if any) comply with these requirements.
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

#### Unidirectional Agent-to-Server Communication

Agents use unidirectional agent-to-server connection via the polling protocol: the agent establishes an HTTP(S) connection to the TeamCity Server, and polls the server periodically for server commands.

It is recommended to use __HTTPS__ for agent-to-server communications (check related [server configuration notes](how-to.md#Configure+HTTPS+for+TeamCity+Web+UI)). If the agents and the server are deployed into a secure environment, agents can be configured to use plain HTTP URL for connections to the server as this reduces transfer overhead. Note that the data travelling through the connection established from an agent to the server includes build settings, repository access credentials and keys, repository sources, build artifacts, build progress messages and build log. In case of using the HTTP protocol that data can be compromised via the "man in the middle" attack.

#### Bidirectional Communication

The bidirectional communication is a legacy connection between the agent and the server and it needs to be specifically enabled (see the example below). When enabled, it requires the agent to connect to the server via HTTP (or HTTPS) and the server to connect to the agent via HTTP.

The data that is transferred via the connections established by the server to agents is passed via an unsecured HTTP channel and thus is potentially exposed to any third party that may listen to the traffic between the server and the agents. Moreover, since the agent and server can send "commands" to each other, an attacker that can send HTTP requests and capture responses may in theory trick the agent into executing an arbitrary command and perform other actions with a security impact.

The communication protocol used by TeamCity agents is determined by the value of the `teamcity.agent.communicationProtocols` server [internal property](configuring-teamcity-server-startup-properties.md#TeamCity+internal+properties). The property accepts a comma-separated ordered list of protocols (`polling`  and `xml-rpc` are supported protocol names) and is set to `teamcity.agent.communicationProtocols=polling` by default. If several protocols are specified, the agent tries to connect using the first protocol from this list and if it fails, it will try to connect via the second protocol in the list. `polling` means unidirectional protocol, `xml-rpc` - older, bidirectional communication.

#### Changing Communication Protocol

* To change the communication protocol __for all agents__, set the TeamCity server `teamcity.agent.communicationProtocols` server [internal property](configuring-teamcity-server-startup-properties.md#TeamCity+internal+properties). The new setting will be used by all agents which will connect to the server after the change. To change the protocol for the existing connections, restart the TeamCity server.
* By default, the agent's property is not configured; when the agent first connects to the server, it receives it from the TeamCity server. To change the protocol __for an individual agent__ after the initial agent configuration, change the value of the `teamcity.agent.communicationProtocols` property in the [agent's properties](build-agent-configuration.md). The agent's property overrides the server property. After the change the agent will restart automatically upon finishing a running build, if any.

[//]: # (Internal note. Do not delete. "Setting up and Running Additional Build Agentsd283e376.txt")

## Estimating Hardware Requirements for Agent

The __agent__ hardware requirements are basically determined by the builds that are run. Running TeamCity agent software introduces a requirement for additional CPU time (but it can usually be neglected comparing to the build process CPU requirements) and additional memory: about 500 MB. The disk space required corresponds to the disk usage by the builds running on the agent (sources checkouts, downloaded artifacts, the disk space consumed during the build; all that combined for the regularly occurring builds). Although you can run a build agent on the same machine as the TeamCity server, the recommended approach is to use a separate machine (it may be virtual) for each build agent. If you chose to install several agents on the [same machine](setting-up-and-running-additional-build-agents.md#Installing+Several+Build+Agents+on+the+Same+Machine), consider the possible CPU, disk, memory or network bottlenecks that might occur. The [Performance Monitor](performance-monitor.md) build feature can help you in analyzing live data.

If you consider cloud deployment for TeamCity agents (for example, on Amazon EC2), also review [Setting Up TeamCity for Amazon EC2](setting-up-teamcity-for-amazon-ec2.md#Estimating+EC2+Costs)

A note on the agents' setup in JetBrains internal TeamCity installation:   
We use both separate machines each running a single agent and dedicated "servers" running several virtual machines each of them having a single agent installed. Experimenting with the hardware and software we settled on a configuration when each core7i physical machine runs 3 virtual agents, each using a separate hard disk. This stems form the fact that our (mostly Java) builds depend on HDD performance in the first place. But YMMV.

## Network Traffic Between Server and Agents

The traffic mostly depends on the settings as some of them include transferring binaries between the agent and the server.   
The most important flows of traffic between the agent and the server are:
* agent retrieves commands from the server: these are typically build start tasks which basically include a dump of the build configuration settings and the full set of build parameters. The latter can be large (e.g. megabytes) in case of a large build chain. The parameters can be reviewed on the build's [Parameters tab](working-with-build-results.md#Parameters);
* agent periodically sends current status data to the server (this includes all the agents parameters which can be reviewed on the agent's [Agent Parameters](viewing-build-agent-details.md#Agent+Parameters) tab);
* during the build, the agent sends  build log messages and parameters data back to the server. These can be reviewed on the [Build Log](working-with-build-results.md#Build+Log) and [Parameters](working-with-build-results.md#Parameters) tabs of the build;
* (when the server-side checkout mode is used) the agent downloads the sources before the build (as a full or incremental patch) from the server;
* (when an [artifact dependency](artifact-dependencies.md) is configured) the agent downloads build artifacts of other builds from the server before starting a build;
* (when artifacts are configured for a build) the agent uploads build artifacts to the server;
* some runners (like coverage or code analysis) include automatic uploading of their results' reports to the server.