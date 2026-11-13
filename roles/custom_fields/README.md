# custom_fields

Creates global custom fields in YouTrack and attaches them to projects. It sets the empty-value setting on each
attachment when it differs from the declaration.

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
  `enum[1]`, `enum[*]` or `state[1]`.
- `bundle` (str, optional): the name of an existing bundle. Required for `enum[1]`, `enum[*]` and `state[1]`. The role
  fails when it is missing for these types. It is the default bundle of a new field, and each attachment must use it.
- `projects` (list of str, optional): short names of the projects the field is attached to. Default `[]`.
- `can_be_empty` (bool, optional): whether an issue in each attached project may leave the field empty. Default `true`.

## Behavior

- Creates a missing field. For an enum or state type, the named bundle becomes the field's default bundle.
- Fails when a field with the same name exists with another type. The role never changes the type of a field.
- Fails when a named bundle does not exist. Check mode reports this instead of failing.
- Fails when a named project does not exist.
- Attaches the field to each named project that has no attachment yet.
- Fails when an existing attachment uses another bundle than the declaration. The role does not change the bundle of an
  attachment.
- Sets `canBeEmpty` on an attachment when it differs from `can_be_empty`.
- Never deletes a field and never detaches a field from a project.
- In check mode, reports each field, attachment and update it would change, and makes no write request.

## Known limits

- In check mode, a bundle that does not exist yet cannot be confirmed as the default bundle of a field. The run says so.
- Two request shapes are not in the documented samples. Each carries a `# shortcut:` comment in its task: the default
  bundle request, and the attachment taking its bundle from the field default. A live server must confirm both.
