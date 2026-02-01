# How to Setup Parsec and Moonlight Using Tailscale
# And Use a VPS as Relay Server to Reduce Latency

---

### This guide will help you to:
- Resolve 6023/6024 error on Parsec
- Setup port forwarding in a VPS
- Setup Tailscale to work with Parsec and Moonlight
- Open ports without modifying router's config

### When do you need/follow this guide:
- Setup your PC as host for low latency game streaming. Note that this guide is good when you go outside of your network/home
- You can't setup Port Forwarding
- You're not the owner of the network
- You don't have the permission to config the network
- You can PAY about 1-2$ per month for renting a VPS
- Your network IP address is not public (you can check it [here](https://ping.eu/ping/))


Usually, your internet provider will not give you a public IP address. But some providers can grant you one if you contact them.

---

## I. Setting up

### Install Tailscale, Moonlight and/or Parsec
- [Install Moonlight](https://github.com/moonlight-stream/moonlight-docs/wiki/Setup-Guide)
- [Install Parsec](https://parsec.app/downloads)

Install a program to create a private network and connect your devices without public IP
- [Install Tailscale](https://tailscale.com/kb/1347/installation) (I'm using it in this guide)
- [Install Zerotier](https://www.zerotier.com/download/)

Install these program on both your host and client PC.

### Now, you need to change the config file of Parsec on the host PC:
#### 1. Exit Parsec, this is required before changing the json file
#### 2. Open the json file:
   - Press Win+R
   - Insert: ```%AppData%```
   - Open "Parsec" folder
   - Add this to file "config.json":
   ```json
   "app_custom_address": {
       "value": "<ip-address>"
   },
   ```
   The "value" is the Tailscale IP address of the host PC. Example:
   ```json
   "app_custom_address": {
       "value": "100.64.0.3"
   },
   ```
#### 3. Save the file and start Parsec
   - NOTE: if you want to go back to normal Parsec connection without using Tailscale, you need to remove this config
   - From here, you can connect to the host PC with `Parsec` without 6023 error

### For Moonlight
You just need to add new PC using the Tailscale IP address of the Host PC. It may be detected automatically

---

From here, both Client and Host must connect to Tailscale.<br>
After done above steps, now you can connect to the host without any error but you may face the high latency problem.<br>
Now, it's time to setup the VPS as relay server to reduce the latency.

---

## II. Setup VPS
- You must rent a VPS (or some provider can give you a free trial)
- Your VPS must have a public IP address (usually have)
- Choose the provider that places their server in your country/city because this will give you the lowest ping
- Choose the lowest configuration because this doesn't need high performance and this will save your money. In my case, I choose 1 CPU core, 1 GB of RAM, 100 Mbps with unlimited data usage
- Choose Linux as Operating System. Ubuntu/Debian is recommended

If your friend/etc... home network have the public IP, you can use a TV box or some small linux machine (with linux) and make it as a VPS but you need to setup port forwarding in the Router. B)

#### 0. Test the ping between peers. This is my result:
```text
Host PC <-> Client PC = 100ms :(
Host PC <-> VPS       = 4ms
VPS     <-> Client PC = 4ms
```
=> So, If we can make the traffic through the vps, we can get a good ping.
#### 1. Now inside the VPS console:
- Update your VPS:
```bash
sudo apt update
sudo apt upgrade
```
- [Install Tailscale for Linux](https://tailscale.com/kb/1031/install-linux)
- Start Tailscale:
```bash
sudo tailscale up
```
- Now check the connection status:
```bash
sudo tailscale status
```
This is my result (I removed some information):
```text
100.64.0.5      ubuntu              linux -     // This is my VPS address
100.64.0.3      host-pc             windows -   // Host PC address
100.64.0.4      phone               android -   // Client PC address
```

---

#### 2. Setup port forwarding in VPS
Install `netfilter-persistent` to save your config if VPS shutdown/reboot:
```bash
sudo apt install netfilter-persistent -y
```

Install `iptables-persistent`:
```bash
sudo apt install iptables-persistent
sudo iptables-save > /etc/iptables/rules.v4
```

Enable port forward:
```bash
sudo nano /etc/sysctl.conf
```
Remove `#` in this line:
```text
net.ipv4.ip_forward=1
```
Press Ctrl+O -> Enter to save then press Ctrl+X

Apply:
```bash
sudo sysctl -p
```

- Now forward a port to Host PC, in my case, I use port 9000:
```bash
sudo iptables -t nat -A PREROUTING -p udp --dport 9000 -j DNAT --to-destination 100.64.0.3:9000
```
In here:
- "udp" is the protocol. Some cases it will be "tcp"
- "9000" is the port number
- "100.64.0.3" is the Host PC IP address used in Tailscale

### For Moonlight, you need to forward the following ports:
TCP 47984, 47989, 48010
UDP 47998, 47999, 48000, 48002, 48010

- Simply use following commands:
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 47984 -j DNAT --to-destination 100.64.0.3:47984
sudo iptables -t nat -A PREROUTING -p tcp --dport 47989 -j DNAT --to-destination 100.64.0.3:47989
sudo iptables -t nat -A PREROUTING -p tcp --dport 48010 -j DNAT --to-destination 100.64.0.3:48010
sudo iptables -t nat -A PREROUTING -p udp --dport 47998 -j DNAT --to-destination 100.64.0.3:47998
sudo iptables -t nat -A PREROUTING -p udp --dport 47999 -j DNAT --to-destination 100.64.0.3:47999
sudo iptables -t nat -A PREROUTING -p udp --dport 48000 -j DNAT --to-destination 100.64.0.3:48000
sudo iptables -t nat -A PREROUTING -p udp --dport 48002 -j DNAT --to-destination 100.64.0.3:48002
sudo iptables -t nat -A PREROUTING -p udp --dport 48010 -j DNAT --to-destination 100.64.0.3:48010
```

- Save and reload the config:
```bash
sudo netfilter-persistent save && sudo netfilter-persistent reload
```

- Check if rule exist:
```bash
sudo iptables-save
```

---

Now change the port number in Parsec:
- Host PC, change the "Host Start Port" to 9000
- Client PC, change "Client Port" to 9000
___
- Before starting connect, you MUST change the IP address in the Parsec config file to the IP of the VPS.
- For Moonlight, you MUST add new computer using the IP of the VPS.
- In my case, it is 100.64.0.5
- Or you can try using the VPS public IP address, with this, Client PC don't need to install Tailscale
- Start connect and check the ping.

---

Good luck!
