### Speedtest base Failover:
In this scenario we will have one Mikrotik located on Internet with address of speed-server.4cloud4.com, and the other Mikrotik Public IP should be allowed to access the bandwidth server module of this Mikrotik over the internet. The Mikrotik which is our gateway will check the connectivity to 8.8.8.8 and 4.2.2.4 first and then check the speed test to speed-server.4cloud4.com, and set the priority base on the result.

### Solution:
Here is a step-by-step guide to configure your Mikrotik routers to achieve failover based on ping and bandwidth test results. This configuration will include setting up the routes with different distances and using a script to automate the testing and adjustment of the routes.

1.  **Initial Setup:**
    
    -   Connect each WAN interface to its respective modem.
    -   Assign IP addresses to each WAN interface.
    
2.  **Create Routes with Different Distances:**
    
    -   Create routes for each WAN with different distances.
    
    ```
    /ip route
    add dst-address=0.0.0.0/0 gateway=10.0.1.1 distance=1 check-gateway=ping
    add dst-address=0.0.0.0/0 gateway=10.0.2.1 distance=2 check-gateway=ping
    add dst-address=0.0.0.0/0 gateway=10.0.3.1 distance=3 check-gateway=ping
    ```
    
    
3.  **Script to Perform Ping and Bandwidth Test:**
    
    -   Create a script that will ping 8.8.8.8 and 4.2.2.4, perform bandwidth tests, and adjust route distances based on the results.

```
/system script
add name=updateRoutes script="
:local ping1 [:ping 8.8.8.8 interface=WAN1 count=3];
:local ping2 [:ping 8.8.8.8 interface=WAN2 count=3];
:local ping3 [:ping 8.8.8.8 interface=WAN3 count=3];

:local result1 0;
:local result2 0;
:local result3 0;

:foreach i in=$ping1 do={:set result1 ($result1 + $i);}
:foreach i in=$ping2 do={:set result2 ($result2 + $i);}
:foreach i in=$ping3 do={:set result3 ($result3 + $i);}

:if ($result1 < $result2 && $result1 < $result3) do={
    /ip route set [find gateway=10.0.1.1] distance=1
    /ip route set [find gateway=10.0.2.1] distance=2
    /ip route set [find gateway=10.0.3.1] distance=3
} else {
    :if ($result2 < $result1 && $result2 < $result3) do={
        /ip route set [find gateway=10.0.2.1] distance=1
        /ip route set [find gateway=10.0.1.1] distance=2
        /ip route set [find gateway=10.0.3.1] distance=3
    } else {
        :if ($result3 < $result1 && $result3 < $result2) do={
            /ip route set [find gateway=10.0.3.1] distance=1
            /ip route set [find gateway=10.0.1.1] distance=2
            /ip route set [find gateway=10.0.2.1] distance=3
        }
    }
}

:local speed1 [:tool bandwidth-test address=speed-server.4cloud4.com user=admin password=yourpassword interface=WAN1 duration=10s];
:local speed2 [:tool bandwidth-test address=speed-server.4cloud4.com user=admin password=yourpassword interface=WAN2 duration=10s];
:local speed3 [:tool bandwidth-test address=speed-server.4cloud4.com user=admin password=yourpassword interface=WAN3 duration=10s];

:if ($speed1 < $speed2 && $speed1 < $speed3) do={
    /ip route set [find gateway=10.0.1.1] distance=1
    /ip route set [find gateway=10.0.2.1] distance=2
    /ip route set [find gateway=10.0.3.1] distance=3
} else {
    :if ($speed2 < $speed1 && $speed2 < $speed3) do={
        /ip route set [find gateway=10.0.2.1] distance=1
        /ip route set [find gateway=10.0.1.1] distance=2
        /ip route set [find gateway=10.0.3.1] distance=3
    } else {
        :if ($speed3 < $speed1 && $speed3 < $speed2) do={
            /ip route set [find gateway=10.0.3.1] distance=1
            /ip route set [find gateway=10.0.1.1] distance=2
            /ip route set [find gateway=10.0.2.1] distance=3
        }
    }
}
```

4. **Schedule the Script:**
   - Schedule the script to run at regular intervals to ensure the routes are always optimized.
   ```
   /system scheduler
   add name=updateRoutesSchedule interval=5m on-event=updateRoutes
   ``` 

This setup will periodically check the ping times and bandwidth of each WAN interface and adjust the routing distances accordingly, ensuring the best possible performance and failover configuration. Adjust the script and intervals as needed to fit your specific requirements.
