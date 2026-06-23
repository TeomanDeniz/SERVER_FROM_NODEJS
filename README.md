
# Setting up the machine

## Cloudflared install

```sh
apt update && apt upgrade -y
reboot
```

After restart:

```sh
apt install curl -y
```

[Cloudflare](https://pkg.cloudflare.com/index.html)

```sh
apt update
apt install nodejs ufw npm git vim rsync cloudflared -y
cloudflared tunnel login
cloudflared tunnel create HTTP_DOMAIN_NAME
cloudflared tunnel route dns HTTP_DOMAIN_NAME DOMAIN.NAME
vim ~/.cloudflared/config.yml
```

```
tunnel: HTTP_DOMAIN_NAME
credentials-file: /PATH/TO/.cloudflared/????????-????-????-????-????????????.json

ingress:
   - hostname: DOMAIN.NAME
     service: http://127.0.0.1:3000
   - service: http_status:404
```

```sh
cloudflared service install
```

## Connect Github Account to Server

```cmd
type %USERPROFILE%\.ssh\id_ed25519.pub
```
or
```sh
cat ~/.ssh/id_ed25519.pub
```

Copy the result and paste there at `~/.ssh/authorized_keys`

## Connect your Github Accont

```
ssh-keygen
cat /root/.ssh/*.pub
```

Copy the `ssh-ed25519 AA...1 root@...` and paste it into your Github Settings in SSH section.

## Security

```sh
vim /etc/ssh/sshd_config
```

```conf
PermitRootLogin yes # EDIT IF "no"
PasswordAuthentication no # EDIT
PubkeyAuthentication yes
```

```sh
systemctl restart ssh
```

## Alias Setup

```sh
vim ~/.bashrc
```

```sh
pop() {
  git -C ~/__BACKEND_PATH__ pull || return 1
  cp ~/.env ~/__BACKEND_PATH__/.env
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

alias status='systemctl status SERVER'
alias rn='nr'
alias log='cat /var/log/SERVER.log'
alias rmlog='echo "" > /var/log/SERVER.log'
alias logrm='rmlog'

export PS1='[ SERVER ] - (\w): '
```

```sh
source ~/.bashrc
```

# Ngnix

```sh
ufw allow 443
ufw allow 80
ufw allow 22
ufw enable
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
ExecStart=/bin/sh -c 'exec /usr/bin/node /root/__BACKEND_PATH__/MAIN.js >> /var/log/SERVER.log 2>&1'
WorkingDirectory=/root/__BACKEND_PATH__/
Restart=always
RestartSec=3
User=root

[Install]
WantedBy=multi-user.target
```

```sh
systemctl daemon-reload
systemctl start SERVER
systemctl enable SERVER
systemctl status SERVER
```

# Auto Reset system daily

```sh
timedatectl set-timezone Europe/Istanbul
```

```sh
vim /etc/systemd/system/DAILY_RESET_SERVER.service
```

```ini
[Unit]
Description=Restart Node app

[Service]
Type=oneshot
ExecStart=/bin/systemctl restart SERVER.service
```

```sh
vim /etc/systemd/system/DAILY_RESET_SERVER.timer
```

```ini
[Unit]
Description=Restart the SERVER app every day at 06:00 Turkey time

[Timer]
OnCalendar=*-*-* 06:00:00 Europe/Istanbul
Persistent=true

[Install]
WantedBy=timers.target
```

# Extra:

```sh
vim /etc/motd
```

```

Custom message when login success.

```

