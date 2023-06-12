+++
title = "Wire GUARD(?) simple setup cheat sheet"
date = "2023-06-11"
+++

Just a simple cheat sheet for a quick(**only CLI**) wireguard setup

### Generating Keys

You need to generate a key pair for both nodes (server and client).

#### Private Key:
```bash
wg genkey | sudo tee /etc/wireguard/private.key ## Can be changed to whathever directory you want
chmod go= /etc/wireguard/private.key ## Should match the previous directory
```

#### Public Key:

This one you can derive from the private, so I wont store it in a file, but you can if you want

```bash
cat /etc/wireguard/private.key | wg pubkey | sudo tee
```

### Node settings file

Then you need to make a file for the connection on the node(server), usually it stays on `/etc/wireguard/wg0.conf` and it's like this:

```ini
[Interface]
PrivateKey = *server private key*
Address = 10.8.0.1/24 *here you can pick any private IP range*
ListenPort = 51820 *your port, this is the default, but you can change it to whathever you want*
PostUp =  iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE; iptables -A INPUT -p udp -m udp --dport 51820 -j ACCEPT; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT;
# This PostUp line is (as I understand, and I don't know a lot about iptables) so you can route the traffic that comes from the clients, so they can access the internet through the server (like a "normal"/comercial vpn)

# Also, if you're going to use it, remember to change the network interface and the ip range if needed

[Peer]
PublicKey = *client public key*
AllowedIPs = 10.8.0.3/32 *clients ip*
```

Then you need another config file for the node(client):

```ini
[Interface]
PrivateKey = *client private key*
Address = 10.8.0.3/32 *client ip*
DNS = 9.9.9.9 *DNS Server, I usually go with Quad9, but again you can pick whathever*

[Peer]
PublicKey = *server public key*
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = *server ip and port, like this: ip:port*
PersistentKeepalive = 0 *if you want the connection to always stay up*
```

PS: Here we're separating client and server, but to wireguard there's only nodes, the difference, is that certain nodes(like our server node) can relay traffic.

### Starting up the connection

#### Server

On the server(node), you can use a wireguard tool:

```bash
wg-quick up wg0 ## or whathever the name for your config is
```

#### Client

On the client(node) there's a [bunch of GUIs](https://www.wireguard.com/install/) you can use, but if you want you can just use the CLI.

When I find time, I'll figure it out how to add images here properly (￣▽￣*)ゞ

