# Common CLI Snippets

Commands grouped by platform and level: **Beginner**, **Intermediate**, **Advanced**.
Only scan or capture traffic on networks you own or have permission to test.

---

## Linux

### Beginner
- Show IP config: `ip addr show`
- Short IP summary: `ip -br addr`
- Show routing table: `ip route`
- Show default gateway: `ip route | grep default`
- Show interfaces and link state: `ip link show`
- Show your IP addresses only: `hostname -I`
- Test connectivity: `ping -c 4 8.8.8.8`
- Test DNS: `ping -c 4 example.com`
- Show DNS servers: `cat /etc/resolv.conf` or `resolvectl status`
- Show NetworkManager status: `nmcli device status`
- Restart networking: `sudo systemctl restart NetworkManager`
- Bring an interface up or down: `sudo ip link set eth0 up` / `sudo ip link set eth0 down`

### Intermediate
- Listening ports and processes: `sudo ss -tulpn`
- Connection summary: `ss -s`
- Established connections: `ss -tn state established`
- Trace the path: `traceroute example.com` or `tracepath example.com`
- Live path and loss stats: `mtr example.com`
- DNS lookup: `dig example.com +short`
- Query a specific DNS server: `dig @8.8.8.8 example.com`
- Reverse DNS lookup: `dig -x 8.8.8.8`
- Other record types: `dig example.com MX`, `dig example.com TXT`
- Show ARP/neighbor table: `ip neigh show`
- Check a TCP port: `nc -zv 10.0.10.5 22`
- HTTP headers only: `curl -I https://example.com`
- Verbose HTTP/TLS debug: `curl -v https://example.com`
- Interface errors and drops: `ip -s link show eth0`
- Link speed and duplex: `sudo ethtool eth0`
- Which process uses a port: `sudo lsof -i :80`
- NetworkManager logs: `journalctl -u NetworkManager --since "1 hour ago"`
- Set a temporary IP: `sudo ip addr add 10.0.10.50/24 dev eth0`
- Add a static route: `sudo ip route add 10.0.20.0/24 via 10.0.10.1`
- Apply Netplan safely (auto-rollback): `sudo netplan try`
- Apply Netplan: `sudo netplan apply`

### Advanced
- Capture DNS traffic: `sudo tcpdump -i eth0 -nn port 53`
- Capture one host to a file: `sudo tcpdump -i eth0 host 10.0.10.5 -w capture.pcap`
- Capture only TCP SYN packets: `sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'`
- Read a capture: `tcpdump -nn -r capture.pcap`
- Which route the kernel picks: `ip route get 8.8.8.8`
- Policy routing rules: `ip rule show`
- Show firewall rules (nftables): `sudo nft list ruleset`
- Show firewall rules (iptables): `sudo iptables -L -n -v --line-numbers`
- NAT table (iptables): `sudo iptables -t nat -L -n -v`
- Check IP forwarding: `sysctl net.ipv4.ip_forward`
- Enable IP forwarding: `sudo sysctl -w net.ipv4.ip_forward=1`
- Connection tracking table: `sudo conntrack -L`
- TCP details per connection: `ss -ti`
- Create a VLAN interface:
  - `sudo ip link add link eth0 name eth0.10 type vlan id 10`
  - `sudo ip addr add 10.0.10.2/24 dev eth0.10`
  - `sudo ip link set eth0.10 up`
- Show bridges and MAC table: `bridge link` / `bridge fdb show`
- Bandwidth test: `iperf3 -s` (server), `iperf3 -c <server-ip>` (client)
- Host discovery (ping sweep): `nmap -sn 10.0.10.0/24`
- Check specific ports: `nmap -Pn -p 22,80,443 10.0.10.5`
- Service and version detection: `nmap -sV -p 1-1024 10.0.10.5`
- Follow the whole DNS chain: `dig +trace example.com`

---

## Cisco IOS

### Beginner
- Enter privileged mode: `enable`
- Enter config mode: `configure terminal`
- Show interfaces: `show ip interface brief`
- Show VLANs: `show vlan brief`
- Show running config: `show running-config`
- Show saved config: `show startup-config`
- Save config: `copy running-config startup-config`
- Show version, uptime, and model: `show version`
- Show interface details and errors: `show interfaces GigabitEthernet0/1`
- Show logs: `show logging`
- Test connectivity: `ping 10.0.10.1` / `traceroute 10.0.10.1`

### Intermediate
- Show port status (switch): `show interfaces status`
- Show trunk ports: `show interfaces trunk`
- Show MAC address table: `show mac address-table`
- Show routing table: `show ip route`
- Show ARP table: `show ip arp`
- Show connected devices (CDP): `show cdp neighbors detail`
- Show connected devices (LLDP): `show lldp neighbors`
- Show ACLs: `show access-lists`
- Show DHCP leases: `show ip dhcp binding`
- Show NTP status: `show ntp status`
- Show only lines you want: `show running-config | section interface`
- Filter output: `show ip route | include 10.0`

### Advanced
- Spanning tree: `show spanning-tree` / `show spanning-tree vlan 10`
- EtherChannel: `show etherchannel summary`
- Port security: `show port-security` / `show port-security interface Gi0/1`
- OSPF neighbors: `show ip ospf neighbor`
- OSPF database: `show ip ospf database`
- Routing protocols in use: `show ip protocols`
- EIGRP neighbors: `show ip eigrp neighbors`
- BGP summary: `show ip bgp summary`
- NAT translations: `show ip nat translations`
- NAT stats: `show ip nat statistics`
- Interface counters: `show interfaces counters errors`
- Live debug output in SSH session: `terminal monitor`
- Debug (use with care in production): `debug ip ospf adj`
- Stop all debugging: `undebug all`

### Config examples

Create a VLAN and set access and trunk ports:

```
vlan 10
 name USERS
!
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
!
interface GigabitEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
```

Inter-VLAN routing with an SVI:

```
interface vlan 10
 ip address 10.0.10.1 255.255.255.0
 no shutdown
```

Static default route:

```
ip route 0.0.0.0 0.0.0.0 10.0.0.1
```

OSPF:

```
router ospf 1
 network 10.0.10.0 0.0.0.255 area 0
```

Extended ACL:

```
ip access-list extended USERS-IN
 permit tcp 10.0.10.0 0.0.0.255 any eq 443
 permit udp 10.0.10.0 0.0.0.255 any eq 53
 deny ip any any log
!
interface vlan 10
 ip access-group USERS-IN in
```

NAT overload (PAT):

```
access-list 1 permit 10.0.10.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/0 overload
!
interface GigabitEthernet0/1
 ip nat inside
interface GigabitEthernet0/0
 ip nat outside
```

Port security:

```
interface GigabitEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
```

SSH access (replace the placeholder password):

```
hostname SW1
ip domain-name lab.local
crypto key generate rsa modulus 2048
username admin secret <choose-a-strong-password>
line vty 0 4
 transport input ssh
 login local
```

---

## pfSense / OPNsense

### Web UI paths

pfSense
- Check firewall log: Status > System Logs > Firewall
- Live states table: Diagnostics > States
- Packet capture: Diagnostics > Packet Capture
- Ping and traceroute: Diagnostics > Ping / Traceroute
- DNS lookup: Diagnostics > DNS Lookup
- Interface status: Status > Interfaces
- DHCP leases: Status > DHCP Leases
- Firewall rules: Firewall > Rules
- NAT rules: Firewall > NAT
- Gateway status: Status > Gateways
- Backup config: Diagnostics > Backup & Restore

OPNsense
- Live firewall log: Firewall > Log Files > Live View
- Firewall rules: Firewall > Rules
- NAT rules: Firewall > NAT
- Packet capture: Interfaces > Diagnostics > Packet Capture
- Ping and traceroute: Interfaces > Diagnostics > Ping / Traceroute
- DHCP leases: Services > DHCPv4 > Leases
- Backup config: System > Configuration > Backups

### Shell (SSH or console)
- Interface config: `ifconfig`
- Routing table: `netstat -rn`
- Listening sockets: `sockstat -4 -l`
- Show firewall rules loaded: `pfctl -sr`
- Show NAT rules: `pfctl -sn`
- Show state table: `pfctl -ss`
- Show firewall counters: `pfctl -si`
- Show a table (for example blocked hosts): `pfctl -t <tablename> -T show`
- Capture traffic: `tcpdump -ni em0 host 10.0.10.5`
- Watch firewall log live (pfSense): `tail -f /var/log/filter.log`
- Watch firewall log live (OPNsense): `tail -f /var/log/filter/latest.log`
- Ping from a specific interface: `ping -S 10.0.10.1 10.0.10.5`

---

## Windows (quick reference)

- Full IP config: `ipconfig /all`
- Clear DNS cache: `ipconfig /flushdns`
- Renew DHCP: `ipconfig /release` then `ipconfig /renew`
- Routing table: `route print`
- ARP table: `arp -a`
- Active connections with process ID: `netstat -ano`
- Trace route: `tracert example.com`
- DNS lookup: `nslookup example.com`
- Test a TCP port (PowerShell): `Test-NetConnection 10.0.10.5 -Port 443`
- IP addresses (PowerShell): `Get-NetIPAddress`

---

## Troubleshooting Order (bottom to top)

1. **Physical / link:** cable, `ip link show`, `show interfaces status`
2. **IP address:** `ip addr show`, `show ip interface brief`
3. **Gateway:** `ping <default-gateway>`
4. **Routing:** `ip route`, `show ip route`, `traceroute`
5. **DNS:** `dig example.com`, then `ping 8.8.8.8` to separate DNS from routing
6. **Firewall / ACL:** `nft list ruleset`, `show access-lists`, firewall logs
7. **Service:** `ss -tulpn`, `nc -zv <host> <port>`, `curl -v`
