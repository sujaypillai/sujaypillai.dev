# Docker images on Github Package Registry


![GithubAnnouncement](/images/github-package-registry-01.png)

{{% admonition question %}}
**Ever wondered to have your source code and packages in one place?**

[GitHub Package Registry](https://github.com/features/packages) is a package management service that makes it easy to publish public or private packages next to your source code. It is fully integrated with GitHub, so you can use the same search, browsing, and management tools to find and publish packages as you do for your repositories. You can also use the same user and team permissions to manage code and packages together. 

It supports familiar package management tools:
* JavaScript (npm)
* Java (Maven)
* Ruby (RubyGems),
* .NET (NuGet)
* Docker images
{{% /admonition %}}

## Authenticating to GitHub Package Registry
You can use the docker cli to use GitHub Package Registry to publish and retrieve docker images.
{{< highlight console >}}
$ docker login docker.pkg.github.com -u sujaypillai -p
{{< /highlight >}}

## Publishing a package (docker image)
Let's try to build the docker image for the repository - [buildxdemo](https://github.com/sujaypillai/buildxdemo) which has a Dockerfile within the project. After checking out the project execute the docker build

{{< highlight console >}}
$ docker build -t docker.pkg.github.com/sujaypillai/buildxdemo/demo . 
Sending build context to Docker daemon   98.3kB
Step 1/6 : FROM golang AS build
---> c4d6fdf2260a
....
Step 6/6 : CMD ["/hello"]
---> Using cache
---> 38c5db73c5a7
Successfully built 38c5db73c5a7
Successfully tagged docker.pkg.github.com/sujaypillai/buildxdemo/demo:latest
{{< /highlight >}}

The tagging of the image should follow the below pattern -

{{< highlight console >}}
docker.pkg.github.com/owner/repository/image_name:version
{{< /highlight >}}

{{% admonition info %}}
Because upper case letters aren't supported, you must use lowercase letters for the repository owner even if the GitHub user or organization name contains uppercase letters.
GitHub Package Registry supports multiple top-level Docker images per repository.
{{% /admonition %}}

Push the image to GitHub Package Registry:
{{< highlight console >}}
$ docker push docker.pkg.github.com/sujaypillai/buildxdemo/demo
The push refers to repository [docker.pkg.github.com/sujaypillai/buildxdemo/demo]
bf1eeaedd583: Layer already exists
latest: digest: sha256:c8ddfad1bfe68db341bc34d1cbe1f093de1eb8b73463396c67603934d8e49d46 size: 528
03f6ad6134dc: Layer already exists
v1: digest: sha256:b9ce4d4b077075e698c37a99f3e0a54d43d5583e164c6b493ad25a7f467edc40 size: 528
{{< /highlight >}}

You can access your packages from this URL by replacing **OWNER** with your GitHub user or organization name and **REPOSITORY** with your repository name:
{{< highlight console >}}
https://github.com/OWNER/REPOSITORY/packages
{{< /highlight >}}
i.e. https://github.com/sujaypillai/buildxdemo/packages and it should look similar to -
![GithubPackageRegistry](/images/github-package-registry-02.png)

## Package Insights
![GithubPackageRegistryInsights](/images/github-package-registry-03.png)
Docker images hosted on GitHub include details and download statistics, along with their entire history, reference command to pull image from command line and the instruction to use it as base image in `Dockerfile`.

As Github package registry doesn't support [Image Manifest v2,Schema 2](https://docs.docker.com/registry/spec/manifest-v2-2/) it doesn't support multi-arc docker images. Below is the error what I got when trying to build a multi-arc docker image using buildx and push to Github package registry -
![GithubPackageRegistryNoSupportForImageManifestv2](/images/github-package-registry-04.png)

{{% admonition bug %}}
Please upvote this issue on [GitHub](https://github.community/t5/GitHub-API-Development-and/Handle-multi-arch-Docker-images-on-GitHub-Package-Registry/m-p/31650) if you would love to see multi-arc docker image support on Github package registry.
{{% /admonition %}}