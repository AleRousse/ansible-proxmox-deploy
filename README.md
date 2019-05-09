Creates virtual machines and lxc container with Proxmox and installs (unattended) a distribution on it.
=========
This role is based on Inoxio proxmox_vms (https://github.com/inoxio/ansible-role-proxmox-vms)
     
Example playbook
----------------

```
- hosts: <host_name> # see hosts file
  remote_user: root

  roles:
    - role: proxmox_deploy
      proxmox:
        api_user: <encrypted user>
        api_password: <encrypted password>
        api_host: <encrypted host>
```

Example host
----------------
```
type: vm
hypervisor: pve
node: prmx0
distribution: 'debian-stretch'
locale: en_US
timezone: America/Toronto
mirror: debian.mirror.rafal.ca
user_username:
user_password: 
memory_size: 2048
net: '{"net0":"virtio,bridge=vmbr0,tag=70,firewall=1"}'
virtio: '{"virtio0":"local-lvm:24,cache=writeback,discard=on"}'
network:
  ip: 
  netmask: 
  gateway: 
  nameserver: 
  domainname: 
additional_packages:
   - curl
   - gnupg
```