# Role Name

Currently this role only installs OpenProject, configuration needs to be done using the wizard.

In the future the `/etc/openproject/installer.dat` file could be created by this role and `openproject configure` could be run.

## Notes

* For some reason `openproject configure` seems to fail if the server is not rebooted before it is run.
* The default Apache config doesn't work with Let's Encrypt and the `Includes` for non existant files causes a problem, the following config with the [users role](https://git.coop/webarch/users) clobbers it, increment the STS time and re run the `users` role after each  `openproject configure` colobbers the config...

```yaml
users:
  openproject:
    users_apache_virtual_hosts:
      openproject:
        users_apache_server_name: "{{ inventory_hostname }}"
        users_apache_vhost_docroot: false
        users_apache_sts:
          - max-age=155520224
        users_apache_include_optional:
          - "/etc/openproject/addons/apache2/includes/vhost/*.conf"
          - "/etc/openproject/addons/apache2/custom/vhost/*.conf"
        users_apache_proxy_pass:
          - path: /
            url: http://127.0.0.1:6000/
            reverse: true
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
