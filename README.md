# Guacamole Docker
This repository is taken from https://github.com/boschkundendienst/guacamole-docker-compose but has been updated to include some new features.This includes
support for SAML and upstream SSL encryption. This has been tested and working with Keycloak 17.

## Installation

This includes all relevant packages needed. The only thing to do is to download and copy the guacamole-auth-saml-1.3.0.jar file into the config/extensions folder.

You have to use 1.3.0 (not 1.4.0) due to a bug in the 1.4.0 release.

Download the folder and run the following commands
```shell
chmod +x prepare.sh
./prepare.sh
docker-compose up
```

This will create the database init script and generate NGINX SSL keys for communication. The upstream server doesn't validate these but uses the SSL for local encryption across the network.

## NGINX Configuration

This image includes NGINX which encrypts local communications; however is configured to work from an upstream NGINX reverse proxy.

The NGINX configuration of that upstream server is shown below

```
server {
	listen 443 ssl http2;
	server_name guac.vizu.co.nz;

	ssl_certificate /etc/pki/tls/certs/cloudflare-vizu.co.nz.crt;
	ssl_certificate_key /etc/pki/tls/private/cloudflare-vizu.co.nz.key;

	#Require validation
	ssl_client_certificate /etc/pki/tls/certs/cloudflare_origin_rsa.crt;
	ssl_verify_client on;

	location / {
		#proxy_pass http://10.160.8.110:9080/guacamole/;
		proxy_pass https://10.160.8.110/guacamole/;
		proxy_ssl_verify off;
		proxy_buffering off;
		proxy_http_version 1.1;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection $http_connection;
		proxy_cookie_path /guacamole/ /;
		client_max_body_size 4096m;

		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-Proto "https";
		proxy_set_header Host $host;
		proxy_set_header X-Forwarded-Host $host;
		proxy_set_header X-Forwarded-Port $server_port;
		proxy_redirect off;
	}
}
```