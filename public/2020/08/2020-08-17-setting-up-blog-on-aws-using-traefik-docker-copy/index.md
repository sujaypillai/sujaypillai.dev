# Running Hugo in Docker + Traefik


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

## Definition of docker stack to run Traefik

{{< gist sujaypillai 8947d8baa3663b9da50e15f394efee6e >}}
