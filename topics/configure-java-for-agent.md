[//]: # (title: Configure Java for Agent)
[//]: # (auxiliary-id: Configure Java for Agent)

A TeamCity build agent is a Java application ([supported Java versions](supported-platforms-and-environments.md#TeamCity+Agent)).

A build agent contains two processes:
* Agent Launcher — a Java process that launches the agent process.
* Agent — the main process for a Build Agent; runs as a child process for the agent launcher.

The (Windows) `.exe` TeamCity distribution comes bundled with 64-bit Amazon Corretto 8.   
If you run a previous version of the TeamCity agent, you will need to repeat the agent installation to update the JVM.

Using 64-bit JDK (not JRE) is recommended. JDK is required for some build runners like [IntelliJ IDEA Project](intellij-idea-project.md), Java [Inspections](inspections.md), and [Duplicates](duplicates-finder-java.md). If you do not have Java builds, you can install JRE instead of JDK.   
On switching from 32- to 64-bit, you might need to double the `-Xmx` memory value for the main agent process (see related notes for [build agents](configuring-build-agent-startup-properties.md) and [server](installing-and-configuring-the-teamcity-server.md#Setting+Up+Memory+settings+for+TeamCity+Server)).
{product="tc"}

Using 64-bit JDK (not JRE) is recommended. JDK is required for some build runners like [IntelliJ IDEA Project](intellij-idea-project.md), Java [Inspections](inspections.md), and [Duplicates](duplicates-finder-java.md). If you do not have Java builds, you can install JRE instead of JDK.   
On switching from 32- to 64-bit, you might need to double the `-Xmx` memory value for the main agent process (see [related notes](configuring-build-agent-startup-properties.md)).
{product="tcc"}

<anchor name="java-paths"/>

For the `.zip` agent installation you need to install the appropriate Java version. Make it available via `PATH` or available in one of the following places:
* the `<Agent home>/jre` directory
* in the directory pointed to by the `TEAMCITY_JRE`, `JAVA_HOME` or `JRE_HOME` environment variables (check that you only have one of the variables defined).
* if you plan to run the agent via Windows service, make sure to set the `wrapper.java.command` property in the `<agent home>\launcher\conf\wrapper.conf` file to a valid path to the Java executable

<anchor name="SettingupandRunningAdditionalBuildAgents-UpgradingJavaonAgents"/>

### Upgrading Java on Agents

If you are trying to launch an agent, and it is not able to find the required Java version (currently Java 8) in any of the [default locations](setting-up-and-running-additional-build-agents.md#java-paths), the agent will report an error on starting, the process will not launch, and the agent will be shown as disconnected in the TeamCity UI.

If a build agent uses a Java version older than Java 8, you will see the corresponding warning on the agent's page and a [health item](server-health.md) in the web UI.

It is recommended to use latest Java 8, 64-bit version.  
OpenJDK 8 (for example, bundled [Amazon Corretto](https://aws.amazon.com/corretto/)) 1.8.0_161 or later. [Oracle Java 8](http://www.oracle.com/technetwork/java/javase/downloads/) is also supported.

To update Java on agents, do one of the following:
* If the agent details page in the TeamCity UI displays a Java version note with the corresponding action, you can switch to using newer Java: if the appropriate Java version of the same bitness as the current one is detected on the agent, the agent page provides an action to switch to using that Java automatically. Upon the action invocation, the agent process is restarted (once the agent becomes idle, i.e. finishes the current build if there is one) using the new Java.
* (Windows) Since the build agent Windows installer comes bundled with the required Java, you can just manually reinstall the agent using the Windows installer (`.exe`) obtained from the TeamCity server __Agents__ page. See [installation instructions](setting-up-and-running-additional-build-agents.md#Installing+via+Windows+installer). It is important to uninstall the previous version of the agent before installing the updated agent: invoke `Uninstall.exe` in the agent home directory, clear all the "_Remove_" checkboxes, and click __Uninstall__.
* Install a required Java on the agent into one of the standard locations, and restart the agent — the agent should then detect it and provide an action to use a newer Java in the web UI (see above).
* Install a required Java on the agent and [configure the agent](#Configuring+Java) to use it.

<note>

In a rare case of updating the Java for the process that launches the TeamCity agent, use one of the options for the agent Java upgrade. Another way for an agent started as a __Windows service__, is to stop the service, change the `wrapper.java.command` variable in `buildAgent\launcher\conf\wrapper.conf` to point to the new `java.exe` binary, and restart the service.
</note>

