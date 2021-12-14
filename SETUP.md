This is a log of the steps I've taken so far in setting up ```tmp``` on
my server.

1. Set tmp and www.tmp CNAME records for my domain, smlavine.com.
2. Make a new HTTP site using nginx. This is where the uploaded files
will be accessible by the world.

This is the nginx server config I wrote:

	server {
	        listen 80;
	        listen [::]:80;
	
	        root /var/www/tmp;
	
	        index data/index.html;
	
	        server_name tmp.smlavine.com www.tmp.smlavine.com;
	
	        location / {
	                try_files $uri $uri.html $uri/data/index.html $uri/ =404;
	        }
	}

One thing you may notice is that the index file is set to
```data/index.html``` instead of the usual ```index.html```. Because
uploaded files will be available at the top level of this site, any
files of our own we want to serve have to be in a subdirectory.

These are the commands I ran to set up the HTTP frontend:

	nvim /etc/nginx/sites-available/tmp  # See above
	mkdir -p /var/www/tmp/data
	nvim /var/www/tmp/data/index.html  # Write whatever you want
	ln -s /etc/nginx/sites-available/tmp /etc/nginx/sites-enabled/tmp
	# Let us encrypt. Follow the prompts to select these domains
	# (tmp.* and www.tmp.*). I also selected the option to modify
	# the config to automatically redirect HTTP traffic to HTTPS.
	letsencrypt
	systemctl restart nginx  # Use your web browser to check it works!

3. TODO: mention steps to create tmp user which I had already done.
TODO: continue writing this as I go.

The layout and presentation of this file will probably change with time,
but I'm just getting the ideas down for now.
