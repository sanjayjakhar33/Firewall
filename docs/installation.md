# Installation

## Ubuntu prerequisites

Install Git, Python 3.12+, `python3-venv`, Ansible Core 2.16+, and a currently supported AWX deployment method. The official AWX Operator requires Kubernetes; do not install an unsupported Docker-only AWX distribution. For a small lab, use a supported Kubernetes distribution documented by the current AWX Operator release, or use an existing AWX instance.

```bash
sudo apt update
sudo apt install -y git python3-venv
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install 'ansible-core>=2.16,<2.22'
ansible-galaxy collection install -r requirements.yml
```

For AWX, build an execution environment from the repository definition after installing `ansible-builder`:

```bash
python3 -m pip install ansible-builder
ansible-builder build --file execution-environment.yml --tag fortigate-awx-ee:2.6.0
```

Push that image to a registry reachable by AWX and select it on the job templates. The image must contain the pinned `fortinet.fortios` collection.

## Network gate

From Ubuntu, confirm the bridge route and HTTPS port:

```bash
set -a; source .env; set +a
ip route get "$FORTIGATE_HOST"
nc -vz -w 5 "$FORTIGATE_HOST" 443
curl -kI --connect-timeout 5 "https://$FORTIGATE_HOST"
```

A FortiGate HTTP 401/403 response still proves transport reachability. Replace `FORTIGATE_HOST` with the local value; never commit it if it is sensitive.

## Local execution

```bash
cp .env.example .env
$EDITOR .env
set -a; source .env; set +a
ansible-inventory -i inventories/lab/hosts.yml --graph
ansible-playbook -i inventories/lab/hosts.yml playbooks/01_test_connection.yml
```

Use `FORTIGATE_VALIDATE_CERTS=true` with a trusted certificate. `false` is only a lab fallback for a self-signed certificate and does not make the connection secure against interception.
