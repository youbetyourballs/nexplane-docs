# Cloudflare Connector

The Cloudflare connector manages WAF rules, firewall rules, access policies, DNS, and SSL configuration.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| API Token | Yes | Cloudflare API token with Zone:Edit and Firewall:Edit permissions |
| Zone ID | Yes | Cloudflare Zone ID |
| Account ID | Yes | Cloudflare Account ID |

## Capabilities

| Action | Description |
|--------|-------------|
| WAF rules | Create, update, and delete WAF custom rules |
| Firewall rules | Create, update, and delete firewall rules |
| Access policies | Create and update Zero Trust access policies |
| Block IP | Add an IP address to a firewall block list |
| SSL mode | Update the SSL/TLS encryption mode |
| DNS management | Create, update, and delete DNS records |

All actions support rollback to the pre-change state.
