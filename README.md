<!-- markdownlint-disable MD013 MD041 -->
<!-- markdownlint-disable MD013 MD041 -->

# satellite_users

This role allows to create, delete and modify Satellite users.

It makes use of the Red Hat certified collection [`redhat.satellite`](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/satellite/docs/), specifically
of the module [`redhat.satellite.user`](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/satellite/content/module/user/).

To use the certified collection `redhat.satellite` you need to be a Red Hat subscriber. If you don't own any subscriptions, you can make use of
[Red Hat's Developer Subscription](https://developers.redhat.com/articles/faqs-no-cost-red-hat-enterprise-linux) which is provided at no cost by Red Hat.

You can also make use of the upstream collection [`theforeman.foreman`](https://docs.ansible.com/ansible/latest/collections/theforeman/foreman/index.html), but you'd need to
change the module names from `redhat.satellite` to `theforeman.foreman` - but I have *not* tested this.

This role supports all available module options of `redhat.satellite.user` at the time of this writing (06.04.2024).

## Requirements

This role requires the collection `redhat.satellite`. It is specified via `collections/requirements.yml`.

## Role Variables

| variable                                     | default                      | required | description                                                                    |
| :---------------------------------           | :--------------------------- | :------- | :----------------------------------------------------------------------------- |
| `sat_users`                                  | unset                        | true     | Users to create/update/delete                                                  |
| `satellite_username`                         | unset                        | true[^1] | Username to authenticate with against the Satellite API                        |
| `satellite_password`                         | unset                        | true[^1] | Password of the user to authenticate with against the Satellite API            |
| `satellite_server_url`                       | unset                        | true[^1] | URL to the Satellite API (including http/s://)                                 |
| `satellite_validate_certs`                   | `true`                       | false    | Whether to validate certificates when connecting to the Satellite API          |
| `sat_quiet_assert`                           | `true`                       | false    | Whether to quiet assert                                                        |

[^1]: This role also supports passing the Satellite URL, username and password via environment variables. Please check `vars/main.yml` for the exact variables to use.

## Variable `sat_users`

An extended example of only the `sat_users` variable is illustrated down below:

```yaml
---
sat_users:
  - login: 'steffen'
    admin: true
    auth_source: 'Internal'
    default_location: 'loc-core'
    default_organization: 'org-core'
    description: 'My description'
    firstname: 'Steffen'
    lastname: 'Scheib'
    locale: 'en'
    locations:
      - 'loc-core'
      - 'loc-special'
    mail: 'steffen@example.com'
    organizations:
      - '{{ sat_initial_organization }}'
    state: 'present'
    timezone: 'Berlin'
    user_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256

  - login: 'auditor'
    admin: false
    auth_source: 'Internal'
    default_location: 'loc-core'
    default_organization: 'org-core'
    description: 'My description'
    locale: 'en'
    locations:
      - 'loc-core'
    roles:
      - 'Auditor'
      - 'custom_role'
    state: 'present'
    timezone: 'Berlin'
    user_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256
...
```

The only required attributes is `login`. Everything else can be mixed and matched. A validation of the attributes is only performed on `login`.

Please note that using `user_password` leads to the module `redhat.satellite.user` always returning a `changed` status. This is expected, as the password is stored encrypted in
Satellite and therefore it is not possible to make this operation idempotent.

## Dependencies

None

## Example Playbook

```yaml
---
- name: 'Configure users in a Red Hat Satellite'
  hosts: 'all'
  gather_facts: false
  roles:
    - role: 'satellite_smart_proxies'
      vars:
        satellite_server_url: 'https://satellite.example.com'
        satellite_username: 'steffen'
        satellite_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          [..]
        satellite_validate_certs: true
        sat_quiet_assert: false
        sat_users:
          - login: 'steffen'
            admin: true
            auth_source: 'Internal'
            default_location: 'loc-core'
            default_organization: 'org-core'
            description: 'My description'
            firstname: 'Steffen'
            lastname: 'Scheib'
            locale: 'en'
            locations:
              - 'loc-core'
              - 'loc-special'
            mail: 'steffen@example.com'
            organizations:
              - '{{ sat_initial_organization }}'
            state: 'present'
            timezone: 'Berlin'
            user_password: !vault |
                  $ANSIBLE_VAULT;1.2;AES256

          - login: 'auditor'
            admin: false
            auth_source: 'Internal'
            default_location: 'loc-core'
            default_organization: 'org-core'
            description: 'My description'
            locale: 'en'
            locations:
              - 'loc-core'
            roles:
              - 'Auditor'
              - 'custom_role'
            state: 'present'
            timezone: 'Berlin'
            user_password: !vault |
                  $ANSIBLE_VAULT;1.2;AES256
...
```

## License

[`GPL-2.0-or-later`](LICENSE)
