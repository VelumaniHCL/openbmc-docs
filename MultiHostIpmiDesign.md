# Multi-host IPMI design

Author:
  Velumani T(velu),  [velumanit@hcl](mailto:velumanit@hcl.com)
  Kumar T(kumar_t), [thangavel.k@hcl.com](mailto:thangavel.k@hcl.com)

Primary assignee:
 
Other contributors:
 
Created:
 June 26, 2020

## Problem Description
The current version of openbmc does not support multi-host implementation in ipmi 
commands handling. We have a multi-host system and proposing the design to 
support multi-host.

As detailed below the hosts are connected in the ipmb interface, all host 
related communication is based on ipmb. The openbmc uses ipmbbridged to manage 
ipmb busses and the ipmb messages are routed to ipmid.

Issue 1: ipmbridged does not support more than 2 channels
Issue 2: ipmid does not have the information on which ipmb channel the request 
has come from. The ipmid handlers should have the host details to fetch the 
host specific responses.

## Background and References
IPMI and IPMB System architecture:
       
       +-----------+       +------------+      +--------+
       |           |       |            | ipmb1|        |
       |           |       |            |------| Host-1 |
       |           |       |            |      |        |
       |           |       |            |      +--------+
       |           |       |            |
       |           |       |            |                   
       |           | dbus  |            |      +--------+   
       | ipmid     |-------| Ipmbbridged| ipmb2|        |   
       |           |       |            |------| Host-2 |   
       |           |       |            |      |        |   
       |           |       |            |      +--------+   
       |           |       |            |
       |           |       |            |                   
       |           |       |            |      +--------+   
       |           |       |            | ipmb |        |   
       |           |       |            |------| Host-N |   
       |           |       |            |      |        |   
       +-----------+       +------------+      +--------+   
Hosts are connected with ipmb interface, the hosts can be 1 to N. The ipmb 
request coming from the hosts are routed to ipmid by the ipmbbridged.
The ipmd requests are routed from ipmid or any service by d-bus interface and
the ipmbbridged routes to ipmb interface.
## Requirements
The current version of openbmc does not support multi-host implementation in ipmi 
commands handling. We have a multi-host system and proposing the design to 
support multi-host.

## Proposed Design

To address issue1 and issue2, we propose the following design changes in 
ipmbbridged and ipmid.

Changes in ipmbbridged:
-----
ipmbbridged to send the channel details from where the request is received

**Change : Sending Host detail as additional parameter**

While routing the ipmb requests coming from the host channel,
We will be adding new entry in the 
 the ipmbbridged 
adds the ipmb bus details configured in the json file key "type". 
In the above example the ipmb request coming from "/dev/ipmb-4" the ipmb will 
send "host2" as optional parameter in the d-bus interface to ipmid.

Changes in ipmid:
--------
Receive the optional parameter sent by the ipmbbriged as host details, while 
receiving the parameter in the executionEntry method call the same will be 
passed to the command handlers in both common and oem handlers.
The command handlers can make use of the host information to fetch host 
specific data.

For example, host1 send a request to get boot order from bmc, bmc maintains 
data separately for each host. When this command comes to ipmid the commands 
handler gets the host in which the command received. The handler will fetch
host1 boot order details and respond from the command handler. This is 
applicable for both common and oem handlers.


## Alternatives Considered
NA

## Impacts
There may be an impact in ipmid command handler functions as the context will be  sent as parameter.

## Testing

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI5NzQ3MDMxNiwyMDY1MjE0NDM4LC0xOT
g3NDU4NjkzXX0=
-->