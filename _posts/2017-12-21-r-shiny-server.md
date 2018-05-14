---
layout: post
title: R Shiny Server
date:   2017-12-21 10:18:00
tags: R Statistics
subclass: 'post tag-fiction'
navigation: True
logo: 'assets/images/ghost.png'
cover: 'assets/images/rainier-3.jpg'
author: hackz
categories: hackz
---

Setting up your own shiny server can be helpful to host a few small projects that you want to share with a few collaborators. I am going to go through the setup of starting a server, like through Amazon EWS or Digital Ocean, and through the necessary steps that you will need to take in order to be able to get access to that server securely. These instructions are specific to `ubuntu` 16.04 but shouldn't change that much across different versions.  

The first thing to do is set up a new user that isn't `ubuntu` that we can log in with via ssh later and then we can disable logging in with the `ubuntu` user, which is way to common a name. Check out the following code to create a new user and give it `sudo` privileges.  

```
# before you do anything lets update our system
sudo apt-get update && sudo apt-get dist-upgrade
# create newuser
sudo adduser newuser
# grant sudo privileges
sudo usermod -aG sudo newuser
```

You are also going to set up authorized keys for this user and make sure to disable password logins in the `/etc/ssh/sshd_config` file by uncommenting or changing the line `PasswordAuthentication yes` to `no`, after we have tested that our ssh keys are working, followed by a `sudo systemctl reload sshd`. Depending on how may precautions you want to take it may be a good idea to also change the port number for ssh protocol in the same file just so your server isn't such an easy target for automated attacks.  

```
# On home computer
# follow the instructions and be sure to remember save location
ssh-keygen -t rsa
# copy the credentials to the server
ssh-copy-id newuser@server_ip
cat ~/.ssh/key.pub | ssh newuser@server_ip "cat >  ~/.ssh/authorized_keys"
 # test out the key
ssh -i ~/.ssh/key.pub newuser@server_ip
```

We now want to setup a firewall for added security so that we are exactly in control of what process may access our system. The command `sudo ufw app list` will list all of the programs that are directly compatible with the simple firewall Ubuntu uses which is likely only `OpenSSH` at this point. We can tell the firewall to allow ssh to the default port by running `sudo ufw allow OpenSSH`. We can allow ssh into other ports, if you took the precautions mentioned earlier, with the command `sudo ufw allow <port number>/tcp`. So your command list might look something like this.

```
# check out the apps ufw supports
sudo ufw app list
# allow OpenSSH
sudo ufw allow OpenSSH
# allow ssh to some other <port number> if necessary
sudo ufw allow <port number>/tcp
# enable the firewall
sudo ufw enable
# see the programs that are allowed by the firewall
sudo ufw status
```

We now want to set up a web server and the easiest one to work with on Ubuntu is nginx. The instructions below are pretty self explanatory after the previous explanation.  

```
# install nginx
sudo apt-get install nginx
# make sure nginx shows up on the list now for the firewall
sudo ufw app list
# allow nginx for both http and https protocol
sudo ufw allow 'Nginx Full'
# check the status to make sure nginx is good
sudo ufw status
# make sure nginx is running you should see Active (running) as the status
systemctl status nginx
```

You can then check to make sure that the webserver is running by visiting your ip in a web browser. You should see the welcome page for nginx. More details for setting up the system can be found [here](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04). Because we also want to run encrypted transactions over https as well as a bot that will renew your certificates after they expire which happens in 30 days when using the free ssl certification service offered by Lets Encrypt.  

```
# get the cert bot lets encrypt ppa for Ubuntu
sudo add-apt-repository ppa:certbot/certbot
# gotta update your system again
sudo apt-get update
# install certbot
sudo apt-get install certbot python-certbot-nginx
# get your initial certificates
sudo certbot certonly --nginx -d your.ip.or.domain
```

We can make sure that the full process of renewal is automated by editing `/etc/letsencrypt/renewal/example.com.conf` where `example.com` is your ip address or DNS and adding the following file `renew_hook = systemctl reload rabbitmq`. You can test out the renewal process with a dry run `sudo certbot renew --dry-run`.  

It may be helpful to set up a web address that links to your ip at this point but that is beyond the scope of this tutorial. Before we get started with R shiny install the last thing that you need to do is install the latest version of `R`.  

```
sudo add-apt-repository "deb http://cran.rstudio.com/bin/linux/ubuntu $(lsb_release -sc)/"
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9
sudo add-apt-repository ppa:marutter/rdev
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install r-base
```

With that out of the way you should be able to use the [Digital Ocean guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-shiny-server-on-ubuntu-16-04) for installing a R shiny server and be good to go.
