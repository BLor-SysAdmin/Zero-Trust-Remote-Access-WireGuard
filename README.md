# Cryptographic Remote Access Tunnels & Perimeter Security Engineering

![Homelab Master Network Topology](https://raw.githubusercontent.com/BLor-SysAdmin/HTC-Network-Administration-Labs/main/homelab-topology-blueprint.png)

## Executive Architecture Overview
This security infrastructure portfolio capstone showcases the engineering and deployment of a kernel-integrated, split-tunnel WireGuard VPN solution. In modern system environments, exposing internal administrative panels, server terminals, or hypervisor dashboards directly to the public WAN presents an unacceptable attack surface. This project eliminates that vulnerability by establishing a strict Zero Trust perimeter. Outside clients are completely blocked from seeing internal infrastructure ports unless they possess valid, authenticated cryptographic public keys, allowing for secure road-warrior remote network management over unsecure public cellular connections.

## System Topology Blueprint
*   **Perimeter Gateway Edge:** OPNsense Core Firewall Security Appliance [4.3]
*   **VPN Engine Layer:** Native WireGuard Kernel Implementation (FreeBSD Module Integration) [4.3]
*   **Cryptographic Primitive Protocol:** Curve25519 (Asymmetric Public/Private Key Exchange Pairings)
*   **Network Addressing Scheme:**
    *   **Tunnel Virtual Interface:** `wg0` (Subnet Pool: `10.50.0.1/24`) [4.3]
    *   **Physical Public Listening Endpoint:** UDP Port `51820` (Mapped via Residential Edge NAT)

## Technical Features Implemented
1.  **Kernel-Integrated Cryptographic Handshaking:** Deployed WireGuard directly into the firewall core engine, utilizing asymmetric key pairs to enforce strict identity validation before any transit routing occurs [4.3].
2.  **Stateful Firewall Group Isolation:** Configured dedicated, top-down security policies inside the OPNsense **`[WireGuard group]`** matrix [4.3]. Applied traffic filtering logic to permit authenticated VPN packets to cross internal subnet switches to reach server segments (`10.0.1.x`), while keeping internal management ports entirely stealth and invisible to public internet port scanners [4.3].
3.  **Split-Tunnel Routing Architecture:** Programmed mobile endpoint peer routing tables (`AllowedIPs = 10.0.1.0/24`) to direct critical corporate lab traffic through the encrypted tunnel while simultaneously allowing standard public internet traffic to route locally out the phone's native interface, maximizing bandwidth and connection speed.
4.  **Source NAT Outbound Masquerading:** Configured custom outbound NAT traversal profiles across the firewall engine interface, ensuring return packets from internal backend virtual machines cleanly translate back down the tunnel to mobile client clients without causing routing loop drop-offs.

## Secure Perimeter Network Architecture

```text
 [ Mobile Phone Off-Site (Cellular Data) ]
                     │
                     │  (Encrypted UDP Transit - WireGuard Tunnel)
                     ▼
       [ Physical ISP Residential Router ]
                     │  (Port Forwarded UDP 51820 Core Pipe Entry)
                     ▼
          [ OPNsense Firewall Gateway ] ───► Handshake Validated via Cryptographic Keys?
                     │                                         │
                     │                                ┌────────┴────────┐
                     │                                ▼                 ▼
                     │                             [ NO ]             [ YES ]
                     │                       (Packet Deleted)  (Enforces Rule Policies)
                     ▼                                                  │
          [ WireGuard Group Rules ] ◄───────────────────────────────────┘
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
   [ Target: Internet ]    [ Target: Internal Lab ]
 (Routes out Local WAN)   (Allowed IPs: 10.0.1.x Subnets)
                                 │
                                 ├──► http://10.0.1 (Pi-hole NOC Dashboard)
                                 └──► http://10.0.1.189:3000  (Grafana Telemetry Cockpit)
```
