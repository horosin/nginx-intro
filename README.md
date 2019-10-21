# NGINX intro workshop

## Presentation
Available here:
[presentation.pdf](presentation.pdf)

## Just serve some static files
```
docker run -v $(pwd):/usr/share/nginx/html/ -p 8080:80 --rm nginx:alpine
```

## Error pages setup
```
server {
    listen       80;
    listen [::]:80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
    }

    error_page 404 /404.html;
    location = /custom_404.html {
            root /usr/share/nginx/html;
            internal;
    }

    error_page 500 502 503 504 /50x.html;
    location = /custom_50x.html {
            root /usr/share/nginx/html;
            internal;
    }

    location /testing {
        fastcgi_pass unix:/does/not/exist;
    }
}

```

## Proxying

### Proxy spec
```
location /web2/ {
    proxy_pass http://web2/;
}
```

### Default proxy settings
```
# default proxy settings
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

## HTTPS

### Generate self-signed certificates
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./nginx-selfsigned.key -out ./nginx-selfsigned.crt
```

### Server setup
```
server {
    listen       80;
    listen [::]:80;

    server_name localhost;

    return 302 https://$server_name$request_uri;
}
```

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name  localhost;
    ...

    ssl_certificate     /certs/nginx-selfsigned.crt;
    ssl_certificate_key /certs/nginx-selfsigned.key;
}
```

## Generate .htpasswd
```
sudo htpasswd -c /etc/apache2/.htpasswd user1
```

Config:
```
auth_basic "Administrator’s Area";
auth_basic_user_file /.htpasswd; 
```