# linux-containers

These are my scripts for setting up LXD/LXC quickly in a sane way with name resolution and an IPv4 range that you'll remember. The thing I love about LXD/LXC is that the containers act like extremely light VMs, but share resources like an application container, so they feel and function like another (headless) computer. I've tried running containers in docker like this, but they perform poorly and require a lot of configuration changes to function at all. With these, I can spin up docker in one, podman in the next, and iotedge in another, running them all simultaneously with the same images, names, and exposed ports.

This was tested on Ubuntu 22.04.2 LTS, so YMMV on anything else, but it should be portable to any Linux distro with a suitable kernel and snap installed.

# Install
## Automated
Just execute the following, answer the prompts, and it'll do the dirty work.
```
./init
```

## Manual
If you don't care about name resolution or the IPv4 range (will be random), these are all you need to get it set up before you can launch containers:
```
sudo snap install lxd
lxd init --minimal
```

# Create system containers
## Automated
This is my base build, which will do some things that you probably want and a handful you don't. It sets up your UID and GID for mapping, mounts a home directory of your choice, renames the ubuntu user and updates with the information of your current user, and includes cloud-init that loads the environment. That environment probably isn't exactly what you want, so you'll likely want to change the set of packages, perhaps add or remove some keys, run anything unique to your environment, etc.

This one includes the following:
* A handful of fonts, for GUI interfaces.
* Microsoft Edge
* VS Code
* ~~docker & docker-compose~~ (uncomment some lines in the cloud-init.tmpl for these)

I organize my containers by [local hostname]-[name]. So on my server I might have sparrow-innotech, sparrow-dev, sparrow-test, etc. On my laptop I might have osprey-innotech, osprey-dev, etc. If you used the automated ./init above, this name with .lxd appended will resolve to the container. This makes it substantially easier to connect via SSH to them, which I'll cover below.

### Create home directory
Before running this, I recommend creating a user directory for use in each container, rather than using your home directory.
```
export DIR=eric-innotech
cp -R /etc/skel ~/$DIR
umask 077 && mkdir ~/$DIR/.ssh
cp ~/.ssh/eric.pub ~/$DIR/.ssh/authorized_keys
```
### Create container
Now run this and answer the prompts:
```
./create
```

## Manual
This gets complicated, I highly recommend reading all the documentation on https://linuxcontainers.org/

Create and start:
```
lxc launch ubuntu:lts [container name]
```

# Connecting
Assuming you used ./init, you can refer to the container by `[container name]` and append `.lxd`.

You can always login as root (in the container) by container name:
```
lxc shell [container name]
```

You can always execute commands in the container under rooti (in the container):
```
lxc exec [container name] [command(s)]
```

If you used my container creation script, included an authorized_keysi with your public, and have ssh-agent running with your private key added, you can login with ssh:
```
lxc ls
ssh [container ip address]
```

If you used ./init and ./create and have ssh-agent running with your private key added, you can login with ssh like this:
```
ssh [container name].lxd
```
## Examples
### Open a browser window specifying a profile
```
ssh -Y [container name].lxd 'microsoft-edge-stable --profile-directory="innotech"'
```
### Open VS Code
```
ssh -Y [container name].lxd code
```
### Open an SSH session with X11 forwarding in a new terminal tab
```
/usr/bin/xfce4-terminal --tab -x ssh -Y [container name].lxd 
```
