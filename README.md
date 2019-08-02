# NGINX - Configuração em modo Proxy usando SSL

Conteúdo do arquivo /etc/nginx/nginx.conf

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
        worker_connections 768;
        # multi_accept on;
}


http {

	server {
	    server_name seusite.com.br www.seusite.com.br;

	    # IP interno / endereço da sua aplicação interna sendo executada de um container docker ou VM
	    set $upstream 127.0.0.1:81;

	    location / {
        	proxy_pass_header Authorization;
	        proxy_pass http://$upstream;
	        proxy_set_header Host $host;
	        proxy_set_header X-Real-IP $remote_addr;
	        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	        proxy_http_version 1.1;
	        proxy_set_header Connection “”;
	        proxy_buffering off;
	        client_max_body_size 0;
	        proxy_read_timeout 36000s;
	        proxy_redirect off;
	    }

	    listen 443 ssl; # managed by Certbot
	    ssl_certificate /etc/letsencrypt/live/seusite.com.br/fullchain.pem; # managed by Certbot
	    ssl_certificate_key /etc/letsencrypt/live/seusite.com.br/privkey.pem; # managed by Certbot
	    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
	    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
	}

	server {
	    if ($host = seusite.com.br) {
	        return 301 https://$host$request_uri;
	    } # managed by Certbot

	    server_name seusite.com.br www.seusite.com.br;
	    listen 80;
	    return 404; # managed by Certbot
	
	}

}
```
