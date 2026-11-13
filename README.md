# ansible-youtrack

Ansible collection `jpe_ansible.youtrack`: roles that make a JetBrains YouTrack configuration match a declared
configuration. They manage enum and state bundles, global custom fields, and issue link types.

## Installation

Add the collection to a `requirements.yml` file and install it into a project-local collections path:

```yaml
collections:
  - name: https://github.com/jpe-ansible/ansible-youtrack.git
    type: git
    version: <release tag>
```

```sh
ANSIBLE_COLLECTIONS_PATH=./collections ansible-galaxy collection install -r requirements.yml
```

The collection requires Ansible 2.17 or later. It has no collection dependencies.

## Usage

Reference a role by its fully qualified name. Set the connection variables once for the play. Run the bundles role
before the custom fields role, because enum and state fields use the bundles.

```yaml
- name: Match the YouTrack configuration
  hosts: localhost
  gather_facts: false
  vars:
    youtrack_url: https://youtrack.example.com
    youtrack_api_key: "{{ lookup('ansible.builtin.env', 'YOUTRACK_API_KEY') }}"
  roles:
    - role: jpe_ansible.youtrack.bundles
      vars:
        youtrack_enum_bundles:
          - name: Types
            values:
              - name: Bug
                color_id: "1"
              - name: Feature
    - role: jpe_ansible.youtrack.custom_fields
      vars:
        youtrack_custom_fields:
          - name: Type
            field_type: enum[1]
            bundle: Types
            projects:
              - DEMO
            can_be_empty: false
    - role: jpe_ansible.youtrack.link_types
      vars:
        youtrack_link_types:
          - name: Part
            source_to_target: Part of
            target_to_source: Part of
            aggregation: true
```

Read the API key from a secret store or an environment variable at run time. Do not write it into a playbook.

| Role | Manages |
| --- | --- |
| [`bundles`](roles/bundles/README.md) | Enum and state bundles and their values. |
| [`custom_fields`](roles/custom_fields/README.md) | Global custom fields and their project attachments. |
| [`link_types`](roles/link_types/README.md) | Issue link types and their directions. |

### Connection variables

Every role takes these variables.

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `youtrack_url` | Yes | None | Base URL of the YouTrack server, for example `https://youtrack.example.com`. |
| `youtrack_api_key` | Yes | None | Token from a user who can administer the instance. Sent as a bearer token and never logged. |
| `youtrack_validate_certs` | No | `true` | Verify the server certificate. Set to `false` only for a test server with a self-signed certificate. |

## Behavior

- **Idempotent.** Each role reads the current state first and writes only what differs. A second run with the same
  input changes nothing.
- **Additive.** The roles never delete anything. They do not remove a bundle, a bundle value, a field, a project
  attachment, or a link type. Items you do not declare stay as they are.
- **Check mode.** Read requests run in check mode. Write requests are skipped, because the `uri` module does not
  support check mode. A check run reports no changes and does not preview the writes it would make.
- **Errors.** An HTTP error stops the run and reports the status code. The API key does not appear in the output.

## Testing

The `molecule/argument_specs` scenario checks each role's argument spec with valid and invalid input. Run it from
`molecule/argument_specs` with `molecule test -s default`. It needs a running Docker daemon.

The test suite checks argument validation only. It does not run the tasks against a YouTrack server. Live behavior is
not yet verified. Tasks whose request shape is unconfirmed carry a `# shortcut:` comment.

## Development

The repository ships a Nix development shell. Run `direnv allow` once. Then run `pre-commit run --all-files` from the
repository root.

## Contributing

Open a pull request. Keep each commit to one change.

## License

Apache License 2.0. See [LICENSE](LICENSE).
