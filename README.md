# Setting Up a Guest Wireless Network in Cisco Packet Tracer

## Overview

This document outlines the setup of a guest wireless network using Cisco Packet Tracer. The goal is to provide internet access to guest users while isolating them from the internal network and presenting a simulated captive portal.

## Topology

                                
                               Router (2911)
                               (Gi0/0 connected to SW Fa0/1)
                                   |
                               Switch (2960-24TT)
                              /      |      \
                             /       |       \
      (VLAN 10) Internal PC (PC-PT) --o (Fa0/3) (Fa0/4) o-- Server (Server-PT) (Captive Portal)
                                               |
                                (VLAN 20)       |
                             (Fa0/2) o---------- Wireless AP (AP-PT)
                                               | (Port 0 connected to SW Fa0/2)
                                               | (Broadcasts "Guest Wifi" SSID)
                                           Guest PC (PC-PT)
                                           (Connects wirelessly to AP-PT)

## Device Models

* **Router:** Cisco 2911
* **Switch:** Cisco 2960-24TT
* **Wireless Access Point:** Generic AP-PT
* **PCs:** Generic PC-PT (One for internal, one for guest)
* **Server:** Generic Server-PT (Captive Portal)


## Configuration Details

### Router (Router1)

enable
configure terminal
hostname Router1
interface GigabitEthernet0/0
no shutdown
exit
interface GigabitEthernet0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
no shutdown
ip access-group GUEST_ACCESS in
ip nat inside
exit
ip dhcp pool Guest_DHCP
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 8.8.8.8 4.4.4.4
lease 1 0 0
exit
ip dhcp excluded-address 192.168.20.1 192.168.20.10
interface GigabitEthernet0/1
no shutdown
ip nat outside
exit
ip access-list extended GUEST_ACCESS
permit tcp any host 192.168.10.10 eq 80
permit tcp any host 192.168.10.10 eq 443
permit udp any eq 53 any eq 53
permit icmp any any
no deny tcp any any eq 80
no deny tcp any any eq 443
permit ip any any
exit
ip access-list extended GUEST_INTERNET_ALLOWED
permit ip 192.168.20.0 0.0.0.255 any
exit
ip nat inside source list GUEST_INTERNET_ALLOWED interface GigabitEthernet0/1 overload
end
write memory

### Switch (Switch0)

enable
configure terminal
hostname Switch0
vlan 10
name Internal
exit
vlan 20
name Guest
exit
interface FastEthernet0/1
switchport mode trunk
exit
interface FastEthernet0/2
switchport mode access
switchport access vlan 20
exit
interface FastEthernet0/3
switchport mode access
switchport access vlan 10
exit
interface FastEthernet0/4
switchport mode access
switchport access vlan 10
exit
end
write memory


### Wireless Access Point (AP-PT)

* **SSID:** `Guest Wifi`
* **Security:** WPA2-PSK
* **Passphrase:** `secure123`
* **VLAN:** 20 (Bridged Mode)

### Captive Portal Server (Server-PT)

* **IP Address:** `192.168.10.10/24`
* **Subnet Mask:** `255.255.255.0`
* **Default Gateway:** `192.168.10.1`
* **DNS Server:** (Optional)
* **HTTP Service:** Enabled with a basic terms of service page (`index.html`).

### Internal PC (PC-PT)

* **IP Address:** `192.168.10.2/24`
* **Subnet Mask:** `255.255.255.0`
* **Default Gateway:** `192.168.10.1`
* **DNS Server:** (Optional)

### Guest PC (PC-PT)

* Obtains IP address via DHCP from the Router (in the `192.168.20.0/24` range).

## Testing

1.  Guest PC connects to the `Guest Wifi` Wi-Fi using the password `secure123`.
2.  Guest PC obtains an IP address in the `192.168.20.0/24` range via DHCP.
3.  Guest PC can ping the Captive Portal Server at `192.168.10.10`.
4.  Guest PC can access the web service on the Captive Portal Server at `http://192.168.10.10`, viewing the terms of service page.
5.  Guest PC can access the simulated internet (e.g., ping `8.8.8.8` or `4.4.4.4`).
6.  Guest PC cannot ping the Internal PC (`192.168.10.2`), demonstrating isolation.

## Conclusion

This setup provides a functional guest wireless network with a simulated captive portal and appropriate access restrictions using VLANs, ACLs, and NAT within the Cisco Packet Tracer environment.

## Example Resources

* Example YouTube URL: [https://youtu.be/eiqm6VrCD7A](https://youtu.be/eiqm6VrCD7A)
