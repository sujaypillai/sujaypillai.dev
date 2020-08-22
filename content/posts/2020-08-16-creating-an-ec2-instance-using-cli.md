---
layout: post
title: "Creating an EC2 instance using CLI"
date: 2020-08-17T12:10:56+08:00
lastmod: 2020-08-17T12:10:56+08:00
draft: false
tags: ['docker', 'aws']
categories: ['Cloud']
---

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

### Open port 22 (SSH protocol)to connect to your instance.

{{< highlight console >}}
aws ec2 authorize-security-group-ingress --group-name docker-machine --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name docker-machine --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name docker-machine --protocol tcp --port 443 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name docker-machine --protocol tcp --port 2376 --cidr 0.0.0.0/0
{{< /highlight >}}

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
ap-southeast-1
--count 1 --instance-type t2.micro
