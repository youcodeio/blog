# Build YouCode.io - Part 1 - Let's talk about containers and infrastructure

*English is not my first language, so the whole story may have some mistakes… corrections and fixes will be greatly appreciated.*

Welcome to the first episode of a serie of tutorials about how we built the new version of [YouCode](http://youcode.io). Each member of the team will present his work, his tech choices, to give you an inside look of how a bunch of students step into the developer world ;) 
First article will be about the infrastructure, and why we decided to go full containers.

### Containers & microservices
![containers](http://cdn.meme.am/instances/500x/56291573.jpg)

Containers are really hype. Everyone talks about containers, how it's great, and how it'll able developers to scale their apps. Don't get me wrong, containers are really cool, but not magical. many people see containers like lightweight Virtual Machines. it's far more powerful than that! Best way to see a container is to see them as Unix processes.

What you really need is:

- application + dependencies
- Runtime environment 

The first one represent the image of the container, and the second one things like cgroups, namespaces, env vars,etc... Do you really need sudo, ls, cd and everything packaged for an OS inside containers? The answer is no. It's in fact counterproductive! 

For example: golang official image for running golang app in Docker is pretty heavy: around 700Mb

[![](https://badge.imagelayers.io/golang:latest.svg)](https://imagelayers.io/?images=golang:latest 'Get your own badge on imagelayers.io')

Why? Here's the answer: because it's based on a full OS! A bit to much to deploy a single 10Mb binary app, especially when you want to deploy ten or more  microservices. Furthermore, many official docker images are actually vulnerable to CVE like heartbleed, Ghost, and so on(More info about containers security [here](https://docs.google.com/presentation/d/1toUKgqLyy1b-pZlDgxONLduiLmt2yaLR0GliBB7b3L0/pub?start=false&loop=false#slide=id.p)).
When you begin to ship such small containers, you begin to see Docker not as a lightweight VirtualBox, but as a Operating System builder. For those who speaks french, here's a grat talk about [Use Docker as the Operation System Builder ](https://www.youtube.com/watch?v=3bg9ij4XwW4&index=29&list=PLX43hvHJe56drtQEhvrGGpVvUhaM5hkvM) by [Quentin Adam](https://twitter.com/waxzce).

With that vision, we can now see Docker as a way to build and ship tiny apps, that we could call microservices to be even more hype. Microservices are a way to architecture your app. The same way you decompose your app into functions and classes, you'll create tiny apps that do one thing, and do it well (Hello UNIX philosophy!). They'll usually communicate with REST or RPC features.
For more information about microservices, here's a [good introduction](http://martinfowler.com/articles/microservices.html), and [another one](https://www.nginx.com/blog/introduction-to-microservices/).

### Meet CoreOS
![CoreOS](https://upload.wikimedia.org/wikipedia/commons/thumb/f/f4/Coreos-wordmark-horiz-color.png/220px-Coreos-wordmark-horiz-color.png)


Let's talk about operating systems. We need a operating sytem that:

- Is up-to-date
- Designed to run Docker
- nothing more

Meet [CoreOS](http://coreos.com)! 

> CoreOS is designed to give you compute capacity that is dynamically scaled and managed, similar to how infrastructure might be managed at large web companies like Google.

CoreOs comes with 2 awesomes features:

- Fleet
- etcd

Fleet is like systemd but in a distributed way. You can write services which will start containers on your cluster with some rules like "do not run on this machine because there's service X". Go [there](https://coreos.com/fleet/docs/latest/launching-containers-fleet.html) to have a glimpse of the power of fleet.

Etcd is like Zookeeper, Consul, or other highly-available key value store. It'll be used to declare services endpoints to avoid writing IP in config files.

A great overview of CoreOS is available over [here](https://coreos.com/using-coreos/).

## Deploy CoreOS on OVH Public Cloud
![OVH](https://www.ovh.com/fr/news/logos/with-baseline/logo-ovh-avec-150DPI.png)

We will now deploy 3 instances of our cluster on OVH. Deployment is quite easy, you just need to gain access to Horizon interface by following [this tutorial](https://www.ovh.com/fr/g1773.creer_un_acces_a_horizon). Horizon is the official interface for OpenStack. Download CoreOS image [here](https://coreos.com/os/docs/latest/booting-on-openstack.html), upload it on section Images (container-format is bare and disk-format is isqcow2). During upload, add your SSH key in Security, and change default security to authorize port communication for TODO Port. Go into Instances, clic on "Launch instance" and in Post-creation, add [this cloud-config](https://coreos.com/os/docs/latest/booting-on-openstack.html#cloud-config) file (don't forget to change etcd discovery token). 

A few minutes later, you'll have your instances running. To check, you can try to see your whole cluster with fleet client. You can install it with something like yaourt or brew. Add FLEETCTL_TUNNEL=one.of.the.ip in your .zshrc/.bashrc and run:
> $ fleetctl list-machines
>
> MACHINE        IP        METADATA
>
> 157920f7...    149.202.171.189    -
>
> c749f643...    149.202.171.19    -
>
> d8a5ff7f...    149.202.171.190    -

If you see all your machines, congrats! You just deployed your first cluster ;-)

## Deploy your first container with Fleet

So now we have our cluster, let's deploy some apps. CoreOS doesn't provide software like apt, pacman or brew, only Docker! So let's deploy our first container: traefik.

> Træfɪk is a modern HTTP reverse proxy and load balancer made to deploy microservices with ease. It supports several backends (Docker, Mesos/Marathon, Consul, Etcd, Zookeeper, BoltDB, Rest API, file…) to manage its configuration automatically and dynamically.

This awesome reverse-proxy will automaticly generate the configuration file from source like etcd and Docker. Perfect for us, because we need to automatize everything! Did I mention that there's a Tiny docker image included? 

Let's start writing our first service for fleet!
// Todo

## Deploy high-availabilty services with Fleet

// Todo

## Now what?

Now, we have our proper infrastructure, fully automized, ready to receive our microservices! Stay tuned for another blog post!

### About the author

Pierre Zemb is a engineer student in a french engineer school called ISEN Brest and he's working at OVH as a part-time internship. He considers himself as a polyglot developer, with strong interests in golang, infrastructure, APIs, microservices, containers, and other backend stuffs... You can find him on [his website](https://pierrezemb.fr).
