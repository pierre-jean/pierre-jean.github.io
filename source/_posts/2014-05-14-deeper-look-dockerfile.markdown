---
layout: post
title: "A first look into Dockerfiles"
date: 2014-05-14 14:45:54 +0200
comments: true
categories: docker
---

A [week ago][docker-yabage-post], I introduced the framework [Docker][docker-site]. Docker is a lightview virtualized environment. It allows to build, managed and run containers to deploy easily an app in an iso environment.
I will introduce today how to create containers interactively and through Dockerfile.

<!-- More -->

Hands on containers and images
------------------------------

If you're not familiar with Docker, I strongly recommand you to have a look to their [interactive tutorial][docker-interactive-tutorial]. It is well done, efficient, and get you into the swing of things. The [Dockerfile tutorial][docker-dockerfile-tutorial] will also give you all the basis. I will try here to sum up the more important concepts.

### What is a container?

A container can be represented by two main components:

 - A running job
 - A filesystem modified by the job

The filesystem itself is a multilayered union filesystem, with the top layer saving the current modifications, and the underlying layers read only images. Let's have a deeper look on what is a multilayered union filesystem.

### Images and AUFS

Let's build our own customized image! First of all, let's pull an image from the docker index.

{% codeblock lang:console %}
~$ sudo docker pull ubuntu:precise
{% endcodeblock %}

This will download an image with the files of Ubuntu Precise distribution (without the Kernel, as it uses the host kernel).

{% img /images/docker/docker-image-creation-00.png %}

We will then interact with it by launching a job from this filesystem.

{% codeblock lang:console %}
~$ sudo docker run ubuntu apt-get install -y memcached
[...]
[=> container id is 9fb69e798e67]
{% endcodeblock %}

When executing this command, we create a container. This container will create a writable layer for its filesystem, and base it upon the image *ubuntu*. It will then launch the process `apt-get` with the argument `install -y memcached`

{% img /images/docker/docker-image-creation-01.png %}

I didn't affect any name to the container, but the output gave me its id. If I want to save the current state of the filesystem, I can commit it thanks to the id of the container:

{% codeblock lang:console %}
~$ sudo docker commit 9fb69e798e67
{% endcodeblock %}

{% img /images/docker/docker-image-creation-02.png %}

And the container will automatically create another layer upon your newly created image. Note that you are only saving **the filesystem state**. The **memory state won't be saved** in the image.

{% img /images/docker/docker-image-creation-03.png %}

Note also that the status of your container is depending of the status of the job your run. Once the job finished, the container is stopped.

If you want to organize your images, it is better to commit them under your local repository.

{% codeblock lang:console %}
~$ sudo docker commit 9fb69e798e67 yabage/ubuntu-memcached
{% endcodeblock %}

Finally, you can of course run a new container based on your newly created image, and commit other modification upon it.

{% codeblock lang:console %}
~$ sudo docker run -i -t yabage/ubuntu-memcached /bin/bash
{% endcodeblock %}

{% img /images/docker/docker-image-creation-04.png %}

The Dockerfile
--------------

You have seen how to build an image by interacting direclty with the container. Nevertheless, most of the time you will want to share your images through "recipes" allowing others to **build** themselves your images.
This is what **Dockerfiles** are for.

The Dockerfile lists the instruction on how to build your image and what to run on the container.

As you want to obtain the same image in every circonstances, you will have to **avoid** any operation that **doesn't result in a controlled and guaranted state**. For instance, you should not do any `apt-get upgrade` in a Dockerfile, as you don't controlled the result : indeed, depending the date your launching it, the upgrade could be different. Moreover, for technical reasons, there is a high chance that the upgrade fails. If you want to upgrade your distribution, you should update the base image your Dockerfile rely on.

Now let's see how to build the first container in a Dockerfile:

{% codeblock %}

FROM ubuntu:precise #the base image of this build script

RUN apt-get install -y memcached

{% endcodeblock %}

Well...that's all. You know have a container with memcached installed.

I kept this Dockerfile minimalistic on purpose, put you will want to define the maintainer of the Dockerfile with the tag `MAINTAINER`, define the default process to launch with `CMD` and a lot of other usefull instructions.
For now I will stick to this version.

I would like to focus your attention on a point: each RUN instruction will commit the current layer and create a new one upon it. **The state of the memory is forgotten** between two `RUN` instructions. Only the filesystem is kept.

To build an image, you put these instructions in a file called *Dockerfile*, then executes from the same folder:

{% codeblock lang:console %}
sudo docker build .
{% endcodeblock %}

It will build the container and return its id. If you want to name the container when building, you can do it with the `-t` option

{% codeblock lang:console %}
sudo docker build -t ubuntu-memcached .
{% endcodeblock %}

You have seen a very quick introduction to Dockerfile. I will soon write an article about how to write a proper Docker image, following good practices, so that you can share them with your friends and the community!

[docker-yabage-post]: http://pierre-jean.baraud.fr/blog/2014/05/07/docker/
[docker-site]: http://docker.io
[docker-interactive-tutorial]: https://www.docker.io/gettingstarted/#
[docker-dockerfile-tutorial]: https://www.docker.io/learn/dockerfile/