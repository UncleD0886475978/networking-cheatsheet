# Quick Troubleshooting Steps

## No Internet Access
1. `ip addr show` — confirm interface has an IP
2. `ip route` — confirm default gateway exists
3. `ping 8.8.8.8` — test raw connectivity
4. `host github.com` — test DNS

## Can't Reach Internal Host
1. Confirm same VLAN/subnet
2. Check firewall rules between VLANs
3. `traceroute <host>` to find where it drops
