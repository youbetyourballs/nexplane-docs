# Jira Connector

The Jira connector creates and syncs tickets for Nexplane change requests.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| URL | Yes | Jira instance URL (e.g., `https://acme.atlassian.net`) |
| Email | Yes | Jira user email |
| API Token | Yes | Jira API token |
| Project Key | Yes | Jira project key (e.g., `SEC`) |

## Capabilities

When a change request is created in Nexplane, a corresponding Jira ticket is created automatically. Status transitions in Nexplane (approved, executed, completed) update the Jira ticket status.
