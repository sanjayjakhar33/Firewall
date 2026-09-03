# AWX Credential

Create a custom credential type with a secret field for the FortiGate API token and an injector that exports `FORTIGATE_API_TOKEN` to the job environment. Inject `FORTIGATE_HOST`, `FORTIGATE_VDOM`, and `FORTIGATE_VALIDATE_CERTS` as non-secret environment values or approved inventory configuration.

Never place the token in extra vars, surveys, source, or job templates. Attach this credential only to the FortiGate job templates. Validate masking with a connection test and review the AWX job output.
