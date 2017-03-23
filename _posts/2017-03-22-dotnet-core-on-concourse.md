---
layout: single
title:  ".NET Core on Concourse"
date:   2017-03-22 21:18:39 +0000
categories: docker concourse "dotnet core"
author_profile: true
---

If you're not familiar with concourse you should go check it out, it's great.

Great thing about concourse is that because the builds can be run in containers, as long as there's a container that can build your project, it can be made to work. It doesn't really require anything special to get it to work, once you have it set up.

First thing you'll need is to [install concourse](https://concourse.ci/installing.html). I won't cover installing it here since Concourse already have a good documentation on installing it. If you're unsure which method to use I'd recommend docker since it is by far the simplest to setup on any platform (it's also what I'm using here).

Also whichever method you use you'll also want to install the command line tool [fly](https://concourse.ci/downloads.html) to your path. This will enable you send commands to your concourse install.

If you download the repo [from here](https://github.com/simonhdickson/HelloWorldDotNetCore) you can just skip this step but if you want to build it yourself from scratch you'll need to [install .NET Core](https://www.microsoft.com/net/core#windowscmd) locally. Again it really doesn't matter which way you install it, as long as you have the command line tools you'll be able to create a project. Then run the following command in a new folder:

```
dotnet new console -l f#
```

If you have another .NET Core project lying around feel free to use that, we just need something to build.

### Executing a build on Concourse

First thing we'll need is a build script to run, put it in `ci/build.sh`:

```
#!/bin/bash
cd HelloWorldDotNetCore

dotnet restore
dotnet build
```

Drop a file in the directory called `build.yml` with the following content:

```
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

This is all we need to execute a build on Concourse. Now we can run a test build using:

```
fly -t ci login -c http://127.0.0.1:8080
fly -t ci execute -c build.yml
```

It should pull down the docker image and run the build and show it all as if you were running it on your own machine. In reality `fly` is uploading our local directory to the server and executing it there. This is handy if you just want to check that your local copy builds but really it isn't all that useful for a CI server.


### Installing a pipeline
Now this is a real power of concourse. You can make really big and complex pipelines using Concourse but for now we'll just set up a simple one that just triggers our build from github. 

Add a file called `ci/pipeline.yml` with this content:

```
resources:
- name: HelloWorldDotNetCore
  type: git
  source:
    uri: https://github.com/simonhdickson/HelloWorldDotNetCore
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

You can see the output from each of the steps, trigger/cancel builds. However the UI is not fully fledged, most of the functionally of Concourse is only accessible using the API.

### Additional notes

![AUFS error](/assets/media/docker/docker-aufs-error.png)

If you add this line to the Docker advanced settings in Docker on Windows it should resolve the issue:

![Docker Advanced Settings](/assets/media/docker/docker-advanced-aufs.png)

```
{
"registry-mirrors": [],
"insecure-registries": [],
"storage-driver": "aufs"
}
```
