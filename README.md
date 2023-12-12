# Setting up a Web Server with SSH, Nginx, and a Simple Backend Service

## Create a User and Configure SSH

```bash
sudo useradd -ms /bin/bash web
sudo passwd web
sudo usermod -aG sudo web
sudo cp -r .ssh /home/web
sudo chown -R web:web /home/web/.ssh
```
## Edit SSH configuration:
```
cd /etc/ssh
sudo vim sshd_config
sudo systemctl restart ssh.service
```
# Install and Configure Nginx
```
sudo apt update
sudo apt install nginx
```
##Configure Firewall (UFW):
```
sudo apt install ufw
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
sudo ufw status
```
## Create a simple HTML page:
```
cd /var/www/html
sudo vim index.html
```
## Configure Nginx default site:
```
sudo vim /etc/nginx/sites-available/default
sudo nginx -t
```
# Set Up Simple Backend Service
```
## Upload files via SFTP:
```
sftp -i .ssh/do-key web@64.23.142.184
put -r \Users\drham\Downloads\hello-server
```
## Move and configure backend service:
```
sudo mkdir /opt/backend/
sudo cp ~/hello-server /opt/backend/
sudo chmod +x /opt/backend/hello-server
```
## Create a systemd service:
```
sudo vim /etc/systemd/system/hello-server.service
sudo systemctl daemon-reload
sudo systemctl start hello-server
```
#files edited
#sudo vim /etc/nginx/sites-available/default

Default server configuration
server {
    listen 80;
    listen [::]:80;

    root /var/www/index.html;

    index index.html;

    server_name 24.144.85.252;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }

    location /api {
        # Define the reverse proxy settings
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

     location /hey {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /echo {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}


## cd /var/www/html/index.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>

 /etc/systemd/system/hello-server.service

[Unit]
Description=A web api that says hello
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=www-data
Restart=on-failure

# Specify the path to your backend binary
ExecStart=/opt/backend/your-backend-binary
WorkingDirectory=/opt/backend

# Redirect stdout and stderr to log files
StandardOutput=file:/var/log/hello-server/output.log
StandardError=file:/var/log/hello-server/error.log
SyslogIdentifier=hello-server

[Install]
WantedBy=multi-user.target

