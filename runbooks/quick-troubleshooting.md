# Quick Troubleshooting Steps

Work from the bottom up: cable, IP, gateway, routing, DNS, firewall, service.
Change one thing at a time and write down what you changed.

## No Internet Access

1. `ip link show` - confirm the interface is UP
2. `ip addr show` - confirm the interface has an IP
3. `ip route` - confirm a default gateway exists
4. `ping <gateway-ip>` - test the local network
5. `ping 8.8.8.8` - test raw connectivity
6. `host github.com` - test DNS
7. `curl -I https://github.com` - test a real web request

If step 5 works but step 6 fails, the problem is DNS, not the internet.

## No IP Address / DHCP Problems

1. `ip addr show` - an address starting with 169.254 means DHCP failed
2. `nmcli device status` - confirm the interface is connected
3. `sudo dhclient -r && sudo dhclient -v eth0` - release and renew
4. `journalctl -u NetworkManager | grep -i dhcp` - read DHCP logs
5. Check the switch port is in the right VLAN (`show vlan brief`)
6. Check the DHCP server is running and has free leases

## Can't Reach Internal Host

1. Confirm same VLAN/subnet
2. `ping <host-ip>` - test basic reachability
3. `ip neigh show` - check the host appears in the ARP table
4. `traceroute <host>` - find where it drops
5. Check firewall rules between VLANs
6. `nc -zv <host-ip> <port>` - test the specific port
7. Check the host's own firewall (`sudo nft list ruleset`)

## DNS Not Working

1. `cat /etc/resolv.conf` or `resolvectl status` - check DNS servers
2. `dig example.com +short` - test with your DNS server
3. `dig @8.8.8.8 example.com +short` - test with a public DNS server
4. If only step 3 works, your DNS server is the problem
5. `dig -x <ip>` - test reverse lookup
6. `sudo resolvectl flush-caches` - clear the local cache

## Service or Port Not Reachable

1. `sudo ss -tulpn` (on the server) - is the service listening?
2. `sudo systemctl status <service>` - is it running?
3. `nc -zv <server-ip> <port>` (from the client) - can you connect?
4. `curl -v http://<server-ip>:<port>` - see the full request
5. Check firewall rules on the server, network, and client
6. `sudo tcpdump -i eth0 -nn port <port>` - see if packets arrive

## Can't SSH Into a Host

1. `ssh -v user@host` - verbose output shows where it fails
2. `nc -zv <host> 22` - is port 22 open?
3. `sudo systemctl status ssh` (on the host) - is the service running?
4. `sudo ss -tlnp | grep :22` - is it listening?
5. `ls -ld ~/.ssh && ls -l ~/.ssh` - check key permissions (folder 700, private key 600)
6. `sudo journalctl -u ssh --since "10 min ago"` - read server logs

## Slow Network

1. `ping -c 50 <host>` - look for packet loss and high latency
2. `mtr -rw -c 50 <host>` - find which hop is slow
3. `ip -s link show eth0` - look for errors and drops
4. `sudo ethtool eth0` - check speed and duplex (look for a mismatch)
5. `iperf3 -c <server-ip>` - measure real bandwidth
6. `top` - check whether CPU is the bottleneck

## VLAN / Trunk Problems (Cisco)

1. `show vlan brief` - is the port in the right VLAN?
2. `show interfaces trunk` - is the VLAN allowed on the trunk?
3. `show interfaces status` - check port state, speed, and duplex
4. `show interfaces status err-disabled` - find ports shut down by errors
5. `show mac address-table vlan <id>` - is the host's MAC learned?
6. Check for a native VLAN mismatch on both ends of the trunk
7. Recover an err-disabled port: `shutdown` then `no shutdown`

## Firewall Blocking Traffic (pfSense / OPNsense)

1. Status > System Logs > Firewall - look for blocked entries (filter by source IP)
2. Diagnostics > States - is there an active state for the connection?
3. Firewall > Rules - rules run top to bottom, first match wins
4. Check the rules on the correct interface (traffic is filtered where it enters)
5. Diagnostics > Packet Capture - confirm packets reach the firewall
6. Shell: `pfctl -sr` shows the loaded rules

## Wi-Fi Problems (Linux)

1. `nmcli radio wifi` - is Wi-Fi turned on?
2. `rfkill list` - is it blocked? Fix with `sudo rfkill unblock wifi`
3. `nmcli device wifi list` - do you see the network?
4. `nmcli device wifi connect "<SSID>" password "<password>"` - connect
5. `iw dev wlan0 link` - check signal strength

## Before You Finish

- Save network device configs (`copy running-config startup-config`)
- Write down the cause and the fix
- Test again from the original client
