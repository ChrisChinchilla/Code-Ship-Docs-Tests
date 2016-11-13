# Docker for Mac

Recently out of private beta, Docker's new native applications aim to replace the current ways for running Docker on Windows and Mac, and creating a better experience for developers using those platforms.

The previous solution, Docker Toolbox used Virtualbox to create a small Linux virtual machine that hosted your images and containers. It worked pretty well, but could be unreliable at times and required workarounds that sometimes resulted in unexpected outcomes or not working at all.

Docker for Mac removes the dependency on VirtualBox and instead uses virtualization technology that is already part of Mac OS X, [HyperVisor](https://developer.apple.com/library/mac/documentation/DriversKernelHardware/Reference/Hypervisor/).

Docker for Windows uses Microsoft's virtualization technology, [Hyper-V](https://www.microsoft.com/en-us/server-cloud/solutions/virtualization.aspx).

These changes aim to make your Docker containers run faster than before, take up less disk space, and fit better into your Operating System.

I am a Mac user, so will focus on the Mac version of Docker's new application, but will highlight any significant differences with the Windows version.

## Install and Setup

Download the native application [for your platform here](http://www.docker.com/products/docker).

Successive updates to the application have made the installation process and the resulting application increasingly 'more native' and better integrated with the operating system.

As the application uses newer technologies only available in newer machines and OS versions, it has minimum requirements, which are:

### Mac minimum requirements

- A 2010 or newer model, with Intel's hardware support for memory management unit (MMU) virtualization.
- OS X 10.10.3 Yosemite or newer, as the Hypervisor framework used is available in Yosemite onwards.
- At least 4GB of RAM
- VirtualBox prior to version 4.3.30 must not be installed as it will cause issues with Docker for Mac

### Windows minimum requirements

- 64bit Windows 10

## Co-Existing with Docker Toolbox

If you are using Docker Toolbox, your images and containers can typically coexist together. This is thanks to Docker Toolbox using Virtualbox to host images and containers, and installing command line tools to more 'Linux' path locations. Docker for Mac (and Windows) are fully native to the host platform and install everything into locations you would expect (i.e. The _Applications_ folder), only using symlinks to make certain tools accessible on the command line.

![Cracking Open the Docker Application](docker_mac_package.png)

When you first run the Docker application it will check your system for compatibility and requirements, show a welcome screen and then start the Docker process. Your main interaction with the Docker application will be via a menu bar item, to stop and start the Docker process, open Kitematic for GUI access to your containers, find documentation and access preferences.

## preferences and configuration

![Preferences](docker_mac_prefs.png)

### General

The _General_ pane has settings for configuring the specs of the virtual machine, updates and excluding the virtual machine from backups (mac only), which I think is a simple but useful feature to have, as it can end up being a large file.

![Docker Image File](docker_mac_vm_file.png)

### Advanced

Many enterprise users of Docker use their own registries for custom images. The advanced settings pane lets you define custom registries to search for images that you trust.

The application should automatically detect any HTTP(s) proxy settings you have at an operating system level, but you can check them here. Whilst not a part of this preference pane, it will also automatically detect any VPN settings you have, allowing access to any containers running within it.

### File Sharing

Whilst sharing volumes between Docker containers and the host operating system was possible with Docker Toolbox, it could be slow and suffer permissions issues. Docker for Mac uses a new file system created by Docker called 'osxfs', I can't find much detail on the new file system, but there are some [here](https://docs.docker.com/docker-for-mac/osxfs/).

You can add or remove share local paths to share with containers using the _+_ and _-_ buttons, but these paths shouldn't overlap, i.e. Not _Users_ and _Users/homefolder_.

## Using Docker Natively

Little of the process for using Docker has changed, except that it requires less steps.

For example, with the application running you can use Kitematic or the command line to download and start images as containers. Here's the 'hello world' image running in Kitematic:

![New Localhost](docker_mac_localhost.png)

Notice something else cool there? No more custom IP addresses to remember! All your Docker containers now run on `localhost` and will be port mapped to the address.

![Application Running](docker_mac_browser.png)

Other Docker commands such as `docker-compose` and `docker-machine` work, but for Machine (and thus Swarm) you will need to define a [non-native driver](https://docs.docker.com/machine/drivers/). This means you can manage Docker Machine from your Mac, but the machines will still be hosted elsewhere, and still need to be managed by the traditional `eval $(docker-machine env default)` commands.

## The Future?

I personally use Docker for rapid testing and prototyping ideas, and so rarely take my containers of my Mac. Of course, few people will use Docker with Mac or Windows in production, so many might ask if there is much point in the Docker team making native applications for these platforms. Still, a lot of developers will be using these platforms during development and I for one thank the Docker team for making my experience feel much more friendly and accessible. The applications are still considered betas, and the team welcomes any feedback you have to help them improve, so if you also appreciate the effort made, [then help them out](https://forums.docker.com/c/docker-for-mac).
