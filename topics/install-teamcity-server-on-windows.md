[//]: # (title: Install TeamCity Server on Windows)
[//]: # (auxiliary-id: Install TeamCity Server on Windows)

## Download TeamCity Server

Go to the [JetBrains website](http://www.jetbrains.com/teamcity/download/) and download one of the following TeamCity Server distributions:
* __.exe__: executable which provides the installation wizard for Windows platforms and allows installing the server as a Windows service.
* __.tar.gz__: archive with a "portable" version.

Or, install it from a __Docker image__. All the information related to the TeamCity Server Docker images are described on [Docker Hub](https://hub.docker.com/r/jetbrains/teamcity-server/).

Specifics of the `.exe` and `.tar.gz` distributions:
* They include a Tomcat version tested to work fine with the respective version of TeamCity. You can use an [alternative Tomcat version](install-non-bundled-java-and-tomcat.md#Use+Another+Version+of+Tomcat), but other combinations are not guaranteed to work correctly.
* On Windows, they provide better error reporting for some scenarios (like a missing Java installation).
* It is possible to configure the installation by changing the startup script and JRE options.
* They come bundled with a build agent distribution and a startup script which allows for easy TeamCity server evaluation with one agent.
* They come bundled with the devPackage for the [TeamCity plugin development](https://plugins.jetbrains.com/docs/teamcity/developing-teamcity-plugins.html).

## Install from Executable

To install the TeamCity server, run the executable (`.exe`) file and follow the installation instructions.

You have options to install the TeamCity web server and one build agent that can be run as a Windows service. If you opted to install the services, you can use the standard Windows _Services_ app to manage the service. Otherwise, use standard [scripts](start-teamcity-server.md).

Make sure the user account specified for the service has:
* the [log on as service](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc794944(v=ws.10)?redirectedfrom=MSDN) right
* write permissions for the [TeamCity Data Directory](teamcity-data-directory.md)
* write permissions for the [TeamCity Home](teamcity-home-directory.md), that is directory where TeamCity has been installed
* all the permissions necessary to work with external systems like VCSs

By default, the Windows service is installed under the `SYSTEM` account. To change it, use the Services applet (__Control Panel | Administrative Tools | Services__).

If you did not change the default port (`8111`) during the installation, the TeamCity web UI can be accessed via [`http://localhost/`](http://localhost/) in a web browser running on the same machine where the server is installed. Note that port 8111 can be used by other programs. In this case, you can specify another port during the installation and use [`http://localhost:<port>/`](http://localhost:<port>/) address in the browser.

>During the server setup, you can select either an internal database or an existing external database. By default, TeamCity uses an HSQLDB database that does not require configuring. This database suites the purposes of testing and evaluating the system. For [production purposes](configure-server-installation.md#Configuring+Server+for+Production+Use), using a standalone external database is highly recommended.
>
{type="warning"}

If you want to edit the TeamCity server's service parameters, memory settings or system properties after the installation, refer to the [this article](server-startup-properties.md).

## Install from tar.gz

>We recommend running the TeamCity server under a dedicated user account.

To install the server, unpack the `TeamCity<version number>.tar.gz` archive. You can use the `tar xfz TeamCity<version number>.tar.gz` command under Linux \* and WinZip, WinRar, or similar utility under Windows.

>\* We suggest that you use GNU tar to unpack. For example, Solaris 10 tar is reported to truncate too long file names and may cause a `ClassNotFoundException` when using the server after such unpacking. Consider getting GNU tar at [Solaris packages](http://sunfreeware.com/) or using the `gtar xfz` command.

Ensure that JRE or JDK are installed and the `JAVA_HOME` environment variable is pointing to the Java installation directory (see [recommended Java versions](supported-platforms-and-environments.md#TeamCity+Server)).