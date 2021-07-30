---
title: Running Docker Desktop on Windows 11 in Azure"
date: 2021-07-30T19:14:11+08:00
draft: false
---

In this blog we will setup a Windows 11 machine in Azure and then install Docker Desktop on top of it. As of today we don't have Windows 11 image available in Azure's Marketplace. So first we will install the latest Windows 10 image and then by leveraging the Windows Insider program we can upgrade the box to Windows 11.

So lets start with installing a Windows 10 box using the image -

![Win10_Installation_in_azure](/images/ddwin11/01-CreateVm.png)



![Enable_Windows_Insider](/images/ddwin11/02-EnableWindowsInsider.png)

