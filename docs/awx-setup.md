# AWX Setup

AWX is required, but its supported deployment method changes over time. Use the current AWX Operator documentation for installation on Kubernetes. Do not use an obsolete Docker Compose recipe.

Create these AWX resources:

- Organization: `FortiGate Lab`
- Inventory: `FortiGate-Lab`, containing host `fortigate01` and no token
- Project: `FortiGate Ansible Automation`, SCM Git, repository `<MY-GIT-REPOSITORY>`, branch `main`
- Execution Environment: an image containing Ansible Core compatible with the selected collection and `fortinet.fortios` installed from `requirements.yml`
- Credential: custom secret-backed credential injecting all four `FORTIGATE_*` environment variables

Inject these environment values in the AWX credential or an AWX-supported secret mechanism. Extra variables do not populate `lookup('env', ...)` and must not be used for this inventory:

```text
FORTIGATE_HOST=<lab host>
FORTIGATE_VDOM=<lab vdom>
FORTIGATE_API_TOKEN=<secret>
FORTIGATE_VALIDATE_CERTS=false
```

Do not put the token in extra variables, survey defaults, Git, inventory, or job output. Test with a deliberately invalid token only in the AWX credential UI, then remove it.

## Job execution

Set the inventory, project, execution environment, and credential on each job template. Add the read-only `00_preflight.yml` job before write jobs. Enable job history and retain logs according to lab policy. Use `--check` through the AWX job options only as an advisory dry run because network modules may not model every field.
