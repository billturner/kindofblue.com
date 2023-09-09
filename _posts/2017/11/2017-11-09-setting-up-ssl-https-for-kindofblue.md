---
layout: post
title:  "Setting up SSL/HTTPS with Let's Encrypt for kindofblue.com"
date: 2017-11-09 22:06:00 -0500
description: "I use Let's Encrypt and a handy utility to set up SSL/HTTPS for the kindofblue.com domain"
tags:
- nginx
- security
- site
---
With an upcoming version of Firefox set to [warn users when they're browsing on a site without SSL encryption enabled](https://blog.mozilla.org/security/2017/01/20/communicating-the-dangers-of-non-secure-http/), I figured it wouldn't hurt to get it all set up for this domain - `kindofblue.com` - as an experiment. I knew that with [Let's Encrypt](https://letsencrypt.org/ "The Let's Encrypt site"), the SSL certificates are free. It just requires a bit of work to get them set up.

Luckily, at DigitalOcean - where I host this site/domain - they have [this handy guide for getting HTTPS set up with LetsEncrypt](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04) (for [Ubuntu Linux](https://ubuntu.org/)). Since I'm using the same HTTP server (`nginx`), I was able to follow along without needing any extra help.

I did have to adjust my DNS setup at [Namecheap](https://namecheap.com/), since I had a wildcard (`*`) catch-all domain record. Once all I had was my necessary DNS entries (minus the wildcard), I followed along with the instructions pretty much as given.

Here's the quick version of the setup instructions:

```
# add certbot repository
sudo add-apt-repository ppa:certbot/certbot

# update package list
sudo apt-get update

# install the python certbot client
sudo apt-get install python-certbot-nginx
```

Next, make sure that you have an `nginx` configuration file with the proper domains listed. In my case it was in the file `/etc/nginx/sites-available/kindofblue.com` where the config included:

```
server {
  server_name kindofblue.com www.kindofblue.com;
  ...
}
```

They then explain to make sure that if you're using a firewall, to allow both HTTP and HTTPS traffic through. Their example is with the `ufw` utility, but I double checked that I'm allowing both kinds of traffic through with the firewall I'm using, `iptables`:

```
$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
...
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:https
...
```

We now need to get a certificate using the `certbot` client (again, using my own domain as example):

```
sudo certbot --nginx -d kindofblue.com -d www.kindofblue.com
```

You'll be asked a few questions, but the process is quite quick. When it asks for the HTTPS settings, I asked it to make all non-HTTPS requests redirect to the HTTPS domain. If you look in your appropriate nginx configuration file, it should look like this:

```
# Redirect non-https traffic to https
if ($scheme != "https") {
  return 301 https://$host$request_uri;
} # managed by Certbot
```

If that section is still commented out, then you'll need to uncomment and then restart `nginx` with: `sudo service nginx restart`

Once that's done, and there were no hiccups in any of the steps above, you should now be serving your site securely. So now, this blog is now served only over HTTPS! This was _so_ much easier than I had expected it to be. And as a part of the `certbot` setup, it installs a recurring task to update the certificate when needed - since Let's Encrypt certs only last for 3 months at a time.

Once I get everything set up and going with [Lists of Bests](https://kindofblue.com/tag/lists-of-bests/), then I'll _sort of_ know what I'm doing.
