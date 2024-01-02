# SHMEM <--> UDP gateway

For your convenience, there are 2 scripts to set up environment variables:
- Linux: _variables.sh_
- Windows: _variables.bat_

These are the variables they contain, which you should modify according to your system:
- **NDDSHOME**: path to the installation of RTI Connext Professional.
- **NDDS_QOS_PROFILES**: path to the _qos.xml_ file containing QoS profiles.
- **SHMEM_DOMAIN**: the domain for the applications using SHMEM.
- **UDP_DOMAIN**: the domain for the applications using UDP (mainly the DomainParticipant of RTI Routing Service).
- **APPS**: total number of DomainParticipants on the localhost. By default, Connext will try to reach out to the first 5 created applications on SHMEM, therefore, we need to increase that number if there are more than 5 applications.
- **CDS_IP_ADDRESS**: the IP address of the host that contains RTI Cloud Discovery Service.
- **CDS_PORT**: the UDP port that CDS will use.

On Linux, you can use the environment variables by sourcing the file:
```
$ source variables.sh
```

On Windows, simply run it:
```
> variables.bat
```

For convenience, the rest of the README will use the Linux variable sign ($) instead of the Windows variable signs (%%).

## Multicast example

1. On terminal 1, source the variables script and run an RTI DDS Ping publisher on SHMEM acting as a local publisher:
    ```
    $ source variables.sh
    $ $NDDSHOME/bin/rtiddsping -pub -domain $SHMEM_DOMAIN -qosProfile "example_library::shmem_profile"
    ```

2. On terminal 2, source the variables script and run an RTI DDS Ping subscriber on UDP acting as a remote subscriber:
    ```
    $ source variables.sh
    $ $NDDSHOME/bin/rtiddsping -sub -domain $UDP_DOMAIN -qosProfile "example_library::multicast"
    ```

3. At this point, there should be no communication between both applications. They are on different domains and they're using different transports. On terminal 3, on the same host as the SHMEM application, source the variables script and start Routing Service:
    ```
    $ source variables.sh
    $ $NDDSHOME/bin/rtiroutingservice -cfgFile RS_config_multicast.xml -cfgName gateway_SHMEM_and_UDP
    ```

4. The subscriber should now be receiving data. For instance:
    ```
    ...
    Current alive publisher tally is: 1
    rtiddsping, issue received: 0000002
    rtiddsping, issue received: 0000003
    rtiddsping, issue received: 0000004
    rtiddsping, issue received: 0000005
    ...
    ```

5. You can now shutdown the 3 applications with Ctrl+C.


## Multicast-less example (CDS)

1. On terminal 1, source the variables script and run an RTI DDS Ping publisher on SHMEM acting as a local publisher:
    ```
    $ source variables.sh
    $ $NDDSHOME/bin/rtiddsping -pub -domain $SHMEM_DOMAIN -qosProfile "example_library::shmem_profile"
    ```

2. Install the CDS package. For instance: _rti_cloud_discovery_service-7.2.0-host-x64Linux.rtipkg_

3. On terminal 2, source the variables script and start CDS:
    ```
    $NDDSHOME/bin/rticlouddiscoveryservice -cfgFile CDS_config.xml -cfgName cds_all_domains_udpv4
    ```

4. On terminal 3, source the variables script and run an RTI DDS Ping subscriber on UDP acting as a remote subscriber:
    ```
    $ source variables.sh
    $ $NDDSHOME/bin/rtiddsping -sub -domain $UDP_DOMAIN -qosProfile "example_library::no_multicast"
    ```
    
5. At this point, there should be no communication between both applications. They are on different domains and they're using different transports. On terminal 4, on the same host as the SHMEM application, source the variables script and start Routing Service:
    ```
    $ source variables.sh
    $ $NDDSHOME/bin/rtiroutingservice -cfgFile RS_config_with_CDS.xml -cfgName gateway_SHMEM_and_UDP
    ```

6. The subscriber should now be receiving data. For instance:
    ```
    ...
    Current alive publisher tally is: 1
    rtiddsping, issue received: 0000002
    rtiddsping, issue received: 0000003
    rtiddsping, issue received: 0000004
    rtiddsping, issue received: 0000005
    ...
    ```

7. You can now shutdown the 3 applications with Ctrl+C.