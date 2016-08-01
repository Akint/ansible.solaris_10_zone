solaris\_10\_zone
=========

Creates Local Whole-Root Zone on Solaris 10 host.

Requirements
------------

Ansible 2.0+.
Python 'netaddr' module is required. It may be installed via
```sh
pip install netaddr
```

Role Variables
--------------

- **sz_host** - ansible inventory hostname of host system (mandatory)
- **sz_ip_type** - zonecfg ip-type value. Must be either '*shared*' (default) or '*exclusive*'
- **sz_ip** - IP address of the zone in the 'host/prefix' form (mandatory)
- **sz_gateway** - default gateway for then zone (excluzive-ip zone only, mandatory)
- **sz_interface** - primary interface for the zone
  - exclusive-ip zone: mandatory
  - shared-ip zone: will be auto-detected if valid **sz_ip** or **sz_network** is provided
- **sz_dns_server** - list of DNS nameservers (mandatory)
- **sz_dns_domain** - DNS domain
- **sz_dns_search** - list of DNS search domains
- **sz_root_password** - encrypted root password as in /etc/shadow. Default password - 'qw3rty'.
- **sz_root_ssh_key** - SSH public key for user 'root' (optional). If provided, SSH password login will be disabled.
- **sz_common_config** - zonecfg sections common (e.g., limits). Defaults to *'set autoboot=true;'*
- **sz_zonepath** - zonepath, defaults to /zones/*inventory_hostname*
- **sz_timezone** - timezone, defaults to UTC
- **sz_terminal** - terminal, defaults to vt100
- **sz_locale** - locale, defaults to C
- **sz_network** - network of the zone (optional). Used for auto-detection of primary interface of shared-ip zone


Instead of defining **sz_ip**, the following variables may be used:
- **sz_address** - IP address of the zone
- **sz_netmask** - netmask of the zone

Dependencies
------------

None.

Example Playbook
----------------

Inventory:
```ini
[zones]
testz sz_ip=192.0.2.11/25
testz2 sz_address=192.0.2.12 sz_network=192.0.2.0 sz_common_config="set autoboot=true; add rctl; set name=zone.cpu-cap; add value (priv=privileged,limit=100,action=deny); end;"
testz3 sz_ip_type=exclusive sz_ip=192.0.2.141/25 sz_gateway=192.0.2.129 sz_interface=aggr10001

[zones:vars]
sz_host=test-host.example.com
sz_dns_servers=['192.0.2.5', '192.0.2.6']
sz_dns_domain=example.com
sz_dns_search=['example.com']
sz_root_ssh_key="ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAEEAt1duWjzN8XOy4EP+q8zLxk1ApCoSvgZ1JqheEFz+iuPZtpox1gHYxBOnu2Ske/u2Mny9OHCLsw4X2SgKLwyFSw=="

[hosts]
test-host.example.com
```

All zones will be created on test-host.example.com.
- *testz* will be simple shared-ip zone, primary interface will be auto-detected.
- *testz2* will be simple shared-ip zone, primary interface will be auto-detected from sz\_network. Additional resource limit (*zone.cpu-cap*) is applied to the zone.
- *testz3* will be created as exclusive-ip zone, *aggr10001* will be used as primary interface.

Playbook:
```yml
---
- hosts: zones
  gather_facts: no
  remote_user: root

  roles:
    - role: solaris_10_zone
```

License
-------

BSD

Author Information
------------------

Anton Alekseyev 2016
