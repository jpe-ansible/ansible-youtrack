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
      link_types_declared:
        - name: Part
          source_to_target: Part of
          target_to_source: Part of
          aggregation: true
```

## Inputs

| Variable | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `youtrack_url` | str | Yes | None | Base URL of the YouTrack server, for example `https://youtrack.example.com`. No trailing slash. |
| `youtrack_api_key` | str | Yes | None | Permanent token of a user with low-level admin write access. Sent as a bearer token and never printed. |
| `youtrack_validate_certs` | bool | No | `true` | Verify the server certificate. Set to `false` only for a test server with a self-signed certificate. |
| `link_types_declared` | list of dict | No | `[]` | Issue link types to ensure. |

Each entry of `link_types_declared` has:

- `name` (str, required): the link type name. The link type is matched by this name, and the name is never changed.
- `source_to_target` (str, required): the label shown from the source issue to the target issue, for example
  `Part of`.
- `target_to_source` (str, required): the label shown from the target issue back to the source issue, for example
  `Part of`.
- `directed` (bool, optional): whether the link has a direction. Default `false`.
- `aggregation` (bool, optional): whether the link describes an aggregation. Default `false`.

## Behavior

- Creates a link type that is missing, with its labels and flags.
- Updates the labels and flags of an existing link type when any of them differs from the declaration. A flag that the
  declaration omits counts as `false`.
- Never renames or deletes a link type.
- In check mode, reports each link type it would create or update, and makes no write request.

