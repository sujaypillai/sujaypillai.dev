# Creating an EC2 instance using CLI


This blog post highlights the steps taken by me for creating an EC2 instance to be used as a part of docker swarm environment to host my blogging site on AWS using [Traefik](https://docs.traefik.io/). Below are the commands I used to spin up an EC2 instance using the [AWS CLI version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) .


### Find the image id corresponding to Amazon Linux 2 AMI

{{< highlight console >}}
aws ec2 describe-images --owners amazon \
    --filters 'Name=name,Values=amzn2-ami-hvm-2.0.????????-x86_64-gp2' 'Name=state,Values=available' \
    --output json | jq -r '.Images | sort_by(.CreationDate) | last(.[]).ImageId'

ami-01f7527546b557442
{{< /highlight >}}

### Create a security group

{{< highlight console >}}
aws ec2 create-security-group --group-name docker-machine --description "Docker Machine"

{
    "GroupId":"sg-0f1144719b279f8cb"
}
{{< /highlight >}}

### Open port 22 (SSH protocol)to connect to your instance and other ports for docker swarm.

{{< highlight console >}}
aws ec2 authorize-security-group-ingress --group-name docker-machine --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name docker-machine --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name docker-machine --protocol tcp --port 443 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name docker-machine --protocol tcp --port 2377 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name docker-machine --protocol tcp --port 7946 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name docker-machine --protocol udp --port 7946 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name docker-machine --protocol udp --port 4789 --cidr 0.0.0.0/0
{{< /highlight >}}

The following PORTS are added to security group:

  Port                      |     Description
 ---------------------      | -------
  22                        | SSH to the docker node
  80,443                    | HTTP,HTTPS connection to the docker node
  2377                      | Cluster management & raft sync communications
  7946                      | Control plane gossip discovery communication between all nodes
  4789                      | Overlay network traffic (container ingress networking).


### Create a Key Pair to connect to EC2
{{< highlight console >}}
aws ec2 create-key-pair --key-name inlets --query 'KeyMaterial' --output text > inlets.pem
{{< /highlight >}}

### Create an EC2 instance

{{< highlight console >}}
aws ec2 run-instances --image-id ami-01f7527546b557442 \
                      --security-group-ids sg-0f1144719b279f8cb \
                      --instance-type t2.micro         \
                      --key-name inlets 
{{< /highlight >}}