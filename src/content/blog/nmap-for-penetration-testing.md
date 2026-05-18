---
title: 'Nmap for Penetration Testing'
description: 'Beginner notes on using Nmap during authorized penetration testing, including host discovery, port scanning, service detection, OS detection, NSE scripts, and reporting.'
pubDate: 'May 18 2026'
---

Nmap is a network scanning tool used to discover hosts, identify open ports, enumerate services, detect operating systems, and check for known vulnerabilities.

From a penetration testing perspective, the basic workflow is:

1. Discover live hosts.
2. Identify open ports.
3. Enumerate services and versions.
4. Identify the operating system.
5. Check for known vulnerabilities.
6. Validate findings with another tool or manual testing.

**Warning:** Only scan systems you own or have explicit written permission to test. Even basic scanning can trigger alerts, create noisy logs, or violate rules if it is done outside an authorized lab or engagement.

## Confirm Your Network

Before scanning, confirm what network you are connected to.

Two useful commands are:

`route`

This shows the default gateway and routing table.

`ifconfig`

This shows your IP address and network interface information.

An example local network might look like this:

`192.168.1.0/24`

The `/24` means the network covers addresses from:

`192.168.1.1` through `192.168.1.254`

Understanding your network range matters because you do not want to accidentally scan systems outside the scope of your lab or authorized test.

## Host Discovery

Before scanning ports, it is common to find which hosts are alive.

### ARP Scan

ARP discovery is useful on a local network.

`sudo nmap -PR -sn 192.168.1.0/24`

The `-PR` option tells Nmap to use ARP requests. The `-sn` option tells Nmap to do host discovery only and skip port scanning.

This is fast on a local subnet because ARP is how devices normally find each other on the same network. Instead of deeply scanning every possible IP address, you first find which devices are actually online.

### Save Discovered IPs

After finding live hosts, save them to a file:

`nano iplist.txt`

Example:

<pre class="plain-output">192.168.1.7
192.168.1.9
192.168.1.13
192.168.1.14
192.168.1.254</pre>

This lets you scan only live targets later.

### ICMP Ping Scan

ICMP discovery can be useful when scanning a host outside your local network.

`sudo nmap -PE -sn scanme.nmap.org`

The `-PE` option sends an ICMP Echo Request, similar to a normal ping.

The limitation is that firewalls may block ICMP. If a host does not respond, that does not always mean it is offline.

### ACK Scan

TCP-based discovery can help when ICMP is blocked.

`sudo nmap -PA80 -sn scanme.nmap.org`

The `-PA80` option sends a TCP ACK packet to port 80. If the host replies with a TCP reset packet, Nmap knows the host is alive.

This matters because some networks block ping but still respond to certain TCP packets.

## Port States

When scanning, Nmap reports port states.

### Open

The port is accepting connections. This is the main thing you care about during enumeration.

### Closed

The port is reachable, but nothing is listening on it. This confirms the host exists, but the port itself is not useful.

### Filtered

Nmap cannot determine if the port is open because something is blocking the probe, usually a firewall.

### Unfiltered

The port is reachable, but Nmap cannot tell if it is open or closed. This may be worth investigating with a different scan type.

### Open | Filtered

Nmap cannot determine whether the port is open or filtered. This is common with UDP scans.

## Basic TCP Port Scan

A basic scan looks like this:

`nmap 192.168.1.7`

By default, Nmap scans the top 1,000 most common TCP ports.

Example output might look like this:

<pre class="plain-output">PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http</pre>

This tells us that SSH and HTTP are open. At this point, I would not assume either service is vulnerable. I would treat them as leads for more enumeration.

## Scan a List of Hosts

Once you have saved live hosts into `iplist.txt`, you can scan the list:

`nmap -iL iplist.txt`

The `-iL` option tells Nmap to read targets from a file.

This is cleaner than scanning the entire subnet after host discovery.

## Scan Specific Ports

To scan for SSH:

`nmap -p 22 -iL iplist.txt`

To scan for HTTP:

`nmap -p 80 -iL iplist.txt`

To scan multiple ports:

`nmap -p 21,22,23,80 -iL iplist.txt`

This is useful because you can quickly check which machines have specific services exposed.

A machine with FTP, Telnet, and HTTP open could be a strong target for further testing, especially if the services are old or misconfigured.

## ACK Scan for Filtered Ports

An ACK scan can help identify firewall behavior:

`sudo nmap -sA -iL iplist.txt`

The `-sA` option checks whether ports are filtered or unfiltered.

This scan does not directly tell you if a port is open, but it can show whether traffic is being filtered.

## UDP Scanning

UDP is often overlooked, but important services use UDP.

`sudo nmap -sU -iL iplist.txt`

Examples of UDP services include:

<pre class="plain-output">53  DNS
67  DHCP
69  TFTP
123 NTP
161 SNMP</pre>

## Service and Version Detection

Finding an open port is useful, but knowing what service is running is more useful.

`sudo nmap -sV -iL iplist.txt`

The `-sV` option asks Nmap to identify services and versions running on open ports.

Example output:

```text
80/tcp open  http  Apache httpd 2.4.52
21/tcp open  ftp   vsftpd 3.0.3
```

The version number lets you research known vulnerabilities.

For example:

- Apache 2.4.52 vulnerabilities
- vsftpd 3.0.3 vulnerabilities

Version detection is one of the most important scans in a penetration test because it gives you the information needed for deeper research.

## OS Detection

Nmap can also attempt to identify the operating system of each host:

`sudo nmap -O -iL iplist.txt`

Example findings might include:

- Windows 7
- Windows XP
- Linux
- Ubuntu

Operating system information helps you understand the target environment.

A Windows XP or Windows 7 machine is especially concerning because those systems are outdated and often vulnerable.

## Timing Options

Nmap timing is controlled with `-T`.

Example:

`nmap -T5 scanme.nmap.org`

Timing levels:

- `-T0`: Paranoid, very slow
- `-T1`: Sneaky
- `-T2`: Polite
- `-T3`: Normal, default
- `-T4`: Aggressive
- `-T5`: Insane, fastest

Faster scans finish quicker, but they are noisier and may be less accurate. Slower scans are quieter, but they can take a very long time.

## Scan Behavior Options

Nmap includes options that can change how scan traffic appears on the network. In a professional test, these should only be used when they are allowed by the rules of engagement.

### Decoy Scan

`sudo nmap 192.168.1.7 -D RND:20`

The `-D RND:20` option adds 20 random decoy IP addresses into the scan traffic.

### Randomize Host Order

`sudo nmap -iL iplist.txt --randomize-hosts`

The `--randomize-hosts` option scans hosts in a random order instead of sequentially.

### Spoof MAC Address

`sudo nmap 192.168.1.9 --spoof-mac 0`

The `--spoof-mac 0` option randomizes your MAC address for the scan.

## Nmap Scripting Engine

Nmap includes the Nmap Scripting Engine, also called NSE. Scripts can gather deeper information and check for known vulnerabilities.

On Kali, scripts are usually stored here:

`/usr/share/nmap/scripts`

Nmap also has an official script library online.

The default script option is:

`nmap -sC 192.168.1.7`

A common beginner command combines default scripts with version detection:

`nmap -sC -sV 192.168.1.7`

## Vulnerability Scan

Nmap can run a broader set of vulnerability detection scripts:

`nmap --script vuln -iL iplist.txt`

This is an easy way to check many known vulnerabilities at once.

The tradeoff is that it can take longer and may produce a lot of output.

Example vulnerability categories include:

- RDP vulnerabilities
- SMB vulnerabilities
- FTP vulnerabilities
- SSL/TLS issues

Vulnerability scripts are useful, but they are not perfect. A script result should be treated as a lead until it is validated.

## Final Thoughts

Nmap is one of the most useful tools for learning penetration testing because it teaches you how to do reconnaissance and map out a network.

The goal is not just to run commands, but to understand what each scan is trying to answer. The basic process is simple: find the systems, identify the open ports, understand the services behind them, and decide what needs a closer look.
