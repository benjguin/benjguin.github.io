# How to share a volume with a specific docker container in Visual Studio Codespaces

> Visual Studio Codespaces provides cloud-powered development environments for any activity - whether it's a long-term project, or a short-term task like reviewing a pull request

I use it to have a development environment dedicated to a repo.  
I like it because I have a Debian environment that includes most of the tools I need including Docker.  

You'll find information in the documentation at <https://docs.microsoft.com/en-us/visualstudio/codespaces/overview/what-is-vsonline>.

One of the things I found tricky was to `docker run` with a volume that points back to my development environment.

## what is the problem?

I created a sample repo: <https://github.com/benjguin/sample200902>

I create a Visual Studio Codespace for it.  
I go to <https://online.visualstudio.com/environments>, then `Create Codespace` with the following parameters:  
- Codespace Name: sample200902
- Git Repository: https://github.com/benjguin/sample200902.git
- (keep default for the rest)

From the command line part of Visual Studio (CTRL-`), I have:

```log
codespace:~/workspace/sample200902$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
443c621d0ef5        1da2d08caca9        "/usr/local/share/do…"   44 seconds ago      Up 42 seconds                           codespaces_5d85d6
```

The container I see (`443c621d0ef5`) happens to be the container from which the command line executed.  
In other words, the development environment is not the docker host. It can control docker on the host, but the development environment runs in a container.  

So if I try to run docker with a volume pointing to my current folder, that will not work.

```log
codespace:~/workspace/sample200902$ docker run -it --rm -v $PWD:/my-vol alpine sh
/ # ls -al /my-vol
total 8
drwxr-xr-x    2 root     root          4096 Sep  2 16:33 .
drwxr-xr-x    1 root     root          4096 Sep  2 16:33 ..
```

Why? Because $PWD points to current working dir on the host, not on the dev container.

## How to workaround it?

Both development environment and container can meet in a common folder on the docker host.

By inspecting our development environment we can find one:

```log
codespace:~/workspace/sample200902$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
443c621d0ef5        1da2d08caca9        "/usr/local/share/do…"   35 minutes ago      Up 35 minutes                           codespaces_5d85d6
codespace:~/workspace/sample200902$ docker inspect 44 | grep containerTmp --before=5 --after=5
                "/root/.vsonline/.vsoshared:/home/codespace/.vsonline/.vsoshared",
                "/.vsonline/vsoagent/mount:/.vsonline/bin",
                "/var/lib/docker/vsonlinemount/workspace:/home/codespace/workspace",
                "/var/run/docker.sock:/var/run/docker.sock:ro",
                "/usr/bin/docker:/usr/bin/docker:ro",
                "/mnt/containerTmp:/tmp"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
--
                "RW": false,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/mnt/containerTmp",
                "Destination": "/tmp",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
```

basically, `/mnt/containerTmp` on the host is `/tmp` in the dev container.

So let's try this:

```log
codespace:~/workspace/sample200902$ cp some-content.txt /tmp/content-copy.txt
/ # codespace:~/workspace/sample2009docker run -it --rm -v /mnt/containerTmp/:/my-vol alpine sh
/ # ls /my-vol/*.txt
/my-vol/content-copy.txt
/ # cat /my-vol/content-copy.txt
hey, I'm a text file in the dev environment.
/ # 
```

I had to use that in a project where we needed to interact with binary files from within the container, and be able to test from within the development environment.  
This trick helped me!

:-) b
