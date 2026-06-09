# 🚀 Remote Gaming Over Internet (Ultra-Low Latency Setup)

This guide addresses the challenge of high-performance remote gaming over the internet, enabling you to access your gaming rig from any location with ultra-low latency. The specific applications and tools required for this setup are detailed in the subsequent steps.

### 📊 Performance Comparison

| Option | Solution | Latency | Complexity | Security | Cost | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
|[1](#-option-1-port-forwarding)| **Public/Static IP + Port Forwarding** | ⭐⭐⭐⭐⭐ (Lowest) | Medium | ⚠️ Low | Low/Med | Direct connection, exposes IP/Ports. |
|[2](#️-option-2-vpn--tunnel)| **Public/Static IP + Tailscale** | ⭐⭐⭐⭐ (Low) | Easy | ✅ High | Low | Perfect mix of speed and security. |
|[3](#-option-3-using-a-vps-as-a-relay)| **Tailscale + VPS (Self-hosted Relay)** | ⭐⭐⭐⭐ (Low) | High | ✅ High | Medium | Best for CGNAT/Restricted networks. |
|[4](#-option-4-tailscale-only-and-no-public-ip)| **Tailscale Only (No Public IP)** | ⭐⭐ (Variable) | Very Easy | ✅ High | Free | Easiest, but may lag if relaying. |


---

## Host PC Setup

First you need a PC to host your gaming session. Then, pick a host and client program below:

<details>
<summary><b>🌙 Moonlight / Sunshine (Recommended)</b></summary>

**Pros:**

- Moonlight (Client) app supports many platforms: Windows, Linux, Android (TV), iOS, etc.
- Open-source and free

**Folk version:**
- I use **Apollo** and **Artemis** — they have better touch support for tablets, and support to create virtual screen on client app.<br>
   The setup is similar to Sunshine but Client App is currently only available on Android. You can still use moonlight app to connect.

</details>

<details>
<summary><b>🎮 Parsec</b></summary>

**Pros:**

- Good latency
- Has virtual screen support
- Friend sharing feature

**Cons:**

- I had issues with fullscreen gaming on Android before (don't know if they fixed this)
- Requires account registration and authentication

</details>

<details>
<summary><b>🔫 Other Providers</b></summary>

- If you know any, please tell. The above are currently the best for me.
- Chrome Remote Desktop: One time setup and stable but performance sometimes shuttering. I recommend you install this parallel because this may help you check and connect in case you got issue with other app and forgot to turn off VPN (this can connect on any network).
- Reemo: I did not tested this but seem require subscription for better use.

</details>

### Try First Connection

In case both devices are on the **same network**, you will be able to connect to your PC after the above installation.

Now when you go outside and connect to a different network like **4G/5G (mobile data)** on phone or **coffee shop's WiFi**, you will be unable to connect and the following issues occur:

- **Parsec:** Error 6023 / 6024
- **Moonlight:** Your PC will show as offline


<details>
<summary><b>❓ What happened? — Why can't I connect from a different network?</b></summary>

### The Problem: NAT & Private Networks

When both your host PC and client device are on the **same local network** (e.g., same home WiFi), they can discover and reach each other directly using **private IP addresses** (like `192.168.x.x`). Everything works out of the box.

However, when you leave your home network and connect via **mobile data** or a **public WiFi**, your client device is now on a **completely different network** — separated by the internet. Here's what goes wrong:

1. **Your host PC is behind NAT (Network Address Translation):**
   - Your home router assigns a private IP to your PC (e.g., `192.168.1.100`), which is only reachable **inside** your home network.
   - The router translates outgoing connections using its **public IP**, but **blocks all unsolicited incoming connections** from the internet by default.

2. **No direct path to your PC:**
   - From the outside, your host PC's private IP is invisible. The only visible address is your router's public IP, and the router has no rule to forward incoming traffic to your PC.
   - This is why **Moonlight** shows your PC as offline — it simply cannot reach it.
   - This is why **Parsec** throws error 6023/6024 — the connection attempt times out or is refused.

3. **No port forwarding configured:**
   - Remote gaming protocols (like Sunshine/Parsec) listen on specific ports. Without port forwarding on your router, incoming traffic on those ports is dropped.

</details>

### 👉 The Solution (Next Steps)

To make your host PC reachable from the internet, you need one of these approaches:

- **Port Forwarding:** Manually open the required ports on your home router and forward them to your PC. (Simple but exposes ports to the internet)
- **VPN / Tunnel (e.g., Tailscale, ZeroTier):** Create a virtual private network that connects your devices as if they were on the same LAN. (Recommended — more secure, no port forwarding needed)
- **Reverse Proxy / Relay Server:** Use a middleman server to bridge the connection. (More complex setup)

These solutions will be covered in the next section.

## ✅ Check if you have a Public IP

This step will determine your gaming performance. It is the best to have a public IP.

- Go to [ping.eu](https://ping.eu/ping/) and note your public IP.
- Use a mobile device on 4G/5G (not on your home WiFi) and try to ping that IP.
- If it shows `100% packet loss`, your IP might be behind CGNAT or your router blocks ICMP.

<details>
<summary><b>How to get a Public IP</b></summary>

- Contact your ISP: Ask them to disable CGNAT or provide a "Public IP for camera access". Usually, they grant this within an hour.
- Buy a Static IP: More expensive, but your IP will never change.
</details>


#### If you don't have public IP, don't worry, go to next section to fix this

## 🚀 Option 1: Port Forwarding

The method providing the best latency and reliability if you have a **Public IP**.

**Pros:**
- Lowest possible latency (Direct connection).
- No third-party relay needed.

**Cons:**
- Requires a Public IP (not CGNAT).
- IP changes when the router restarts (unless using DDNS).
- Security risk: Your IP and ports are exposed to the internet.


### Setup Port Forwarding
- Access your main router's settings.
- Ensure your PC has a static local IP (e.g., `192.168.1.100`).
- Forward the following ports to your PC's local IP:
  - **Moonlight:** TCP 47984, 47989, 47990, 48010 | UDP 47998, 47999, 48000, 48002, 48010
  - **Parsec:** UDP 8000-8011 (Default, can be customized)

After this, you can connect to your PC from anywhere.
But as I said before, except static IP,the IP may change after router reboot.
To fix this, find a ddns provider and then you can connect to the PC via a domain like: `mypc.ddns.com`


### If you  have public IP but can't or don't want to change router's config
In this case, you can refer to section IV to install a tunnel.<br>
Note that you **don't need a VPS/ relay server** to reduce latency because the public IP did the best.


## ✈️ Option 2: VPN / Tunnel

If you can't get a Public IP or don't want to mess with router settings, use a Mesh VPN like **Tailscale**.

### Tailscale Setup
Tailscale creates a virtual network where all your devices can see each other as if they were on the same LAN, even across the internet.

1. [Install Tailscale](https://tailscale.com/kb/1347/installation) on both the **Host PC** and **Client device**.
2. Log in with the same account on both.
3. Note the Tailscale IP of your Host PC (e.g., `100.99.1.1`).

#### Configuring Parsec for Tailscale:
Parsec often needs a nudge to use the VPN IP:
1. Exit Parsec completely.
2. Open `%AppData%\Parsec\config.json`.
3. Add the Tailscale IP to `app_custom_address`:
   ```json
   "app_custom_address": "100.99.1.1"
   ```
4. Restart Parsec. This fixes the **6023 error**.

#### Configuring Moonlight for Tailscale:
- Simply "Add PC" manually in the Moonlight app using the Host's Tailscale IP (e.g., `100.99.1.1`).

---

## 🚁 Option 3: Using a VPS as a Relay

Sometimes Tailscale's default relays (DERP) are slow or far away. By renting a cheap VPS in your city, you can create your own high-speed relay.
<br>
❌ If you have **public IP**, please skip this section because there may no more performance gain.

### 1. Rent a VPS
- **Location:** Crucial! Pick a provider with a data center in your city or country.
- **Specs:** Lowest tier is fine (1 vCPU, 1GB RAM, 100Mbps+).
- **OS:** Ubuntu is recommended.

### 2. VPS Setup
Connect to your VPS via SSH and run:
```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

### 3. Enable Port Forwarding on VPS
We want the VPS to receive traffic and forward it to your Host PC via Tailscale.

**Enable IP Forwarding:**
```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

**Install Firewall Tools:**
```bash
sudo apt install iptables-persistent -y
```

**Forward Ports (Example for Parsec on Port 9000):**
Replace `100.99.1.1` with your Host PC's Tailscale IP.
```bash
sudo iptables -t nat -A PREROUTING -p udp --dport 9000 -j DNAT --to-destination 100.99.1.1:9000
```

**Forward Ports for Moonlight:**
```bash
# TCP
for port in 47984 47989 47990 48010; do
  sudo iptables -t nat -A PREROUTING -p tcp --dport $port -j DNAT --to-destination 100.99.1.1:$port
done
# UDP
for port in 47998 47999 48000 48002 48010; do
  sudo iptables -t nat -A PREROUTING -p udp --dport $port -j DNAT --to-destination 100.99.1.1:$port
done
```

**Save Rules:**
```bash
sudo netfilter-persistent save
```

### 4. Final Configuration
- **NOTE:** the provider usually give you a public IP and it **will not change**
- **Parsec:** Set "Host Start Port" to `9000` on the Host PC. In `config.json`, change `app_custom_address` to the **VPS's Tailscale IP** or VPS's Public IP
- **Moonlight:** Add the **VPS's IP** in the Moonlight app.

### Why use a VPS? (Performance Comparison)

Without a VPS, if you are far from a Tailscale DERP server, your latency might look like this:
- **Host PC <-> Client PC:** ~100ms to 200ms (High latency, unplayable)

With a VPS located in your city:
- **Host PC <-> VPS:** 4ms
- **VPS <-> Client PC:** 4ms
- **Total Latency:** ~8ms to 15ms (Feels like local gaming!)

---

## 🚲 Option 4: Tailscale only and no public IP
Based on the country, the latency may be high (100ms on my country).
You are still able to connect but the experiment is not as good as other Option.

## 💡 Important Tips & Troubleshooting

- **Tailscale Ping:** You can test the connection between your devices using `tailscale ping <ip-address>` to see if they are connecting directly or via a relay.
- **Firewall:** If it still doesn't work, ensure your VPS provider's dashboard (e.g., AWS, DigitalOcean, Oracle Cloud) also has the ports open in their "Security Groups" or "Networking Firewall" settings.
- **Parsec Reset:** If you want to go back to a normal connection without Tailscale/VPS, remember to remove the `app_custom_address` line from your `config.json`.
- **Bandwidth:** Ensure your VPS has enough bandwidth. 1080p 60fps gaming typically requires 20-30Mbps.

## How to turn your PC on when you are not home

- Buy a module that support
- Turn on feature wake on AC /power on BIOS and use a smart plug
- Wake on lan with port forwarding (risky)
- Wake on lan use a local modules

---
~ Good luck and happy gaming! 🚀
