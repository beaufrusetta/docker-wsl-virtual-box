# Run Docker on WSL with a VirtualBox Based Daemon

I'm a fan of Windows, Ubuntu, and Docker. I'm not so much a fan of Hyper-V (I have never really been able to get the settings right to have a seamless experience with any of the operating systems I've installed). I am a big fan and also incredibly comfortable with VirtualBox since I've used it for...I dunno...umpteen (?) years. I also like the WSL integration Microsoft has put in to Windows, but it obviously lacks a full out-of-the-box kernel experience, so you can't run the Docker daemon from within it. Docker Desktop for Windows essentially uses a Hyper-V backend (read: "virtual machine") and connects Docker binaries (that the installer drops in WSL/Powershell/Cmd) to the Docker Daemon through some magic that I don't care to unpack (`<austin powers voice>`***or do I?***`</austin powers voice>`).

My desire is to be able to run VirtualBox on my Windows machine (because - shout out to Microsoft - y'all are doing very well executing your comeback, and I moved all my development back to Windows), take advantage of Ubuntu on Windows with WSL, have a seamless Docker experience, and make sure all of this jives with VS Code.

Before we kick off, **I'm making some assumptions**:

1. You already are running the latest GA flavor of Windows 10 Pro.
2. You have [VirtualBox](https://www.virtualbox.org/wiki/Downloads) installed on your system.
3. You have a VM already built with [Ubuntu Server 20.04](https://ubuntu.com/download/server).
4. You are reading this and thinking, "man, this guy seems pretty freakin awesome".

All good? ***Even on #4?*** Sweet. Carry on. :metal:

## Install Docker on Ubuntu Server 20.04 vm

Pretty straight-forward here. You could have installed Docker when you built your VM image, but for whatever reason, I don't trust those installers...and thus always just do my own installation of Docker. The snippet below will install Docker, enable the Docker service within systemd, and add the logged in user account (**you**) to the `docker` user group (this is helpful so you aren't running `sudo` all the time).

```
$ sudo apt update && sudo apt install -y docker.io
$ sudo systemctl enable --now docker
$ sudo usermod -aG docker $USER
```

In order to enable `sudo`-less command execution, sign out of the system and sign back in, then run the following:

```
$ docker version
```

## Install Docker on WSL (Ubuntu)

Some ***additional*** assumptions that I'm making:
1. You have WSL enabled on your Windows machine.
2. You have the latest and greatest flavor of Ubuntu running in WSL. (at the time of writing, it was 20.04)

```
$ sudo apt update && sudo apt install -y docker.io
```

The Docker daemon isn't going to startup and run in WSL at this time. The hope is that it will run in WSL2 with a full kernel...but...this tutorial isn't for that. What you want is the Docker client binaries in order to connect to the remote host.

## Helpful Links
* https://success.docker.com/article/how-do-i-enable-the-remote-api-for-dockerd
* https://docs.docker.com/engine/security/
