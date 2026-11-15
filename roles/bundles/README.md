# bundles

Ensures enum, state, version and build bundles exist in YouTrack with their declared values. It creates a missing bundle and adds a
missing value. It updates a declared value when its description, color or resolved flag differs.

## Usage

```yaml
roles:
  - role: jpe_ansible.youtrack.bundles
    vars:
      youtrack_url: https://youtrack.example.com
      youtrack_api_key: "{{ lookup('ansible.builtin.env', 'YOUTRACK_API_KEY') }}"
      bundles_enum:
        - name: Types
          values:
            - name: Bug
              color_id: "1"
      bundles_state:
        - name: States
          values:
            - name: Submitted
              is_resolved: false
            - name: Verified
              is_resolved: true
              description: Issue fix is verified
```

## Inputs

| Variable | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `youtrack_url` | str | Yes | None | Base URL of the YouTrack server, for example `https://youtrack.example.com`. No trailing slash. |
| `youtrack_api_key` | str | Yes | None | Permanent token of a user who can administer bundles. Sent as a bearer token and never printed. |
| `youtrack_validate_certs` | bool | No | `true` | Verify the server certificate. Set to `false` only for a test server with a self-signed certificate. |
| `bundles_enum` | list of dict | No | `[]` | Enum bundles to ensure. |
| `bundles_state` | list of dict | No | `[]` | State bundles to ensure. |
| `bundles_version` | list of dict | No | `[]` | Version bundles to ensure. |
| `bundles_build` | list of dict | No | `[]` | Build bundles to ensure. |

Each entry of `bundles_enum` has:

- `name` (str, required): the bundle name. The bundle is matched by this name.
- `values` (list of dict, required): the values the bundle must contain. Each value has:
  - `name` (str, required): the value name. The value is matched by this name within its bundle.
  - `color_id` (str, optional): a palette index, for example `"1"`. Left as it is when omitted.
  - `description` (str, optional): the value description. Left as it is when omitted.

Each entry of `bundles_state` has:

- `name` (str, required): the bundle name. The bundle is matched by this name.
- `values` (list of dict, required): the values the bundle must contain. Each value has:
  - `name` (str, required): the value name. The value is matched by this name within its bundle.
  - `is_resolved` (bool, required): whether an issue in this state counts as resolved.
  - `color_id` (str, optional): a palette index, for example `"2"`. Left as it is when omitted.
  - `description` (str, optional): the value description. Left as it is when omitted.

Each entry of `bundles_version` has:

- `name` (str, required): the bundle name. The bundle is matched by this name.
- `values` (list of dict, required): the values the bundle must contain. Each value has:
  - `name` (str, required): the value name. The value is matched by this name within its bundle.
  - `description` (str, optional): the value description. Left as it is when omitted.
  - `color_id` (str, optional): a palette index, for example `"1"`. Left as it is when omitted.
  - `released` (bool, optional): whether the version is released. Left as it is when omitted.
  - `archived` (bool, optional): whether the version is archived. Archiving sets a flag and deletes nothing. Left as it
    is when omitted.
  - `release_date` (int, optional): the release date as epoch milliseconds in UTC, for example `1576367999999`. Left as
    it is when omitted.

Each entry of `bundles_build` has:

- `name` (str, required): the bundle name. The bundle is matched by this name.
- `values` (list of dict, required): the values the bundle must contain. Each value has:
  - `name` (str, required): the value name. The value is matched by this name within its bundle.
  - `description` (str, optional): the value description. Left as it is when omitted.
  - `color_id` (str, optional): a palette index, for example `"1"`. Left as it is when omitted.
  - `assemble_date` (int, optional): the assembly date as epoch milliseconds in UTC, for example `1627636495000`. Left
    as it is when omitted.

## Behavior

- Creates a missing bundle empty, then adds each declared value to it.
- Adds a declared value that is missing.
- Updates a declared value when a declared attribute differs from the stored value: its description, color, resolved
  flag, release date, released flag, archived flag or assembly date.
- Leaves values that are not declared as they are.
- Fails when a bundle declares the same value name twice.
- Never renames or deletes a bundle or a value.
- In check mode, reports each bundle, value and update it would change, and makes no write request.

## Known limits

- Value order is not managed. YouTrack keeps the order it assigns.
