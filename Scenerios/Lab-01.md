# Lab 01 - Snooping Byte

## Scenario 🎫

> Several new PCs were recently connected to the network.
>
> The users at Host A and Host C are reporting that they are unable to reach the cloud-hosted timecard system. The system should be reachable at timecards.com; however, users are encountering an error.
>
> Someone has suggested that the issue may be related to DNS.

##### The root cause is unknown at the start of the investigation.

## Lab 01 Topology

<img width="1001" height="613" alt="image" src="https://github.com/user-attachments/assets/518013ab-e30a-47d1-a4ea-ac0e4b4c0c60" />

---

## Troubleshooting Process

### Step 1 - Verify the problem

I started by testing the issue from the affected hosts so I could confirm the problem myself before making any changes.

- Host A -> ping timecards.com -> unsuccessful
- Host C -> ping timecards.com -> unsuccessful

The issue was confirmed.

### Step 2 - Determine the scope

I tested Host B to see if the issue was only affecting the two reported hosts or if it was a larger network problem.

- Host B -> ping timecards.com -> unsuccessful

Since Host B also failed, the issue was affecting all three hosts.

I then tested the timecard server directly by IP address to see if the problem was specifically related to DNS.

- Host A -> ping 214.1.1.10 -> unsuccessful
- Host B -> ping 214.1.1.10 -> unsuccessful
- Host C -> ping 214.1.1.10 -> unsuccessful

Since the server was also unreachable by IP address, DNS did not appear to be the main issue.

### Step 3 - Check internal connectivity

I tested the hosts' default gateways and the DNS server to see if the hosts could communicate normally inside the local network.

- All hosts -> default gateway -> successful
- All hosts -> DNS server -> successful

Internal connectivity appeared to be working.

### Step 4 - Test toward the Internet edge

I tested connectivity to the Edge router to see how far traffic could travel before failing.

- Host A -> Edge router -> successful
- Host B -> Edge router -> successful
- Host C -> Edge router -> successful

Since all three hosts could reach the Edge router, the internal path to the edge of the network appeared to be working.

### Step 5 - Check routing

I used the Admin PC to SSH into the Edge router so I could check the routing table and routing configuration.

I wanted to see if a missing or incorrect route was preventing traffic from reaching the external network.

- Expected routes were present
- Routing appeared correct

At this point, I had confirmed that the hosts could reach the Edge router and the expected routes were present, but the external destination was still unreachable.

## Escalation Point

After checking routing on the Edge router, I got stuck.

I had already confirmed that:

- The hosts could reach their default gateways
- The DNS server was reachable
- The hosts could reach the Edge router
- The expected routes were present

I knew the problem was likely near the Internet edge, but I did not know what to test next.

At this point, I reviewed the Witty Networks answer.

#### Result: Escalated

## Root Cause

The Edge router was using a NAT pool with private IP addresses.

The internal hosts were being translated to addresses in the `192.168.100.0/24` range before traffic was sent toward the simulated Internet.

Because those translated addresses were still private, the traffic could not be routed across the simulated public Internet.

## What I Missed

I missed checking NAT on the Edge router.

After confirming that routing looked correct, I should have asked what else the Edge router needed to do before private internal addresses could communicate with the Internet.

That should have led me to check NAT.

## What I Learned

One troubleshooting question that I want to remember going forward is:

> What must be true for communication to work? 

This would have helped me in this scenario. Once I confirmed that the hosts could reach the Edge router and routing looked correct, I should have thought about what else had to be true for private internal hosts to communicate with the Internet.

That would have helped lead me toward NAT.

Going forward, I want to use this question when I get stuck instead of just guessing what to check next.

## Video Walkthrough

YouTube video coming soon ⌛

## Lab Source

This troubleshooting scenario was created by [WittyNetworks.net](https://wittynetworks.net/).


