# Kubecon EU 2025 Hacking Up a Storm

This is the repository where we're keeping all the files needed for the Kubecon EU 2025 Tutorial - [Hacking Up a Storm](https://kccnceu2025.sched.com/event/1tx72/tutorial-hacking-up-a-storm-with-kubernetes-rory-mccune-datadog-marion-mccune-scotsts-iain-smart-amberwolf?iframe=yes&w=100%&sidebar=yes&bg=no).

## Content in this repository

- `kubecon-eu-25-slides.pdf` - PDF of our slides for the tutorial
- `Commands/tutorial-commands.md` - This is all of the commands that we'll use in the tutorial
- `manifests` - These are all the manifests that we'll deploy in the main scenario
- `CVE-2020-8554` - This directory contains files for the bonus scenario we'll have a look at it we get time


## Pre-Requisites

You'll need [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) installed, which requires Docker but should work on Windows, MacOS, or Linux.

## Setup Information

If you're in the tutorial and missed the slides on setup, do not panic! Here's the steps you'll need to get the environment set-up. 

Then 

```bash
cd Kubernetes-EU-2025
```

Then create the cluster

```bash
kind create cluster --config kind-cluster.yaml
```

and add all the manifests in the `manifests` directory


```bash
kubectl apply -f manifests/
```

## Getting to the developer environment

We'll be running our "attacks" from a pod in the cluster. To get to that shell, use this command, and you'll be good to go.

```bash
kubectl exec -it -n dev deployments/workstation -- /bin/bash
```