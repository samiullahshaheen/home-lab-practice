# Attack 09 — ARP Poisoning (Bettercap)

## Objective
ARP has no authentication, so an attacker can trick a victim and the
gateway into sending traffic through them instead of each other. This lab
performs a full-duplex ARP poisoning MITM attack and captures plaintext
credentials from intercepted traffic.

## Environment
| Role | Host | IP |
|---|---|---|
| Attacker | Kali Linux | 192.168.249.141 |
| Victim | Windows 10 | 192.168.249.140 |
| Gateway | VMware host-only | 192.168.249.2 |

Tool: bettercap
Target for credential capture: public HTTP login test page (plaintext, no TLS)

## Steps
1. Checked IP/ARP config on victim and attacker.
2. Enabled IP forwarding on Kali:
   `sudo sysctl -w net.ipv4.ip_forward=1`
3. Ran bettercap and poisoned both directions:
   ```
   sudo bettercap -iface eth0
   set arp.spoof.targets 192.168.249.140
   set arp.spoof.fullduplex true
   arp.spoof on
   net.probe on
   net.sniff on
   ```
4. Verified on the victim — `arp -a` showed the gateway's MAC replaced by
   the attacker's MAC.
5. Browsed to an insecure HTTP login page on the victim and submitted
   test credentials.
6. Bettercap captured the full plaintext POST request — username and
   password visible in the clear.
7. Cleaned up: stopped spoofing/sniffing, disabled IP forwarding.

## Why This Is Dangerous
ARP poisoning gives an attacker a silent man-in-the-middle position on
any LAN where it's not blocked — no exploit, no malware, no user
interaction beyond just being on the same network segment. Once in that
position, the attacker sees everything unencrypted crossing the wire:
plaintext HTTP logins, session cookies (enabling session hijacking even
without the password), internal DNS queries revealing what systems a
target talks to, and metadata from HTTPS traffic (SNI, destination IPs)
useful for reconnaissance. It also opens the door to further attacks —
DNS spoofing, SSL stripping, or injecting malicious content into
unencrypted responses. On most real corporate networks this is trivial
to pull off from a single compromised or rogue device, and it leaves
almost no trace unless something is specifically watching Layer 2.

## Remediation
- **Dynamic ARP Inspection (DAI)** on managed switches — validates
  ARP packets against a trusted binding table (DHCP snooping), drops
  spoofed replies at the switch port.
- **Port security / static ARP entries** for critical hosts (servers,
  gateways) where feasible.
- **Network segmentation (VLANs)** to shrink the broadcast domain an
  attacker can poison.
- **Encrypt everything** — HTTPS/TLS everywhere removes the payoff even
  if the MITM position is achieved; plaintext HTTP should not exist on
  a modern network.
- **ARP monitoring tools** (arpwatch, osquery ARP table polling) to
  detect MAC/IP binding changes and alert on anomalies.
- **802.1X port-based authentication** to prevent unauthorized devices
  from joining the network segment in the first place.

This attack was performed entirely within an isolated VMware host-only
lab network with no internet-facing or third-party systems in scope.