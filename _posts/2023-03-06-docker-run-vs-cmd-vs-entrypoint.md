---
layout: single
title:  ""
date:   2023-03-06 18:45:00 +0530
tags: docker cmd entrypoint run container
---

# Docker: `RUN` vs `CMD` vs `ENTRYPOINT` simplified


I’ve noticed that people get confused when asked about the difference between CMD, ENTRYPOINT, and RUN commands used for
writing a Dockerfile. This blog tried to simplify the difference between these three commands so that you can get hold of the
concepts easily.

Let’s use the analogy of Chocolate making factory to understand the concepts. First, to make the chocolate, we need machines
suited for this purpose. Thus, we need to install these machines in the factory to make the chocolates. Now let’s assume that our
factory has started manufacturing chocolates. These chocolates are now available for us consumers to consume. Now you’ve 
liked the chocolates of this factory, you have decided to ask the factory if they could make customized chocolates for you, and
hurray! they’ve agreed to do so as well.

The above analogy has shown the use of all three commands in the Docker world:

`RUN`: Installing the machines in the factory. This is the stage when the factory is not yet ready to function.

`CMD`: The factory now produces a type of chocolate that is available in the market.

`ENTRYPOINT`: The factory accepted the request to make custom chocolates. It would not have been possible if we requested the factory to make furniture for us.

We now explain these commands in technical terms:

`RUN` is used to execute a command during the image build process. It is typically used to install dependencies or to configure the container image.

`CMD` is used to specify the default command that should be executed when a container is started. This command can be
overridden at runtime by specifying a new command when the container is started.

`ENTRYPOINT` is used to specify the executable that should be run when a container is started. Unlike CMD, ENTRYPOINT is not
overridden by the command provided at runtime. Instead, additional arguments can be passed to the ENTRYPOINT command
at runtime.

Here is a sample Dockerfile showing the use of all three commands:

```dockerfile
FROM alpine
# Executed during build stage.
RUN apk add --no-cache python3
# This is the default command that will run once a container is started without any additional argument.
CMD ["-c", "print('Hello, World!')"]
# This is the command that will be run when the container is started with extra arguments.
ENTRYPOINT ["python3"]
```

In this example, the `RUN` command installs the `python3` package during the build process. The `CMD` command specifies that the
default command to run when the container is started is to print `“Hello, World!”`. Finally, the `ENTRYPOINT` command specifies that the `python3` executable should be run when the container is started.

If we run this container with docker run, we will see that the output is “Hello, World!”, as specified by the CMD command.

```bash
$ docker build . -t test
$ docker run test
Hello, World!
```
However, if we run the container with additional arguments, such as `docker run <image> -c "print('Goodbye, World!')"`, we will see that the output is `“Goodbye, World!”`, because the `ENTRYPOINT` executable is python3, and we are passing additional
arguments to it at runtime.

```bash
docker run test -c 'print("Goodbye, World!")'
Goodbye, World!
```

In conclusion, `RUN` is used to execute a command during the build process, `CMD` specifies the default command to run when the container is started, and `ENTRYPOINT` specifies the executable that should be run when the container is started


>Note: Above examples are tested with Docker version `23.0.1`.