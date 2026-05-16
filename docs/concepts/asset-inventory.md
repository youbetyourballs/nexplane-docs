# Asset Inventory

Asset Inventory is Nexplane's live view of your infrastructure. Assets are discovered automatically via connectors and serve as the targets for change requests.

## Asset Types

| Type | Examples |
|------|---------|
| Server / Endpoint | EC2 instances, Azure VMs, GCP Compute instances, bare-metal hosts |
| Cloud Account | AWS accounts, GCP projects, Azure subscriptions |
| Firewall / Network | Security groups, NSGs, Cloudflare zones, Palo Alto rules |
| Identity | IAM users, Okta users, AD accounts, Entra ID users |
| Application | Kubernetes deployments, registered services |
| Key Pair | EC2 key pairs |
| Storage Bucket | S3 buckets, Azure blob containers, GCP storage buckets |
| Database | RDS instances, PostgreSQL servers |
| Agent | Machines with a registered Nexplane Agent |

## How Discovery Works

Each connector polls its external system on a configured schedule (default: daily) and syncs discovered resources into Nexplane as assets. You can trigger a manual ingest from the connector detail page at any time.

Assets are identified by a stable key derived from the external system's ID (e.g., EC2 instance ID, Okta user ID). Duplicate detection prevents the same resource appearing twice even if multiple connectors could discover it.

## Tagging

Assets can be tagged manually or in bulk. Tags flow through to change requests and are used by the AI Planning Assistant to filter targets (e.g., "all assets tagged `environment:prod` running `service:payments-api`").

## Asset Detail Pages

Each asset has a detail page showing:

- Metadata (type, region, tags, associated connector)
- Network information (for servers/instances)
- Change request history
- Compliance status (if applicable)
- For servers: **Change IP** button to launch the IP Migration Wizard

## Search

Asset search supports filtering by type, tag, connector, and free-text name/ID. The search box in the Asset Inventory page uses the same index.
