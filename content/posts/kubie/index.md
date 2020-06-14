+++
title = "Managing a software orchestra - Startup Edition (Part 1)"
date = "2020-06-14"
tags = ["kubernetes", "rancher", "docker", "aws", "distributed"]
author = "nats"
description = "Wetting my feet on the world of container orchestration, given a limited budget and an MVP deliverable timeline."
+++

{{< toc >}}

# Where we at?

SO I'm working on improving the infrastructure of a simulation software, written
N years ago by software engineers of the old at a startup where I am currently
and very recently employed.

The simulation software itself is just a CLI tool, reading inputs from a
[MongoDB](www.mongodb.com) database[^1] and outputting it into another database.

{{< figure src="shitty_draw.png" position="center" style="width:500px" caption="Amazing, unnecessary and totally legible drawing by me" >}}

In theory, this should be a simple setup. But our internal users' use case is to
run hundreds of simulations. No prob, just call the simulation tool hundreds of
time? Nope, as usual in the world of software engineering, nothing is ever
simple; let's look at the problems we encountered:

1. **The memory problem**. We have on-premise machines that have monstrous
   **64GB** RAM sticks -- but we're encountering out-of-memory exception
   whenever we run our software! The OOME signal kills the simulation because
   it's using more than the available RAM.

   - Furthermore, the default install of MongoDB takes up around 50% of the RAM
     even when it's not doing anything! That's 32GB out of 64GB that could have
     been used elsewhere, what's up with that?
   - We minimized the OOME signals by allocating another 64GB of swap space.
     That's probably ok as a short-term solution, since the on-premise computers
     use SSD (I assume).

2. **The parallelization[^2] problem**. A simulation can take up to 10-40
   minutes to finish -- and that's with a pretty hefty machine fit for a
   mainframe.

   - I can't create more than 1 instance of the simulation in a single machine
     without encountering a looming OOME signal.
   - I can create a script that invokes 100 simulations serially though. It is
     inefficient.
   - I can create new simulation instances on other machines that can connect to
     the main MongoDB and that solves the parallelization problem. For that, we
     need proper infrastructure - not hackish bash scripts.

Given this scenario, you could deduce some assumptions: that the tool is
unoptimized memory-wise and cpu-wise. This is a problem that our org has to live
with unless we can allocate someone diving into the complex code of the
simulation and have enough sprint cycles to actually implement it.

> I can empathize with the previous developers. Given a complex domain (like the
> domain where our simulator is for), it is super super hard to give time for
> optimization of a codebase that have grown organically and now comprises of
> around a hundred thousand lines of code.

[^1]: It's weird to say MongoDB database, but that's probably the correct term?
[^2]: Yes, not concurrency. The app is too opaque for me to do concurrency :D

# Where do we want to be?

When I arrived at this company, the solution that was previously proposed is to
have an infrastructure that would run many instances of our simulation tool on
different machines or EC2 instances. Here's how they envisioned it:

{{< figure src="shitty_draw_distributed.png" position="center" style="width:800px" caption="Yes, that's an orchestra conductor stick." >}}

> Did I mention that it's written in [**Go**](http://golang.org)? It's a
> garbage-collected programming language so it would require monumental effort
> on our part to optimize it memory-wise. Our app does not exactly fit the terms
> of `microservices`, it looks more of a `macroservice` what with it consuming
> RAM like a monster :D

Pretty simple right? We use [Redis](redis.io) as our task queue and MongoDB as
result backend + input/outputs. With a single command to the _simulation
master_, it will queue a task to the task queue and then one of the workers
would pick it up and work on it. It's actually pretty ingenious when I first saw
it -- maybe because I haven't worked on a distributed app setup in my whole
software development career.

The problem now is **how do we implement it?**

# What I did

As I've said, I have absolutely zero experience in the world of modern
distributed computing. I did get an assignment in my undergraduate days that
requires parallelization between multiple machines, but that's in ye olden days.
We have [Docker](https://www.docker.com/) now and reproducible builds and
environments! It's the modern era!

{{< figure src="moby_logo.png" position="center" style="width: 300px" caption="TODO(nmcapule): marquee" >}}

We have Docker hub uploads of our app so containerization is almost a solved
problem[^3] now in our org. My task is to:

- See what are my options for container orchestration so that all components in
  our distributed architecture can talk with each other.
- Actually implement it on a reasonable timeframe.

[^3]: We can improve it. A colleague introduced me to multi-stage builds!

When I first started on this task, I've roamed the internetz for my options:

1. Amazon Elaster Container Services
2. Docker Swarm
3. Kubernetes
   - Google Kubernetes Engine
   - Amazon Elastic Kubernetes Services
   - kubeadm + kubelet
   - Rancher

## Amazon ECS

Our POC distributed infrastructure is originally created with
[Amazon Elastic Container Services](https://aws.amazon.com/ecs/). I tried it,
but there's too much UI cruft in AWS console -- which is too sad when compared
to the UI from Google Cloud Platform.

I'm terribly sorry for dismissing a solution because of its UI, but in ECS'
current state, I'd rather use a CLI tool than deal with its UI. Speaking of, AWS
ECS does have a CLI tool -- but there's too much doc that I'd rather spend time
learning another OSS orchestration tool than deal with ECS **vendor lock-in**.

## Docker Swarm

I have tried [Docker Swarm](https://docs.docker.com/engine/swarm/) before, I
even have a rough [post for it](https://nmcapule.dev/posts/swarm/).

I really like it. I could even use the same `docker-compose.yml` that our dev
environment uses into production. I only need to install the `docker` CLI. I can
easily see our org using it. But this google search changed my mind:

{{< image src="swarm_search.png" position="center" style="width: 600px" >}}

Whoops. I know that Docker Swarm entering EOL is still a conjecture with little
weight in it, but I'd rather learn to use an established solution than use a
tool with a hint of EOL. The problem is that this is a self-fulfilling cycle for
Docker Swarm; they really need to up their marketing if they want Docker Swarm
to be a serious entrant to the container orchestration world.

## Kubernetes

And here we are at last. I've heard about [Kubernetes](https://kubernetes.io/)
from Hacker News. I saw my previous colleague use it in production. I know it is
complex and probably hard to learn. But this is the only proven solution that I
know will support our app deployments for a long time to come.

Google, Amazon and Azure has their own version of it. There's a large community
around it. I know I'm safe in the large and firm lap of Kubernetes :D

So I've invested some time to learn it over a weekend. Lots of concepts to
study, although I eventually found out that I only needed a small subset of it.

I've evaluated some of the ways to implement it in our distributed architecture.
I'm sorry if I've skipped Azure and Google, it's a function of practicality and
we already have an AWS account anyway.

### Amazon Elastic Kubernetes Services

We're already in Amazon, so we might as well use their Kubernetes offering. It's
cool. I just don't like the UI as usual hehe.

### kubeadm + kubelet

Manually creating a cluster with `kubeadm` and a node with `kubelet`. It's a
hard no for me. I tried it, lots of headache and Google search.

Maybe my dev ops skills aren't up to par yet. I was debugging an issue in this
setup when I found a github issue post that said something along the lines of
[^4]:

```
I give up with kubeadm. I'm migrating to Rancher.
```

And that's how I found out about...

[^4]: I'm sorry, I can no longer find this post. But thank you whoever you are.

### Rancher

{{< figure src="rancher.png" position="center" style="width:260px" caption="It's love at first sight" >}}

Bless the developers and visionaries that created
[Rancher](https://rancher.com/). You are all freaking awesome and I freaking
love all of you :D

- I learned a lot of Kubernetes concepts and relationships using this tool.

- It's even open-source: https://github.com/rancher/rancher

- You only need one docker command to install it:

  ```sh
  sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
  ```

- And the best boon of all, that UI. That sweet sweet UI.

  {{< image src="rancher_ui.png" position="center" style="width: 600px" >}}

For fellow new dev ops folks to the container orchestration, I'm telling you to
start with this tool. Learn the basics of Kubernetes, deploy a simple cluster
and then use this tool. I mean it.

Enough honeymoon about Rancher. I'll go detail it in the next post and how I
setup our distributed architecture with it -- probably next week. I'm tired. See
ya!

> I looked around the internet, I'm seeing competitors of Rancher but I no
> longer have the energy to explore them. I did a lot of dev ops stuff this week
> and I wanna rest haha so forgive me folks.
