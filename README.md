# playbook_acs_single_node

Install CloudStack with KVM hypervisor on a single node. 

Implements:

- http://docs.cloudstack.apache.org/en/latest/quickinstallationguide/qig.html
- http://docs.cloudstack.apache.org/en/latest/installguide/hypervisor/kvm.html

## Requirements

### OVS Setup on Host

Prepare two OpenVSwitch switches, one with a VLAN trunk and one flat/untagged.

The VLANs will be used as following:
- VLAN 22: Public network
- VLANs 100-109 for guest traffic.

For future use the following additional VLANs can be prepared
- VLAN 20: (optional) Hypervisor access
- VLAN 21: (optional) Management network
- VLAN 23: (optional) Storage traffic

The access port is for the management networks an the hypervisor access, if those should be on an untagged network.

```
ovs-vsctl add-br ovsvlan
ovs-vsctl set port ovsvlan vlan_mode=trunk
ovs-vsctl set port ovsvlan trunks=20,21,22,23,100,101,102,103,104,105,106,107,108,109

ovs-vsctl add-br ovsflat
```
### Libvirt Networks

Prepare two libvirt networks for the two ovs switches:
```
<network>
  <name>ovsflat</name>
  <forward mode='bridge'>
  </forward>
  <bridge name='ovsflat'/>
  <virtualport type='openvswitch'/>
</network>

<network>
  <name>ovsvlan</name>
  <forward mode='bridge'>
  </forward>
  <bridge name='ovsvlan'/>
  <virtualport type='openvswitch'/>
  <portgroup name='vlan-all' default='yes'>
    <vlan>
      <tag id='20'/>
      <tag id='21'/>
      <tag id='22'/>
      <tag id='100'/>
      <tag id='101'/>
      <tag id='102'/>
      <tag id='103'/>
      <tag id='104'/>
      <tag id='105'/>
      <tag id='106'/>
      <tag id='107'/>
      <tag id='108'/>
      <tag id='109'/>
    </vlan>
  </portgroup>
</network>
```

### Start interfaces and set gateway IPs
We need gateway IPs for the public network and the management network. Guest networks are isolated through virtual routers and don't need gateway IPs.
```
ip link add link ovsvlan name ovsvlan.22 type vlan id 22
ip link set up ovsvlan.22
ip addr add 172.16.22.1/24 dev ovsvlan.22

ip link set up ovsflat
ip addr add 172.16.21.1/24 dev ovsflat
```

### Don't Forget IP Forwarding an a NAT Rule
```
echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -I POSTROUTING 1 -s 172.16.1.0/16 \! -d 172.16.1.0/16 -j MASQUERADE
```

### Prepare VM for ACS Single Node Installation

- Minumum RAM 4,5GB
- Minimum CPUs 3
- Minimum Disk 17 GB

#### NICs for VM

- 1st NIC on ovsflat
- 2nd NIC on ovsvlan

```
[mel@coco ~]$ sudo virsh attach-interface --domain acs-mgmt --type network --source ovsflat --model virtio --config
Interface attached successfully

[mel@coco ~]$ sudo virsh attach-interface --domain acs-mgmt --type network --source ovsvlan --model virtio --config
Interface attached successfully

[mel@coco ~]$ sudo virsh domiflist acs-mgmt
 Interface   Type      Source    Model    MAC
-------------------------------------------------------------
 -           bridge    natbr0    virtio   52:54:00:66:f9:08
 -           network   ovsflat   virtio   52:54:00:92:3d:f4
 -           network   ovsvlan   virtio   52:54:00:af:2c:12
 
 # Delete bridge interface using "virsh edit"
```
 In VM:

```
# more /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-cloudbr0*
::::::::::::::
/etc/sysconfig/network-scripts/ifcfg-eth0
::::::::::::::
NAME="eth0"
TYPE=Ethernet
DEVICE=eth0
ONBOOT=yes
HOTPLUG=no
BOOTPROTO=none
TYPE=Ethernet
BRIDGE=cloudbr0
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
STP=yes
::::::::::::::
/etc/sysconfig/network-scripts/ifcfg-cloudbr0.20
::::::::::::::
DEVICE=cloudbr0.20
NAME=cloudbr0.20
BOOTPROTO=none
ONPARENT=yes
IPADDR=10.0.100.40
GATEWAY=10.0.100.1
PREFIX=24
VLAN=yes
DNS1=10.0.100.1
DNS2=8.8.8.8
NM_CONTROLLED=no
```
### Use playbook_baseinstall

Prepare OS with playbook_baseinstall.

## TODO

- role_hv_kvm_acs_agent changes settings in iptables and libvirt configuration set up previously by role_hv_kvm. Check!
- role_hv_kvm_acs_agent seems to change firewallsettings. Check!

## Usage

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

Unencrypted variables in host_vars/<hostname>/general.yml:
```
role_app_nfs_exports:
  /export/secondary: "*(rw,async,no_root_squash,no_subtree_check)"
  /export/primary: "*(rw,async,no_root_squash,no_subtree_check)"

role_app_mysql_target_version: "5.7"
```

Passwords encrypted with Ansible-Vault in host_vars/<hostname>/vault_vars.yml:
```
role_app_mysql_rootpassword: <MySQL password for user root for setting up MySQL>
role_app_acs_mgmt_databasepassword: <MySQL password for user root used to configure ACS DBs> # Must be the same as above
role_app_acs_mgmt_cloudpassword: <MySQL password for user cloud, that is configured during ACS installation.>
```
## License
GNU General Public License v.3.0

## Author Information
Melanie Desaive, m.desaive@mailbox.org

