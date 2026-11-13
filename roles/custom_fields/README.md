# custom_fields

Creates global custom fields in YouTrack, attaches them to projects, and updates the empty-value setting on existing
project attachments.

## Usage

```yaml
roles:
  - role: jpe_ansible.youtrack.custom_fields
    vars:
      youtrack_url: https://youtrack.example.com
      youtrack_api_key: "{{ lookup('ansible.builtin.env', 'YOUTRACK_API_KEY') }}"
      youtrack_custom_fields:
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

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `youtrack_url` | Yes | None | Base URL of the YouTrack server. |
| `youtrack_api_key` | Yes | None | Token that can administer fields and projects. Sent as a bearer token and never logged. |
| `youtrack_validate_certs` | No | `true` | Verify the server certificate. |
| `youtrack_custom_fields` | No | `[]` | Custom fields to ensure. |

Each custom field has:

- `name` (required): the field name.
- `field_type` (required): one of `string`, `text`, `integer`, `float`, `date`, `date and time`, `period`, `enum[1]`,
  `enum[*]`, `state[1]`, or `state[*]`.
- `bundle` (required for `enum` and `state` types): the name of a bundle. Create the bundle with the
  [bundles](../bundles/README.md) role first.
- `projects` (optional, default `[]`): short names of the projects to attach the field to.
- `can_be_empty` (optional, default `true`): whether an issue may leave the field empty.

## Behavior

- Creates a missing field. For `enum` and `state` types, the field's default bundle is the bundle you name.
- Fails when a field with the same name exists with a different type. The role never changes a field type.
- Fails when the named bundle does not exist, and when a named project does not exist.
- Attaches the field to each named project that has no attachment yet.
- Updates `can_be_empty` on an existing project attachment when it differs from the declaration.
- Never deletes a field and never detaches a field from a project.

## Known limits

- The bundle of an existing project attachment is not compared with the declared bundle.
- The request shapes for attachments, the empty-value update, and the default bundle are not confirmed against a live
  server. They carry `# shortcut:` comments in the tasks.
