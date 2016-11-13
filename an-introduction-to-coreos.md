# An Introduction to CoreOS

If you're reading this blog, then you have a rough idea of what containers are and why you want to use them. Docker has made it easy to experiment with containers, and is slowly making it easier to deploy and manage them in production environments. There are a still a lot of gaps in what Docker offers (for free) and others have stepped up to fill them.

CoreOS is one such option, and is not just a container management system, but an entire (Linux-based) operating system designed run containers.

## Components

CoreOS consists of three crucial components that perform specific functions.

### Configuration and Service Discovery

`etcd` is a globally distributed key-value store that allows the nodes in a cluster to exchange configuration with each other and also be aware of what services are available across the cluster. Information can be ready from `etc` via a command line utility or via an http endpoint.

### Application Management and Scheduling

`fleet` is a cluster-wide init (the first process that runs all other processes) system that interacts with the `systemd` init system running on each individual node. This means you can initiate and manage individual processes on each node from a central point.

### Applications

There's no package manager in CoreOS, all applications run inside containers. These can be using Docker, or CoreOS's native container engine, rkt (rocket).

## Getting Started

As an entire operating system, getting started with CoreOS means you will need to install the OS on a couple of nodes to test it properly. If you are a Mac or Windows user you will need to use [Vagrant](https://coreos.com/os/docs/latest/booting-on-vagrant.html), or try a pre-configured cluster on a hosting provider such as [AWS](https://coreos.com/os/docs/latest/booting-on-ec2.html) or [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-coreos-cluster-on-digitalocean).

Once you have an installation of CoreOS you need to define the cluster in a configuration file that conforms to the 'cloud-config' format, it offers a lot of [configuration options](https://coreos.com/os/docs/latest/cloud-config.html) that you can send to the cluster. I am experimenting with the Vagrant images, and once installed you will find a _config.rb.sample_ file that you can rename (to _config.rb_) and change to match the number of instances you would like in the cluster, for example:

```yaml
$num_instances=3
```

You also need to un-comment and change the CoreOS update channel:

```yaml
$update_channel='stable'
```

During the build process the Vagrant script will write the default cloud-config to the _user-data_ file, so make a copy of the supplied example file with `cp user-data.sample user-data`.

Start your cluster with `vagrant up`, and when the cluster is ready, you can connect to a node with `vagrant ssh core-01 -- -A`.

## Deploy an Application

The best way to see how CoreOS might be able to help you is to get started with a more real world example. This is familiar territory with Docker images, containers and commands.

```bash
docker run --name mongo-db -d mongo
```

This command will start one instance of a MongoDB container called 'mongo-db'.

Whilst this is standard Docker practise it doesn't help you embrace the full power and flexibility of CoreOS. What if the Mongo instance crashes, or an instance restarts? This is where Fleet and it's control of `systemd` comes to the rescue.

To make use of systemd, you need to create a service representing the application you want to run, this is called a 'unit file'.

On one of the machines in your cluster, create _mongo.service_ inside _/etc/systemd/system_.

```ini
[Unit]
Description=MongoService
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill mongo-db
ExecStartPre=-/usr/bin/docker rm mongo-db
ExecStartPre=/usr/bin/docker pull mongo
ExecStart=/usr/bin/docker run --name mongo-db -d mongo

[Install]
WantedBy=multi-user.target
```

Enable and then start the service:

```bash
sudo systemctl enable /etc/systemd/system/mongo.service
sudo systemctl start mongo.service
```

And now you can use the time honoured `docker ps` to see your running containers.

This is still relevant to a single node in the cluster, to start the service on the cluster, and not worry about where exactly it runs, you need to use fleet.

```bash
fleetctl start mongo.service
```

And check the container started.

```bash
fleetctl list-units
```

[Read this document](https://coreos.com/docs/launching-containers/launching/getting-started-with-systemd/) for more advanced ideas for unit files.

### Spreading Availability

If you want to ensure that instances on your service run on individual nodes, rename the service file to _mongo@.service_, and add the following line to the bottom of the file:

```ini
...
[X-Fleet]
Conflicts=mongo@*.service
```

And now you can start multiple instances of the service, with one running on each machine:

```bash
fleetctl start mongo@1
fleetctl start mongo@2
```

And again use `fleetctl list-units` to check if your instances are running and where.

All machines in the cluster are in regular contact with the node elected as leader in the cluster. If one node fails, system units that were running on it will be marked for rescheduling, as and when a suitable replacement is available.

You can simulate this situation by logging into one node, stopping the fleet process with `sudo systemctl stop fleet`, wait a couple of minutes, and restarting it with `sudo systemctl start fleet`. You can read the fleet logs `sudo journalctl -u fleet` to get more insight into what happens.

You can now add the new service to the bottom of your cloud-config file to make sure it starts when systemd starts on a machine.

```yaml
units:
  - name: mongo.service
    command: start
```

[Fleet offers more configuration options](https://coreos.com/fleet/docs/latest/launching-containers-fleet.html) for advanced setups, such as running a unit across an entire cluster or scheduling units based upon the capacity or location of machines.

## And There's more

The components outlined above are the basic components of CoreOS, but there are a couple of others useful to container-based applications that work very well with CoreOS.

### Kubernetes

The most popular container management system created by Google runs even better on CoreOS and offers a higher (and more visual) level of container management that fleet. [Read the CoreOS installation guide](https://coreos.com/kubernetes/docs/latest/getting-started.html) for more details.

### R(oc)k(e)t

Discussing rkt could be a full article in itself but rkt is a Linux native container runtime. This means it wont work on MacOS or Windows without using a virtual machine. It's designed to fit more neatly into the Linux ecosystem, leveraging system level init systems instead of using it's own custom methods (like Docker). It uses the [appc standards](https://github.com/coreos/rkt/blob/master/Documentation/app-container.md), so in theory, most of your Docker images should work with rkt too.

## Getting to the Core

If you're experienced with Linux and the concepts of init systems, then you'll likely find CoreOS a compelling tool for using with your Docker images. The project (and team) is growing, recently opening an office in Europe thanks to new funding sources. I'd love to know how you find it.
