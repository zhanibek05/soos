# SoOS Prep

# Bash history

```bash
HISTCONTROL=ignorespace # will ignore lines that start with space
```

```bash
history -d linenumber # will delete entry from history file
```

```bash
history -w # to save
```

# Installations

```bash
sudo dnf install tmux
tmux
```

ctrl+b %

```bash
sudo dnf install git neovim
```

docker:

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf-3 config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
```

[https://docs.docker.com/engine/install/fedora/](https://docs.docker.com/engine/install/fedora/)

[https://docs.fedoraproject.org/en-US/quick-docs/installing-docker/](https://docs.fedoraproject.org/en-US/quick-docs/installing-docker/)

podman:

```bash
sudo dnf -y install podman
```

[https://podman.io/docs/installation](https://podman.io/docs/installation)

zsh

```bash
sudo dnf install zsh
chsh -s $(which zsh)
```

# Services

nginx.service:

```bash
[Unit]
Description=nginx web server
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
PrivateDevices=yes
PrivateTmp=true
SyslogLevel=err

ExecStart=/usr/bin/nginx
ExecReload=/usr/bin/nginx -s reload
Restart=on-failure
KillMode=mixed
KillSignal=SIGQUIT
TimeoutStopSec=5

[Install]
WantedBy=multi-user.target
```

simple example
 /etc/systemd/system/myapp.service

```bash
[Unit]
Description=My Go App
After=network.target

[Service]
User=appuser
ExecStart=/opt/myapp/myapp
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
# Users

```bash
usermod && useradd
```

# Permissions

rwx:

000 - 0 no permissions

001 - 1 only execute

010 - 2 only write

011 - 3 write and execute

100 - 4 only read

101 - 5 read and execute

110 - 6 read and write

111 - 7 read, write and execute 

rwx rwx rwx

user group others

chmod u/g/o/a -/+/= 

u - user, g - group, o - others, a - all

“-” - forbid

“+” - permit (give access)

“=” - assigning value

```bash
chmod g+w file.txt # group can edit file
chmod a+wx file.txt # all can edit and execute file
chmod go=rw file.txt # group and others can read and write
```

```bash
chmod 555 file.txt
```

```bash
chown <user>:<group> file
```

# NFTables

To block everything except 22, 80, 443 and 8080:

```bash
# ip ip6 inet
#flush ruleset
table ip mytable {
	chain INPUT {
		type filter hook input priority filter; policy accept;
		# 1) allow all loopback traffic
    iifname lo accept
    
		tcp dport 22 accept
		tcp dport 80 accept
		tcp dport 443 accept
		tcp dport 8080 accept
		reject
	}
}
```

```bash
nft -f input.nft # to add
nft list ruleset

# Add a new table with family "inet" and table "filter":
sudo nft add table inet filter

# Add a new chain to accept all inbound traffic:
sudo nft add chain inet filter input \{ type filter hook input priority 0 \; policy accept \}

# Add a new rule to accept several TCP ports:
sudo nft add rule inet filter input tcp dport \{ telnet, ssh, http, https \} accept

# Add a NAT rule to translate all traffic from the `192.168.0.0/24` subnet to the host's public IP:
sudo nft add rule nat postrouting ip saddr 192.168.0.0/24 masquerade

# Show rule handles:
sudo nft --handle --numeric list chain family table chain

# Delete a rule:
sudo nft delete rule inet filter input handle 3

# Save current configuration:
sudo nft list ruleset > /etc/nftables.conf

```

To enable:

```bash
sudo systemctl enable nftables
sudo systemctl start  nftables
```

To configure NAT

```bash
table ip nat {
	chain prerouting {
		type nat hook prerouting priority dstnat; policy accept;
		iifname "eth0" tcp dport { 80, 443 } dnat to 192.168.2.1
	}
	chain postrouting {
		type nat hook postrouting priority srcnat; policy accept;
		oifname "eth0" masquerade
	}
}
```

To configure Round Robin

```bash
table ip rb {
	сhain prerouting {
		type nat hook prerouting priority dstnat; policy accept;
		dnat to numgen inc mod 2 map { 0 : 192.168.2.1, 1 : 192.168.2.2 }
	}
	chain postrouting {
		type nat hook postrouting priority srcnat; policy accept;
		oifname "eth0" masquerade
	}
}



```

# Scripting

```bash
#!/usr/bin.bash
#shebang
```

Bash scripting example:



# Crontab


```bash

# Edit the crontab file for the current user:
crontab -e

# Edit the crontab file for a specific user:
sudo crontab -e -u user

# Replace the current crontab with the contents of the given file:
crontab path/to/file

# View a list of existing cron jobs for current user:
crontab -l

# Remove all cron jobs for the current user:
crontab -r

# Sample job which runs at 10:00 every day (* means any value):
0 10 * * * command_to_execute

# Sample crontab entry, which runs a command every 10 minutes:
*/10 * * * * command_to_execute

# Sample crontab entry, which runs a certain script at 02:30 every Friday:
30 2 * * Fri /absolute/path/to/script.sh


# crontab format
* * * * *  command_to_execute
- - - - -
| | | | |
| | | | +- day of week (0 - 7) (where sunday is 0 and 7)
| | | +--- month (1 - 12)
| | +----- day (1 - 31)
| +------- hour (0 - 23)
+--------- minute (0 - 59)

# example entries
# every 15 min
*/15 * * * * /home/user/command.sh


```



# Containers

Dockerfile example:

```Dockerfile
# Dockerfile
FROM golang:1.24-alpine AS builder

WORKDIR /app

COPY go.mod ./
COPY main.go ./
RUN go build -o server

#stage 2
FROM alpine:latest

RUN adduser -D appuser

COPY --from=builder /app/server .

USER appuser

EXPOSE 8080

CMD ["./server"]
```


Build docker image run and push to registry:

```bash
docker login # login 

docker build -t zhanibek05/myapp:latest .

docker run -p 8080:8080 zhanibek05/myapp:latest 

docker push zhanibek05/myapp:latest

```
