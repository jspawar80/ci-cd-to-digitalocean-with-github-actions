## Preriqisites  ( This document refered for ubuntu linux only )

This Github Action pipeline setups and deploys the NodeJs app with PM2 and also setups the nginx configuration for the given domain.

* Install NodeJs, NPM, PM2, Nginx
* Make sure you point the IP to correct domain/subdomain

### 1. Create `/var/app` Dir in server

### 2. Create Github Action file in repo path `.github/workflows/deploy.yaml`

```
name: Build & Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy NodeJS app
        uses: appleboy/ssh-action@v0.1.2
        with:
          host: ${{secrets.SSH_HOST}} # IP address of the server you wish to ssh into
          port: ${{secrets.SSH_PORT}} 
          key: ${{secrets.SSH_KEY}} # Private or public key of the server
          username: ${{ secrets.SSH_USERNAME }} # User of the server you want to ssh into     
          script: |
             if [ -d "/var/app/${{ secrets.APP_NAME }}" ]; then cd /var/app/${{ secrets.APP_NAME }} && git pull && pm2 restart ${{ secrets.APP_NAME }}; else cd /var/app/ && git clone git@github.com:infonextsolutions/${{ secrets.APP_NAME }}.git && cd /var/app/${{ secrets.APP_NAME }} && npm i && pm2 delete ${{ secrets.APP_NAME }} || : pm2 list && pm2 start index.js -f --name "${{ secrets.APP_NAME }}" && echo "setting Nginx" && ls && sed 's/example.com/${{ secrets.DOMAIN_NAME }}/g' ./nginx/nginx.conf > /etc/nginx/sites-available/${{ secrets.DOMAIN_NAME }} && sed -i 's/APP_PORT/${{ secrets.APP_PORT }}/g' /etc/nginx/sites-available/${{ secrets.DOMAIN_NAME }} && service nginx reload && service nginx restart &&  echo "Error: Directory /var/app/${{ secrets.APP_NAME }} dir does not exists."; fi
```

### 3. Setup Action secrets in Github Repo

* `APP_NAME` >> Same as Repo Name
* `SSH_HOST` >> Host/IP of the server
* `SSH_KEY` >>  SSH Private Key 
* `SSH_USERNAME` >>  SSH username i.e (ubuntu,root)
* `DOMAIN_NAME` >> Domain name for the application
* `APP_PORT` >> Application port ( can be found in Application Index file )


### 3. Create nginx config for domain in root folder of repo at `nginx/nginx.conf`

```
server {
  listen 80;
  server_name example.com;
        return 301 https://$host$request_uri;
server_tokens off;
}
 
server {
  listen 443 ssl;
  server_name example.com;
  server_tokens off;
  location / {
      proxy_pass http://localhost:3003/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
  }
        ssl_certificate         /etc/nginx/ssl/nextsolutions.in/server.crt;
        ssl_certificate_key     /etc/nginx/ssl/nextsolutions.in/server.key;
        ssl_dhparam  /etc/nginx/ssl/dhparam.pem;
        ssl_session_cache shared:SSL:50m;
        ssl_session_timeout 1440m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;

        ssl_ciphers "ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS";
}
```
ssh-keygen -t ed25519 -a 200 -C "Example@gmail.com"
chmod 700 .ssh/authorized_keys
ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts

### Optional ( If certificate path is diffrent ) just change the path in below lines
 
```
       ssl_certificate         /etc/nginx/ssl/nextsolutions.in/server.crt;
       ssl_certificate_key     /etc/nginx/ssl/nextsolutions.in/server.key;
```
