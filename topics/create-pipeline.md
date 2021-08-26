[//]: # (title: Create Pipeline)
[//]: # (auxiliary-id: Create Pipeline)

_This tutorial assumes that you have already installed and started your trial TeamCity instance as described [here](quick-setup-guide.md). We also suggest that you learn how to [run a simple build](configure-and-run-your-first-build.md)._

In TeamCity terms, a pipeline is called a _build chain_.

Builds perform various CI/CD jobs. When you connect them into a sequence, they form a chain. Builds inside a chain can use the same revision of the source project and pass artifacts to one another. Such chains can be quite complex and contain dozens of builds connected in series or in parallel. They are often designed to compile, test, and deploy a certain project, but you can create them for any other goal.

In this tutorial, we will explain the basics of creating pipelines in TeamCity and learn how to create a chain like this:

<img src="buildChainSimple.png" width="674" alt="Simple build chain in TeamCity"/>

While running this chain, TeamCity will (1) build a Spring Boot application and (2) create its Docker image. Then, it will (3-4) check the app with two sets of tests and (5) report the test results.

## Import Sample Project

TeamCity allows saving a project's configuration in Kotlin or XML. That's what we did with the sample project, so you can easily import it to your server. To do this:
1. Go to __Administration | Projects__ and click __Create project__.
2. In the _Repository URL_ field, enter the [sample project's](https://github.com/mkjetbrains/TodoApp-NoChain-KTS) URL and click __Proceed__.  
   >__Fork sample projects to get full control__  
   >Note that sample repositories are restricted to edit. To be able to tweak the source code during the tutorial, you need to fork both the settings repo and the [source code repo](https://github.com/mkjetbrains/todoapp-backend). Then, change the values of `namesAndTags` and `url` parameters in the `.teamcity/settings.kts` file, so they correspond to your forks.  
   >This step is completely optional — feel free to just use our sample sources for a quick walkthrough.
3. TeamCity will detect the `settings.kts` file, which corresponds to a TeamCity project's settings saved in Kotlin format. Leave the default settings and proceed.
4. TeamCity will import the sample project's settings and redirect you to its __General Settings__ page. Here, you can scroll a bit and see the _TodoBackend_ subproject. Click it to view all the created build configurations.

Now you can start chaining them.

## Configure Artifact Dependency

The _TodoApp_ build configuration compiles a `.jar` application and publishes it to the `build/libs/` directory. The _TodoImage_ configuration builds a Docker image out of this `.jar`.

To pass the `.jar` from one configuration to another, we need to create an __artifact dependency__ between them. This way, when each new _TodoApp_ build finishes and produces an artifact, TeamCity will use this artifact in the following _TodoImage_ build.

A dependency determines how one build depends on another, and thus is created in the settings of the dependent build. In our case, it's _TodoImage_. Let's go to its settings and add an artifact dependency:
1. Open the __Dependencies__ settings tab (you might need to click __Show more__ to display this item) and click __Add new artifact dependency__.
2. Select _TodoApp_ as a build configuration to depend on.
3. Choose to get artifacts from the build from the same chain, as we are about to chain these builds.
4. In _Artifacts rules_, specify that we want to import the specific artifact as `todo.jar` — enter `todo.jar => build/libs/todo.jar`.  
   You can read about patterns of artifact rules and other details related to artifact dependencies [here](artifact-dependencies.md).
   <img src="chaindemo1.png" width="539" alt="Simple build chain in TeamCity"/>

>To simplify step 4 in the future, you can use the artifact browser (![popup-artifacts-tree.png](popup-artifacts-tree.png)). When there is at least one finished dependent build that already produced some artifacts, TeamCity can show them in a tree, so you can choose them in a handy way.

At this point, you can start the first _TodoApp_ build and, after its finish, run a _TodoImage_ build. As a result of this build, TeamCity will produce a Docker image. Note that to compose a Docker image, a [TeamCity agent](build-agent.md) needs to have [Docker](https://www.docker.com/) installed and running on its machine, so make sure to install it in advance.

## Configure Snapshot Dependency

Passing artifacts automatically is great, but might get unpredictable unless there is a way to ensure they are processed within the same context on all stages of the chain.

To create a real synchronized pipeline, you need to connect builds with a _snapshot dependency_. We use the word _snapshot_ to describe a specific state of the project's sources, or basically a specific commit. If you connect multiple builds with snapshot dependencies, they are guaranteed to process the same sources.

>In special cases, you can create a chain where revision synchronization is disabled. Read more about advanced features of snapshot dependencies [here](snapshot-dependencies.md).

Let's compliment our existing artifact dependency with a snapshot dependency:
1. Open the __Dependencies__ settings tab of _TodoImage_ and click __Add new snapshot dependency__.
2. Select _TodoApp_ as a build config to depend on.
3. Leave the default settings and save the dependency.

Note that a build chain is a sequence of builds connected with snapshot dependencies. Some of the builds might also be connected with artifact dependencies, but this is not a mandatory condition.

## Run Simple Chain

At this point, the first two builds are already chained together, and you can run your first chain.

When you run any build from a chain, whether it's the last one or medium one, TeamCity gathers all the other chained builds into a sequence, according to their dependencies. As you saw on our sample chain's scheme, _TodoImage_ always runs after _TodoApp_; _Test1_ and _Test2_ start only after _TodoImage_ finishes and run in parallel to each other.

Let's run the _TodoImage_ build with the __Run__ button. Notice how TeamCity automatically runs a new _TodoApp_ build first and, after its finish, launches the following _TodoApp_ build.

>If you had at least one finished _TodoApp_ build before running a chain and this build has a suitable revision, TeamCity can simply reuse it as the first stage of the chain. You can disable this optimization mechanism anytime in the snapshot dependency settings.

To view the statuses of all chained builds, go to __Build Configuration Home__ of any of these builds and open the __Dependencies__ tab of __Build Results__:

<img src="simpleBuildChain.png" width="1217" alt="Simple build chain in TeamCity"/>

Our experimental UI offers three modes of representing dependencies: _timeline_, _list_, and _chain_. When you create more advanced chains, try monitoring them with each of these modes and choose the most convenient one for your tasks.

## Configure Trigger and Checkout Rules

You already know the basics of creating chains, or pipelines, in TeamCity. However, to become effective and production-ready, a chain needs to be automated further.

### Add VCS Trigger

As we explained in the [first build guide](configure-and-run-your-first-build.md), TeamCity offers a variety of build triggers. Triggers run builds automatically if certain conditions are satisfied. The most popular type of trigger is a [VCS trigger](configuring-vcs-triggers.md), and that's the trigger we will use in this tutorial.

A VCS trigger starts a new build whenever it detects changes in the project's sources. You can define what repository and even the exact files it will monitor. Let's add a trigger in the _TodoImage_ settings:

1. Open the __Triggers__ page and click __Add new trigger__.
2. Open advanced options and then enable the option to _trigger a build on changes in snapshot dependencies_. This way, this trigger will also react to the changes relevant to the _TodoApp_ config.  
   It's often convenient to add a single trigger at the end of the chain and enable this option to consider the previous builds. Whenever you want to change the triggering settings, you will be able to do this in one place.
3. Leave other settings default and save the trigger.

<img src="vcs-trigger-settings.png" width="778" alt="Simple build chain in TeamCity"/>

Now, if you change the sample project's code, TeamCity will detect it and run the chain.

### Restrict Checkout Scope

Every chain stage is responsible for its own task. And in most cases, different build configurations need to monitor different parts of the source project. For example, our _TodoImage_ is mostly interested in changes made to `Dockerfile`, and it makes sense to restrict its scope to it. This way, when you change the app's logic, only _TodoApp_ will be triggered, and _TodoImage_ will run as its dependency build. Without such restrictions, TeamCity would start both of these builds per any change in the source repo, which will waste resources and could create mess.

You can define the scope of monitored sources in each build configuration's __Version Control Settings__:
1. Opposite the VCS root, click __Edit checkout rules__.
2. Enter the rules using [this syntax](vcs-checkout-rules.md) and save them.  
   For _TodoImage_, the rules will be:  
  
      ```Shell
      -:.
      +:docker
      ```
   First, we exclude the whole repository scope from the checkout, and then we include only the `docker` directory.  
   <img src="chaindemo-checkout-rules.png" width="542" alt="Simple build chain in TeamCity"/>      
   For _TodoApp_, the rule is only to exclude this directory but monitor all the other files: `-:docker`.

## Complete Chain with Tests

Builds in a chain can run in parallel. Let's explore this on the example of tests.

As you can see in the project's __General Settings__, it has three other build configurations: _Test1_, _Test2_, and _TestReport_. According to our [target scheme](#), _Test1_ and _Test2_ should depend on _TodoImage_, which means you need to create a snapshot dependency on it in both of these builds. If there are at least two suitable build agents on your server, TeamCity will be able to run these builds in parallel to each other; otherwise, it will start one after another.

As you might remember, our VCS trigger in _TodoImage_ considers only preceding builds (that is _TodoApp_) and won't be able to launch tests. We can add triggers in both test builds, but TeamCity provides a more straight-forward option — creating an extra [composite build](composite-build-configuration.md), that is _TestReport_. A composite build can run without an agent and accumulate results of the preceding builds in a chain. Moreover, it will aggregate and report the results of _Test1_ and _Test2_ in one place. Just what we need.

So, to complete this tutorial:
1. Add snapshot dependencies from _TestReport_ on _Test1_ and _Test2_.
2. Add a VCS trigger in _TestReport_, similarly to [how we did it](#Configure+Trigger+and+Checkout+Rules) for _TodoImage_. After that, you can safely remove the trigger from _TodoImage_, as the new one will trigger the whole chain.

The build chain mechanism in TeamCity is very flexible and designed to satisfy the needs of every project. You will also notice that build chains are much easier to monitor than scattered builds. Detailed statuses of all chained builds are displayed in the __Dependencies__ tab of __Build Results__.

Proceed with our getting started tutorials to learn about the other type of build configuration — _deployment_.

## Takeaway

* A build chain is a sequence of builds connected with snapshot dependencies. A snapshot corresponds to a certain commit in the source code.
* Builds in a chain can pass artifacts to each other, if you configure artifact dependencies between them.
* Builds in a chain can run sequentially or in parallel. You can create chains with dozens of builds, and only the number of available build agents limits how many of them can run simultaneously.
* When any chained build is triggered, TeamCity composes and runs the whole chain from start to finish. As triggers can only consider preceding builds, it is convenient to add one VCS trigger in the very last build of a chain.
* You can limit what scopes of the source projects are relevant to each build configuration. This allows preventing excessive build runs.
* You can create a logical _composite_ configuration to gather the results of multiple dependency builds. Such a configuration doesn't require a build agent and only serves as an aggregator.



