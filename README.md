# Access local webservices using VPS and WireGuard

What is the purpose of this project? There are different use cases but my motivation is to access my web services hosted inside my homelab LAN from the Internet via a VPS through a VPN tunnel. Therefore, we want to redirect the incoming traffic at the VPS to a reverse proxy in my LAN. Theoretically, this project should also enable you to access a webservice that is hosted behind a firewall that would normaly block your requests to that webservice.

When the term 'server' is mentioned in the following, it always means a VPS, while the term 'client' refers the computer on which the reverse proxy is deployed.

## Setup the WireGuard VPN tunnnel

Install WireGuard both on the server and on the client
```bash
sudo apt install wireguard
```
Once Wireguard is installed, generate a key pair on your server and your client
```bash
sudo wg genkey | tee privatekey | wg pubkey > publickey
```
After you have successfully created your key pair, you have to create a WireGuard config file on your server and your client. You can create this file in any directory you want, but by default WireGuard is looking inside ```/etc/wireguard``` so I recommend that you place your config files there. I will not get into the details on how you can configure WireGuard. If you want to learn more about configuring WireGuard, I recommend you to read the documentation mentioned in the resources. We will start with the configuration of our server
```bash
# server config
[Interface]
Address = 10.0.0.1/24
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o <network-interface> -j MASQUERADE;
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o <network-interface> -j MASQUERADE;
ListenPort = <port>
PrivateKey = <server-privatekey>
```
Please make sure that you change the above references (defined in '<>') according to your system settings. After you have created the config file of the server go ahead and create the config file of your client as following.
```bash
# client config
[Interface]
PrivateKey = <client-privatekey>
Address = 10.0.0.2/24

[Peer]
PublicKey = <server-public-key>
Endpoint = <public-ip-server>:<listen-port>
AllowedIPs = 10.0.0.1/24
PersistentKeepAlive = 30
```
Again make sure that you change the above references (defined in '<>') according to your system settings. Now that you have created the client's config file we can establish a connection. 

At first we start our WireGuard network interface on the server and the client using the following command
```bash
sudo wg-quick up <path-to-config-file> # e.g. /etc/wireguard/wg0.conf
```
Now your WireGuard network interfaces should be up and running, which you can check by typing the command ```sudo wg``` . Before you can actually use the WireGuard tunnel you have to add your client to your server's config file. Just type the following command
```
sudo wg set wg0 peer <client-publickey> allowed-ips 10.0.0.2/32
```
Alternatively, you can manually add the client to server's config file by adding the following lines to it.
```bash
# server config
[Peer]
PublicKey = <client-publickey>
AllowedIPs = 10.0.0.2/24
```

Now your client should be included in the server's config file. You can check this by using the command ```sudo wg showconfig wg0``` . You should see your client inside the ```[Peer]``` section of the config.

## Redirecting the traffic
Now that the tunnel has been successfully established, we have have to redirect the traffic from our server through the tunnel to our client. Therefore we just have to add new rules to ```iptables```. For information about ```iptables```, check out the tutorial mentioned in the resources.
```bash
iptables -t nat -A PREROUTING -i <network-interface> -p tcp --dport <server-port> -j DNAT --to-destination <client-ip>:<client-port>
iptables -t nat -A POSTROUTING -j MASQUERADE
```
### Resources
- iptables tutorial: https://www.karlrupp.net/en/computer/nat_tutorial
- Blog article: https://blog.cavelab.dev/2021/03/vps-wireguard-iptables/
- Wireguard quick start guide: https://www.wireguard.com/quickstart/
- Unoffical WireGuard Documentation: https://github.com/pirate/wireguard-docs
