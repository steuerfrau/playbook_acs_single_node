# playbook_acs_single_node

Install CloudStack with KVM hypervisor on a single node. 

Implements the guides from

## Usage

## Requirements
### Necessary pre-install tasks

Needs a baseinstall according to my "playbook_baseinstall".

Needs two Linux bridges installed (not OpenVSwitch) using:
```
[root@acs-mgmt ~]# more /etc/sysconfig/network-scripts/ifcfg-eth* /etc/sysconfig/network-scripts/ifcfg-cloudbr*
::::::::::::::
/etc/sysconfig/network-scripts/ifcfg-eth0
::::::::::::::
DEVICE=eth0
ONBOOT=yes
HOTPLUG=no
BOOTPROTO=none
TYPE=Ethernet
BRIDGE=cloudbr0
::::::::::::::
/etc/sysconfig/network-scripts/ifcfg-eth1
::::::::::::::
DEVICE=eth1
ONBOOT=yes
HOTPLUG=no
BOOTPROTO=none
TYPE=Ethernet
BRIDGE=cloudbr1
::::::::::::::
/etc/sysconfig/network-scripts/ifcfg-cloudbr0
::::::::::::::
DEVICE=cloudbr0
TYPE=Bridge
ONBOOT=yes
BOOTPROTO=none
IPV6INIT=no
IPV6_AUTOCONF=no
DELAY=5
IPADDR=172.16.21.40
GATEWAY=172.16.21.1
NETMASK=255.255.255.0
DNS1=8.8.8.8
STP=yes
::::::::::::::
/etc/sysconfig/network-scripts/ifcfg-cloudbr1
::::::::::::::
DEVICE=cloudbr1
TYPE=Bridge
ONBOOT=yes
BOOTPROTO=none
IPV6INIT=no
IPV6_AUTOCONF=no
DELAY=5
STP=yes
```
### Necessary Roles
The following roles are linked as submodules and necessary for this playbook.

- role_app_acs_mgmt 
- role_app_mysql 
- role_app_nfs 
- role_hv_kvm 
- role_hv_kvm_acs_mgmt 
- role_mod_repositories 
- role_mod_selinux

## Necessary Variables

```
role_app_nfs_exports:
  /export/secondary: "*(rw,async,no_root_squash,no_subtree_check)"
  /export/primary: "*(rw,async,no_root_squash,no_subtree_check)"

role_app_mysql_target_version: "5.7"
```
