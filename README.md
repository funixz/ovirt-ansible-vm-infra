oVirt Virtual Machine Infrastructure
====================================

The `ovirt.vm-infra` role manages the virtual machine infrastructure in oVirt.
This role also creates inventory of created virtual machines it defines if
`wait_for_ip` is set to `true` and state of virtual machine is `running`.
All defined virtual machines are part of `ovirt_vm` inventory group.
Role also creates `ovirt_tag_{tag_name}` groups if there are any
tags assigned to the virtual machine and places all virtual machines with that 
tag into that inventory group.

For example, see the following variable structure:

```yaml
vms:
  - name: myvm1
    tag: mytag1
    profile: myprofile

  - name: myvm2
    tag: mytag2
    profile: myprofile
```

..The role will create inventory group `ovirt_vm` in which will be both virtual
machines `myvm1` and `myvm2`. The role also creates inventory group `ovirt_tag_mytag1`
with virtual machine `myvm1` within, and inventory group `ovirt_tag_mytag2` with virtual
machine `myvm2` within.

Note
----
Please note that when installing this role from Ansible Galaxy you are instructed to run following command:

```bash
$ ansible-galaxy install ovirt.vm-infra
```

This will download the role to the directory with the same name as you specified on the
command-line, in this case `ovirt.vm-infra`. Note that it is case sensitive, so if you specify,
for example, `OVIRT.vm-infra` it will download the same role, but it will add it to the directory named
`OVIRT.vm-infra`, so you later always have to use this role with upper case prefix. Do be careful how
you specify the name of the role on command-line.

For the RPM installation, we install three legacy names `ovirt.vm-infra`, `oVirt.vm-infra` and `ovirt-vm-infra`.
As such, you can use any of those names. The documentation and examples in this repository are using the name `ovirt.vm-infra`.
The `oVirt.vm-infra` and `ovirt-vm-infra` role name styles are deprecated.

Requirements
------------

 * Ansible version 2.5 or higher
 * Python SDK version 4.2 or higher

Limitations
-----------

 * Does not support Ansible Check Mode (Dry Run / -C).

Role Variables
--------------

| Name                           | Default value |                                              |
|--------------------------------|---------------|----------------------------------------------|
| vms                            | UNDEF         | List of dictionaries with virtual machine specifications.   |
| affinity_groups                | UNDEF         | List of dictionaries with affinity group specifications.   |
| wait_for_ip                    | false         | If true, the playbook should wait for the virtual machine IP reported by the guest agent.  |
| wait_for_ip_version            | v4            | Specify which IP version to wait for (Either v4 or v6).  |
| debug_vm_create                | false         | If true, logs the tasks of the virtual machines being created. The log may contain passwords. |
| vm_infra_create_single_timeout | 180           | Time in seconds to wait for a VM to be created and started (if state is `running`). |
| vm_infra_create_poll_interval  | 15            | Polling interval. The time in seconds to wait between checking the state of a VMs.  |
| vm_infra_create_all_timeout    | vm_infra_create_single_timeout * (vms.length) | Total time (in seconds) to wait for all VMs to be created/ started. |
| vm_infra_wait_for_ip_retries   | 5             | Number of retries when checking if a VM is reporting it's IP address. |
| vm_infra_wait_for_ip_delay     | 5             | Polling interval of IP address. Time in seconds to wait between checking if a VM reports its IP address. |


The `vms` and `profile` variables can contain the following attributes. Note that if you define the same variable in both, the value in `vms` has precendence:

| Name               | Default value         |                                            |
|--------------------|-----------------------|--------------------------------------------|
| name               | UNDEF                 | Name of the virtual machine to create.     |
| tag                | UNDEF                 | Name of the tag to assign to the virtual machine. Only administrator users can use this attribute.  |
| cloud_init         | UNDEF                 | Dictionary with values for Unix-like Virtual Machine initialization using cloud-init. See the below <i>cloud_init</i> section for more detailed description. |
| cloud_init_nics    | UNDEF                 | List of dictionaries representing network interafaces to be setup by cloud-init. See the below <i>cloud_init_nics</i> section for more detailed description. |
| profile            | UNDEF                 | Dictionary specifying the virtual machine hardware. See the table below.  |
| state              | present               | Should the Virtual Machine be stopped, present or running. Takes precedence before `state` value in profile. |
| nics               | UNDEF                 | List of dictionaries specifying the NICs of the virtual machine. See below for more detailed description.   |
| cluster            | UNDEF                 | Name of the oVirt/ RHV cluster where the virtual machine will be created. |
| clone              | No                    | If `yes` then the disks of the created VM will be cloned and independent of the template.  This parameter is used only when state is `running` or `present` and the VM doesn't already exist.  |
| template           | Blank                 | Name of template from which the virtual machine is to be derived.   |
| template_version   | UNDEF                 | Version number of the template to be used for the VM. By default the latest available version of the template is used.   |
| memory             | UNDEF                 | Amount of virtual machine memory for the VM.               |
| memory_max         | UNDEF                 | Upper bound of virtual machine memory up to which memory hot-plug can be performed. |
| memory_guaranteed  | UNDEF                 | Amount of minimal guaranteed memory of the Virtual Machine. Prefix uses IEC 60027-2 standard (for example 1GiB, 1024MiB). <i>memory_guaranteed</i> parameter can't be lower than <i>memory</i> parameter. |
| cores              | 1                     | Set the number of CPU cores used by the the virtual machine. (number of cores per socket)          |
| sockets            | UNDEF                 | Set the number of virtual CPU sockets of the virtual machine. (sockets contain cores)  |
| cpu_shares         | UNDEF                 | Set the CPU shares for this Virtual Machine. |
| cpu_threads        | UNDEF                 | Set the number of CPU threads for this Virtual Machine. (number of threads per core) |
| disks              | UNDEF                 | List of dictionaries specifying the additional virtual machine disks. See below for more detailed description. (does not include the boot disk) |
| nics               | UNDEF                 | List of dictionaries specifying the NICs of the virtual machine. See below for more detailed description.   |
| custom_properties  | UNDEF                 | Properties sent to VDSM to configure various hooks.<br/> Custom properties is a list of dictionary which can have following values: <br/><i>name</i> - Name of the custom property. For example: hugepages, vhost, sap_agent, etc.<br/><i>regexp</i> - Regular expression to set for custom property.<br/><i>value</i> - Value to set for custom property. |
| high_availability  | UNDEF                 | Whether or not the node should be set highly available. |
| high_availability_priority | UNDEF                 | Indicates the priority of the virtual machine inside the run and migration queues. Virtual machines with higher priorities will be started and migrated before virtual machines with lower priorities. The value is an integer between 0 and 100. The higher the value, the higher the priority. If no value is passed, default value is set by oVirt/ RHV engine. |
| io_threads         | UNDEF                 | Number of IO threads used by virtual machine. 0 means IO threading disabled. |
| description        | UNDEF                 | Description of the Virtual Machine. |
| operating_system   | UNDEF                 | Operating system of the Virtual Machine. For example: rhel_7x64 |
| type               | UNDEF                 | Type of the Virtual Machine. Possible values: `desktop`, `server` or `high_performance` |
| graphical_console  | UNDEF                 | Assign graphical console to the virtual machine.<br/>Graphical console is a dictionary which can have following values:<br/><i>headless_mode</i> - If true disable the graphics console for this virtual machine.<br/><i>protocol</i> - 'VNC', 'Spice' or both. |
| storage_domain     | UNDEF                 | Name of the storage domain where all virtual machine disks should be created. Considered only when template is provided.|
| state              | present               | Should the Virtual Machine be `stopped`, `present` or `running`.|
| ssh_key            | UNDEF                 | SSH key to be deployed to the virtual machine. This is parameter is kept for backward compatibility and has precendece before <i>authorized_ssh_keys</i> in <i>cloud_init</i> dictionary. |
| domain             | UNDEF                 | The domain of the virtual machine. This is parameter is kept for backward compatibility and has precendece before <i>host_name</i> in <i>cloud_init</i> dictionary.|
| lease              | UNDEF                 | Name of the storage domain upon which this virtual machine lease resides. |
| root_password      | UNDEF                 | The root password of the virtual machine. This is parameter is kept for backward compatibility and has precendece before <i>root_password</i> in <i>cloud_init</i> dictionary.|

The items in `disks` list of the `profile` dictionary can contain following attributes:

| Name               | Default value  |                                              |
|--------------------|----------------|----------------------------------------------|
| size               | UNDEF          | The size of the additional disk. |
| name               | UNDEF          | The name of the additional disk.  |
| storage_domain     | UNDEF          | The name of storage domain where disk should be created. |
| interface          | UNDEF          | The interface of the disk. |
| format             | UNDEF          | Specify format of the disk.  <ul><li>cow - If set, the disk will by created as sparse disk, so space will be allocated for the volume as needed. This format is also known as thin provisioned disks</li><li>raw - If set, disk space will be allocated right away. This format is also known as preallocated disks.</li></ul> |
| bootable           | UNDEF          | True if the disk should be bootable. |

The item in `nics` list of `profile` dictionary can contain following attributes:

| Name               | Default value  |                                              |
|--------------------|----------------|----------------------------------------------|
| name               | UNDEF          | The name of the network interface.           |
| interface          | UNDEF          | Type of the network interface.               |
| mac_address        | UNDEF          | Custom MAC address of the network interface, by default it's obtained from MAC pool. |
| network            | UNDEF          | Logical network which the VM network interface should use. If network is not specified, then Empty network is used. |
| profile            | UNDEF          | Virtual network interface profile to be attached to VM network interface. |

The `affinity_groups` list can contain following attributes:

| Name               | Default value       |                                              |
|--------------------|---------------------|----------------------------------------------|
| cluster            | UNDEF (Required)    |  Name of the cluster of the affinity group.  |
| description        | UNDEF               |  Human readable description.                 |
| host_enforcing     | false               |  <ul><li>true - VM cannot start on host if it does not satisfy the `host_rule`.</li><li>false - VM will follow `host_rule` with soft enforcement.</li></ul>|
| host_rule          | UNDEF               |  <ul><li>positive - VM's in this group must run on this host.</li> <li>negative - VM's in this group may not run on this host</li></ul> |
| hosts              | UNDEF               |  List of host names assigned to this group.  |
| name               | UNDEF (Required)    |  Name of affinity group.                     |
| state              | UNDEF               |  Whether group should be present or absent.  |
| vm_enforcing       | false               |  <ul><li>true - VM cannot start if it cannot satisfy the `vm_rule`.</li><li>false - VM will follow `vm_rule` with soft enforcement.</li></ul> |
| vm_rule            | UNDEF               |  <ul><li>positive - all vms in this group try to run on the same host.</li><li>negative - all vms in this group try to run on separate hosts.</li><li>disabled - this affinity group does not take effect.</li></ul> |
| vms                | UNDEF               |  List of VM's to be assigned to this affinity group. |
| wait               | true                |  If true, the module will wait for the desired state. |

The `cloud_init` dictionary can contain following attributes:

| Name                | Description                                          |
|---------------------|------------------------------------------------------|
| host_name           | Hostname to be set to Virtual Machine when deployed. |
| timezone            | Timezone to be set to Virtual Machine when deployed. |
| user_name           | Username to be used to set password to Virtual Machine when deployed. |
| root_password       | Password to be set for user specified by user_name parameter. By default it's set for root user. |
| authorized_ssh_keys | Use this SSH keys to login to Virtual Machine. |
| regenerate_ssh_keys | If True SSH keys will be regenerated on Virtual Machine. |
| custom_script       | Cloud-init script which will be executed on Virtual Machine when deployed. This is appended to the end of the cloud-init script generated by any other options. |
| dns_servers         | DNS servers to be configured on Virtual Machine. |
| dns_search          | DNS search domains to be configured on Virtual Machine. |
| nic_boot_protocol   | Set boot protocol of the network interface of Virtual Machine. Can be one of none, dhcp or static. |
| nic_ip_address      | If boot protocol is static, set this IP address to network interface of Virtual Machine. |
| nic_netmask         | If boot protocol is static, set this netmask to network interface of Virtual Machine. |
| nic_gateway         | If boot protocol is static, set this gateway to network interface of Virtual Machine. |
| nic_name            | Set name to network interface of Virtual Machine. |
| nic_on_boot         | If True network interface will be set to start on boot. |

The `cloud_init_nics` List of dictionaries representing network interafaces to be setup by cloud init. This option is used, when user needs to setup more network interfaces via cloud init.
If one network interface is enough, user should use cloud_init nic_* parameters. cloud_init nic_* parameters are merged with cloud_init_nics parameters. Dictionary can contain following values.

| Name                | Description                                          |
|---------------------|------------------------------------------------------|
| nic_boot_protocol   | Set boot protocol of the network interface of Virtual Machine. Can be one of none, dhcp or static. |
| nic_ip_address      | If boot protocol is static, set this IP address to network interface of Virtual Machine. |
| nic_netmask         | If boot protocol is static, set this netmask to network interface of Virtual Machine. |
| nic_gateway         | If boot protocol is static, set this gateway to network interface of Virtual Machine. |
| nic_name            | Set name to network interface of Virtual Machine. |
| nic_on_boot         | If True network interface will be set to start on boot. |

Dependencies
------------

None.

Example Playbook
----------------

```yaml
---
- name: oVirt infra
  hosts: localhost
  connection: local
  gather_facts: false

  vars_files:
    # Contains encrypted `engine_password` varibale using ansible-vault
    - passwords.yml

  vars:
    engine_url: https://ovirt-engine.example.com/ovirt-engine/api
    engine_user: admin@internal
    engine_cafile: /etc/pki/ovirt-engine/ca.pem

    httpd_vm:
      cluster: production
      domain: example.com
      template: rhel7
      memory: 2GiB
      cores: 2
      ssh_key: ssh-rsa AAA...LGx user@fqdn
      disks:
        - size: 10GiB
          name: data
          storage_domain: mynfsstorage
          interface: virtio

    db_vm:
      cluster: production
      domain: example.com
      template: rhel7
      memory: 4GiB
      cores: 1
      ssh_key: ssh-rsa AAA...LGx user@fqdn
      disks:
        - size: 50GiB
          name: data
          storage_domain: mynfsstorage
          interface: virtio
      nics:
        - name: ovirtmgmt
          network: ovirtmgmt
          profile: ovirtmgmt

    vms:
      - name: postgresql-vm-0
        tag: postgresql_vm
        profile: "{{ db_vm }}"
      - name: postgresql-vm-1
        tag: postgresql_vm
        profile: "{{ db_vm }}"
      - name: apache-vm
        tag: httpd_vm
        profile: "{{ httpd_vm }}"

    affinity_groups:
      - name: db-ag
        cluster: production
        vm_enforcing: true
        vm_rule: negative
        vms:
          - postgresql-vm-0
          - postgresql-vm-1

  roles:
    - ovirt.vm-infra
```

The example below shows how to use inventory created by `ovirt.vm-infra` role in follow up play.

```yaml
---
- name: Deploy apache VM
  hosts: localhost
  connection: local
  gather_facts: false

  vars_files:
    # Contains encrypted `engine_password` varibale using ansible-vault
    - passwords.yml

  vars:
    wait_for_ip: true

    httpd_vm:
      cluster: production
      state: running
      domain: example.com
      template: rhel7
      memory: 2GiB
      cores: 2
      ssh_key: ssh-rsa AAA...LGx user@fqdn
      disks:
        - size: 10GiB
          name: data
          storage_domain: mynfsstorage
          interface: virtio

    vms:
      - name: apache-vm
        tag: apache
        profile: "{{ httpd_vm }}"

  roles:
    - ovirt.vm-infra

- name: Deploy apache on VM
  hosts: ovirt_tag_apache

  vars_files:
    - apache_vars.yml

  roles:
    - geerlingguy.apache
```

[![asciicast](https://asciinema.org/a/111662.png)](https://asciinema.org/a/111662)

License
-------

Apache License 2.0
