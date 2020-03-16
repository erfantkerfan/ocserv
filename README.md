# ocserv

* `sudo apt install ocserv`
* `sudo apt install software-properties-common`
* `sudo add-apt-repository ppa:certbot/certbot`
* `sudo apt update`
* `sudo apt upgrade`
* `sudo apt install certbot`
* `sudo certbot certonly --standalone --preferred-challenges http --agree-tos --email your-email-address -d yourVPNdomain.com`
* `sudo nano /etc/ocserv/ocserv.conf`
then do these:
```
auth = "pam[gid-min=1000]" -> comment
#auth = "plain[passwd=./sample.passwd]" -> change to -> auth = "plain[passwd=/etc/ocserv/ocpasswd]"
udp-port = 443 -> comment
tcp-port = 443 -> change to -> tcp-port = PortYouLikeToRun
server-cert = /etc/ssl/certs/ssl-cert-snakeoil.pem -> chnage to -> server-cert = /etc/letsencrypt/live/yourVPNdomain.com/fullchain.pem
server-key = /etc/ssl/private/ssl-cert-snakeoil.key -> change to -> server-key = /etc/letsencrypt/live/yourVPNdomain.com/privkey.pem
max-clients = 16 -> change to -> max-clients = 0
max-same-clients = 2 -> remeber for later of you want to change!
try-mtu-discovery = false -> chnage to -> try-mtu-discovery = true
default-domain = example.com -> change to -> default-domain = yourVPNdomain.com
ipv4-network = 192.168.1.0 -> change to -> ipv4-network = 10.10.10.0
#tunnel-all-dns = true -> uncomment
dns = 192.168.1.2 -> change to -> dns = 1.1.1.1
route = 10.10.10.0/255.255.255.0 -> comment
route = 192.168.0.0/255.255.0.0 -> comment
no-route = 192.168.5.0/255.255.255.0 -> comment
tls-priorities = "NORMAL:%SERVER_PRECEDENCE:%COMPAT:-VERS-SSL3.0" -> change to -> tls-priorities = "NORMAL:%SERVER_PRECEDENCE:%COMPAT:-VERS-SSL3.0:-VERS-TLS1.0:-VERS-TLS1.1"
```
save and restart VPN using
* `sudo systemctl restart ocserv`

sudo cp /lib/systemd/system/ocserv.service /etc/systemd/system/ocserv.service
sudo nano /etc/systemd/system/ocserv.service
```
Requires=ocserv.socket -> comment
Also=ocserv.socket -> comment
```
* `sudo systemctl daemon-reload`
* `sudo systemctl stop ocserv.socket`
* `sudo systemctl disable ocserv.socket`
* `sudo systemctl restart ocserv.service`
* `sudo ocpasswd -c /etc/ocserv/ocpasswd username`
* `sudo nano /etc/sysctl.conf`
```
net.ipv4.ip_forward=1 -> uncomment
net.core.default_qdisc=fq -> add to end
net.ipv4.tcp_congestion_control=bbr -> add to end
```
* `sudo sysctl -p`
* `sudo ufw allow 22
* `sudo nano /etc/default/ufw
```
DEFAULT_FORWARD_POLICY="DROP" -> change to -> DEFAULT_FORWARD_POLICY="ACCEPT"
```
with this command find the name of your network(interface):
* `ip addr`
* `sudo nano /etc/ufw/before.rules`
paste this at the end of the file and replace ens160 with your network(interface)
```
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o ens3 -j MASQUERADE

# End each table with the 'COMMIT' line or these rules won't be processed
COMMIT
```
* `sudo crontab -e`
```
@daily certbot renew --quiet && systemctl restart ocserv -> add this
```
