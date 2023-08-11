+++
title = "Complete Data Loss: A Root Cause Analysis"
date = 2023-08-11T15:18:22-05:00
draft = false
toc = true
images = []
tags = ["rook", "kubernetes"]
categories = ["kubernetes"]
projects = []
+++


## The Event

On Thursday, August 10th 2023, at about 5:30 AM, all data that was hosted on my self-hosted, private Kubernetes cluster was lost, as well as the cluster itself.

## Background

Migration to a Dual Stack configuration of Kubernetes was being made, in which the server would be able to expose all services both over IPv4 as well as IPv6, to allow for faster connections for users connecting over IPv6.

During this migration, a misconfiguration left the [API Server](https://kubernetes.io/docs/concepts/overview/kubernetes-api/) unable to respond to inqueries.

Due to an [outstanding issue](https://github.com/rook/rook/issues/4274) in Rook Ceph, the storage system utilized on the server, the manager for Rook Ceph upon being unable to receive configurations proceeded to remove __all__ aspects of configuration, including the storage cluster itself.

This issue was not previously known to the operator, however, backups of volumes were not yet in place at the time the event.

## Risk Reduction Actions Taken

Research into other storage systems as well as means to back them up has been conducted. Unfortunately, other storage systems do not meet with the current and future goals of the project. Backup of non-recoverable data to a cloud storage will ensure no data loss can occur.

## Prevention strategies

1. __Cloud backup:__ (Estimated cost ~$5/mo per 250 GiB)

    Every hour, any Persistant Volume that has data that is deemed to not be easily recoverable will be backed up to DigitalOcean Spaces, an object storage solution. Backups will be be made hourly, with one week's backups retained.
2. __Testing of changes in Virtual Machines__

    All future changes that affect the cluster as a whole will be tested in a virtual machine, located on the operator's workstation.