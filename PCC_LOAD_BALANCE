To solve the issue with persistent connections for secure websites when using PCC (Per Connection Classifier) in Mikrotik, you can implement a strategy to ensure that once a connection is marked, it stays on the same WAN link. This approach helps maintain persistence for secure connections, such as HTTPS, and avoids issues with websites like YouTube.

Here's a step-by-step guide to achieve this:

1.  **Mark New Connections**: Create a firewall rule to mark new connections with a unique identifier.
2.  **Mark Established and Related Connections**: Ensure that once a connection is marked, all related and established connections follow the same mark.
3.  **Routing Based on Connection Mark**: Create routes that use the connection marks to distribute traffic across multiple WAN links.

Here is a sample configuration for Mikrotik:

### Step 1: Mark New Connections

plaintext

Copy code

```
/ip firewall mangle
add chain=prerouting connection-mark=no-mark in-interface=LAN action=mark-connection new-connection-mark=WAN1_conn passthrough=yes
add chain=prerouting connection-mark=no-mark in-interface=LAN action=mark-connection new-connection-mark=WAN2_conn passthrough=yes 
```
### Step 2: Mark Established and Related Connections

plaintext

Copy code

```
/ip firewall mangle
add chain=prerouting connection-mark=WAN1_conn in-interface=LAN action=mark-routing new-routing-mark=to_WAN1 passthrough=yes
add chain=prerouting connection-mark=WAN2_conn in-interface=LAN action=mark-routing new-routing-mark=to_WAN2 passthrough=yes
``` 

### Step 3: Routing Based on Connection Mark

plaintext

Copy code

```
/ip route
add dst-address=0.0.0.0/0 gateway=<WAN1_Gateway> routing-mark=to_WAN1 check-gateway=ping
add dst-address=0.0.0.0/0 gateway=<WAN2_Gateway> routing-mark=to_WAN2 check-gateway=ping
```

### Example of Configuring PCC

plaintext

Copy code

```
/ip firewall mangle
add chain=prerouting dst-address-type=!local in-interface=LAN per-connection-classifier=both-addresses-and-ports action=mark-connection new-connection-mark=WAN1_conn passthrough=yes
add chain=prerouting dst-address-type=!local in-interface=LAN per-connection-classifier=both-addresses-and-ports action=mark-connection new-connection-mark=WAN2_conn passthrough=yes

add chain=prerouting connection-mark=WAN1_conn in-interface=LAN action=mark-routing new-routing-mark=to_WAN1 passthrough=yes
add chain=prerouting connection-mark=WAN2_conn in-interface=LAN action=mark-routing new-routing-mark=to_WAN2 passthrough=yes

/ip route
add dst-address=0.0.0.0/0 gateway=<WAN1_Gateway> routing-mark=to_WAN1 check-gateway=ping
add dst-address=0.0.0.0/0 gateway=<WAN2_Gateway> routing-mark=to_WAN2 check-gateway=ping
```

### Notes:

-   Replace `<WAN1_Gateway>` and `<WAN2_Gateway>` with the actual gateway IPs of your WAN interfaces.
-   Adjust the configuration to match your specific network setup.

This configuration should ensure that once a connection is established, it remains on the same WAN link, thus preventing issues with HTTPS and other secure connections.
