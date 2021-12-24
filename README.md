# tmp - Simple temporary file uploads over SSH

_(This is still in the design stage.)_

# SYNOPSIS

	... | ssh tmp@tmp.example.com

# DESCRIPTION

tmp is a simple service for temporary file uploads over SSH. The file to
be uploaded is provided to ssh as stdin, and upon successful upload a
link to the file will be put to stdout. On an error, like exceeding a
size limit, a message will be put to stderr.

# OPTIONS

A custom time limit can be provided:

	$ cat report.pdf | ssh tmp@tmp.example.com 14d
	https://tmp.example.com/b4r8t75.pdf
	$ cat LICENSE | ssh tmp@tmp.example.com 2m
	https://tmp.example.com/z5rq6zr.txt

By default, files are deleted 2 days after they are uploaded.

A custom file name can be provided:

	$ cat wow.mp3 | ssh tmp@tmp.example.com 1w wow.mp3
	https://tmp.example.com/wow.mp3
	$ curl https://sr.ht/~smlavine/tmp | ssh tmp@tmp.example.com 2d tmp
	https://tmp.example.com/tmp.html

The name cannot be more than 128 bytes long, and must contain only
letters, numbers, dots, underscores, and hyphens. **No spaces**. Some
Unicode characters might work but it is not guaranteed.

A file ending will be appended if one is not provided.

By default, a file is given a randomly generated name. Characters in the
name are from the string "abcdefghijkmnpqrstuvwxyz23456789". Letters and
numbers that might cause confusion with others in the set are removed.

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

	useradd -m tmp  # If you want, set a shell for easier administration
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
a personal blog, you must create a new subdomain for the tmp service
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

# BUGS

A custom name cannot be provided unless a custom time limit is also
provided. Another way of designing this could have been to use options
instead:

	$ ... | ssh tmp@tmp.example.com -- -t 1w -n file.txt

The ```--``` is necessary to prevent ```ssh``` from parsing the ```-t```
as its own. This syntax is a bit too long for my liking, but I may
consider instead persuing something like

	$ ... | ssh tmp@tmp.example.com t=1w n=file.txt
	$ ... | ssh tmp@tmp.example.com n=recording.mp4 foo=baz bar

---
TODO: continue writing this as I write the service.
