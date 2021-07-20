[//]: # (title: Create Pipeline)
[//]: # (auxiliary-id: Create Pipeline)

In TeamCity terms, a pipeline is called a _build chain_.

Builds are jobs that perform various CI/CD tasks. When you connect them into a sequence, they form a chain. Builds inside a chain can use the same revision of the source project and pass artifacts to one another.

Build chains can be quite complex and contain dozens of builds connected in series or in parallel. They are often designed to build, test, and deploy a certain project, but you can create them for any other goal.

In this tutorial, we will learn how to create a simple chain like this:

<img src="buildChainSimple.png" width="699" alt="Simple build chain in TeamCity"/>

>__Project Demo__  
>This project is available on our [demo server](https://demo.teamcity.com/project/GuestbookAws). Sign in as a guest to explore it. When exploring, note that you can switch between the TeamCity classic and experimental UI — the toggle is in the upper right corner.

## Create Builds

Our sample chain consists of 6 builds. Let's create them one by one:




