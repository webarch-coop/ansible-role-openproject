# Role Name

Currently this role only installs OpenProject, configuration needs to be done using the wizard.

In the future the `/etc/openproject/installer.dat` file could be created by this role and `openproject configure` could be run.

## Notes

* For some reason `openproject configure` seems to sometimes fail if the server is not rebooted before it is run.
* The default Apache config doesn't work with Let's Encrypt and the `Includes` for non existant files causes a problem, the following config with the [users role](https://git.coop/webarch/users) replicates that provided bu OpenProject:

```yaml
users:
  openproject:
    users_apache_virtual_hosts:
      openproject:
        users_apache_server_name: "{{ inventory_hostname }}"
        users_apache_vhost_docroot: false
        users_apache_expires: medium
        users_apache_directories:
          "/opt/openproject/public":
            users_apache_override:
              - "None"
            users_apache_options:
              - "-Indexes"
        users_apache_headers:
          - type: request
            action: setifempty
            arg: X-Forwarded-Proto https
        users_apache_alias:
          - url: /assets
            path: /opt/openproject/public/assets
          - url: /uploads
            path: /opt/openproject/public/uploads
        users_apache_locations:
          - location: "/"
            proxy_pass: http://127.0.0.1:6000/
            reverse: true
          - location: "/assets"
            proxy_pass: "!"
          - match: "^/sys"
            authname: Local connections only
            authtype: None
            require:
              - local
    users_editor: vim
    users_home_mode: "0755"
    users_shell: /bin/bash
    users_skel: /usr/local/etc/skel.d/www
    users_state: present
    users_system: true
```

## License

GNU General Public License v3.0 (GPLv3)

## Author Information

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
