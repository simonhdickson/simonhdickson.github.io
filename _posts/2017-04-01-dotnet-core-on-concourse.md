---
layout: single
title:  ".NET Core on Concourse"
date:   2017-04-01 08:18:39 +0000
categories: docker concourse "dotnet core"
author_profile: true
---

If you're not aware of Concourse, the main idea behind it is that your build server should be completely separate from your actual builds. If you delete your Concourse server it should be easy to just re-create your builds again. In Concourse building is all done using containers, so you don't have to worry about configuring workers.

## Setup

First thing you'll need is to [install concourse](https://concourse.ci/installing.html). I won't cover installing it here since Concourse already have documentation on installing it. If you're unsure which method to use I'd recommend docker since it is by far the simplest to setup on any platform (it's also what I'm using here). Whichever method you use you'll also want to install the command line tool [fly](https://concourse.ci/downloads.html) to your path. This will enable you send commands to your concourse install.

If you have another .NET Core project lying around feel free to use that or download the repo [from here](https://github.com/simonhdickson/HelloWorldDotNetCore).

If you want to build it yourself from scratch you'll need to [install .NET Core](https://www.microsoft.com/net/core#windowscmd) locally. Then run the following command in a new folder:

```bash
dotnet new console -l f#
```

## Executing a build on Concourse

First thing we'll need is a build script to run, put it in `ci/build.sh`:

```bash
#!/bin/bash
cd HelloWorldDotNetCore

dotnet restore
dotnet build
```

Drop a file in the directory called `build.yml` with the following content:

```yml
platform: linux

image_resource:
  type: docker-image
  source:
    repository: microsoft/dotnet

inputs:
- name: HelloWorldDotNetCore

run:
  path: sh
  args: ["HelloWorldDotNetCore/ci/build.sh"]
```

To give a quick run down of what this all means:

```yml
image_resource:
  type: docker-image
  source:
    repository: microsoft/dotnet
```

Our build is going to run on the docker container microsoft/dotnet, this will be pulled from DockerHub if the worker doesn't already have it installed.

```yml
inputs:
- name: HelloWorldDotNetCore
```

Out task depends on an input called `HelloWorldDotNetCore`. In this case this particular case this is a git resource, but it could be almost anything (we could even take the docker image to use as a parameter if we chose too).

```yml
run:
  path: sh
  args: ["HelloWorldDotNetCore/ci/build.sh"]
```

We tell it to run the `build.sh` file we created earlier. One important thing to notice is that our current directory is not inside our resource. This is because we could have multiple inputs and also Concourse has no way of knowing that `HelloWorldDotNetCore` is suppose to be a folder, it could instead have just been a file.

This is all we need to execute a build on Concourse. We can run a test build using:

```
fly -t ci execute -c build.yml
```

Whoops, first we need to login:

```
fly -t ci login -c http://127.0.0.1:8080
```

By default the login is `concourse` and `changeme`, unless of course you changed it. Now re-run the execute command.

It should pull down the docker image and run the build and show it all as if you were running it on your own machine. What is really happening is that `fly` is uploading your local directory to the concourse and executing the build inside a container. This is really quite useful since we can perform test builds on our CI server without needing to check the changes in first (no more `Fix build` commits... at least in theory).

However this isn't really going to work as a CI solution, we need to configure the build to run automatically. We can do this by configuring a pipeline. 

## Installing a pipeline

You can create quite complex pipelines in Concourse if you want, if you look at their documentation you'll find examples of what you can do. For now we'll just set up a simple one that just triggers our build from github (or wherever your project is hosted). 

Add a file called `ci/pipeline.yml` with this content:

```yml
resources:
- name: HelloWorldDotNetCore
  type: git
  source:
    uri: {{git-url}}
    branch: master

jobs:
- name: hello-world-app
  plan:
  - get: HelloWorldDotNetCore
    trigger: true
  - task: tests
    file: HelloWorldDotNetCore/build.yml
```

Now we have to tell Concourse about our pipeline so it knows what we want it to do:

```
fly -t ci set-pipeline -c ci\pipeline.yml -p HelloWorldApp
```

The pipeline starts disabled but following the console output to enable it. When you navigate to the pipeline in your browser you should see something like this:

![Concourse Dashboard](/assets/media/concourse/dashboard.png)

You can see the output from each of the steps, trigger/cancel builds. You'll notice that the UI is fairly limited in terms of what you can do. The idea behind this is to force you to put your pipeline and configuration into code rather than customizing the Concourse CI server itself.

### Additional notes

While running this build on my Windows machine I encountered a strange error while trying to execute the build:

![AUFS error](/assets/media/docker/docker-aufs-error.png)

After googling didn't really find much, I found an option to use the aufs storage driver in the docker documentation. I didn't think it would actually do anything, but surprisingly it did fix it. If you add this line to the Docker advanced settings in Docker on Windows it should resolve the issue:

![Docker Advanced Settings](/assets/media/docker/docker-advanced-aufs.png)

```
{
"registry-mirrors": [],
"insecure-registries": [],
"storage-driver": "aufs"
}
```

Now I don't really understand why this fixes the issue, as far as my understanding goes this setting really should make no difference. If someone does know why I'd love to hear it.
