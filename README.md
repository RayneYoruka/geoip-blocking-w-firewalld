# Firewalld GeoIP + ASN + VPN/Datacenter Blacklist Updater

A Bash script for automatically maintaining large IPv4 and IPv6 blacklists using **firewalld `hash:net` ipsets**.

The script can combine several independently configurable sources:

- 🌍 **GeoIP country ranges** from [IPdeny](https://www.ipdeny.com/)
- 🚫 **Bad ASN ranges** resolved through `bgpq4` / Internet Routing Registry data from [bountyyfi/bad-asn-list](https://github.com/bountyyfi/bad-asn-list)
- 🛡️ **Known VPN ranges** from [X4BNet/lists_vpn](https://github.com/X4BNet/lists_vpn)
- 🏢 **Datacenter / hosting ranges** from [X4BNet/lists_vpn](https://github.com/X4BNet/lists_vpn)

Each source can be enabled or disabled independently.

---

## Features

- 🌍 GeoIP blocking by country
  - IPv4 and IPv6
  - Multiple countries supported
  - Duplicate countries automatically removed
- 🚫 Bad ASN blocking
  - Downloads the curated ASN list from `bountyyfi/bad-asn-list`
  - Resolves ASNs into announced IPv4/IPv6 prefixes using `bgpq4`
- 🛡️ X4B VPN blocking
  - Separate IPv4 and IPv6 ipsets
  - Uses X4B's dedicated VPN network lists
- 🏢 X4B datacenter blocking
  - Separate IPv4 and IPv6 ipsets
  - Covers hosting/datacenter networks and other non-eyeball networks
- 🔀 Every source is independently configurable
- 📦 Automatically collapses overlapping and adjacent CIDRs
- 🛡️ Rejects invalid CIDRs and default routes
- 🔎 Detects overlapping ASN networks
- ⚙️ Automatically calculates `hash:net` `maxelem` capacity with headroom
- 🔄 Uses staged ipsets before replacing active blacklists
- ✅ Validates generated data before promotion
- ✅ Verifies permanent and runtime firewalld configuration
- 🧹 Removes temporary/staged ipsets after successful updates
- ♻️ Suitable for periodic execution with cron
- 🌐 Supports both IPv4 and IPv6 throughout the entire pipeline

---

## How it works

The updater does not directly modify the active blacklist while downloading or generating data.

Instead, each update follows this general process:

1. Download the enabled source lists.
2. Generate and normalize the blacklist data.
3. Validate CIDRs and reject unsafe/default routes.
4. Resolve configured ASNs into IPv4/IPv6 prefixes.
5. Collapse overlapping/adjacent networks.
6. Calculate an appropriate `hash:net` capacity.
7. Create temporary `-new` ipsets.
8. Load and verify the generated data.
9. Attach the staged ipsets to the `drop` zone.
10. Promote the staged data into the stable ipsets.
11. Remove obsolete staged ipsets.
12. Reload and verify the final runtime configuration.

This means an update is built and validated before it becomes the active blacklist.

---

## Configuration

The main configuration is located near the beginning of the script.

### Source toggles

```bash
ENABLE_GEOIP="true"
ENABLE_ASN="true"
ENABLE_X4B_VPN="false"
ENABLE_X4B_DATACENTER="false"
```

Each source is independent:

| Setting | Description |
| --- | --- |
| `ENABLE_GEOIP` | Enable country-based blocking |
| `ENABLE_ASN` | Enable bad ASN blocking |
| `ENABLE_X4B_VPN` | Enable known VPN network blocking |
| `ENABLE_X4B_DATACENTER` | Enable datacenter/hosting network blocking |

Set a value to `true` to enable the source or `false` to disable it.

If a source was previously enabled and is later disabled, its corresponding firewalld ipsets and zone sources are removed during the next successful update.

### Countries

When GeoIP blocking is enabled, configure the countries using their ISO Alpha-2 codes:


For example:

```bash
ZONES="us ru cn de fr"
```

The country lists are downloaded from IPdeny.

See the [IPdeny country list documentation](https://www.ipdeny.com/ipblocks/) for available country codes.

---

## Sources

### GeoIP - IPdeny

IPv4 country ranges:

```text
https://www.ipdeny.com/ipblocks/data/aggregated/<country>-aggregated.zone
```

IPv6 country ranges:

```text
https://www.ipdeny.com/ipv6/ipaddresses/aggregated/<country>-aggregated.zone
```

Both address families are downloaded and maintained independently.

---

### Bad ASN list

The updater uses the curated list from:

https://github.com/bountyyfi/bad-asn-list

The list contains ASNs associated with VPN providers, datacenters, hosting services, scanners, proxies, and other sources commonly associated with unwanted traffic.

The ASN numbers are resolved into announced networks using `bgpq4`.

The resulting networks are normalized before being loaded into firewalld.

---

### X4B VPN

The X4B VPN source provides known VPN network ranges:

```text
https://raw.githubusercontent.com/X4BNet/lists_vpn/refs/heads/main/output/vpn/ipv4.txt
```

```text
https://raw.githubusercontent.com/X4BNet/lists_vpn/refs/heads/main/output/vpn/ipv6.txt
```

These are maintained separately from the datacenter lists.

---

### X4B Datacenter

The X4B datacenter source provides networks classified as VPNs and/or datacenter networks:

```text
https://raw.githubusercontent.com/X4BNet/lists_vpn/refs/heads/main/output/datacenter/ipv4.txt
```

```text
https://raw.githubusercontent.com/X4BNet/lists_vpn/refs/heads/main/output/datacenter/ipv6.txt
```

X4B describes this list as covering networks that are not directly considered normal "eyeball" networks, making it considerably broader than the dedicated VPN list.

See the [X4BNet/lists_vpn](https://github.com/X4BNet/lists_vpn) project for details about how these lists are generated.

---

## Firewalld ipsets

The updater maintains separate ipsets for each source and address family.

### GeoIP

```text
geoip-blacklist-ip4
geoip-blacklist-ip6
```

### Bad ASN

```text
asn-blacklist-ip4
asn-blacklist-ip6
```

### X4B VPN

```text
x4b-vpn-ip4
x4b-vpn-ip6
```

### X4B Datacenter

```text
x4b-datacenter-ip4
x4b-datacenter-ip6
```

All enabled ipsets are attached as sources to the configured firewalld zone, which defaults to:

```text
drop
```

---

## Dependencies

The script requires:

- `bash`
- `firewalld`
- `python3`
- `curl`
- `bgpq4`
- Python's standard-library `ipaddress` module

On RHEL/CentOS/Rocky/AlmaLinux systems, make sure firewalld is installed and running before using the updater.

`bgpq4` is required only when the Bad ASN source is enabled.

---

## Installation

Clone the repository:

```bash
git clone https://github.com/RayneYoruka/geoip-blocking-w-firewalld.git
cd geoip-blocking-w-firewalld
```

Review the configuration at the beginning of the script:

```bash
nano firewalld
```

Make sure the desired sources and countries are configured.

Then run:

```bash
bash firewalld
```

The script must be run with sufficient privileges to modify firewalld.

---

## Example configuration

A conservative configuration using only GeoIP and bad ASN blocking:

```bash
ENABLE_GEOIP="true"
ENABLE_ASN="true"
ENABLE_X4B_VPN="false"
ENABLE_X4B_DATACENTER="false"

ZONES="us ru cn de fr"
```

A more aggressive configuration enabling all available sources:

```bash
ENABLE_GEOIP="true"
ENABLE_ASN="true"
ENABLE_X4B_VPN="true"
ENABLE_X4B_DATACENTER="true"

ZONES="us ru cn de fr"
```

The four sources remain separate even when all are enabled.

---

## Example output

A successful update will report the number of normalized prefixes loaded into each source:

```text
[i] Calculated capacities:
  GeoIP IPv4        : 8883 -> maxelem 65536
  GeoIP IPv6        : 3282 -> maxelem 65536
  ASN IPv4          : 39979 -> maxelem 65536
  ASN IPv6          : 5979 -> maxelem 65536
  X4B VPN IPv4      : 11200 -> maxelem 65536
  X4B VPN IPv6      : 498 -> maxelem 65536
  X4B DC IPv4       : 44414 -> maxelem 65536
  X4B DC IPv6       : 8752 -> maxelem 65536
```

The final firewalld zone will contain the enabled ipsets, for example:

```text
ipset:geoip-blacklist-ip4
ipset:geoip-blacklist-ip6
ipset:asn-blacklist-ip4
ipset:asn-blacklist-ip6
ipset:x4b-vpn-ip4
ipset:x4b-vpn-ip6
ipset:x4b-datacenter-ip4
ipset:x4b-datacenter-ip6
```

---

## Large ipset handling

Large blacklists can contain tens of thousands of networks.

The updater therefore calculates `maxelem` dynamically instead of relying on the default ipset capacity.

A safety margin is included when calculating the required capacity, and the value is rounded upward to a suitable power-of-two capacity.

This is particularly important for large GeoIP, ASN, and X4B datacenter lists.

---

## Cron

The lists are intended to be refreshed periodically.

For example, a weekly cron entry:

```cron
0 4 * * 0 /path/to/firewalld >> /var/log/firewalld-geoip.log 2>&1
```

The exact update frequency is up to the administrator.

More frequent updates can be useful for sources that change regularly, while weekly updates may be sufficient for a relatively static environment.

---

## Important considerations

These lists are **blocklists**, not authoritative classifications of malicious traffic.

Blocking an entire country, ASN, VPN provider, or datacenter can cause legitimate users or services to become inaccessible.

In particular:

- GeoIP databases can contain inaccuracies.
- ASNs can contain both legitimate and unwanted traffic.
- Datacenter networks can host perfectly legitimate services.
- VPN detection lists are not guaranteed to identify every VPN.
- X4B's datacenter list is intentionally broader than its dedicated VPN list.

Use the source combinations appropriate for your environment.

---

## Supported operating systems

The script is primarily intended for Linux systems using firewalld.

It has been used with:

- RHEL / Rocky Linux / AlmaLinux / CentOS-family systems
- Debian / Ubuntu systems with firewalld

The exact behavior of firewalld and its backend can vary between distributions and versions.

Always test changes on a non-critical system before deploying a new configuration to production.

---

## Disclaimer

Use this software at your own risk.

The author assumes no responsibility for damages, connectivity loss, blocked legitimate traffic, or other consequences resulting from the use of these lists or this script.

Always review the configuration and test it before deploying it to production systems.

## **Known issues:**

There is currently a bug within firewallD with the backend of nftables, if you load a large ipset, it might simply crash or stop responding; See this [commit](https://github.com/firewalld/firewalld/pull/1544).

Currently, until that update is merged in to most distributions, the "easiest" yet temporary fix is to edit ```/etc/firewalld/firewalld.conf``` and change ```FirewallBackend=nftables``` to ```FirewallBackend=iptables```
