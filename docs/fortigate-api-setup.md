# FortiGate API Setup

1. Create a dedicated local API administrator, separate from human administrators who use MFA.
2. Restrict trusted hosts to the Ubuntu VM or lab management subnet.
3. Give the account only the read/write permissions required for the test address, service, policy, monitoring, and backup operations.
4. Enable HTTPS administrative access on the FortiGate interface used by Ubuntu.
5. Generate an API token and store it only in the local secret store or AWX credential.
6. Record FortiOS version, VDOM, certificate behavior, and API permissions.

Do not automate interactive MFA. API tokens are machine credentials and must be rotated and revoked independently from human accounts.

The collection accepts `access_token` and marks it sensitive. This repository maps `FORTIGATE_API_TOKEN` to the host variable at runtime. The token is not present in inventory source, vars, playbooks, or output.
