---
layout: post
title: "Add Authentication to Shiny Server With Nginx"
categories: r
tags: [r, shiny]
---



Shiny Server is a great tool, but I’ve always found it odd that there was no built-in password authentication. Sure, the Shiny Pro edition has SSL auth., but even for open source projects, I’m not really crazy about just anyone hitting my server whenever they want.

To solve this little problem, I whipped up two work-arounds. One solution uses an Nginx server with basic authentication and the second uses Nginx with SSL auth.

## Ubuntu vs. CentOS
From here on out, we’ll be using the same locations and .conf files for both. The one CentOS specific difference is to make sure we disaple SELinux, otherwise our reverse-proxy will go into a bad gateway.


{% highlight bash %}
sed -i /etc/selinux/config -r -e 's/^SELINUX=.*/SELINUX=disabled/g'
{% endhighlight %}

## Deploy Shiny Server with Nginx Basic Authorization

The trick is to have Shiny only serve to the localhost and have Nginx listen to localhost and only serve to users with a password. This is fairly straight forward and involves editing the Nginx default.conf as well as the Shiny Server conf.

First, make sure you’ve got Nginx installed.


{% highlight bash %}
sudo apt-get install nginx

{% endhighlight %}

Nginx uses ufw firewall on Ubuntu, so you’ll have to start ufw and enable the correct ports.


{% highlight bash %}
sudo ufw enable

sudo allow 'Nginx Full'
{% endhighlight %}

Also, make sure you’ve got Apache2-utils, you’ll use this to store the usernames and passwords.


{% highlight bash %}
sudo apt-get install apache2-utils

{% endhighlight %}

Before you go on, shut down both Shiny and Nginx


{% highlight bash %}
sudo systemctl stop nginx

sudo systemctl stop shiny-server

{% endhighlight %}

Next, you’ll need to edit the Nginx default.conf file.


{% highlight bash %}
sudo nano /etc/nginx/sites-available/default

{% endhighlight %}

Copy and paste the following into your default.conf


{% highlight bash %}
server http {

  map $http_upgrade $connection_upgrade {
      default upgrade;
      ''      close;
    }

  server {
    listen 80;
  
    location / {
      proxy_pass http://127.0.0.1:3838/;
      proxy_redirect http://127.0.0.1:3838/ $scheme://$host/;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
      proxy_read_timeout 20d;
    }
  }
{% endhighlight %}

Once that’s done, you’ll need to edit Shiny Server’s conf file so it only serves to loaclhost. Otherwise users would be able to creep around your authentication by going to port 3838.


{% highlight bash %}
sudo nano /etc/shiny-server/shiny-server.conf

{% endhighlight %}

Copy and paste the below to your shiny-server.conf.


{% highlight bash %}
server{
    listen 3838 127.0.0.1;
    
    location / {
    site_dir /srv/shiny-server;
    log_dir /var/log/shiny-server;
    directory_index on;
    }
}

{% endhighlight %}

Now it’s time to create some usernames and passwords.


{% highlight bash %}
cd /etc/nginx
sudo htpasswd -c /etc/nginx/.htpasswd exampleuser
{% endhighlight %}

Restart Nginx and Shiny.


{% highlight bash %}
sudo systemctl start nginx

sudo systemctl start shiny-server

{% endhighlight %}

Ta-da, now you’ve got a password protected Shiny Server! Note, Shiny is now served by port 80 instead of port 3838!

![](https://github.com/keberwein/keberwein.github.io/blob/master/images/sslnginxauth1.png?raw=true)

## Deploy Shiny Server with Nginx SSL Authorization

This is basically the same as above, but we’re going to direct the reverse-proxy to port 443 with SSL instead of port 80. The only “gotcha” is we’ll need a signed SSL certificate to view the page. There’s two ways to go about this: use a self-signed certificate with IP addresses or to use a trusted certificate with a domain name. Since this is just testing, I’ll use the self-signed method. If you need a trusted certificate, there’s a good tutorial on using letsencrypt to get a free trusted cert.

First we have to create a self-signed certificate. This is going to live in the nginx folder for ease of use.


{% highlight bash %}
cd /etc/nginx
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/server.key -out /etc/nginx/server.crt
{% endhighlight %}

Now use the same nginx default.conf method as above but add lines to read the SSL cert.


{% highlight bash %}
# Redirect all traffic from port 80 to SSL port
server {
    listen 80;
    return 301 https://$host$request_uri;
}
# Set reverse proxy to port 443
server {
    listen 443 ssl;
        ssl on;
        ssl_certificate /etc/nginx/server.crt;
        ssl_certificate_key /etc/nginx/server.key;
    
    location / {    
        proxy_pass http://127.0.0.1:3838;
        proxy_redirect http://127.0.0.1:3838/ https://$host/;
        auth_basic "Username and Password are required"; 
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
{% endhighlight %}

The changes to shiny-server.conf are the same as above.


{% highlight bash %}
server{
    listen 3838 127.0.0.1;
    
    location / {
    site_dir /srv/shiny-server;
    log_dir /var/log/shiny-server;
    directory_index on;
    }
}
{% endhighlight %}

If everything is working correctly, you should be staring at an ugly error message in your browser telling you that this is an “unsafe website.” This is due to the self-signed certificate. Just ignore that, add an exception and you should be confronted with a login box.

![](https://github.com/keberwein/keberwein.github.io/blob/master/images/sslauthnginx2.png?raw=true)

This is purely for testing purposes. This hasn’t been fully tested so don’t go putting it into production. If you really want to take things a step further, I would look into getting a trusted cert with [letsencrypt](http://www.morphatic.com/2015/12/01/finally-free-ssl-certs-lets-encrypt/), so you won’t have to deal with the ugly error page.

One more thing, the above is a VERY basic Nginx setup, the full-monty for the Nginx conf file would probably look something like this:


{% highlight bash %}
# Redirect all traffic from port 80 to SSL port
server {
    listen 80;
    return 301 https://$host$request_uri;
}
# Set reverse proxy to port 443
server {
    listen 443 ssl;
        ssl on;
        ssl_certificate /etc/nginx/server.crt;
        ssl_certificate_key /etc/nginx/server.key;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    
    location / { 
        proxy_set_header        Host $host;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Proto $scheme;

        proxy_pass http://127.0.0.1:3838;
        proxy_redirect http://127.0.0.1:3838/ https://$host/;
        auth_basic "Username and Password are required"; 
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
{% endhighlight %}

