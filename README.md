# PlexVPSForwardingTailscale

The purpose of the below code is to provide a simple way to configure Remote Access to Plex through the use of a VPS (for example, to bypass a CGNAT).

This script will:
- Perform an initial update of your system
- Configure a Tailscale VPN tunnel between your Plex Server and the VPS server
- Configure the IPTables rules to forward packets from your VPS to your Plex Server

## Prerequisites:
- You must have a Tailscale account, and Tailscale must be set up and running on your Plex Server (you should be able to see your device with its associated Tailscale IP Address once you have logged into Tailscale.com).
- You must have a VPS with a fixed/static IP Address. This guide presumes that this is a fresh VPS install (tested on Ubuntu LTS 22.04).

The following command can be run to automatically configure the VPS. You will need to accept a few prompts as the script executes. Additionally, as it installs Tailscale, it will provide a link to activate the Tailscale for the VPS which you must copy into your browser, this will activate the Tailscale node on your device.

## Script (all one line)

sudo apt-get update && sudo apt-get upgrade && curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null && curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list && sudo apt-get update && sudo apt-get install tailscale && sudo tailscale up && iptables -t nat -A POSTROUTING -o tailscale0 -j MASQUERADE && iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 32400 -j DNAT --to 1.2.3.4:32400 && iptables -A FORWARD -p tcp -d 1.2.3.4 --dport 32400 -j ACCEPT && sudo apt install iptables-persistent && sysctl -w net.ipv4.ip_forward=1

NOTE: In the above script, you must replace 1.2.3.4 with the Tailscale IP Address of your Plex Server (ex. 100.101.143.491).
You can find this Tailscale IP Address by going to Tailscale.com, logging in, finding the machine you set up in Prerequesite #1, and copying the address from the site.

## Plex Configuration

In Plex, go to Settings -> Network -> Custom server access URLs

Add in the Static IP Address of your VPS (ex. http://12.392.47.382:32400)


## Script Broken Down into Components

The above script is a one-line script to configure the VPS and routing. The script is broken down into components below:

### Update

sudo apt-get update && sudo apt-get upgrade

### Enable Tailscale

curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null

curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list

sudo apt-get update && sudo apt-get install tailscale

sudo tailscale up

### Add IPTables Routing

iptables -t nat -A POSTROUTING -o tailscale0 -j MASQUERADE

iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 32400 -j DNAT --to 100.104.157.143:32400

iptables -A FORWARD -p tcp -d 100.104.157.143 --dport 32400 -j ACCEPT

### Make IPTables Persistent & Enable Packet Forwarding

sudo apt install iptables-persistent && sysctl -w net.ipv4.ip_forward=1
