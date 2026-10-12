# Lab 02 - Wayward Proctor

## Scenario 🎫

The Admin is unable to reach the intranet webpage hosted on the Server. Investigate and resolve the issue, if possible.

The root cause is unknown at the start of the investigation.

---

## Lab 02 Topology

<img width="983" height="620" alt="image" src="https://github.com/user-attachments/assets/e1636ad3-b2c7-4044-84a3-8985f646fc15" />

## Reported Symptoms

- Admin PC cannot reach the intranet webpage
- Server IP address: `10.10.30.250`
- The issue appears to involve communication between the Admin network and the Server network

---

## Troubleshooting Process

### Step 1 - Verify the problem

I started by testing from the Admin PC.

#### Ping test

- Destination: `10.10.30.250`
- Result: unsuccessful

At first, I thought the failed ping confirmed that the Server was unreachable.

Later, I found that ICMP was being blocked by an ACL on RTR1, so the failed ping did not actually confirm whether the intranet webpage itself was unavailable.

#### Webpage test

I then tested the actual reported service.

- URL: `https://10.10.30.250`
- Result: unsuccessful

This confirmed the reported problem.

<img width="685" height="254" alt="image" src="https://github.com/user-attachments/assets/3e258bda-41f1-47f5-b864-1f3573cde4af" />

---

### Step 2 - Decide where to start

I decided to start at Layer 3 because the issue appeared to involve communication between two different networks.

My first goal was to confirm whether the Admin PC could reach its default gateway.

---

### Step 3 - Check the Admin PC and local connectivity

#### Check IP configuration

Command:

`ipconfig /all`

I reviewed the Admin PC network configuration.

#### Test the default gateway

- Destination: `10.10.20.1`
- Result: successful

#### Conclusion

- The Admin PC could reach RTR1
- Local connectivity from the Admin PC appeared to be working
- I continued investigating Layer 3

---

### Step 4 - Check RTR1

I connected to RTR1 using SSH.

#### Routing check

I reviewed the routing table and routing configuration.

- Expected routes were present
- Routing configuration appeared correct

#### ACL discovery

While checking RTR1, I found an extended ACL named `BLOCK_PING`.

The ACL contained:

`deny icmp any any`

This statement blocks all ICMP traffic.

The ACL was applied on RTR1's upstream-facing interface. Because of that, ICMP traffic returning through that interface could not pass through RTR1 toward the Admin PC. This explained why the Admin PC could not successfully ping the Server.

The ACL also contained separate entries for ICMP echo and echo-reply, but those entries were redundant because `deny icmp any any` already blocked all ICMP traffic.

#### Conclusion

- The failed ping was caused by the ICMP ACL
- The failed ping did not prove that the intranet webpage was unreachable
- IP routing itself appeared to be working
- I continued investigating closer to the Server

---

### Step 5 - Identify SW2

I wanted to check SW2, but I did not know its management IP address.

#### Attempt to use CDP

I tried:

`show cdp neighbors`

SW2 did not appear.

I found that CDP was not enabled globally on RTR1, so I enabled it.

I ran:

`show cdp neighbors`

again.

SW2 still did not appear.

I then checked the running configuration and noticed that CDP was also disabled on the interface connecting RTR1 to SW2.

I enabled CDP on that interface.

#### Mistake

I expected SW2 to appear immediately after enabling CDP.

When it did not appear right away, I initially stopped using that method.

Later, I realized that I had not given CDP enough time to populate the neighbor information.

#### Result

After waiting:

- SW2 appeared as a CDP neighbor
- I was able to identify the IP address needed to connect to SW2

---

### Step 6 - Move back to Layer 2 and check SW2

Since Layer 3 routing appeared correct, I moved back to Layer 2 and checked the switch configuration closer to the Server.

I connected to SW2 using SSH.

#### Check Server-facing port

While reviewing the running configuration, I found:

- Server-facing port VLAN: VLAN 10
- Expected Server VLAN: VLAN 30

The Server belongs to the `10.10.30.0/24` network.

#### Conclusion

The Server-facing interface was assigned to the wrong VLAN.

---

## Root Cause

- The switch port connecting SW2 to the Server was configured as an access port in VLAN 10
- The Server belongs to the `10.10.30.0/24` network
- The Server-facing port should have been assigned to VLAN 30
- The incorrect VLAN assignment prevented the Admin PC from reaching the intranet webpage

---

## Resolution

I changed the Server-facing switch port from VLAN 10 to VLAN 30.

Command:

`switchport access vlan 30`

---

## Validation

I tested the original reported problem again from the Admin PC.

### Webpage test

- URL: `https://10.10.30.250`
- Result: successful

The intranet webpage loaded successfully.

<img width="788" height="248" alt="image" src="https://github.com/user-attachments/assets/472ef072-5618-437e-a7c7-01865a8e0f4f" />

The original problem was resolved.

---

## What I Learned

### Test the actual reported service

The reported problem was that the Admin could not reach the intranet webpage.

I initially used ping to test the Server, but ICMP was being blocked by an ACL.

The failed ping did not prove that the webpage itself was unavailable.

Testing the webpage from the browser was the correct way to confirm the original problem.

### Troubleshooting does not always move in one direction

I started at Layer 3 because the issue looked like a routing or reachability problem.

After checking RTR1 and finding that routing appeared correct, I moved back to Layer 2.

The actual root cause was a Layer 2 VLAN assignment issue on SW2.

### Give discovery protocols time

I almost stopped using CDP because SW2 did not appear immediately.

CDP sends advertisements every 60 seconds by default, so neighbor information may not appear right away after CDP is enabled.

I learned to give CDP time to discover the neighbor before assuming that it is not working.

### Main takeaway

The symptoms of a problem do not always point directly to the layer where the root cause exists.

In this lab:

- The symptom looked like a Layer 3 reachability problem
- Ping was affected by an ACL
- Routing was working
- The actual root cause was at Layer 2

---

## Video Walkthrough

YouTube video coming soon ⌛

---

## Tool

- Cisco Packet Tracer

---

## Lab Source

This troubleshooting scenario was created by [WittyNetworks.net](https://wittynetworks.net/).
