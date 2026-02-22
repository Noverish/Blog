```conf


server {
  listen 80;

  location /auth {
    rewrite ^/auth(?<realurl>/.*)$ $realurl break;
    proxy_set_header 'X-Real-IP' $remote_addr;
    proxy_pass http://auth;
  }

  location /api/ {
    rewrite ^/api/(.*)$ /$1 break;
    proxy_set_header 'X-Real-IP' $remote_addr;
    proxy_pass http://api;
  }

  location /api/socket.io {
    rewrite ^/api/(.*)$ /$1 break;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_pass http://api;
    proxy_buffering off;
    # proxy_redirect off;
  }

  location / {
    add_header Content-Type text/plain;
    return 200 "Hello, World!";
  }
}

```