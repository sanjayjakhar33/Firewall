# Rollback

`13_rollback.yml` is a deliberate lab cleanup: it requires `-e rollback_confirm=true` and removes only the named test policy, service, and address. It does not restore an arbitrary full configuration.

Recommended sequence:

1. Run `11_backup_config.yml` before any change.
2. Apply one change set.
3. Run `12_verify_config.yml`.
4. If verification fails, inspect the failure and run `13_rollback.yml -e rollback_confirm=true` when cleanup is known to be safe.
5. Verify the named objects are gone.

A full FortiGate configuration restore can overwrite unrelated changes, affect interfaces and access, and vary by FortiOS version. Use the FortiGate GUI/CLI documented restore process only with console or out-of-band access and an operator-reviewed backup. Do not represent the local API backup as a transactional rollback.
