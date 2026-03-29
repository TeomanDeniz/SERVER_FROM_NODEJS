
# Setting up the machine

```sh
apt update && apt upgrade -y
apt install nodejs ufw npm git nginx certbot python3-certbot-nginx -y
```

## Connect Github Account to Server

```
ssh-keygen
cat /root/.ssh/*.pub
```

Copy the `ssh-ed25519 AA...1 root@...` and paste it into your Github Settings in SSH section.

## Alias Setup

Add these to `~/.bashrc`

```sh
pop() {
  git -C "__FRONTEND_PATH__" pull || return 1
  cp ".env" "__FRONTEND_PATH__/.env"
}
nr() {
  systemctl daemon-reload
  systemctl restart SERVER
}
u() {
  read -p "Are you sure? (Y/N): " c
  [[ "$c" =~ ^[Yy]$ ]] || return 1
  echo "Updating server..."
  pop || return 1
  nr
}

alias rn='nr'
alias log='cat /var/log/SERVER.log'
alias rmlog='echo "" > /var/log/SERVER.log'
alias logrm='rmlog'
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
ExecStartPre=/bin/sh -c 'echo "" > /var/log/SERVER.log'
ExecStart=/bin/sh -c 'exec /usr/bin/node __BACKEND_PATH__/INDEX.js >> /var/log/SERVER.log 2>&1'
WorkingDirectory=__BACKEND_PATH__/
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
systemctl enable SERVER
systemctl status SERVER
```
