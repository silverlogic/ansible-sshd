OpenSSH Server
==============

[![Build Status](https://travis-ci.org/willshersystems/ansible-sshd.svg?branch=master)](https://travis-ci.org/willshersystems/ansible-sshd) [![Ansible Galaxy](http://img.shields.io/badge/galaxy-willshersystems.sshd-660198.svg?style=flat)](https://galaxy.ansible.com/willshersystems/sshd/)

This role configures the OpenSSH daemon. It:

* By default configures the SSH daemon with the normal OS defaults.
* Works across a variety of UN*X like distributions
* Can be configured by dict or simple variables
* Supports Match sets
* Supports all sshd_config options. Templates are programmatically generated.
  (see [meta/make_option_list](meta/make_option_list))
* Tests the sshd_config before reloading sshd.

**WARNING** Misconfiguration of this role can lock you out of your server!
Please test your configuration and its interaction with your users configuration
before using in production!

**WARNING** Digital Ocean allows root with passwords via SSH on Debian and
Ubuntu. This is not the default assigned by this module - it will set
`PermitRootLogin without-password` which will allow access via SSH key but not
via simple password. If you need this functionality, be sure to set
`ssh_PermitRootLogin yes` for those hosts.

Requirements
------------

Tested on:

* Ubuntu precise, trusty
* Debian wheezy, jessie
* FreeBSD 10.1
* EL 6,7 derived distributions
* Fedora 22, 23
* OpenBSD 6.0

It will likely work on other flavours and more direct support via suitable
[vars/](vars/) files is welcome.

Role variables
---------------

Unconfigured, this role will provide a sshd_config that matches the OS default,
minus the comments and in a different order.

* sshd_skip_defaults

If set to True, don't apply default values. This means that you must have a
complete set of configuration defaults via either the sshd dict, or sshd_Key
variables. Defaults to *False*.

* sshd_manage_service

If set to False, the service/daemon won't be touched at all, i.e. will not try
to enable on boot or start or reload the service.  Defaults to *True* unless
running inside a docker container (it is assumed ansible is used during build
phase).

* sshd_allow_reload

If set to False, a reload of sshd wont happen on change. This can help with
troubleshooting. You'll need to manually reload sshd if you want to apply the
changed configuration. Defaults to the same value as ``sshd_manage_service``.

* sshd_authorized_principals_file

The path where the authorized principals files will be stored.  Only used if `sshd_authorized_principals` is also set.
Can be used to set your sshd_AuthorizedPrincipalsFile config. e.g. `sshd_AuthorizedPrincipalsFile: '{{ sshd_authorized_principals_file }}'`

* sshd_authorized_principals

A dict of lists specifying which principals are allowed to login to which users.  Each key in the dict is the name of the user.  The value of the dict is a list of principals that are allowed to login to that user. e.g.

```yaml
sshd_authorized_principals:
  root:
    - root-everywhere
    - bobby
    - ryan
```

* sshd_trusted_user_ca_keys_file

The path where the trusted user CA keys file will be stored.  Only used if `sshd_trusted_user_ca_keys` is also set.
Can be used to set your sshd_TrustedUserCAKeys config. e.g. `sshd_TrustedUserCAKeys: '{{ sshd_trusted_user_ca_keys }}'`

* sshd_trusted_user_ca_keys

A list of CA public keys to trust.

* sshd

A dict containing configuration.  e.g.

```yaml
sshd:
  Compression: delayed
  ListenAddress:
    - 0.0.0.0
```

* ssh_...

Simple variables can be used rather than a dict. Simple values override dict
values. e.g.:

```yaml
sshd_Compression: off
```

In all cases, booleans correctly rendered as yes and no in sshd configuration.
Lists can be used for multiline configuration items. e.g.

```yaml
sshd_ListenAddress:
  - 0.0.0.0
  - '::'
```

Renders as:

```
ListenAddress 0.0.0.0
ListenAddress ::
```

* sshd_match

A list of dicts for a match section. See the example playbook.

* sshd_match_1 through sshd_match_9

A list of dicts or just a dict for a Match section.

Dependencies
------------

None

Example Playbook
----------------

**DANGER!** This example is to show the range of configuration this role
provides. Running it will likely break your SSH access to the server!

```yaml
---
- hosts: all
  vars:
    sshd_skip_defaults: true
    sshd:
      Compression: true
      ListenAddress:
        - "0.0.0.0"
        - "::"
      GSSAPIAuthentication: no
      Match:
        - Condition: "Group user"
          GSSAPIAuthentication: yes
    sshd_UsePrivilegeSeparation: no
    sshd_match:
        - Condition: "Group xusers"
          X11Forwarding: yes
  roles:
    - role: willshersystems.sshd
```

Results in:

```
# Ansible managed: ...
Compression yes
GSSAPIAuthentication no
UsePrivilegeSeparation no
Match Group user
  GSSAPIAuthentication yes
Match Group xusers
  X11Forwarding yes
```

Template Generation
-------------------

The [sshd_config.j2](templates/sshd_config.j2) template is programatically
generated by the scripts in meta. New options should be added to the
options_body or options_match.

To regenerate the template, from within the meta/ directory run:
`./make_option_list >../templates/sshd_config.j2`

License
-------

LGPLv3


Author
------

Matt Willsher <matt@willsher.systems>

&copy; 2014,2015 Willsher Systems Ltd.
