`alternc-slavedns` Role
=========

Install and cofiguration of alternc-slavedns (Dynamic redundant DNS server for AlternC managed bind zones)


Requirements
------------

This role installs alternc-slavedns on a host, and configures it as the slave of an AlternC hosting server, presumably (but not necessary) installed with the [ansible udelarinterior.alternc role](https://galaxy.ansible.com/udelarinterior/alternc).


Role Variables
--------------

Most significant variables (the packages repository, the AlternC desktop we are slave of, the access credentials, ...) are shared with the AlternC main server, so they deserve to be set in an ansible group that contains the AlternC master and the slave we are installing (an "alternc cluster group"). 

Variables are: 

* AlternC packages repository and key. Presently: 
```
alternc_repo: 'http://debian.koumbit.net/debian'
alternc_branch: 'alternc35 main'
alternc_key: 'key.asc'
```

Variables' default values are built from the AlternC main roles set of variables.

* The web protocol to access the list of domains to be secondary DNS:

`alternc_slavedns_protocol`: `http` or `https`, defaults to `alternc_desktop_protocol` if defined (master role's variable), or `http` if not

* The AlternC server (or other system) to gather the list of domains from: 

`alternc_slavedns_master`: `panel.mydomain.org`, defaults to `alternc_debconf_desktopname` (master role's variable) if defined

* The IP of the BIND master server (generally the same AlternC server) to configure slave zones 

`alternc_slavedns_master_ip`: defaults trying to solve `alternc_slavedns_master` dns

* The next variables, clearly shared with the main role, define the credentials to access the AlternC master(s) we are slave of. For example:

```
alternc_slavedns_credentials:
- hostname: slavedns1.mydomain.org
  login: slavedns1
  password: "{{ vault_password1 }}"
- hostname: slavedns2.mydomain.org
  login: slavedns2
  password: "{{ vault_password2 }}"
```
The following variables are inspired by the alternc-slavedns script ones. Default value of one of them (`alternc_slavedns_named`) has been updated to recent distos versions: 

```
# The login to access the web page of the list of zones in the AlternC server (or other system)
alternc_slavedns_login_query:  '[?hostname==`{{ inventory_hostname }}`].login'
alternc_slavedns_login_cluster: '{{ alternc_slavedns_credentials | json_query( alternc_slavedns_login_query ) }}'
alternc_slavedns_login: "{{ '' if alternc_slavedns_login_cluster | length == 0 else alternc_slavedns_login_cluster | first }}"

# The password to access the web page of the list of zones in the AlternC server (or other system)
alternc_slavedns_password_query:  '[?hostname==`{{ inventory_hostname }}`].password'
alternc_slavedns_password_cluster: '{{ alternc_slavedns_credentials | json_query( alternc_slavedns_password_query ) }}'
alternc_slavedns_password: "{{ '' if alternc_slavedns_password_cluster | length == 0 else alternc_slavedns_password_cluster | first }}"

# Other parameters of the alternc-slavedns script (see /usr/sbin/alternc-slavedns in the host)
alternc_slavedns_confdir: /etc/alternc/slavedns
alternc_slavedns_cachedir: /var/cache/slavedns
alternc_slavedns_bind_dir: /etc/bind/slavedns
alternc_slavedns_bind_include: /etc/bind/slavedns.conf
alternc_slavedns_wgetrc: ${HOME}/.wgetrc
alternc_slavedns_wget: wget
alternc_slavedns_wget_flags: -q
alternc_slavedns_named: "systemctl restart bind9"    # package default: "/etc/init.d/bind restart"
alternc_slavedns_named_checkconf: /usr/sbin/named-checkconf
alternc_slavedns_defaults_conf: defaults.conf
alternc_slavedns_debug: 'false'
```

Dependencies
------------

No dependencies.

Example Playbook
----------------

Considering an AlternC panel is installed on `panel.mydomain.org` and confirued with the same alternc slave variables, you can install AlternC slave DNS  an with the following playbook:

```YAML
- name: AlternC slave dns install and config
  hosts: myslave.mydomain.org
  remote_user: deploy
  become: yes

  vars:
    alternc_debconf_desktopname: panel.mydomain.org
    alternc_slavedns_credentials:
    - hostname: myslave.mydomain.org
      login: slavedns1
      password: my_very_secret
    

  roles:
  - udelarinterior.alternc_slavedns

```
License
-------

(c) Universidad de la República (UdelaR), Red de Unidades Informáticas de la UdelaR en el Interior. Licenced under GPL-v3

Author Information
------------------

Daniel Viñar Ulriksen, aka @ulvida
