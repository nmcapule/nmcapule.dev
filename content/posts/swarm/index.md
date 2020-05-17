+++
title = "Checking out Docker Swarm"
date = "2020-05-17"
author = "Nathaniel M. Capule"
tags = ["docker", "linode", "linux"]
description = "I'm trying to learn how to setup a Docker Swarm since it is super relevant to my work right now so I am now checking out its Getting Started tutorial."
+++

I'm trying to learn how to setup a docker swarm since it is super relevant to my
work right now and I'm checking out Docker Swarm's
[Getting Started tutorial](https://docs.docker.com/engine/swarm/swarm-tutorial/create-swarm/).

> If you're also trying to learn Docker Swarm, I highly suggest that you go and
> do the tutorial. There's nothing profound in my blog entry haha unless you
> want to see real IPs and hostnames.

{{< toc >}}

## Setup

So I'm hosted on Linode. I have three instances:

- octarine (172.104.72.234) - My main devbox. I'll make this the manager
- anoia (139.162.6.209) - Nanode worker #1.
- pedestriana (172.105.188.43) - Nanode worker #2.

### Installing docker

The easiest way to install docker for all the nodes is to run the auto-install
scripts provided from the docker website:

```shell
$ # from manager node: octarine
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
$ # from worker node: anoia
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
$ # from worker node: pedestriana
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

I saw it here:
[Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/)

### Naming each nodes' hostnames

For each node, set the name to be easily identifiable by Docker.

```shell
$ # from manager node: octarine
$ hostnamectl set-hostname octarine
$ systemctl restart docker
$ # from worker node: anoia
$ hostnamectl set-hostname anoia
$ systemctl restart docker
$ # from worker node: pedestriana
$ hostnamectl set-hostname pedestriana
$ systemctl restart docker
```

## Creating the swarm

To create a swarm:

```shell
$ # from manager node: octarine
$ docker swarm init --advertise-addr
```

This will output a command for worker nodes to join the swarm. If you missed it,
you can try:

```shell
$ # from manager node: octarine
$ docker swarm join-token worker
```

To let the swarm join the worker, input the join-token command from the outputs
above. For example:

```shell
$ # from worker node: anoia
$ docker swarm join --token <snippy snip> 172.104.72.234:2377
$ # from worker node: pedestriana
$ docker swarm join --token <snippy snip> 172.104.72.234:2377
```

Now you can list out all the nodes in the swarm.

```shell
$ # from manager node: octarine
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
4uc6teexll445i8b6leelx1n1     anoia               Ready               Active                                  19.03.8
kjra2g1en8alleakcw752h0em *   octarine            Ready               Active              Leader              19.03.8
svufaptcal6gjpbeiqjwhnkce     pedestriana         Ready               Active                                  19.03.8
```

## Running a service

To run a test docker service, type in this command from the manager node:

```shell
$ # from manager node: octarine
$ docker service create --replicas 1 --name helloworld alpine ping docker.com
```

Now you can list out all running docker services via:

```shell
$ # from manager node: octarine
$ docker service ls
```

## Inspecting the service in the swarm

To check out the swarm, type the following in the manager node's terminal:

```shell
$ # from manager node: octarine
$ docker service inspect --pretty helloworld
$ # or alternately: docker service inspect helloworld
```

Where `helloworld` is the name of the running service you want to inspect.

## Scaling the service in the swarm

To scale the service, type the following in the manager node's terminal:

```shell
$ # from manager node: octarine
$ docker service scale helloworld=5
```

To inspect how the service is scaled across nodes:

```shell
$ # from manager node: octarine
$ docker service ps helloworld
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
mqzayd4nwjju        helloworld.1        alpine:latest       anoia               Running             Running 2 minutes ago
1ccqvvtu16sw        helloworld.2        alpine:latest       pedestriana         Running             Running 23 seconds ago
q1o34yvi51ox        helloworld.3        alpine:latest       anoia               Running             Running 29 seconds ago
8a6andyi2e5i        helloworld.4        alpine:latest       octarine            Running             Running 29 seconds ago
zbn2tb52cmtu        helloworld.5        alpine:latest       pedestriana         Running             Running 23 seconds ago
```

## Deleting the service

To cleanup a service, type the following in the manager node's terminal:

```shell
$ # from manager node: octarine
$ docker service rm helloworld
```

## Apply rolling updates to a service

Type the following in the manager node's terminal:

```shell
$ # from manager node: octarine
$ docker service create \
  --replicas 3 \
  --name redis \
  --update-delay 10s \
  redis:3.0.6
$ docker service inspect --pretty redis
ID:             kgdw5ln88sz194810pys89e2k
Name:           redis
Service Mode:   Replicated
 Replicas:      3
Placement:
UpdateConfig:
 Parallelism:   1
 Delay:         10s
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:         redis:3.0.6@sha256:6a692a76c2081888b589e26e6ec835743119fe453d67ecf03df7de5b73d69842
 Init:          false
Resources:
Endpoint Mode:  vip
```

Apply the rolling update:

```shell
$ # from manager node: octarine
$ docker service update --image redis:3.0.7 redis
```

When I checked the service, it looks like this:

```shell
$ # from manager node: octarine
$ docker service ps redis
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                 ERROR               PORTS
sqrwloqa7wb8        redis.1             redis:3.0.7         pedestriana         Running             Running about a minute ago
0m8f1du1jsjg         \_ redis.1         redis:3.0.6         pedestriana         Shutdown            Shutdown about a minute ago
t40h4hy96tdk        redis.2             redis:3.0.7         anoia               Running             Running about a minute ago
pgbweqyh3lnx         \_ redis.2         redis:3.0.6         anoia               Shutdown            Shutdown about a minute ago
rci3086ynmbs        redis.3             redis:3.0.7         octarine            Running             Running 49 seconds ago
yqxtf6ty2n5a         \_ redis.3         redis:3.0.6         octarine            Shutdown            Shutdown 54 seconds ago
```

## Draining a worker node

So draining apparently means the node will no longer accept new tasks, and will
try to stop any running tasks.

To drain a node, type the following in the manager node's terminal:

```shell
$ # from manager node: octarine
$ docker node update --availability drain anoia
$ docker node inspect --pretty anoia
ID:                     4uc6teexll445i8b6leelx1n1
Hostname:               anoia
Joined at:              2020-05-17 14:14:52.551119478 +0000 utc
Status:
 State:                 Ready
 Availability:          Drain
 Address:               139.162.6.209
 ...snip...
$ docker service ps redis
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE             ERROR               PORTS
sqrwloqa7wb8        redis.1             redis:3.0.7         pedestriana         Running             Running 7 minutes ago
0m8f1du1jsjg         \_ redis.1         redis:3.0.6         pedestriana         Shutdown            Shutdown 7 minutes ago
mbfxm2j8zi56        redis.2             redis:3.0.7         octarine            Running             Running 35 seconds ago
t40h4hy96tdk         \_ redis.2         redis:3.0.7         anoia               Shutdown            Shutdown 36 seconds ago
pgbweqyh3lnx         \_ redis.2         redis:3.0.6         anoia               Shutdown            Shutdown 7 minutes ago
rci3086ynmbs        redis.3             redis:3.0.7         octarine            Running             Running 6 minutes ago
yqxtf6ty2n5a         \_ redis.3         redis:3.0.6         octarine            Shutdown            Shutdown 6 minutes ago
```

Notice that all tasks under `anoia` are now set to _Shutdown_.

To return the worker to active, type the following in the manager node's
terminal:

```shell
$ # from manager node: octarine
$ docker node update --availability active anoia
```

## What's next?

There's another tutorial for
[Use swarm mode routing mesh](https://docs.docker.com/engine/swarm/ingress/),
but I doubt that it'll be applicable to my current job. I'm skipping it!

## Cleanup

```shell
$ # from manager node: octarine
$ docker service rm redis
```

### Remove a node from the swarm

So there are two ways:

- The worker node says it wants to leave the swarm

  ```shell
  $ # from worker node: anoia
  $ docker swarm leave
  ...snip...
  Node left the swarm.
  ```

  It's still in the manager's node list, but inactive.

  ```shell
  $ # from manager node: octarine
  $ docker node ls
  ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
  4uc6teexll445i8b6leelx1n1     anoia               Down                Active                                  19.03.8
  kjra2g1en8alleakcw752h0em *   octarine            Ready               Active              Leader              19.03.8
  svufaptcal6gjpbeiqjwhnkce     pedestriana         Ready               Active                                  19.03.8
  $ # might as well remove it too
  $ docker node rm anoia
  ```

- The manager node kicks a worker node from the swarm

  ```shell
  $ # from manager node: octarine
  $ docker node rm --force pedestriana
  $ docker node ls
  ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
  kjra2g1en8alleakcw752h0em *   octarine            Ready               Active              Leader              19.03.8
  ```

For manager nodes, they can just also leave the swarm.

```shell
$ # from manager node: octarine
$ docker swarm leave --force
$ docker node ls
Error response from daemon: This node is not a swarm manager. Use "docker swarm init" or "docker swarm join" to connect this node to swarm and try again.
```

That's it!

{{< figure src="docker.gif" position="center" caption="I saw this at https://medium.com/beakyn/docker-containers-for-development-environment-the-good-the-bad-and-the-ugly-4778c039e3b2 please don't arrest me :D" >}}

## FAQ

### What happens if I reboot the worker node while it's in the swarm?

So I tried this out with `anoia`:

```shell
$ # from worker node: anoia
$ sudo reboot now
```

And as expected from the manager node, it recognized it as down:

```shell
$ # from manager node: octarine
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
4uc6teexll445i8b6leelx1n1     anoia               Down                Active                                  19.03.8
kjra2g1en8alleakcw752h0em *   octarine            Ready               Active              Leader              19.03.8
svufaptcal6gjpbeiqjwhnkce     pedestriana         Ready               Active                                  19.03.8
```

But once `anoia`'s reboot has completed, the manager node says it's up again!

```shell
$ # from manager node: octarine
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
4uc6teexll445i8b6leelx1n1     anoia               Ready               Active                                  19.03.8
kjra2g1en8alleakcw752h0em *   octarine            Ready               Active              Leader              19.03.8
svufaptcal6gjpbeiqjwhnkce     pedestriana         Ready               Active                                  19.03.8
```
