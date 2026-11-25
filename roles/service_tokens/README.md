# service_tokens

Ensures named Hub permanent tokens exist for existing YouTrack service users. Newly created token values are returned
to the calling play in a fact so that the play can store them in its secret store. The role never stores or prints
token values.

## Usage

```yaml
roles:
  - role: jpe_ansible.youtrack.service_tokens
    vars:
      youtrack_url: https://youtrack.example.com
      youtrack_api_key: "{{ lookup('ansible.builtin.env', 'YOUTRACK_API_KEY') }}"
      service_tokens_declared:
        - login: svc-example
          name: example-automation
```

The calling play can store newly minted values from `service_tokens_minted` in its secret store. The fact contains
only values created during that run. `service_tokens_present` lists the declared names that exist after the run.

## Inputs

| Variable | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `youtrack_url` | str | Yes | None | Base URL of the YouTrack server, for example `https://youtrack.example.com`. No trailing slash. |
| `youtrack_api_key` | str | Yes | None | Permanent token of a Hub administrator. Sent as a bearer token and never printed. |
| `youtrack_validate_certs` | bool | No | `true` | Verify the server certificate. Set to `false` only for a test server with a self-signed certificate. |
| `service_tokens_hub_url` | str | No | `{{ youtrack_url }}/hub/api/rest` | Hub REST base URL. No trailing slash. |
| `service_tokens_hub_service_id` | str | No | None | Hub service ID used as the scope for new tokens. When set, the role does not list services. |
| `service_tokens_declared` | list of dict | No | `[]` | Named permanent tokens to ensure for existing service users. |
| `service_tokens_remint` | list of str | No | `[]` | Names of declared tokens to delete and create again on this run. |

Each entry of `service_tokens_declared` has:

- `login` (str, required): the existing Hub service user's login. The role never creates users.
- `name` (str, required): the permanent token's name. It must be unique for its user and across the declaration list,
  because `service_tokens_minted` is keyed by name.

## Outputs

| Fact | Type | Description |
| --- | --- | --- |
| `service_tokens_minted` | dict | Maps each name created during this run to its token value. This fact is hidden from task output. |
| `service_tokens_present` | list of str | Names of the declared tokens that exist after the run. |

## Behavior

- Finds exactly one Hub user for each login and never creates a user.
- Uses the sole Hub service for the YouTrack application as the scope for a new token. Set
  `service_tokens_hub_service_id` to choose a service without listing services.
- Leaves a matching token unchanged when its name is not in `service_tokens_remint`.
- Creates a missing token, or deletes and creates a matching token named in `service_tokens_remint`.
- Stops if more than one token for a user has a declared name.
- In check mode, reads the Hub, reports tokens it would create or re-create as changed, and makes no write request.
- Makes no write request on a second run with the same declaration and an empty re-mint list.
- Sends no token value to task output or a file.
