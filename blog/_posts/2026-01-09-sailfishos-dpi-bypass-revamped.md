---
layout: post
title:  "SailfishOS DPI bypass tutorial (Revamped)"
date:   2026-01-09 11:16:00 +0300
categories: sailfishos restrictions
---

### This is a revamped version of my ByeDPI installation guide for 2026.

Some countries block some services access. For example, recently YouTube and Discord got blocked in Russia, where I live. This little guide will help you bypass these restrictions on SailfishOS.

Local service blocks are usually done by DPI (Deep Package Inspection). The provider looks into the package and, if notices that it is going to a blocked service, returns an error instead of routing it to the destenation.

There are many methods to bypass DPI, but most of them run as a proxy server modifying packages structure so they are not understandable by the provider but understandable by the destenation. I don't have a big knowledge on this, if you want to read more about this I recommend you reading [this](https://t.me/endermanch/269) post in Enderman's telegram channel. 

So, we need to run a proxy server on our phone and use it as the global proxy server in settings. Before, I used [SpoofDPI](https://github.com/xvzc/SpoofDPI), but now the method used there is patched (at least in Russia). There is also [zapret](https://github.com/bol-van/zapret), but I haven't tried it. For now, I'm using [ByeDPI](https://github.com/hufrea/byedpi).

ByeDPI has a problem, it uses SOCKS. And SailfishOS only supports HTTP proxy. We could also use another popular tool [zapret](https://github.com/bol-van/zapret), which is better for our case because it uses HTTP, but that is for another tutorial. Later I might release the tutorial for zapret as well. If I will then I'll mark so at the top of the page.

So, we decided to use ByeDPI. Since it uses SOCKS, we need a SOCKS to HTTP forwarder, and we'll use Privoxy for this.

1. Install Chum, and install Privoxy from there.
2. Enable Privoxy
    - Enable developer tools and set an SSH password first, then open freshly installed Terminal app
    - When asked, enter your password
    - You can use up/down arrows to navigate through command history
    ```bash
devel-su systemctl enable harbour-privoxy
devel-su systemctl start harbour-privoxy
    ```
1. Download ByeDPI from [releases page](https://github.com/hufrea/byedpi/releases/latest), unpack it
    - For armv7hl devices, download armv7l version
    - For aarch64 devices, download aarch64 version
    - For i486 devices, download i686 version
2. In Terminal, move ByeDPI binary to `/usr/local/bin/`:
    - When asked, enter your password
    - You can autocomplete paths using tab key
    - Replace `/home/defaultuser/Downloads/ciadpi-aarch64` with your actual binary path
    ```bash
cd ~
devel-su mv /home/defaultuser/Downloads/ciadpi-aarch64 /usr/local/bin/ciadpi
    ```
1. Try running `ciadpi`. It should show nothing now. If it shows an error, you have done something wrong. Press control+c to stop byedpi
2. Configure a systemd service to run byedpi in the background
   1. For this, I recommend you installing file browser from OpenRepos
   2. Open file browser, enable showing hidden files
   3. Go to home (`~`), then `.config/systemd/user`
   4. In up menu, select Create new..., empty text file and `byedpi.service` as the name
   5. Click on newly created file, swipe to left, and select Edit in the menu
   6. Type:
        ```systemd
[Unit]
Description=ByeDPI
Documentation=https://github.com/hufrea/byedpi
Wants=network-online.target
After=network-online.target nss-lookup.target
[Service]
NoNewPrivileges=yes
StandardOutput=null
StandardError=journal
EnvironmentFile=-/etc/byedpi.conf
EnvironmentFile=-%h/.config/byedpi.conf
ExecStart=ciadpi $BYEDPI_OPTIONS
TimeoutStopSec=5s
PrivateTmp=true
ProtectSystem=full
[Install]
WantedBy=default.target
        ```
    1. Save the file
1. Add the ByeDPI configuration
   Create a file at `~/.config/byedpi.conf` (`~` is the home directory) with the following contents:
   `BYEDPI_OPTIONS="-o1 -o25+s -T3 -At"`
   These are the options which work for me currently. If these don't work for you, modify them accordingly.
2. Enable byedpi
    ```bash
systemctl reload-daemon --user
systemctl enable --user --now byedpi
    ```
1. Configure Privoxy to forward byedpi
   1. For this, I recommend you installing file browser and the root extension from OpenRepos
   2. Open file browser with root mode
   3. Go to Root (`/`), then `usr`, `share`, `harbour-privoxy` and finally `conf`
   4. Click on `config.sailfish`, swipe to left, and select Edit in the menu
   5. At the top, insert a new line and type: `forward-socks5 / 127.0.0.1:1080 .`. Do not forget the dot at the end
2. Restart Privoxy in Terminal: `devel-su systemctl restart harbour-privoxy`
3.  Configure SailfishOS to use Privoxy
    1. Open Settings, WLAN, Advanced and enable Global proxy
    2. Choose Manual and click Add proxy
    3. Type `127.0.0.1` as the address and `8118` as the port
    4. Click Save
4.  Done! To temporarily disable the proxy, use the Global proxy toggle. You can also add it to the top menu.

## Uninstallation

1. Disable global proxy in SailfishOS settings. Reset other proxy fields if needed
2. Disable Privoxy:
    ```bash
devel-su systemctl stop harbour-privoxy
devel-su systemctl disable harbour-privoxy
    ```
3. Uninstall Privoxy in Chum or on the home screen
4. Disable ByeDPI:
    ```bash
systemctl disable --user --now byedpi
    ```
5. Remove ByeDPI service
   1. For this, I recommend you installing file browser from OpenRepos
   2. Open file browser
   3. Remove files at `~/.config/systemd/user/byedpi.service` and `~/.config/byedpi.conf` (`~` is the home directory)
   4. Remove `byedpi.service` file
6. Remove ByeDPI binary. In terminal: `devel-su rm -f /usr/local/bin/ciadpi`
7. Done! ByeDPI or Privoxy is no longer running in the background.
