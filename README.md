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
* `sudo cp /lib/systemd/system/ocserv.service /etc/systemd/system/ocserv.service`
* `sudo nano /etc/systemd/system/ocserv.service`
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
PASTE IT AT THE END! I KNOW THERE WOULD BE A DUPLICATE IN FILE!
I'M WISE! TRUST ME!
```
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
-F
-A POSTROUTING -o ens160 -j MASQUERADE

# End each table with the 'COMMIT' line or these rules won't be processed
COMMIT
```
* `sudo crontab -e`
```
@daily certbot renew --quiet && systemctl restart ocserv -> add this
```


# ocserv CA (optional)
* `sudo apt install gnutls-bin`
* `sudo mkdir /etc/ocserv/ssl/ && cd /etc/ocserv/ssl/`
* `sudo certtool --generate-privkey --outfile ca-privkey.pem`
* `sudo nano ca-cert.cfg`
add these line:
```
# X.509 Certificate options

# The organization of the subject.
organization = "yourVPNdomain.com"

# The common name of the certificate owner.
cn = "yourVPNdomain.com"

# The serial number of the certificate.
serial = 001

# In how many days, counting from today, this certificate will expire. Use -1 if there is no expiration date.
expiration_days = -1

# Whether this is a CA certificate or not
ca

# Whether this certificate will be used to sign data
signing_key

# Whether this key will be used to sign other certificates.
cert_signing_key

# Whether this key will be used to sign CRLs.
crl_signing_key
```
* `sudo certtool --generate-self-signed --load-privkey ca-privkey.pem --template ca-cert.cfg --outfile ca-cert.pem`
* `sudo certtool --generate-privkey --outfile client-privkey.pem`

for every user do this:

* `sudo nano client-cert.cfg`
```
# X.509 Certificate options
# The organization of the subject.
organization = "yourVPNdomain.com"

# The common name of the certificate owner.
cn = "username"

# A user id of the certificate owner.
uid = "username"

# In how many days, counting from today, this certificate will expire. Use -1 if there is no expiration date.
expiration_days = 3650

# Whether this certificate will be used for a TLS server
tls_www_client

# Whether this certificate will be used to sign data
signing_key

# Whether this certificate will be used to encrypt data (needed
# in TLS RSA ciphersuites). Note that it is preferred to use different
# keys for encryption and signing.
encryption_key
```
* `sudo certtool --generate-certificate --load-privkey client-privkey.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-privkey.pem --template client-cert.cfg --outfile client-cert.pem`

foe AES:
* `sudo certtool --to-p12 --load-privkey client-privkey.pem --load-certificate client-cert.pem --pkcs-cipher aes-256 --outfile client.p12 --outder`

for 3DES:
* `sudo certtool --to-p12 --load-privkey client-privkey.pem --load-certificate client-cert.pem --pkcs-cipher 3des-pkcs12 --outfile ios-client.p12 --outder`
* `sudo nano /etc/ocserv/ocserv.conf`
```
auth = "plain[passwd=/etc/ocserv/ocpasswd]" -> change to -> enable-auth = "plain[passwd=/etc/ocserv/ocpasswd]"
auth = "certificate" -> uncomment
ca-cert = /etc/ssl/certs/ssl-cert-snakeoil.pem -> chnage to -> ca-cert = /etc/ocserv/ssl/ca-cert.pem
```
* `sudo systemctl restart ocserv`
