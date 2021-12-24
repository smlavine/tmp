# Setting up tmp on a server

It is assumed that you have the following already installed on your system:

- A C library
- shadow (```useradd```, ```passwd```)
- Standard UNIX utilities (```ln```, ```mkdir```, etc.)
- Some text editor (in this guide ```vi``` is used)
- An SSH server (probably ```openssh-server```)
- systemd
- nginx
- letsencrypt / certbot

Additionally it is assumed that you have some domain name under your
control. In this guide that domain will be referred to as ```example.com```.

These steps were written against a Vultr VPS running Debian GNU/Linux 10.

## Create a new system user

This is the user that people will SSH into when using ```tmp```. It
doesn't matter what this user is called, but it makes sense to me to
call it tmp.

	useradd -m tmp
	chsh -s /bin/bash tmp  # Optional, for easier administration
	passwd tmp  # For now, at least, give the user a strong password

But when people log in as tmp over SSH, we don't want to give them
access to a shell on our server. We only want them to be able to use our
```tmp``` service. To accomplish this, add this to your
```/etc/ssh/sshd_config``` file:

	Match User tmp
		ForceCommand /home/tmp/tmp-login

Make sure to ```systemctl reload sshd``` and ```systemctl restart
sshd``` or it will not take effect.

You don't have to enable SSH login to this user yet, but it may be
helpful for testing purposes. If so, add your public SSH key to
```/home/tmp/.ssh/authorized_keys```.

## Set up a new subdomain

If you aren't already using your domain for anything, this step is
optional. But if you are already using it for another site, for example
a personal blog, you must create a new subdomain for ```tmp```
because we are making a new nginx site for it.

Well, now that I think about it, this isn't _strictly_ necessary, if
instead you want files to be available in a subdirectory in a site
already existing on the server. But you'll have to do things a bit
differently in a step or two in that case.

The specifics of how you set up your subdomain will depend on your
domain registrar, but it should result in two records that look
something like this:

	tmp	CNAME	example.com	300
	www.tmp	CNAME	example.com	300

## Make a new HTTP site with nginx

This is where uploaded files will be accessible by the world.

This is the nginx server config I wrote:

	server {
		listen 80;
		listen [::]:80;

		root /var/www/tmp;

		index data/index.html;

		server_name tmp.example.com www.tmp.example.com;

		location / {
			try_files $uri $uri.html $uri/data/index.html $uri/ =404;
		}
	}

One thing you may notice is that the index file is set to
```data/index.html``` instead of the usual ```index.html```. Because
uploaded files will be available at the top level of this site, any
files of our own we want to serve have to be in a subdirectory.

These are the commands I ran to set up this nginx site:

	vi /etc/nginx/sites-available/tmp  # See above
	mkdir -p /var/www/tmp/data
	vi /var/www/tmp/data/index.html  # Write whatever you want
	ln -s /etc/nginx/sites-available/tmp /etc/nginx/sites-enabled/tmp
	# Let us encrypt. Follow the prompts to select these domains
	# (tmp.* and www.tmp.*). I also selected the option to modify
	# the config to automatically redirect HTTP traffic to HTTPS.
	letsencrypt
	systemctl restart nginx

At this point you should have a live webpage at your equivilant of
```tmp.example.com```. Point your web browser to that address to see for
yourself. Make sure you've created a file at ```data/index.html```.

