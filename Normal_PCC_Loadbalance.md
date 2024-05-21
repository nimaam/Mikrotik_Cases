# Normal PCC Load balance (base on Source Address)
PCC matcher will allow you to divide traffic into equal streams with the ability to keep packets with specific set of options in one particular stream (you can specify this set of options from src-address, src-port, dst-address, dst-port)

### Theory
PCC takes selected fields from IP header, and with the help of a hashing algorithm converts selected fields into 32-bit value. This value then is divided by a specified _Denominator_ and the remainder then is compared to a specified _Remainder_, if equal then the packet will be captured. You can choose from src-address, dst-address, src-port, dst-port from the header to use in this operation.


### IP Addresses for reference purpose

```
/ip address
add address=192.168.0.1/24 network=192.168.0.0 broadcast=192.168.0.255 interface=Local
add address=192.168.1.2/24 network=192.168.1.0 broadcast=192.168.1.255 interface=WAN1
add address=192.168.2.2/24 network=192.168.2.0 broadcast=192.168.2.255 interface=WAN2
add address=192.168.3.2/24 network=192.168.3.0 broadcast=192.168.3.255 interface=WAN3
add address=192.168.4.2/24 network=192.168.4.0 broadcast=192.168.4.255 interface=WAN4
```

### Add NET ALLOWED users Address list, to make sure only allowed users get internet access. Make sure to modify this as per your requirements, we can use this list later for other management purposes
```
/ip firewall address-list
add address=192.168.0.1-192.168.0.255 list=allowed_users
```

### Accept Connections
```
/ip firewall mangle
add action=accept chain=prerouting in-interface=WAN1
add action=accept chain=prerouting in-interface=WAN2
add action=accept chain=prerouting in-interface=WAN3
add action=accept chain=prerouting in-interface=WAN4
```

### Mangle Section

#### Marking connections for 4 dsl distribution
```
add chain=prerouting dst-address-type=!local in-interface=Local per-connection-classifier=src-address:4/0 action=mark-connection new-connection-mark=WAN1_conn passthrough=yes src-address-list=allowed_users
add chain=prerouting dst-address-type=!local in-interface=Local per-connection-classifier=src-address:4/1 action=mark-connection new-connection-mark=WAN2_conn passthrough=yes src-address-list=allowed_users
add chain=prerouting dst-address-type=!local in-interface=Local per-connection-classifier=src-address:4/2 action=mark-connection new-connection-mark=WAN3_conn passthrough=yes src-address-list=allowed_users
add chain=prerouting dst-address-type=!local in-interface=Local per-connection-classifier=src-address:4/3 action=mark-connection new-connection-mark=WAN4_conn passthrough=yes src-address-list=allowed_users
```

#### Marking Routing Marks to be used by ROUTES Section
```
add chain=prerouting connection-mark=WAN1_conn in-interface=Local action=mark-routing new-routing-mark=to_WAN1
add chain=prerouting connection-mark=WAN2_conn in-interface=Local action=mark-routing new-routing-mark=to_WAN2
add chain=prerouting connection-mark=WAN3_conn in-interface=Local action=mark-routing new-routing-mark=to_WAN3
add chain=prerouting connection-mark=WAN4_conn in-interface=Local action=mark-routing new-routing-mark=to_WAN4
```
### Adding ROUTE for marked routes (done by mangle earlier)
```
/ip route
add dst-address=0.0.0.0/0 gateway=192.168.1.1 routing-mark=to_WAN1 check-gateway=ping
add dst-address=0.0.0.0/0 gateway=192.168.2.1 routing-mark=to_WAN2 check-gateway=ping
add dst-address=0.0.0.0/0 gateway=192.168.3.1 routing-mark=to_WAN3 check-gateway=ping
add dst-address=0.0.0.0/0 gateway=192.168.4.1 routing-mark=to_WAN4 check-gateway=ping
```

### DEFAULT ROUTES, OR Fail over routes , 
just incase in any router goes offline, then these default routes as per distance, will be used as default
```
add dst-address=0.0.0.0/0 gateway=192.168.1.1 distance=1 check-gateway=ping
add dst-address=0.0.0.0/0 gateway=192.168.2.1 distance=2 check-gateway=ping
add dst-address=0.0.0.0/0 gateway=192.168.3.1 distance=3 check-gateway=ping
add dst-address=0.0.0.0/0 gateway=192.168.4.1 distance=4 check-gateway=ping
```
### NAT/MASQUERADE 
the requests going on each interface (used by ROUTES)

```
/ip firewall nat
add chain=srcnat out-interface=WAN1 action=masquerade src-address-list=allowed_users
add chain=srcnat out-interface=WAN2 action=masquerade src-address-list=allowed_users
add chain=srcnat out-interface=WAN3 action=masquerade src-address-list=allowed_users
add chain=srcnat out-interface=WAN4 action=masquerade src-address-list=allowed_users
```
#### DNS Setting
Now Configure DNS server so users can resolve host names using your mikrotik.
```
/ip dns set allow-remote-requests=yes cache-max-ttl=1w cache-size=5000KiB max-udp-packet-size=512 servers=8.8.8.8
```
