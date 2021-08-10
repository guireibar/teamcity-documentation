[//]: # (title: System Requirements)
[//]: # (auxiliary-id: System Requirements)

This article contains general recommendations on choosing and configuring the environment for TeamCity Server and Agents, as well as the network connection between them and dedicated external database. If you have specific questions that are not covered here, please contact our support via any convenient [feedback channel](feedback.md).

## TeamCity Server Requirements

### Choosing Server OS/Platform

TeamCity Server can run on any recent version of Windows, Linux, or macOS. Requirements to the server's operating system are listed [here](supported-platforms-and-environments.md#TeamCity+Server). We also recommend that you review the [requirements](supported-platforms-and-environments.md) to the integrations you plan to use. For example, the following functionalities may require or work better when TeamCity Server is installed under Windows:
* VCS integration with Azure DevOps Server (TFS)
* VCS integration with VSS
* Windows domain logins (can work under Linux, but may be less stable), especially NTLM HTTP authentication
* NuGet feed on the server (can work under Linux, but may be less stable)
* Agent push to Windows machines

If you have no specific preferences, Linux platforms may be more preferable in general. They are more effective in terms of file system operations and maintenance. The final decision on OS may depend on your company's available resources and established practices.

### Estimating Hardware Requirements for Server

The server's hardware requirements depend on the server load, which in its turn significantly depends on the type of your builds and server usage patterns. This section contains notes on various hardware-related aspects.

#### CPU

TeamCity can utilize multiple CPU cores, so increasing their number might make sense. For a production TeamCity usage we recommend using at least 4 CPU cores.

#### Memory

TeamCity utilizes memory resources for Maven integration, version control integration, Kotlin DSL execution, and so on. See the [notes](installing-and-configuring-the-teamcity-server.md#Setting+Up+Memory+settings+for+TeamCity+Server) on using the main process memory. Most likely, 4 GB of memory will be enough to run up to 100 concurrent builds (agents), support up to 200 online users, and work with medium-sized repositories. If your server is considerably bigger, we suggest that you scale the memory amount respectively.

#### Disk

TeamCity loads the disk to work with temp directories (`<[TeamCity Home](teamcity-home-directory.md)>/temp` and OS's default `temp` directory) and `<[TeamCity Data Directory](teamcity-data-directory.md)>/system`.

Note that __performance of a TeamCity server highly depends on the disk performance__. As TeamCity stores large amounts of data under `<[TeamCity Data Directory](teamcity-data-directory.md)>/system` (most notably, VCS caches and build results), it is important to ensure that the access to the disk is fast (in particular, reading/writing files in multiple threads and listing files with attributes). Ensuring that the disk has good performance is especially important if you plan to store the Data Directory on a network drive. However, it is recommended using local storage for the `TeamCity Data Directory/system/caches` directory. See also [notes on choosing the Data Directory location](teamcity-data-directory.md#Recommendations+as+to+choosing+Data+Directory+Location).

The free disk space requirements are mainly determined by the number of builds stored on the server and the artifacts / build log size in each of them. The disk space is also used to store VCS-related caches: it can take about twice as much space as the checkout size of all the VCS roots configured on the server.

If builds generate large amount of data (artifacts/build logs/test data), we suggest that you use a fast disk for storing the `.BuildServer/system` directory and fast network between agents and the server.

#### Network

Network resources are used for the traffic from VCS servers to clients (browsers, IDEs, and so on) and to/from build agents (to send sources or receive build results, logs, and artifacts).

#### Server Load Formula

The load on the server depends on:
* number of build configurations
* number of builds in the history
* number of builds running daily
* amount of data consumed and produced by the builds (size of the used sources and artifacts, size of the build log, number and output size of unit tests, number of inspections and duplicates' hits, size and number of produced artifacts, and so on)
* clean-up rules configured
* number of agents and their utilization
* number of users having TeamCity web pages open
* number of users logged in from IDE plugin
* number and type of VCS roots as well as the configured interval of checking for changes. VCS checkout mode is relevant too: the server checkout mode generates greater server load. Specific types of VCS also affect server load, but they can be roughly estimated based on the native VCS client performance.
* number of changes detected by TeamCity per day in all the VCS roots
* the size of the repositories TeamCity works with
* total size of the sources checked out by TeamCity daily

#### Example Server Configurations

The following hardware configuration is capable of handling up to 100 concurrently running builds. It is implied that this machine hosts only TeamCity server — not agents or database.
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

The general recommendation for deploying a large-scale TeamCity installation is to start with a reasonable hardware while considering hardware upgrade. You can increase the load on the server gradually (for example, add more projects), while monitoring the performance stats, and then decide on necessary hardware or software improvements. There is also a [benchmark plugin](https://plugins.jetbrains.com/plugin/9127-benchmark) which can be used to estimate the number of simultaneous builds the current server installation can handle.  

We also recommend that you stick to the best practices of server administration, like keeping disk defragmentation on a reasonable level.  

If you need to increase the number of concurrently running builds (agents) by some factor, be prepared to increase the CPU, memory, database, and HDD access speeds by the same factor to achieve the same performance. If you increase the number of builds per day, be prepared to increase the disk size respectively.

### Scaling Server Depending on Agents Number

TeamCity can stably work with 500+ build agents (500 concurrently running builds which are actively logging build runtime data). In synthetic tests, the server was functioning fine with as many as 1000 concurrent builds (the server with 8 cores, 32 GB of total memory running under Linux, and a MySQL server running on a separate comparable machine).

The server load produced by each build depends on the amount of this build's data (logs, tests, and failure details, inspections/duplicates issues number, and so on). Keeping the amount of data reasonably constrained (publishing large outputs as build artifacts, not printing those into standard output, tweaking inspection profiles to report a limited set of the most important inspection hits, and so on) will help scale the server to handle more concurrent builds.

If you need many more agents/parallel builds, it is recommended to use a [multinode setup](multinode-setup.md) and distribute the projects between them.

>We are constantly improving the TeamCity performance and are willing to work closely with organizations running large TeamCity installations. This way, we can study performance issues and improve TeamCity to handle larger loads.
>
{type="note"}

See also related posts: [maximum number of agents which TeamCity can handle](http://blog.jetbrains.com/teamcity/2015/08/benchmarking-teamcity/) and [description of a substantial TeamCity setup](http://blogs.jetbrains.com/teamcity/2011/09/05/improving-performance-and-scalability-of-your-teamcity-server/).

### Configuring TeamCity Server for Performance

This section gives recommendations on tweaking the TeamCity server setup for better performance. It assumes that the server is already [configured for production use](installing-and-configuring-the-teamcity-server.md#Configuring+Server+for+Production+Use).

* Regularly review [Server Health](server-health.md) reports (including hidden ones).
* Use a separate [reverse proxy server](how-to.md#Set+Up+TeamCity+behind+a+Proxy+Server) (for example, NGINX) to handle HTTPS.
* Use a separate server for the external database. Monitor the database performance.
* Monitor the server's CPU and I/O performance. Increase hardware resources as necessary.
* Make sure [clean-up](clean-up.md) is configured for all the projects with a due retention policy. Check __Administration | Clean-Up__ to make sure that the clean-up is performed regularly.
* Consider ensuring good I/O performance for the `<[TeamCity Data Directory](teamcity-data-directory.md)>/system/caches` directory: for example, move it to a separate local drive.
* Regularly [archive](archiving-projects.md) obsolete projects.
* Regularly review the installed not bundled plugins and remove those not essential for the server operation.
* Consider using [agent-side checkout](vcs-checkout-mode.md) whenever possible.
* Make sure the build logs occupy a reasonable amount of space (tens of megabytes at most, but better less than 10 MB).
* If lots of VCS roots are configured on the server, consider configuring [repository commit hooks](configuring-vcs-post-commit-hooks-for-teamcity.md) instead of using polling for changes; or at least increase [VCS polling interval](configuring-vcs-roots.md#Common+VCS+Root+Properties) to 300 seconds or more.
* If the server is used by more than 1000 users, consider reducing the frequency of background UI requests by increasing the [UI refresh intervals](teamcity-tweaks.md#Web+Page+Refresh+Interval).
* When regularly exceeding 500 concurrently running builds which log a lot of data, consider switching to a [multinode setup](multinode-setup.md).

## TeamCity Agent Requirements

This section lists requirements to the environment and OS user suitable for running a TeamCity [build agent](build-agent.md) process.

### Common Requirements

The agent Java process has to:
* be able to open outbound HTTP connections to the server URL that is configured via the `serverUrl` property in the [`buildAgent.properties`](build-agent-configuration.md) file (typically the same address you use in the browser to view the TeamCity UI).  
  Sending requests to the paths under the configured URL should not be limited. See also the recommended [reverse proxy settings](how-to.md#Set+Up+TeamCity+behind+a+Proxy+Server). Ensure that any firewalls installed on the agent or server machines, network configuration and proxies (if any) comply with these requirements.
* have full permissions (read/write/delete) to the following directories recursively: [`<agent home>`](agent-home-directory.md) (necessary for automatic agent upgrade and agent tools support), [`<agent work>`](agent-work-directory.md), [`<agent temp>`](agent-home-directory.md#Agent+Directories), and agent system directory (set by `workDir`, `tempDir`, and `systemDir` parameters in the `buildAgent.properties` file).
* be able to launch processes (to run builds).
* be able to launch nested processes with the following parent process exit (this is used during the agent upgrade).

### Requirements to Windows-based Agents

The Windows user who runs an agent process must:
* be able to run as a Windows service (see also this [Microsoft Knowledge Base article](https://support.microsoft.com/en-us/help/325349/how-to-grant-users-rights-to-manage-services-in-windows-server-2003)).
* be able to start/stop service (to run as Windows service, necessary for the agent upgrade to work, see also [Microsoft KB article](https://support.microsoft.com/en-us/help/325349/how-to-grant-users-rights-to-manage-services-in-windows-server-2003)).
* be able to debug programs (required for take process dump functionality).
* be able to reboot the machine (required for agent reboot functionality).
* be a member of the _Performance Monitor Users_ group (to be able to [monitor performance](performance-monitor.md) of a build agent run as a Windows [service](setting-up-and-running-additional-build-agents.md#Build+Agent+as+a+Windows+Service)).

Note on granting rights:

To learn how to grant users necessary rights, see [this article](https://support.microsoft.com/en-us/kb/325349). You can assign rights to manage services with the Microsoft [SubInACL](http://www.microsoft.com/downloads/details.aspx?FamilyID=e8ba3e56-d8fe-4a91-93cf-ed6985e3927b&displaylang=en) utility, a command-line tool enabling administrators to directly edit security information. The tool uses the following syntax:

```Shell
SUBINACL /SERVICE \\MachineName\ServiceName /GRANT=[DomainName]UserName[=Access]

```

For example, to grant start/stop rights, you might need to execute the command:

```Shell
subinacl.exe /service TCBuildAgent /grant=<user login name>=PTO

```

### Requirements to Linux-based Agents

The Linux user who runs an agent process must be able to run the `shutdown` command (for the agent machine reboot and shutdown functionality when running in a cloud environment). If you are using `systemd`, it should not kill the processes on the main process exit (use [`RemainAfterExit=yes`](https://serverfault.com/questions/660063/teamcity-build-agent-gets-killed-by-systemd-when-upgrading)). See also [how to set up automatic agent start under Linux](setting-up-and-running-additional-build-agents.md#Automatic+Agent+Start+under+Linux).

### Build-related Permissions

Every build process is launched by a TeamCity agent. It shares the environment and is executed under the OS user used by the TeamCity agent. Ensure that the TeamCity agent is configured accordingly. See [Known Issues](known-issues.md#Windows+service+limitations) for related Windows Service Limitations.

<anchor name="SettingupandRunningAdditionalBuildAgents-ServerDataTransfers"/>
<anchor name="SettingupandRunningAdditionalBuildAgents-Agent-ServerDataTransfers"/>

### Estimating Hardware Requirements for Agent

The __agent__ hardware requirements are determined by the builds they run. Running a TeamCity agent software introduces a requirement for additional CPU time (but it can usually be neglected comparing to the build process CPU requirements) and additional memory: about 500 MB. The disk space required corresponds to the disk usage by the builds running on the agent (sources checkouts, downloaded artifacts, the disk space consumed during the build; all that combined for the regularly occurring builds).

Although you can run a build agent on the same machine as the TeamCity server, the recommended approach is to use a separate machine (it may be virtual) for each build agent. If you chose to install several agents on the [same machine](setting-up-and-running-additional-build-agents.md#Installing+Several+Build+Agents+on+the+Same+Machine), consider the possible CPU, disk, memory or network bottlenecks that might occur. The [Performance Monitor](performance-monitor.md) build feature can help analyze live data.

If you consider cloud deployment for TeamCity agents (for example, on Amazon EC2), also see [this section](setting-up-teamcity-for-amazon-ec2.md#Estimating+EC2+Costs)

>__Agents' setup in JetBrains internal TeamCity installation__  
>We use both separate machines running single agents and dedicated "servers" running several virtual machines, one per agent. After experimenting with the hardware and software, we settled on a configuration where each core7i physical machine runs 3 virtual agents: each uses a separate hard disk as our builds are mostly Java and greatly depend on the HDD performance.

## Estimating Network Traffic Between Server and Agents

The traffic mostly depends on your settings as some of them include transferring binaries between the agent and the server.

The most important flows of traffic between an agent and the server are:
* Agent retrieves commands from the server: these are typically build start tasks which include a dump of the build configuration settings and the full set of build parameters. These parameters can be reviewed on the build's __[Parameters](working-with-build-results.md#Parameters)__ tab.
* Agent periodically sends current status data to the server (this includes all the agents parameters which can be reviewed on the agent's __[Agent Parameters](viewing-build-agent-details.md#Agent+Parameters)__ tab).
* During the build, the agent sends build log messages and parameters data back to the server. These can be reviewed on the __[Build Log](working-with-build-results.md#Build+Log)__ and __[Parameters](working-with-build-results.md#Parameters)__ tabs of the build.
* (when the server-side checkout mode is used) The agent downloads the sources before the build (as a full or incremental patch) from the server.
* (when an [artifact dependency](artifact-dependencies.md) is configured) The agent downloads build artifacts of other builds from the server before starting a build.
* (when artifacts are configured for a build) The agent uploads build artifacts to the server.
* Some runners (like coverage or code analysis) include automatic uploading of their results' reports to the server.

## Estimating External Database Capacity

If a TeamCity server is used extensively, the database performance starts to play a greater role. To ensure production performance and reliability, an [external database](setting-up-external-database.md) must be used. Its size and performance are crucial aspects to consider.

If you decide to run an external database on the same machine as the server (__not recommended__), choose the server's hardware with the database engine requirements in mind.

It is hard to estimate exact requirements when setting up or migrating to an external database, as the required capacity greatly depends on how TeamCity is used. The database size requirements naturally vary based on the amount of stored data (number of builds, number of tests, and so on). The active database usage can be estimated at several gigabytes of data per year.

### Database Size

The size of the database depends on:
* how many builds are started daily
* how many tests are reported from builds
* [clean-up](clean-up.md) rules (retention policy)
* clean-up schedule

The recommended initial size is 4 GB. When migrating from the internal database, we suggest at least doubling the size of the current internal database. For example, the size of the external database (without the redo log files) of the internal TeamCity server in JetBrains is about 50 GB. Setting your database to grow automatically helps increase file sizes to a predetermined limit when necessary, which minimizes the effort to monitor the disk space.

Allocating 1 GB for the redo log (see the table below) and undo files is sufficient in most cases.

### Database Performance

The following factors can affect the database performance:
* type of database (RDBMS)
* number of agents (number of builds running in parallel)
* number of web pages opened by all users
* [clean-up](clean-up.md) rules (retention policy)

It is advised to place the [`TeamCity Data Directory`](teamcity-data-directory.md) and database data files on physically different hard disks (even when both the TeamCity server and RDBMS share the same host).

Placing redo logs on a separate physical disk is also recommended — especially if there are 50 or more agents.

### Database-Specific Notes

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