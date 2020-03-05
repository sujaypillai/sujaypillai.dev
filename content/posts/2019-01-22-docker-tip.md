---
layout: post
title: Connect to Docker daemon over ssh using docker-compose
date: 2019-01-22T16:22:42+08:00
lastmod: 2019-01-22T16:22:42+08:00
draft: false
tags: ['docker', 'dockertip','docker-compose']
categories: ['Docker']
---

One of the cool feature released with Docker 18.09 is the ability to connect to a docker daemon over ssh. This post would walk you through the steps to connect to a docker daemon running on [DigitalOcean](https://www.digitalocean.com/).

To create a remote docker host we will use a tool called docker-machine : 

{{< highlight console >}}
docker-machine create --driver DigitalOcean --digitalocean-access-token xxxx do-node2
{{< /highlight >}}

![digitalocean-nodes](/images/dm1.png)

Here is a detailed [documentation](https://docs.docker.com/machine/examples/ocean/) about setting up a droplet on DigitalOcean. You may verify this by logging into your DigitalOcean account as shown below

![digitalocean-nodes](/images/do1.png)

When the Droplet is created, Docker generates a unique SSH key and stores it on your local system in `~/.docker/machine/machines` and for this droplet named do-node2 its generated under `~/.docker/machine/machines/do-node2` in my case. This location would contain a file called `id_rsa` which is used under the hood to access the droplet directly with docker-machine ssh command as shown below

![digitalocean-nodes](/images/do2.png)

To connect to the above remote docker daemon we would add the same private key to the authentication agent using the below command - 

{{< highlight console >}}
ssh-add -k ~/.docker/machine/machines/do-node2/id_rsa
{{< /highlight >}}

Now issue the command :

{{< highlight console >}}
docker -H ssh://root@165.227.106.59 run -d -P nginx
{{< /highlight >}}

![digitalocean-nodes](/images/do3.png)

On executing the above command your docker client will connect to the DigitalOcean droplet to find that in order to run the nginx container it needs to pull it from DockerHub and then starts the container on the port `32768` (with docker doing the port mapping for you because of flag -P). Below is what you get when you try to access the nginx container using the droplet ip in a browser:

![digitalocean-nodes](/images/do4.png)

If you don't want to use the above flag every time with the docker command you may set the environment variable called DOCKER_HOST as shown below -

{{< highlight console >}}
$ export DOCKER_HOST=ssh://root@165.227.106.59

$ docker image ls
REPOSITORY  TAG       IMAGE ID            CREATED             SIZE
nginx       latest    7042885a156a        3 weeks ago         109MB

$ docker ps --format "{{.ID}} :: {{.Image}} {{.Ports}} {{.Names}}"
19d5a2d4f7f0 :: nginx 0.0.0.0:32768->80/tcp sleepy_cohen
{{< /highlight >}}

Let's try to use docker-compose now -

![digitalocean-nodes](/images/do5.png)

{{% admonition note %}}
The above command failed because docker-compose version < 1.24.0 doesn't support ssh connection to a remote docker engine. The latest version of docker-compose (as of writing 1.24.0-rc1) supports connecting to remote docker engine using ssh protocol. So lets upgrade to the latest docker-compose
{{% /admonition %}}

![digitalocean-nodes](/images/do6.png)

Now let's create a docker-compose file using the same nginx image and this time exposing the container on port 8000.

![digitalocean-nodes](/images/do7.png)

As you see on executing the command `docker-compose -f docker-compose-ngix.yml up` it spinned up a container on my DigitalOcean droplet named do-node2 having ip 165.227.106.59 and if you try to access the same ip with the port 8000 you may see the default page for `nginx`.