# PlexVPSForwardingTailscale

The purpose of the below code is to provide a simple way to configure Remote Access to Plex through the use of a VPS (for example, to bypass a CGNAT or to set up Remote Access when you are not able to Port Forward on your local network).

This script will:
- Perform an initial update of your system
- Configure a Tailscale VPN tunnel between your Plex Server and your VPS server
- Configure the IPTables rules to forward packets from your VPS to your Plex Server
- Install Fail2Ban to protect against unauthorized SSH access
- Disable ipv6

## Prerequisites:
- You must have a Tailscale account, and Tailscale must be set up and running on your Plex Server (you should be able to see your device with its associated Tailscale IP Address once you have logged into Tailscale.com).
- You must have a VPS with a fixed/static IP Address. This guide presumes that this is a fresh VPS install (tested on Ubuntu LTS 22.04).

The following command can be run to automatically configure the VPS. You will need to accept a few prompts as the script executes. Additionally, as it installs Tailscale, a link will be provided in the terminal which you need to copy and paste into your web browser to activate Tailscale for your VPS, this will activate the Tailscale node on your device.

## Step 1: Script (One Line)

Copy and paste the below code into the terminal on your VPS (this should all be one line) and hit enter:

sudo apt-get update && sudo apt-get upgrade && curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null && curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list && sudo apt-get update && sudo apt-get install tailscale && sudo tailscale up && sudo iptables -t nat -A POSTROUTING -o tailscale0 -j MASQUERADE && sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 32400 -j DNAT --to 1.2.3.4:32400 && sudo iptables -A FORWARD -p tcp -d 1.2.3.4 --dport 32400 -j ACCEPT && sudo apt install iptables-persistent && sudo sysctl -w net.ipv4.ip_forward=1 && sudo apt install fail2ban -y && sudo systemctl start fail2ban && sudo systemctl enable fail2ban && sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1 && sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1

NOTE: In the above script, you must replace 1.2.3.4 with the Tailscale IP Address of your Plex Server (ex. 100.101.143.491).
You can find this Tailscale IP Address by going to Tailscale.com, logging in, finding the machine you set up in Prerequesite #1, and copying the address from the site.

## Step 2: Plex Configuration

In Plex, go to Settings -> Network -> Custom server access URLs

Add in the Static IP Address of your VPS (ex. http://12.392.47.382:32400)


# Script Broken Down into Components

The above script is a one-line script to configure the VPS and routing. The below section breaks down the script into its components (running the script above will complete the install, the below scripts are for information purposes/do not need to be run separate):

### 1. Update VPS

sudo apt-get update

sudo apt-get upgrade

### 2. Install and Enable Tailscale

curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null

curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list

sudo apt-get update

sudo apt-get install tailscale

sudo tailscale up

### 3. Add IPTables Routing/Packet Forwarding

sudo iptables -t nat -A POSTROUTING -o tailscale0 -j MASQUERADE

sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 32400 -j DNAT --to 1.2.3.4:32400

sudo iptables -A FORWARD -p tcp -d 1.2.3.4 --dport 32400 -j ACCEPT

### 4. Make IPTables Persistent & Enable Packet Forwarding

sudo apt install iptables-persistent

sudo sysctl -w net.ipv4.ip_forward=1

### 5. Install and Start Fail2Ban

sudo apt install fail2ban -y

sudo systemctl start fail2ban

sudo systemctl enable fail2ban

### 6. Disable ipv6

sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1

sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
