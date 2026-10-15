# Lab 02 - Wayward Proctor

## Scenario 🎫

> The Admin PC is unable to reach the intranet webpage hosted on the Server.
>
> Investigate the issue and resolve it, if possible.

##### The root cause is unknown at the start of the investigation.

## Lab 02 Topology

<img width="764" height="451" alt="image" src="https://github.com/user-attachments/assets/b58c6729-8c16-4faa-aac1-673a215346cc" />

---

## Troubleshooting Process

### Step 1 - Verify the problem

I first tried to ping the Server at `10.10.30.250` to see if the Admin PC could reach it.

- Admin PC -> ping 10.10.30.250 -> unsuccessful

I initially treated the failed ping as confirmation of the issue, but the reported problem was actually access to the intranet webpage.

I then tested the actual service in the web browser.

<img width="604" height="278" alt="image" src="https://github.com/user-attachments/assets/b8276b8d-9a91-421a-966e-2d9bcd13d58e" />

The reported issue was confirmed.

### Step 2 - Decide where to start

Since the Admin PC and Server were on different networks, I decided to start by checking Layer 3 connectivity.

I wanted to make sure the Admin PC had the correct network configuration and could reach its default gateway before troubleshooting farther into the network.

### Step 3 - Check the Admin PC

I used: `ipconfig /all` to check the Admin PC's network configuration.

I then tested its default gateway.

- Admin PC -> 10.10.20.1 -> successful

The Admin PC could reach its gateway, so local connectivity appeared to be working.

### Step 4 - Check RTR1

I SSH into to RTR1 to check whether routing was preventing the Admin PC from reaching the Server network.

- Expected routes were present
- Routing appeared correct

While checking RTR1 running config, I also found an ACL blocking ICMP traffic.

This explained why the ping test failed, but it did not explain why the intranet webpage was unreachable because other IP traffic was still permitted.

Since routing appeared correct, I decided to continue toward SW2 and the Server.

### Step 5 - Identify SW2

- I wanted to check SW2, but I did not know its management IP address.

- I decided to use CDP to identify the switch.

- I ran: `show cdp neighbors` on RTR1, but SW2 did not appear in the output.

- So I checked running config and found that CDP was disabled globally on RTR1, so I enabled it with `cdp run` and checked for neighbors again.

- SW2 still did not appear.

- I then checked the running config again and found that CDP was also disabled on the interface connecting RTR1 to SW2.

- I enabled CDP on that interface with `cdp enable` and checked for neighbors once again.

- SW2 still did not appear immediately.

## Escalation Point

I expected SW2 to appear immediately after enabling CDP.

When it did not appear, I stopped using that method and did not know what to try next.

At this point, I reviewed the Witty Networks answer.

I learned that I had not given CDP enough time to populate the neighbor information.

#### Result: Escalated

## Root Cause

After SW2 became visible through CDP, the server-facing switch port was checked.

The port was assigned to VLAN 10 instead of VLAN 30.

The Server uses the `10.10.30.0/24` network, so its access port needed to be assigned to VLAN 30.

## What I Missed

I did not give CDP enough time after enabling it.

I expected the neighbor information to appear immediately. When it did not, I assumed the method was not working.

CDP sends advertisements every 60 seconds by default, so I should have waited before moving away from that test.

## What I Learned

I learned that I need to understand how a troubleshooting tool works before deciding that the test failed.

In this case, using CDP was the right direction, but I gave up on it too quickly.

I also learned that I need to test the actual service being reported. The issue was an unreachable webpage, so testing the webpage was more important than relying only on ping.

This lab also showed me that troubleshooting does not always move through the OSI model in one direction. I started by checking Layer 3, but the root cause ended up being a Layer 2 VLAN configuration problem.

## Video Walkthrough

YouTube video coming soon ⌛

## Lab Source

This troubleshooting scenario was created by [WittyNetworks.net](https://wittynetworks.net/).
