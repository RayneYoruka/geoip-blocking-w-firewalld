![demo](./demo.gif)

# Firewall GeoIP script for firewalld

For those who need to block unwanted IPv4 and IPv6 target ranges by countries at server level using firewalld. I, myself have some use cases where servers are front facing the Internet, without WAF or CDN, so I wanted to easily secure them using firewalld. 

## Usage

- [ ] Clone this repo
- [ ] Define the countries ISO Alpha-2 code you want to block at line 18 (ZONES). [Reference](https://www.ipdeny.com/ipblocks/)
- [ ] Execute `bash firewalld`
- [ ] Define a @weekly cronjob 

## Supported Operating Systems

- [ ] Debian/Ubuntu
- [ ] RHEL/CentOS

## Contribute

All suggestions, feedback, or bug reports are welcome. Feel free to submit a PR. 

## Disclaimer

Use this script at your own risk! The author assumes no responsibility for any damages of any kind. It is strongly recommended you test this out on a test server before implemeting on production servers.

## **Known issues:**

There is currently a bug within firewallD with the backend of nftables, if you load a large ipset, it might simply crash or stop responding; See this [commit](https://github.com/firewalld/firewalld/pull/1544).

Currently, until that update is merged in to most distributions, the "easiest" yet temporary fix is to edit ```/etc/firewalld/firewalld.conf``` and change ```FirewallBackend=nftables``` to ```FirewallBackend=iptables```

I currently run this on both Debian 13 and Rocky 9, I've encountered the issue in both systems, I've switched them to iptables for the time being.


[AlmaLinux](https://almalinux.org/fi/blog/2026-03-13-test-firewalld-ipset-performance-fix/) is seeking testers to test their release.
