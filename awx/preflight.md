# Preflight Job

Create an optional `00 - FortiGate Lab Preflight` job template for `playbooks/00_preflight.yml`. Run it before any write job. It checks the configured source/destination interfaces, destination address object, and policy ID. It performs read-only API calls and fails before writes when prerequisites are missing.
