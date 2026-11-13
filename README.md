# ansible-youtrack

Ansible collection `jpe_ansible.youtrack`. Its roles make a JetBrains YouTrack configuration match a declared
configuration. They manage enum and state bundles, global custom fields, and issue link types.

[![CI Gate][ci-badge]][ci-runs] [![License][license-badge]](LICENSE)

[ci-badge]: https://github.com/jpe-ansible/ansible-youtrack/actions/workflows/ci-gate.yml/badge.svg?branch=develop
[ci-runs]: https://github.com/jpe-ansible/ansible-youtrack/actions/workflows/ci-gate.yml
[license-badge]: https://img.shields.io/badge/license-Apache--2.0-blue.svg

## Installation

Build the collection from this repository and install the tarball:

```sh
ansible-galaxy collection build --output-path ./build --force
ansible-galaxy collection install ./build/jpe_ansible-youtrack-*.tar.gz
```

The collection requires Ansible 2.17 or later. It has no collection dependencies.

## Usage

1. Create a permanent token for a YouTrack user who can administer bundles, custom fields and link types.
2. Export the token. Do not write it into a file.

   ```sh
   export YOUTRACK_API_KEY=<token>
   ```

3. Run a check run first. It reads the server and writes nothing.

   ```sh
   ansible-playbook -i localhost, examples/site.yml --check
   ```

4. Run again without `--check`. Then run it a second time. The second run changes nothing.

The example playbook is in [`examples/site.yml`](examples/site.yml). Its declarations are in
[`examples/vars/youtrack.yml`](examples/vars/youtrack.yml). A minimal play looks like this:

```yaml
- name: Match the YouTrack configuration
  hosts: localhost
  gather_facts: false
  vars:
    youtrack_url: https://youtrack.example.com
    youtrack_api_key: "{{ lookup('ansible.builtin.env', 'YOUTRACK_API_KEY') }}"
    bundles_enum:
      - name: Types
        values:
          - name: Bug
  roles:
    - jpe_ansible.youtrack.bundles
```

Run the bundles role before the custom fields role. An enum or state field uses a bundle, so the bundle must exist.

## Roles

| Role | Manages | Reference |
| --- | --- | --- |
| `jpe_ansible.youtrack.bundles` | Enum and state bundles and their values. | [README](roles/bundles/README.md) |
| `jpe_ansible.youtrack.custom_fields` | Global custom fields and their project attachments. | [README](roles/custom_fields/README.md) |
| `jpe_ansible.youtrack.link_types` | Issue link types and their labels and flags. | [README](roles/link_types/README.md) |

## Variable reference

### Connection variables

Every role takes these variables.

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `youtrack_url` | Yes | None | Base URL of the YouTrack server, for example `https://youtrack.example.com`. No trailing slash. |
| `youtrack_api_key` | Yes | None | Permanent token of a user with the rights the role needs. Sent as a bearer token and never printed. |
| `youtrack_validate_certs` | No | `true` | Verify the server certificate. Set to `false` only for a test server with a self-signed certificate. |

### Declaration variables

Each role takes the lists shown below. An empty list, the default, changes nothing.

| Variable | Role | Default | Entry fields |
| --- | --- | --- | --- |
| `bundles_enum` | `bundles` | `[]` | `name`, `values` (each value: `name`, `color_id`, `description`) |
| `bundles_state` | `bundles` | `[]` | `name`, `values` (each value: `name`, `is_resolved`, `description`) |
| `custom_fields_declared` | `custom_fields` | `[]` | `name`, `field_type`, `bundle`, `projects`, `can_be_empty` |
| `link_types_declared` | `link_types` | `[]` | `name`, `source_to_target`, `target_to_source`, `directed`, `aggregation` |

The entry fields, their types and their defaults are in each role's README.

## Behavior

### Idempotency

Each role reads the current state of its items first, reading every page of each list. It writes only what differs
from the declaration. A run with the same declaration as the last run changes nothing.

### Check mode

With `--check`, each role reads the server and makes no write request. The `uri` module does not support check mode,
so each write is skipped by a condition. Each change the role would make is reported as a debug message. The run
counts it as changed.

`--diff` shows nothing for these roles. The `uri` module returns no diff data, so the predicted changes appear as
messages instead.

Check mode cannot confirm the default bundle of a custom field whose bundle does not exist yet. The run reports that
instead of guessing.

### Additive only

The roles create and update. They never delete a bundle, a bundle value, a custom field, a project attachment or a
link type. They never rename one either. An item that the declaration does not name stays as it is.

### Refusals

The run stops at the item that fails. Items handled earlier in the same run keep their changes. The run stops in these
cases:

- A custom field exists with another field type. The role never changes a field type.
- A custom field that is not an enum or state field declares `can_be_empty: false`. Only enum and state fields have
  this setting.
- An attachment uses another bundle than the declaration. The role never changes the bundle of an attachment.
- A named bundle or project does not exist.
- A bundle declares the same value name twice.
- A list has more entries than the read limit of 10000. The run stops rather than act on a partial list.

### Errors

An HTTP error stops the run. The message gives the HTTP status and the server's response text, cut to 300 characters.
The API key is never printed. Requests that carry it set `no_log`.

## Testing

The repository has two offline scenarios. Run each one from the repository root.

- `molecule/argument_specs` checks each role's argument spec with valid and invalid input. Run it with
  `molecule test -s argument_specs`.
- `molecule/behavior` runs each role against a stub of the YouTrack REST API. It checks the requests sent, paging with
  a read limit, check-mode predictions, a refused read, a mismatched attachment and the empty-value setting of each
  field kind. It needs a running Docker daemon. Run it with `molecule test -s behavior`.

Neither scenario contacts a YouTrack server. A live server must prove these points:

- A second run with the same declaration changes nothing.
- The color write shape of an enum bundle value.
- The `$type` that the bundle create request sends.
- The `$type` and bundle that the default bundle request sends.
- Whether an attachment takes the field's default bundle on attach.
- Whether an update accepts `directed` and `aggregation`, and whether it applies the body in full or in part.
- Whether `isAutoAttached` is writable on create.
- Paging through a list longer than one page.
- Certificate handling with `youtrack_validate_certs` set to `false`.

The request shapes that the docs do not state carry a `# shortcut:` comment in the task. The four comments are on the
bundle create, the value color write, the default bundle request, and the attachment that takes its bundle from the
default.

## Compatibility

The request shapes were checked against these JetBrains YouTrack REST API pages on 2026-11-12. The check used the
pages only. No server was contacted.

- [Enum bundles](https://www.jetbrains.com/help/youtrack/devportal/resource-api-admin-customFieldSettings-bundles-enum.html)
- [State bundles](https://www.jetbrains.com/help/youtrack/devportal/resource-api-admin-customFieldSettings-bundles-state.html)
- [Custom fields](https://www.jetbrains.com/help/youtrack/devportal/resource-api-admin-customFieldSettings-customFields.html)
- [Project custom fields](https://www.jetbrains.com/help/youtrack/devportal/resource-api-admin-projects-projectID-customFields.html)
- [Issue link types](https://www.jetbrains.com/help/youtrack/devportal/resource-api-issueLinkTypes.html)
- [Operations with a specific issue link type](https://www.jetbrains.com/help/youtrack/devportal/operations-api-issueLinkTypes.html)

The collection needs Ansible 2.17 or later.

## Development

The repository ships a Nix development shell. Run `direnv allow` once. Then run `pre-commit run --all-files` from the
repository root.

## Contributing

Open a pull request. Keep each commit to one change.

## License

Apache License 2.0. See [LICENSE](LICENSE).

---

Documentation: <https://docs.jacobpevans.com>
