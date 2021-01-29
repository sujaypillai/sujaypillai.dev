---
layout: post
title: "Azure Data Factory - Self Hosted Integration Runtime on Windows Container"
date: 2021-01-29T12:10:56+08:00
lastmod: 2021-01-29T12:10:56+08:00
draft: false
tags: ['azure', 'docker']
categories: ['Cloud']
---

If you are looking for how to get started with Azure Data Factory (ADF), I will definitely recommend you to go through
[Cathrine Wilhelmsen's](https://twitter.com/cathrinew) -- [Beginner’s Guide to Azure Data Factory](https://www.cathrinewilhelmsen.net/series/beginners-guide-azure-data-factory/). It helped me to understand the basics for ADF before starting work on a project I recently had a chance to participate at my workplace.

The compute infrastructure that is required by ADF provides the following data integration capabilities across different network environment:

1. Data Flow Execution
2. Data Movement Execution
3. Dispatch of Activities
4. SSIS Package Execution

Data Factory offers three types of [Integration Runtime (IR)](https://docs.microsoft.com/en-us/azure/data-factory/concepts-integration-runtime), and you should choose the type that best serve the data integration capabilities and network environment needs you're looking for. These three types are:

1. Azure
2. Self-hosted
3. Azure-SSIS

This blog will help you setup the Self-Hosted Integration Runtime (SHIR) in Windows Container and show you the steps to run it on a Windows machine as well as on Azure Container Instance.

### Prerequisites

1. Windows Container

    The Windows container feature is available on Windows Server (Semi-Annual Channel), Windows Server 2019, Windows Server 2016, and Windows 10 Professional and Enterprise Editions (version 1607 and later).

2. Docker Version 2.3 or later.
3. Self-Hosted Integration Runtime Version 4.11.7512.1 and later. [Download](https://www.microsoft.com/download/details.aspx?id=39717) the latest version available.

### Build the Docker image

To build the docker image we will  be using Windows 10 Professional with [Docker Desktop 3.1.0](https://www.docker.com/products/docker-desktop) in Windows Container mode.

Download the the latest source code for the SHIR windows container support [here](https://github.com/Azure/Azure-Data-Factory-Integration-Runtime-in-Windows-Container). 


Download the latest version of [SHIR](https://www.microsoft.com/en-us/download/details.aspx?id=39717) into `SHIR` folder [5.2.7681.6 in this case]

![SHIR_Location](/images/shir_location.png)

On the shell navigate to the folder where you downloaded the project from github to build the docker image. Execute the command below:

```powershell
cd "pathToTheGithubCheckout"
docker build -t sujaypillai/adfir:5.2.7681.6 .
```

![SHIR_BuildDockerImage](/images/shir_windowsimage_build.png)

You can find the same image on DockerHub as `sujaypillai/adfir:5.2.7681.6`

To register your SHIR Windows container you would need a valid Authentication Key which is passed as an environment variable `AUTH_KEY` in the command below:

```powershell
docker run -d -e NODE_NAME="SHIRWindowsContainer" \
              -e AUTH_KEY="GET_THE_KEY_FROM_IR_SETUP" \
              -e ENABLE_HA=true \
              -e HA_PORT=8060 sujaypillai/adfir:5.2.7681.6
```

To get the `AUTH_KEY` you will have to create a SHIR and get the value from the settings tab as shown below:

![SHIR_Setup](/images/shir_01.png)

![SHIR_Setup](/images/shir_02.png)


```powershell
docker run -d -e NODE_NAME="SHIRWindowsContainer01" \
              -e AUTH_KEY="IR@18bbe05f-4f23-4dc4-85bc-310ec29fc549@adf-sp-lab@ServiceEndpoint=adf-sp-lab.southeastasia.datafactory.azure.net@RUW7AAC7Voog0gJcFZ7tL9le/yE9PxTXNQj8WItroZk=" \
              -e ENABLE_HA=true \
              -e HA_PORT=8060 sujaypillai/adfir:5.2.7681.6
```

![SHIR_Setup](/images/shir_03.png)

![SHIR_Setup](/images/shir_04.png)

You can get more details about your SHIR by clicking on the `Monitor` icon:

![SHIR_Setup](/images/shir_05.png)

Now that we have the image pushed on DockerHub how about running it on [ACI](https://azure.microsoft.com/en-us/services/container-instances/) ?

Set the environment variables required by the SHIR docker image [sujaypillai/adfir:5.2.7681.6](https://hub.docker.com/layers/135093603/sujaypillai/adfir/5.2.7681.6/images/sha256-0e4e3846d083033908f2b09202be393d63047f5282463ee06138c4658cca1839?context=explore)

![SHIR_Setup](/images/shir_07a.png)

Lets deploy the container instance using Powershell as below:

![SHIR_Setup](/images/shir_07.png)

This being a windows container it takes sometime for the image to be pulled down and for the ACI instance to be up and running.

![SHIR_Setup](/images/shir_09.png)

Once the container is running you should see itself registering it as a SHIR.

![SHIR_Setup](/images/shir_08.png)

Thus we were able to run the SHIR Docker container on a virtual machine as well as an Azure Container Instance.
