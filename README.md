# FortiGate Ansible Automation Lab

This repository automates deliberately named `ANSIBLE-TEST-*` FortiGate objects through Ansible and AWX. It is designed for a VMware Workstation lab first and must be adapted and re-tested before UAT or production use.

## Scope and safety

The project performs real create, modify, delete, verification, and backup operations. It never flushes configuration or deletes all policies. Destructive playbooks target only the names in `vars/lab.yml`. Use a dedicated FortiGate API administrator, never a human MFA account, and never commit a token or exported configuration.

## Architecture

Windows host -> VMware Workstation (bridged networking) -> FortiGate VM and Ubuntu VM -> AWX -> Ansible -> `fortinet.fortios` -> FortiOS REST API over HTTPS.

The Ubuntu VM and FortiGate management interface must be on a reachable network. Bridged mode normally places both guests on the physical LAN, so restrict FortiGate administrative access to the Ubuntu VM's address and do not expose the API to the Internet.

## How to access the GUIs

There are two separate web interfaces. Use a normal human administrator account and MFA for the FortiGate GUI. The Ansible API token is for AWX jobs only and is never entered into either browser.

### FortiGate GUI

From a browser on the Windows host, or from a browser inside Ubuntu, open:

```text
https://FORTIGATE_HOST
```

Use the FortiGate human administrator username, password, and MFA. A certificate warning is expected if the trial VM uses a self-signed certificate. Do not disable FortiGate security controls to remove the warning; install a trusted lab certificate later if desired.

In the FortiGate GUI, use the VDOM selector to choose `FORTIGATE_VDOM`, then verify changes under the relevant firewall address, service, and policy pages. The GUI is for human inspection and administration; Ansible uses the separate HTTPS REST API on the same management address.

### AWX GUI

AWX is opened at the address exposed by its supported Kubernetes deployment. The exact URL depends on whether you configure an Ingress, NodePort, or a temporary port-forward:

```text
https://AWX_HOSTNAME_OR_IP
```

From the Ubuntu VM, identify the service and any Ingress address with:

```bash
kubectl get svc -n awx
kubectl get ingress -n awx
kubectl get pods -n awx
```

For temporary lab-only access, port-forward the AWX web service on the Ubuntu VM. The service name may differ if your AWX custom resource has another name:

```bash
kubectl port-forward -n awx svc/awx-service 8080:80
```

Then open `http://127.0.0.1:8080` in a browser on Ubuntu. To access that temporary tunnel from the Windows browser, create an SSH tunnel from Windows to Ubuntu while the port-forward remains running:

```text
ssh -L 8080:127.0.0.1:8080 UBUNTU_USER@UBUNTU_HOST
```

Then open `http://127.0.0.1:8080` on Windows. Prefer a properly secured Ingress or restricted NodePort for regular lab use. Do not expose an unauthenticated port-forward or AWX service to the public Internet.

In AWX, the normal navigation is:

```text
Organizations -> FortiGate Lab
Inventories -> FortiGate-Lab
Projects -> FortiGate Ansible Automation
Templates -> select a job template -> Launch
Jobs -> select the job -> Output
```

The AWX administrator creates these resources. An automation operator needs permission to use the project, inventory, credential, templates, and workflow without being given unnecessary administrative access.

## How the complete flow works

1. You log in to the FortiGate GUI with a human MFA account to inspect the lab and confirm the target VDOM.
2. You store the API token in the AWX credential, or in local `.env` for direct Ubuntu testing. Git contains only placeholders.
3. AWX checks out the Git project and starts the selected execution environment.
4. Ansible reads `inventories/lab/hosts.yml` and `vars/lab.yml`. The `httpapi` connection opens HTTPS to `FORTIGATE_HOST` and sends the token to the FortiOS API for `FORTIGATE_VDOM`.
5. The selected FortiOS module compares the requested object with the current state. It creates or updates only when needed, which produces idempotent repeated runs.
6. The backup job stores a restricted timestamped response locally before changes. AWX job history records task results, while sensitive tasks are masked.
7. The verification job reads the address, service, and policy facts and fails if the expected state is absent or incorrect.
8. If a known lab verification failure requires cleanup, the operator launches rollback with explicit confirmation. It removes only the named `ANSIBLE-TEST-*` objects and policy; it does not restore arbitrary configuration.

The FortiGate GUI and AWX output are complementary: use the GUI to inspect actual device state and AWX to inspect the automation task history. A successful AWX job means the API calls completed; still verify the resulting policy in the FortiGate GUI during initial lab testing.

## Prerequisites

- FortiOS version recorded and checked against the selected collection release
- Ubuntu 24.04 or another supported Linux VM, Python 3.12+, Git, and network route to FortiGate TCP/443
- Ansible Core 2.16+ and the current `fortinet.fortios` collection
- AWX installed using a currently supported AWX deployment method and an execution environment containing this collection
- FortiGate VDOM, HTTPS administrative access, and a dedicated API token with least privilege

The collection currently documents Ansible Core 2.16+ and Python 3.12+ as requirements. Do not infer full FortiOS/AWX compatibility from that floor: record the exact FortiOS and AWX versions and verify them before pinning a release.

## Clone and deploy on the Ubuntu VM

Run these commands on the Ubuntu VM that can reach the FortiGate over the bridged VMware network. Replace only the repository URL; do not paste the API token into a command line.

```bash
sudo apt update
sudo apt install -y git python3-venv
git clone <MY-GIT-REPOSITORY> ~/fortigate-ansible-automation
cd ~/fortigate-ansible-automation
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install 'ansible-core>=2.16,<2.22'
ansible-galaxy collection install -r requirements.yml
```

The collection currently resolves to `fortinet.fortios` 2.6.0 when this project is installed. Before UAT or production, pin the exact release tested with your FortiOS, Ansible Core, Python, and AWX versions.

## Files to edit after cloning

1. Copy `.env.example` to `.env` and edit the four values locally:

	```bash
	cp .env.example .env
	chmod 600 .env
	$EDITOR .env
	```

	Set `FORTIGATE_HOST`, `FORTIGATE_VDOM`, `FORTIGATE_API_TOKEN`, and `FORTIGATE_VALIDATE_CERTS`. The token belongs only in this untracked `.env` file or the AWX secret credential.

2. Edit `vars/lab.yml` for lab-safe values. At minimum verify:

	- test address and modified address subnets
	- test service ports
	- `test_policy.srcintf` and `test_policy.dstintf`
	- `test_policy.dstaddr`, which must already exist on the FortiGate or be changed to an existing lab-safe object
	- policy ID `9001` is unused by another policy

3. Edit `inventories/lab/hosts.yml` only if your inventory design differs. Do not put credentials there.

Do not edit the playbooks to add hostnames, passwords, tokens, or production values. Do not commit `.env`, backups, or generated collection files.

## Phase 1 deployment checks

From the Ubuntu VM, confirm the bridged network before running Ansible:

```bash
ip -brief address
ip route get "$FORTIGATE_HOST"
nc -vz -w 5 "$FORTIGATE_HOST" 443
curl -kI --connect-timeout 5 "https://$FORTIGATE_HOST"
```

The `curl` request may return `401` or `403`; that still confirms HTTPS transport. A timeout or refused connection must be fixed in VMware/FortiGate networking first.

Load the local environment and validate inventory and collection discovery:

```bash
set -a; source .env; set +a
ansible --version
ansible-galaxy collection list | grep fortinet.fortios
ansible-inventory -i inventories/lab/hosts.yml --graph
```

Run the read-only preflight before any write:

```bash
ansible-playbook -i inventories/lab/hosts.yml playbooks/00_preflight.yml
```

It checks that the configured interfaces and destination object exist and that policy ID `9001` is unused or already belongs to `ANSIBLE-TEST-POLICY`.

## Complete end-to-end lab test

Run the following sequence only after the connection test succeeds. These are real configuration changes.

```bash
ansible-playbook -i inventories/lab/hosts.yml playbooks/01_test_connection.yml
ansible-playbook -i inventories/lab/hosts.yml playbooks/00_preflight.yml
ansible-playbook -i inventories/lab/hosts.yml playbooks/11_backup_config.yml

ansible-playbook -i inventories/lab/hosts.yml playbooks/02_create_address.yml
ansible-playbook -i inventories/lab/hosts.yml playbooks/02_create_address.yml
ansible-playbook -i inventories/lab/hosts.yml playbooks/03_modify_address.yml

ansible-playbook -i inventories/lab/hosts.yml playbooks/05_create_service.yml
ansible-playbook -i inventories/lab/hosts.yml playbooks/05_create_service.yml
ansible-playbook -i inventories/lab/hosts.yml playbooks/06_modify_service.yml

ansible-playbook -i inventories/lab/hosts.yml playbooks/08_create_policy.yml
ansible-playbook -i inventories/lab/hosts.yml playbooks/09_modify_policy.yml
ansible-playbook -i inventories/lab/hosts.yml playbooks/12_verify_config.yml

ansible-playbook -i inventories/lab/hosts.yml playbooks/10_delete_policy.yml
ansible-playbook -i inventories/lab/hosts.yml playbooks/07_delete_service.yml
ansible-playbook -i inventories/lab/hosts.yml playbooks/04_delete_address.yml
```

The second create runs should report no unnecessary changes. Confirm every create, modify, and delete in the FortiGate GUI. The policy test requires the source/destination interfaces and `TEST-DESTINATION` to be valid in this lab.

For a targeted rollback cleanup after a failed test:

```bash
ansible-playbook -i inventories/lab/hosts.yml playbooks/13_rollback.yml -e rollback_confirm=true
```

This removes only the named test objects and policy. It does not restore arbitrary configuration.

## AWX deployment after local testing

Do local Ansible testing first. Then configure a supported AWX installation using the current AWX Operator documentation. AWX requires a supported Kubernetes deployment method; this repository does not include an obsolete Docker-only AWX deployment.

In AWX create:

1. Organization: `FortiGate Lab`.
2. Inventory: `FortiGate-Lab`, containing `fortigate01` in group `fortigates`.
3. Project: `FortiGate Ansible Automation`, pointing to this Git repository and `main`.
4. Execution Environment: Ansible Core/Python compatible with `requirements.yml`, with `fortinet.fortios` installed.
5. Credential: a secret-backed custom credential injecting all four `FORTIGATE_*` variables as documented in `awx/credentials.md`.
6. Job templates: the 13 templates documented in `awx/job-templates.md`.
7. Workflow: the validation -> backup -> change -> verify flow documented in `awx/workflows.md`.

Never put the token in AWX extra variables, surveys, inventory source, Git, or job output. Attach the credential only to the FortiGate templates and confirm token masking with a connection test.

## Before pushing to Git

Run from the repository root:

```bash
git status --short
git diff --check
git grep -n -I -E 'FORTIGATE_API_TOKEN=([^R]|$)|password[[:space:]]*:' -- ':!*.md' || true
git check-ignore .env backups/example.json .venv
```

The `.env.example` file may contain placeholders. Review the complete diff, especially `vars/lab.yml`, before staging. Backups and local collections must remain ignored.

Then commit and push from the local clone using your own Git identity:

```bash
git add .
git commit -m "Build FortiGate Ansible automation lab"
git push origin main
```

Do not force-push or commit a populated `.env`. If a secret was ever staged, stop and rotate it before publishing.

## What is and is not ready

The repository is ready for local syntax validation, collection installation, and a real lab test after the local values above are set. It is not yet proven against your FortiGate until `01_test_connection.yml` succeeds. Full end-to-end validation also depends on your FortiOS version, API permissions, existing interfaces, destination object, certificate, and AWX execution environment.

Run write playbooks only against the lab and only after the connection test succeeds. Each supports Ansible check mode where the collection module supports it; network modules cannot promise a perfect dry run for every FortiOS field.

## Workflow

1. Test connection.
2. Run the backup playbook before a change.
3. Create or modify the address, service, and policy.
4. Run verification.
5. On a known verification failure, run the explicit rollback cleanup playbook.

The object playbooks are idempotent because the FortiOS modules reconcile by object name/policy ID rather than appending duplicates. The backup is a local API response and may contain sensitive configuration; it is ignored by Git, should be encrypted or stored outside Git, and should not be published.

## AWX and lifecycle documentation

See `docs/installation.md`, `docs/fortigate-api-setup.md`, `docs/awx-setup.md`, `docs/troubleshooting.md`, `docs/rollback.md`, `docs/security.md`, `docs/testing.md`, and the files under `awx/`.

## Version and credential notes

The API token is injected from the `FORTIGATE_API_TOKEN` environment variable in local execution. In AWX, use a custom credential with an injectors configuration that sets that environment variable, or an equivalent secret-backed credential supported by your AWX version. The token is marked `no_log` by the collection and is never printed intentionally.
