[//]: # (title: Start TeamCity Server)
[//]: # (auxiliary-id: Start TeamCity Server)


After installation, the TeamCity web UI can be accessed via a web browser. The default addresses are [`http://localhost/`](http://localhost/) for Windows distribution and [`http://localhost:8111/`](http://localhost:8111/) for the `tar.gz` distribution.

If you cannot access the TeamCity web UI after successful installation, please refer to the [Troubleshooting TeamCity Installation](#Troubleshooting+TeamCity+Installation) section.

The build server and one build agent will be installed by default for Windows, Linux or macOS. If you need more build agents, refer to the [Installing Additional Build Agents](setting-up-and-running-additional-build-agents.md) section.


>During the server setup you can select either an internal database or an existing external database. By default, TeamCity uses an HSQLDB database that does not require configuring. This database suites the purposes of testing and evaluating the system. For [production purposes](), using a standalone external database is highly recommended.
>
{type="warning"}