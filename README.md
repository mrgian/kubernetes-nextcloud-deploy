# Nextcloud deploy on Kubernetes with MySQL
Kubernetes objects definitions for deploying Nextcloud with MySQL

## How to deploy
Deploying example on a fresh Arch Linux install with Minikube

### Preparation
Install Docker, Minikube, kubectl and nginx:
```zsh
sudo pacman -S docker minikube kubectl nginx
```
Add user to docker group:
```zsh
sudo gpasswd -a $USER docker
```
Reboot:
```zsh
sudo reboot
```

### Make persistent data directories
These dirs will be used as persistent volumes in Kubernetes.
```zsh
cd
mkdir data
mkdir data/nextcloud
mkdir data/nextcloud/data
mkdir data/nextcloud/config
mkdir data/nextcloud/apps
mkdir data/mysql
```

### Start Minikube
Start Minikube specifying in which dir persistent volumes will be mounted
```zsh
minikube start --mount --mount-string="$HOME/data:/home/docker/data" --driver=docker
```

### Apply configurations
```zsh
git clone https://github.com/mrgian/kubernetes-nextcloud-deploy.git
cd kubernetes-nextcloud-deploy
kubectl apply -f mysql-config.yaml
kubectl apply -f mysql-secret.yaml
kubectl apply -f mysql.yaml
kubectl apply -f nextcloud.yaml
```

### Check Nextcloud service URL
```zsh
minikube service nextcloud-service
```

The Nextcloud instance will be available at the shown URL.

## Configure reverse proxy in Nginx
If you are running Kubernetes in Minikube, you may want your Nextcloud instance to be accessible from outside.
You can setup a reverse proxy with Nginx

Edit nginx.conf:
```zsh
sudo nano /etc/nginx/nginx.conf
```

Configure the reverse proxy:
```zsh
server {
    listen       80;
    server_name  localhost;
    
    location / {
        proxy_pass http://192.168.49.2:30001;
    }
}
```

Restart Nginx:
```zsh
sudo systemctl restart nginx
```
