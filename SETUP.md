This is a log of the steps I've taken so far in setting up ```tmp``` on
my server.

1. Create a new user. You can name it whatever you want but ```tmp```
seems like a good one. This is the user that people will SSH into to use
the service.

	useradd -m tmp  # If you want, set a shell for easier administration
	passwd tmp  # For now, at least, give the user a strong password

2. Add the following to the end of your ```/etc/ssh/sshd_config```:

	Match User tmp
		ForceCommand /home/tmp/tmp-login

This will make it so that those logging in as ```tmp``` can do nothing
except interact with the tmp service.

3. If you have password authentication disabled, add your public SSH key
to ```/home/tmp/.ssh/authorized_keys```.

4. Set tmp and www.tmp CNAME records for my domain, example.com.

5. Make a new HTTP site using nginx. This is where the uploaded files
will be accessible by the world.

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

These are the commands I ran to set up the HTTP frontend:

	vi /etc/nginx/sites-available/tmp  # See above
	mkdir -p /var/www/tmp/data
	vi /var/www/tmp/data/index.html  # Write whatever you want
	ln -s /etc/nginx/sites-available/tmp /etc/nginx/sites-enabled/tmp
	# Let us encrypt. Follow the prompts to select these domains
	# (tmp.* and www.tmp.*). I also selected the option to modify
	# the config to automatically redirect HTTP traffic to HTTPS.
	letsencrypt
	systemctl restart nginx  # Use your web browser to check it works!

TODO: continue writing this as I go.

The layout and presentation of this file will probably change with time,
but I'm just getting the ideas down for now.
