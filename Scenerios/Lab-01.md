# Lab 01 - Snooping Byte

## Scenario 🎫

Several new PCs were recently connected to the network.

The users at Host A and Host C are reporting that they are unable to reach the cloud-hosted timecard system. The system should be reachable at timecards.com; however, users are encountering an error.

Someone has suggested that the issue may be related to DNS.

The root cause is unknown at the start of the investigation.

## Lab 01 Topology

<img width="1283" height="614" alt="image" src="https://github.com/user-attachments/assets/015dbd01-437c-4d6b-a170-7ca4a7b2c4a3" />

## Reported Symptoms

- Host A cannot reach timecards.com
- Host C cannot reach timecards.com
- The issue was reported after new PCs were connected
- DNS was suggested as a possible cause, but had not been confirmed

## Troubleshooting Process

### Step 1 - Verify the problem

I started by testing the reported issue from Host A and Host C.

- Host A -> ping timecards.com -> unsuccessful
- Host C -> ping timecards.com -> unsuccessful

The issue was confirmed on both hosts.

### Step 2 - Determine the scope

I tested Host B to see if the problem was only affecting Host A and Host C.

- Host B -> ping timecards.com -> unsuccessful

Host B had the same problem, so the issue was affecting all three hosts.

I then tested the timecard server directly by IP address:

`214.1.1.10`

- Host A -> unsuccessful
- Host B -> unsuccessful
- Host C -> unsuccessful

Since the destination also failed when I used the IP address directly, I started thinking that DNS might not be the main issue.

### Step 3 - Check host configuration and internal connectivity

I checked the network settings on each host.

- DNS Server: `172.16.30.250`
- Host A Default Gateway: `10.10.10.1`
- Host B Default Gateway: `10.20.20.1`
- Host C Default Gateway: `10.30.30.1`

I then tested connectivity to the DNS server and each host's default gateway.

- Host A -> DNS server -> successful
- Host A -> default gateway -> successful
- Host B -> DNS server -> successful
- Host B -> default gateway -> successful
- Host C -> DNS server -> successful
- Host C -> default gateway -> successful

At this point, the hosts had working local and internal network connectivity.

The DNS server was reachable, and each host could also reach its default gateway.

### Step 4 - Test the path toward the Internet

I continued testing farther through the network.

- Host A -> Edge router -> successful
- Host B -> Edge router -> successful
- Host C -> Edge router -> successful

Since all three hosts could reach the Edge router, the internal network path appeared to be working.

I continued investigating the Edge router and the path toward the Internet.

### Step 5 - Check routing

I checked the routing table and routing configuration on the Edge router.

The expected routes were present, and the routing configuration appeared correct.

Since the hosts could reach the Edge router and the expected routes were present, I moved on to checking NAT.

### Step 6 - Check NAT

The internal hosts use private IP addresses, so NAT has to be working for them to reach the Internet.

When I checked the NAT configuration, I found that the Edge router was using this NAT pool:

`192.168.100.10 - 192.168.100.50`

The pool was configured with private IPv4 addresses.

Private IPv4 addresses are not publicly routable on the Internet, so the translated traffic would not be able to reach the simulated public network correctly.

## Root Cause

The Edge router was configured to translate the internal hosts using a NAT pool of `192.168.100.10 - 192.168.100.50`.

Because that range uses private IPv4 addresses, the translated traffic could not be routed across the simulated public Internet.

## Resolution

I replaced the incorrect NAT pool configuration with PAT using the Edge router's upstream interface.

1. Removed the NAT statement that was using the NAT pool:

   `no ip nat inside source list NAT_LIST pool NAT_POOL`

2. Configured PAT using the Edge router's upstream interface:

   `ip nat inside source list NAT_LIST interface g0/0/1 overload`

3. Removed the unused NAT pool:

   `no ip nat pool NAT_POOL 192.168.100.10 192.168.100.50 netmask 255.255.255.0`

## Validation

After changing the NAT configuration, I tested the original issue again.

The hosts were now able to reach the timecard system successfully.

<img width="1919" height="257" alt="image" src="https://github.com/user-attachments/assets/1ac964d3-952b-4b4c-8eaf-c941ead1f91d" />

During validation, I also noticed that the scenario says:

`timecards.com`

but the DNS server has:

`timecard.com`

configured.

That was a separate issue I noticed while testing.

## What I Learned

One of the biggest things I learned from this lab was not to assume that the suggested cause is correct.

The scenario suggested DNS, but testing the server directly by IP address also failed. That told me there was a connectivity problem beyond just name resolution.

I also learned how useful successful tests can be.

Being able to reach the default gateways, DNS server, and Edge router helped me narrow the problem down instead of checking random configurations.

This lab also showed me that I need to get better at thinking about what each test tells me before moving to the next command.

That is something I want to improve as I work through more troubleshooting scenarios.

## Video Walkthrough

YouTube video coming soon ⌛

## Tool

- Cisco Packet Tracer

## Lab Source

This troubleshooting scenario was created by [WittyNetworks.net](https://wittynetworks.net/).



