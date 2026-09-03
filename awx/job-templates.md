# AWX Job Templates

All templates use inventory `FortiGate-Lab`, project `FortiGate Ansible Automation`, the FortiGate credential, and the compatible execution environment.

Add this read-only gate before the write templates:

| Name | Playbook | Extra variables |
|---|---|---|
| 00 - FortiGate Lab Preflight | 00_preflight.yml | none |

| Name | Playbook | Extra variables |
|---|---|---|
| 01 - FortiGate Connection Test | 01_test_connection.yml | none |
| 02 - Create Test Address | 02_create_address.yml | none |
| 03 - Modify Test Address | 03_modify_address.yml | none |
| 04 - Delete Test Address | 04_delete_address.yml | none |
| 05 - Create Test Service | 05_create_service.yml | none |
| 06 - Modify Test Service | 06_modify_service.yml | none |
| 07 - Delete Test Service | 07_delete_service.yml | none |
| 08 - Create Test Firewall Policy | 08_create_policy.yml | none |
| 09 - Modify Test Firewall Policy | 09_modify_policy.yml | none |
| 10 - Delete Test Firewall Policy | 10_delete_policy.yml | none |
| 11 - Backup FortiGate Configuration | 11_backup_config.yml | none |
| 12 - Verify Configuration | 12_verify_config.yml | none |
| 13 - Rollback | 13_rollback.yml | `rollback_confirm=true` only after review |

Use launch-time surveys only for non-secret, reviewed lab parameters. Keep destructive templates limited to authorized users.
