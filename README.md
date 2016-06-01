# Docker Alpine Nginx PHP-FPM
This is a Dockerfile to build a container image for Nginx and PHP-FPM. 

## Installation
To build the container : 
```
docker build -t ${USER}/lnp https://github.com/captnbp/docker-simple-lnp.git
```

## Running
To simply run the container:

```
sudo docker run --name lnp -p 8080:80 -d ${USER}/lnp
```
You can then browse to http://\<docker_host\>:8080 to view the default install files.


### Nginx configuration
You can also add a custom Nginx file for your site. For example nginx-site.conf
```
server {
	listen   80; ## listen for ipv4; this line is default and implied
	listen   [::]:80 default ipv6only=on; ## listen for ipv6

	root /usr/share/nginx/html;
	index index.php index.html index.htm;

	# Disable sendfile as per https://docs.vagrantup.com/v2/synced-folders/virtualbox.html
	sendfile off;

	#error_page 404 /404.html;

	# redirect server error pages to the static page /50x.html
	#
	error_page 500 502 503 504 /50x.html;
	location = /50x.html {
		root /usr/share/nginx/html;
	}

	# Rewrites
	location / {
		try_files $uri $uri/ /index.php;

		# PHP engine
		location ~ \.php$ {
			try_files      $uri =404;
			fastcgi_pass   unix:/var/run/php-fpm.sock; # Can be different
			fastcgi_index  index.php;
			fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
			include        fastcgi_params;
		}
	}

        location ~* \.(jpg|jpeg|gif|png|css|js|ico|xml)$ {
                expires           5d;
        }

	# deny access to . files, for security
	#
	location ~ /\. {
    		log_not_found off; 
    		deny all;
	}

}
```

Then add it in your container :
```
ADD nginx-site.conf /etc/nginx/sites-enabled/default.conf
```

## Example for Owncloud
Here is a Dockerfile to create an Owncloud Docker :
```
FROM captnbp/docker-simple-lnp

MAINTAINER Benoît Pourre <benoit.pourre@gmail.com>

ADD nginx-site.conf /etc/nginx/sites-enabled/default.conf

COPY src /usr/share/nginx/html/
```


