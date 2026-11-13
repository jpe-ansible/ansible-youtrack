# bundles

Creates enum and state bundles in YouTrack, adds their declared values, and updates values that differ.

## Usage

```yaml
roles:
  - role: jpe_ansible.youtrack.bundles
    vars:
      youtrack_url: https://youtrack.example.com
      youtrack_api_key: "{{ lookup('ansible.builtin.env', 'YOUTRACK_API_KEY') }}"
      youtrack_enum_bundles:
        - name: Priorities
          values:
            - name: Must
              color_id: "20"
              description: Needed for the release.
      youtrack_state_bundles:
        - name: Work States
          values:
            - name: In Review
              is_resolved: false
            - name: Done
              is_resolved: true
```

## Inputs

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `youtrack_url` | Yes | None | Base URL of the YouTrack server. |
| `youtrack_api_key` | Yes | None | Token that can administer bundles. Sent as a bearer token and never logged. |
| `youtrack_validate_certs` | No | `true` | Verify the server certificate. |
| `youtrack_enum_bundles` | No | `[]` | Enum bundles to ensure. |
| `youtrack_state_bundles` | No | `[]` | State bundles to ensure. |

Each enum bundle has a `name` and a `values` list. Each enum value has:

- `name` (required): the value name.
- `color_id` (optional): a color id from the YouTrack color palette, as a string.
- `description` (optional): the value description.

Each state bundle has a `name` and a `values` list. Each state value has:

- `name` (required): the value name.
- `is_resolved` (required): `true` when issues in this state count as resolved.
- `description` (optional): the value description.

## Behavior

- Creates a missing bundle empty, then adds each declared value to it.
- Adds a declared value that is missing.
- Updates a declared value when its description, color, or resolved flag differs from the declaration.
- Leaves values that are not declared as they are.
- Matches bundles by name and values by name within their bundle.
- Never renames or deletes a bundle or a value.

## Known limits

- Value order is not managed. YouTrack keeps the order it assigns.
- The request shapes for state values and value updates are not confirmed against a live server. They carry
  `# shortcut:` comments in the tasks.
