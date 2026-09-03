# AWX Workflow

Create a workflow with these nodes:

```text
Connection Validation
        |
      success
        v
Backup
        |
      success
        v
Create/Modify Configuration
        |
      success
        v
Verify
   |          |
success     failure
   v          v
  END      Rollback (manual review / explicit confirmation)
```

For the first lab workflow, connect the failure branch to a review-controlled rollback job rather than automatically restoring arbitrary configuration. The rollback job must receive `rollback_confirm=true` only when the operator has confirmed that targeted cleanup is safe. Do not add ServiceNow approval yet.
