[//]: # (title: Deploy Build)
[//]: # (auxiliary-id: Deploy Build)

This article gives an overview of the deployment instruments offered by TeamCity.

Most often, a deployment build is a finishing stage of a pipeline, or [build chain](build-chain.md). However, if your project is rather simple and you build it with a single configuration, you can deploy its result with just the final step of this configuration — it's totally up to you.

Depending on your environment, all the deployment processes can be executed on a [build agent](build-agent.md) or in any third-party system. If you prefer the latter option, consider [detaching an agent from a build](#Detach+Build+from+Agent) to optimize costs you spend on agents

## Deployment Build Configuration

When you create or edit a build configuration, you can change its type from _Regular_ to _Composite_ or _Deployment_. We talked about composite builds in the [previous tutorial](create-pipeline.md#Complete+Chain+with+Tests). Unlike them, _deployment builds_ work almost like regular ones: you can add the same steps and features or tweak the same settings. In addition to that, a deployment config offers several enhancements for an easier and transparent workflow:
* The __Run__ button of such a build changes to __Deploy__.
* All dependency builds get an extra __Deployments__ section of __Build Results__, from where you can quickly deploy the product.
* Personal builds are disabled to prevent any accidental deployment.

See [more details](deployment-build-configuration.md) about these and other features. We suggest that you use deployment configs to deliver any medium to large software.

## Ways to Deploy Build in TeamCity

Here's how you can deploy your TeamCity build:
* __Via a command line__, using any general runner like [Command Line](command-line.md) or [PowerShell](powershell.md). This is the most straightforward approach. Just add a build step, select any such runner, and enter commands as if in a regular terminal. The benefits you get from TeamCity in this case are automation, synchronization with the previous build stages, and a convenient view of build results in the TeamCity UI.
* __Using a specific runner for your platform__. For example, if you run .NET projects, the best way to deploy them is via our [.NET runner](net.md). It supports all the relevant .NET commands such as `pack` or `publish` and offers a variety of other features.
* __Using a deployer__. TeamCity offers several build runners dedicated to deployment: [SMB Upload](smb-upload.md), [FTP Upload](ftp-upload.md), [SSH Upload](ssh-upload.md), [SSH Exec](ssh-exec.md), and [Container Deployer](container-deployer.md). They can upload build artifacts via different protocols and let you configure the upload process via the UI.

## Detach Build from Agent

If you use a third-party solution for deployment, TeamCity can offer you one more convenience.

In a best-practice setup, when you configure a build script that triggers a deployment process elsewhere, you also program it to wait for the deployment results. They will affect the build status in TeamCity and will be published in this build's results.  
A deployment procedure can be quite long, and during all this time the TeamCity build will just hang there waiting. What's more important, it will occupy its build agent, and agent time is the most valuable resource in TeamCity. Luckily, TeamCity allows detaching a build from an agent if the agent is no longer necessary for any practical means. This agent can switch to running another build. On a server with many projects and agents, such optimization can make a big difference.

Learn how to [detach builds from agents](detaching-build-from-agent.md).
