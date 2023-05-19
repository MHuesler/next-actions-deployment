# Configure NextJS auto-deployment using GitHub Actions

## On VPS
1. Install NodeJS
2. Install Nginx and configure port proxy where the Next app will be served (also setup certbot)
```
# redirect http to https
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  server_name mydomain.com www.mydomain.com;
  return 301 https://$server_name$request_uri;
}

server {
  # listen on *:443 -> ssl; instead of *:80
  listen 443 ssl http2 default_server;
  listen [::]:443 ssl http2 default_server;

  server_name flowy.email;

  ssl_certificate /etc/letsencrypt/live/mydomain.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/mydomain.com/privkey.pem;
  include snippets/ssl-params.conf;

  location / {
    # reverse proxy for next server
    proxy_pass http://localhost:7000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;

    # we need to remove this 404 handling
    # because next's _next folder and own handling
    # try_files $uri $uri/ =404;
  }

  location /_next/static {
     add_header Cache-Control "public, max-age=3600, immutable";
     proxy_pass http://127.0.0.1:7000/_next/static;
  }

  location ~ /.well-known {
    allow all;
  }
}
```
4. Install Pm2

## Actions Workflow
1. In GitHub Repo -> Actions -> Select NodeJS Workflow
2. Edit publish.yml

```
name: Node.js CI

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

jobs:
  build:
    runs-on: self-hosted

    strategy:
      matrix:
        node-version: [18.x]
        
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: npm run build --if-present
    - run: pm2 reload homepage
```

3. Settings -> Actions -> Add Runner

## On VPS
1. Follow Commands from GitHub Add Runner
2. Start pm2 `pm2 start npm --name "next-js" --start`
3. In actions-runner folder
  3.1. `sudo ./svc.sh install`
  3.2. `sudo ./svc.sh start`
  3.3. `sudo ./svc.sh status`
