[//]: # (title: Create Pipeline)
[//]: # (auxiliary-id: Create Pipeline)

In TeamCity terms, a pipeline is called a _build chain_.

Builds perform various CI/CD jobs. When you connect them into a sequence, they form a chain. Builds inside a chain can use the same revision of the source project and pass artifacts to one another. These chains can be quite complex and contain dozens of builds connected in series or in parallel. They are often designed to compile, test, and deploy a certain project, but you can create them for any other goal.

In this tutorial, we will learn how to create a simple chain like this:

<img src="buildChainSimple.png" width="674" alt="Simple build chain in TeamCity"/>

It builds a Spring Boot application, creates its Docker image, and then checks it with two parallel sets of tests.

## Import Sample Project

>This section explains how to set up a sample project on your own server. To proceed with setting up a build chain, go [here](#Configure+Artifact+Dependencies).

TeamCity allows saving project's settings in Kotlin or XML. That's what we did with the sample project, so you can easily import it to your server. To do this:
1. Go to __Administration | Projects__ and click __Create project__.
2. In the _Repository URL_ field, enter the [sample project](https://github.com/mkjetbrains/TodoApp-NoChain-KTS) URL and click __Proceed__.
3. TeamCity will detect the `settings.kts` file, which corresponds to a TeamCity project's settings saved in Kotlin format. Leave the default settings and proceed.
4. TeamCity will import the sample project's settings and redirect you to its __Global Settings__ page. Here, you can find the _TodoBackend_ subproject. Click it to see all the created build configurations.

Now you can start chaining these builds.

## Configure Artifact Dependencies

The _TodoApp_ config compiles a `.jar` application and publishes it to the `build/libs/` directory on a [build agent](build-agent.md). The _TodoImage_ config builds a Docker image out of this `.jar`.

To connect these two configs together, we need to create an __artifact dependency__ between them. This way, when each new _TodoApp_ build finishes and produces an artifact, TeamCity will start a new _TodoImage_ build that will process this artifact.

>__Important to understand__  
>If builds are connected by dependencies (which means they form a chain), they always run together. When you run any build from a chain, whether it's the last one or medium one, TeamCity queues all the other chained builds into a sequence, according to their dependencies.  
>As you can see on the diagram above, _TodoImage_ always runs after _TodoApp_; _Test1_ and _Test2_ start only after _TodoImage_ finishes and run in parallel to each other.

