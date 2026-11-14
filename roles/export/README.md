# export

Reads the configuration of a YouTrack server and writes it to one local JSON file. The role sends only `GET` requests
and changes nothing on the server.

## Usage

```yaml
roles:
  - role: jpe_ansible.youtrack.export
    vars:
      youtrack_url: https://youtrack.example.com
      youtrack_api_key: "{{ lookup('ansible.builtin.env', 'YOUTRACK_API_KEY') }}"
      export_dest: youtrack-config.json
```

## Inputs

| Variable | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `youtrack_url` | str | Yes | None | Base URL of the YouTrack server, for example `https://youtrack.example.com`. No trailing slash. |
| `youtrack_api_key` | str | Yes | None | Permanent token of a user who can read the administration settings. Sent as a bearer token and never printed. |
| `youtrack_validate_certs` | bool | No | `true` | Verify the server certificate. Set to `false` only for a test server with a self-signed certificate. |
| `export_dest` | path | Yes | None | Path of the JSON file to write. The file is written with mode `0600`. |
| `export_sections` | list of str | No | Every section | Names of the sections to read, from the [Sections](#sections) table. |
| `export_page_size` | int | No | `100` | Entries requested per page. Pages are read until one comes back empty. |

## Sections

A section named in the table reads one endpoint. A list section reads every page. An object section reads one object.
A section read per entry needs its parent section in `export_sections`. The attributes each section requests are listed
in [`defaults/main.yml`](defaults/main.yml).

| Section | Read | REST path |
| --- | --- | --- |
| `issue_link_types` | list | `/api/issueLinkTypes` |
| `custom_field_prototypes` | list | `/api/admin/customFieldSettings/customFields` |
| `custom_field_defaults` | object, per entry of `custom_field_prototypes` | `/api/admin/customFieldSettings/customFields/{id}/fieldDefaults` |
| `bundles_enum` | list, with values | `/api/admin/customFieldSettings/bundles/enum` |
| `bundles_state` | list, with values | `/api/admin/customFieldSettings/bundles/state` |
| `bundles_owned` | list, with values | `/api/admin/customFieldSettings/bundles/ownedField` |
| `bundles_version` | list, with values | `/api/admin/customFieldSettings/bundles/version` |
| `bundles_build` | list, with values | `/api/admin/customFieldSettings/bundles/build` |
| `bundles_user` | list, with groups and individuals | `/api/admin/customFieldSettings/bundles/user` |
| `projects` | list | `/api/admin/projects` |
| `project_custom_fields` | list, per entry of `projects` | `/api/admin/projects/{id}/customFields` |
| `project_time_tracking` | object, per entry of `projects` | `/api/admin/projects/{id}/timeTrackingSettings` |
| `agile_boards` | list | `/api/agiles` |
| `board_sprints` | list, per entry of `agile_boards` | `/api/agiles/{id}/sprints` |
| `tags` | list | `/api/tags` |
| `saved_queries` | list | `/api/savedQueries` |
| `work_item_types` | list | `/api/admin/timeTrackingSettings/workItemTypes` |
| `work_time_settings` | object | `/api/admin/timeTrackingSettings/workTimeSettings` |
| `roles` | list | `/api/roles` |
| `groups` | list | `/api/groups` |
| `users` | list | `/api/users` |
| `appearance_settings` | object | `/api/admin/globalSettings/appearanceSettings` |
| `locale_settings` | object | `/api/admin/globalSettings/localeSettings` |
| `system_settings` | object | `/api/admin/globalSettings/systemSettings` |
| `rest_settings` | object | `/api/admin/globalSettings/restSettings` |

## Output

The file is one JSON object. A list section holds a list. An object section holds an object. A per-entry section holds
an object keyed by the `id` of each parent entry. A section the server refuses with 403 or 404 holds `{"error": <status>}`.
The `meta` key holds the server URL, the export time, and each section's item count or error status.

```json
{
  "issue_link_types": [
    {"id": "20-1", "name": "Part", "sourceToTarget": "Part of", "targetToSource": "Part of", "directed": false,
     "aggregation": true, "readOnly": false}
  ],
  "project_time_tracking": {"0-1": {"enabled": true}},
  "tags": {"error": 403},
  "meta": {
    "server": "https://youtrack.example.com",
    "exported_at": "2026-01-01T00:00:00Z",
    "sections": [
      {"name": "issue_link_types", "count": 1},
      {"name": "tags", "error": 403}
    ]
  }
}
```

## Behavior

- Every request is a `GET`. No task in the role sends another method.
- Every task that sends the API key or receives server data sets `no_log`. The file is written with `content`, so
  `--diff` shows no file content.
- Lists are read page by page with `$top` and `$skip` until a page comes back empty. A list of more than 10000 entries
  stops the run instead of being cut short.
- A section that answers 403 or 404 is recorded as `{"error": <status>}`, and the run continues. Any other status stops
  the run.
- The API key is never written to the file.
- `meta.exported_at` is the time of the export that last changed the data. A run whose data matches the file keeps the
  file's timestamp, so a second run reports no change.
- A destination that already exists and is not JSON stops the run.
- In check mode the reads run. The file write reports a change when the file would change, and writes nothing.

## Not exported

- Workflows and packages. The REST API documents no resource for them.
- E-mail notification settings, the license, and the database backup settings. The administrator e-mail address in the
  system settings is not read either.
- Organizations, assigned roles, the permissions catalog, project teams, user profiles, group membership, field
  conditions and custom field instances. This version does not read them.
- Service accounts. The REST API documents no filter for them, so `users` lists every user with `id`, `login`, `name`
  and `banned`. E-mail addresses are never read.
- Runtime data: issues, articles, work items, and per-user tags and saved queries.
