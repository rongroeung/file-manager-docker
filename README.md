# Setup File Manager Using Docker
## I. Create Working Directory for File Manager
### 1. Create a folder on path `/opt`
```
cd /opt/
mkdir /opt/file-manager
mkdir /opt/file-manager/data
mkdir /opt/file-manager/var
mkdir /opt/file-manager/var/prod
```

### 2. Create `docker-compose.yml` file
```
cd /opt/file-manager/var/prod
vi docker-compose.yml
```
#### >>> Paste the following content:
```
version: '3.8'

services:
  filebrowser:
    image: hurlenko/filebrowser
    ports:
      - 7003:8080
    volumes:
      - /opt/file-manager/data:/data
    environment:
      - FB_BASEURL=/filebrowser
    restart: always
```

### 3. Start Container Using docker compose
```
cd /opt/file-manager/var/prod
docker compose up -d
```

## II. Configure HTTPS Using Nginx Proxy

### 1. Add Configuration File for Internal Service
#### >>> Create a `file-manager.conf` on path `/etc/nginx/conf.d`
```
vi /etc/nginx/conf.d/file-manager.conf
```
#### >>> Paste the following content:
```
server {
    listen 7004 ssl;
    server_name crossroadscambodia.church;

    ssl_certificate /opt/https-httpd/fullchain.pem;
    ssl_certificate_key /opt/https-httpd/privkey.pem;

    location / {
        proxy_pass http://192.168.10.111:7003;  # URL of file-manager service
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
#### >>> Replace `7004` with your desired https port.
#### >>> Replace `crossroadscambodia.church` with your actual domain name.
#### >>> Replace `http://192.168.10.111:7003` with your actual service url.
#### >>> Update `/opt/https-httpd/fullchain.pem` and `/opt/https-httpd/privkey.pem` with the paths to your SSL certificate and key files.
#### >>> If you don't have SSL certificates, you may generate using this guideline: https://github.com/rongroeung/apache-httpd-https?tab=readme-ov-file#i-generate-ssltls-certificate

### 2. Test Nginx Configuration
#### >>> Before restarting Nginx, it's a good practice to test the configuration for syntax errors:
```
nginx -t
```
#### >>> If the test is successful, you should see: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok.

### 3. Restart Nginx
#### >>> After verifying the configuration, restart Nginx for the changes to take effect:
```
systemctl restart nginx
```

## III. Access File Manager in Browser
### 1. Login to File Manager
#### >>> URL: https://crossroadscambodia.church:7004/
#### >>> The default username is `admin`, and the password is `admin`

### 2. Change Password for `admin` (Optional)
#### >>> Go to `Settings` -> `Profile Settings` -> `Change Password` -> Input Your New Password -> `UPDATE`

### 3. Create New User (Optional)
#### >>> Go to `Settings` -> `User Management` -> `New` -> Input `Username` and `Password` -> Set Permissions -> `SAVE`
