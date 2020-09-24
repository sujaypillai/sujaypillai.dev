---
layout: post
title: "Running Hugo in Docker + Traefik"
date: 2020-08-17T12:10:56+08:00
lastmod: 2020-08-17T12:10:56+08:00
draft: false
tags: ['docker', 'aws']
categories: ['Cloud']
---

This post basically highlights how you can setup your blogging site (or any static website) using [Hugo](https://gohugo.io/documentation/) running in a Docker Swarm environment with [Traefik](https://docs.traefik.io/). This whole setup is done on AWS, but it could be replicated to any cloud service provider. In my case its only a single node docker swarm cluster. 


## Install Docker on an Amazon EC2 instance

* The steps to spin up EC2 instance on AWS is [here](https://www.sujaypillai.dev/2020/08/2020-08-16-creating-an-ec2-instance-using-cli/)

* Connect to your instance
{{< highlight console >}}
ssh -i /path/inlets.pem ec2-user@ec2-54-254-122-14.ap-southeast-1.compute.amazonaws.com
{{< /highlight >}}

* Update the installed packages and package cache on your instance, install docker and start the docker service.
{{< highlight console >}}
sudo yum update -y
sudo amazon-linux-extras install docker
sudo service docker start
{{< /highlight >}}

* Add the ec2-user to the docker group so you can execute Docker commands without using sudo.
{{< highlight console >}}
sudo usermod -a -G docker ec2-user
{{< /highlight >}}


## Initialize the Docker Swarm
{{< highlight console >}}
docker swarm init --advertise-addr eth0
{{< /highlight >}}

## List nodes in the swarm
{{< highlight console >}}
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
9fj7jdv4zfkgp6nlg229qqtvr *   sujaypillai.dev     Ready               Active              Leader              18.06.1-ce

{{< /highlight >}}

## Definition of docker stack to run Traefik

{{< gist sujaypillai 8947d8baa3663b9da50e15f394efee6e >}}

* The above stack uses Traefik version 1.7.13
* Traefik routes all incoming http/https request from the public address of the server and relays those request to the targeted service running privately on the server following some rules. This is achieved by publishing the ports 80 & 443.
* You can configure Traefik to use an ACME provider (Let's Encrypt) for automatic certificate generation. All the flags `acme.*` are related to ACME configuration and the certificates generated are stored in a volume named `traefik_certs`.
* Use `htpasswd` utility to create an encrypted password and save the generated password in a file named htpasswd which is passed in the config section.
{{< highlight console >}}
htpasswd -c htpasswd administartor
{{< /highlight >}}
* The `EMAIL` & `DOMAIN` are set as an environment variable on the host.


## Deploying the Traefik service as proxy
{{< highlight console >}}
docker stack deploy -c docker-stack-traefik.yml proxy
{{< /highlight >}}

{{< highlight console >}}
docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                 PORTS
utjpltombxnr        proxy_traefik       replicated          1/1                 traefik:1.7.13        
{{< /highlight >}}

{{< highlight console >}}
docker service ps proxy_traefik
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE          ERROR               PORTS
ksaooufxkgsn        proxy_traefik.1     traefik:1.7.13      sujaypillai.dev     Running             Running 8 months ago                       *:443->443/tcp,*:80->80/tcp
{{< /highlight >}}

## Deploying blogging site (Hugo)

{{< gist sujaypillai 0090fa3c8fbb84b00c1e2f4b7a1f7a90 >}}

{{< highlight console >}}
docker stack deploy -c docker-stack-sujaypillai.yml web
{{< /highlight >}}

{{< highlight console >}}
docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                 PORTS
utjpltombxnr        proxy_traefik       replicated          1/1                 traefik:1.7.13      
5s2blx0kicj0        web_website         replicated          1/1                 klakegg/hugo:0.63.2   *:1313->1313/tcp  
{{< /highlight >}}

{{< highlight console >}}
docker service ps web_website
ID                  NAME                IMAGE                 NODE                DESIRED STATE       CURRENT STATE          ERROR               PORTS
dewhzxakrfei        web_website.1       klakegg/hugo:0.63.2   sujaypillai.dev     Running             Running 8 months ago  
{{< /highlight >}}

