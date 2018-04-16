Copy-config role
================

This role is used to push the backup running-configuration into the device. This role merges the configuration in the template file with the running-configuration of the Dell EMC Networking device.

The copy-config role requires an SSH connection for connectivity to a Dell EMC Networking device. You can use any of the built-in Dell EMC Networking OS connection variables, or the *provider* dictionary.

Installation
------------

    ansible-galaxy install Dell-Networking.dellos-copy-config

Role variables
--------------

- No predefined variables are part of this role
- Use *host_vars* or *group_vars* as part of the template file
- Configuration file is host-specific
- Copy the host-specific configuration to the respective file under the template directory in *<host_name>.j2* format
- Variables and values are case-sensitive

Connection variables
--------------------

Ansible Dell EMC Networking roles require connection information to establish communication with the nodes in your inventory. This information can exist in the Ansible *group_vars* or *host_vars* directories, or in the playbook itself.

| Key         | Required | Choices    | Description                                         |
|-------------|----------|------------|-----------------------------------------------------|
| ``host`` | yes      |            | Specifies the hostname or address used for connecting to the remote device over the specified transport.) |
| ``port`` | no       |            | Specifies the port used to build the connection to the remote device; if unspecified, the value defaults to 22 | 
| ``username`` | no       |            | Specifies the username that authenticates the CLI login for connection to the remote device; if value is unspecified, the ANSIBLE_NET_USERNAME environment variable value is used | 
| ``password`` | no       |            | Specifies the password that authenticates the connection to the remote device; if value is unspecified, the ANSIBLE_NET_PASSWORD environment variable value is used | 
| ``authorize`` | no       | yes, no\*   | Instructs the module to enter privileged mode on the remote device before sending any commands; if value is unspecified, the ANSIBLE_NET_AUTHORIZE environment variable value is used, and the device attempts to execute all commands in non-privileged mode . This key is supported only in dellos9 and dellos6. | 
| ``auth_pass`` | no       |            | Specifies the password to use if required to enter privileged mode on the remote device; if *authorize* is set to no, key not applicable; if value is unspecified, the ANSIBLE_NET_AUTH_PASS environment variable value is used . This key is supported only in dellos9 and dellos6. | 
| ``provider`` | no       |            | Passes all connection arguments as a dictonary object; all constraints (such as required, choices) must be met either by individual arguments or values in this dictionary | 

> **NOTE**: Asterisk (\*) denotes the default value if none is specified.

Dependencies
------------

The *dellos-copy-config* role is built on modules included in the core Ansible code. These modules were added in Ansible version 2.2.0.

Example playbook
----------------

This example uses the *dellos.dellos-copy-config* role to push the configuration file into the device. It creates a *hosts* file with the switch details and corresponding variables. It writes a simple playbook that only references the *dellos-copy-config* role. By including the role, you automatically get access to all of the tasks to push configuration file.

**Sample hosts file**

    leaf1 ansible_host= <ip_address> ansible_net_os_name= <OS name(dellos9 or dellos10)>

**Sample host_vars/leaf1**

    hostname: leaf1
    provider:
      host: "{{ hostname }}"
      username: xxxxx 
      password: xxxxx
    # This variable shall be applied in the below jinja template for each host by defining here
    dellos_bgp
       asn: 64801

**Sample roles/Dell-Networking.dellos-copy-config/templates/leaf1.j2**

    ! Leaf1 BGP profile on Dell OS10 switch
    snmp-server community public ro
    hash-algorithm ecmp crc
    !
    interface ethernet1/1/1:1
     no switchport
     ip address 100.1.1.2/24
     ipv6 address 2001:100:1:1::2/64
     mtu 9216
     no shutdown
    !
    interface ethernet1/1/9:1
     no switchport
     ip address 100.2.1.2/24
     ipv6 address 2001:100:2:1::2/64
     mtu 9216
     no shutdown
    !
    router bgp {{ dellos_bgp.asn }}
     bestpath as-path multipath-relax
     bestpath med missing-as-worst
     router-id 100.0.2.1
     !
     address-family ipv4 unicast
     !
     address-family ipv6 unicast
     !
     neighbor 100.1.1.1
      remote-as 64901
      no shutdown
     !
     neighbor 100.2.1.1
      remote-as 64901
      no shutdown
     !
     neighbor 2001:100:1:1::1
      remote-as 64901
      no shutdown
      !
      address-family ipv4 unicast
       no activate
       exit
      !
      address-family ipv6 unicast
       activate
       exit
     !
     neighbor 2001:100:2:1::1
      remote-as 64901
      no shutdown
      !
      address-family ipv4 unicast
       no activate
       exit
      !
      address-family ipv6 unicast
       activate
       exit
     !

**Simple playbook to setup to push configuration file into device - leaf.yaml**

    - hosts: leaf1
      roles:
         - Dell-Networking.dellos-copy-config

**Run**

    ansible-playbook -i hosts leaf.yaml

(c) 2017 Dell EMC
