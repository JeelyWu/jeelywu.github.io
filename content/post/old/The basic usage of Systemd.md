---
title: "Note: The basic usage of Systemd"
date: 2022-08-20T15:18:23+08:00
---

Systemd is a system and service manager for Linux. In this post, I will document its basic usage.

### Check if it is installed, or its version in your machine

```shell
systemctl --version
```

Yes, **systemd** providers the **systemctl** command.  We always use the systemctl command to deal with our service.


### The Man page

```shell
man systemd 
```

You can type this command in your Linux server to find the document of systemd


### The directories
There are two important directories for systemd. One is `/usr/lib/systemd/system`, in which all the services are stored in.
And another directory is `/etc/systemd/system`, which stored the service which will automatically start when the system starts.
In general, The services stored in the second directory are a soft link to the first directory.

### The systemctl command

```
systemctl [option] service-name
```

You can use the systemctl command to manipulate your service. This command has two arguments, the option which I will show below and the service name

### The options
There are seral options for systemctl command:
- start: start a service
- stop: stop a service
- restart: restart a service
- reload: reload a service (in some cases, we need to make the new config take effect)
- status: check the status of a service
- is-active: check whether the service is active now.
- is-enable: check whether the service starts automatically when the system boot.
