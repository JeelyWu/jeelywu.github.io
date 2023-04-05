---
title: Use Docker Compose Healthy Check To Run Command Periodically in a Container
date: 2023-03-25T23:27:32+08:00
---

In my work, I have a domain that use the [certbot](https://hub.docker.com/r/certbot/certbot) container to renew the ssl certificate from letsencrypt.

It has two containers work together, the certbot container and the nginx container. 

If the certbot container renew the certificate successfully, it will replace the old certificate file with new one. So, the nginx needs to reload the certificate file.

Unfortunately, the certbot container doesn't have a way to notify the nginx container to reload the certificate file.  

So in early of this week, our domain faced a problem that the certificate expired even the new certificate was generated successfully!

After to reload nginx manually, the problem was solved. But I don't want to do it manually again, caz it is hard to remember to do it, in every 3 months.

After searching on the Internet, I found it is not easy to do that when the certbot and nginx is running in different containers. 

Yes I know, certbot provide a way to run a command after renew the certificate, but not work in my case.
Here is some options can work in my case:
- Use the `inotifywait` command in host machine to watch the certificate file change, then login into the nginx container to reload.
- Use the `inotifywait` command in nginx container to watch the certificate file change, then run the nginx reload command in nginx container.
- Use the `crontab` in host machine to log into nginx container and run the nginx reload command periodically.
- Use the docker compose `Healthy Check` to run the nginx reload command periodically.

Let me analyze the options one by one.

The option by using `inotifywait`, you know, the command `inotifywait` is a native command in Linux, you need to install it at first.
There is a different package which contain the `inotifywait` command in different Linux distribution, you can google it if you insterested.
After install the `inotifywait` command, you have to write a shell to watch the file change by using the command. 
The option `inotifywait` has a little complicated


The option by using `crontab`, it is easy, but not the best choice. 
Because, you need to config the nginx reload command in crontab, which is another place, if you are using docker-compose to start up series of containers.
It is easy to ignore when you are migrate the nginx container to another machina.

So the docker compose `Healthy Check` is the prefect one to solve this problem.