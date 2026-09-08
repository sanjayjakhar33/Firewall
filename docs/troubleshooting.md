# Troubleshooting

## Timeout or connection refused

Check VMware bridged mode, guest IPs, FortiGate interface administrative access, Ubuntu routing, and TCP/443. Confirm the FortiGate is not on a different VLAN or behind a host firewall.

## TLS failure

Use a trusted FortiGate certificate and set `FORTIGATE_VALIDATE_CERTS=true`. For an isolated lab self-signed certificate, `false` may permit initial testing; document and replace it.

## 401/403 or permission failure

Confirm the API token, VDOM, API administrator trusted hosts, and profile permissions. MFA for human login does not satisfy API authentication.

## Invalid access token from Ansible modules

The `fortinet.fortios` collection modules use the `ansible.netcommon.httpapi` persistent connection. With the tested FortiOS 8.0.0 build 167, that connection can report `Invalid access token` even when the same token succeeds with direct `curl` and `ansible.builtin.uri` requests. This is a known incompatibility in this lab and is not resolved by regenerating a verified token or changing collection versions.

This project uses `ansible.builtin.uri` with a Bearer token against the FortiGate REST API instead. Do not reintroduce `fortinet.fortios` modules or `connection: httpapi`; preserve the targeted GET-then-write checks and task-level secret masking used by the roles.

## Object or policy validation failure

Check the FortiOS version's schema, interface names, destination object name, policy ID 9001 availability, and that referenced objects exist. This project intentionally does not create `TEST-DESTINATION`; define that lab object separately or change the variable to an existing lab-safe destination.

## REST API schema mismatch

Record Ansible Core, Python, FortiOS, and AWX versions. Check the FortiGate API Explorer for the installed FortiOS release, especially object field names and the response shape under `results`, before changing a request body.
