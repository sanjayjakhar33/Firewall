# Lab Testing Checklist

Run in order, recording AWX job IDs and FortiGate GUI/API observations.

- Connectivity: connection playbook succeeds.
- Authentication failure: invalid AWX credential fails cleanly without exposing the token.
- Address create: object `ANSIBLE-TEST-SERVER` appears.
- Address idempotency: repeat create and expect no change.
- Address modify: subnet changes to the configured modified value.
- Address delete: only the named address disappears.
- Service create, repeat, modify, and delete: verify `ANSIBLE-TEST-HTTPS` and its ports.
- Policy create: verify policy ID 9001, references, action, and interfaces.
- Policy modify: verify the intended logging change.
- Policy delete: verify only policy ID 9001 is removed.
- Backup: verify a restricted, timestamped file exists outside Git publication.
- Failure handling: use an invalid interface or destination in a disposable lab run and confirm failure without broad deletion.
- Rollback: run the explicit confirmed cleanup and verify the named test objects are absent.
- Check mode: run `ansible-playbook --check` and record which changes the collection can predict.
