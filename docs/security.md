# Security

- Use a dedicated API identity, separate from MFA-protected human administration.
- Restrict trusted hosts and never expose FortiGate API access to the public Internet.
- Keep `.env`, tokens, AWX credential values, and backups outside Git.
- Use trusted TLS certificates where possible; disabling validation is only a temporary isolated-lab exception.
- Review policy interfaces, destination objects, schedule, NAT, and logging before applying.
- Use `no_log` only around token-bearing or backup-content tasks so ordinary troubleshooting remains visible.
- Rotate and revoke tokens, and check Git history before publishing.
- Treat backups as sensitive configuration; encrypt them or store them outside the repository.
