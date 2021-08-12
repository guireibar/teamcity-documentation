[//]: # (title: Create Pipeline)
[//]: # (auxiliary-id: Create Pipeline)

_This tutorial assumes that you have already installed and started your trial TeamCity instance as described [here](quick-setup-guide.md). We also suggest that you learn how to [run a simple build](configure-and-run-your-first-build.md)._

In TeamCity terms, a pipeline is called a _build chain_.

Builds perform various CI/CD jobs. When you connect them into a sequence, they form a chain. Builds inside a chain can use the same revision of the source project and pass artifacts to one another. Such chains can be quite complex and contain dozens of builds connected in series or in parallel. They are often designed to compile, test, and deploy a certain project, but you can create them for any other goal.

In this tutorial, we will explain the basics of creating pipelines in TeamCity and learn how to create a chain like this:

<img src="buildChainSimple.png" width="674" alt="Simple build chain in TeamCity"/>

First sections show how to create a simple chain (1-2) that builds a Spring Boot application and creates its Docker image. In extra sections, you can learn how to extend and improve this chain.

## Import Sample Project

>This section explains how to set up a sample project on your own server. To proceed with setting up a build chain, go [here](#Configure+Artifact+Dependencies).

TeamCity allows saving project's settings in Kotlin or XML. That's what we did with the sample project, so you can easily import it to your server. To do this:
1. Go to __Administration | Projects__ and click __Create project__.
2. In the _Repository URL_ field, enter the [sample project's](https://github.com/mkjetbrains/TodoApp-NoChain-KTS) URL and click __Proceed__.  
   
  >Note that the sample settings lead to a source repository whose editing is restricted. To be able to tweak the source settings during the tutorial, we suggest that you fork both the settings project and the [source code project](https://github.com/mkjetbrains/todoapp-backend), and then respectively adjust the `namesAndTags` and `url` values in the `.teamcity/settings.kts` file. This is completely optional — feel free to use our sample sources for a quick walkthrough.
3. TeamCity will detect the `settings.kts` file, which corresponds to a TeamCity project's settings saved in Kotlin format. Leave the default settings and proceed.
4. TeamCity will import the sample project's settings and redirect you to its __Global Settings__ page. Here, you can find the _TodoBackend_ subproject. Click it to see all the created build configurations.

Now you can start chaining these builds.

## Configure Artifact Dependencies

The _TodoApp_ build configuration compiles a `.jar` application and publishes it to the `build/libs/` directory on a [build agent](build-agent.md). The _TodoImage_ config builds a Docker image out of this `.jar`.

To connect these two configs together, we need to create an __artifact dependency__ between them. This way, when each new _TodoApp_ build finishes and produces an artifact, TeamCity will use this artifact in the following _TodoImage_ build.

__A dependency determines how one build depends on another, and thus is created in the settings of the dependent build__. In our case, it's _TodoImage_. Let's go to its settings and add an artifact dependency:
1. Open the __Dependencies__ settings tab (you might need to click __Show more__ to display this item) and click __Add new artifact dependency__.
2. Select _TodoApp_ as a build config to depend on.
3. Choose to get artifacts from the build from the same chain, as we want to always process the freshest artifact.
4. In _Artifacts rules_, specify that we want to import the specific artifact as `todo.jar` — enter `todo.jar => build/libs/todo.jar`.

>To simplify step 4 in the future, you can use the artifact browser (![ArtifactsBrowserIcon.png](ArtifactsBrowserIcon.png)). When there is at least one finished dependent build which already produced some artifacts, TeamCity can show them in a tree, so you can choose them in a handy way.  
>You can try this approach already: run the first _TodoApp_ build manually and then select `todo.jar` in the artifact browser when performing step 4.

<img src="chaindemo1.png" width="539" alt="Simple build chain in TeamCity"/>

Read about patterns of artifact rules and other details related to artifact dependencies [here](artifact-dependencies.md).

At this point, you can already try starting the first _TodoImage_ build and watch how TeamCity composes and runs the chain of two builds. Note that to compose a Docker image, a [TeamCity agent](build-agent.md) needs to have [Docker](https://www.docker.com/) installed and running on its machine.

## Configure Snapshot Dependencies

Passing artifacts automatically is great, but might get unpredictable unless there is a way to ensure they are processed within the same context on all stages of the chain.

To create a synchronized pipeline, you need to connect builds with a _snapshot dependency_. We use the word _snapshot_ to describe a specific state of the project's sources, or basically the same commit. If you connect multiple builds with snapshot dependencies, they will all use the same sources — and this helps prevent potential conflicts.

>In special cases, you can create a chain where revision synchronization is disabled. Read more about advanced features of snapshot dependencies [here](snapshot-dependencies.md).

Let's compliment our existing artifact dependency with a snapshot dependency:
1. Open the __Dependencies__ settings tab and click __Add new snapshot dependency__.
2. Select _TodoApp_ as a build config to depend on.
3. Leave the default settings and save the dependency.

## Run Simple Chain

At this point, our first two builds are already chained together, and you can run your first chain. Note that to compose a Docker image in the second build, a [TeamCity agent](build-agent.md) needs to have [Docker](https://www.docker.com/) installed and running on its machine.

>__Important to understand__  
>Build in a chain always run together. When you run any build from a chain, whether it's the last one or medium one, TeamCity gathers all the other chained builds into a sequence, according to their dependencies.  
>As you can see on the diagram above, _TodoImage_ always runs after _TodoApp_; _Test1_ and _Test2_ start only after _TodoImage_ finishes and run in parallel to each other.

Let's run our _TodoImage_ build with the __Run__ button. Notice how TeamCity automatically runs a new _TodoApp_ build first and, after its finish, launches the following _TodoApp_ build.  
However, if you had at least one finished _TodoApp_ build before running a chain and this build has a suitable revision, TeamCity can simply reuse it as the first stage of the chain. You can disable such optimization mechanism anytime in the snapshot dependency settings.

In any case, you can go to __Build Configuration Home__ of any chained build, open the __Dependency__ tab of __Build Results__, and view the chain on the diagram:

<img src="simpleBuildChain.png" width="1217" alt="Simple build chain in TeamCity"/>

Our experimental UI offers three modes of representing dependencies: timeline, list, and chain. When you create more advanced chains, try monitoring them with each of these modes and choose the most convenient one for your tasks.

## Configure Triggers and Checkout Rules

You already know the basics of creating chains, or pipelines, in TeamCity. However, to become effective and production-ready, a chain needs to me automated further.

As we explained in the [first build guide](configure-and-run-your-first-build.md), TeamCity offers a variety of build triggers. Triggers run builds automatically if certain conditions are satisfied. The most popular type of trigger is a [VCS trigger](configuring-vcs-triggers.md), and that's the trigger we will use in this tutorial.

A VCS trigger starts a new build whenever it detects changes in the project's sources. You can define what repository and even exact files it monitor. Let's add a trigger in the _TodoImage_ settings:

1. Open the __Triggers__ page and click __Add new trigger__.
2. Enable advanced options and then turn on the option to _trigger a build on changes in snapshot dependencies_. This way, this trigger will also react on changes relevant for the _TodoApp_ config. It's often convenient to add a single trigger at the end of the chain and just enable this option to consider the previous builds. This way, if you want to change the trigger settings, you can do this in one place.
3. Leave other settings default and save the trigger.

Now, if you change the sample project's code, TeamCity will detect it and run the chain.

Every chain stage is responsible for its own task. That is, in most cases, different build configs need to monitor different parts of the source project. Our _TodoImage_ is mostly interested in changes made to `Dockerfile`, and it makes sense to restrict its scope to it. This way, when you change the logic itself, only _TodoApp_ will be triggered, and _TodoImage_ will run as its dependency build. Without such restrictions, TeamCity would start both of these builds per any change to the source repo.

You can define the scope of monitored sources in a build configuration's __Version Control Settings__:
1. Opposite our only VCS root click __Edit checkout rules__.
2. Enter the rules using [this syntax](vcs-checkout-rules.md) and save them.

For _TodoImage_, the rules will be:

```Shell
-:.
+:docker
```

First, we exclude the whole repository scope from checkout, but then we include only the `docker` directory.

For _TodoImage_, the rule is only to exclude this directory but monitor all the other files: `-:docker`.

## Complete Chain with Tests






