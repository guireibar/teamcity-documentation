[//]: # (title: Install and Start TeamCity Server)
[//]: # (auxiliary-id: Install and Start TeamCity Server;Installation;Installing and Configuring the TeamCity Server)

Your instance of TeamCity Cloud is installed automatically after your register an account: no extra actions are required. After the server is ready, an invitation link will be sent to your email.
{product="tcc"}

## Preliminaries
{product="tc"}

TeamCity Server is a web application responsible for the core functionality of TeamCity. It provides a user interface, distributes the jobs (builds) to TeamCity agents, and aggregates their results. This section contains articles related to installing and starting your own instance of TeamCity Server.

Before installing the server, make sure to:
1. Estimate your [system requirements](how-to.md#Estimate+Hardware+Requirements+for+TeamCity).
2. Read about [supported platforms](supported-platforms-and-environments.md).
3. Select a convenient installation package, as described [below](#Select+TeamCity+Installation+Package).

## Select TeamCity Installation Package
{product="tc"}

TeamCity installation package is identical for both Professional and Enterprise Editions.

The [TeamCity download](http://www.jetbrains.com/teamcity/download/) page on the official JetBrains site provides the following installation options:

<table><tr>

<td>

Target

</td>

<td>

Option

</td>

<td>

Note

</td></tr><tr>

<td>

[Windows](install-teamcity-server-on-windows.md)

</td>

<td>

`TeamCity<version_number>.exe`

</td>

<td>

Executable Windows installer bundled with Tomcat and Java 1.8 JRE.

</td></tr><tr>

<td>

[Windows](install-teamcity-server-on-windows.md) and [Linux/macOS](install-teamcity-server-on-linux-and-macos.md)

</td>

<td>

`TeamCity<version_number>.tar.gz`

</td>

<td>

Archive for manual installation bundled with a Tomcat servlet container.

</td></tr><tr>

<td>

[Docker (Linux, Windows)](https://hub.docker.com/r/jetbrains/teamcity-server/)

</td>

<td>

Docker

</td>

<td>

The official JetBrains TeamCity server Docker image.

</td></tr>

</table>