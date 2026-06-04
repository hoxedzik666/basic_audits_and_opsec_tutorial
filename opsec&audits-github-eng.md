# Introduction in a few words...

Hey guys, it's me again, hah. In this tutorial series, we will primarily learn about "ethical" attack methods on your own creations, like web applications, which we'd more accurately call a Security Audit. And we'll start exactly with this process, because in these tutorials we want to focus mainly on efficiency and results. Eventually, we will reach a stage where we'll mostly be prompting Gemini, but chill the fuck out bro, first we are going to learn the basic auditing tools from scratch by hand...

### Why will AI even be important in this series?

==**Answering this very stupid question... Even though I really don't feel like it XD.**==

- **_Why should you be worse and slower than the rest of the rapidly developing market of AI agents and models?_**
    

**So it's two in one, because we'll also learn how to write cool prompts (starting with Gemini).**

**Of course, I don't need to explain that I will strictly be using a system from my beloved Linux family...**

**_I realize I'm in the minority when it comes to OS choices, but do I feel bad about it?_**

- _I'll just politely skip answering that question :P_
    

> For now, my base system is Debian 13. If you want, install Kali Linux or whatever Linux-family OS suits you...
> 
> - Keep in mind, though, that my package manager will be **APT**. So, if you install something like _Arch_, you'll have to use pacman for installing programs, updating the system, etc.
>     
> - _If you install Kali, my dear, which is based on Debian, your package manager will also be apt... I'm only saying this in case you forgot, because at this stage you already knew that, right?_
>     

**Alright, but for those who chose Ubuntu, Debian, or any other distro not designed for pentesters by default, I'm providing the tools we'll be installing for later use.**

Okay, so Mr. Handyman, we should probably start with...

> - an update?
>     
> - installing my favorite browser?
>     

_Bullshit, pardon my French. In most cases, you should start by modifying the sudoers file, and I'm telling you this now so you don't go digging through Google later and getting pissed off :P_



```bash
sudo nano /etc/sudoers

#enter your account password....
```

_Now find this line:_



```text
# User privilege specification
root    ALL=(ALL:ALL) ALL
```

and right underneath it, add:



```text
# User privilege specification
root    ALL=(ALL:ALL) ALL
your_account_name    ALL=(ALL:ALL) ALL
```

==**_You save it using the combination CTRL + X / y / enter :)_**==

**Okay, now you can finally proceed to update the package manager by typing:**



```bash
sudo apt update && sudo apt full-upgrade -y 

#admin account password 
#now we will create a bash script using nano that will install all the tools for us. Let's start by creating a folder 

cd /home/$USER/Documents 
&& mkdir scripts_sh && cd scripts_sh && nano tools.sh 

#now paste what's below into the tools.sh file, save it, and then follow the instructions. 
```

Here is the complete Bash installation script for **Debian 13 (Trixie)**, using the `apt` package manager.

The script automatically installs all required runtime environments (Java JRE for DirBuster, Ruby for WhatWeb, Perl for Nikto, Python for sqlmap/dirsearch, and Go for Nuclei), installs the tools, starts and configures the Tor service, and pairs it with Proxychains4.



```bash
#!/bin/bash

# Security tools installation script for Debian 13 (Trixie)
# Includes: nmap, sqlmap, nikto, nuclei, whatweb, dirbuster, ffuf, dirsearch, tor, proxychains4

# Abort script execution on any command failure
set -e

echo "[+] Updating system package lists..."
sudo apt update

echo "[+] Installing required dependencies and runtime environments..."
# default-jre  -> for DirBuster (Java)
# python3, python3-pip, python3-venv -> for sqlmap, dirsearch
# ruby         -> for WhatWeb
# perl         -> for Nikto
# golang-go    -> to compile and install Nuclei
# git, curl, wget, unzip -> helper tools for downloading resources
sudo apt install -y \
    apt-transport-https \
    curl \
    wget \
    git \
    unzip \
    python3 \
    python3-pip \
    python3-venv \
    default-jre \
    ruby \
    perl \
    golang-go

echo "[+] Installing tools available directly from apt repositories..."
# Note: The tool 'ffuz' from your prompt is actually 'ffuf' (Fuzz Faster U Fool)
sudo apt install -y \
    nmap \
    sqlmap \
    nikto \
    whatweb \
    dirbuster \
    ffuf \
    dirsearch \
    tor \
    proxychains4

echo "[+] Installing and compiling Nuclei using the Go environment..."
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# Install the latest version of Nuclei directly from the official Go repository
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Copy binary to system location for global access
if [ -f "$HOME/go/bin/nuclei" ]; then
    sudo cp "$HOME/go/bin/nuclei" /usr/local/bin/
    echo "[+] Nuclei installed successfully in /usr/local/bin/nuclei"
else
    echo "[!] Go install failed. Attempting to download precompiled binary from GitHub..."
    ARCH=$(uname -m)
    if [ "$ARCH" = "x86_64" ]; then ARCH_SUFFIX="amd64"; else ARCH_SUFFIX="386"; fi
    LATEST_NUCLEI=$(curl -s https://api.github.com/repos/projectdiscovery/nuclei/releases/latest | grep -oP '"browser_download_url": "\K[^"]+linux_'"$ARCH_SUFFIX"'\.zip' | head -n 1)
    wget -O nuclei.zip "$LATEST_NUCLEI"
    unzip -o nuclei.zip nuclei
    sudo mv nuclei /usr/local/bin/
    rm -f nuclei.zip
fi

echo "[+] Starting and enabling Tor service autostart..."
sudo systemctl enable tor
sudo systemctl start tor

echo "[+] Configuring Proxychains4 to work with the Tor network..."
# Tor listens by default as a SOCKS5 proxy on port 9050.
# We configure proxychains4 to route traffic through this port.
if [ -f /etc/proxychains4.conf ]; then
    # Remove default/old socks4 entries if they exist, and add current SOCKS5 for Tor
    if ! grep -q "socks5[[:space:]]*127.0.0.1[[:space:]]*9050" /etc/proxychains4.conf; then
        echo "socks5 127.0.0.1 9050" | sudo tee -a /etc/proxychains4.conf > /dev/null
        echo "[+] Added SOCKS5 (Tor) configuration to /etc/proxychains4.conf"
    fi
else
    echo "[!] File /etc/proxychains4.conf not found. Ensure the package was installed correctly."
fi

echo -e "\n[+] VERIFYING TOOL INSTALLATION:"
echo "--------------------------------------------------"
nmap --version | head -n 1 || echo "[-] Nmap: Error"
sqlmap --version | head -n 1 || echo "[-] Sqlmap: Error"
nikto -Version | head -n 1 || echo "[-] Nikto: Error"
nuclei -version | head -n 1 || echo "[-] Nuclei: Error"
whatweb --version | head -n 1 || echo "[-] Whatweb: Error"
ffuf -V || echo "[-] ffuf: Error"
dirsearch --version | head -n 1 || echo "[-] Dirsearch: Error"
tor --version | head -n 1 || echo "[-] Tor: Error"
proxychains4 -h 2>&1 | head -n 1 || echo "[-] Proxychains4: Error"
echo "[-] DirBuster (requires X11/Wayland GUI environment to launch)"
echo "--------------------------------------------------"
echo "[+] Installation completed successfully!"
```

### How to run the script:

1. Save the above code to a file, e.g., `install_tools.sh`.
    
2. Grant execution permissions:

```bash
chmod +x install_tools.sh
```

3. Run the script (required `sudo` privileges will be invoked inside the script):

```bash
./install_tools.sh
```

**Okay, let's also install Anonsurf, it might come in handy someday, right?**

## Installation and Operation of Debian Anonsurf (ParrotSec Anonsurf Port)

**Anonsurf** is a powerful tool ported from the ParrotSec distribution that allows you to anonymize the entire system network traffic using the TOR network and `iptables` rules. Additionally, the package includes the **Pandora** module, which takes care of wiping the RAM.

### Description of this repository

This version combines the `anonsurf` and `pandora` packages from ParrotSec into one, introducing a few important fixes:

- Uses DNS servers from **Private Internet Access** (instead of FrozenDNS).
    
- Contains fixes for users who do not use the `resolvconf` application.
    
- Removed unnecessary features, such as the Graphical User Interface (GUI) and running the Iceweasel browser in RAM.
    

### 1. Installation

Installation is incredibly simple because the repository includes a ready-made installation script. To install the tool on your Debian system, run the following commands in the terminal:

Bash

```bash
# Cloning the official repository
git clone https://github.com/anoopmsivadas/debian-anonsurf.git

# Navigating to the downloaded project directory
cd debian-anonsurf

# Granting execution permissions to the installer
chmod +x installer.sh

# Running the installer with superuser (root) privileges
sudo ./installer.sh
```

After the installer finishes, both modules (`anonsurf` and `pandora`) will be fully ready to use.

### 2. Using Pandora (RAM Wiping)

**Pandora** automatically overwrites and wipes the RAM during system shutdown to prevent the recovery of sensitive data.

You can also trigger it manually at any time:

Bash

```bash
sudo pandora bomb
```

> ⚠️ **WARNING:** Running this command will instantly clear the entire system cache, which will result in the dropping of active tunnels and SSH sessions, among other things.

### 3. Using Anonsurf

**Anonsurf** routes all system traffic through the TOR network using IPTables rules.

> ⚠️ **IMPORTANT NOTE:** Do **NOT** run this tool as a system service (e.g., `sudo service anonsurf start`). Instead, run it directly using the command `sudo anonsurf $COMMAND`.

#### Basic commands:

- **Start anonymization:**

```bash
sudo anonsurf start
```

*Starts system-wide traffic tunneling through the TOR proxy using iptables rules.*

- **Stop anonymization:**

```bash
sudo anonsurf stop
```

*Restores original iptables settings and returns to a clean, direct internet connection.*

- **Restart service:**

```bash
sudo anonsurf restart
```

*Executes the stop procedure followed by the start procedure.*

- **Change identity (IP):**

```bash
sudo anonsurf change
```

*Changes your identity on the network by restarting the TOR service and fetching a new IP address.*

- **Check status:**

```bash
sudo anonsurf status
```

*Checks if AnonSurf is running correctly and if your traffic is securely tunneled.*

#### I2P network-related functions:

- **Start I2P services:**

```bash
sudo anonsurf starti2p
```

- **Stop I2P services:**

```bash
sudo anonsurf stopi2p
```

### 4. Let's test it out...

**Okay, now after typing into the terminal:**

```bash
anonsurf
```

![[anonsurf.png]]

**_We should get an effect like the one described above on how to start it, so I'll skip that, but while we're at it, I'll show you how to route Firefox browser traffic through the TOR network._** **_< yes, I know anonsurf routes all network traffic >_**

## Advanced Firefox Configuration for Privacy and Anonymity

Although system tools (like Anonsurf) route traffic at the OS level, the web browser itself is a massive source of data leaks without proper configuration and plugins. Below is a complete guide on how to turn standard Firefox into an armored, identity-protecting browser.

### 1. Manual Traffic Routing via Tor Network (SOCKS5 Proxy)

If you don't want to tunnel the entire system, but only the traffic from the Firefox browser itself, you can configure it to directly use the local Tor client (which by default listens on port `9050` as a SOCKS5 proxy):

1. Open the Firefox menu and go to **Settings** or type `about:preferences` in the address bar.
    
2. On the **General** tab, scroll all the way down to the **Network Settings** section and click the **Settings...** button.
    
3. Select the **Manual proxy configuration** option.
    
4. Find the **SOCKS Host** field and enter the IP address: `127.0.0.1` and port: `9050`.
    
5. Ensure that the **SOCKS v5** option below is selected.
    
6. **CRUCIAL STEP:** Check the **Proxy DNS when using SOCKS v5** box. If you don't do this, your browser will send DNS queries outside the Tor network, leading to an immediate leak of your queries!
    
7. Click **OK** to save changes.
    

### 2. Disabling WebRTC in Firefox

**WebRTC (Web Real-Time Communication)** is a technology built into browsers that allows for direct audio/video communication and P2P data transfer (e.g., on Discord, Zoom, or Google Meet) without intermediary servers.

Even though it's very useful, it poses a **huge privacy risk**, because it can bypass proxies, VPNs, and even the Tor network, querying your system directly for your local and external IP address using the STUN protocol.

#### How to completely disable WebRTC in Firefox:

1. Type `about:config` in the browser's address bar and hit Enter.
    
2. Accept the risk warning by clicking **Accept the Risk and Continue**.
    
3. In the search bar, type the phrase: `media.peerconnection.enabled`.
    
4. Double-click on the found item (or use the toggle button on the right) to change its value from **true** to **false**.
    
5. From now on, WebRTC is completely blocked and will not reveal your real IP.
    

### 3. Essential Privacy-Enhancing Add-ons

To install the following add-ons, go to the official extension store: **Firefox Add-ons** (`about:addons` -> "Find more add-ons" section at the very bottom).

Here is a list and brief description of the most important plugins you need to deploy:

- **NoScript Security Suite (NoJS)**
    
    - _Description:_ Blocks the execution of all JavaScript, Java, Flash, and other active content on websites. JavaScript is the most powerful weapon in the hands of tracking systems and hackers (it enables hardware profiling, browser exploitation, or reading system parameters). NoScript allows precise enabling of scripts only for trusted domains (whitelist).
        
- **CanvasBlocker**
    
    - _Description:_ Protects against an advanced tracking method called _Canvas Fingerprinting_ (HTML5 canvas fingerprinting). Instead of completely blocking graphic elements (which could break website layouts), CanvasBlocker generates fake noise or random data when attempting to read the unique properties of your graphics card and rendering engine. Because of this, you look like a completely new user to every tracker.
        
- **Privacy Badger**
    
    - _Description:_ An add-on created by the EFF (Electronic Frontier Foundation). It does not rely on simple, static blocklists. Instead, it analyzes the behavior of scripts on the sites you visit. If it detects that a script is tracking you across different websites without your consent, Privacy Badger will automatically block it from sending and receiving data.
        
- **User-Agent Switcher and Manager (User Agent Controller)**
    
    - _Description:_ Allows for easy spoofing of the `User-Agent` header. This header tells the server what operating system (e.g., Linux, Windows, macOS) and browser you are using. With this plugin, you can pretend you're browsing the web from an Android phone or an iPhone, which makes it harder to link your sessions and protects against profiling.
        
- **Nuke Anything / Nuke Data**
    
    - _Description:_ Allows you to remove any elements from a website (e.g., annoying popups, AdBlock blocking scripts, login forms, or tracking elements) directly from the context menu (right-click). Additionally, it makes it easier to quickly destroy session data associated with a selected element or the entire site.
        
- **Clear Cache (e.g., Clear Cache Button)**
    
    - _Description:_ Adds a simple button to the browser toolbar that allows you to completely clear the browser cache with one click. The cache stores website files, which can be used for so-called "cache tracking" (user identification based on previously saved image files or scripts).
        
- **Cookie Quick Manager**
    
    - _Description:_ An advanced tool for full control over cookies. It allows you to view, edit, delete, as well as export and import cookies for individual domains or globally. Ideal for investigating sessions, cleaning up traces after audits, and protecting against session hijacking.
        

### 4. Browser Configuration: No History, No Cookies, and No Telemetry

To ensure Firefox does not save any data locally and does not send diagnostic reports to developers (telemetry), go to **Settings** -> **Privacy & Security** and configure the following options:

#### History and Cookies:

- In the **History** section, under the _Firefox will:_ option, select **Use custom settings for history**.
    
- Uncheck the options:
    
    - _Remember browsing and download history_.
        
    - _Remember search and form history_.
        
- You can also check the **Always use private browsing mode** option – Firefox will then run in a mode where it leaves absolutely no traces after closing.
    
- Check the **Clear history when Firefox closes** option and in the settings next to it, check everything: cookies, cache, active logins, etc.
    
- In the **Cookies and Site Data** section, check the option **Delete cookies and site data when Firefox is closed**.
    

#### Blocking Telemetry (Sending data to the developer):

Scroll down to the **Firefox Data Collection and Use** section. **Uncheck** all the following options:

- _Allow Firefox to send technical and interaction data to Mozilla_.
    
- _Allow Firefox to install and run studies_.
    
- _Allow Firefox to send backlogged crash reports on your behalf_.
    

## Security Theory: Why are just proxies and VPNs not enough?

### 1. WebRTC and IP Leaks (WebRTC Leak)

As mentioned above, **WebRTC** is used for direct P2P communication. Normally, when a browser wants to connect to a server, the request goes through a configured proxy (e.g., Tor) or a VPN tunnel.

However, WebRTC works differently. To establish a direct connection with another user (who might be behind a NAT firewall), WebRTC sends special requests to **STUN** (Session Traversal Utilities for NAT) servers. These requests are sent via the **UDP** protocol directly from your operating system.

Because UDP requests in WebRTC technology are executed at the level of the system's low-level network sockets, they can query the system for the addresses assigned to all physical network cards (e.g., your local IP in the home network `192.168.1.X` and your real, public ISP IP). The STUN server sends this data back, and a JavaScript script embedded on the website can read it.

This is how a website finds out what your real IP address is, even though the VPN icon on your taskbar is glowing green, and HTTP traffic is going through Tor.

### 2. DNS and How You Can Get Caught (DNS Leak)

**DNS (Domain Name System)** is a system that translates domain names (e.g., `niebezpiecznik.pl`) into server IP addresses (`185.117.84.120`).

#### What is a DNS Leak?

When you connect to a VPN network, your network traffic is encrypted. However, the operating system may still send DNS resolution queries to your local Internet Service Provider (ISP) DNS servers instead of through the secure VPN tunnel.

#### How can you get caught because of this?

1. **The Internet Service Provider (ISP) sees everything:** Your ISP logs every DNS query sent from your home IP. Even if the content of the communication with the website itself is encrypted or goes through a VPN, your provider knows exactly which domains you asked for and at what times.
    
2. **Traffic Correlation:** Let's assume you access a hidden site via Tor or a VPN. If at the same time your system sends an unencrypted DNS query about this domain to your ISP's DNS server, observers can easily link this query (associated with your real name and home address) with the traffic that appeared a moment later on the target server from the Tor/VPN exit node address.
    
3. **DNS Poisoning (DNS Spoofing/Hijacking):** By using unsecured DNS servers, you are exposed to someone substituting a fake IP address for the requested domain, directing you to a phishing site (e.g., a fake bank login panel).
    

### 3. Canvas Fingerprinting and the Power of JavaScript

#### What are Canvases?

The `<canvas>` element in the HTML5 standard is used to generate graphics on the fly using JS scripts.

When you visit a site utilizing **Canvas Fingerprinting**, a script instructs your browser to render a hidden (invisible to the eye) image containing text of a specific font, shading, textures, and WebGL lighting effects.

Because graphic rendering depends on:

- The model and manufacturer of your graphics card (GPU),
    
- The version of graphics drivers in your system,
    
- The set of installed system fonts,
    
- The browser's rendering engine,
    
- The operating system version,
    

that same image will be rendered at the level of individual pixels in a slightly different way on different computers. The browser then converts this image into Base64 format and generates a unique hash from it. This hash acts as a **digital fingerprint**. Since the probability of two people having an identical hardware/software configuration is minimal, a tracker can flawlessly identify you across different websites, even if you change your IP, clear your cookies, and use private mode.

#### How much information do we leave behind with JavaScript enabled?

JavaScript allows websites to read almost the full specification of your computer. Without your knowledge, a site can retrieve:

- The number of processor cores (`navigator.hardwareConcurrency`),
    
- The amount of RAM (in some browsers),
    
- The exact battery charge level and information on whether the computer is plugged in,
    
- A full list of installed plugins and fonts,
    
- The exact screen resolution, color depth, and number of monitors,
    
- Sensor data (e.g., accelerometer in a phone),
    
- Behavioral data: the precise dynamics of mouse movements and typing speed on the keyboard (which allows for the unambiguous identification of a human, not just a machine).
    

#### Why is completely disabling JavaScript practically impossible?

Theoretically, the simplest way to maintain anonymity is to completely disable JS in the browser. In practice, however, **the modern internet doesn't work without JavaScript**.

1. **SPA Applications (Single Page Applications):** Most modern portals (e.g., Gmail, Twitter, banking panels, reservation systems) are applications written in frameworks like React, Angular, or Vue. The server sends an empty HTML document to the browser, which is entirely built and rendered only by JavaScript on your computer. Without JS, you will only see a blank white page.
    
2. **Authorization and Sessions:** Login mechanisms, security token generation (JWT), form handling, and bot protections (e.g., reCAPTCHA, Cloudflare) require active JavaScript. Without it, you won't log into any account.
    
3. **Dynamic Content:** Without JS, video players, interactive maps, live chats, or dynamically loaded comments won't run.
    

Completely disabling JS takes us back to the internet of the 90s. Therefore, instead of turning it off completely, a hybrid approach is used: **NoScript** to block scripts on suspicious sites, and masking/noise-adding tools (like **CanvasBlocker** or the built-in tracking protection mechanisms in the Tor Browser), which allow JS to run but feed tracking scripts fake, normalized data.

**Okay, I actually have a few examples that will illustrate this nicely for you.**

![[Zrzut ekranu_20260604_181325.png]]

> **This site you see is used to test my custom WAF. It's a simple debugger I wrote for testing, but the script has functions to detect if the user has JS, webrtc, canvases, etc. enabled.**

> This time, by sheer luck, the script leaked an IP via WebRTC that is the same as the one set on my VPN, but that's also an advantage of browser add-ons and settings, seriously :) whatever

![[pseudo-prawilna-hakerka/img/2.png]]

**Okay, let's deal with the canvases. To access this panel, find the CanvasBlocker icon on the toolbar, click it, and select settings. Now, as an example, I will block canvas creation... (this is not a good solution...) - we rather set it to fake it, this is just an example.**

![[pseudo-prawilna-hakerka/img/3.png]]

**_Voila, the fingerprint is no longer generating._**

![[4.png]]

**Alright, time for JS handling. We select the NoJS icon and click here where I pointed to block the execution of all scripts the browser encounters.**

![[5.png]]

**_The effect? Beautiful, right? But in reality, most sites won't let us in at all :P_**

![[6.png]]

**AHA! I want to point out that what we just did with Firefox does not guarantee full anonymity... it helps a lot, but it is not a guarantee of security!**

**_We should probably also secure the operating system, install something like fail2ban etc..._**

- _okay, let's get right on that..._
    

### 1. Firewall and Protection: UFW + Fail2Ban

First off, we need to build a wall on the ports and kill the bots that will be knocking on your system. Even if you have beautifully configured SSH logins using ED25519 keys (which is the only right way to go), Chinese and Russian scanners will be constantly clogging your logs trying to break into root.

We install the package:



```bash
sudo apt update && sudo apt install ufw fail2ban -y
```

**UFW (Uncomplicated Firewall) Configuration:** We close everything from the outside, open what's necessary. **IMPORTANT:** Before you enable the firewall, make sure you allowed the SSH port, otherwise you'll cut off access to your own machine!



```bash
# Set default rules: nothing comes in, everything goes out
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH port (default is 22, if you changed it - provide yours)
sudo ufw allow ssh

# Enable the firewall
sudo ufw enable
```

**Fail2Ban Configuration:** Fail2ban scans system logs and applies bans (via iptables/ufw) to IP addresses that have provided incorrect passwords too many times or behave suspiciously.

1. We copy the main config file (never edit `jail.conf` because an update will overwrite it):
    



```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

2. Scroll down to the `[sshd]` section and make sure it looks something like this (change `enabled` to `true`):
    



```toml
[sshd]
enabled = true
port    = ssh
filter  = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```

_(Bantime = 3600 is an hour in solitary for every bot after 3 failed attempts. Save CTRL+X / Y / Enter)._

3. Restart service:
    



```bash
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
```

### 2. Sealing DNS (DNS over TLS via systemd-resolved)

This is absolutely crucial. Standard DNS flies across the network in **plaintext**. Your ISP might not know what you're downloading, but they can clearly see the domain names your system resolves (e.g., `nmap.org`, `kali.org`, or audit targets).

We'll configure the built-in Debian daemon `systemd-resolved` so that all DNS traffic from the system goes through an encrypted TLS tunnel (DNS over TLS - DoT). We'll use Quad9 resolvers (9.9.9.9 and 149.112.112.112) – it's a great privacy-focused foundation based in Switzerland, and most importantly, unlike popular commercial solutions, their servers don't load aggressive blocklists or verifications that could make life miserable when scanning web apps.

1. Edit the resolver file:
    



```bash
sudo nano /etc/systemd/resolved.conf
```

2. Uncomment (remove `#`) and modify the lines in the `[Resolve]` section so they look exactly like this:
    



```toml
[Resolve]
DNS=9.9.9.9 149.112.112.112
#FallbackDNS=
Domains=~.
DNSSEC=yes
DNSOverTLS=yes
#MulticastDNS=yes
#LLMNR=yes
#Cache=yes
```

3. Save and restart the service:
    



```bash
sudo systemctl restart systemd-resolved
sudo systemctl enable systemd-resolved
```

4. **Verification:** Type the command `resolvectl status`. If you see `DNSOverTLS: yes` in the section for your interface (e.g., `eth0`), then you are sealed. Your DNS is now encrypted at the level of Debian itself.
    

### 3. Auditd (The System Snitch)

`Auditd` is a powerful tool that allows you to track every operation at the Linux kernel level. Who, when, and with what touched a given file or changed permissions. When testing your own leaky apps for an audit, it's worth having control over what processes are up to in the file system.

1. Installation:
    



```bash
sudo apt install auditd audispd-plugins -y
```

2. Rules configuration takes place in `/etc/audit/rules.d/audit.rules`. You can set up monitoring for modification attempts of key files, for example:
    



```bash
sudo nano /etc/audit/rules.d/audit.rules
```

Add this at the bottom if you want to know, for instance, when any web script (or user) tries to touch your password or log files:



```text
-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k shadow_changes
-w /var/log/auth.log -p wa -k auth_logs_accessed
```

_(The `-w` flag is the path, `-p wa` is write/append permissions, and `-k` is your tag, by which you can easily find it in the logs)._

3. Restart:
    



```bash
sudo systemctl restart auditd
```

If you want to check what's happening with your system regarding the added rules, just type:



```bash
sudo ausearch -k passwd_changes
```

### 4. Macchanger (Hardware Spoofing)

A useful tool from the pentester's arsenal if you are attacking/auditing something on a LAN or connecting from a foreign access point. It spoofs the physical MAC address of your network card.



```bash
sudo apt install macchanger -y
```

_(During installation, it will ask if it should change the MAC automatically every time you plug in a cable/turn on the interface. I recommend selecting **NO**, so you do it fully consciously)._

To change the MAC to a fully random one on an interface like `eth0` (first you have to bring the interface down):



```bash
sudo ip link set dev eth0 down
sudo macchanger -r eth0
sudo ip link set dev eth0 up
```

_Note: If you're doing this on a server to which you only have remote access (SSH), ignore macchanger, because after bringing the interface down, you will lose contact with it!_

**I would definitely buy something like Mullvad (VPN) on top of this, which by the way is very easy to pay for fully anonymously ;) using cryptocurrencies.**

[https://mullvad.net/en](https://mullvad.net/en)

## Addendum: Advanced Traffic Routing – VPN over Tor vs Tor over VPN

Hey! Since I mentioned **Mullvad** in the previous section, it's time to step up to a higher level of network initiation. You will often hear concepts like **VPN over Tor** and **Tor over VPN** in the OPSEC world. Although they sound similar, they represent completely different approaches to routing traffic, have radically different applications, and configuring them (especially the first "trick") requires iron discipline and proper rules on the firewall (`iptables`/`ufw`).

Let's break this down into primary factors, without beating around the bush.

### 1. VPN over Tor – What is this "trick"?

This is one of the most twisted network configurations. Traffic from your computer flows as follows: `Your system (Apps) ➔ VPN Tunnel ➔ Local SOCKS5 (Tor) ➔ Tor Network (3 nodes) ➔ VPN Server ➔ Internet`

In practice, this means **you are establishing a VPN tunnel inside the Tor network**. Your VPN client connects to the VPN server not directly, but through the local Tor SOCKS5 proxy port (`127.0.0.1:9050`).

#### What does this trick involve and why is it brilliant?

1. **Bypassing Tor Bans:** This is the biggest advantage. A huge part of the internet (Cloudflare, VOD services, forums, stores) blocks or pesters users coming from public IP addresses of Tor Exit Nodes with CAPTCHAs. In this configuration, the target site sees the **VPN server's IP**, not Tor's! You browse the internet anonymously through Tor, but with the hassle-free speed and access of a traditional VPN.
    
2. **Full anonymity from the VPN provider:** Even if your VPN provider logs traffic or gets seized by authorities, they have no idea who you are. They only see that the connection to their server is coming from a random Tor exit node. If you additionally created an account through Tor and paid for it anonymously (e.g., Monero or cash), you are a ghost.
    
3. **Encryption against malicious Tor exit nodes:** Traffic leaving the Tor network and entering the VPN is fully encrypted with the VPN key. No malicious Tor Exit Node operator will peek at your unencrypted packets or inject malicious code into you.
    
4. **Hiding the real IP from the VPN and websites:** Your IP is protected by a triple layer of Tor before it even reaches the VPN server.
    

#### What are the downsides?

- **No access to `.onion` domains:** Because all traffic exiting the VPN goes to the clearnet, you lose the ability to open hidden `.onion` services directly.
    
- **Turtle speed:** Traffic goes through 3 encrypted Tor nodes scattered around the world, and then through the VPN server. Delays (ping) will be massive (often over 1000 ms), and bandwidth heavily limited.
    
- **TCP protocol only:** Tor only supports TCP traffic. This means **you cannot use the WireGuard protocol** (which only works on UDP) directly through Tor. You must configure your VPN client to operate on the **OpenVPN protocol in TCP mode**.
    

#### Configuring VPN over Tor using iptables (Debian/Ubuntu)

For this trick to work and be 100% safe, we have to build an armored firewall. If the VPN client loses its connection, the system might try to connect directly to the internet, which would immediately reveal your real IP (a leak).

Our firewall must execute the following plan:

1. Allow the Tor service (system user `debian-tor`) to freely connect to the internet to build circuits.
    
2. Allow localhost traffic (`lo`) so the VPN client can connect to the local Tor port (`127.0.0.1:9050`).
    
3. Allow full traffic through the virtual VPN interface (`tun0`).
    
4. **Block all remaining outgoing traffic** (no other applications can send packets directly through your `eth0` / `wlan0` network card).
    

Save the following script as e.g., `/home/$USER/scripts_sh/vpn_over_tor_fw.sh`:

Bash

```bash
#!/bin/bash
# Script configuring strict iptables rules for VPN over Tor

# Ensure the script is run as root
if [ "$EUID" -ne 0 ]; then
  echo "[!] Run this script with sudo privileges!"
  exit 1
fi

# Network variables
TOR_UID=$(id -u debian-tor)  # Fetch UID of the Tor service user
VPN_INT="tun0"              # OpenVPN virtual interface
LO_INT="lo"                 # Loopback (localhost)

echo "[+] Clearing old rules..."
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

echo "[+] Setting default policy (DROP)..."
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP

echo "[+] Allowing localhost traffic (required to communicate with SOCKS5 proxy)..."
iptables -A INPUT -i $LO_INT -j ACCEPT
iptables -A OUTPUT -o $LO_INT -j ACCEPT

echo "[+] Allowing Tor service (UID: $TOR_UID) direct outbound access..."
iptables -A OUTPUT -m owner --uid-owner $TOR_UID -j ACCEPT

echo "[+] Allowing full communication through VPN interface ($VPN_INT)..."
iptables -A INPUT -i $VPN_INT -j ACCEPT
iptables -A OUTPUT -o $VPN_INT -j ACCEPT

echo "[+] Allowing related and established packets (ESTABLISHED, RELATED)..."
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

echo "[+] Rules successfully loaded! Your system is now cut off from the direct internet."
echo "[+] TOR is the only program allowed to connect directly."
```

Grant permissions and run the script:

Bash

```bash
chmod +x vpn_over_tor_fw.sh
sudo ./vpn_over_tor_fw.sh
```

#### Configuring the OpenVPN client for Tor SOCKS5

Now you have to modify the `.ovpn` configuration file from your VPN provider (e.g., the downloaded OpenVPN TCP config file from Mullvad).

1. Open the `.ovpn` file in a text editor:
    
    Bash
    
    ```bash
    nano mylocation_vpn.ovpn
    ```
    

2.  Make sure the configuration forces the **TCP** protocol (look for the line `proto tcp` or change `proto udp` to `proto tcp`).
3.  Add the following directive at the very bottom, which instructs the OpenVPN client to tunnel the connection through the local Tor proxy:
    
    ```text
    socks-proxy 127.0.0.1 9050
    ```

4. Save the file and launch the VPN connection:
    
    Bash
    
    ```bash
    sudo openvpn --config mylocation_vpn.ovpn
    ```

When OpenVPN successfully establishes a connection, your system will have full internet access, but all traffic will pass through the Tor network and exit through the VPN server!

---

### 2. Tor over VPN – The Standard Protective Shield

This is a much simpler and widely recommended configuration. Traffic flows like this:
`Your system ➔ Encrypted VPN Tunnel ➔ VPN Server (e.g., Mullvad) ➔ Tor Network ➔ Internet`

In this scenario, you connect to the VPN first, and only then do you launch Tor (e.g., Tor Browser).

#### Why do we do this? (Advantages)

1.  **Hiding Tor usage from ISP:** Your internet provider (and local monitoring systems) only see that you are connecting to a VPN server. They don't know you are using Tor, which prevents your home connection from being automatically flagged as "suspicious."
2.  **Protection against malicious Entry Nodes:** The first Tor network node (Guard Node) does not see your real home IP address, only the VPN server's IP address.
3.  **No configuration hassles:** Works out-of-the-box with any protocol (including the ultra-fast **WireGuard**).

#### Step by step: How to do this on Mullvad?

Mullvad is an ideal choice for this setup because it doesn't require providing any data upon registration, has a hard built-in Kill Switch, and a great app for Linux.

#### Step 1: Install official Mullvad VPN client on Debian 13

We add the official Mullvad repository and install the app:

```bash
# Download GPG key
sudo curl -fsSLo /usr/share/keyrings/mullvad-keyring.asc https://repository.mullvad.net/deb/mullvad-keyring.asc

# Add repository to apt sources
echo "deb [signed-by=/usr/share/keyrings/mullvad-keyring.asc] https://repository.mullvad.net/deb/stable main" | sudo tee /etc/apt/sources.list.d/mullvad.list

# Install application
sudo apt update && sudo apt install mullvad-vpn -y
```

#### Step 2: Mullvad Configuration (CLI or GUI)

You can manage Mullvad directly from the terminal:

Bash

```bash
# Log into your account (provide your 16-digit number)
mullvad account login YOUR_ACCOUNT_NUMBER

# Enable local Kill Switch (blocking traffic outside the VPN)
mullvad lockdown-mode set on

# Set protocol to WireGuard (fastest and most secure)
mullvad relay set tunnel-protocol wireguard

# Connect to a random, secure server (e.g., Switzerland or Sweden)
mullvad connect
```

You can also check the connection status:

Bash

```bash
mullvad status
```

#### Step 3: Launching the Tor network

Once Mullvad is connected and protecting your entire system, you simply run Tor:

- **Option A (Tor Browser - recommended for browsing):** Download and run the official Tor Browser. It will automatically connect to the Tor network through Mullvad's secure, encrypted tunnel.
    
- **Option B (System Tor + Proxychains for pentest tools):** Run the Tor service in the background:
    

```bash
sudo systemctl start tor
```

    Now you can run any tools (e.g., nmap or sqlmap) through the Tor network by typing `proxychains4` before the command:
    ```bash
    proxychains4 nmap -sT -PN target_ip
    ```

The packets will first go through the Mullvad server, then through 3 Tor nodes, and finally hit the target!

### 💡 Alternative: Mullvad Browser – Tor Privacy with VPN Speed

If you care about armor-plated protection against profiling (Canvas Fingerprinting, WebRTC blocking, script blocking, etc.), but don't want to suffer the agonies associated with the low speed of the Tor network, Mullvad, in collaboration with the **Tor Project**, has created a dedicated browser – **Mullvad Browser**.

It is a modified version of the Tor Browser that:

- Has identical mechanisms for protection against browser identification (fingerprinting).
    
- **Does not use the Tor network** – instead, it sends traffic directly through your network connection (i.e., through Mullvad VPN).
    
- Gives you the maximum speed of your connection while maintaining the highest browser privacy standards.
    

You can download the Mullvad Browser directly from Mullvad's website or via your package manager if it's available in your distro. It's an excellent compromise for daily, secure work!

## Do you think I wouldn't sink to a more paranoid LVL? B-)

**What if we added VirtualBox with a Whonix system, which has its own Tor gateway, to the current configuration?**

> _Generally, this is one of the worst ideas unless we make certain concessions. Why is it a shitty idea? Because of the time any scan will take... oh my god, the command **nmap -v -A** would take a few days with good winds :P_

## **But if someone still wanted to try...**

### 1. Network Architecture: What will your footprint look like?

When you fire up this setup, the packets leaving your browser in the Whonix Workstation will go through absolute cryptographic hell before reaching the destination server. It looks like this:

**Phase 1 (Your Host - Debian 13):** Your system -> Your ISP -> Host Tor Network (Entry -> Middle -> Exit) -> VPN Server

**Phase 2 (Virtual Machine - Whonix):** From Whonix's perspective, its "internet provider" becomes your VPN server. Whonix takes this encrypted traffic and packs it into _its own_ Tor network. VPN Server -> Whonix Tor Entry -> Whonix Tor Middle -> Whonix Tor Exit -> TARGET SERVER.

**Result:** Your traffic flies through a minimum of **7 hops** in different countries, 6 of which are Tor nodes, and right in the middle sits a VPN tunnel. Your "Entry Node" for the main, visible Tor is the IP address of a commercial VPN, not your home internet.

### 2. Why is this brilliant? (Advantages)

- **Ultimate Compartmentalization:** If malicious code on the audited site manages to break out of the browser (so-called sandbox escape) and infects the Whonix Workstation, it will learn absolutely nothing. Whonix Workstation only has a local IP (usually 10.152.152.x) and no physical hardware access.
    
- **Protection against Entry Guard de-anonymization:** Attacks on the Tor network often involve controlling the entry and exit nodes. In this setup, even if someone compromises your Whonix Tor, they will only reach your VPN's IP address. To go further, they would have to break the VPN logs, and then untangle the first Tor loop on your host. Good luck.
    

### 3. Brutal Reality (Disadvantages and "Not a fucking chance")

Before you start celebrating, we need to talk about physics and protocols. You just built a monster called **Tor-over-Tor** (separated only by a VPN). The Tor Project officially advises against such practices for two reasons:

- **TCP Meltdown:** Tor transmits data in packets using the TCP protocol, which guarantees packet delivery. When you pack TCP inside OpenVPN (which we forced to TCP), and all that inside a second Tor, chaos ensues. If one packet at the Whonix level is delayed, the inner TCP asks for retransmission. But the outer TCP (from the host) also sees the delay and also retransmits. The network starts choking on its own duplicates.
    
- **Speed and Timeouts:** Your ping will be counted in seconds (often 2000-5000 ms). Bandwidth will drop to a dozen or so kilobytes per second.
    

**What does this mean for a pentester?** Forget about using automated scanners like FFUF, Dirb, or Dirsearch from inside Whonix. All HTTP requests will catch a `timeout` (request time expired). The scanner will assume the site doesn't exist, while in reality, your 7-layer tunnel just didn't manage to respond in time. This setup is strictly only suitable for very slow, manual reconnaissance (reading forums, analyzing source code in the browser).

### 4. How to configure it (Setup for dummies)

If, despite the warnings, you want to do this and feel like Edward Snowden on steroids, here is how we'll tie it together:

**Step 1: Preparing the Host (Debian 13)** You must have our `vpn_over_tor_firewall.sh` script running from the previous lesson. Use the `curl ifconfig.me` command to check if your IP address is the VPN server address. Everything must go out through the `tun0` interface.

**Step 2: VirtualBox Installation**



```bash
sudo apt update
sudo apt install virtualbox virtualbox-ext-pack -y
```

**Step 3: Importing Whonix**

1. You go (via an armored fox) to the Whonix.org website and download the image for VirtualBox (the `.ova` file - contains Gateway and Workstation).
    
2. Open VirtualBox -> `File` -> `Import Appliance` -> Select the downloaded file.
    
3. Important: **Do not change anything in the virtual machines' network settings!** Whonix automatically configures its interfaces. The Gateway uses NAT mode (meaning it takes internet from your host), and the Workstation uses "Internal Network" (connects to the Gateway).
    

**Step 4: Routing Magic (Happens automatically)** Since your host (Debian 13) is bound by our hard IPTables rules, VirtualBox, wanting to provide internet to the Whonix Gateway, will be ruthlessly redirected to the `tun0` interface. You don't need to write additional rules. VirtualBox blindly obeys your Debian's routing table.

**Step 5: Firing up the systems**

1. Always start the **Whonix Gateway** machine first in VirtualBox. Wait for it to load and connect to the Tor network (it will use your VPN for this). Run the `upgrade-nonroot` command in its terminal to update it.
    
2. Only when the Gateway is running, launch the **Whonix Workstation**.
    

### Summary

You've messed with tracking systems so much that the NSA would probably have to blow a mid-sized country's power budget just to track you down. You have hardware spoofing (MAC), system protection (AppArmor in Whonix), sealed host DNS, Tor plugged into a VPN, which feeds a second Tor.

For targeted strikes and secure Darknet browsing (e.g., to check if your client's data leaked there)? Brilliant. For aggressive vulnerability scanning and Fuzzing? Useless due to lag.

The decision is yours, the architecture stands ready! What are we taking on next?

### If a question popped into your head:

> **_and how could I make this optimal so it actually makes sense?_**

**You asked a very smart question.** Since this 7-layer beast (Debian -> VPN -> Tor -> Whonix -> Tor) chokes on its own packets, how do you do it to "have your cake and eat it too"? Meaning, preserving absolute paranoia and anonymity, but simultaneously being able to run Nuclei or FFUF full throttle, without waiting a week for the result?

The answer is: **You cannot cheat the laws of physics and TCP architecture. You must change your approach and separate the anonymization layer from the striking layer.**

Instead of pushing heavy traffic (scanning 10,000 ports or thousands of web paths) through the Tor network from your home computer, professional pentesters (and those from the darker side of the force) build architecture based on a **Jump Server / VPS**.

Here is how you do it optimally and practically.

### The New Architecture: "The Ghost VPS"

Instead of doing "Inception" locally on your desk, we move the heavy machinery to the cloud, and your concrete Debian serves only as a command terminal.

**How does this look in practice?**

1. **Your computer:** Your Debian 13 with VPN-over-Tor running (our iptables script from the previous lesson) or simply Whonix. You are completely hidden.
    
2. **Buying a "Dirty" VPS:** Through a browser in Tor, you buy a cheap VPS (Virtual Private Server) from an offshore provider (e.g., in Switzerland, Iceland, or Romania) that accepts cryptocurrency payments (preferably Monero - XMR, because it's untraceable). You provide fake details, zero connection to your person.
    
3. **Installing the arsenal:** You log into this server via SSH and upload our `install_tools.sh` script from the first post there. It is this VPS that now has Nmap, Nuclei, and FFUF.
    
4. **Attack / Audit:** From your 100% anonymous Debian at home, you connect via SSH through Tor to your new VPS. You issue the command. The scan flies from the VPS level to the target server.
    

### Why does this make massive sense and solve all problems?

- **Speed and no timeouts (No lag):** Scanners run on the VPS utilize its direct, powerful internet connection (often 1 Gbps or 10 Gbps). The scan goes at full speed, without any delays.
    
- **Bypassing Cloudflare and Exit Node bans:** The target server (audited application) does not see traffic from the Tor network! It sees a clean, normal IP of a commercial cloud provider. No WAFs (Web Application Firewalls) or CAPTCHAs will block your scanners automatically.
    
- **Full UDP support:** Since it's the VPS hitting the target directly, you can use UDP scanning in Nmap as much as you want.
    
- **Ultimate anonymity:** Your traffic from you to the VPS is just plain text in an SSH terminal (it uses bytes of data, so Tor easily handles it without lag). Even if the target realizes it's being audited and files an Abuse report, they will block or shut down the VPS. Your real home address (or the IP of your private VPN) never touched the target server. You burn the $5 VPS and buy the next one.
    

### What if you absolutely MUST scan from your local computer (Whonix)?

If for some reason you don't want to set up an external VPS and insist on scanning from inside Whonix, you must adapt your tools to the fact that you have giant ping (lag). Scanners default to sending hundreds of requests per second and waiting briefly for a response. In Tor, this will end in a massacre and False Negatives.

You must "castrate" the tools so they work slowly but effectively:

> **1. FFUF (Directory fuzzing in web apps)** Forget multithreading. You must drastically increase the response wait time and slow down the rate-limit (number of requests). `ffuf -w wordlist.txt -u https://cel.com/FUZZ -t 1 -timeout 30 -rate 2` _(The `-t 1` flag is just 1 thread, `-timeout 30` tells it to wait up to half a minute for an answer, `-rate 2` sends only 2 requests per second)._
> 
> **2. Nmap (Port scanning)** You must order Nmap to treat the connection like it's very, very bad: `nmap -sT -Pn -n --max-retries 4 --scan-delay 2s --max-scan-delay 10s -T2 cel.com` _(You force it to do full TCP Connect `-sT`, disable pinging `-Pn`, order it to take a 2-second break between packets `--scan-delay 2s`, and use a slow `-T2` template)._

> **3. Nuclei (Vulnerability scanning)** Similarly, you must reduce concurrency and increase wait time: `nuclei -u https://cel.com -c 2 -bs 2 -timeout 25 -retries 3` _(Only two concurrent queries `-c 2`, small batches `-bs 2`, and a giant timeout)._

### I think most people just realized what kind of guy they're dealing with here hah

**_of course I could complicate this even more XD_**

**And I hope the knowledge I'm passing on, which I shouldn't be :) will be appreciated. If you want, I won't mind a small tip in XMR (if you don't know what XMR is) - I don't know why you're reading this :'P**

#### ☕ Tip Me / Was this useful? Buy me a beer!

If my tools or guides proved useful to you and you want to support my development, you can throw some spare change to the addresses below. We respect privacy, so Monero is always welcome! 🥷

|**Cryptocurrency**|**Network**|**Wallet Address**|
|---|---|---|
|🟠 **Bitcoin (BTC)**|Bitcoin|`bc1q_YOUR_BTC_ADDRESS_HERE`|
|🦇 **Monero (XMR)**|Monero|`4_YOUR_XMR_ADDRESS_HERE`|
|💎 **Ethereum (ETH)**|ERC-20|`0x_YOUR_ETH_ADDRESS_HERE`|
|💵 **Tether (USDT)**|TRC-20 (Tron)|`T_YOUR_USDT_ADDRESS_HERE`|

**But alright, returning to the topic, remember you can use these detailed configurations interchangeably, you don't always have to sit on this 3000-layer beast and suffer xD**

> _To be honest, that example from the end is probably only suitable if you want to fly to North Korea and become an anti-government white-hat journalist_

## Okay, let's move on to something really simple...

**I prepared a complete package for you (ZIP file at the top) consisting of three files. Of course, I made sure our application doesn't use any external libraries (CDN), so everything will work fully locally and instantly, without the stress of some external WAF or Cloudflare blocking your style sheets.**

### What will you find in the package?

The package contains three files that work together:

1. **`setup_lamp.sh`** – The main script for your Debian 13. It installs and configures the LAMP stack (Apache2, MariaDB, PHP), creates a new database, and replaces the default Apache page with our vulnerable app.
    
2. **`database.sql`** – The database structure, which creates a users table and injects "secret" flags (notes) into it that you'll have to steal using SQLi.
    
3. **`index.php`** – Our actual attack target. A simple website written in pure PHP without any input filtering.
    

### 1. Installation Script (`setup_lamp.sh`)

This script does all the dirty work. It installs the server, creates a database user, imports tables, and grants permissions for the browser files.



```bash
#!/bin/bash
# Script for automated installation and configuration of the LAMP environment on Debian 13

set -e
echo "[+] Updating packages..."
sudo apt update

echo "[+] Installing Apache2, MariaDB (MySQL) and PHP..."
sudo apt install -y apache2 mariadb-server php libapache2-mod-php php-mysql

echo "[+] Configuring and starting services..."
sudo systemctl enable apache2
sudo systemctl start apache2
sudo systemctl enable mariadb
sudo systemctl start mariadb

echo "[+] Creating database and lab user..."
sudo mysql -e "CREATE DATABASE IF NOT EXISTS lab_db;"
sudo mysql -e "CREATE USER IF NOT EXISTS 'lab_user'@'localhost' IDENTIFIED BY 'StrongDBPass123!';"
sudo mysql -e "GRANT ALL PRIVILEGES ON lab_db.* TO 'lab_user'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"

echo "[+] Importing vulnerable database structure..."
if [ -f "./database.sql" ]; then
    sudo mysql lab_db < ./database.sql
fi

echo "[+] Deploying vulnerable web application (index.php)..."
if [ -f "./index.php" ]; then
    sudo rm -f /var/www/html/index.html
    sudo cp ./index.php /var/www/html/index.php
    sudo chown -R www-data:www-data /var/www/html/
    sudo chmod -R 755 /var/www/html/
fi

echo "[+] Restarting Apache server..."
sudo systemctl restart apache2
echo "Your local laboratory is ready: http://localhost/"
```

### 2. Leaky Code (What will we be attacking?)

Let's look into the heart of the problem, meaning exactly what I broke in the `index.php` file so you could hit it.

**SQL Injection (SQLi) Vulnerability:** A classic of the genre. The `id` parameter is passed in a GET request and literally concatenated as raw text into the SQL query, instead of using so-called _Prepared Statements_. Additionally, I left MySQL error display enabled, so you can practice _Error-Based SQL Injection_ techniques.

PHP

```php
// Fragment of vulnerable index.php code
if (isset($_GET['id'])) {
    $id = $_GET['id'];
    
    // VULNERABILITY: Direct injection of variable into SQL query
    $query = "SELECT id, username, role, secret_note FROM users WHERE id = " . $id;
    $result = $conn->query($query);
}
```

**Cross-Site Scripting (XSS) Vulnerability:** I made a small simulator of an "add comment" section. The form flies via the POST method, and PHP receives it and spits it directly into the HTML structure. No verification like `htmlspecialchars()` or `strip_tags()`. Just a plain, shameless `echo`.

PHP

```php
// Fragment of vulnerable index.php code
$xss_output = "";
if (isset($_POST['comment'])) {
    // VULNERABILITY: Lack of filtering for special HTML/JS characters
    $xss_output = $_POST['comment'];
}
```

The page structure itself then contains: `<div> <?php echo $xss_output; ?> </div>`, which means that if you throw a `<script>` tag in there, the browser will read it as full-fledged JavaScript code in the page source.

### How to run this, Mr. Handyman?

1. Download the ZIP file provided at the very top.
    
2. Extract everything to a single folder on your Debian (e.g., `~/Documents/lab_pentest`).
    
3. Enter this folder in the terminal:
    

```bash
cd ~/Documents/lab_pentest
```

4.  Grant execute permissions to the installation script:

    ```bash
chmod +x setup_lamp.sh
```

5. Fire up the environment:
    

```bash
./setup_lamp.sh
```

Once the script finishes grinding, launch your armored Firefox and type in the address bar: `http://localhost/` or your local IP address of the machine you set this up on.

You now have your own, private testing ground where you can completely legally practice manual commands for cutting off SQL queries and executing malicious JavaScript payloads. Let me know once you get the machine running – in the next step, I'll show you how to approach this manually, without using automated harvesters. Ready to hack into your own server? B-)

**video here**

### 1. WhatWeb (Reconnaissance / Fingerprinting)

**What is it for?** WhatWeb is a so-called *Web Scanner* of the new generation, but I prefer to call it a "digital detective". It's not for cracking or hacking. It's for **fingerprinting**, meaning identifying the technologies a site is running on. Before you hit a target, you must know what you're dealing with. With one shot, WhatWeb will tell you: what CMS it is (WordPress, Joomla?), what PHP version, what server (Apache/Nginx/IIS), what JS libraries the site uses (jQuery, React), and even if it detected any web application firewalls (WAF).

**Aggression Levels:** This is a key WhatWeb feature. It defines how much noise you make in the server logs.

*   **-a 1 (Stealthy / Passive):** Default. Makes only one HTTP GET request. Quiet as a ghost.
*   **-a 3 (Aggressive):** Hits deeper. Guesses folders, downloads more files, looks for hidden paths. Louder, but detects more.
*   **-a 4 (Heavy):** Will make the admin's logs look like medieval times. Scans absolutely every plugin and every possible configuration file.

**Most important parameters and usage examples:**

```bash
# Fast, quiet shot at a single target (level 1)
whatweb example.com

# Full, chatty scan (Verbose) - will show you exactly why it detected a given software
whatweb -v example.com

# Aggressive scan looking for hidden technologies
whatweb -a 3 https://example.com

# Mass scanning from a file (e.g., list of subdomains)
whatweb -i list_of_subdomains.txt

# User-Agent spoofing (so you don't introduce yourself as a scanner)
whatweb --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64)" example.com

# Saving results to a nice JSON file (ideal for further script processing)
whatweb --log-json=results.json example.com
```

### 2. Nikto (The old veteran of dirty work)

**What is it for?** Nikto is a Perl-written classic that's been around for years, but can still dig a skeleton out of the closet. It's a typical **WWW server scanner**. It's not subtle. It blasts the target with thousands of requests (often around 6500+ tests), looking for: old, leaky server versions, default config files (that the admin forgot to delete), unsecured directories (e.g., `.git/`, `backup/`) and basic header issues (e.g., missing anti-Clickjacking headers).

If the admin left a `phpinfo.php` file on the server or an old `database.zip` archive, Nikto will find it.

**Most important parameters and usage examples:**

Bash

```bash
# Standard scan of a single target
nikto -h http://10.10.10.10

# Scanning a specific port (if the web app isn't on 80/443)
nikto -h 10.10.10.10 -p 8080

# Scanning while bypassing simple IDS/WAF systems (Evasion techniques)
nikto -h example.com -evasion 1   # 1 - Adds random URL encoding characters (e.g., %2e instead of a dot)
nikto -h example.com -evasion A   # A - Uses a carriage return in requests

# The -Tuning parameter (Selecting specific tests to cut down time)
# 1 - Config files, 4 - XSS, 8 - OS Command Execution (RCE), 9 - SQLi
nikto -h example.com -Tuning 1489

# Forcing the scan through our Tor proxy (127.0.0.1:9050) - if you aren't using our iptables script
nikto -h example.com -useproxy http://127.0.0.1:9050

# Saving results to HTML format (generates a cool, readable report)
nikto -h example.com -Format htm -o report_nikto.html
```

### 3. Nuclei (The modern machine of destruction)

**What is it for?** Nuclei (by ProjectDiscovery) is currently the absolute king of web audits. Unlike Nikto, Nuclei doesn't have hardcoded tests. It operates based on **YAML templates**. The template database is updated daily by the community worldwide. When a new, critical vulnerability drops (e.g., a CVE for Exchange servers, Log4j, or WordPress plugin flaws), a template lands in the Nuclei database within hours, and you can immediately scan hundreds of your machines for it.

It's hellishly fast (written in Go), can look for data leaks, RCE vulnerabilities, XSS, subdomain takeovers, and cloud misconfigurations (AWS/Azure).

**Most important parameters and usage examples:**

Before you even fire it up, always type `nuclei -ut` (Update Templates) after installation to download the latest attack vectors!

Bash

```bash
# Simple shot at all basic vulnerabilities for a single address
nuclei -u https://example.com

# Gold: Scanning an entire list of URLs from a file
nuclei -l my_targets.txt

# Scanning using specific tags (e.g., we check ONLY cve vulnerabilities and wordpress flaws)
nuclei -u https://example.com -tags cve,wordpress

# Scanning with a specific template type (e.g., only looking for exposed tokens and passwords)
nuclei -u https://example.com -t exposures/

# Optimization and aggression options (scan speed)
# -c (concurrency) - Number of templates run at once
# -rl (rate-limit) - Number of requests per second
nuclei -l targets.txt -c 50 -rl 150

# Enabling Auto-Scan mode
# Wappalyzer technology will connect with WhatWeb recon and launch only templates
# that match detected technologies (saves a ton of time!)
nuclei -u https://example.com -as

# Scanning while pushing traffic through our Tor (Proxy)
nuclei -u https://example.com -proxy socks5://127.0.0.1:9050

# Saving results to a markdown file
nuclei -u https://example.com -o results_nuclei.md
```

### 4. Nmap (The all-seeing eye of the network)

**What is it for?** Nmap (Network Mapper) is the absolute godfather of infrastructure scanning. While WhatWeb or Nuclei hit web applications (OSI layer 7), Nmap checks what is actually open on the server (layers 3 and 4). Before you start digging into websites, you need to know if the target hasn't left an open SSH port, MySQL database, or an FTP panel with guest access. Nmap maps the network, detects service versions, the OS, and thanks to the NSE engine (Nmap Scripting Engine) it can independently catch (and even exploit) basic vulnerabilities.

**Speed Levels (Timing Templates):** From `-T0` (Paranoid – extremely slow, bypasses IDS) to `-T5` (Insane – blasts like a machine gun, loud as hell). The most common golden mean is `-T4`.

**Most important parameters and usage examples:**

Bash

```bash
# Simple, standard scan of the 1000 most popular ports
nmap 10.10.10.10

# Aggressive scan: Detects OS (-O), service versions (-sV), runs default scripts (-sC)
nmap -A -T4 10.10.10.10

# Stealth scan (Stealth SYN scan) - doesn't establish a full TCP connection, harder to detect by firewalls
nmap -sS 10.10.10.10

# Scanning ALL 65535 ports (takes some time, but hides nothing)
nmap -p- 10.10.10.10

# Ultra-fast scan (forces sending a minimum of 1000 packets per second)
nmap -p- --min-rate 1000 10.10.10.10

# Using NSE scripts: Searching for specific vulnerabilities (e.g., for SMB or FTP services)
nmap -p 445 --script smb-vuln-* 10.10.10.10
nmap --script vuln 10.10.10.10

# Saving results to all formats (XML, normal, Greppable) for later
nmap -p- -sV -oA scan_results 10.10.10.10
```

### 5. SQLMap (The automated database sniper)

**What is it for?** SQLMap is a tool that pulls out databases like a magician pulls a rabbit out of a hat. While in our lab on `index.php` you learned manual SQL Injection, in real life, when you have e.g., time delays (Time-Based Blind SQLi), doing it manually would take you weeks. You point SQLMap to a leaky parameter in a URL or a request file, and it autonomously determines the DB type (MySQL, PostgreSQL, Oracle), the vulnerability type, and allows you to dump entire tables, passwords, and even – if permissions allow – get access to the server shell (OS shell).

**Risk and aggression levels:**

- `--level` (1-5): The higher it is, the more places it checks (e.g., at level 3 it also checks the User-Agent header, and at 5 it checks everything including cookies).
    
- `--risk` (1-3): Risk 1 is safe. Risk 3 might add malicious entries to the database or break the target application (use with brains on production).
    

**Most important parameters and usage examples:**



```bash
# Simple test of a GET parameter vulnerability on a page
sqlmap -u "http://localhost/index.php?id=1"

# Once vulnerability is confirmed, we dump the list of all databases
sqlmap -u "http://localhost/index.php?id=1" --dbs

# Selecting a specific database and dumping its table list (-D = database)
sqlmap -u "http://localhost/index.php?id=1" -D lab_db --tables

# Extracting (dumping) all data from the 'users' table in the 'lab_db' database
sqlmap -u "http://localhost/index.php?id=1" -D lab_db -T users --dump

# Attacking via a POST form (e.g., login)
sqlmap -u "http://example.com/login.php" --data="username=admin&password=test"

# Automation: Agree to everything (--batch) and use random headers (--random-agent)
sqlmap -u "http://example.com/page?id=1" --batch --random-agent --dbs

# Tunneling traffic through Tor (so you don't burn your IP during a heavy attack)
sqlmap -u "http://example.com/page?id=1" --tor --tor-type=SOCKS5
```

### 6. XSStrike (The XSS Surgeon)

**What is it for?** Most cheap scanners blindly send thousands of popular payloads like `<script>alert(1)</script>` and pray one works. XSStrike does it differently. It's an intelligent system that analyzes the document (DOM), checks what context your input landed in (e.g., inside an HTML attribute, or inside a JS script), and generates **a single, perfectly tailored payload** that will bypass filters (WAF). It also has a great fuzzer and can scan an entire site looking for hidden parameters that are vulnerable to Cross-Site Scripting.

**Most important parameters and usage examples:**



```bash
# Basic scan of a specific URL parameter
python3 xsstrike.py -u "http://localhost/index2.php?search=test"

# Bypassing WAF (Web Application Firewall) with an injected delay
python3 xsstrike.py -u "http://example.com/search?q=test" --timeout 5 --delay 2

# Fuzzing hidden parameters on a page (if, for example, you don't know the hidden field name)
python3 xsstrike.py -u "http://example.com/page" --fuzzer

# Crawling the entire site for XSS vulnerabilities (depth = 3)
python3 xsstrike.py -u "http://example.com" --crawl -l 3

# Attacking via POST method (instead of GET) in a form
python3 xsstrike.py -u "http://localhost/index.php" --data "comment=test"
```

### 7. FFUF (Fuzz Faster U Fool)

**What is it for?** FFUF is a Go-written, absurdly fast web application fuzzer. It replaced older tools like DirBuster or Dirb. What is Fuzzing for? You take a massive text file (a so-called wordlist, e.g., SecLists), which contains hundreds of thousands of directory names, files, keywords, or parameter names. FFUF inserts these words sequentially in place of the keyword `FUZZ` and sees what the server replies. Thanks to it, you will find hidden login panels (e.g., `/admin_panel_123`), forgotten backup files (e.g., `/database.sql.bak`), or unsecured APIs.

**Most important parameters and usage examples:**



```bash
# Searching for hidden directories (e.g., replaces the word FUZZ with 'admin', 'backup', 'login')
# Will return anything with HTTP code 200 (OK)
ffuf -u "http://example.com/FUZZ" -w /usr/share/wordlists/dirb/common.txt

# Searching for specific file extensions (e.g., .php, .txt, .zip)
ffuf -u "http://example.com/FUZZ" -w wordlist.txt -e .php,.txt,.zip

# Fuzzing GET parameters (we are looking for hidden variables, e.g., ?debug=1)
ffuf -u "http://example.com/index.php?FUZZ=1" -w parameters.txt

# Hiding junk: Excluding results based on word count, lines, or size. 
# (-fs 42) -> Ignore results where the page weight is exactly 42 bytes (a common "false positive")
ffuf -u "http://example.com/FUZZ" -w wordlist.txt -fs 42

# Searching for hidden virtual hosts (VHosts/Subdomains) via headers
ffuf -u "http://example.com" -H "Host: FUZZ.example.com" -w subdomains.txt -mc 200

# Increasing speed (-t is threads) and saving to JSON format
ffuf -u "http://example.com/FUZZ" -w wordlist.txt -t 100 -o results_ffuf.json
```

### Chapter Summary: Foundations, OPSEC, and Arsenal Ready for Action B-)

Phew, alright my guy, let's take off the ski masks for a second. That was a massive dump of knowledge, but if you made it through all of this and set up the system according to these guidelines, congratulations – you just leaped past 90% of the TikTok "hackers" who fire up Kali Linux on their home WiFi and wonder why their IP gets banned.

Let's do a quick recap of what we're ending this chapter with:

- **Armored System:** You have a sealed Debian (UFW, Fail2ban, Auditd) and secured DNS. Nothing leaks out without your knowledge.
    
- **Privacy and Traffic Routing:** We configured Firefox so it doesn't spill logs, we cut the head off WebRTC and Canvases. What's more, you now know the difference between the **Tor-over-VPN** mode (standard shield) and **VPN-over-Tor** (bypassing Cloudflare and Exit Node bans with UDP support).
    
- **Own Testing Ground:** We set up a local lab (LAMP) with `index.php` and `index2.php` applications that are practically begging for malicious code injection. Everything is fully secure, without going out to the external network.
    
- **The Holy Seven of the Auditor:** You have a powerful arsenal installed, updated, and broken down into commands:
    
    1. **WhatWeb** – for silent terrain recon.
        
    2. **Nikto** – for finding old, forgotten dirt on the server.
        
    3. **Nuclei** – for merciless execution of the newest vulnerabilities and CVEs.
        
    4. **Nmap** – for mapping ports and the network layer.
        
    5. **SQLMap** – for autonomously sucking out leaky databases.
        
    6. **XSStrike** – for precisely bypassing WAFs and cutting with XSS like a scalpel.
        
    7. **FFUF** – for fuzzing at the speed of light and discovering hidden directories.
        

**What's next, crazy?** Enough with the dry theory and configuration. You have the tools, you have the knowledge about their flags, and you have a target (your local range). In the next chapter, we take off the training wheels, open the terminal, and go full throttle with a practical audit. We'll fuzz paths live, throw SQLMap at GET parameters, and parse scan results to finally automate it all using AI (Gemini).

Fire up your local servers, take notes on the commands, and see you in the next part. Take care, bye! 🥷

_(And if the knowledge hit the spot, a reminder about the crypto table above! Monero is always appreciated!)_
