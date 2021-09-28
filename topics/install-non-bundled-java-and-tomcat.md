[//]: # (title: Install Non-Bundled Java and Tomcat)
[//]: # (auxiliary-id: Install Non-Bundled Java and Tomcat)

TeamCity Server is a JVM web application that runs on a Tomcat application server.

TeamCity Server requires a Java SE JRE installation to run. A compatible JRE version is bundled in the TeamCity `.exe` installer but needs to be installed separately when using other distributions. Check the current [recommended version](supported-platforms-and-environments.md#TeamCity+Server) and install it as described [below](#Install+Java).

The `.exe` and `.tar.gz` distributions include a Tomcat version tested to work fine with the respective version of TeamCity. You can use an [alternative Tomcat version](#Use+Another+Version+of+Tomcat), but other combinations are not guaranteed to work correctly.

## Install Java

TeamCity selects the Java version to run the server process as follows:
* By default, if your TeamCity installation has a bundled JRE (the `<[TeamCity Home](teamcity-home-directory.md)>\jre` directory exists), it will be used to run the TeamCity server process. To use a different JRE, specify its path via the `TEAMCITY_JRE` environment variable.
* If there is no `<[TeamCity Home](teamcity-home-directory.md)>\jre` directory present, TeamCity looks for the `JRE_HOME` or `JAVA_HOME` environment variable pointing to the installation directory of JRE or JVM (Java SDK) respectively. If both variables are declared, JRE will be used.

The necessary steps to update the Java installation depend on the distribution used.

If your TeamCity installation has a bundled JRE (there is the `<[TeamCity Home](teamcity-home-directory.md)>\jre` directory), update it by installing a newer JRE per installation instructions and copying the content of the resulting directory to replace the content of the existing `<[TeamCity Home](teamcity-home-directory.md)>\jre` directory.   
  If you also run a TeamCity agent from the `<[TeamCity Home](teamcity-home-directory.md)>\buildAgent` directory, install JDK (Java SDK) installation instead of JRE and copy content of JDK installation directory into `<[TeamCity Home](teamcity-home-directory.md)>\jre`.

[//]: # (Internal note. Do not delete. "Installing and Configuring the TeamCity Serverd172e906.txt")

>If you use a different Java version, specified via an environment variable (`TEAMCITY_JRE`, `JRE_HOME`, or `JAVA_HOME`), make sure it is available for the process launching the TeamCity server (it is recommended to set a global OS environment variable and restart the system). The variable should point to the home directory of the installed JRE or JVM (Java SDK) respectively.

### Update from 32-bit to 64-bit Java

TeamCity server is bundled with the 64-bit JVM but can run under both 32- and 64-bit versions.

If you need to update 32-bit Java to 64-bit JVM, note that the memory usage is almost doubled when switching from 32- to 64-bit. Make sure to specify at least twice as much memory as for 32-bit JVM. Read how to [change memory settings](configure-server-installation.md#Configure+Memory+Settings+for+TeamCity+Server).

To update to 64-bit Java, either use the bundled version of Java or:
* Update Java to be used by the server.
* [Set JVM memory options](server-startup-properties.md). It is recommended to set the following options for 64-bit JVM: `-Xmx4g -XX:ReservedCodeCacheSize=450m`.

## Use Another Version of Tomcat

To use another version of the Tomcat web server instead of the bundled one, you need to perform the Tomcat upgrade/patch.

When using an `.exe` distribution of TeamCity, we suggest that you:
* Back up the current [TeamCity Home](teamcity-home-directory.md).
* Delete/move from the TeamCity Home directories that are also present in the Tomcat distribution.
* Unpack the Tomcat distribution to the TeamCity Home directory.
* Copy TeamCity-specific files from the previously backed-up/moved directories to the TeamCity Home:
    * files under `bin` which are not present in the Tomcat distribution
    * review differences between the default Tomcat `conf` directory and one from TeamCity, update Tomcat files with TeamCity-specific settings (`teamcity-*` files, and portions of `server.xml`)
    * delete the default Tomcat `webapps/ROOT` directory and replace it with the one provided by TeamCity