///////////////////////////// NGNIX /////////////////////////////////////////
sudo apt update
sudo apt install nginx
sudo nano /etc/nginx/sites-available/default

sudo nginx -t
sudo service nginx restart

client_max_body_size 100M;
proxy_read_timeout 300;
proxy_connect_timeout 300;
proxy_send_timeout 300;

sudo ln -s /etc/nginx/sites-available/server1 /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/server2 /etc/nginx/sites-enabled/

sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d studenthunter.org -d www.studenthunter.org

server {
 server_name link.uz;
 root /var/www/html;
 index index.html;

 location / {
  try_files $uri $uri /index.html;
 }

    listen [::]:443 ssl http2;
    listen 443 ssl http2;
    ssl_certificate /etc/letsencrypt/live/studenthunter.uz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/studenthunter.uz/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}
server {
    if ($host = link.com) {
        return 301 https://$host$request_uri;
    }

    if ($host = www.studenthunter.uz) {
        return 301 https://$host$request_uri;
    } 

 server_name link.com;
    return 404;
}

server {
 server_name api.{link.com} link.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    listen [::]:443 ssl http2;
    listen 443 ssl http2;
    ssl_certificate /etc/letsencrypt/live/api.studenthunter.uz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.studenthunter.uz/privkey.pem; 
    include /etc/letsencrypt/options-ssl-nginx.conf; 
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; 
}

server {
    if ($host = api.link.com) {
        return 301 https://$host$request_uri;
    } 

    if ($host = www.api.link.com) {
        return 301 https://$host$request_uri;
    }

    server_name api.link.com www.api.link.com;
    return 404; 
}
/////////////////////////////////////////////////////////////////////////////////////////

//////////////////// CD //////////////////////////////////////////////
name: CD

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: connect to server and deploy changes
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.KEY }}
          port: 22
          script: |
            cd project/student-hunter/
            git pull origin main
            pm2 restart 0



script: |
            cd /var/www/html
            git pull origin main
//////////////////////////////////////////////////////////////////////

/////////////////////////////// PM2 MONGODB /////////////////////////////////////////////
npm install --global yarn
npm install pm2 -g

pm2 start yarn --name "studenthunter" -- dev

mongodump mongodb://localhost:27017/studenthunter
mongorestore dump/
////////////////////////////////////////////////////////////////////////////////////////

////////////////////////////////// SSH ////////////////////////////////////////////////
ssh-keygen
ssh-copy-id username@remote_host
cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"
cat ~/.ssh/id_rsa.pub
cat ~/.ssh/id_rsa
////////////////////////////////////////////////////////////////////////////////////////
