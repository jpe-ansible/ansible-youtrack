# link_types

Creates issue link types in YouTrack and updates their labels and direction flags when they differ from the
declaration.

## Usage

```yaml
roles:
  - role: jpe_ansible.youtrack.link_types
    vars:
      youtrack_url: https://youtrack.example.com
      youtrack_api_key: "{{ lookup('ansible.builtin.env', 'YOUTRACK_API_KEY') }}"
      youtrack_link_types:
        - name: Depends
          source_to_target: depends on
          target_to_source: is required for
          directed: true
```

## Inputs

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `youtrack_url` | Yes | None | Base URL of the YouTrack server. |
| `youtrack_api_key` | Yes | None | Token that can administer link types. Sent as a bearer token and never logged. |
| `youtrack_validate_certs` | No | `true` | Verify the server certificate. |
| `youtrack_link_types` | No | `[]` | Link types to ensure. |

Each link type has:

- `name` (required): the link type name. The role matches on this name and never changes it.
- `source_to_target` (required): the label shown from the source issue to the target issue.
- `target_to_source` (required): the label shown from the target issue back to the source issue.
- `directed` (optional, default `false`): whether the link has a direction.
- `aggregation` (optional, default `false`): whether the link is an aggregation.

## Behavior

- Creates a link type that is missing, with its labels and flags.
- Updates the labels and flags of an existing link type when any of them differ from the declaration.
- Never renames or deletes a link type.

## Known limits

- The request shape for updating an existing link type is not confirmed against a live server. It carries a
  `# shortcut:` comment in the task.
