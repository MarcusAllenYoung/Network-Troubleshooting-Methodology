# Lab 01 - Snooping Byte

## Scenario 🎫

> Several new PCs were recently connected to the network.
>
> The users at Host A and Host C are reporting that they are unable to reach the cloud-hosted timecard system. The system should be reachable at timecards.com; however, users are encountering an error.
>
> Someone has suggested that the issue may be related to DNS.

##### The root cause is unknown at the start of the investigation.

## Lab 01 Topology

<img width="1283" height="614" alt="Lab 01 Topology" src="https://github.com/user-attachments/assets/015dbd01-437c-4d6b-a170-7ca4a7b2c4a3" />

---

## Troubleshooting Process

### Step 1 - Verify the problem

- Host A -> ping timecards.com -> unsuccessful
- Host C -> ping timecards.com -> unsuccessful

The issue was confirmed.

### Step 2 - Determine the scope

- Host B -> ping timecards.com -> unsuccessful
- All three hosts -> ping 214.1.1.10 -> unsuccessful

Since the server was also unreachable by IP address, DNS did not appear to be the main issue.

### Step 3 - Check internal connectivity

- All hosts -> default gateway -> successful
- All hosts -> DNS server -> successful

Internal connectivity appeared to be working.

### Step 4 - Test toward the Internet edge

- Host A -> Edge router -> successful
- Host B -> Edge router -> successful
- Host C -> Edge router -> successful

The internal path to the Edge router appeared to be working.

### Step 5 - Check routing

I checked the routing table and routing configuration on the Edge router.

- Expected routes were present
- Routing appeared correct

---

## Escalation Point

I got stuck after checking routing on the Edge router.

I had narrowed the issue toward the Internet edge, but I did not think to check the NAT configuration.

At this point, I reviewed the Witty Networks answer.

Result: Escalated

---

## Root Cause

The Edge router was using a NAT pool with private IP addresses.

The internal hosts were being translated to addresses in the `192.168.100.0/24` range before traffic was sent toward the simulated Internet.

Because those translated addresses were still private, the traffic could not be routed across the simulated public Internet.

---

## What I Missed

I missed NAT.

After confirming that the hosts could reach the Edge router and routing looked correct, I should have checked what else the Edge router was doing with outbound traffic.

That should have led me to NAT.

---

## What I Learned

I learned not to stop after narrowing the issue to a device.

I also need to think about the different functions that device is responsible for.

I also learned not to assume the suggested cause is correct. DNS was suggested, but direct IP connectivity also failed.

---

## Video Walkthrough

YouTube video coming soon ⌛

---

## Tool

- Cisco Packet Tracer

---

## Lab Source

This troubleshooting scenario was created by [WittyNetworks.net](https://wittynetworks.net/).


