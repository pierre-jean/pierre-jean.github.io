---
layout: post
title: "A deeper look into Dockerfiles"
date: 2014-05-19 17:18:54 +0200
comments: true
categories: Docker
---

I introduced in [a previous post][first-look-dockerfiles] how to create [Docker][docker-site] images interactively and with a *Dockerfile*.

In this post, I will focus on good practices and see how a proper repository is realized. Indeed, in my previous post, my example was a little bit trivial, and if you want to create your own images through a Dockerfile, you will surely bump into difficulties: how do I manage interactive installation that ask a user input during install? How should I configure my application after installation? And many others...

<!-- More -->

Trusted Builds: a good way to learn
-----------------------------------

When you commit your image in a local repository, or push it into a remote repository, you only push the built image, as a file.
Trusted build is a mechanism to build automatically an image from its sources: the docker index will built the image each time a commit is done on the public github repository corresponding to the docker image.
This is a great way to study popular images and see how their maintainers manage difficulties you can have with the settings of some images.

You can browse and search into the official Docker index of repositories from the [website][docker-index], or interact with it in command line with `docker search`, `docker pull` and `docker push`.
Let's have a look to the *mysql* image of the repository *tutum*, available [here][tutum-mysql-docker]. As it is a trusted build, you have access to the [github page][tutum-mysql-github] from where the image is built.

Dockerfile
----------

Let's have a look the docker file, and the good practices it includes.

{% codeblock %}
FROM ubuntu:trusty
MAINTAINER Fernando Mayo <fernando@tutum.co>

# Install packages
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install supervisor mysql-server pwgen

# Add image configuration and scripts
ADD start.sh /start.sh
ADD run.sh /run.sh
ADD supervisord-mysqld.conf /etc/supervisor/conf.d/supervisord-mysqld.conf
ADD my.cnf /etc/mysql/conf.d/my.cnf
ADD mysqld_charset.cnf /etc/mysql/conf.d/mysqld_charset.cnf
ADD create_mysql_admin_user.sh /create_mysql_admin_user.sh
ADD import_sql.sh /import_sql.sh
RUN chmod 755 /\*.sh

EXPOSE 3306
CMD ["/run.sh"]
{% endcodeblock %}

### FROM command

The FROM command defines the base image for this image.

{% codeblock %}
FROM ubuntu:trusty
{% endcodeblock %}

You can see that the maintainer used a tag to define precisely which version of Ubuntu to use. **You should always define a tagged version of your base image** to precisely define which release of the distribution your image relies on.

### MAINTAINER

The maintainer is vital tag, to define the author of the image and a way to contact it.

{% codeblock %}
MAINTAINER Fernando Mayo <fernando@tutum.co>
{% endcodeblock %}

### COMMENTS

You can add comments with the character `#`. You should always add comments to explain the goal of each block of instructions.

### RUN command

After each `RUN` instruction, the image is committed, and the following RUN instruction is executed on the newly committed image.

{% codeblock %}
# Install packages
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install supervisor mysql-server pwgen
{% endcodeblock %}

We can notice several practices here:

 - No usage of `apt-get upgrade`: Indeed, you just want to add on this layer what is needed for the container. If you want to upgrade the system, you should **upgrade the base image**, as it is its role to offer the system environment.

 - Avoid interaction: the `RUN` command is **executed in a non-interactive way**. As a consequence, you don't want to been ask for confirmation when installing package: you must use the `-y` option in `apt-get install -y <packages>`. Moreover some packages ask questions during installation about account creation, default configuration, etc. It is the case for mysql-server for instance. That's why the author put the variable `DEBIAN_FRONTEND` to `noninteractive`, in order to inform that there won't be any interaction during installation.

 - Finally, there is the `apt-get update`. It is the most litigious command. If you install your packages via `apt-get`, you have no choice but to update the index of packages before the install: you don't want to have an outdated cache and broken links during installation. Nevertheless, it also means that the Dockerfile can create slightly different images depending of the date of build. Indeed, the version of a package or a dependency of a package you want to install can have change in the repository. The only work around is to install your packages via a direct link to a binary package or from source. Anyway, the consequence can be neglected as you don't want to rebuild your image every day, and you should build it once and use it / share it as long as you want to use the same environment.

### ADD command

{% codeblock %}
# Add image configuration and scripts
ADD start.sh /start.sh
ADD run.sh /run.sh
ADD supervisord-mysqld.conf /etc/supervisor/conf.d/supervisord-mysqld.conf
ADD my.cnf /etc/mysql/conf.d/my.cnf
ADD mysqld_charset.cnf /etc/mysql/conf.d/mysqld_charset.cnf
ADD create_mysql_admin_user.sh /create_mysql_admin_user.sh
ADD import_sql.sh /import_sql.sh
{% endcodeblock %}

The maintainer has added two types of external files here:

 -  **Configuration files**: you want your image to be operational immediately after the build. You have to provide working default configurations that allow to use the package in rational conditions. You must of course allow the user to override the configuration settings (by adding its own configuration files, or with environment variables).

 - **Scripts files**: As you want to automate all setup steps, it is a good idea to wrap your launcher inside scripts that could executes some  checking and actions for you (*Database creation*, *Account creation*, etc.).

### EXPOSE command

The Dockerfile defines the ports you want to expose to the host system to access the service you will run on the container, with the instruction `EXPOSE`.

{% codeblock %}
EXPOSE 3306
{% endcodeblock %}

Even if you can tell on which host port you want to map the local port, this is good practice to let Docker framework dynamically map the port to the host. Indeed, if you map yourself the port, you won't be able to launch several containers from the same Dockerfile, as the first one will lock the port for itself.

### CMD command

Finally, you can tell Docker which process it should execute by default when launching a container.

{% codeblock %}
CMD ["/run.sh"]
{% endcodeblock %}

Here, as the maintainer wrapped the process into a script to automate account creation, it launches the script instead of the process.

Supervisor
----------

If you study the scripts, you may have noticed that the maintainer doesn't launch the mysql server directly, but launch instead a process called **Supervisord**. [Supervisor][supervisord-site] is a process control system, a little bit like `init`, that allow you to manage the execution of several processes.

Indeed, I told you in my previous article that a Docker container can only run one job: there isn't any `init` running instance to manage the lifecycle of several process executions in a Docker container. Nevertheless, you will certainly want to be able to manage several processes or job in a same container: for example, running a SSH server and at the same time another kind of server. You can use Supervisor to do that.

Better, as explained in this [article][supervisor-docker], you can use inheritance to include the Supervisor configuration files from you base image, to launch the services the base image already provide in parallel with the jobs you define in your own configuration.

README.md
---------

Finally, the author of the image included a README.md explaining how to build, launch and configure the container. It is really handy and you always should include it if you create your own trusted build. The README is displayed on the[ Docker index website][tutum-mysql-docker] when you look for the image.

What's next?
------------

You have seen how is built a popular docker image. If you want to create your own Docker image, you should search for similar images in the Docker index and analyze their Dockerfiles. All trusted build are available through their github page, so it's a really easy task.

I hope this post will help you to create great images for yourself and the community!

[first-look-dockerfiles]: http://pierre-jean.baraud.fr/blog/2014/05/14/fist-look-dockerfile/
[docker-site]: https://www.docker.io/
[docker-index]: https://index.docker.io/
[tutum-mysql-docker]: https://index.docker.io/u/tutum/mysql/
[tutum-mysql-github]: https://github.com/tutumcloud/tutum-docker-mysql
[supervisord-site]: http://supervisord.org/
[supervisor-docker]: http://blog.trifork.com/2014/03/11/using-supervisor-with-docker-to-manage-processes-supporting-image-inheritance/

