### Mikrotik Failover behind Modem
If the Mikrotik connect to WAN1 on interface ether2-WAN1 and WAN2 on ether3-WAN2 via the Modem in front of it. The IP address of the ether2-WAN1 is 10.0.1.2/24 and the gateway for it is 10.0.1.1 which is the IP of modem of WAN1. Also the IP on ether3-WAN2 is 10.0.2.2/24 and the gateway for this connection is 10.0.2.1 which is IP of modem of WAN2, so in this scenario we need solution to do the failover in case of disconnection to internet.
In normal failover and setting we do this :
```
/ip route 
add dst=0.0.0.0/0 gateway=10.0.1.1 check=ping
add dst=0.0.0.0/0 gateway=10.0.2.1 check=ping
```

### Problem
The Mikrotik failover configuration described above will only switch to WAN2 if the physical link to the modem on WAN1 is down. However, if the modem is connected to the Mikrotik but the internet connection through the modem is down, Mikrotik won't automatically switch to WAN2 with the basic route checking setup mentioned. This is because Mikrotik is only checking the connectivity to the modem (gateway), not the internet connectivity beyond the modem.

### Solution
To handle the scenario where the modem is connected but the internet is down, you need to use a more advanced method to monitor the actual internet connection. One common approach is to use Netwatch or scripts to ping a reliable internet host (e.g., 8.8.8.8 or any other public IP) and then adjust the routes based on the result.

Here’s how you can set up a basic Netwatch to monitor an external IP and switch routes:

1.  **Add a script to switch to WAN2:**

```
/system script add name=failover-to-wan2 source="{
    /ip route set [find where gateway=10.0.1.1] distance=2
    /ip route set [find where gateway=10.0.2.1] distance=1
}"
```

2.  **Add a script to switch back to WAN1:**

```
/system script add name=failover-to-wan1 source="{
    /ip route set [find where gateway=10.0.1.1] distance=1
    /ip route set [find where gateway=10.0.2.1] distance=2
}"
```

3.  **Add a Netwatch entry to monitor an external IP (e.g., 8.8.8.8) via WAN1:**

```
/tool netwatch add host=8.8.8.8 interval=30s timeout=500ms up-script=failover-to-wan1 down-script=failover-to-wan2
```

With this setup, Mikrotik will ping 8.8.8.8 every 30 seconds. If it fails to get a response, it will execute the `failover-to-wan2` script, changing the routing distances to prioritize WAN2. When it starts getting responses again, it will execute the `failover-to-wan1` script to revert to WAN1 as the primary route.

This way, Mikrotik can detect if the internet connection through WAN1 is down, even if the physical link to the modem is still up.
