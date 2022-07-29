---
title : "Kubernetes"
date : "2022-07-29T20:56:08+02:00"
author : "Santiago Pastor Serrano"
tags : ["kubernetes"]
keywords : ["kubernetes"]
description : "Mi entorno y conocimiento básico de Kubernetes"
---

# Kubernetes

This page contains my knowlage on the tool Kubernetes.
## Basic knowlage

### What is Kubernetes?

Kubernetes is a container orchestration tool. (TLDR helps you manage a lot of containers in a centralized way)

It is a very powerful tool. But, why do we need it. Because a lot of applications now use a lot of services. Thus they use a lot of containers and organizing 10 containers is a pain, but 200 is a suicide. So, Kubernetes lets you manage them in a centralized way so that the task of updating a high availability website isn't as pedantic as updating 40 containers manually. For this, Kubernetes (or more exactly kubectl) uses a yaml file as a template for how to manage the containers.

Before knowing how does this all work, you should better understand the elements of a Kubernetes cluster.

### Components of a Kubernetes cluster

#### Master node

A master node is the server (or servers) designated with the task of telling the Worker nodes what to do. They also are the only access point to the cluster so, if you loose the access to one of these in a production environment, start screaming. XD Just joking, the majority of production environments have a couple of them just in case. The components of a Master node are the following:

- API Server: The entry-point to the cluster. It talks to the different possible clients. (Such as an UI, API or CLI).

- Controller manager: keeps track of what is happening in the cluster.( a node that needs a restart, a container that has shot down, those kinds of things)

- Scheduler: Designates where the pods and containers are going to be created based on workload and available resources.

- Etcd: Where Kubernetes stores the backups of the different states of the cluster. (Config and status data of every node)

#### Worker Node

The worker nodes are the ones that actually do all the work (as the name implies). They can be real or virtual and contain/use: 

- Kublet: Is a containerized process that allows the nodes to speak to each other and exchange information.(Also called "node agent")

Its main goal is to store and use pods (Explained later).

#### Virtual network

Allows the different nodes to talk to each other. (In simple terms, makes the entire cluster into a super machine that has all the resources of all the nodes)

#### Pods

The smallest object that you can admin with Kubernetes. Most of the time, there will only be 1 container inside each pod, but some applications that need more than one service can have multiple containers inside each one (such as WordPress because it needs a database and a Web server). There can be multiple pods inside each worker node.

### Setting up a basic testing enviroment for learning

#### Applications that I used

virtualbox and vagrant

```bash
sudo yay -S virtualbox vagrant
```

#### How to get it up and running



