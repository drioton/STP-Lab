# Spanning Tree Projects (STP, RSTP, PVST+, MSTP)
![STP](images/1.STP.png)
## Overview
We will build a network with three switches connected in a triangle topology and configure different Spanning Tree Protocol versions:

1. **STP (Standard Spanning Tree Protocol)** – Prevents loops but has slow convergence.\
2. **RSTP (Rapid Spanning Tree Protocol)** – Faster recovery time than STP.\
3. **PVST+ (Per VLAN Spanning Tree)** – Separate STP instances for each VLAN.\
4. **MSTP (Multiple Spanning Tree Protocol)** – Groups VLANs into instances for efficiency.



## Project 1: Basic STP Configuration
**Objective**\
Set up Spanning Tree Protocol (STP) to prevent loops in a simple network.

**Topology**:
3 Cisco 2960 switches (SW1, SW2, SW3) connected in a triangle.\
Each switch has trunk ports connecting to the other two.

**Basic Switch Configuration**

```
enable
configure terminal
hostname SW1(2,3)
no ip domain-lookup
line console 0
logging synchronous
exec-timeout 20
end
copy running-config startup-config
```

**Configuration Steps:**

**Step 1: Configure Basic Network**
Connect SW1, SW2, SW3 using trunk links (e.g., Fa0/1 and Fa0/2).\
Assign a management IP (optional).

**Step 2 Configure Trunk Ports on All Switches**
```
Switch(config)# interface range fa0/1 - 2
Switch(config-if-range)# switchport mode trunk
Switch(config-if-range)# exit

Switch(config)# interface range fa0/3 - 4
Switch(config-if-range)# switchport mode trunk
Switch(config-if-range)# exit

```

**Step 3: Enable STP and Verify**
Check if STP is running:

```
Switch# show spanning-tree
Switch# show spanning-tree int f0/1 (2,3,4)
Switch# show spanning-tree detail
```
If not enabled, activate it:
```
Switch(config)# spanning-tree mode pvst
```
**Step 4: Set Root Bridge (SW1 as Root)**
```
SW1(config)# spanning-tree vlan 1 priority 4096
```
(The lowest priority switch becomes the root bridge.)

**Step 5: Verify STP Operation**
```
SW1# show spanning-tree
SW1# show spanning-tree int f0/1
```

**Set Root Bridge SW3 as Root**
```
SW3(config)#spanning-tree vlan 1 priority 0
```
**Verify STP Operation**
```
SW3# show spanning-tree
SW3# show spanning-tree int f0/1
SW3# show spanning-tree int f0/2
SW3# show spanning-tree int f0/3
SW3# show spanning-tree int f0/4
```
**Set No Root Bridge SW3 as Root**
```
SW3#spanning-tree vlan 1 priority 32768
```



+ One port should be in blocking state to prevent loops.
+ If you disconnect an active link, the blocked port should become active.


# Verifying STP (Standard Spanning Tree Protocol)

**Check if STP is running and which switch is the Root Bridge**
```
Switch# show spanning-tree
```

+ Verify the Root Bridge for VLAN 1.
+ Look for Blocking and Forwarding ports.

**Check the STP state of a specific interface**
```
Switch# show spanning-tree interface fa0/1
```

See if the port is in Forwarding, Blocking, or Listening state.

**Test STP failover**
+ Physically disconnect one trunk link and check if a Blocked port becomes active.
+ Run the command again to verify:
```
Switch# show spanning-tree
```

# Verifying Spanning Tree with End Devices

This setup includes **PCs connected to access ports**, allowing you to configure **PortFast and additional STP security features.**

## 1. Enabling PortFast and Testing It

Disable all unused ports (except 1–6):

Go to global configuration mode:

bash
Copy
Edit
Switch# configure terminal
Disable unused ports (let’s assume ports 7–24 are unused):

bash
Copy
Edit
Switch(config)# interface range fa0/7 - 24
Switch(config-if-range)# shutdown
Configure trunk ports (fa0/1–fa0/4):

For the trunk ports between switches, you need to enable Root Guard to prevent any non-root bridge from becoming the Root Bridge:

bash
Copy
Edit
Switch(config)# interface range fa0/1 - 4
Switch(config-if-range)# spanning-tree root-guard
Switch(config-if-range)# exit
Configure end device ports (fa0/5–fa0/6):

For the end device ports, you can enable PortFast so that the ports immediately transition to the Forwarding state:

bash
Copy
Edit
Switch(config)# interface range fa0/5 - 6
Switch(config-if-range)# spanning-tree portfast
Switch(config-if-range)# exit
Verify your configuration:

To check the status of Root Guard on the trunk ports:

bash
Copy
Edit
Switch# show spanning-tree interface fa0/1
To check that PortFast is enabled on the end device ports:

bash
Copy
Edit
Switch# show spanning-tree interface fa0/5
Save your configuration (if everything looks good):

bash
Copy
Edit
Switch# write memory
What will happen:
Trunk ports (fa0/1–fa0/4) will have Root Guard enabled, so if any non-root switch tries to become the Root Bridge, the port will be placed into an errdisable state.
End device ports (fa0/5–fa0/6) will have PortFast enabled, causing them to quickly transition to the Forwarding state, which is useful for devices like computers, printers, etc.
Unused ports (fa0/7–fa0/24) will be shut down to prevent accidental connections.
Summary:
Trunk ports (fa0/1–fa0/4): Apply Root Guard to prevent another switch from becoming the Root Bridge.
End device ports (fa0/5–fa0/6): Apply PortFast to speed up the transition of those ports to the Forwarding state.
Unused ports: Shut them down to keep them inactive.
Let me know if you need further clarification!
PortFast allows an access port to transition immediately to the Forwarding state without going through STP's normal process (Blocking → Listening → Learning → Forwarding).

**Configuration**

Apply PortFast on access ports connected to PCs:

```
Switch(config)# interface fa0/2
Switch(config-if)# spanning-tree portfast
Switch(config-if)# end
```
+ Do this for every PC port.

**Verification**

Check if PortFast is enabled:

```
Switch# show spanning-tree interface fa0/2 portfast
```

**Testing PortFast**
1. Disconnect and reconnect the PC cable.
2. Run:
```
Switch# show spanning-tree interface fa0/2
```
+ The port should immediately transition to Forwarding (instead of waiting 30 seconds).

## 2. Enabling BPDU Guard and Testing It

BPDU Guard **disables the port if it receives BPDU packets**, preventing accidental switch connections on access ports.

**Configuration**

Enable BPDU Guard on all **PortFast ports:**

```
Switch(config)# interface fa0/2
Switch(config-if)# spanning-tree bpduguard enable
Switch(config-if)# end
```

**Verification**

Check if BPDU Guard is enabled:
```
Switch# show spanning-tree interface fa0/2 detail
```
+ Look for BPDU Guard: **Enabled**.


**Testing BPDU Guard**

1. **Connect another switch to fa0/2** instead of a PC.
2. The port should immediately **shut down** upon receiving BPDU packets.
3. Check the error:
```
Switch# show interfaces status err-disabled
```

4. **Reactivate the port** manually:
```
Switch(config)# interface fa0/2
Switch(config-if)# shutdown
Switch(config-if)# no shutdown
```

## 3. Enabling Root Guard and Testing It

Root Guard prevents unauthorized switches from becoming the **Root Bridge.**

**Configuration**
Apply Root Guard on designated ports facing potential unauthorized switches:
```
Switch(config)# interface fa0/3
Switch(config-if)# spanning-tree guard root
Switch(config-if)# end
```
**Verification**
Check if Root Guard is enabled:
```
Switch# show spanning-tree interface fa0/3 detail
```
+ Look for Root Guard: **Enabled.**


**Testing Root Guard**
1. Set another switch to a lower priority so it tries to become the Root Bridge:
```
Switch(config)# spanning-tree vlan 1 priority 0
```
2. The port should go into "Root Inconsistent" state and block traffic.
3. Verify the status:
```
Switch# show spanning-tree inconsistentports
```
## Enabling Loop Guard and Testing It
Loop Guard prevents unidirectional link failures from causing **STP loops.**

**Configuration**
Apply Loop Guard on trunk ports connecting switches:
```
Switch(config)# interface fa0/24
Switch(config-if)# spanning-tree guard loop
Switch(config-if)# end
```

**Verification**
Check if Loop Guard is enabled:
```
Switch# show spanning-tree interface fa0/24 detail
```
+ Look for Loop Guard: **Enabled.**

**Testing Loop Guard**
1. **Disconnect one side of the trunk link.**
2. The switch should detect an inconsistent state and block the port.
3. Verify the status:
```
Switch# show spanning-tree inconsistentports
```
4. Reconnect the trunk cable, and the port should automatically recover.


###Final Notes
1. These configurations improve STP security and network stability.
2. Run the verification commands after configuring each feature.
3. Now, test the failover scenarios to see STP in action!

## Upgrade to RSTP

**Objective:**
Convert STP to **RSTP** for faster convergence.

Configuration Steps:
Step 1: Change STP Mode to RSTP (on all switches)
```
Switch(config)# spanning-tree mode rapid-pvst
```

**Step 2: Verify RSTP Operation**
Check if RSTP is running:
```
Switch# show spanning-tree
```
+ Ports should now be in discarding/learning/forwarding states instead of blocking/listening/forwarding (STP).
+ Test by disconnecting a link and checking if **RSTP recovers faster than STP**.

### Verifying RSTP (Rapid Spanning Tree Protocol)

**Check if RSTP is running**
```
Switch# show spanning-tree
```
+ Ports should now be in **Discarding / Learning / Forwarding instead of Blocking / Listening / Forwarding** (like in STP).

**Verify RSTP state on a specific interface**
```
Switch# show spanning-tree interface fa0/1
```
+ Check if the port has switched to Forwarding faster than with standard STP.

**Test RSTP failover**
1. Disconnect one trunk link and measure how fast RSTP recovers (should be near-instant).
2. Run:
```
Switch# show spanning-tree
```
+ The previously discarding port should immediately switch to Forwarding.

## Project 3: Upgrade to PVST+ (Per-VLAN Spanning Tree)

**Objective:**
Run **separate STP instances** per VLAN.

**Configuration Steps:**
**Step 1: Create VLANs on All Switches**
```
Switch(config)# vlan 10
Switch(config)# vlan 20
Switch(config)# vlan 30
Switch(config)# exit
```
**Step 2: Configure Trunk Ports**
```
Switch(config)# interface range fa0/1 - 2
Switch(config-if-range)# switchport mode trunk
Switch(config-if-range)# switchport trunk allowed vlan 10,20,30
Switch(config-if-range)# exit
```
**Step 3: Set Root Bridge for Each VLAN
Set SW1 as root for VLAN 10, SW2 for VLAN 20, and SW3 for VLAN 30:**
```
SW1(config)# spanning-tree vlan 10 priority 4096
SW2(config)# spanning-tree vlan 20 priority 4096
SW3(config)# spanning-tree vlan 30 priority 4096
```
**Step 4: Verify PVST+ Operation**
```
Switch# show spanning-tree vlan 10
Switch# show spanning-tree vlan 20
Switch# show spanning-tree vlan 30
```
Each VLAN should have a different root bridge, optimizing load balancing.

###  Verifying PVST+ (Per-VLAN Spanning Tree)
** Check STP status for a specific VLAN**
```
Switch# show spanning-tree vlan 10
Switch# show spanning-tree vlan 20
Switch# show spanning-tree vlan 30
```
+ Each VLAN should have a different Root Bridge if configured properly.

**Check root bridge per VLAN**
```
Switch# show spanning-tree vlan 10 root
```
+ This should show which switch is the root bridge for VLAN 10.

**Verify trunk VLAN settings**
```
Switch# show interfaces trunk
```
+ Make sure VLANs 10, 20, 30 are allowed on the trunk.

## Project 4: Migrate to MSTP (Multiple Spanning Tree Protocol)
**Objective:**
Use **MSTP** to group VLANs into instances.

**Configuration Steps:**
**Step 1: Enable MSTP on All Switches**
```
Switch(config)# spanning-tree mode mst
```
**Step 2: Define MST Region** (Must be identical on all switches!)
```
Switch(config)# spanning-tree mst configuration
Switch(config-mst)# name MyMST
Switch(config-mst)# revision 1
Switch(config-mst)# instance 1 vlan 10,20
Switch(config-mst)# instance 2 vlan 30,40
Switch(config-mst)# exit
```
**Step 3: Set Root Bridge for MST Instances**
```
SW1(config)# spanning-tree mst 1 priority 4096
SW2(config)# spanning-tree mst 2 priority 4096
```
**Step 4: Verify MSTP Operation**
```
Switch# show spanning-tree mst
```
+ VLANs 10 and 20 should share MST instance 1.
+ VLANs 30 and 40 should share MST instance 2.

### Verifying MSTP (Multiple Spanning Tree Protocol)

**Check MSTP instance configuration**
```
Switch# show spanning-tree mst
```
+ **Verify:**
1. MST region name
2. Revision number
3. VLAN-to-instance mapping
4. Root bridge per instance

** Check details of each MSTP instance**
```
Switch# show spanning-tree mst detail
```
+ Verify which VLANs belong to which MST instance.
+ Check the port roles (Root, Designated, Alternate, Blocking).

** Check if MSTP switches are in the same region**
```
Switch# show spanning-tree mst configuration
```
+ The Region Name and Revision Number must be the same on all switches.

**Verify VLAN-to-MST instance mapping**
```
Switch# show spanning-tree mst configuration
```
+ Ensure VLANs 10, 20 are in Instance 1, VLANs 30, 40 are in Instance 2.

**Check root bridge for each MST instance**
```
Switch# show spanning-tree mst 1 root
Switch# show spanning-tree mst 2 root
```

+ Each MST instance should have its own Root Bridge.
++Test MSTP failover++
**Disconnect one trunk link** and verify if the **Alternate port switches to Forwarding.**
Run:
```
Switch# show spanning-tree mst
```
+ A previously blocked port should become forwarding.


### Final Steps
Run these verification commands after each configuration to confirm everything is working properly.\
If something isn’t working, check port states, root bridge roles, and VLAN mappings!













