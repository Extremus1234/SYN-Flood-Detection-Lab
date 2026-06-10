# SYN Flood DoS Detection & Analysis Lab

## Objective
Simulate a SYN flood Denial-of-Service attack in an isolated virtual environment, capture and analyze malicious traffic in Wireshark, and identify Indicators of Compromise mapped to the MITRE ATT&CK framework.

---

## Tools & Technologies
| Tool | Purpose |
|---|---|
| VirtualBox | Hypervisor / isolated lab environment |
| Kali Linux | Attacker VM (192.168.56.104) |
| Kali Linux Clone | Target web server VM (192.168.56.103) |
| Python HTTP Server | Simulated web service on port 80 |
| Metasploit Framework | SYN flood module (auxiliary/dos/tcp/synflood) |
| Wireshark | Packet capture and traffic analysis |
| Firefox | Used to trigger initial HTTP traffic and activate Wireshark capture |

---

## Lab Architecture
- Attacker VM: 192.168.56.104
- Target/Web Server VM: 192.168.56.103
- Both VMs on host-only network in VirtualBox
- No external internet traffic — fully isolated environment

---

## Steps

### 1. Environment Setup
- Deployed two Kali Linux VMs in VirtualBox on a host-only network
- Ran ip a on both machines to confirm distinct IP addresses
- Renamed clone VM to distinguish web server from attacker
- Launched Python HTTP server on target VM:
  python3 -m http.server 80
- Verified HTTP connectivity by browsing to 192.168.56.103 in Firefox

### 2. Baseline Traffic Capture
- Started Wireshark on target VM with tcp display filter
- Confirmed normal HTTP GET requests and responses (200 OK)
- Verified web server responding at port 80

### 3. SYN Flood Attack Execution
- Launched Metasploit on attacker VM:
  use auxiliary/dos/tcp/synflood
  set RHOST 192.168.56.103
  set RPORT 80
  run
- Observed: [*] SYN flooding 192.168.56.103:80
- Note: Had to re-enter command and trigger HTTP traffic via Firefox before Wireshark began capturing flood traffic

### 4. Traffic Analysis in Wireshark
- Applied tcp display filter
- Observed mass TCP Retransmission packets flooding the capture
- Identified incomplete three-way handshakes — SYN with no ACK response

| Traffic Type | Observation |
|---|---|
| TCP Retransmissions | Dominated capture — server resource exhaustion |
| Legitimate HTTP | Minimal — 404 GET / HTTP/1.1 visible between flood packets |
| SYN packets | High
