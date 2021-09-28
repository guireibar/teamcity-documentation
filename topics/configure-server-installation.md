[//]: # (title: Configure Server Installation)
[//]: # (auxiliary-id: Configure Server Installation)

## Troubleshoot TeamCity Installation

Upon successful installation, the TeamCity server web UI can be accessed via a web browser. The default address that can be used to access TeamCity from the same machine depends on the installation package and installation options. (Port `8111` is used for `.exe` and `.tar.gz` installations, unless another port is specified).

If the web UI cannot be accessed, check the following:
* The _TeamCity Server_ service is running (if you installed TeamCity as a Windows service).
* The TeamCity server process (Tomcat) is running (a Java process run in the `<[TeamCity Home](teamcity-home-directory.md)>/bin` directory).
* The console output, if you run the server from a console.
* The `teamcity-server.log` and other files in the `<[TeamCity Home](teamcity-home-directory.md)>\logs` directory for error messages.

One of the most common issues with the server installation is using a port that is already used by another program. See [how to change the default port](#Changing+Server+Port).

## Changing Server Port

If another application uses the same port as the TeamCity server, the TeamCity server (Tomcat server) won't start. This will result in "_Address already in use_" errors in the server logs or server console.

If you install a TeamCity server from `.exe`, you can customize the port in the installation wizard.

To change the port of the installed/unpacked server, open the `<[TeamCity Home](teamcity-home-directory.md)>/conf/server.xml` file and set another number in the not commented `<Connector>` XML node:

```XML
<Connector port="8111" ...

```

To apply the changes, [restart the server](start-teamcity-server.md).

If you change the port of an operational TeamCity server, you also need to change it in all the stored URLs of the server (browser bookmarks, agents' `serverUrl` [property](configure-agent-installation.md), URL in user's IDEs, the _Server URL_ setting on the __Administration | Global Settings__ page).  
If you run another Tomcat server on the same machine, you might also need to change other service ports of the Tomcat server (search for `port=` in the `server.xml` file).

If you want to use the HTTPS protocol, it should be enabled separately. The process is specific to the web server used (by default, Tomcat). See notes on how to [configure HTTPS for TeamCity web UI](how-to.md#Configure+HTTPS+for+TeamCity+Web+UI).

## Changing Server Context

By default, the TeamCity server is accessible under the root context of the server address (for example, [`http://localhost:8111/`](http://localhost:8111/) ). To make it available under a nested path instead (for example, [`http://localhost:8111/teamcity/`](http://localhost:8111/teamcity/) ), you need to:
1. Stop the TeamCity server.
2. Rename the `<[TeamCity Home](teamcity-home-directory.md)>\webapps\ROOT` directory to `<[TeamCity Home](teamcity-home-directory.md)>\webapps\teamcity`.
3. [Start the TeamCity server](start-teamcity-server.md).

>After this change, [automatic update](upgrading-teamcity-server-and-agents.md#Automatic+Update) will be disabled for your installation and you will have to upgrade TeamCity [manually](upgrading-teamcity-server-and-agents.md#Manual+Upgrade).
> 
{type="note"}

<anchor name="InstallingandConfiguringtheTeamCityServer-SettingUpMemorysettingsforTeamCityServer"/>

## Configure Memory Settings for TeamCity Server

TeamCity Server has the main process which can also launch child processes. Child processes use available memory on the machine. This section covers the memory settings of the main TeamCity server process only, as it requires special configuration.

As a JVM application, the TeamCity main server process utilizes only memory available to the JVM. The required memory depends on the JVM bitness (32- or 64-bit). The memory used by JVM usually consists of: heap (configured via `-Xmx`) and metaspace (limited by the amount of available native memory), internal JVM (usually tens of Mb), and OS-dependent memory features like memory-mapped files. TeamCity mostly depends on the heap memory. This setting can be manually configured by [passing](server-startup-properties.md#JVM+Options) the `-Xmx` (heap space) option to the JVM running the TeamCity server.

Once you start [using TeamCity for production purposes](#Configure+Server+for+Production+Use) or if you want to load the server during evaluation, you should manually set the appropriate memory settings for the TeamCity server.

To __change the memory settings__, refer to [this article](server-startup-properties.md#JVM+Options). In most cases, it will result in setting `TEAMCITY_SERVER_MEM_OPTS` environment variable to a value like `-Xmx750m`.

We recommend removing the `-XX:MaxPermSize` JVM option from the `TEAMCITY_SERVER_MEM_OPTS` environment variable, if previously configured, since it is ignored in Java 8.

If an `OutOfMemory` errors occur or you consistently see a memory-related warning in the TeamCity UI, increase the setting to the next level.
* __Minimum setting__: for 32-bit Java `-Xmx750m `, for 64-bit Java (bundled) `-Xmx1024m`.
* __Recommended setting__ for __medium__ server use: for 32-bit Java `-Xmx1024m`. Greater settings with the 32-bit Java can cause `OutOfMemoryError` with "Native memory allocation (malloc) failed" JVM crashes or "Unable to create new native thread" messages. For 64-bit Java `-Xmx2048m`.
* __Recommended setting__ for __large__ server use (64-bit Java should be used): `-Xmx4g -XX:ReservedCodeCacheSize=450m`. These settings should be suitable for an installation with up to two hundreds of agents and thousands of build configurations. Custom plugins might require increasing the value defined via the `Xmx` parameter.
* __Maximum settings__ for large-scale server use (64-bit Java should be used): `-Xmx10g -XX:ReservedCodeCacheSize=640m`. Greater values can be used for larger TeamCity installations. However, generally it is not recommended to use values greater than `10g` without consulting TeamCity support.

>__Tips__:  
>* The 32-bit JVM can reliably work with up to 1 Gb heap memory (`Xmx1024m`). (This can be increased to `-Xmx1200m`, but JVM under Windows might crash occasionally with this setting.) If more memory is necessary, the 64-bit JVM should be used with not less than 2 Gb assigned (`-Xmx2048m`). Unless the TeamCity server runs with more than 100 agents or serves very active builds / thousands of users, it's unlikely that you will need to dedicate more than 4 Gb of memory to the TeamCity process.
>* A rule of thumb is that the 64-bit JVM should be assigned twice as much memory as the 32-bit for the same application. If you switch to the 64-bit JVM (for example, on upgrading TeamCity), make sure to adjust the memory setting (`-Xmx`) accordingly. It does not make sense to switch to 64-bit if you dedicate less than the double amount of memory to the application.
>* Large TeamCity installations might benefit from fine-tuning of the memory settings. The amount of memory dedicated to the TeamCity server JVM should not regularly exceed 60% of the total available physical memory on the machine (to allow for nested process and OS-level caches usage). Also, with heaps (`–Xmx`) set to more than 8 Gb, if the machine has many CPU cores (for example, more than 8) and current CPU usage is below 60%, enabling G1 JVM garbage collector via the `-XX:+UseG1GC` JVM option might reduce the length of stop-the-world GC pauses.  
>The recommended approach is to start with the default settings and monitor the used memory using the __Administration | Diagnostics__ page. If the server uses more than 80% of memory consistently without drops for tens of minutes, that is probably a sign to increase the `-Xmx` memory value by another 20%.

[//]: # (Internal note. Do not delete. "Installing and Configuring the TeamCity Serverd172e1122.txt")

## Configure TeamCity Data Directory

The default placement of the TeamCity Data Directory can be changed. See [this article](teamcity-data-directory.md) for details.

### Configure Server for Production Use

An out-of-the-box TeamCity server installation is suitable for evaluation purposes. For production use, you will need to perform additional configuration. It typically includes these steps:
* Check that the server is using a [proper server port](#Changing+Server+Port) and configure [access via HTTPS](how-to.md#Configure+HTTPS+for+TeamCity+Web+UI).
* Make sure the TeamCity server URL and email server settings are correct.
* Configure the server process for OS-dependent [autostart](start-teamcity-server.md) on machine reboot.
* Use a reliable storage for [TeamCity Data Directory](teamcity-data-directory.md).
* Use an [external database](set-up-external-database.md).
* Configure [recommended memory settings](#Configure+Memory+Settings+for+TeamCity+Server). Use "maximum settings" for active or growing servers.
* Plan for regular [backups](teamcity-data-backup.md).
* Plan for regular [upgrades](upgrading-teamcity-server-and-agents.md) to the latest TeamCity releases.
* Consider adding the `teamcity.installation.completed=true` line into the `<[TeamCity Data Directory](teamcity-data-directory.md)>\conf\teamcity-startup.properties` file — this will prevent the server from creating an administrator user if no such user is found.

Make sure to read the [notes on configuring the server for performance](how-to.md#Configuring+TeamCity+Server+for+Performance) and [security notes](security-notes.md).

## Extra Recommendations

* After a successful server start, any TeamCity page request will prompt for the server administrator's username and password. Make sure no one can access the server pages until the administrator account is set up.
* If you have a lot of projects or build configurations, we recommend you avoid using the __Default agent__ in order to free up the TeamCity server resources. The TeamCity Administrator can [disable](build-agents-configuration-and-maintenance.md#Enabling%2FDisabling+Agents+via+UI) the default agent on the __Agents__ page of the web UI.


