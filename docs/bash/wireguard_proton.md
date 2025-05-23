# Setting up Proton VPN using Wireguard
Instructions for setting up Proton VPN to work on Linux (Fedora KDE). Adapted
from Proton's [own instructions](https://protonvpn.com/support/wireguard-linux#how-to-manually-configure-wireguard).

## Install Wireguard
```bash
# install wireguard
sudo dnf install wireguard-tools
```

## Download your WireGuard configuration file from Proton VPN
Log into your Proton VPN account in a browser and follow the
[instructions](https://protonvpn.com/support/wireguard-configurations/) on
Proton's site to create and download a Wireguard configuration file.

Rename the file, if necessary, to make the total filename under 15 characters
in length. Then move it into the `/etc/wireguard` directory.

The following commands can no be used (substitute your configuration file's
filename, minus the .conf extension, in place of `config_filename`).

```bash
# manually turn on VPN
sudo wg-quick up config_filename

# check VPN status
sudo wg

# turn off VPN
sudo wg-quick down config_filename
```

## Configuring the VPN to start automatically on startup

```bash
# TODO test/fix: this is not working for me
#   possibly being blocked by SELinux?

# configure to start on boot
sudo systemctl enable wg-quick@config_filename.conf

# start now
sudo systemctl start wg-quick@config_filename.conf
```

## Firewall Configuration
If necessary (it wasn't for me), open WireGuard's default port for the VPN.

```bash
sudo firewall-cmd --add-port=51820/udp --permanent
sudo firewall-cmd --reload
```