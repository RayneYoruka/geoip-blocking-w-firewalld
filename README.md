# Firewalld GeoIP + Bad ASN Blacklist Updater

A Bash script for automatically maintaining large GeoIP and bad-ASN blacklists with **firewalld** and `hash:net` ipsets.

## Features

- 🌍 GeoIP blocking using [IPdeny](https://www.ipdeny.com/)
  - IPv4 and IPv6
  - Configurable country list
  - Automatically deduplicates countries
- 🚫 Bad ASN blocking using [bountyyfi/bad-asn-list](https://github.com/bountyyfi/bad-asn-list)
- 🔎 Uses `bgpq4` / IRR data to resolve ASNs into announced IPv4/IPv6 prefixes
- 📦 Automatically collapses overlapping CIDRs
- 🛡️ Rejects invalid CIDRs, overlapping ASN networks, and default routes
- ⚙️ Dynamically calculates `hash:net` `maxelem` capacity with headroom
- 🔄 Uses staged ipsets and validation before replacing the active blacklists
- ✅ Verifies both permanent and runtime firewalld configuration
- 🧹 Automatically removes temporary/staged ipsets after successful promotion
- ♻️ Designed to be safely run periodically via cron

## Requirements

- firewalld
- curl
- bgpq4
- Python 3
- Root privileges
  
## Supported Operating Systems

- ✅ RHEL/CentOS/Rocky/Alma
- ❓ Debian/Ubuntu (Not tested yet, original script worked on them with firewalld)

  

## Configuration & Usage

Edit the script and configure the countries you want blocked:

- Clone this repo
- Define the countries ISO Alpha-2 code you want to block at line 38 (ZONES). [Reference](https://www.ipdeny.com/ipblocks/)

`ZONES="ru ch"` ...
- Execute `bash firewalld.updated`
- Define a @weekly cronjob 



## Disclaimer

Use this script at your own risk! The author assumes no responsibility for any damages of any kind. It is strongly recommended you test this out on a test server before implemeting on production servers.

## **Known issues:**

There is currently a bug within firewallD with the backend of nftables, if you load a large ipset, it might simply crash or stop responding; See this [commit](https://github.com/firewalld/firewalld/pull/1544).

Currently, until that update is merged in to most distributions, the "easiest" yet temporary fix is to edit ```/etc/firewalld/firewalld.conf``` and change ```FirewallBackend=nftables``` to ```FirewallBackend=iptables```

I currently run this on both Debian 13 and Rocky 9, I've encountered the issue in both systems, I've switched them to iptables for the time being.


[AlmaLinux](https://almalinux.org/fi/blog/2026-03-13-test-firewalld-ipset-performance-fix/) is seeking testers to test their release.
