# Copp Manager Redesign Test Plan

## Related documents

| **Document Name** | **Link** |
|-------------------|----------|
| Syslog Source IP HLD | []|


## Overview

SSIP is a feature which allows user to change UDP packet source IP address.
Any configured address can be used for source IP mangling.
This might be useful for security reasons.

SSIP also extends the existing syslog implementation with VRF device and server UDP port configuration support. The feature doesn't change the existing DB schema which makes it fully backward compatible.

### Scope

The test is to verify SSIP take effect after do relevant SSIP configuration   

### Scale / Performance

No scale/performance test involved in this test plan

### Related **DUT** CLI commands
```
User interface:

config
|--- syslog
     |--- add <server_ip> OPTIONS
     |--- del <server_ip>

show
|--- syslog

Options:

config syslog add

-s|--source - source ip address
-p|--port - server udp port
-r|--vrf - vrf device

```

### Supported topology
The test will be supported on ptf32, ptf64, t1 and t2.


### Test cases #1 -  Configure syslog server with VRF/Source:unset/unset
1. Configure syslog sever with VRF/Source:unset/unset like below:
```
config syslog add '2.2.2.2' 

```
2. Check syslog config by show syslog, the result should like below:
```
# show syslog
SERVER      SOURCE      PORT    VRF
----------  ----------  ------  --------
2.2.2.2     N/A     N/A     N/A
```
3. Check the corresponding interface of rsyslog sever can receive the corresponding syslog message with port 514
4. Del syslog server config like below:
```
config syslog del '2.2.2.2' 

```
4. Check syslog config by show syslog, the relevant config should be removed
5. Check the corresponding interface of rsyslog sever can not receive any corresponding syslog message with port 514
6. repeat steps 1-5 by setting different port, and check the port of received syslog message will be changed to the set one


### Test cases #2 -  Configure syslog server with VRF/Source: unset/set
1. Configure syslog sever with VRF/Source: unset/set like below:
```
config syslog add '2.2.2.2' --source "1.1.1.1"

```
2. Check syslog config by show syslog, the result should like below:
```
# show syslog
SERVER      SOURCE      PORT    VRF
----------  ----------  ------  --------
2.2.2.2     1.1.1.1     N/A     N/A
```
3. Check the corresponding interface of rsyslog sever can receive the corresponding syslog message with port 514 and src ip 1.1.1.1
4. Del syslog server config like below:
```
config syslog del '2.2.2.2' 

```
4. Check syslog config by show syslog, the relevant config should be removed
5. Check the corresponding interface of rsyslog sever can not receive any corresponding syslog message with port 514 and src ip 1.1.1.1
6. repeat steps 1-5 by setting different port, and check the port of received syslog message will be changed to the set one


### Test cases #3 -  Configure syslog server with VRF/Source: set/unset
1. For vrf: Default, mgmt, Vrf-data, Configure syslog sever with VRF/Source: set/unset like below:
```
config syslog add '2.2.2.2' --vrf "Default"
config syslog add '3.3.3.3' --vrf "mgmt"
config syslog add '4.4.4.4' --vrf "Default"

```
2. Check syslog config by show syslog, the result should like below:
```
# show syslog
SERVER      SOURCE      PORT    VRF
----------  ----------  ------  --------
2.2.2.2       N/A        N/A     Default
3.3.3.3       N/A        N/A     mgmt
4.4.4.4       N/A        N/A     Vrf-data

```
3. Check the corresponding interface of rsyslog sever can receive the corresponding syslog message with port 514
4. Del syslog server config like below:
```
config syslog del '2.2.2.2'
config syslog del '3.3.3.3'
config syslog del '4.4.4.4' 

```
4. Check syslog config by show syslog, the relevant config should be removed
5. Check the corresponding interface of rsyslog sever can not receive any corresponding syslog message with port 514
6. repeat steps 1-5 by setting different port, and check the port of received syslog message will be changed to the set one


### Test cases #4 -  Configure syslog server with VRF/Source: set/set
1. For vrf: Default, mgmt, Vrf-data, Configure syslog sever with VRF/Source: set/set like below:
```
config syslog add '2.2.2.2' ---source "1.1.1.1" --vrf "Default"
config syslog add '3.3.3.3' ---source "5.5.5.5" --vrf "mgmt"
config syslog add '4.4.4.4' ---source "6.6.6.6" --vrf "Vrf-data"

```
2. Check syslog config by show syslog, the result should like below:
```
# show syslog
SERVER      SOURCE      PORT    VRF
----------  ----------  ------  --------
2.2.2.2     1.1.1.1      N/A     Default
3.3.3.3     5.5.5.5      N/A     mgmt
4.4.4.4     6.6.6.6      N/A     Vrf-data

```
3. Check the corresponding interface of rsyslog sever can receive the corresponding syslog message with correct port and source ip
4. Del syslog server config like below:
```
config syslog del '2.2.2.2'
config syslog del '3.3.3.3'
config syslog del '4.4.4.4' 

```
4. Check syslog config by show syslog, the relevant config should be removed
5. Check the corresponding interface of rsyslog sever can not receive any corresponding syslog message with correct port and source ip
6. repeat steps 1-5 by setting different port, and check the port of received syslog message will be changed to the set one


### Test cases #5 -  Removing source IP config, check the relevant syslog config will be removed
1. Configure syslog sever with VRF/Source: set/set like below:
```
config syslog add '2.2.2.2' ---source "1.1.1.1" --vrf "Default"

```
2. Check syslog config by show syslog, the result should like below:
```
# show syslog
SERVER      SOURCE      PORT    VRF
----------  ----------  ------  --------
2.2.2.2     1.1.1.1      N/A     Default
```
3. Remove the config of source IP
4. Check the relevant syslog config should be removed
