
# Setting up the machine

```sh
apt update && apt upgrade -y
apt install nodejs ufw npm git nginx certbot python3-certbot-nginx -y
```

Add these to `~/.bashrc`

```sh
alias log='cat /var/log/SERVER.log'
alias pop='git -C "__FRONTEND_PATH__" pull'
alias nr='systemctl restart node'
alias rn='systemctl restart node'
```

# Ngnix

```sh
ufw allow 443
ufw allow 80
```

Move your `NGNIX/http.conf` and `NGNIX/https.conf` files inside `/etc/nginx/conf.d`

```sh
systemctl start nginx
systemctl enable nginx
```

# SSL Certificate

```sh
certbot --nginx -d YOUR_DOMAIN -d YOUR_DOMAIN_2
```

# Setting up NODE service

```sh
vim /etc/systemd/system/SERVER.service
```

```ini
[Unit]
Description=SERVER
After=network.target

[Service]
ExecStartPre=rm -rf /var/log/node.log
ExecStart=/bin/sh -c 'exec /usr/bin/node __BACKEND_PATH__/INDEX.js >> /var/log/SERVER.log 2>&1'
WorkingDirectory=/root/ETICARET/NODE
Restart=always
RestartSec=1
User=root
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

```sh
systemctl daemon-reload
systemctl start SERVER
systemctl status SERVER
```
