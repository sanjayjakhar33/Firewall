# Troubleshooting

## Timeout or connection refused

Check VMware bridged mode, guest IPs, FortiGate interface administrative access, Ubuntu routing, and TCP/443. Confirm the FortiGate is not on a different VLAN or behind a host firewall.

## TLS failure

Use a trusted FortiGate certificate and set `FORTIGATE_VALIDATE_CERTS=true`. For an isolated lab self-signed certificate, `false` may permit initial testing; document and replace it.

## 401/403 or permission failure

Confirm the API token, VDOM, API administrator trusted hosts, and profile permissions. MFA for human login does not satisfy API authentication.

## Object or policy validation failure

Check the FortiOS version's schema, interface names, destination object name, policy ID 9001 availability, and that referenced objects exist. This project intentionally does not create `TEST-DESTINATION`; define that lab object separately or change the variable to an existing lab-safe destination.

## Collection mismatch

Record Ansible Core, Python, FortiOS, collection, and AWX versions. Reinstall with `ansible-galaxy collection install -r requirements.yml --force` only after reviewing the tested version.
