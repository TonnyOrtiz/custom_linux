# Fix Wireless Hotspot on Fedora 42
How to fix the issue of devices connecting to the wireless Hotspot on fedora 42 not getting internet but connecting to the hotspot.



### 1. Direct iptables rules if firewalld forwarding fails(Needed)
``` bash
sudo iptables -A FORWARD -i wlp0s20f0u9i2 -o enp4s0 -j ACCEPT
sudo iptables -A FORWARD -i enp4s0 -o wlp0s20f0u9i2 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -t nat -A POSTROUTING -s 10.42.0.0/24 -o enp4s0 -j MASQUERADE
```
### 2. Make it Persistent
First save the current rules with iptables-save. Choose a directory and file.
```bash
sudo sh -c "iptables-save > /home/tonny/Documents/iptables/rules.v4"
```

### 3. Create a Demon that restores the the rules each time the system boots
This needs revision since maybe there is another way to do this.

Create a file:
```bash
sudo vi /etc/systemd/system/iptables-restore.service
```

Paste the following:
```
[Unit]
Description=Restore iptables rules
After=network.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c "/usr/sbin/iptables-restore < /home/tonny/Documents/iptables/rules.v4"
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

```

### 4. Enable the daemon
```bash
sudo systemctl daemon-reload
sudo systemctl enable iptables-restore.service
sudo systemctl start iptables-restore.service
sudo systemctl status iptables-restore.service
```

### 5. Other things that maybe made it worked but I did not noticed changes:

#### 5.1 Enable masquerade with firewalld
```bash
sudo firewall-cmd --zone=nm-shared --add-masquerade --permanent
sudo firewall-cmd --reload
```
#### 5.2 Allow forwarding (all traffic) for the hotspot zone
```bash
sudo firewall-cmd --zone=nm-shared --add-forward --permanent
sudo firewall-cmd --reload
```
#### 5.3 Check DNS
```bash
sudo firewall-cmd --zone=nm-shared --add-service=dns --permanent
sudo firewall-cmd --reload
```

#### 5.4 Confirm IP forwarding
If 1 is good
```bash
cat /proc/sys/net/ipv4/ip_forward
```

### 6. Docker
This is probably not needed but I read on forums that maybe docker was responsible for the problem

#### 6.1 Tell docker not to manage iptables
```bash
sudo vi /etc/docker/daemon.json
```

#### 6.2 Add the following
```bash 
{
  "iptables": false
}
```
#### 6.3 Then restart Docker
```bash
sudo systemctl restart docker
sudo systemctl restart NetworkManager
```
