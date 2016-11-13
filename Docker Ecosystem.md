# Docker Ecosystem

Attend any tech-related event or read any tech-related article over the past eighteen months and you will likely have heard of Docker and have a inkling of what it is and does.

In short Docker builds upon concepts of the past, but packages them better. Docker is a tool for creating 'containers' that can hold just what you need for a discrete application or technology stack. Unlike Virtual Machines, these containers share the same resources for managing interactions between the containers and the host machine. This makes Docker containers quick, light, secure and shareable.

Personally, as a technology writer and presenter I have found Docker invaluable for creating presentations and demos. I can assemble a stack of components I need, run them and then destroy them again, keeping my system clean and uncluttered with packages and data I no longer need.

## Project Orca

Many developers could see a clear use case for Docker during development and testing, but struggled to understand how best it could work in production. A flurry of 3rd party tools and services emerged to help developers deploy, configure and manage their Docker workflows from development to production. Docker, Inc (the company behind the Docker project) has been building their own 'official' toolkit through a series of takeovers and product releases.

Project Orca was announced at last years DockerCon US, details are a little vague, and is especially confusing as it shares a name with [a failed political technology platform](https://en.wikipedia.org/wiki/ORCA_(computer_system)).

In my opinion, Orca is more the strategy behind the consolidation of Dockers growing portfolio of products than an actual project or product.

In this article I will present a summary of what's available as part of Docker's official toolset, how they can help you and how the pieces fit together.

## Docker Hub

At the heart of any project using Docker is a _[Dockerfile](https://docs.docker.com/engine/reference/builder/)_. This file contains instructions for Docker on how to build an image. Let's look at a simple example:

```
FROM python:2.7
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
```

In this example, the Dockerfile pulls a particular version of an existing image, copies the current local directory into the file system of the container, sets it as the working directory and then installs Python dependencies from a text file via the `pip` command.

The [Docker Hub](https://hub.docker.com/) is the official source of pre-written Dockerfiles, providing public (for free) and private (paid) repositories for images. If you're looking for a Dockerfile to suit your needs, search the hub first, using project documentation, downloads, and stars to help guide your decision.

![Searching Docker Hub](docker_es_hub_search.png)

## Docker Engine

**Duplication of above?** The Docker Engine builds Dockerfiles and turns them into usable containers. It is the core of Docker and nothing else will run without it. There are several options for installing the Docker Engine depending on your operating system, [find more details here](https://docs.docker.com/engine/installation/).


To start a container based upon an image on the Docker Hub, pull it's image and run it. Continuing the Python example:

```bash
docker pull python
docker run -it --rm --name script-name -v "$PWD":/usr/src/appname -w /usr/src/appname python:3 python app.py
```

Thus pulls the latest python image and then starts a container that runs a Python script and exits when complete. There are some other options set and the `run` command provides many more, [read a complete guide here](https://docs.docker.com/engine/reference/commandline/run/).

When a Docker `run` command starts to become more complex it may be a better idea to create your own custom Dockerfile. To start a container based on a local Dockerfile, run the follwoing inside the directory containing the file:

```bash
docker build -t my_image .
```

This will create an image named `my_image`. Start a container based on the image by running:

```bash
docker run -name my_image_container -i -t my_image
```

This starts a container named `my_image_container` based upon your custom `my_image`.

## Kitematic

For those of you who would rather avoid the command line, [Kitematic](https://www.docker.com/products/docker-kitematic) is a great GUI tool for Mac OS, and Windows. Search for the image you need, create a container and you're good to go. Kitematic offers basic configuration options, but for more advanced settings, you may need to dive into the command line.

![Docker Kitematic](docker_es_kitematic.png)

Your containers appear on the left hand side where they can be started, stopped, restarted and most usefully, you can find container logs and direct ssh (The _exec_ button) access.

![Extra Buttons](docker_es_kitematic_buttons.png)

## Docker Machine and Swarm

The first steps towards using Docker in production are [Machine](https://www.docker.com/products/docker-machine), and Swarm, providing a simple set of tools for moving and scaling your local projects to a variety of virtualization and cloud providers.

For example, to create a Docker instance on Azure:

```bash
docker-machine create -d azure --azure-subscription-id="XXX" --azure-subscription-cert="/mycert.pem" ecodemo
```

This command creates a Ubuntu 12.04 based VM named `ecodemo` with Docker pre-installed. Each provider requires different parameters and authentication methods and default settings can be overridden. Read more details in the [documentation](https://docs.docker.com/machine/concepts/) here.

When combined with [Swarm](https://www.docker.com/products/docker-swarm), Machine can create clusters of Docker instances that can be treated as one single, large Docker instance.

Every Swarm cluster needs a master instance, this is created with the following command.

```bash
docker-machine create
  -d virtualbox
  --swarm
  --swarm-master
  --swarm-discovery token://TOKEN_ID
  swarm-master
```

This creates a Docker instance in virtualbox and sets it as a master node in a Swarm cluster. The `TOKEN_ID` is important as it helps all nodes in a luster identify each other. Aside from creating a token manually, there are [discovery systems](https://docs.docker.com/swarm/discovery/) you can use to help manage this process.

Add Docker instances to the Swarm cluster, using the same `TOKEN_ID`

```bash
docker-machine create
  -d virtualbox
  --swarm
  --swarm-discovery token://TOKEN_ID
  swarm-node-n
```

`swarm-node-n` is unique name for each node in the cluster.

Now instead of starting containers on individual VMs, you can start containers on the cluster and the master node will allocate it to the most available and capable node.

## Docker Compose

[Compose](https://www.docker.com/products/docker-compose) makes assembling applications consisting of multiple components (and thus containers) simpler as you can declare all of them in a single configuration file started with one command.

Here's an example of a Compose file (named _docker-compose.yml_) that creates three instances of the [Crate](http://crate.io) database and an instance of the PHP framework, [Laravel](https://laravel.com/) (with some extra configuration). Crucially, the containers are linked with the `links`  configuration option.

```yaml
crate1:
  image: crate
  ports:
    - "4200:4200"
    - "4300:4300"
crate2:
  image: crate
crate3:
  image: crate
  volumes:
    - ./data:/importdata
laravelcomposer:
  image: dylanlindgren/docker-laravel-composer
  volumes:
    - /laravel-application:/var/www
  command: --working-dir=/var/www install
  links:
    - crate1
laravelartisan:
  image: dylanlindgren/docker-laravel-artisan
  links:
    - crate1
  volumes_from:
    - laravelcomposer
  working_dir: /var/www
  command: serve --host=0.0.0.0:8080
  ports:
    - "8000:8000"
    - "8080:8080"
```

All these instances and their configuration can now be started by running the following command in the same directory as the _docker-compose.yml_ file:

```bash
docker-compose up
```

![Docker Compose Example](docker_es_compose.png)

You can use similar sub-commands as the `docker` command to affect all containers started with `docker-compose`. For example, `docker-compose stop` will stop all containers started with `docker-compose`.

## Docker Cloud

Automated management and orchestration of containers has been the main piece of the Docker puzzle filled by 3rd party services until Docker Inc acquired TuTum (which underpins Docker Cloud) last year. Whilst there is no integrated command line tool (yet), the Docker Cloud service accepts Docker Compose files to setup application stacks, so isn't a large diversion from the rest of the ecosystem.

For Example:

```yaml
crate1:
  image: crate
  ports:
    - "4200:4200"
    - "4300:4300"
  command: crate -Des.network.publish_host=_ethwe:ipv4_
crate2:
  image: crate
  command: crate -Des.network.publish_host=_ethwe:ipv4_
crate3:
  image: crate
  command: crate -Des.network.publish_host=_ethwe:ipv4_
```

This creates three instances of the same image, one with port allocation between the host machine and Docker manually set and automatically set on the others. I will revisit `command` soon.

If you want to scale an application beyond one node (which can run as many containers as it can manage) and one private repository, Docker Cloud is a paid service. This is enough for experimentation purposes. Bear in mind that by default Docker Cloud manages containers hosted on other 3rd party hosting services, you will need to also pay their costs. It's possible to get the Docker Cloud agent running on any Linux host you may manage, [find instructions here](https://docs.docker.com/docker-cloud/getting-started/use-byon/).

![Services Running in Docker Cloud](docker_es_running_services.png)

This screenshot shows 3 Docker containers running across two Digital Ocean instances using a pre-configured rule that allocates containers to hosts based on parameters you set. It will automatically ensure that the quantity of containers you specify are always running.

In the Docker Compose example above you might have noticed `_ethwe:ipv4_`, this is one other great feature of the Docker Cloud. Many distributed applications and services rely on '[Service Discovery](https://en.wikipedia.org/wiki/Service_discovery)' to find other instances of the same service and communicate. When spreading services across data centers and physical machines, this has often required manual declaration of the instances or another way of them finding each other. Docker Cloud includes support for [Weave](https://www.weave.works/) to create a 'soft' network across your real network, meaning all containers and applications can discover each other, no matter where they are hosted. In the example above, we override the default command issued to the container to make sure it receives the information it needs to make use of this feature.

## Data Center

So far, most of the tools covered in this article have been tools you install, host and support yourself. For enterprise users looking for a higher guarantee of security, performance and support, Docker offers [Data Center](https://www.docker.com/products/docker-datacenter).

It uses much of the same toolkit covered here but adds a private registry for your images, a private cloud, premium support and 3rd party integrations with providers likely to appeal to enterprise users. These include user management with LDAP and Active Directory, container monitoring and logging.

## Conclusion

As you will see from my screenshots and your own experiments with these tools, they still feel like a set of connected, but loosely coupled products, not a cohesive 'suite'. Returning to Project Orca, I think it aims to focus on building consistency between all these projects, making each one a logical stepping stone to the next, all from one GUI or CLI. It aims to not only answer the question on many Developer's lips, "Why should I use Docker", but also "Why would I not use Docker".
