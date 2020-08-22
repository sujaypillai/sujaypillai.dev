---
layout: post
title: "Migrating from AWS to Azure"
date: 2020-08-21T12:10:56+08:00
lastmod: 2020-08-21T12:10:56+08:00
draft: false
tags: ['azure', 'aws']
categories: ['Cloud']
---

![Docker-GitHubAction](/images/docker-github-action-01.png)
Last week Docker released its first [Github Action](https://help.github.com/en/actions/getting-started-with-github-actions/about-github-actions) called [docker/build-push-action](https://github.com/docker/build-push-action) . This has been developed by Docker after keeping into consideration to simplify basic workflow of: building an image, tagging it, logging into Docker Hub, and pushing the image to a registry.

My previous blog [Docker images on Github Package Registry]({{< ref "2019-09-26-docker-github-package-registry.md" >}}) showcased how we could achieve the above workflow with Github package registry. If you look at the workflow definition [dockerimage.yml](https://github.com/sujaypillai/buildxdemo/blob/master/.github/workflows/dockerimage.yml) for this we had to write three small actions to achieve the above workflow. Lets replace [***we will actually create a new workflow definiton***] this with Docker's new GitHub action.

{{% admonition tip %}}
You can create more than one workflow in a repository. You must store workflows in the `.github/workflows` directory in the root of your repository.
{{% /admonition %}}

## Adding a workflow template:
On the main page of your repository navigate to Actions and click on `New workflow` button.

If you don't see ***Build and push Docker Images*** template in the list then click on `Set up a workflow yourself`
![Docker-GitHubAction](/images/docker-github-action-05.png)
\
\
Search for the template `Build and push Docker images` from the Marketplace.
![Docker-GitHubAction](/images/docker-github-action-06.png)
\
\
Make the changes in the editor and commit the action to your repository using the `Start Commit` button.
![Docker-GitHubAction](/images/docker-github-action-07.png)

If you check the workflow definition it has an environment variable `DOCKER_BUILDKIT` set to 1 which helps to switch to the new builder mechanism available in Docker from `18.09`.

## Creating encrypted secrets:
{{% admonition info %}}
***Encrypted secrets allow you to store sensitive information, such as access tokens, in your repository.***
{{% /admonition %}}

1. Under your repository name, click `Settings` > `Secrets`.
    ![Docker-GitHubAction](/images/docker-github-action-08.png)
2. Add the two secrets `DOCKER_USERNAME` (your dockerhub username) & `DOCKER_PASSWORD` (your PAT token from Docker Hub). 

    Docker Hub lets you create personal access tokens as alternatives to your password. Refer this [link](https://docs.docker.com/docker-hub/access-tokens/) to create your PAT.
3. You can access these secrets in your workflow using the `secrets` context.
    ```
    with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    ```

## Tagging options:
There are 3 options to tag your images with this action:
1. `tags` - Comma-delimited list of tags. These will be added to the registry/repository to form the image's tags.

    Example:

    ```yaml
    tags: tag1,tag2
    ```
2. `tag_with_ref` - Boolean value. Defaults to `false`.
    Automatically tags the built image with the git reference. The format of the tag depends on the type of git reference with all forward slashes replaced with `-`.

    For pushes to a branch the reference will be `refs/heads/{branch-name}` and the tag will be `{branch-name}`. If `{branch-name}` is master then the tag will be `latest`.

    For pull requests the reference will be `refs/pull/{pull-request}` and the tag will be `pr-{pull-request}`.

    For git tags the reference will be `refs/tags/{git-tag}` and the tag will be `{git-tag}`.

    Examples:

    |Git Reference|Image tag|
    |---|---|
    |`refs/heads/master`|`latest`|
    |`refs/heads/my/branch`|`my-branch`|
    |`refs/pull/2/merge`|`pr-2-merge`|
    |`refs/tags/v1.0.0`|`v1.0.0`|
3. `tag_with_sha` - Boolean value. Defaults to `false`.

    Automatically tags the built image with the git short SHA prefixed with `sha-`.

    Example:

    |Git SHA|Image tag|
    |---|---|
    |`2e0f87924a562826da882179798e2e083680c2d9`|`sha-2e0f879`|

    ![Docker-GitHubAction](/images/docker-github-action-09.png)

## References:    
* [Build+Push official Docker GitHub action](https://github.com/docker/build-push-action)
* [Getting started with GitHub Actions](https://help.github.com/en/actions/getting-started-with-github-actions)
* [Context and expression syntax for GitHub Actions](https://help.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions)