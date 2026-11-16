# custom_fields

Creates global custom fields in YouTrack and attaches them to projects. It sets the empty-value setting on each
bundle-backed attachment when it differs from the declaration.

## Usage

```yaml
roles:
  - role: jpe_ansible.youtrack.custom_fields
    vars:
      youtrack_url: https://youtrack.example.com
      youtrack_api_key: "{{ lookup('ansible.builtin.env', 'YOUTRACK_API_KEY') }}"
      custom_fields_declared:
        - name: Type
          field_type: enum[1]
          bundle: Types
          projects:
            - DEMO
          can_be_empty: false
        - name: Due Date
          field_type: date
          projects:
            - DEMO
```

### Project overrides

A project in `project_overrides` takes its own bundle or empty-value setting. The field-level `bundle` is still the
default bundle of a new field.

```yaml
custom_fields_declared:
  - name: Type
    field_type: enum[1]
    bundle: Types
    projects:
      - DEMO
    project_overrides:
      - project: SAMPLE
        bundle: Kinds
        can_be_empty: false
```

The field attaches to `DEMO` with the bundle `Types`. It attaches to `SAMPLE` with the bundle `Kinds`, and an issue in
`SAMPLE` cannot leave `Type` empty.

## Inputs

| Variable | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `youtrack_url` | str | Yes | None | Base URL of the YouTrack server, for example `https://youtrack.example.com`. No trailing slash. |
| `youtrack_api_key` | str | Yes | None | Permanent token of a user who can administer fields and projects. Sent as a bearer token and never printed. |
| `youtrack_validate_certs` | bool | No | `true` | Verify the server certificate. Set to `false` only for a test server with a self-signed certificate. |
| `custom_fields_declared` | list of dict | No | `[]` | Global custom fields to ensure. |

Each entry of `custom_fields_declared` has:

- `name` (str, required): the field name. The field is matched by this name.
- `field_type` (str, required): one of `string`, `text`, `integer`, `float`, `date`, `date and time`, `period`,
  `enum[1]`, `enum[*]`, `state[1]`, `version[1]`, `version[*]`, `build[1]`, `build[*]`, `ownedField[1]`, `ownedField[*]`,
  `user[1]` or `user[*]`.
- `bundle` (str, optional): the name of an existing bundle of the field's kind. Required for every bundle-backed type
  (see the table below). The role fails when it is missing. It is the default bundle of a new field, and each
  attachment must use it.
- `projects` (list of str, optional): short names of the projects the field is attached to. Default `[]`.
- `can_be_empty` (bool, optional): whether an issue in each attached project may leave the field empty. Default `true`.
  Only bundle-backed types take `false`. The other types have no empty-value setting. A workflow rule can make a field
  of another type required.
- `project_overrides` (list of dict, optional): projects that take their own bundle or empty-value setting. Default `[]`.
  Each entry has:
  - `project` (str, required): the short name of the project. Each project appears once in the list.
  - `bundle` (str, optional): the name of an existing bundle of the field's kind. Only bundle-backed types take a bundle.
  - `can_be_empty` (bool, optional): whether an issue in this project may leave the field empty. Only bundle-backed
    types take `false`.

  A project named in `project_overrides` is attached like a project in `projects`. If a project appears in both lists,
  its entry wins. A key that an entry omits falls back to the field-level `bundle` or `can_be_empty`.

### Bundle-backed types

A bundle-backed type takes a bundle of its own kind. The role reads the bundles of a kind only when a declared field
uses that kind.

| Field type | Bundle kind | Bundle `$type` | Project attachment `$type` |
| --- | --- | --- | --- |
| `enum[1]`, `enum[*]` | enum | `EnumBundle` | `EnumProjectCustomField` |
| `state[1]` | state | `StateBundle` | `StateProjectCustomField` |
| `version[1]`, `version[*]` | version | `VersionBundle` | `VersionProjectCustomField` |
| `build[1]`, `build[*]` | build | `BuildBundle` | `BuildProjectCustomField` |
| `ownedField[1]`, `ownedField[*]` | owned | `OwnedBundle` | `OwnedProjectCustomField` |
| `user[1]`, `user[*]` | user | `UserBundle` | `UserProjectCustomField` |

## Behavior

- Creates a missing field. For a bundle-backed type, the named bundle becomes the field's default bundle.
- Fails when a field with the same name exists with another type. The role never changes the type of a field.
- Fails when a named bundle does not exist. Check mode reports this instead of failing.
- Fails when a named project does not exist.
- Attaches the field to each named project that has no attachment yet. The attach request of a project in
  `project_overrides` names that entry's bundle.
- Fails when an existing attachment uses another bundle than the one declared for its project. The role does not change
  the bundle of an attachment.
- Sets `canBeEmpty` on a bundle-backed attachment when it differs from the setting declared for its project.
- Fails when a field of another type declares `can_be_empty: false`, in the field or in a project override.
- Fails before it writes the field when a project override names a bundle for a field type without one, declares
  `can_be_empty: false` for a field type without an empty-value setting, names a project twice, or names a bundle that
  does not exist.
- Never deletes a field and never detaches a field from a project.
- In check mode, reports each field, attachment and update it would change, and makes no write request.

## Known limits

- In check mode, a bundle that does not exist yet cannot be confirmed as the default bundle of a field. The run says so.
- In check mode, the empty-value setting of a new attachment is not predicted. The run sets it after the attach request
  when it differs.
