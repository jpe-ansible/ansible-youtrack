# AI Agents Configuration

Ansible collection `jpe_ansible.youtrack` declares YouTrack enum and state bundles, global custom fields and issue link
types as code.

## Rules

- Do not add `.py`, `.sh`, or `.rb` files; roles have no `files/` directories.
- Use generic examples only: `https://youtrack.example.com` and the project short name `DEMO`.
- Every task that sends the API key sets `no_log: true`.
- No role removes a bundle, value, field, attachment or link type in this version.
