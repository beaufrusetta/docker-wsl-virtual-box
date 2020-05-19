# Run Docker on WSL with a VirtualBox'd Daemon (w/MTLS)

I'm a fan of Windows, Ubuntu, and Docker. I'm not so much a fan of Hyper-V (I have never really been able to get the settings right to have a seamless experience with any of the operating systems I've installed). I am a big fan and also incredibly comfortable with VirtualBox since I've used it for...I dunno...umpteen (?) years. I also like the WSL integration Microsoft has put in to Windows, but it obviously lacks a full out-of-the-box kernel experience, so you can't run the Docker daemon from within it. Docker Desktop for Windows essentially uses a Hyper-V backend (read: "virtual machine") and connects Docker binaries (that the installer drops in WSL/Powershell/Cmd) to the Docker Daemon through some magic that I don't care to unpack (`<austin powers voice>`***or do I?***`</austin powers voice>`).

My desire is to be able to run VirtualBox on my Windows machine (because - shout out to Microsoft - y'all are doing very well executing your comeback, and I moved all my development back to Windows), take advantage of Ubuntu on Windows with WSL, have a seamless Docker experience, and make sure all of this jives with VS Code.

Before we kick off, **I'm making some assumptions**:

1. You already are running the latest GA flavor of Windows 10 Pro.
2. You have [VirtualBox](https://www.virtualbox.org/wiki/Downloads) installed on your system.
3. You have a VM already built with [Ubuntu Server 20.04](https://ubuntu.com/download/server).
4. You are reading this and thinking, "man, this guy seems pretty freakin awesome".

All good? ***Even on #4?*** Sweet. Carry on. :ok_hand:

***Please make note*** of my use of `you@server` and `you@client` in the code snippets below to distinguish between where the commands should be run.

## Step 1: Install Docker on Ubuntu Server 20.04 vm

Pretty straight-forward here. You could have installed Docker when you built your VM image, but for whatever reason, I don't trust those installers...and thus always just do my own installation of Docker. The snippet below will install Docker, enable the Docker service within systemd, and add the logged in user account (**you**) to the `docker` user group (this is helpful so you aren't running `sudo` all the time).

```
you@server:~$ sudo apt update && sudo apt install -y docker.io
you@server:~$ sudo systemctl enable --now docker
you@server:~$ sudo usermod -aG docker $USER
```

In order to enable `sudo`-less command execution, sign out of the system and sign back in, then run the following:

```
you@server:~$ docker version
```

This command should display both client/server version information without utilizing `sudo`. 

## Step 2: Install Docker on WSL (Ubuntu)

Some ***additional*** assumptions that I'm making:
1. You have WSL enabled on your Windows machine.
2. You have the latest and greatest flavor of Ubuntu running in WSL. (at the time of writing, it was 20.04)

```
you@client:~$ sudo apt update && sudo apt install -y docker.io
```

The Docker daemon isn't going to startup and run in WSL at this time. The hope is that it will run in WSL2 with a full kernel...but...this tutorial isn't for that. What you want is the Docker client binaries in order to connect to the remote host.

## Step 3: Configure Docker Daemon to Listen for Remote Requests

From within the VirtualBox instance (Ubuntu Server) running the Docker Daemon - do the following to enable the remote API for dockerd:

1. Create a file at `/etc/systemd/system/docker.service.d/startup_options.conf` with the following content:

    ```
    # /etc/systemd/system/docker.service.d/startup_options.conf
    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd -h fd:// -H tcp://0.0.0.0:2376
    ```

2. Reload the configuration:

    ```
    you@server:~$ sudo systemctl daemon-reload
    ```

3. Restart the Docker daemon so it loads the new configuration options:

    ```
    you@server:~$ sudo systemctl restart docker.service
    ```

At this point, you ***could*** utilize this Docker daemon as is from your client application, but it's insecure and this is **NOT** recommended. In the next step, you will configure both the client and the server to utilize MTLS to establish a connection to the daemon.

## Step 4: Secure Route to Docker Daemon with MTLS

(placeholder)

## Step 5: Set DOCKER_HOST on WSL
Easiest way to persist this is to put the `DOCKER_HOST` global variable into your `~/.bash_profile` script. If you don't have one, create one (`touch ~/.bash_profile`), then: 

```
you@client:~$ touch ~/.bash_profile && echo $DOCKER_HOST=192.168.56.101:2376 >> ~/.bash_profile
```

## Helpful Links

I stitched together content for this "how-to" utilizing information from these pages I found on the inter-webs:

* https://success.docker.com/article/how-do-i-enable-the-remote-api-for-dockerd
* https://docs.docker.com/engine/security/
* https://docs.docker.com/engine/security/https/
