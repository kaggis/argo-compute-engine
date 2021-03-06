---
title: Compute Engine documentation | ARGO
page_title: Compute - Installation
font_title: 'fa fa-cog'
description: This document describes the installation procedure of the ARGO compute engine
---


## Prerequisites

- You will need a RHEL 6.x or similar OS (base installation) to proceed. Note that the following instructions have been tested against CentOS 6.x OSes.
- Make sure that on your host an ntp client service is configured properly.

The first step is to install (as root user) the EPEL release package repository configuration via yum:

    # yum install epel-release

Create a new file with filename `/etc/yum.repos.d/argo.repo` and place within it the following contents:

    [argo-prod]
    name=ARGO Product Repository
    baseurl=http://rpm.hellasgrid.gr/mash/centos6-arstats/$basearch
    enabled=0
    gpgcheck=0
    
    [argo-devel]
    name=ARstats Development Repository
    baseurl=http://rpm.hellasgrid.gr/mash/centos6-arstats-devel/$basearch
    enabled=0
    gpgcheck=0

You will also need to install the cloudera repository for Hadoop components to be retrieved (although you will not install any Hadoop cluster, some libraries from the Hadoop ecosystem are needed). Under a file named `/etc/yum.repos.d/cloudera-cdh5.repo` place the following contents:

    [cloudera-cdh5]
    # Packages for Cloudera's Distribution for Hadoop, Version 5, on RedHat or CentOS 6 x86_64
    name=Cloudera's Distribution for Hadoop, Version 5
    baseurl=http://archive.cloudera.com/cdh5/redhat/$releasever/$basearch/cdh/5/
    gpgkey = http://archive.cloudera.com/cdh5/redhat/$releasever/$basearch/cdh/RPM-GPG-KEY-cloudera
    gpgcheck = 1
    enabled = 1

You will also need to install the EGI trustanchors repository (this is required on the host for communicating with topology providing services). Under a file named `/etc/yum.repos.d/EGI-trustanchors.repo` place the following contents:

    [EGI-trustanchors]
    name=EGI-trustanchors
    baseurl=http://repository.egi.eu/sw/production/cas/1/current/
    enabled=1
    gpgcheck=1
    gpgkey=http://repository.egi.eu/sw/production/cas/1/GPG-KEY-EUGridPMA-RPM-3


## Installation

Install (via `pip`) the latest version of the pymongo library:

    # yum install python-pip
    # pip install --upgrade pymongo

Install the component:

    # yum install ar-compute --enablerepo=argo-prod


## Configuration

Edit the `/etc/ar-compute-engine.conf` configuration file and

	- set the values of the `mongo_host` and `mongo_port` variables to point to a running mongo service
	- set the value of the `mode` variable to `local` or `cluster`
	- set the values of the `prefilter_clean` and `sync_clean` variables to either `true` of `false`

Under the folder `/etc/cron.d/` place two cronjobs that will handle hourly and daily calculations.

For the daily caclulations edit `/etc/cron.d/ar_job_cycle_daily` and place the following contents.

    0 0 * * * root /usr/libexec/ar-compute/standalone/job_cycle.py -d $(/bin/date --utc --date '-1 day' +\%Y-\%m-\%d)

Optionally - for the hourly caclulations edit `/etc/cron.d/ar_job_cycle_hourly` and place the following contents:

    55 * * * * root /usr/libexec/ar-compute/standalone/job_cycle.py -d $(/bin/date --utc  +\%Y-\%m-\%d)

More details on configuring A/R jobs (reports) can be found [here][internal_1].

## Log files

The compute engine uses by default the system syslog to log any messages. You may change this behaviour by editing the configucation file `/etc/ar-compute-engine.conf`. You may also wish to change the logging level by setting the `log_level` value to your preference.

[internal_1]: /guides/argo-compute-engine/report-configuration/