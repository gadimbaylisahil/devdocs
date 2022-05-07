# Overview

Most of notes in Deployment section of diary contains information from [Deployment From Scratch](https://deploymentfromscratch.com/) book by [Joseph](https://strzibny.name/).

Steps involved in deployments:

- Provisioning of hardware

- Installing and configuring software

- Deploying code

> Server provisioning is the process of making hardware ready with correct infrastructure and essential software packages to keep them running. Infrastructure as a Code(IaC) aims to automate this process. Example automation tool: Terraform.

> Once resources are provisioned, it's time for configuration. Configuration is about maintaining a desired state that can support running applications. Examples can be NGNIX web server config, database configuration, firewall settings and so on. Example tools: Chef, Puppet, Ansible...

> Simplest form for deployment of code is through tools such as SFTP, to copy the application codebase over to servers and run the web servers. However, application deployments these days include further steps such as data migrations, bundling of assets. Example tools: Capistrano.

> Then there are Linux Containers where we push containers as code to servers where containers are initialized and built from. Example tools for automatization of containers: Kubernetes, OpenShift.

