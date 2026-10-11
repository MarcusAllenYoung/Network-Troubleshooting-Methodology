# Lab 01 - Snooping Byte

## Scenario 🎫

Several new PCs were recently connected to the network. The users
at hosts A and C are reporting that they are unable to reach the cloud
hosted timecard system. The system should be reachable at timecards.com;
however, users are encountering an error. Someone has suggested that
the issue may be related to DNS. Investigate and resolve the issue, 
if possible.

The root cause is unknown at the start of the investigation.

## Topology

<img width="1283" height="614" alt="image" src="https://github.com/user-attachments/assets/015dbd01-437c-4d6b-a170-7ca4a7b2c4a3" />

## Reported Symptoms

- Host A cannot reach timecards.com
- Host C cannot reach timecards.com
- The issue was reported after new PCs were connected
- DNS has been suggested as a possible cause, but has not been confirmed

## Troubleshooting Process

### Step 1 - Confirm the issue

I started by testing the problem from Host A and Host C.

- Host A -> ping timecards.com -> unsuccessful
- Host C -> ping timecards.com -> unsuccessful

The issue was confirmed on both hosts.

### Step 2 - I gathered more information

I tested Host B to see if the issue was only affecting Host A and Host C.

- Host B -> ping timecards.com -> unsuccessful
- Host B had the same problem, so the issue was affecting all three hosts.

I then tried to reach the timecard server directly by IP address (214.1.1.10)

- All three hosts were unable to reach the server by IP address.

Since the destination also failed when I used the IP address directly, I started thinking that DNS might not be the main issue.

### Step 3 - Check host network configuration

I checked the network settings on each host.

- DNS Server: 172.16.30.250
- Host A Default Gateway: 10.10.10.1
- Host B Default Gateway: 10.20.20.1
- Host C Default Gateway: 10.30.30.1

I then tested connectivity to the DNS server and each host's default gateway.

- Host A -> DNS server -> successful
- Host A -> default gateway -> successful
- Host B -> DNS server -> successful
- Host B -> default gateway -> successful
- Host C -> DNS server -> successful
- Host C -> default gateway -> successful

At this point, the hosts had working local and internal network connectivity.

The DNS server was reachable, and the hosts could also reach their default gateways.

### Step 4 - Continue testing up the network

I kept testing farther through the network.

- Host A -> Edge router -> successful
- Host B -> Edge router -> successful
- Host C -> Edge router -> successful

Since all three hosts could reach the Edge router, I continued looking farther upstream.

### Step 5 - Check routing

I checked the routing table and routing configuration.

The routes looked correct and the expected networks were present.

Since routing appeared to be working, I moved on to checking NAT on the Edge router.

### Step 6 - Check NAT

The internal hosts use private IP addresses, so NAT has to be working for them to reach the Internet.

When I checked the NAT configuration, I noticed that the NAT pool was using private IP addresses. Private addresses are not routable on the Internet.

## Root Cause

The NAT pool on the Edge router was configured with private IP addresses.

Because of that, the internal hosts were being translated to private addresses that could not be used to reach the Internet.

## Resolution

I removed the incorrect NAT pool configuration and configured PAT using the Edge router's upstream interface.

#ip nat inside source list NAT_LIST interface g0/0/1 overload

## Validation

After fixing the NAT configuration, the hosts were able to reach the timecard system successfully.

During validation, I also noticed that the scenario says: timecards.com, but the DNS server has: timecard.com configured.

That was a separate issue I noticed while testing.

## What I Learned

One of the biggest things I learned from this lab was not to assume the suggested cause is correct.

The scenario suggested DNS, but testing the server directly by IP address also failed. That told me there was a connectivity problem beyond just name resolution.

I also saw how useful successful tests can be.

Being able to reach the default gateways, DNS server, and Edge router helped me narrow the problem down instead of checking random configurations.

I still need to get better at thinking about what each test tells me before moving to the next command, but this lab helped me start building that process.

## Video Walkthrough

YouTube video coming soon.

## Tool

- Cisco Packet Tracer

## Lab Source

This troubleshooting scenario was created by [WittyNetworks.net](https://wittynetworks.net/).



