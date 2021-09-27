[//]: # (title: Install TeamCity Server on Linux and macOS)
[//]: # (auxiliary-id: Install TeamCity Server on Linux and macOS)

## Download TeamCity Server

Go to the [JetBrains website](http://www.jetbrains.com/teamcity/download/) and download the __.tar.gz distribution__ with the "portable" version of the TeamCity server.

Or, install it from a __Docker image__. All the information related to the TeamCity Server Docker images are described on [Docker Hub](https://hub.docker.com/r/jetbrains/teamcity-server/).

Specifics of the `.tar.gz` distributions:
* They include a Tomcat version tested to work fine with the respective version of TeamCity. You can use an alternative Tomcat version, but other combinations are not guaranteed to work correctly.
* It is possible to configure the installation by changing the startup script and JRE options.
* They come bundled with a build agent distribution and a startup script which allows for easy TeamCity server evaluation with one agent.
* They come bundled with the devPackage for the [TeamCity plugin development](https://plugins.jetbrains.com/docs/teamcity/developing-teamcity-plugins.html).

## Install from tar.gz

>We recommend running the TeamCity server under a dedicated user account.

To install the server, unpack the `TeamCity<version number>.tar.gz` archive. You can use the `tar xfz TeamCity<version number>.tar.gz` command \*.

>\* We suggest that you use GNU tar to unpack. For example, Solaris 10 tar is reported to truncate too long file names and may cause a `ClassNotFoundException` when using the server after such unpacking. Consider getting GNU tar at [Solaris packages](http://sunfreeware.com/) or using the `gtar xfz` command.

Ensure that JRE or JDK are installed and the `JAVA_HOME` environment variable is pointing to the Java installation directory (see [recommended Java versions](supported-platforms-and-environments.md#TeamCity+Server)).
