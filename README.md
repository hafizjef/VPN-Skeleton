# VPN-Skeleton
Networking related skeleton config for fast deployments.
_This config examples are written for Ubuntu, some changes are needed for other systems_

## OpenVPN
- Install OpenVPN and easy-rsa
- Copy `server.conf` to `/etc/openvpn/server.conf` and make your adjustment 
- Enable packet forwarding 

```bash
$ sudo apt-get install openvpn easy-rsa
# echo 1 > /proc/sys/net/ipv4/ip_forward
```

- Edit `/etc/sysctl.conf` to make packet forwarding permanent

```bash
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```

### Creating a Certificate Authority and Server-Side Certificate & Key
- First copy over the Easy-RSA generation scripts.
- Then make the key storage directory.
- Easy-RSA has a variables file we can edit to create certificates exclusive to 
our person, business, or whatever entity we choose. 
This information is copied to the certificates and keys, and will help identify 
the keys later.

```bash
cp -r /usr/share/easy-rsa/ /etc/openvpn
mkdir /etc/openvpn/easy-rsa/keys
vim /etc/openvpn/easy-rsa/vars
```

- Change the file accordingly

```bash
export KEY_COUNTRY="US"
export KEY_PROVINCE="TX"
export KEY_CITY="Dallas"
export KEY_ORG="Company Name"
export KEY_EMAIL="admin@company.org"
export KEY_OU="MYOrganizationalUnit"

export KEY_NAME="server"
```

- Generate the Diffie-Hellman parameters; this can take several minutes.

```bash
openssl dhparam -out /etc/openvpn/dh2048.pem 2048
```

- Change working dir & initialize the PKI
- Generate a Certificate and Key for the Server

```bash
source ./vars
./clean-all
./build-ca

./build-key-server server
```

- Move the Server cert and key

```
cp /etc/openvpn/easy-rsa/keys/{server.crt,server.key,ca.crt} /etc/openvpn
```

- Add & save iptables rules

```bash
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
iptables-save > /etc/network/iptables.rules
```